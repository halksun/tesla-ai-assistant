# 04 — 安装连接器并完成 OAuth

## Clone（示例路径）

```bash
git clone https://github.com/naoiseirwin/Tesla-Grokbot-Connector.git \
  /workspace/tesla-grokbot-connector
```

## 私密配置

目录建议：`/workspace/.tesla-mcp/`（`700`），配置文件 `config.env`（`600`）。

最小示例（占位符）：

```bash
TESLA_REGION=cn
TESLA_CLIENT_ID=your_client_id_here
TESLA_CLIENT_SECRET=your_client_secret_here
TESLA_TOKEN_KEY=your_base64_token_key_here
TESLA_AUTH_URL=https://auth.tesla.cn/oauth2/v3/authorize
TESLA_TOKEN_URL=https://auth.tesla.cn/oauth2/v3/token
# TESLA_ENABLE_LOCATION=true   # 仅当车主同意
```

生成 Token Key（在私密终端，勿把输出贴进聊天）：

```bash
uv run --frozen --directory /workspace/tesla-grokbot-connector tesla-mcp gen-key
```

## 授权

```bash
uv run --frozen --directory /workspace/tesla-grokbot-connector tesla-mcp auth
```

- 浏览器在 **Grok Bot 电脑**打开。
- 需要密码 / MFA 时，让车主 **Take over** 桌面完成；禁止要求用户把密码或验证码发到聊天。
- 成功后令牌应落在加密文件，例如：`/workspace/.tesla-mcp/tokens.json.enc`。

中国区若授权页或换票失败，优先核对 `TESLA_AUTH_URL` / `TESLA_TOKEN_URL` 是否指向 `auth.tesla.cn`，以及 Secret 是否完整、是否被 shell 截断。
