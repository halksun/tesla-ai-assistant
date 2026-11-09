# 01 — 创建 Tesla 中国区开发者应用

1. 打开 [developer.tesla.cn](https://developer.tesla.cn/)（通常需要 +86 手机号）。
2. 创建应用，显示名避免官方感，例如：`GrokBot Vehicle Bridge`。
3. 启用 scopes（只读基线）：
   - `openid`
   - `offline_access`
   - `vehicle_device_data`
4. 如车主要位置，再加 `vehicle_location`（否则不加）。
5. 不要启用 `user_data`。不要启用 `vehicle_cmds`（除非单独走控车文档）。
6. Redirect URI 增加：`http://localhost:8765/callback`（若控制台还要求 HTTPS 回调，可加 GitHub Pages 上的 `callback.html`）。
7. Allowed Origin：填你将托管公钥的域名根（例如 `https://<user>.github.io`）。
8. 在控制台绑定支付方式，设置较小消费上限（常见约 $10）。$0 上限常导致调用失败。
9. 复制 **Client ID** 与 **Client Secret** 到安全存储；**不要**提交到 git，不要发到公开 Issue。

Agent 注意：Secret 中可能含 `$` 等 shell 特殊字符；写入文件或环境变量时避免未加引号的 shell 展开。
