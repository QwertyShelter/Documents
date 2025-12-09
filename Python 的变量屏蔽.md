# Python 的变量屏蔽
```py
a = 10

def fun():
  cal(a)
```
上面是能正常工作的，a 会作为全局变量被传进来。但是：

```py
a = 10

def fun():
  a = a + 10
  cal(a)
```
这样就会报错，因为：

Python 首先在 局部作用域 创建了一个名为 `a` 的变量

但在等号右边计算 `a + 10` 时，它发现当前作用域已经有了 `a` 变量（虽然还没赋值）

根据 Python 的 LEGB 规则，它使用局部作用域的 `a`

但此时局部 `a` 还没有值，所以报错：`UnboundLocalError: local variable 'a' referenced before assignment`
