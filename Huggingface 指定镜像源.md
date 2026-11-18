# Huggingface 指定镜像源

```cmd
HF_ENDPOINT=https://hf-mirror.com \
.venv/bin/hf download Wan-AI/Wan2.2-I2V-A14B \
  --local-dir ./Wan2.2-T2V-A14B
```

如果失败，可能是 `huggingface_hub` 版本过旧导致重定向回了 `huggingface.co`，升级版本即可
