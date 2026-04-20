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
