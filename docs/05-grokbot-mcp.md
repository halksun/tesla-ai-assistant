# 05 — 接到 Grok Bot（stdio MCP）

不要走「插件市场里搜 Tesla」。在 Grok Bot 上添加 **自定义 stdio MCP**：

| 字段 | 建议值 |
| --- | --- |
| Name | `tesla` |
| Transport | stdio command |
| Command | `uv` `run` `--frozen` `--directory` `/workspace/tesla-grokbot-connector` `tesla-mcp` |

挂载后，助手应能看到只读工具（名称以实际连接器为准），例如：

- 列表车辆
- 状态 / 气候 / 里程 / 软件
- 历史与报告
- `wake_and_refresh`（需确认与预算）

**不要**同时默认挂载 commands MCP。

多助手共用同一台 Grok Bot 电脑时，同一 stdio 命令可被多个 Agent 使用；密钥仍只存在该电脑私密目录。
