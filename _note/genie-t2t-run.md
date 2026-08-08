---
title: genie-t2t-run
date: 2026-08-08
---

## 基本命令

Dialog 0 用 `-c` / `-e` / `--prompt_file`，后续每个 dialog 各加一组 `--concurrent-config` + 输入：

```sh
./genie-t2t-run \
  --concurrent --priority 200 --log error \
  --iterations 2 \
  --profile profile.json \
  -c llm.json --prompt_file prompt.txt \
  -t embedding_table.bin,uint16,0.00031563572701998055,-32937 \
  --concurrent-config vlm.json \
  --concurrent-embed inputs_embeds.bin,uint16,0.00031563572701998055,-32937 \
  --concurrent-config vlm.json \
  --concurrent-embed inputs_embeds.bin,uint16,0.00031563572701998055,-32937 \
  > log.txt 2> timing.txt
```

对应关系：

| dialog | 来源 | profile 里的 component |
|---|---|---|
| D0 (LLM) | `-c` + `--prompt_file` | `dialog0` |
| D1 (VLM) | 第 1 组 `--concurrent-config` + `--concurrent-embed` | `dialog1` |
| D2 (VLM) | 第 2 组 `--concurrent-config` + `--concurrent-embed` | `dialog2` |

`--concurrent-config` 和输入参数的**数量必须相等**，否则 main.cpp 里直接报错退出。配对是按出现顺序做的，所以成对写更不容易出错。

## 三个容易踩的点

1. **`--profile` 的输出文件必须不存在**。parseCommandLineInput 里是「文件能打开就报错退出」，所以重跑前要 `rm -f profile.json`。你的 run_benchmark.sh 就是先 `rm -f log` 再跑，然后把 `log` 拷成 `profile.json`。

2. **VLM 用 `-t` 时 LUT 是全局共享的**。`-t`（`g_embeddingLut`）只有一份，所有 embedding dialog 共用同一张表 —— 这也是为什么示例里 D1/D2 用同一个 `vlm.json`。若 D0 是 LLM（没有 `-e`），量化参数会从第一个 VLM dialog 补齐。如果你的 VLM 配置不需要 token-to-embedding 回填（纯 prefill embedding），`-t` 那行可以整个去掉。

3. **`--concurrent-strideN` 的分隔符是冒号 `W:T`，不是 `@`**。main.cpp 第 88 行的注释写成了 `W@T`，但解析代码找的是 `:`；help 文本是对的。

## 加上调度权重

固定权重（LLM 高、两个 VLM 低）：

```sh
  --concurrent-stride 4,2,1
```

动态时间线 —— 两个 VLM 先跑，LLM 在 5s 时加入，VLM 在 20s 恢复：

```sh
  --concurrent-stride0 "0:0,1:5000" \
  --concurrent-stride1 "4:0,0:5000,4:20000" \
  --concurrent-stride2 "1:0,0:5000,1:20000"
```

注意 `--concurrent-stride`（扁平）和 `--concurrent-strideN`（时间线）可以混用：扁平值只在该 dialog 没有 `strideN` 时才生效。但 `--concurrent-stride` 与 `--concurrent-priorities` 互斥。

## 用现成脚本更省事

你仓库里已经有封装好的流程，写一行 usecase 就行：

```sh
echo "llm_2vlm: llm,vlm,vlm --iter 2 --concurrent-stride 4,2,1" > my_cases.txt
./scripts/run_benchmark.sh -f my_cases.txt -o ./results/demo --iter 2
```

它会自动展开成上面那串参数，并把 `profile.json`、`timing.txt`、`log.txt` 分目录归档，再交给 plot_perf.py 出图。模型路径在脚本顶部的 `LLM_CONFIG` / `VLM_CONFIG` / `VLM_EMBED_ARG` 处集中配置。

一点说明：上面示例里的 `uint16,0.00031563572701998055,-32937` 是直接取自 run_benchmark.sh 的 `VLM_EMBED_ARG`，属于特定模型的量化编码，换模型时需要相应替换。

> 解释下 `VLM_EMBED_ARG="inputs_embeds.bin,uint16,0.00031563572701998055,-32937"`，inputs_embeds.bin 是什么数据？

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
                                      → 按 <image> 占位符拼接 → 量化到 uint16 → inputs_embeds.bin
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

> decode 阶段生成的 token 需要经 LUT 查表转成 embedding。解释一下

## 为什么需要这一步

LLM 的输入层永远是 embedding 向量，不是 token id。prefill 时你直接给了 `inputs_embeds.bin`，所以没问题。但 decode 阶段模型每步输出的是 logits，采样后得到一个整数 token id —— 这个 id 要作为下一步的输入，必须先变回 embedding 向量。

```
prefill:  inputs_embeds.bin ──────────────→ [LLM]
decode 1:                       [LLM] → logits → sample → token id ─┐
                                                                    │ ← 断层
decode 2:  ??? ←────────────────────────────────────────────────────┘
```

纯文本模式（`-p`）下这个转换在 Genie 内部完成，因为模型图里带了 embedding 层。但 VLM 走 `-e` 路径时，输入层被替换成了直接吃向量，图里没有 embedding 查表算子，Genie 就把这一步**回调给应用层**——这就是 `-t` 存在的原因。

## 回调机制

`GenieDialog_embeddingQuery` 的第 5 个参数是 `t2eCallback`。每生成一个 token，Genie 就调用它一次，让你把向量填进 `embedding` 指针指向的缓冲区。

