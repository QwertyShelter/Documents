### `torch.optim.lr_scheduler.SequentialLR` 会重置 step 计数器

对比下面两段代码
```py
# SequentialLR 版本
warm_up_steps = max(1, optimizer_cfg.warm_up_steps)
cosine_steps = max(1, max_steps)
warm_up = torch.optim.lr_scheduler.LinearLR(
    optimizer,
    start_factor=1 / warm_up_steps,
    end_factor=1.0,
    total_iters=warm_up_steps,
)
cosine = torch.optim.lr_scheduler.CosineAnnealingLR(
    optimizer,
    T_max=cosine_steps,
    eta_min=optimizer_cfg.lr * 0.1,
)
scheduler = torch.optim.lr_scheduler.SequentialLR(
    optimizer,
    schedulers=[warm_up, cosine],
    milestones=[warm_up_steps],
)

# 手动实现版本
warm_up_steps = max(1, optimizer_cfg.warm_up_steps)
cosine_steps = max(1, max_steps)

def lr_lambda(step: int) -> float:
    if step < warm_up_steps:
        # 与原始 LinearLR 行为一致：从 1 / warm_up_steps 线性上升到 1.0
        alpha = step / warm_up_steps
        warmup_factor = (1.0 / warm_up_steps) + alpha * (1.0 - 1.0 / warm_up_steps)
        return warmup_factor

    cosine_step = min(step - warm_up_steps, cosine_steps)
    cosine_factor = 0.1 + 0.9 * (1.0 + math.cos(math.pi * cosine_step / cosine_steps)) / 2.0
    extra_decay = 0.1 if step > decay_step else 1.0
    return cosine_factor * extra_decay

scheduler = torch.optim.lr_scheduler.LambdaLR(optimizer, lr_lambda=lr_lambda)
```
这里面看似 `cosine_steps` 是一样的，都是直接作为 cosineLR 的 `T_max`，但是 `SequentialLR` 内部维护的其实是一个局部计数器（Local Step），切换 Scheduler 会让 step 清零

因此手写版本应该改成
```py
warm_up_steps = max(1, optimizer_cfg.warm_up_steps)
# SequentialLR 会重置计数器，手动实现版本要自己修正 T_max
cosine_steps = max(1, max_steps - warm_up_steps)

def lr_lambda(step: int) -> float:
    if step < warm_up_steps:
        # 与原始 LinearLR 行为一致：从 1 / warm_up_steps 线性上升到 1.0
        alpha = step / warm_up_steps
        warmup_factor = (1.0 / warm_up_steps) + alpha * (1.0 - 1.0 / warm_up_steps)
        return warmup_factor
  
    cosine_step = min(step - warm_up_steps, cosine_steps)
    cosine_factor = 0.1 + 0.9 * (1.0 + math.cos(math.pi * cosine_step / cosine_steps)) / 2.0
    extra_decay = 0.1 if step > decay_step else 1.0
    return cosine_factor * extra_decay

scheduler = torch.optim.lr_scheduler.LambdaLR(optimizer, lr_lambda=lr_lambda)
```


### CosineAnnealingLR 递推更新模式

`CosineAnnealingLR` 在 PyTorch 中的实现并不是纯粹的 closed-form 计算，而是采用了基于当前 `learning_rate` 的递推更新机制。因此在从 checkpoint 恢复训练后，即使修改 `T_max`，`learning_rate` 通常会从当前值平滑延续，而不会出现明显跳变。

相比之下，基于 `LambdaLR` 的手动实现通常是显式函数形式（lr = f(epoch)），恢复训练时会根据新的调度函数重新计算当前 epoch 对应的 `learning_rate`，因此当 `T_max` 改变时会导致学习率不连续。

需要注意的是，`CosineAnnealingLR` 的这种“连续性”来源于其状态依赖更新，而不是严格遵循新的 cosine 曲线，因此在理论上其学习率轨迹已经偏离了修改后的目标调度函数。
