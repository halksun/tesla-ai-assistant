# 03 — Partner 域名注册

前置：开发者应用已创建、公钥已可经 HTTPS 访问、Client ID/Secret 已在**私密环境**中。

```bash
export TESLA_CLIENT_ID='...'          # 注意引号，防止特殊字符被 shell 吃掉
export TESLA_CLIENT_SECRET='...'
export TESLA_REGION=cn

uv run --frozen --directory /workspace/tesla-grokbot-connector \
  tesla-mcp register-partner --domain <your-public-key-domain>
```

检查：

- 命令应能读到 Tesla 侧存储的公钥首行（对比你托管的公钥）。
- **不要**把 partner/client credentials token 打印到聊天或日志汇总里。

每个区域注册一次。只有中国区车辆时，使用 `cn` 即可。
