# 只读信息表

本页列出 Tesla AI Assistant 当前只读连接器（`tesla-mcp`）能够返回的信息。依据是上游 [Tesla-Grokbot-Connector](https://github.com/naoiseirwin/Tesla-Grokbot-Connector) 的工具实现，以及 Grok Bot 上已接好的只读工具。

- 这是字段和能力清单，**不含**任何真实车辆数据、完整 VIN、精确坐标、账号或密钥。
- 车睡着时，多数查询会返回本地缓存，并注明不是实时数据；默认不会为了查询去唤醒车辆。
- 距离单位沿用 Fleet API 的英里，温度是摄氏度。
- 本仓库不包含控车代码。控车是另一套代码库。

## 当前已接通的工具

| 工具 | 会读到什么 | 会不会唤醒车辆 |
| --- | --- | --- |
| `tesla_list_vehicles` | 账号下的车辆列表（VIN 已脱敏） | 否 |
| `tesla_get_status` | 电量、续航、充电、是否插枪、锁车状态 | 否 |
| `tesla_get_climate` | 车内外温度、空调是否开启、设定温度 | 否 |
| `tesla_get_odometer` | 里程表，以及可选时间段内的行驶里程 | 否 |
| `tesla_get_software` | 已安装软件版本、待更新状态 | 否 |
| `tesla_query_history` | 本地历史曲线 | 否，也不调用 Tesla |
| `tesla_report` | 周里程、续航趋势、低电量摘要；充电汇总需另开授权 | 否 |
| `tesla_usage` | 本月调用次数、费用估算、唤醒额度 | 否，只读本地记录 |
| `tesla_get_location` | 车辆位置（需单独同意 `vehicle_location`） | 否 |
| `tesla_wake_and_refresh` | 唤醒后写入一份新快照 | **会唤醒**，但仍不是控车 |

当前已接上的连接器里，还没有登记 `tesla_get_charging_history`。充电历史要另开 `vehicle_charging_cmds` 才有；该权限同时允许充电控制，只读连接器本身不会去启动或停止充电。

## 车辆列表 `tesla_list_vehicles`

| 字段 | 含义 |
| --- | --- |
| `vin_masked` | 脱敏后的车架号 |
| `name` | 车辆名称（Tesla 返回时才有） |
| `model` | 车型（Tesla 返回时才有） |
| `state` | 在线、睡着或离线等状态 |

## 电量与锁车 `tesla_get_status`

实时或缓存。车不在线时尽量用本地快照，并带说明。

| 字段 | 含义 |
| --- | --- |
| `vin_masked` | 脱敏车架号 |
| `name` | 车辆名称 |
| `freshness` | `live`（实时）或缓存 |
| `state` | 当时车辆状态 |
| `as_of` | 这条数据的时间 |
| `battery_level_percent` | 电池电量百分比 |
| `battery_range_miles` | 额定续航（英里） |
| `est_battery_range_miles` | 预估续航（英里） |
| `charging_state` | 充电状态，例如未插枪、充电中 |
| `charge_limit_percent` | 充电上限百分比 |
| `charge_power_kw` | 当前充电功率（千瓦） |
| `minutes_to_full_charge` | 充满大约还需多少分钟 |
| `plugged_in` | 是否已插充电枪 |
| `locked` | 是否已锁车 |
| `note` | 若为缓存或尚无快照时的说明 |

## 气候 `tesla_get_climate`

| 字段 | 含义 |
| --- | --- |
| `vin_masked` / `name` / `freshness` / `state` / `as_of` | 同上 |
| `inside_temp_c` | 车内温度（摄氏度） |
| `outside_temp_c` | 车外温度（摄氏度） |
| `climate_on` | 空调是否已开 |
| `driver_set_temp_c` | 驾驶侧设定温度 |
| `passenger_set_temp_c` | 乘客侧设定温度 |

只读连接器能看到设定温度，但不能把温度改掉，也不能开关空调或调节风速。官方 Fleet API 目前也没有调节风扇风速的指令。

## 里程 `tesla_get_odometer`

| 字段 | 含义 |
| --- | --- |
| `vin_masked` / `name` / `freshness` / `state` / `as_of` | 同上 |
| `odometer_miles` | 里程表读数（英里） |
| `period` | 可选。传入 `7d`、`30d` 或 `this_week` 时，用本地快照估算该时段行驶里程 |

## 软件 `tesla_get_software`

| 字段 | 含义 |
| --- | --- |
| `vin_masked` / `name` / `freshness` / `state` / `as_of` | 同上 |
| `software_version` | 已安装版本（`vehicle_state.car_version`） |
| `update_status` | 软件更新状态（`vehicle_state.software_update.status`） |
| `update_version` | 待安装或相关版本（`software_update.version`） |

某固件若未返回对应字段，值为空。

## 位置 `tesla_get_location`（需单独同意）

默认关闭。需要车主明确同意并授予 `vehicle_location` 后才可查询。

| 字段 | 含义 |
| --- | --- |
| `vin_masked` / `name` / `freshness` / `state` / `as_of` | 同上 |
| `precision` | `coarse`（默认，约 1 公里）或 `precise` |
| `latitude` / `longitude` | 纬度、经度。精确坐标只出现在当次响应里，不会写入本地历史 |
| `note` | 精度与是否为缓存的说明 |

实时读取位置时，车上有可能出现定位图标。

## 本地历史 `tesla_query_history`

不调用 Tesla，也不唤醒车辆。可查询的指标：

| 指标 | 含义 |
| --- | --- |
| `battery_level` | 电量百分比 |
| `battery_range` | 续航（英里） |
| `odometer` | 里程表（英里） |
| `inside_temp` | 车内温度 |
| `outside_temp` | 车外温度 |

返回按 `hour`、`day` 或 `week` 分桶的时间序列。起止日期按配置时区计算。

## 汇总报告 `tesla_report`

| 报告类型 | 含义 |
| --- | --- |
| `weekly_mileage` | 近 7 天里程，按本地里程表快照把增量分摊到各天 |
| `range_trend` | 把续航按电量折算到满电，比较约 30 天与 90 天趋势 |
| `low_charge` | 未插枪且电量低于约 30% 时提醒；快照超过 6 小时会注明 |
| `charging_summary` | 对比本月与上月充电电量和费用。仅在已开启充电历史授权时可用 |

## 用量 `tesla_usage`

只读本地记录，不是特斯拉官方账单。

| 字段 | 含义 |
| --- | --- |
| `month` | 统计月份 |
| `calls` | 本月调用次数，按 data / wake / command 分类 |
| `estimated_usd` | 估算费用（美元） |
| `unit_prices_usd` | 各类单价（美元） |
| `wake_budget` | 本地唤醒额度使用情况 |
| `note` | 估算口径说明 |

## 唤醒刷新 `tesla_wake_and_refresh`

这是只读连接器里唯一会向车辆发送 `wake_up` 的工具。必须显式确认，并受频次上限约束（默认 30 分钟最多 1 次、24 小时最多 3 次）。它只用来拉一份新快照，不能开空调、加热方向盘、开关车窗或锁车。

## 本页刻意不写的内容

- 完整 VIN、精确位置、车辆真实读数、账号、Token、Client Secret
- 任何控车指令或控车代码
