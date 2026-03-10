# `view()` 和 `reshape()` 函数的区别：

### `view()`
只是重新解释内存布局，不会复制数据

优点：
- 最快
- 不复制内存

缺点：
- 要求 contiguous

### `reshape()`
先尝试 view，如果不行就 copy 一份 tensor

优点：

- 更安全

不容易 crash

- 缺点：
- 可能会复制 tensor（略慢）