不需要重量化时是纯内存拷贝（main.cpp）：

```cpp
size_t lutIndex = static_cast<size_t>(token) * embeddingSize;
int8_t* embeddingSrc = static_cast<int8_t*>(g_embeddingLut.get()) + lutIndex;
std::copy(embeddingSrc, embeddingSrc + embeddingSize, embeddingDst);
```

LUT 就是一张 `[vocab_size, hidden_size]` 的扁平表，token id 直接当行号偏移。所谓"查表"就是这一行指针算术。

## 重量化路径

问题在于 LUT 和输入层往往用不同的量化编码——LUT 是权重量化的产物，输入层是激活量化的产物，两者 scale/offset 无关。所以拷贝前要做一次域转换（main.cpp）：

$$\text{dst} = \frac{S_{lut}}{S_{in}} \cdot \text{src} + \left(\frac{S_{lut}}{S_{in}} \cdot Z_{lut} - Z_{in}\right)$$

推导很直接：两边表示同一个实数值，$S_{lut}(x_{lut} + Z_{lut}) = S_{in}(x_{in} + Z_{in})$，解出 $x_{in}$ 即得。代码里的 `g_requantScale` / `g_requantOffset` 正是这两个系数，在每次 query 前算一遍。

注意模板参数 `<F, T>`：源类型 F 和目标类型 T 可以不同（比如 LUT 是 `uint16`、输入层是 `uint8`），所以 `numElements` 是按 **T** 的大小算的，越界检查却按 **F** 算：

```cpp
size_t numElements = embeddingSize / sizeof(T);
if ((lutIndex + numElements) * sizeof(F) <= g_embeddingLutSize)
```

`m_t2eCallbacks[g_lutDataType][g_inputDataType]` 是一张二维查找表，按 (LUT 类型, 输入类型) 组合选具体实例化；组合不支持会抛 `Unsupported LUT requantization`。

## 对 benchmark 的实际影响

这个回调在**每个 decode step 都会执行一次**，跑在调用 `GenieDialog_embeddingQuery` 的那个线程上，是 CPU 上的串行循环，长度为 hidden_size。并发跑多个 VLM dialog 时，N 个线程各自在做这件事，虽然相对 HTP 上的一次 forward 很轻，但它落在 CPU 侧、不受 `--concurrent-stride` 的 HTP 权重调度约束——如果观察到 decode 速率与权重比例不完全吻合，这是可能的干扰项之一。

另外前面提到的全局单例问题在这里具体化了：`calculateRequantEncodings()` 写的是全局 `g_requantScale` / `g_requantOffset`，而多个 dialog 线程共享它们。脚本里所有 VLM 用同一份 `VLM_EMBED_ARG`，算出来的值相同，所以这个竞争是良性的；但如果将来给不同 VLM 配不同编码的 embedding，这里会出现真正的数据竞争。

> 是说vlm decode 过程中，输出第二个 token 的输入也是 embedding 吗？

是的,而且不只第二个——**decode 阶段每一步的输入都是 embedding 向量**。

## 为什么

模型图的输入层是固定的。VLM 走 `-e` 路径时,那个入口张量的形状是 `[seq_len, hidden_size]` 的浮点/量化向量,图里根本没有 embedding lookup 算子。所以无论第几步,喂进去的都必须是向量,不可能是 token id。

```
prefill:   inputs_embeds.bin  ([N, hidden])  → [LLM] → logits → sample → tok_1
decode 1:  LUT[tok_1]         ([1, hidden])  → [LLM] → logits → sample → tok_2
decode 2:  LUT[tok_2]         ([1, hidden])  → [LLM] → logits → sample → tok_3
decode 3:  LUT[tok_3]         ([1, hidden])  → [LLM] → ...
```

每一步的 `LUT[tok_n]` 就是 `tokenToEmbedRequantCallback` 干的事。所以那个回调不是只调一次,而是**每生成一个 token 调一次**,总调用次数等于输出长度。

## 与纯文本模式的对比

| 输入 | prefill 输入 | decode 输入 | T2E 由谁做 |
|---|---|---|---|
| 纯文本 `-p` | token ids | token id | Genie 内部(图里有 embedding 层) |
| VLM `-e` | 预算好的 embeds | embedding 向量 | **应用层回调 + `-t` 的 LUT** |

差别只在"谁来做查表",不在"输入是什么"。纯文本模式下模型图前面挂着 embedding 层,你给 id 它自己转;VLM 模式下这层被砍掉了(因为要让图像特征能直接插进序列),转换责任就外移到了 `-t`。

## 这解释了几件事

`-t` 为什么在只有 `-e` 没有 `-p` 时仍然必需 —— 它跟 prefill 无关,纯粹是给 decode 用的。这也是为什么 main.cpp 那条校验允许 `-t` 在并发+`--concurrent-embed` 场景下脱离 `-e` 单独出现:LUT 是全 vocab 的表,与具体是哪张图片无关,多个 VLM dialog 完全可以共享同一份。

另外,如果 `-t` 没给(`g_embeddingLutSize == 0`),`t2eCallback` 会传 `nullptr` 进去。这种情况下模型要么只能做 prefill 拿一个 token,要么依赖 Genie 内部有其他回退路径 —— 具体行为得看 libGenie 里 `GenieDialog_embeddingQuery` 对空回调的处理,我没有读到那部分。