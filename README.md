# Tesla AI Assistant（特斯拉 AI 助手）

## Tesla AI Assistant 产品功能

### 只读查询

- 查看账号下的车辆列表（VIN 已脱敏）
- 查看电量、续航、充电状态、是否插枪、是否锁车
- 查看车内外温度、空调是否开启、设定温度
- 查看里程表，以及近 7 天、30 天或本周行驶里程
- 查看已安装软件版本和待更新状态
- 查看车辆位置（需单独同意位置权限）
- 查看本地历史曲线、周里程和低电量摘要
- 查看本月调用次数、费用估算和唤醒额度
- 在你确认后，可唤醒车辆并刷新一份新快照

完整字段清单见 [docs/read-only-fields.md](docs/read-only-fields.md)。表里只有「能读到什么」，没有真实车辆数据、完整 VIN 或精确位置。

### 车辆控制

- 开启或关闭空调
- 把座舱温度调到指定温度（两侧 15–28°C）
- 开启或关闭方向盘加热（不少车型需先开空调或气候保持）
- 车窗：透气开缝，或关闭（官方只支持这两种，不能调到任意开度）

官方接口目前不能调节风扇风速。开车、解锁、鸣笛、开关前后备箱等不在本产品范围内。

控车默认关闭，必须对每一次具体动作确认后才会执行。公开仓库只说明能力与安全流程，不含真实车辆数据、密钥，也不包含为个人车辆运行的那套私有服务。

> **说明（请先看）**：本仓库以 **特斯拉（Tesla）中国区 + Grok Bot** 为完整可跑通的例子。  
> **方法本身可迁移**：小米汽车、蔚来、小鹏、理想等只要提供官方开放平台 / 车主 API（或等价只读接口），都可以按同一思路做——车主自己的开发者应用、最小权限、密钥不出库、默认只读、控车单独强确认、例行用缓存少唤醒、给 AI Agent 写清检查清单。  
> 各品牌的注册地址、OAuth、权限名和计费不同，请以各品牌官方文档为准；本仓库不提供、也不鼓励非官方破解或越权控车。

> **给 AI Agent 读的开源流程说明**：读完本仓库后，你应能在**车主本人授权**下，复现一套特斯拉中国区 Fleet API → Grok Bot 只读监控的全流程（开发者应用、公钥托管、Partner 注册、OAuth、MCP、例行提醒）。
>
> Compatible with Tesla. **Not affiliated with, endorsed by, or sponsored by Tesla, Inc.**
> 开发者应用请勿使用听起来像官方产品的名称（示例：`GrokBot Vehicle Bridge`）。

## 先读安全（必读）

在动手前请完整阅读 [SECURITY.md](SECURITY.md)。核心红线：

1. **默认只读**：查电量、续航、锁车状态、气候、里程、软件版本。不要默认开控车。
2. **控车有危险**：空调、车窗、方向盘加热等可能造成财物损失或人身风险；必须车主对**每一次具体动作**明确同意。公开仓库里控车默认关闭，不为个人车辆保存或运行控车服务。
3. **密钥与隐私不出库**：Client Secret、Token Key、访问/刷新令牌、VIN、精确位置、车辆昵称、账号密码 **禁止**写入本仓库、Issue、聊天记录或截图明文。
4. **位置权限默认关闭**：需要时由车主单独同意后再开 `vehicle_location`。
5. **尽量不唤醒车辆**：睡着时用本地缓存；唤醒有费用与频次上限。

本仓库**不包含**任何真实凭证或真实车辆数据。示例一律为占位符。

## 本仓库是什么 / 不是什么

| 是 | 不是 |
| --- | --- |
| 中国区 + Grok Bot 端到端操作手册 | 特斯拉官方产品 |
| 给 AI Agent 的可执行检查清单 | 托管云服务 / 公开 MCP URL |
| 对社区连接器的使用说明与中国区补丁提示 | 替车主保管密钥的后端 |

