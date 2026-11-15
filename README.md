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

## 更新记录

- **v1.5.3**：卡片不再依赖「英文实体 ID」—— 实体 ID 被改成中文 / 拼音，
  或卡片配置里填的旧 ID 已失效时，会按实体注册表原名与集成独有属性自动定位数据源；
  找不到电费实体时，用每日明细里的真实分摊电费求和。英文 ID 的原有行为不变。

- **v1.5.2**：电费日历卡片在手机窄屏下改为按宽度自适应，7 列日历不再溢出/被裁切；
  「本年累计」跟随翻页年份变化（翻到 2025 年显示 2025 年累计，标签同步）；
  修正文档中 `entity` 示例为「本月用电量」`this_month_total_usage`，
  并补全「卡片怎么加」的分步说明与常见问题。

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

1. 在 GitHub 新建公开仓库 `ha-csg-enhanced`，把本目录内容推上去（可选：打一个 Release，如 `v1.5.3`）。
2. 对方：HACS → 右上角菜单 → **自定义存储库** → 填仓库 URL、类别选 **Integration** → 添加。
3. 在 HACS 里搜索 `China Southern Power Grid Statistics (Enhanced)` → 下载 → **重启 Home Assistant**。
4. 设置 → 设备与服务 → 添加集成 → 搜索 `China Southern Power Grid Statistics` → 按原集成流程扫码登录。

> 这是原集成的**替换版**（domain 相同）。请勿同时通过 HACS 安装原版，否则会互相覆盖。

## 给使用者：怎么把日历卡片加到仪表盘

> 这一步是在**下载并安装集成的那台 HA** 上操作，全程不用命令行，约 3 分钟。
> 前提已完成：HACS 下载 → **重启 HA** → 设置 → 设备与服务 → 添加集成 → 扫码登录。
> 只下载文件、不登录，是不会有实体和数据的。

### 第 1 步：登记卡片资源（每台 HA 只需做一次）

设置 → 仪表盘 → 右上角 ⋮ → **资源**（Resources）→ **添加资源**：

| 字段 | 填什么 |
| --- | --- |
| URL | `/csg_stat/csg-power-card.js` |
| 资源类型 | **JavaScript 模块**（module） |

保存后按 `Ctrl` + `F5` 强制刷新浏览器。若仪表盘是 YAML 模式
（不是「由 Home Assistant 管理」），则在 `ui-lovelace.yaml` 顶层加：

```yaml
resources:
  - url: /csg_stat/csg-power-card.js
    type: module
```

> 资源用相对路径，换 HA 地址、HTTPS、手机 App 都不用改。
> 若打开 `/csg_stat/csg-power-card.js` 是 404：多半是装完**没重启 HA**，或 HA 版本低于 2024.6。

### 第 2 步：查自己的实体 ID

开发者工具 → **状态** → 搜索框输入 `this_month_total_usage`，
会看到形如 `sensor.1234567890123456_this_month_total_usage` 的实体（本月用电量），
**把整串复制下来**。每个人的缴费号不同，不能直接照抄示例。

> 注意：不要复制 `_this_year_total_usage`（本年累计）。填了它卡片会切成「年度视图」，
> 日历变成 12 个月的格子、翻月按钮变灰，看不到每天的用电量。

### 第 3 步：用「手动卡片」添加（最容易卡住的一步）

这张卡片**没有图形配置界面，不会出现在「添加卡片」的卡片选择器里**，
必须走 YAML 编辑器：

1. 打开目标仪表盘 → 右上角 **编辑仪表盘** → **添加卡片**。
2. 拉到列表最底部，选 **手动 / Manual**（部分版本叫「YAML 卡片编辑器」或「显示代码编辑器」）。
3. 把下面整段粘进去，只把 `entity` 换成第 2 步复制的实体 ID：

```yaml
type: custom:csg-power-card
entity: sensor.1234567890123456_this_month_total_usage
name: 南方电网电费日历
```

4. 点 **保存**，再点右上角 **完成**。

> `entity` **必填**：不写或写错，卡片会一直空白。卡片会自动从实体 ID 推出户号，
> 配置里不需要写账号、密码或缴费号。
> 兼容旧名称 `custom:custom-power-card-v47` / `custom:custom-power-card-v48`。
> v1.5.3 起，实体 ID 被改成拼音 / 中文、或配置里的 ID 已失效时，卡片会自动定位数据源；
> 但为兼容自动化与历史统计，仍建议保持集成生成的英文实体 ID，只改「名称」显示中文。

### 第 4 步：等数据生成

卡片读的是集成自动写入的 `/config/www/csg_daily_cost_history.json`。
首次登录后会在后台逐月抓取历史（几分钟到十几分钟），期间显示「暂无数据」属正常；
也可在开发者工具 → 操作里调用
`china_southern_power_grid_stat.query_daily_cost` 触发一次抓取。

查询页：`http://<你的HA地址>/csg_stat/csg_daily_cost.html`；
卡片底部「详细信息」按钮默认就跳到这里，不用另外配置。

### 可选参数

```yaml
type: custom:csg-power-card
entity: sensor.1234567890123456_this_month_total_usage
account: 1234567890123456                        # 多账户时指定，单账户不用写
detail_url: /csg_stat/csg_daily_cost.html        # 「详细信息」按钮目标
tier_thresholds: [5, 10, 15, 20, 25]            # 日用电量 6 档色阶阈值（度）
entity_prefix: sensor.1234567890123456_          # 实体命名特殊时手工指定前缀
```

### 常见问题

| 现象 | 原因 / 处理 |
| --- | --- |
| 添加卡片里搜不到这张卡 | 正常，卡片无图形编辑器；按第 3 步选「手动 / Manual」 |
| 卡片空白、只有标题 | `entity` 没填或填错；按第 2 步核对实体 ID |
| 日历是 12 个月格子、看不到每天用电、翻月按钮是灰的 | `entity` 填成了 `_this_year_total_usage`；改成 `_this_month_total_usage` |
| 查询页有每天耗电、卡片当月空白 | 当月日历读集成实体的本月每日明细，历史月份才读缓存 JSON；点「◀ 上月」能显示历史月即属当月尚未同步，等一次刷新或重启 |
| 资源 404 | 装完没重启 HA，或 HA < 2024.6；重启后再试 |
| 实体 ID 被改成中文 / 拼音后「本月」空白 | v1.5.3 起卡片会自动识别（含配置里旧 ID 失效的情况）；仍建议在 设置 → 设备与服务 → 实体 里把「实体 ID」改回英文，只改「名称」显示中文即可 |
| 一直「暂无数据」 | 未完成扫码登录，或历史仍在后台抓取；等几分钟或调用 query_daily_cost |
| 某些历史月份没数据 | 该月电网未返回每日日历，属数据源限制 |
| 夏天整月偏橙红 | 南网按**绝对度数**分档，用电高属正常；可用 tier_thresholds 调整 |

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

> 静态资源随集成一起安装，由集成在启动时注册到 `/csg_stat`，**无需手动往 `/config/www` 搬文件**。
> 若 `/csg_stat` 无法访问（极旧版本 HA），可把 `custom_components/china_southern_power_grid_stat/www/`
> 下两个文件复制到 `/config/www/`，然后改用 `/local/csg_daily_cost.html`
> 与 `/local/csg-power-card.js`，并在卡片里加 `detail_url: /local/csg_daily_cost.html`。

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
