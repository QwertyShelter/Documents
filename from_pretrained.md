# 更改为从本地加载模型

```python
model = AnySplat.from_pretrained("checkpoints", local_files_only=True)
```

其中，`checkpoints` 为文件夹路径 `dir`，其中应该包含 `config.json` 和 `model.pt`（或者 model.safetensor）

如果卡住了，检查模型初始化有没有递归地调用，单步调试检查
