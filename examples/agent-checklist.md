# AI Agent 执行检查清单（可复制）

在开始前向车主确认只读范围与低电量阈值，然后按序执行。每一步成功后再进入下一步。

1. [ ] 阅读本仓库 `SECURITY.md` 与 `docs/00-scope-and-consent.md`
2. [ ] 协助车主在 developer.tesla.cn 创建应用（车主本人操作登录）；只开只读 scopes
3. [ ] 生成密钥对到**私密目录**；仅把公钥发布到 HTTPS 路径
4. [ ] `register-partner`（密钥来自环境/安全输入，不进聊天）
5. [ ] Clone upstream 连接器到 Grok Bot 电脑
6. [ ] 写入私密 `config.env`（含 CN auth URL 如需要）并 `chmod 600`
7. [ ] 运行 `tesla-mcp auth`；密码/MFA 用 Take over
8. [ ] 添加 stdio MCP `tesla`
9. [ ] 烟测：list vehicles + get status（不随意 wake）
10. [ ] 配置轮询 / 低电量 / 周汇总例行
11. [ ] 最终自检：git 状态无密钥；未启用控车 MCP

失败时只报告**类别**（例如「Secret 无效」），不要回显密钥内容。
