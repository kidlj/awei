---
title: VLM Embed
---

`VLM_EMBED_ARG` 是 `-e` / `--concurrent-embed` 的参数，格式为 `PATH,TYPE,SCALE,OFFSET`：

| 字段 | 值 | 含义 |
|---|---|---|
| PATH | `inputs_embeds.bin` | 输入 embedding 的原始二进制文件 |
| TYPE | `uint16` | 量化数据类型（只允许 `int8`/`uint8`/`int16`/`uint16`） |
| SCALE | `0.00031563572701998055` | 量化 scale |
| OFFSET | `-32937` | 量化 zero-point |

参数个数只能是 1 或 4，见 main.cpp——写成 2 个或 3 个字段会直接报 `Invalid embedding argument`。

## inputs_embeds.bin 是什么数据

它是**已经过 vision encoder 处理、并与文本 token embedding 拼接好的完整输入序列**，即 HuggingFace 里 `model(inputs_embeds=...)` 的那个张量，而不是图片、不是 token id。

形状上是 `[seq_len, hidden_size]` 的连续数组，无文件头、无 shape 元信息、行优先紧密排列。这也是为什么代码里只用 `getFileSize()` 拿字节数（main.cpp），元素个数由 `fileSize / sizeof(TYPE)` 隐含决定 —— 文件本身不自描述，shape 的正确性完全依赖生成侧和 config 里的模型维度对齐。

典型的生成流程（在 host 上离线做）：

```
图片 → vision encoder → image embeds  ┐
                                      ├→ 按 <image> 占位符拼接 → 量化到 uint16 → inputs_embeds.bin
prompt 文本 → tokenizer → text embeds ┘
```

因为 VLM 的视觉部分不在 Genie 的 LLM 图里跑，所以 benchmark 直接喂预先算好的 embedding，跳过 vision encoder，这样测出来的就是纯 LLM decoder 的并发调度性能。

## 为什么需要 SCALE/OFFSET

用 `uint16` 存说明数据是量化过的。反量化关系是：

$$\text{real} = \text{SCALE} \times (\text{quantized} + \text{OFFSET})$$

代入 offset $-32937$，量化值 32937 对应实数 0，可见分布中心略偏于 uint16 的中点 32768。动态范围约为 $65536 \times 0.000316 \approx 20.7$。

这两个值必须与**模型输入层的量化编码完全一致**，否则数值会整体偏移，输出退化成乱码——它们不是可以随便填的调参项，而是从模型量化产物里读出来的常量。

## 与 `-t` 的配合

`-e`（输入 embedding）和 `-t`（token-to-embedding LUT）各带一套独立编码，程序在 main.cpp 计算重量化系数：

```
g_requantScale  = g_lutScale / g_inputScale;
g_requantOffset = g_requantScale * g_lutOffset - g_inputOffset;
```

用途是：prefill 阶段吃 `inputs_embeds.bin`，decode 阶段生成的 token 需要经 LUT 查表转成 embedding，而 LUT 的量化域和输入层的量化域通常不同，必须做一次重量化对齐。帮助文本里强调的"signedness must be consistent"就是这个约束——两者符号性不一致时该换算不成立。

并发模式下还有个细节：`g_inputScale` / `g_inputOffset` 是全局单例，main.cpp 会用第一个 VLM dialog 的编码去填充。所以多个 VLM dialog 若使用不同量化编码的 embedding，重量化会按第一个的参数执行——脚本里所有 VLM 共用同一个 `VLM_EMBED_ARG`，正好回避了这个问题。

一点补充：以上关于"vision encoder 离线生成"的描述是从 `inputs_embeds` 这一命名约定和 Genie 只跑 LLM 图这一事实推断的，仓库里没有生成该文件的脚本；如果需要确认具体的拼接方式和 shape，得看你们导出这个 bin 的那份 host 侧代码。