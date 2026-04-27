# 完全控制 Random Number Generator 以复现实验

首先，需要控制 `python + numpy + torch + CUDA`
```python
random.seed(seed)
np.random.seed(seed % (2**32 - 1))
torch.manual_seed(seed)
if torch.cuda.is_available():
    torch.cuda.manual_seed_all(seed)
```

其次，Dataset 和 DataLoader 也需要控制

```python
# 给 DataLoader 的每个 worker 设置随机种子，确保多线程数据加载的确定性
def _seed_worker(worker_id: int) -> None:
    worker_seed = torch.initial_seed() % (2**32 - 1)
    random.seed(worker_seed)
    np.random.seed(worker_seed)

train_loader = DataLoader(
    self.dataset_train,
    batch_size=self.batch_size,
    shuffle=True,
    num_workers=self.num_workers,
    pin_memory=self.pin_memory,
    drop_last=True,
    generator=_make_generator(self.train_seed),      # Dataset 的 RNG
    worker_init_fn=_seed_worker,                     # 每个进程 set 一个随机数
    # 进程是否关闭，如果设为 False 要关闭进程的话可以控制更严谨的随机数
    persistent_workers=self.persistent_workers if self.num_workers > 0 else False,
)

train_loader = DataLoader(
    dataset_train,
    batch_size=batch_size,
    shuffle=True,
    num_workers=num_workers,
    pin_memory=pin_memory,
    drop_last=True,
    generator=_make_generator(train_loader_seed),      # DataLoader 的 RNG
    worker_init_fn=_seed_worker,                       # 每个进程 set 一个随机数
    persistent_workers=train_persistent_workers,       # 进程是否关闭，如果设为 False 要关闭进程的话可以控制更严谨的随机数
)
```