**推荐上游连接器（只读 MCP 实现）**：
[naoiseirwin/Tesla-Grokbot-Connector](https://github.com/naoiseirwin/Tesla-Grokbot-Connector)

本仓库描述如何在 **Grok Bot 云电脑**上安装该连接器、完成中国区注册与授权，并配置例行任务。实现代码以 upstream 为准；中国区 Auth Host 差异见 [patches/](patches/)。

## 架构一览

```text
车主 Tesla 账号 (CN)
        │  OAuth（浏览器 Take over，勿把密码贴进聊天）
        ▼
车主自己的 Tesla 开发者应用 (developer.tesla.cn)
        │  Client ID / Secret（仅存私密配置）
        ▼
Partner 公钥 HTTPS 托管（如 GitHub Pages）
        │  register-partner
        ▼
Grok Bot 电脑上的 stdio MCP：tesla-mcp（只读）
        │  加密令牌：/workspace/.tesla-mcp/tokens.json.enc
        ▼
助手例行：轮询缓存 / 低电量提醒 / 周汇总（不默认唤醒）
```

没有公网 MCP、没有 ngrok、没有把车控暴露到互联网的设计。

## 给 AI Agent 的总流程（按顺序）

详细步骤拆在 `docs/`。摘要：

1. **确认范围** — 只读？是否要位置？是否绝不控车？写进记忆/偏好。见 [docs/00-scope-and-consent.md](docs/00-scope-and-consent.md)
2. **创建开发者应用** — [developer.tesla.cn](https://developer.tesla.cn/)（需 +86），scopes 见下表。见 [docs/01-developer-app.md](docs/01-developer-app.md)
3. **生成并托管公钥** — `/.well-known/appspecific/com.tesla.3p.public-key.pem`。见 [docs/02-public-key-github-pages.md](docs/02-public-key-github-pages.md)
4. **register-partner** — 用 client-credentials，勿把 partner token 打进聊天。见 [docs/03-partner-registration.md](docs/03-partner-registration.md)
5. **安装连接器并授权** — clone upstream，写私密 `config.env`，`tesla-mcp auth`，浏览器 Take over。见 [docs/04-install-and-oauth.md](docs/04-install-and-oauth.md)
6. **挂上 Grok Bot MCP** — stdio 命令，不用插件市场。见 [docs/05-grokbot-mcp.md](docs/05-grokbot-mcp.md)
7. **配置例行** — 不唤醒轮询、低电量、周汇总。见 [docs/06-routines.md](docs/06-routines.md)
8. **验收与故障排查** — [docs/07-verification-and-troubleshooting.md](docs/07-verification-and-troubleshooting.md)

### 建议 Scopes（只读基线）

| Scope | 是否开启 |
| --- | --- |
| `openid` / `offline_access` | 必须 |
| `vehicle_device_data` | 必须（车辆状态） |
| `vehicle_location` | 仅车主明确要求时 |
| `vehicle_charging_cmds` | 默认否（即使只要充电历史，该 scope 也允许充电控制） |
| `vehicle_cmds` | **默认否**（控车） |
| `user_data` | **否**（家庭地址等） |

### 中国区端点提示

- 开发者控制台：`https://developer.tesla.cn/`
- 区域：`TESLA_REGION=cn`
- Fleet API host：连接器内建 `fleet-api.prd.cn.vn.cloud.tesla.cn`
- 授权/换票：中国区车辆通常需要 `auth.tesla.cn`（见 [patches/README.md](patches/README.md)）。若连接器默认 `auth.tesla.com`，请在私密配置中显式设置：

```bash
TESLA_AUTH_URL=https://auth.tesla.cn/oauth2/v3/authorize
TESLA_TOKEN_URL=https://auth.tesla.cn/oauth2/v3/token
```

- 回调示例：`http://localhost:8765/callback`（个人用途；以控制台实际允许项为准）

### 计费提醒

开发者应用需绑支付方式并设消费上限（常见做法约 **$10**）。默认 $0 / 未绑卡可能导致应用不可用。唤醒等调用可能产生费用，以 Tesla 开发者文档与账单为准。

## 例行任务示例（逻辑，非真实车辆数据）

| 例行 | 建议节奏 | 行为 |
| --- | --- | --- |
| 状态轮询 | 白天每 1–2 小时 | `poll`：车在线才刷新；睡着不唤醒；无事可保持安静 |
| 低电量检查 | 每天晚上 | 仅用最近快照；低于车主设定阈值再提醒 |
| 周汇总 | 每周一次 | 里程/续航趋势/用量；缺快照时如实说明 |

具体 cron 与文案见 [docs/06-routines.md](docs/06-routines.md) 与 [examples/routine-prompts.md](examples/routine-prompts.md)。

## 控车（默认关闭）

产品功能与上方清单一致：开/关空调、设定温度、方向盘加热、车窗透气或关闭。不能调风速。为个人车辆运行的控车服务是另一套代码、另一套服务，不放在本公开仓库。

若要在自己的环境里启用同样的能力：

1. 再读 upstream [docs/COMMANDS.md](https://github.com/naoiseirwin/Tesla-Grokbot-Connector/blob/main/docs/COMMANDS.md)
2. 单独安装 commands MCP，默认 dry-run，与只读连接器分开
3. 增加 `vehicle_cmds` 授权，并按官方要求完成虚拟钥匙配对与命令签名代理
4. **每一次**控制前向车主确认具体动作

## 仓库布局

```text
README.md                 # 本说明（Agent 入口）
SECURITY.md               # 安全与隐私红线
LICENSE                   # MIT
.env.example              # 仅占位符
docs/                     # 分步手册（含只读信息表）
examples/                 # 例行文案、Agent 检查清单
patches/                  # 中国区差异说明与补丁提示（不含密钥）
```

## 免责声明

- 你需遵守 Tesla 开发者条款、当地法律与车辆使用安全要求。
- API、计费、域名策略会变更；以 Tesla 官方文档为准。
- 作者与贡献者不对滥用、密钥泄露、误控车或数据泄露负责。
- 开源此流程是为了说明只读查询与受控的车辆控制能力，不是为了传播未确认的控车或绕过安全机制。为个人车辆运行的服务不在本仓库中。

## 致谢

- 上游实现：[Tesla-Grokbot-Connector](https://github.com/naoiseirwin/Tesla-Grokbot-Connector)
- Tesla Fleet API 文档：以 [developer.tesla.com](https://developer.tesla.com/) / [developer.tesla.cn](https://developer.tesla.cn/) 为准
