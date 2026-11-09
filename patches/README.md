# 中国区差异与补丁提示

上游 [Tesla-Grokbot-Connector](https://github.com/naoiseirwin/Tesla-Grokbot-Connector) 已支持 `TESLA_REGION=cn` 的 Fleet host。
实战中，中国区车辆的**登录与换票**往往需要 `auth.tesla.cn`，而不是全球站 `auth.tesla.com`。

## 不改代码的配置法（优先）

在私密 `config.env` 中显式设置：

```bash
TESLA_REGION=cn
TESLA_AUTH_URL=https://auth.tesla.cn/oauth2/v3/authorize
TESLA_TOKEN_URL=https://auth.tesla.cn/oauth2/v3/token
```

## 可选代码补丁

若你希望连接器在 `region=cn` 时自动默认上述 URL，可参考同目录的
`cn-auth-hosts.patch`（基于 upstream `main` 的示意补丁）。

- 应用前请核对 upstream 是否已合并同类改动，避免重复。
- 补丁**不含**任何密钥。
- 欢迎把经验回馈给 upstream；本 playbook 只提供示意，不承诺与每一版 upstream 行号永久一致。

```bash
cd /workspace/tesla-grokbot-connector
git apply /path/to/this/repo/patches/cn-auth-hosts.patch
# 或手动按补丁内容编辑 src/tesla_mcp/config.py
```
