# China Southern Power Grid Statistics (Enhanced)

南方电网电费统计集成的增强版：修复电费实体 `unknown`、提供**历史每日电费查询**、
阶梯电价计价，并内置独立查询页与 Lovelace 日历卡片。基于
[linysky 的原集成](https://github.com/linysky/ha-china-southern-power-grid-stat) 修改，
按 GPL-3.0 开源（详见 [NOTICE.md](NOTICE.md)）。

> 非官方项目，与南方电网无关；依赖的私有接口可能变更。

## 仓库地址

本包已按仓库 `https://github.com/zerostr-123/ha-csg-enhanced` 配置好：
`manifest.json` 的 `codeowners` / `documentation` / `issue_tracker`
以及 `info.md` 里的链接都已填好，**解压后直接上传即可，无需修改任何占位符**。
如果你 fork 到自己的账号，全仓库搜索 `zerostr-123` 统一替换成你的 GitHub 用户名即可。

开发与验证环境：Home Assistant 2026.8.3；静态资源注册要求 2024.6 及以上。

## 功能

| 组件 | 说明 |
| --- | --- |
| 集成 | 沿用原集成的扫码登录与 16 个实体；修复南网接口业务码 `02` 导致的电费实体 `unknown` |
| 每日电费历史 | 真实每日电量 + 阶梯电价，按官方月账单总额校准，每月合计精确等于账单；历史只增不删 |
| 服务 | `china_southern_power_grid_stat.query_daily_cost`，可按日期区间查询每日电量/电费 |
| 查询页 | `/csg_stat/csg_daily_cost.html`，支持账户、日期区间、来源标记、CSV 导出 |
| 卡片 | `/csg_stat/csg-power-card.js`，日历每格同时显示电费与电量（1 位小数）；南网同款 6 档用电量配色；底部「详细信息」按钮直达查询页 |

## 安装（分享给别人）

HACS 并没有「官方商店上架」这一步：把本仓库推到 GitHub，对方在 HACS 里添加
**自定义存储库**即可。

1. 在 GitHub 新建公开仓库 `ha-csg-enhanced`，把本目录内容推上去（可选：打一个 Release，如 `v1.5.0`）。
2. 对方：HACS → 右上角菜单 → **自定义存储库** → 填仓库 URL、类别选 **Integration** → 添加。
3. 在 HACS 里搜索 `China Southern Power Grid Statistics (Enhanced)` → 下载 → **重启 Home Assistant**。
4. 设置 → 设备与服务 → 添加集成 → 搜索 `China Southern Power Grid Statistics` → 按原集成流程扫码登录。

> 这是原集成的**替换版**（domain 相同）。请勿同时通过 HACS 安装原版，否则会互相覆盖。

## 装完即用（无需手动复制文件）

静态资源随集成一起安装，由集成在启动时注册到 `/csg_stat`：

- **查询页**：浏览器打开 `http://<你的HA地址>/csg_stat/csg_daily_cost.html`
- **卡片资源**：设置 → 仪表盘 → 右上角 → 资源 → 添加
  `/csg_stat/csg-power-card.js`，类型选 **JavaScript 模块**
  （YAML 模式则写 `url: /csg_stat/csg-power-card.js` + `type: module`）

卡片配置示例（把 `<缴费号>` 换成自己的，或直接用任一集成实体）：

```yaml
type: custom:csg-power-card
entity: sensor.<缴费号>_this_month_total_usage
```

多账户时可显式指定：

```yaml
type: custom:csg-power-card
entity: sensor.<缴费号>_this_year_total_usage
account: <缴费号>
```

卡片可选参数：

```yaml
type: custom:csg-power-card
entity: sensor.<缴费号>_this_month_total_usage
detail_url: /csg_stat/csg_daily_cost.html   # 「详细信息」按钮目标，默认即此值
tier_thresholds: [5, 10, 15, 20, 25]       # 日用电量色阶阈值（度），可选
```

日历配色对齐南方电网小程序「智慧用电」，按**绝对日用电量**分 6 档固定色
（不随当月最大/最小值浮动），色值取自官方 App 截图：

| 日用电量 | 颜色 | 色值 |
| --- | --- | --- |
| < 5 度 | 亮绿 | `#7AEB81` |
| 5–10 度 | 黄绿 | `#A6E065` |
| 10–15 度 | 金黄 | `#F7D04F` |
| 15–20 度 | 橙 | `#EA8D3D` |
| 20–25 度 | 橙红 | `#E46435` |
| ≥ 25 度 | 红 | `#DE472C` |

> 兼容旧名称：仍可用 `custom:custom-power-card-v47` 或 `custom:custom-power-card-v48`。户号从 `entity` 自动推导，
> 源码中没有任何硬编码账号。若 `/csg_stat` 无法访问（极旧版本 HA），可把
> `custom_components/china_southern_power_grid_stat/www/` 下两个文件复制到 `/config/www/`，
> 然后改用 `/local/csg_daily_cost.html` 与 `/local/csg-power-card.js`，
> 并在卡片里加 `detail_url: /local/csg_daily_cost.html`。

## 数据在哪

- 缓存文件：`/config/www/csg_daily_cost_history.json`，由集成自动生成、原子写入。
  首次启动后会后台逐月抓取历史（可能几分钟），期间查询页会显示加载失败，稍后刷新即可。
- `sensor.py` 顶部 `KEEP_ALL_HISTORY = True`：历史只增不删。
  设为 `False` 则只保留近 `DAILY_HISTORY_YEARS` 年。
- 阶梯参数默认按广东居民（夏季 260/600、非夏季 200/400 度，加价 0/0.05/0.30 元）。
  其他地区请修改 `TIER_THRESHOLDS_*` / `TIER_SURCHARGES`，并把 `ALGO_VERSION` **加 1**
  以触发旧缓存全量重算（会复用已抓到的每日电量，不重复请求南网）。

## 隐私

本仓库**不含**任何账号、密码、缴费号、电费账单数据、Token、内网 IP 或本机路径；
所有个人数据都在使用者自己的 Home Assistant 内生成并保存，不会上传到任何地方。
示例中的户号 `1234567890123456` 是占位符。

## 许可

GPL-3.0，见 [LICENSE](LICENSE) 与 [NOTICE.md](NOTICE.md)。
