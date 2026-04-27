# CC Switch 中转 API 问题

在 SSH 远程尝试使用 VSCode Codex 插件连接中转的 API 遇到的问题

最开始是 401 API Key Error，原因是中转 API 的 url 并非官方 url，需要在 `config.toml`（远端的）进行设置，如下

```toml
model_provider = "sub2api"
model = "gpt-5.4"
model_reasoning_effort = "high"
disable_response_storage = true

[model_providers.sub2api]
name = "sub2api"
base_url = "http://101.37.159.5:8080"
wire_api = "responses"
requires_openai_auth = true
```

注意上面的 `model` 根据中转教程要改成 `gpt-5.4` 才能被正确识别转发

之后遇到问题：远程对话完全无响应，报错 `502 Upstream request failed`，但是在本地能够正常使用。

原因是远程的 API 路径和本地的可能不同，解决方法：在 base_url 中强补 `/v1`，即

```toml
[model_providers.sub2api]
name = "sub2api"
base_url = "http://101.37.159.5:8080/v1"
wire_api = "responses"
requires_openai_auth = true
```

以及，最终是关闭了远程的代理使用 Codex，原因是中转网址自动配置了代理
