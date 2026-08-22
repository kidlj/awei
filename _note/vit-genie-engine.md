---
title: ViT Genie Engine
date: 2026-08-08
---

> **如果 vit 使用 genie engine，是否也可以设置权重共享和htp priority？当前的 vit qnn engine 可以使用 htp priority 吗？**

---

两个问题的答案不一样:**权重共享 = 不行(即使用 Genie);HTP priority = 用 Genie 就行,当前不行。**

## 一、ViT 走 Genie engine 能否权重共享?

**不能。** `QnnModelRegistry::getOrCreate` 只在 LLM 的 nsp-model.cpp 里调用。ViT 走的是另一个类 `QnnNspImageModel`,它的 nsp-image-model.cpp 里**完全没有 registry 相关调用** —— 我在该文件里搜 `QnnModelRegistry|getOrCreate|setQnnModel`,只匹配到 `initializeModel` 这个词本身,零个注册表调用。

它走的是 nsp-image-model.cpp 的 `populateGraphBinaryInfo(model_filelist, graph_switching, m_use_mmap)`(带参版本,自己加载),而非 LLM 那个借用共享 QnnModel 的无参版本。

这与配置也自洽:`htp_backend_ext_config_e_v_r_2cores.json` 里 `weight_sharing_enabled: false`。

⇒ **N 个 VLM 实例 = N 份 ViT 权重**,这一点我之前的结论对 ViT 仍然成立。要改需要在 `QnnNspImageModel::initializeModel` 里接入同样的 registry(改 libGenie)。

## 二、ViT 走 Genie engine 能否设置 HTP priority?

**可以,而且是自动生效的。** 调度器的拦截点在 `QnnNspGraph::execute()` 内部 —— nsp-graph.cpp:

```cpp
HtpConcurrencyScheduler::Guard guard(s_htp_concurrency_scheduler);
if (!g_qnn_api->graphExecute(...
```

而 ViT 的推理**正是通过同一个 `QnnNspGraph::execute`** 发出的 —— nsp-image-model.cpp:

```cpp
for (auto& nsp_graph : m_nsp_graphs) {
  if (!nsp_graph.execute(0, -1, m_inference_count, graph_switching, lazy_lora, weight_shared_lora)) {
```

只要 ViT 在 libGenie 内执行,就必然经过 `Guard`。再加上调度器是 **thread_local 键控**(`g_htp_dialog_handle`),而 ViT 与 LLM 在同一个 worker 线程上跑,所以 ViT 会**自动继承该线程已注册的权重**,无需额外为 ViT 调 API。

## 三、当前的 vit qnn engine 能用 HTP priority 吗?

**不能。** 两重原因叠加,和之前 Q6 的结论一致:

| 层面 | 状态 |
|---|---|
| `ViTGenie` | `GenieEmbedding_HasRawApis()` 在你们这版 2.46 返回 false ⇒ 直接 throw,从未被实例化 |
| 实际生效的 `qnn::ViT` | 基于 QNN sample-app,直接调 `QnnGraph_execute`,**结构上就在 libGenie 之外** |

调度器只拦 libGenie 内的 `graphExecute`,legacy 路径根本不经过 `Guard`,所以 ViT 目前是**完全不受配额约束的自由抢占**。`ark/docs/genie_htp_priority_design.md` §7.7 描述的就是这个双重失效。

## 汇总

| | 权重共享 | HTP priority |
|---|---|---|
| LLM(现状,Genie) | ✅ QnnModelRegistry 复用 | ✅ 已接入 |
| ViT(现状,legacy qnn) | ❌ ×N | ❌ 完全不受控 |
| ViT(假设切 Genie) | ❌ 仍 ×N,需改 libGenie | ✅ 自动生效,无需改代码 |

所以把 ViT 切到 Genie engine,**白捡的是优先级管控,拿不到的是权重共享**。而想让 ViT 也共享权重,得在 `QnnNspImageModel::initializeModel` 里补上 `QnnModelRegistry::getOrCreate` + `setQnnModel` + 无参 `populateGraphBinaryInfo`,照抄 LLM 那条路径。

需要的话我可以细看 `QnnNspImageModel` 和 `QnnNspModel` 在 context 所有权上的差异,评估这个改动的实际可行性。