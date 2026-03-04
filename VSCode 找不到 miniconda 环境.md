### 安装了 Cursor 后 VSCode 突然找不到 miniconda 环境


解决方法：多了一个 Python Environment 扩展，将其删除后恢复正常


问题本质就是：
- Python Environments 扩展接管了解释器发现机制，但 workspace 里没有注册项目 → 导致它不扫描 conda。

你禁用之后：

- 解释器发现权重新回到
Microsoft Python 扩展（ms-python.python）

- 于是自动 conda 扫描恢复正常

🔎 为什么会突然出现这个问题？

很可能是：

- 你安装了 Cursor

- Cursor 继承了 VSCode 配置

- 自动启用了 Python Environments 扩展

- 并在某个时刻生成了：`"python-envs.pythonProjects": []`

而这个字段一旦存在，就会覆盖默认行为。

🧠 以后避免踩坑的方法

如果你：

- 用 conda

- 不需要多项目环境管理

- 不需要 Dev Container 复杂环境抽象

👉 建议长期禁用 Python Environments 扩展，只保留：

- Python

- Pylance

就最稳定。
