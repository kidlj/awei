---
title: ViT Memory
date: 2026-08-08
---

> **vit buffer 内存的分配方式，是预先分配一段内存供每个推理请求使用还是随着每次推理请求动态创建和销毁。**

| 内存对象 | 分配时机 | 请求间是否复用 | 释放时机 |
|---|---|---:|---|
| `embedding_buffer_` 主 embedding buffer | `EmbeddingRunnerManager` 初始化时 | 是，同一个 handle 内复用 | handle / manager 销毁 |
| VIT 输入预处理 buffer | 首次请求或输入尺寸变化时懒分配 | 是，按线程复用 | 线程退出 |
| 每个 slice 的 `vit_out_tensor` | 每次处理 slice 时创建 | 否 | slice 处理结束后释放 |
| 图片数据 `Image::data` | 每个请求加载图片时创建 | 否 | 当前请求结束 |

关键证据：

1. 主 embedding buffer 在 manager 构造时创建一次：

   embedding_runner_manager.cc

   ```cpp
   embedding_buffer_ =
       Tensor::create({max_context_size_ * hidden_size_}, data_type_);
   ```

   后续请求通过已有 buffer 创建 Tensor view，而不是再次申请实际数据区：

   embedding_runner_manager.cc

   ```cpp
   embeddings =
       Tensor::create(shape, data_type_, embedding_buffer_->host<void>());
   ```

   这种 Tensor 不拥有外部 buffer，析构时不会释放底层内存：

   tensor.cc

2. VIT 输入预处理使用 `static thread_local` buffer，只在尺寸变化时 `resize`，普通请求之间只是 `clear()`：

   minicpmv_vision_embedding_runner.cc

   ```cpp
   static thread_local SliceDataBuffer buffer;
   static thread_local size_t last_pixel_size = 0;

   if (last_pixel_size != pixel_values_size) {
     buffer.resize(pixel_values_size, vpm_size, resampler_size);
     last_pixel_size = pixel_values_size;
   }

   buffer.clear();
   ```

   所以这部分是**懒分配后复用**，并且粒度是“每个线程一份”，不是所有线程共享一份。

3. 但是每个 VIT slice 的输出 Tensor 当前仍然是动态创建的：

   minicpmv_vision_embedding_runner.cc

   ```cpp
   auto vit_out_tensor = Tensor::create(
       {slice_infos[slice_id].length, hidden_size_}, vit_output_type, nullptr);
   ```

   传入 `nullptr` 会分配一块新的拥有型内存：

   tensor.cc

   ```cpp
   host_buffer_ = malloc(bytes);
   own_buffer_ = true;
   ```

   Tensor 析构时释放：

   tensor.cc

   ```cpp
   if (own_buffer_) {
     free(host_buffer_);
   }
   ```

因此，如果你关心的是 **VIT 的大块输入工作区**，答案是：

**首次使用或尺寸变化时分配，后续推理请求复用。**

如果你关心的是 **VIT 每个 slice 的输出 buffer**，答案是：

**每个 slice 推理时动态创建，处理完成后释放。**

另外，当前并行观测代码每个推理任务会创建自己的 model handle，因此主 embedding buffer 通常是**每个 handle 一份**，不是所有请求共享同一个全局 buffer。