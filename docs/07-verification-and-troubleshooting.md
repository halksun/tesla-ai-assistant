# 07 — 验收与排障

## 验收清单

- [ ] `config.env` 不在 git 中；`.gitignore` 覆盖密钥与 `.tesla-mcp/`
- [ ] 公钥 URL 可匿名 HTTPS 访问，内容为 `BEGIN PUBLIC KEY`
- [ ] `register-partner` 成功且公钥校验通过
- [ ] OAuth 完成后存在加密 token 文件
- [ ] Grok Bot 上 `tesla` MCP 工具可见
- [ ] 列表车辆成功（展示可脱敏）
- [ ] 睡着车辆查询返回缓存而非强行 wake
- [ ] 低电量例行已配置且阈值正确
- [ ] 未安装 / 未启用控车命令 MCP（除非车主书面明确要求）

## 常见问题

| 现象 | 可能原因 |
| --- | --- |
| `unauthorized_client` | Secret 不完整、被 shell `$` 截断、未绑卡/$0 限额 |
| 授权或换票失败（CN） | 仍指向 `auth.tesla.com`，应改 `auth.tesla.cn` |
| 公钥校验失败 | Pages 未加 `.nojekyll`、路径错误、仍是私钥、域名与 Allowed Origin 不一致 |
| MCP 无工具 | stdio 命令路径错误、依赖未安装、进程起不来 |
| 总是无最新数据 | 车睡着且例行不唤醒——属预期；需最新时再经确认 wake |

排障时同样禁止把 Secret、Token、精确位置贴进聊天或 Issue。
