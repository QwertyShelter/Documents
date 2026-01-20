# DDP 多卡并行问题

## 问题描述：
单卡运行没有问题，但是多卡运行出现

```cmd
[rank0]: RuntimeError: Expected to mark a variable ready only once. This error is caused by one of the following reasons:
1) Use of a module parameter outside the forward function. Please make sure model parameters are not shared across multiple concurrent forward-backward passes. or try to use _set_static_graph() as a workaround if this module graph does not change during training loop.
2) Reused parameters in multiple reentrant backward passes. For example, if you use multiple checkpoint functions to wrap the same part of your model, it would result in the same set of parameters been used by different reentrant backward passes multiple times, and hence marking a variable ready multiple times. DDP does not support such use cases in default. You can try to use _set_static_graph() as a workaround if your module graph does not change over iterations.
```

## 问题原因：
同一个 parameter 参与了两条（或更多条）反向传播路径，在**单卡**下 PyTorch 会把这些梯度**累加**，但是 DDP 的 reducer 逻辑是：

一个 parameter，在一次 backward 中，只能被 “ready” 一次
- 第一次 rasterization backward → mark ready
- 第二次 rasterization backward → 再次 mark ready
- 💥 DDP 直接抛异常



## 解决方法：
暂时是使用 `detach()` 删除相关模块梯度
