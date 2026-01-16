# Python 数组索引
```py
arr[idx][None]
```
意思是保留原维度数，得到一个 `[1, ...]` 的切片，和下面的写法是一样的效果
```py
arr[idx:idx+1]
```
