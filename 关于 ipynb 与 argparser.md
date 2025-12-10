# 关于 ipynb 和 argparser

由于 ipynb 会向命令行传自己的参数，会导致 argparser 解析失败。最简单的方法是添加如下代码：
```py
import sys
if 'ipykernel' in sys.modules:
    sys.argv = [sys.argv[0]]  # 清理 ipykernel 的参数
```
