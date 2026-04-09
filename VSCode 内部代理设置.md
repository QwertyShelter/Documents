### 设置 VSCode 内部插件的网络代理（例如 Codex）

在 `C:/Users/username/AppData/Roaming/Code/User/settings.json` 中添加如下配置即可

```json
{
    "http.proxy": "http://127.0.0.1:17890",    // 17890 为指定的端口号
    "http.proxySupport": "override",
    "http.proxyStrictSSL": false,
}
```
