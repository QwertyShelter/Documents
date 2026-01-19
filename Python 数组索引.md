# Python 数组索引
```py
arr[idx][None]
```
意思是保留原维度数，得到一个 `[1, ...]` 的切片，和下面的写法是一样的效果
```py
arr[idx:idx+1]
```
`[None]` 就能够起到一个添加维度的作用，None 所在的位置就是添加维度的位置，例如
```py
arr[..., None]  ->  在 arr 末尾添加一个维度
arr.shape = [A, B, C, D]
arr[:, :, None, ...].shape = [A, B, 1, C, D]
```
