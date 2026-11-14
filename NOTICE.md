# NOTICE

本项目是 [linysky/ha-china-southern-power-grid-stat](https://github.com/linysky/ha-china-southern-power-grid-stat)
（版本 1.4.0.post1）的衍生版本（fork + 补丁），原项目采用 **GNU GPL v3.0**，
因此本仓库同样以 GPL-3.0 授权，完整许可证见 [LICENSE](LICENSE)。

原作者：linysky。原项目版权声明与许可在此保留。

## 本仓库相对原版的修改

1. `csg_client/__init__.py`：南网 `charge/queryDayElectricByMPoint` 返回业务码 `02` 时，
   增加参数组合探测与官方月账单降级，修复 6 个电费/阶梯实体恒为 `unknown`。
2. `sensor.py`：新增每日电费历史构建，按阶梯电价反解第一档基准价，
   用最大余数法分摊分账尾差，使每月合计逐分钱等于官方账单；
   缓存只增不删、算法版本升级时自动重算；账单为 0 元（预缴抵扣/未出账）
   的月份保留真实每日电量；新增服务 `china_southern_power_grid_stat.query_daily_cost`。
3. `__init__.py`：注册随集成分发的静态资源目录 `/csg_stat`。
4. `www/csg_daily_cost.html`：独立的每日电费查询页与 CSV 导出。
5. `www/csg-power-card.js`：Lovelace 电费日历卡片，日历格同时显示电费与电量；
   配色对齐南方电网小程序「智慧用电」的 6 档绝对日用电量色阶，
   并提供「详细信息」按钮直达每日电费查询页。

所有修改均以 GPL-3.0 提供；修改后的源码（Corresponding Source）即本仓库全部内容。

## 免责声明

本项目为个人非官方作品，与中国南方电网有限责任公司无关。所依赖的南网接口为
私有接口，随时可能变更或失效；使用者需自行承担风险。仓库中不包含任何账号、
密码、缴费号、账单数据或内网地址。
