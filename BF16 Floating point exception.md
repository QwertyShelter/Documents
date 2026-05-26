# BF16 Floating point exception 问题

BF16 动态范围接近 FP32，但降低了精度，可能导致小梯度直接消失或者 attention softmax 不稳定，但是好处是可以直接**节省一半的显存**，同时**大幅增快训练速度**

因此工业界的标准做法是：
```
Backbone -> BF16
Sensitive heads -> FP32
```
