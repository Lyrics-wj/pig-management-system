# WEB监管端 - 决策分析 - 市场贸易分析

> **文档版本**：v1.1
> **创建时间**：2026-03-10
> **最后更新**：2026-03-10
> **文档状态**：待评审
> **关联企业端PRD**：`doc_prd/WEB/企业端/企业信息管理/养殖信息管理/WEB企业端-养殖信息管理-畜禽及其产品销售记录.prd.md`、`doc_prd/WEB/企业端/企业信息管理/养殖信息管理/WEB企业端-养殖信息管理-饲料和饲料添加剂购进记录.prd.md`、`doc_prd/WEB/企业端/企业信息管理/养殖信息管理/WEB企业端-养殖信息管理-兽药购进记录.prd.md`
> **关联监管端PRD**：`doc_prd/WEB/监管端/决策分析/养殖分析/WEB监管端-养殖分析.prd.md`
> **关联需求文档**：`需求梳理/第4阶段-WEB监管端数据分析与大屏/WEB监管端-市场贸易分析.md`
> **角色与权限基准**：`需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md`
> **变更记录**：v1.1 — 数据采集方案由"外部 API/爬取"变更为"通过 DeepSeek 大模型联网搜索获取实时市场行情数据，每日两次定时采集并持久化到本地"

---

## 3.1 基本信息

- **功能名称**：市场贸易分析
- **关联现有模块/功能**：
  - 用户认证：复用现有 `/login`、`/getInfo`、`/getRouters` 接口与 JWT 机制
  - 权限管理：复用现有角色权限体系，新增"市场贸易分析"权限标识 `admin:analysis:market`
  - 企业端销售记录：`pig_sales_record` 表提供本县生猪收购价数据（各企业畜禽及其产品销售记录中的销售单价）
  - 企业端饲料购进记录：`pig_feed_purchase_record` 表提供本县饲料成本数据
  - 企业端兽药购进记录：`pig_vet_drug_purchase_record` 表提供本县兽药成本数据
  - 系统参数配置：`sys_config` 表存储猪粮比预警阈值参数及 DeepSeek API 配置参数
  - **DeepSeek 大模型服务**：通过 DeepSeek API（`deepseek-chat` 模型，DeepSeek-V3.2）联网搜索能力获取实时市场行情数据，替代传统爬虫/第三方 API 方案。API 兼容 OpenAI 格式，Base URL `https://api.deepseek.com`，128K 上下文长度
  - 养殖分析模块：同属决策分析板块，养殖分析中的成本排行数据可与市场价格对照分析
  - 产业总览大屏：市场行情核心指标会被大屏引用展示
  - 图表组件：ECharts（与大屏及其他分析模块保持技术栈统一，中国地图使用 ECharts 地图组件）
  - 参考文档：`doc_spec/现有业务说明.spec.md`、DeepSeek API 官方文档 `https://api-docs.deepseek.com/zh-cn/`
- **目标用户/场景**：

| 用户角色 | 角色编码（role_key） | 使用场景 | 主要需求 |
|---------|---------------------|---------|---------|
| 县级管理员 | `county_admin` | 产业政策参考、市场行情研判 | 查看市场行情、切换筛选条件、导出分析报告 |
| 县级业务人员 | `county_staff` | 市场研判、数据报告 | 查看市场行情、切换筛选条件、导出分析报告 |
| 乡镇政府业务人员 | `town_gov_staff` | 了解市场动态 | 查看市场行情（不可导出） |
| 乡镇畜牧兽医站 | `town_vet_station` | 养殖技术指导参考 | 查看市场行情（不可导出） |

> 角色编码引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1 角色总览。
>
> 市场贸易分析展示的是全国/区域级市场数据，不涉及本县数据权限过滤，所有监管端角色看到的数据相同。仅"本县收购价"部分与本县企业端销售记录相关。

## 3.2 目标与范围

**目标**：

1. 通过 DeepSeek 大模型联网搜索能力获取实时市场行情数据，展示生猪（外三元）、玉米（14%水分）、豆粕（43%蛋白）、白条猪肉四大品种的市场价格走势和交易趋势。
2. 建立"每日两次定时采集 + 本地持久化"的数据更新机制，确保行情数据时效性（上午、下午各采集一次）。
3. 提供今日市场行情速览 + 7 个分析卡片的仪表盘式布局，帮助监管部门直观了解市场动态。
4. 支持全国/四川省/川东北片区的价格区域切换和多种时间范围筛选。
5. 提供猪粮比价分析，标注盈亏平衡预警区间，辅助研判养殖企业盈亏风险。
6. 提供全国各省猪肉批发价格地图（热力图），直观展示各省价格差异。
7. 支持多品种价格对比分析，观察品种间价格联动关系。
8. 支持市场分析报告导出（Excel）。

**非目标（Non-Goals）**：

- ❌ 不做价格预测模型（价格预测可在后续 Deepseek 智能服务中基于本地持久化的历史数据实现）。
- ❌ 不做实时价格推送或告警通知（数据由定时任务每日两次采集，不做秒级实时推送）。
- ❌ 不做自定义统计维度或拖拽式报表功能。
- ❌ 不在本模块内展示养殖生产数据分析（该职责属于"养殖分析"模块）。
- ❌ 不导出图表图片，仅导出数据表格（Excel 格式）。
- ❌ 不做数据源管理界面（DeepSeek API Key 通过系统参数配置管理，不做独立的数据源管理页面）。
- ❌ 不在前端暴露 DeepSeek API Key 或调用日志详情（API 调用完全在后端完成）。

## 3.3 现状与复用

**现状简述**：

- 目前系统无市场行情数据和市场分析功能，需全部新建市场行情数据表和分析接口。
- 外部市场行情数据（四个品种的全国/四川价格）通过 DeepSeek 大模型的联网搜索能力获取。后端定时任务每日两次调用 DeepSeek API，将大模型返回的结构化市场数据持久化到本地数据库。
- DeepSeek API 使用 OpenAI 兼容格式（`POST /chat/completions`），模型 `deepseek-chat`（DeepSeek-V3.2），支持 JSON Output 模式确保返回结构化数据。
- 本县生猪收购价可从企业端畜禽及其产品销售记录（`pig_sales_record`）中的销售单价聚合得出。
- 本县养殖成本线可从企业端饲料购进记录（`pig_feed_purchase_record`）和兽药购进记录（`pig_vet_drug_purchase_record`）的采购金额估算。
- 养殖分析模块（同属决策分析板块）已建立 `/admin/analysis/` 接口路径体系，市场贸易分析沿用该路径体系。

**复用检查结论**：

**1. 接口复用检查（对照 `doc_spec/现有业务说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户认证（登录） | POST `/login` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取用户信息 | GET `/getInfo` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取路由菜单 | GET `/getRouters` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 系统参数接口 | GET `/system/config/configKey/{configKey}` | ✅ 直接复用 | 获取猪粮比预警阈值参数配置，见现有业务说明.spec.md §三 |
| 今日市场行情速览接口 | 无 | 🆕 新增 `/admin/analysis/market/today` | 无现有接口满足 |
| 生猪交易趋势接口 | 无 | 🆕 新增 `/admin/analysis/market/pig-trend` | 无现有接口满足 |
| 玉米交易趋势接口 | 无 | 🆕 新增 `/admin/analysis/market/corn-trend` | 无现有接口满足 |
| 豆粕交易趋势接口 | 无 | 🆕 新增 `/admin/analysis/market/soybean-trend` | 无现有接口满足 |
| 白条猪肉交易趋势接口 | 无 | 🆕 新增 `/admin/analysis/market/pork-trend` | 无现有接口满足 |
| 猪粮比价分析接口 | 无 | 🆕 新增 `/admin/analysis/market/pig-grain-ratio` | 无现有接口满足 |
| 全国猪肉价格地图接口 | 无 | 🆕 新增 `/admin/analysis/market/price-map` | 无现有接口满足 |
| 价格对比分析接口 | 无 | 🆕 新增 `/admin/analysis/market/price-compare` | 无现有接口满足 |
| 导出市场分析报告接口 | 无 | 🆕 新增 POST `/admin/analysis/market/export` | 无现有接口满足 |
| 手动触发数据采集接口 | 无 | 🆕 新增 POST `/admin/analysis/market/fetch-data` | 县级管理员手动触发 DeepSeek 数据采集 |
| 数据采集日志查询接口 | 无 | 🆕 新增 GET `/admin/analysis/market/fetch-log` | 查看 DeepSeek API 调用记录和采集状态 |

**2. 数据库表复用检查（对照 `doc_spec/现有数据库说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 市场行情日报表 | 无 | 🆕 新建表 `pig_market_price_daily` | 无现有表满足，需新建，存储四个品种的全国/四川/本县每日价格数据 |
| 省份猪肉价格表 | 无 | 🆕 新建表 `pig_market_price_province` | 无现有表满足，需新建，存储全国各省猪肉批发价格（按周更新） |
| DeepSeek 采集日志表 | 无 | 🆕 新建表 `pig_market_fetch_log` | 无现有表满足，需新建，记录每次 DeepSeek API 调用的请求/响应/状态 |
| 畜禽及其产品销售记录表 | `pig_sales_record` | ✅ 直接复用 | 本县生猪收购价数据来源（取已入库销售记录中的销售单价），WEB企业端-畜禽及其产品销售记录 PRD 定义 |
| 饲料购进记录表 | `pig_feed_purchase_record` | ✅ 直接复用 | 本县饲料成本参考数据来源，WEB企业端-饲料和饲料添加剂购进记录 PRD 定义 |
| 兽药购进记录表 | `pig_vet_drug_purchase_record` | ✅ 直接复用 | 本县兽药成本参考数据来源，WEB企业端-兽药购进记录 PRD 定义 |
| 系统参数配置表 | `sys_config` | ✅ 直接复用 | 存储猪粮比预警阈值参数及 DeepSeek API 配置，见现有数据库说明.spec.md |
| 定时任务表 | `sys_job` | ✅ 直接复用 | 注册市场数据采集定时任务（每日两次），见现有数据库说明.spec.md |

## 3.4 功能需求

**功能列表**：

| 功能点 | 优先级 | 说明 |
|-------|-------|------|
| 全局筛选条件 | P0 | 时间范围、价格区域 |
| 今日市场行情速览（统计卡片） | P0 | 4 个品种的今日价格/涨跌/迷你折线 |
| 生猪（外三元）交易趋势 | P0 | 折线图（全国/四川/本县）+ 辅助分析线 + 数据表格 |
| 玉米（14%水分）交易趋势 | P0 | 折线图（全国/四川）+ 数据表格 |
| 豆粕（43%蛋白）交易趋势 | P0 | 折线图（全国/四川）+ 数据表格 |
| 白条猪肉交易趋势 | P0 | 折线图（全国/四川）+ 数据表格 |
| 猪粮比价分析 | P0 | 面积折线图 + 预警区间标注 |
| 全国猪肉批发价格地图 | P1 | 中国地图热力图（按省着色） |
| 价格对比分析 | P1 | 多品种涨跌幅叠加折线图 |
| 市场分析报告导出 | P1 | Excel（仅县级角色可操作） |

**关键流程**：

```
进入市场贸易分析页面
    ↓
系统按默认筛选条件（近半年、全国）加载所有分析卡片
    ↓
页面顶部展示今日市场行情速览（4 个品种行情卡片）
    ↓
用户可调整全局筛选条件（时间范围、价格区域）→ 所有趋势图卡片同步刷新
    ↓
用户可查看各品种交易趋势图表（hover 查看数值），可展开可折叠数据表格
    ↓
用户可查看猪粮比价分析、价格地图、价格对比分析等深度分析卡片
    ↓
用户可导出市场分析报告（Excel，仅县级角色可操作）
```

### 3.4.1 功能流程逻辑

#### 全局筛选条件

| 序号 | 筛选条件 | 控件类型 | 是否必填 | 默认值 | 说明 |
|------|---------|---------|---------|-------|------|
| 1 | 时间范围 | 日期区间选择器 | 选填 | 近半年（当前日期往前推 6 个月） | 支持快捷切换：近一周、近一月、近一季、近半年、近一年 |
| 2 | 价格区域 | 下拉选择（单选） | 选填 | 全国 | 枚举值：`national` 全国、`sichuan` 四川省、`chuandongbei` 川东北片区，仅此 3 种取值。默认 `national`。该筛选影响趋势图中主线展示的数据区域。 |

全局筛选条件变更后，页面所有趋势分析卡片（卡片二至卡片八）**即时同步刷新**数据，无需额外点击"查询"按钮。卡片一（今日行情速览）始终展示全国数据，不受价格区域筛选影响。

---

#### 卡片一：今日市场行情速览

**展示形式**：页面顶部 4 个横向排列的行情卡片

| 序号 | 品种 | 品种编码 | 展示内容 | 价格单位 | 说明 |
|------|------|---------|---------|---------|------|
| 1 | 生猪（外三元） | `pig_outer` | 今日价格、日涨跌（元 + 涨跌箭头）、周涨跌幅（%） | 元/公斤 | 外三元为我国生猪市场主流品种报价基准 |
| 2 | 玉米（14%水分） | `corn` | 今日价格、日涨跌（元 + 涨跌箭头）、周涨跌幅（%） | 元/吨 | 主要饲料原料 |
| 3 | 豆粕（43%蛋白） | `soybean_meal` | 今日价格、日涨跌（元 + 涨跌箭头）、周涨跌幅（%） | 元/吨 | 主要蛋白饲料原料 |
| 4 | 白条猪肉 | `pork` | 今日价格、日涨跌（元 + 涨跌箭头）、周涨跌幅（%） | 元/公斤 | 终端消费市场价格 |

**视觉规范**：

| 状态 | 数字颜色 | 箭头符号 | 说明 |
|------|---------|---------|------|
| 上涨 | 红色 `#f5222d` | ↑ | 日涨跌 > 0 |
| 下跌 | 绿色 `#52c41a` | ↓ | 日涨跌 < 0 |
| 持平 | 灰色 `#8c8c8c` | — | 日涨跌 = 0 |

**迷你折线图（sparkline）**：每个卡片内附一条迷你折线图，展示近 7 天价格走势。折线高度 30px，无坐标轴，仅展示趋势形态。上涨趋势线条颜色 `#f5222d`（红），下跌趋势 `#52c41a`（绿），持平 `#8c8c8c`（灰）。

**数据更新时间**：行情区域右上角标注最后一次数据采集时间，格式"数据更新：YYYY-MM-DD HH:mm"。

**数据源**：`pig_market_price_daily` 表，取 `price_region = 'national'`（全国）且 `price_date` 为最新有数据的日期。数据由后端定时任务通过 DeepSeek API 联网搜索每日两次采集入库。

**无数据处理**：当 DeepSeek API 调用失败或大模型返回数据无法解析导致最新日期数据缺失时，展示最后一次成功采集的数据，数据更新时间标注变为红色文字并附加提示"（数据更新异常，展示最近可用数据）"。

---

#### 卡片二：生猪（外三元）交易趋势

**展示形式**：折线图 + 辅助分析线 + 可折叠数据表格

**折线图配置**：
- X 轴：时间（按日/周/月，与全局筛选的时间范围联动；时间范围 ≤ 90 天按日展示，91-365 天按周展示，> 365 天按月展示）
- Y 轴：价格（元/公斤）
- 展示曲线（共 3 条）：

| 曲线名称 | 颜色与样式 | 数据来源 | 说明 |
|---------|----------|---------|------|
| 全国均价 | 蓝色实线 `#1890ff`，线宽 2px | `pig_market_price_daily`（`product_code = 'pig_outer'`、`price_region = 'national'`） | 主线，始终展示 |
| 四川省均价 | 橙色虚线 `#fa8c16`，线宽 1.5px | `pig_market_price_daily`（`product_code = 'pig_outer'`、`price_region = 'sichuan'`） | 辅线，始终展示 |
| 本县收购价 | 绿色虚线 `#52c41a`，线宽 1.5px | `pig_sales_record`（`status = '4'`、`del_flag = '0'`），按 `sales_date` 分组取 `sale_price` 均值 | 辅线，有数据时展示；无数据时图例标注"本县数据暂未采集"，该曲线不展示 |

- 关键价格节点标注：自动标注所选时间范围内的年度最高点和最低点（以蓝色实心圆点 + 价格数字标注）
- hover Tooltip 展示：日期、全国均价、四川省均价、本县价格（无数据显示"—"）

**辅助分析线**：

| 辅助线名称 | 颜色与样式 | 计算逻辑 | 说明 |
|----------|----------|---------|------|
| 养殖成本线 | 红色水平虚线 `#f5222d`，线宽 1px | 基于 `pig_feed_purchase_record`（`purchase_amount`）和 `pig_vet_drug_purchase_record`（`purchase_amount`）的投入品合计 ÷ 出栏头数（`pig_outbound_record.outbound_quantity`），折算为元/公斤（÷ 出栏均重，出栏均重取系统参数 `market.avg_slaughter_weight`，默认 120 公斤） | 当生猪价格低于成本线时，成本线与价格线之间区域填充半透明红色 `rgba(245,34,45,0.1)` |
| 猪粮比 6:1 参考线 | 灰色水平虚线 `#bfbfbf`，线宽 1px | 当日玉米全国均价（元/吨 ÷ 1000 → 元/公斤）× 6 | 标注文字"猪粮比6:1线"；该线为动态值，随时间变化 |

**可折叠数据表格**（默认收起）：

| 列名 | 数据来源 | 说明 |
|------|---------|------|
| 日期 | `price_date` | 格式 YYYY-MM-DD |
| 全国均价（元/kg） | `pig_market_price_daily`（national） | 保留 2 位小数 |
| 四川省均价（元/kg） | `pig_market_price_daily`（sichuan） | 保留 2 位小数 |
| 本县价格（元/kg） | `pig_sales_record` 聚合均价 | 保留 2 位小数；无数据显示"—" |
| 日涨跌 | `daily_change` | 正值加"+"前缀，红色；负值加"-"前缀，绿色；0 显示"—"灰色 |
| 周涨跌幅 | `weekly_change_rate` | 百分比，正值红色，负值绿色 |

---

#### 卡片三：玉米（14%水分）交易趋势

**展示形式**：折线图 + 可折叠数据表格

**折线图配置**：
- X 轴：时间（粒度规则与生猪趋势一致）
- Y 轴：价格（元/吨）
- 展示曲线（共 2 条）：

| 曲线名称 | 颜色与样式 | 数据来源 |
|---------|----------|---------|
| 全国均价 | 蓝色实线 `#1890ff`，线宽 2px | `pig_market_price_daily`（`product_code = 'corn'`、`price_region = 'national'`） |
| 四川省均价 | 橙色虚线 `#fa8c16`，线宽 1.5px | `pig_market_price_daily`（`product_code = 'corn'`、`price_region = 'sichuan'`） |

- hover Tooltip 展示：日期、全国均价、四川省均价

**可折叠数据表格**（默认收起）：

| 列名 | 说明 |
|------|------|
| 日期 | 格式 YYYY-MM-DD |
| 全国均价（元/吨） | 保留 2 位小数 |
| 四川省均价（元/吨） | 保留 2 位小数 |
| 日涨跌 | 正值红色，负值绿色，0 显示"—" |
| 周涨跌幅 | 百分比 |

---

#### 卡片四：豆粕（43%蛋白）交易趋势

**展示形式**：折线图 + 可折叠数据表格

结构与玉米趋势卡片完全一致，仅数据源不同：

- 品种编码：`soybean_meal`
- Y 轴：价格（元/吨）
- 展示曲线（共 2 条）：

| 曲线名称 | 颜色与样式 | 数据来源 |
|---------|----------|---------|
| 全国均价 | 蓝色实线 `#1890ff`，线宽 2px | `pig_market_price_daily`（`product_code = 'soybean_meal'`、`price_region = 'national'`） |
| 四川省均价 | 橙色虚线 `#fa8c16`，线宽 1.5px | `pig_market_price_daily`（`product_code = 'soybean_meal'`、`price_region = 'sichuan'`） |

**可折叠数据表格**：列定义与玉米趋势表格一致（日期、全国均价(元/吨)、四川省均价(元/吨)、日涨跌、周涨跌幅）。

---

#### 卡片五：白条猪肉交易趋势

**展示形式**：折线图 + 可折叠数据表格

- 品种编码：`pork`
- Y 轴：价格（元/公斤）
- 展示曲线（共 2 条）：

| 曲线名称 | 颜色与样式 | 数据来源 |
|---------|----------|---------|
| 全国批发均价 | 蓝色实线 `#1890ff`，线宽 2px | `pig_market_price_daily`（`product_code = 'pork'`、`price_region = 'national'`） |
| 四川省批发均价 | 橙色虚线 `#fa8c16`，线宽 1.5px | `pig_market_price_daily`（`product_code = 'pork'`、`price_region = 'sichuan'`） |

**可折叠数据表格**：

| 列名 | 说明 |
|------|------|
| 日期 | 格式 YYYY-MM-DD |
| 全国批发均价（元/kg） | 保留 2 位小数 |
| 四川省批发均价（元/kg） | 保留 2 位小数 |
| 日涨跌 | 正值红色，负值绿色，0 显示"—" |
| 周涨跌幅 | 百分比 |

---

#### 卡片六：猪粮比价分析

**展示形式**：面积折线图 + 当前值标注 + 状态提示

**猪粮比计算公式**：

```
猪粮比 = 生猪价格（元/公斤）÷ 玉米价格（元/公斤）
```

> 玉米价格原始数据为元/吨，需先转换为元/公斤（÷ 1000）后再计算猪粮比。

**面积折线图配置**：
- X 轴：时间（粒度规则与生猪趋势一致）
- Y 轴：猪粮比（无单位），刻度从 0 开始
- 展示曲线：全国猪粮比（蓝色实线 `#1890ff`，线宽 2px）

**预警区间标注（背景填充色）**：

| 区间范围 | 区间含义 | 背景填充色 | 说明 |
|---------|---------|----------|------|
| < 5:1 | 亏损预警区间 | 红色 `rgba(245,34,45,0.15)` | 养殖企业面临明显亏损 |
| 5:1 ~ 6:1（含5，不含6） | 盈亏平衡区间 | 黄色 `rgba(250,173,20,0.15)` | 养殖利润微薄 |
| 6:1 ~ 9:1（含6，含9） | 正常盈利区间 | 绿色 `rgba(82,196,26,0.1)` | 养殖正常盈利 |
| 9:1 ~ 12:1（不含9，含12） | 较高盈利区间 | 无填充 | 盈利偏高 |
| > 12:1 | 过度盈利区间 | 浅红色 `rgba(255,77,79,0.1)` | 可能引发产能过度扩张 |

- 预警区间分界线以灰色虚线标注，分界线旁标注数值（5:1、6:1、9:1、12:1）
- 当前猪粮比数值在图表右侧以大字体（24px、加粗）标注，格式如"当前猪粮比：X.XX:1"

**预警阈值系统参数**：

| config_key | config_name | config_value（默认） | 说明 |
|-----------|------------|---------------------|------|
| `market.pig_grain_ratio.loss_threshold` | 猪粮比亏损预警线 | `5` | 低于此值触发亏损预警，整数，最小值 1，最大值 20 |
| `market.pig_grain_ratio.balance_threshold` | 猪粮比盈亏平衡线 | `6` | 低于此值触发盈亏平衡提示，整数，最小值 2，最大值 20 |
| `market.pig_grain_ratio.high_threshold` | 猪粮比过度盈利线 | `12` | 高于此值触发过度盈利提示，整数，最小值 8，最大值 30 |

**下方状态提示文字**（根据当前猪粮比动态展示）：

| 条件 | 提示文字 | 文字颜色 |
|------|---------|---------|
| 猪粮比 < 盈亏平衡线（默认 6:1） | "当前猪粮比低于盈亏平衡线，养殖企业面临亏损风险" | 红色 `#f5222d` |
| 猪粮比 > 过度盈利线（默认 12:1） | "当前猪粮比偏高，需关注产能扩张过快风险" | 橙色 `#fa8c16` |
| 盈亏平衡线 ≤ 猪粮比 ≤ 过度盈利线 | 不展示提示文字 | — |
| 数据不足无法计算 | "暂无足够数据计算猪粮比" | 灰色 `#8c8c8c` |

---

#### 卡片七：全国猪肉批发价格地图

**展示形式**：中国地图热力图

**地图配置**：
- 使用 ECharts 中国地图组件
- 地图按省份着色，颜色深浅表示猪肉批发价格高低
- 色阶范围：以全国最低省份价格为下限（浅色 `#e6f7ff`），全国最高省份价格为上限（深色 `#003a8c`），中间线性插值
- 四川省以 2px 红色实线边框 `#f5222d` 高亮突出显示
- 右侧图例展示价格色阶条（纵向渐变色条 + 最高/最低价格标注）

**hover 交互**：
- 鼠标悬停某省时展示 Tooltip：
  - 省份名称
  - 当日猪肉批发均价：XX.XX 元/公斤
  - 较上周涨跌：+X.XX 元/公斤（红色）或 -X.XX 元/公斤（绿色）

**数据来源**：`pig_market_price_province` 表，取 `price_date` 为最新有数据的日期。

**数据更新说明**：地图右下角标注"数据更新日期：YYYY-MM-DD"和"更新频率：每周"灰色小字。

**无数据处理**：某省无数据时该省区域显示为灰色 `#f0f0f0`，hover 提示"暂无数据"。

---

#### 卡片八：价格对比分析

**展示形式**：多品种叠加折线图

**折线图配置**：
- X 轴：时间（粒度规则与生猪趋势一致）
- Y 轴：涨跌幅（%），基准线为 0%（黑色实线，线宽 0.5px）
- 涨跌幅计算公式：各品种以所选时间范围内第一个数据点的价格为基准（100%），后续价格相对基准的变化百分比 = (当前价格 - 基准价格) / 基准价格 × 100%

**展示曲线（共 4 条）**：

| 品种 | 颜色 | 线型 | 数据来源 |
|------|------|------|---------|
| 生猪（外三元） | 蓝色 `#1890ff` | 实线，线宽 2px | 全国均价 |
| 玉米（14%水分） | 绿色 `#52c41a` | 实线，线宽 1.5px | 全国均价 |
| 豆粕（43%蛋白） | 橙色 `#fa8c16` | 实线，线宽 1.5px | 全国均价 |
| 白条猪肉 | 紫色 `#722ed1` | 实线，线宽 1.5px | 全国均价 |

- hover Tooltip 展示：日期、四个品种各自的涨跌幅百分比
- 支持图例（Legend）点击隐藏/显示数据系列
- 该图用于观察各品种价格联动关系（如饲料价格上涨是否传导至生猪价格）

---

### 3.4.2 数据模型与字段定义

#### 新建表一：市场行情日报表 `pig_market_price_daily`

| 字段名（中文） | 字段名（英文） | 类型 | 必填 | 默认值 | 取值范围/枚举值 | 说明 |
|--------------|--------------|------|------|-------|---------------|------|
| 记录ID | `id` | bigint(20) | 是 | 自增 | — | 主键，自增 |
| 品种编码 | `product_code` | varchar(20) | 是 | — | `pig_outer`（生猪外三元）、`corn`（玉米14%水分）、`soybean_meal`（豆粕43%蛋白）、`pork`（白条猪肉），仅此 4 种取值 | 市场品种标识 |
| 价格区域 | `price_region` | varchar(20) | 是 | — | `national`（全国）、`sichuan`（四川省）、`local`（本县），仅此 3 种取值 | 价格数据所属区域 |
| 价格日期 | `price_date` | date | 是 | — | — | 行情数据对应的日期 |
| 价格 | `price` | decimal(10,2) | 是 | — | 最小值 0.01，最大值 99999999.99 | 价格数值 |
| 价格单位 | `price_unit` | varchar(20) | 是 | — | `元/公斤`、`元/吨`，仅此 2 种取值 | 生猪和白条猪肉为"元/公斤"，玉米和豆粕为"元/吨" |
| 日涨跌 | `daily_change` | decimal(10,2) | 否 | NULL | 最小值 -99999999.99，最大值 99999999.99 | 较前一日涨跌金额，正值上涨，负值下跌，0 持平 |
| 周涨跌幅 | `weekly_change_rate` | decimal(5,2) | 否 | NULL | 最小值 -100.00，最大值 999.99 | 较上周同日涨跌百分比 |
| 数据来源 | `data_source` | varchar(100) | 否 | NULL | 最大长度 100 字符 | 数据来源标识，如"农业农村部信息中心"、"中国养猪网"、"本县企业端销售记录" |
| 创建时间 | `create_time` | datetime | 是 | CURRENT_TIMESTAMP | — | 数据入库时间 |

**索引设计**：
- 主键索引：`id`
- 联合唯一索引：`uk_product_region_date`（`product_code`, `price_region`, `price_date`），防止同一品种同一区域同一天重复入库
- 普通索引：`idx_price_date`（`price_date`）
- 普通索引：`idx_product_code`（`product_code`）

#### 新建表二：省份猪肉价格表 `pig_market_price_province`

| 字段名（中文） | 字段名（英文） | 类型 | 必填 | 默认值 | 取值范围/枚举值 | 说明 |
|--------------|--------------|------|------|-------|---------------|------|
| 记录ID | `id` | bigint(20) | 是 | 自增 | — | 主键，自增 |
| 省份编码 | `province_code` | varchar(10) | 是 | — | 6 位行政区划编码，如 `510000`（四川）、`110000`（北京） | 省级行政区划编码 |
| 省份名称 | `province_name` | varchar(20) | 是 | — | 最大长度 20 字符 | 省份中文名称 |
| 价格日期 | `price_date` | date | 是 | — | — | 数据对应日期 |
| 猪肉批发均价 | `pork_price` | decimal(10,2) | 是 | — | 最小值 0.01，最大值 99999999.99，单位 元/公斤 | 该省当期猪肉批发均价 |
| 较上周涨跌 | `weekly_change` | decimal(10,2) | 否 | NULL | 最小值 -99999999.99，最大值 99999999.99 | 与上一周期同省数据对比的涨跌金额 |
| 数据来源 | `data_source` | varchar(100) | 否 | NULL | 最大长度 100 字符 | 数据来源标识 |
| 创建时间 | `create_time` | datetime | 是 | CURRENT_TIMESTAMP | — | 数据入库时间 |

**索引设计**：
- 主键索引：`id`
- 联合唯一索引：`uk_province_date`（`province_code`, `price_date`），防止同一省份同一日期重复入库
- 普通索引：`idx_price_date`（`price_date`）

#### 新建表三：市场数据采集日志表 `pig_market_fetch_log`

| 字段名（中文） | 字段名（英文） | 类型 | 必填 | 默认值 | 取值范围/枚举值 | 说明 |
|--------------|--------------|------|------|-------|---------------|------|
| 日志ID | `id` | bigint(20) | 是 | 自增 | — | 主键，自增 |
| 采集批次号 | `batch_no` | varchar(32) | 是 | — | 格式 `YYYYMMDD_HHmmss_xxx`（xxx 为3位随机数），最大长度 32 字符 | 同一次定时任务触发的所有 API 调用共享同一批次号 |
| 采集类型 | `fetch_type` | varchar(20) | 是 | — | `daily_price`（日报价格采集）、`province_price`（省份价格采集），仅此 2 种取值 | 采集任务类别 |
| 触发方式 | `trigger_type` | char(1) | 是 | — | `0` 定时任务触发、`1` 手动触发，仅此 2 种取值 | — |
| DeepSeek 模型 | `model` | varchar(30) | 是 | — | `deepseek-chat`，最大长度 30 字符 | 调用的模型标识 |
| 请求 Prompt | `request_prompt` | text | 是 | — | — | 发送给 DeepSeek 的完整 Prompt 内容 |
| 原始响应 | `raw_response` | text | 否 | NULL | — | DeepSeek 返回的原始 JSON 文本（用于问题追溯） |
| 解析后数据条数 | `parsed_count` | int | 否 | 0 | 最小值 0，最大值 99999 | 成功解析并入库的数据记录条数 |
| 采集状态 | `fetch_status` | char(1) | 是 | — | `0` 成功、`1` API 调用失败、`2` 数据解析失败、`3` 部分成功，仅此 4 种取值 | — |
| 失败原因 | `error_message` | varchar(500) | 否 | NULL | 最大长度 500 字符 | API 调用或数据解析失败时的错误信息 |
| Token 消耗-输入 | `prompt_tokens` | int | 否 | NULL | 最小值 0 | 本次调用消耗的输入 token 数 |
| Token 消耗-输出 | `completion_tokens` | int | 否 | NULL | 最小值 0 | 本次调用消耗的输出 token 数 |
| API 耗时(ms) | `api_duration_ms` | int | 否 | NULL | 最小值 0 | DeepSeek API 调用耗时（毫秒） |
| 创建时间 | `create_time` | datetime | 是 | CURRENT_TIMESTAMP | — | 日志创建时间 |

**索引设计**：
- 主键索引：`id`
- 普通索引：`idx_batch_no`（`batch_no`）
- 普通索引：`idx_fetch_status`（`fetch_status`）
- 普通索引：`idx_create_time`（`create_time`）

#### 复用现有表（数据来源）

| 分析内容 | 数据来源表 | 关键聚合字段 | 复用说明 |
|---------|----------|-----------|---------|
| 本县生猪收购价 | `pig_sales_record` | `sale_price`、`sales_date` | 取已入库数据（`status = '4'`），按 `sales_date` 分组计算 `sale_price` 均值；WEB企业端-畜禽及其产品销售记录 PRD 定义 |
| 本县饲料采购金额 | `pig_feed_purchase_record` | `purchase_amount` | 取已入库数据（`status = '4'`），用于估算养殖成本线；WEB企业端-饲料和饲料添加剂购进记录 PRD 定义 |
| 本县兽药采购金额 | `pig_vet_drug_purchase_record` | `purchase_amount` | 取已入库数据（`status = '4'`），用于估算养殖成本线；WEB企业端-兽药购进记录 PRD 定义 |
| 本县出栏数据 | `pig_outbound_record` | `outbound_quantity` | 取已入库数据（`status = '4'`），用于估算头均成本 |

#### 各分析接口通用请求参数

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `startDate` | string | 选填 | 时间范围开始，格式 YYYY-MM-DD，默认当前日期往前推 6 个月 |
| `endDate` | string | 选填 | 时间范围结束，格式 YYYY-MM-DD，默认当前日期 |
| `priceRegion` | string | 选填 | 价格区域，枚举值：`national` 全国、`sichuan` 四川省、`chuandongbei` 川东北片区，默认 `national`，仅此 3 种取值 |

#### 今日市场行情速览响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 行情数据数组（固定 4 个元素） |
| `items[].productCode` | string | 品种编码：`pig_outer`、`corn`、`soybean_meal`、`pork` |
| `items[].productName` | string | 品种名称：生猪（外三元）、玉米（14%水分）、豆粕（43%蛋白）、白条猪肉 |
| `items[].price` | decimal(10,2) | 今日价格（最新有数据日期的价格） |
| `items[].priceUnit` | string | 价格单位："元/公斤"或"元/吨" |
| `items[].dailyChange` | decimal(10,2) | 日涨跌金额，正值上涨、负值下跌、0 持平 |
| `items[].weeklyChangeRate` | decimal(5,2) | 周涨跌幅（%） |
| `items[].sparklineData` | array(decimal) | 近 7 天价格数组（按日期升序，固定 7 个元素；数据不足 7 天时用 null 填充） |
| `updateTime` | string | 数据更新时间，格式 YYYY-MM-DD HH:mm |
| `dataException` | boolean | 数据是否异常（外部数据源不可用时为 true） |

#### 品种交易趋势接口通用响应数据结构

生猪、玉米、豆粕、白条猪肉四个品种的交易趋势接口响应结构统一如下（生猪趋势额外包含本县数据和辅助线数据）：

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 趋势数据数组 |
| `items[].date` | string | 日期，格式 YYYY-MM-DD |
| `items[].nationalPrice` | decimal(10,2) | 全国均价 |
| `items[].sichuanPrice` | decimal(10,2) | 四川省均价，无数据时返回 null |
| `items[].localPrice` | decimal(10,2) | 本县价格（仅生猪趋势接口返回此字段），无数据时返回 null |
| `priceUnit` | string | 价格单位 |
| `hasLocalData` | boolean | 是否有本县数据（仅生猪趋势接口返回此字段） |
| `maxPoint` | object | 时间范围内最高价格点（仅生猪趋势接口返回） |
| `maxPoint.date` | string | 最高点日期 |
| `maxPoint.price` | decimal(10,2) | 最高点价格 |
| `minPoint` | object | 时间范围内最低价格点（仅生猪趋势接口返回） |
| `minPoint.date` | string | 最低点日期 |
| `minPoint.price` | decimal(10,2) | 最低点价格 |
| `costLine` | decimal(10,2) | 养殖成本线价格（仅生猪趋势接口返回），元/公斤，无数据时返回 null |

#### 猪粮比价分析响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `currentRatio` | decimal(5,2) | 当前猪粮比（最新有数据日期），无数据时返回 null |
| `items` | array | 趋势数据数组 |
| `items[].date` | string | 日期，格式 YYYY-MM-DD |
| `items[].ratio` | decimal(5,2) | 猪粮比数值 |
| `items[].pigPrice` | decimal(10,2) | 生猪价格（元/公斤） |
| `items[].cornPrice` | decimal(10,2) | 玉米价格（元/公斤，已从元/吨转换） |
| `thresholds` | object | 预警阈值 |
| `thresholds.lossLine` | int | 亏损预警线（默认 5） |
| `thresholds.balanceLine` | int | 盈亏平衡线（默认 6） |
| `thresholds.highLine` | int | 过度盈利线（默认 12） |
| `warningStatus` | string | 当前预警状态，枚举值：`loss`（低于亏损线）、`balance`（盈亏平衡区间）、`normal`（正常盈利）、`high`（过度盈利）、`none`（无法计算），仅此 5 种取值 |
| `warningMessage` | string | 预警提示文字，无预警时返回空字符串 |

#### 全国猪肉价格地图响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 省份价格数据数组 |
| `items[].provinceCode` | string | 省份编码 |
| `items[].provinceName` | string | 省份名称 |
| `items[].porkPrice` | decimal(10,2) | 猪肉批发均价（元/公斤），无数据时返回 null |
| `items[].weeklyChange` | decimal(10,2) | 较上周涨跌（元/公斤），无数据时返回 null |
| `priceDate` | string | 数据日期，格式 YYYY-MM-DD |
| `minPrice` | decimal(10,2) | 全国最低价 |
| `maxPrice` | decimal(10,2) | 全国最高价 |

#### 价格对比分析响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 趋势数据数组 |
| `items[].date` | string | 日期，格式 YYYY-MM-DD |
| `items[].pigChangeRate` | decimal(5,2) | 生猪涨跌幅（%），以期初价格为基准 |
| `items[].cornChangeRate` | decimal(5,2) | 玉米涨跌幅（%） |
| `items[].soybeanChangeRate` | decimal(5,2) | 豆粕涨跌幅（%） |
| `items[].porkChangeRate` | decimal(5,2) | 白条猪肉涨跌幅（%） |
| `basePrices` | object | 各品种基准价格（所选时间范围第一个有数据日期的价格） |
| `basePrices.pigPrice` | decimal(10,2) | 生猪基准价格（元/公斤） |
| `basePrices.cornPrice` | decimal(10,2) | 玉米基准价格（元/吨） |
| `basePrices.soybeanPrice` | decimal(10,2) | 豆粕基准价格（元/吨） |
| `basePrices.porkPrice` | decimal(10,2) | 白条猪肉基准价格（元/公斤） |
| `baseDate` | string | 基准日期，格式 YYYY-MM-DD |

#### 手动触发数据采集请求数据结构

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `fetchType` | string | 是 | 采集类型，枚举值：`daily_price` 日报价格采集、`province_price` 省份价格采集，仅此 2 种取值 |

#### 手动触发数据采集响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `batchNo` | string | 本次采集的批次号 |
| `fetchStatus` | string | 采集状态：`0` 成功、`1` API 调用失败、`2` 数据解析失败、`3` 部分成功 |
| `parsedCount` | int | 成功入库的数据条数 |
| `errorMessage` | string | 失败原因，成功时为空字符串 |
| `apiDurationMs` | int | API 调用耗时（毫秒） |

#### 数据采集日志列表额外请求参数

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `fetchStatus` | string | 选填 | 采集状态筛选，枚举值：`0`/`1`/`2`/`3`，为空时查询全部 |
| `startTime` | string | 选填 | 时间范围开始，格式 YYYY-MM-DD |
| `endTime` | string | 选填 | 时间范围结束，格式 YYYY-MM-DD |
| `pageNum` | int | 选填 | 页码，默认 1，最小值 1 |
| `pageSize` | int | 选填 | 每页条数，默认 10，最小值 1，最大值 50 |

#### 数据采集日志列表响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `total` | int | 总记录数 |
| `rows` | array | 日志记录数组 |
| `rows[].id` | bigint | 日志ID |
| `rows[].batchNo` | string | 采集批次号 |
| `rows[].fetchType` | string | 采集类型编码 |
| `rows[].fetchTypeName` | string | 采集类型名称（日报价格/省份价格） |
| `rows[].triggerType` | string | 触发方式编码 |
| `rows[].triggerTypeName` | string | 触发方式名称（定时任务/手动触发） |
| `rows[].fetchStatus` | string | 采集状态编码 |
| `rows[].fetchStatusName` | string | 采集状态名称 |
| `rows[].parsedCount` | int | 入库条数 |
| `rows[].promptTokens` | int | 输入 token 消耗 |
| `rows[].completionTokens` | int | 输出 token 消耗 |
| `rows[].apiDurationMs` | int | API 耗时（毫秒） |
| `rows[].errorMessage` | string | 失败原因 |
| `rows[].createTime` | string | 采集时间，格式 YYYY-MM-DD HH:mm:ss |

### 3.4.3 界面交互逻辑

**市场贸易分析页面整体布局**：

- 页面标题：市场贸易分析。
- 页面顶部全局筛选栏（固定，不随页面滚动）：
  - 时间范围（日期区间选择器，默认近半年，左侧有快捷按钮组：近一周、近一月、近一季、近半年、近一年）
  - 价格区域（下拉选择，单选，枚举值：全国、四川省、川东北片区，默认全国）
  - 「手动采集数据」按钮（灰色次按钮，仅 `county_admin` 角色展示，点击弹出确认弹窗后触发 DeepSeek 数据采集）
  - 「采集日志」按钮（文字链接按钮，仅 `county_admin` 角色展示，点击展开采集日志抽屉面板）
  - 「导出市场分析报告」按钮（蓝色主按钮，靠右，仅 `county_admin` 和 `county_staff` 角色展示）
- 分析卡片区域（筛选栏下方，可滚动）：
  - 第一行：今日市场行情速览（4 个行情卡片 + 数据更新时间标注，通栏）
  - 第二行：生猪（外三元）交易趋势（通栏）
  - 第三行：玉米交易趋势（左半栏）+ 豆粕交易趋势（右半栏）
  - 第四行：白条猪肉交易趋势（左半栏）+ 猪粮比价分析（右半栏）
  - 第五行：全国猪肉批发价格地图（通栏）
  - 第六行：价格对比分析（通栏）

**筛选交互**：
- 全局筛选条件变更后，卡片二至卡片八**即时同步刷新**，无需额外点击"查询"按钮。
- 快捷按钮（近一周/一月/一季/半年/一年）点击后自动设置日期区间并刷新。
- 卡片一（今日行情速览）始终展示全国最新数据，不受全局筛选条件影响。

**图表通用交互**：
- 所有图表使用 ECharts 渲染，最小高度 300px。
- 所有图表支持 hover Tooltip 交互。
- 折线图支持图例（Legend）点击隐藏/显示数据系列。
- 地图最小高度 500px。

**可折叠数据表格**：
- 生猪、玉米、豆粕、白条猪肉四个趋势卡片均有可折叠数据表格，默认收起。
- 卡片标题栏右侧有「展开数据」/「收起数据」切换图标按钮。
- 展开后数据表格以分页形式展示，每页 30 条。

**手动采集交互**（仅 `county_admin` 角色可见）：
- 点击「手动采集数据」按钮，弹出确认弹窗："确认立即触发一次市场行情数据采集？（通过 DeepSeek AI 联网获取最新行情数据）"。
- 确认后按钮变为 loading 状态（禁用），显示"正在采集..."，调用 `POST /admin/analysis/market/fetch-data`。
- 采集完成后弹出结果提示：
  - 成功：绿色提示"数据采集成功，本次入库 X 条记录"，页面自动刷新所有分析卡片。
  - 部分成功：橙色提示"数据采集部分成功，入库 X 条记录，Y 条数据异常（超出合理价格范围）"。
  - 失败：红色提示"数据采集失败：{错误原因}，请稍后重试"。
- 为防止滥用，手动采集按钮在一次采集完成后 5 分钟内禁用（灰色 + 倒计时提示"X分X秒后可再次采集"）。

**采集日志抽屉面板**（仅 `county_admin` 角色可见）：
- 点击「采集日志」从右侧滑出抽屉面板，宽度 600px。
- 展示 `pig_market_fetch_log` 表数据，按 `create_time` 降序排列，分页每页 10 条。
- 列表列：

| 列名 | 说明 |
|------|------|
| 采集时间 | `create_time`，格式 YYYY-MM-DD HH:mm:ss |
| 采集类型 | `daily_price` 显示"日报价格"、`province_price` 显示"省份价格" |
| 触发方式 | `0` 显示"定时任务"（灰色 Tag）、`1` 显示"手动触发"（蓝色 Tag） |
| 状态 | `0` 成功（绿色）、`1` API调用失败（红色）、`2` 解析失败（红色）、`3` 部分成功（橙色） |
| 入库条数 | `parsed_count` |
| Token 消耗 | `prompt_tokens` + `completion_tokens`，格式"输入X/输出X" |
| API 耗时 | `api_duration_ms`，格式"X.Xs" |
| 失败原因 | `error_message`，仅状态非成功时展示，超长截断加"..." |

**导出交互**：
- 页面顶部「导出市场分析报告」按钮仅县级角色（`county_admin`、`county_staff`）可见。
- 点击后导出包含所有分析卡片数据表格的完整 Excel 报告。
- 每个分析维度的数据占一个 Sheet 页，共 7 个 Sheet：
  - Sheet 1：今日市场行情速览
  - Sheet 2：生猪（外三元）交易趋势
  - Sheet 3：玉米（14%水分）交易趋势
  - Sheet 4：豆粕（43%蛋白）交易趋势
  - Sheet 5：白条猪肉交易趋势
  - Sheet 6：猪粮比价分析
  - Sheet 7：全国各省猪肉批发价格
- 文件名格式：`市场贸易分析_{起始日期}至{截止日期}_YYYYMMDD_HHmmss.xlsx`，如 `市场贸易分析_2025-09-10至2026-03-10_20260310_143000.xlsx`。
- 导出仅包含数据表格，不含图表图片。

**加载状态**：
- 页面进入和筛选条件变更后，各分析卡片独立展示 loading 状态（旋转加载图标 + "加载中..."文字）。
- 某个接口加载失败时，对应卡片展示错误提示"数据加载失败，请重试"和「重试」按钮，不影响其他卡片正常展示。

**空状态处理**：

| 场景 | 提示文案 |
|------|---------|
| 今日行情速览外部数据异常 | 数据更新时间变红色 + "（数据更新异常，展示最近可用数据）" |
| 趋势图无数据（全部区域均无数据） | "所选时间范围内暂无市场行情数据" |
| 生猪趋势图本县数据缺失 | 图例"本县收购价"显示灰色 + 标注"本县数据暂未采集" |
| 猪粮比无法计算 | "暂无足够数据计算猪粮比" |
| 价格地图某省无数据 | 该省区域灰色，hover 提示"暂无数据" |
| 价格对比分析某品种无基准价格 | 该品种折线不展示，图例标注"数据不足" |

**校验规则**：

| 校验字段 | 校验类型 | 校验规则 | 错误提示文案 |
|---------|---------|---------|------------|
| 时间范围-开始时间 | 逻辑 | 不得晚于结束时间 | 「开始时间不能晚于结束时间」 |
| 时间范围-结束时间 | 逻辑 | 不得早于开始时间 | 「结束时间不能早于开始时间」 |

### 3.4.4 接口路径定义

| 接口名称 | HTTP 方法 | 路径 | 请求参数摘要 | 响应数据摘要 | 备注 |
|----------|-----------|------|-------------|-------------|------|
| 今日市场行情速览 | GET | `/admin/analysis/market/today` | 无（始终返回全国最新数据） | 4 个品种的今日价格、涨跌、迷你折线数据 + 更新时间 + 异常标识 | 🆕 新增接口 |
| 生猪交易趋势 | GET | `/admin/analysis/market/pig-trend` | startDate、endDate、priceRegion | 全国/四川/本县三条趋势线数据 + 最高最低点标注 + 养殖成本线 | 🆕 新增接口 |
| 玉米交易趋势 | GET | `/admin/analysis/market/corn-trend` | startDate、endDate、priceRegion | 全国/四川两条趋势线数据 | 🆕 新增接口 |
| 豆粕交易趋势 | GET | `/admin/analysis/market/soybean-trend` | startDate、endDate、priceRegion | 全国/四川两条趋势线数据 | 🆕 新增接口 |
| 白条猪肉交易趋势 | GET | `/admin/analysis/market/pork-trend` | startDate、endDate、priceRegion | 全国/四川两条趋势线数据 | 🆕 新增接口 |
| 猪粮比价分析 | GET | `/admin/analysis/market/pig-grain-ratio` | startDate、endDate | 猪粮比趋势数据 + 当前值 + 预警阈值 + 预警状态 | 🆕 新增接口 |
| 全国猪肉价格地图 | GET | `/admin/analysis/market/price-map` | 无（始终返回最新一期全国各省数据） | 各省猪肉价格 + 涨跌 + 数据日期 + 价格极值 | 🆕 新增接口 |
| 价格对比分析 | GET | `/admin/analysis/market/price-compare` | startDate、endDate | 四品种涨跌幅百分比趋势数据 + 基准价格 | 🆕 新增接口 |
| 导出市场分析报告 | POST | `/admin/analysis/market/export` | startDate、endDate、priceRegion | Excel 文件流（7 个 Sheet） | 🆕 新增接口 |
| 手动触发数据采集 | POST | `/admin/analysis/market/fetch-data` | fetchType（`daily_price`/`province_price`） | 采集批次号 + 采集状态 | 🆕 新增接口，仅 `county_admin` 可操作 |
| 数据采集日志列表 | GET | `/admin/analysis/market/fetch-log` | pageNum、pageSize、fetchStatus、startTime、endTime | 分页日志列表 | 🆕 新增接口，仅 `county_admin` 可查看 |
| 系统参数查询 | GET | `/system/config/configKey/{configKey}` | configKey | 参数值 | ✅ 复用现有接口，见现有业务说明.spec.md §三 |

## 3.5 非功能需求

- **安全与权限**：
  - 需具备 `admin:analysis:market` 权限标识方可访问市场贸易分析页面。
  - 需具备 `admin:analysis:market:export` 权限标识方可执行导出市场分析报告操作。
  - 需具备 `admin:analysis:market:fetch` 权限标识方可执行手动触发数据采集和查看采集日志操作。
  - 权限分配：

  | 权限标识 | 说明 | 拥有角色 |
  |---------|------|---------|
  | `admin:analysis:market` | 市场贸易分析查看 | `county_admin`、`county_staff`、`town_gov_staff`、`town_vet_station` |
  | `admin:analysis:market:export` | 导出市场分析报告 | `county_admin`、`county_staff` |
  | `admin:analysis:market:fetch` | 手动触发采集 + 查看采集日志 | `county_admin` |

  - 市场行情数据不涉及本县数据权限隔离，所有角色看到的全国/四川数据相同。仅"本县收购价"部分关联本县企业端数据，该部分数据对所有监管端角色统一展示（不按乡镇过滤）。
  - **DeepSeek API Key 安全**：API Key 存储在 `sys_config` 表中，`config_type = 'N'`（非前端可见），后端读取时脱敏处理，前端不可获取明文 Key。
  - 角色定义引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1。

- **性能**：
  - 各分析接口独立返回，单个接口响应时间不超过 3 秒。
  - 页面采用异步并行加载策略：所有分析接口并行请求，各卡片独立渲染，不因某一个慢的接口阻塞整个页面。
  - 行情数据由后端定时任务**每日两次**调用 DeepSeek API 采集（上午 9:00 和下午 14:00），采集到的数据持久化到本地数据库，页面展示时直接从本地表读取，不在用户请求时调用 DeepSeek。
  - 单次 DeepSeek API 调用预期耗时 5-15 秒（含联网搜索），定时任务设置超时 60 秒。
  - 全国各省猪肉价格数据每周采集两次（周一和周四上午 10:00），地图数据可能有 1-3 天延迟。
  - `pig_market_price_daily` 表预计每日新增 12 条记录（4 品种 × 3 区域），年增量约 4380 条；`pig_market_price_province` 表预计每周新增约 31 × 2 = 62 条，年增量约 3224 条；`pig_market_fetch_log` 表预计每日新增 2-4 条，年增量约 1000 条，数据量较小，无需分表分库。

- **DeepSeek API 调用成本估算**：
  - 模型：`deepseek-chat`（DeepSeek-V3.2）
  - 定价：输入 0.5 元/百万 tokens，输出 8 元/百万 tokens（以国内人民币定价为准）
  - 单次日报采集 Prompt 约 500 tokens（输入），预期输出约 2000 tokens，单次成本约 0.0163 元
  - 每日 2 次日报 + 每周 2 次省份采集，月度预估成本：(0.0163 × 2 × 30) + (0.03 × 2 × 4) ≈ 1.2 元/月
  - 含手动触发和重试，月度成本上限预估不超过 5 元

- **兼容性**：桌面端主流分辨率（1280px 及以上）正常显示。图表组件使用 ECharts，中国地图使用 ECharts 地图组件，与大屏及其他分析模块保持技术栈统一。

## 3.6 验收标准

- [ ] 市场贸易分析页面展示全局筛选栏和 8 个分析卡片（含行情速览），布局与 3.4.3 定义一致。
- [ ] 全局筛选条件变更后，卡片二至卡片八即时同步刷新，无需额外点击"查询"按钮；卡片一始终展示全国最新数据不受筛选影响。
- [ ] 所有监管端角色看到的市场行情数据（全国/四川）相同，无数据权限过滤。
- [ ] 乡镇角色不展示「导出市场分析报告」按钮。
- [ ] **今日市场行情速览**：4 个行情卡片正确展示各品种今日价格、日涨跌（红涨绿跌灰持平）、周涨跌幅、迷你折线图（近 7 天），右上角标注数据更新时间。
- [ ] **今日行情异常处理**：外部数据源不可用时，展示最后一次成功采集的数据，数据更新时间变红色并附加"数据更新异常"提示。
- [ ] **生猪交易趋势**：折线图正确展示全国均价（蓝色实线）、四川省均价（橙色虚线）、本县收购价（绿色虚线，无数据时不展示并标注"本县数据暂未采集"），自动标注时间范围内最高/最低价格点。
- [ ] **生猪交易趋势辅助线**：养殖成本线以红色虚线标注，生猪价格低于成本线时区域填充半透明红色；猪粮比 6:1 参考线以灰色虚线标注。
- [ ] **玉米/豆粕/白条猪肉交易趋势**：各品种折线图正确展示全国均价和四川省均价两条趋势线，可折叠数据表格可展开查看明细。
- [ ] **猪粮比价分析**：面积折线图正确展示全国猪粮比趋势，4 个预警区间以对应颜色填充标注；当前猪粮比在图表右侧大字体标注；低于盈亏平衡线时展示红色提示，高于过度盈利线时展示橙色提示。
- [ ] **猪粮比预警阈值**可通过系统管理→参数设置（`sys_config`）调整，无需修改代码。
- [ ] **全国猪肉价格地图**：中国地图按省份着色展示价格高低，四川省红色边框高亮，hover 展示省份名称/价格/涨跌，右侧展示价格色阶图例，无数据省份灰色显示。
- [ ] **价格对比分析**：四品种涨跌幅百分比叠加折线图正确展示，以所选时间范围首日价格为基准，0% 基准线清晰标注，图例可点击隐藏/显示。
- [ ] 所有趋势图表 hover 时展示具体数值 Tooltip。
- [ ] 所有可折叠数据表格默认收起，可通过「展开数据」/「收起数据」按钮切换。
- [ ] 「导出市场分析报告」按钮仅县级角色可见，点击导出 7 个 Sheet 的 Excel 文件。
- [ ] 各分析卡片独立加载，某个加载失败不影响其他卡片展示，失败卡片展示重试按钮。
- [ ] **DeepSeek 数据采集**：后端定时任务每日 9:00 和 14:00 各执行一次 DeepSeek API 调用，采集四品种全国/四川价格数据并写入 `pig_market_price_daily`，联合唯一索引防止重复入库。
- [ ] **DeepSeek 数据采集**：DeepSeek API 返回 JSON 格式结构化数据，后端解析后入库；解析失败时记录错误日志，不影响已有历史数据展示。
- [ ] **DeepSeek 采集日志**：每次 API 调用均记录到 `pig_market_fetch_log` 表，包含 Prompt、原始响应、解析条数、token 消耗、耗时、状态。
- [ ] **手动触发采集**：县级管理员可通过页面按钮手动触发一次数据采集，触发后展示采集结果（成功条数或失败原因）。
- [ ] **DeepSeek API Key 安全**：API Key 存储在 `sys_config` 中，类型为不可前端查看（`config_type = 'N'`），后端接口不返回明文 Key。
- [ ] 省份猪肉价格数据由定时任务每周两次（周一、周四）调用 DeepSeek 采集。
- [ ] 见可交互 HTML 原型。

---

## 附录

### A. 分析卡片与数据表对照总表

| 分析卡片 | 涉及数据表 | 关键聚合字段 |
|---------|----------|-----------|
| 今日市场行情速览 | `pig_market_price_daily` | product_code、price_region=national、price_date（最新） |
| 生猪（外三元）交易趋势 | `pig_market_price_daily`、`pig_sales_record`、`pig_feed_purchase_record`、`pig_vet_drug_purchase_record`、`pig_outbound_record` | product_code=pig_outer、price、sale_price、purchase_amount、outbound_quantity |
| 玉米交易趋势 | `pig_market_price_daily` | product_code=corn、price |
| 豆粕交易趋势 | `pig_market_price_daily` | product_code=soybean_meal、price |
| 白条猪肉交易趋势 | `pig_market_price_daily` | product_code=pork、price |
| 猪粮比价分析 | `pig_market_price_daily` | product_code=pig_outer + corn（计算比值） |
| 全国猪肉价格地图 | `pig_market_price_province` | province_code、pork_price |
| 价格对比分析 | `pig_market_price_daily` | 四品种 national 价格（计算变化率） |
| 数据采集日志（管理功能） | `pig_market_fetch_log` | fetch_status、create_time、parsed_count |

### B. 预警阈值与系统参数配置

| config_key | config_name | config_type | config_value | 说明 |
|-----------|------------|------------|-------------|------|
| `market.pig_grain_ratio.loss_threshold` | 猪粮比亏损预警线 | `Y` | `5` | 整数，1-20 |
| `market.pig_grain_ratio.balance_threshold` | 猪粮比盈亏平衡线 | `Y` | `6` | 整数，2-20 |
| `market.pig_grain_ratio.high_threshold` | 猪粮比过度盈利线 | `Y` | `12` | 整数，8-30 |
| `market.avg_slaughter_weight` | 出栏均重(公斤) | `Y` | `120` | 整数，80-200，用于将头均成本折算为元/公斤 |
| `market.deepseek.api_key` | DeepSeek API密钥 | `N` | — | **不可前端查看**，需在部署时配置，最大长度 128 字符 |
| `market.deepseek.base_url` | DeepSeek API地址 | `Y` | `https://api.deepseek.com` | 可切换为第三方兼容接口地址 |
| `market.deepseek.model` | DeepSeek 模型标识 | `Y` | `deepseek-chat` | 可切换模型，如需推理能力可改为 `deepseek-reasoner` |
| `market.deepseek.timeout_ms` | DeepSeek API超时(ms) | `Y` | `60000` | 单次 API 调用超时时间，整数，最小值 10000，最大值 120000 |
| `market.deepseek.max_retry` | DeepSeek 最大重试次数 | `Y` | `2` | API 调用失败后重试次数，整数，最小值 0，最大值 5 |

### C. 与其他模块的定位区分

| 模块 | 定位 | 数据范围 | 实现阶段 |
|------|------|---------|---------|
| **市场贸易分析（本模块）** | 外部市场行情视角，通过 DeepSeek 大模型联网搜索获取全国/区域级市场行情数据，关注价格走势、猪粮比、价格联动分析 | DeepSeek 采集的外部市场行情数据 + 本县企业端销售数据 | 第4阶段 |
| 养殖分析 | 内部产业分析视角，关注本县养殖生产数据（存栏、出栏、PSY、成本、预警） | 本县已入库养殖数据的聚合统计 | 第4阶段 |
| 产业总览大屏 | 最高层展示视角，将养殖分析、市场贸易分析等核心指标汇聚到一张可视化大屏 | 全部分析数据的精选展示 | 第4阶段（后续） |
| Deepseek 智能服务 | AI 增强分析，基于本地持久化的历史市场数据做价格预测和养殖建议 | 市场行情（本模块已采集）+ 养殖数据 | 第7阶段 |

### D. 导出文件名汇总

| 导出类型 | 文件名格式 | 示例 |
|---------|----------|------|
| 整页分析报告 | `市场贸易分析_{起始日期}至{截止日期}_YYYYMMDD_HHmmss.xlsx` | `市场贸易分析_2025-09-10至2026-03-10_20260310_143000.xlsx` |

### E. 定时任务配置

| 任务名称 | cron 表达式 | 说明 |
|---------|-----------|------|
| 市场行情日报数据采集（上午） | `0 0 9 * * ?` | 每日上午 9:00 执行，调用 DeepSeek API 联网搜索获取当日四个品种全国/四川价格数据，写入 `pig_market_price_daily` |
| 市场行情日报数据采集（下午） | `0 0 14 * * ?` | 每日下午 14:00 执行，同上，用于获取午后更新的行情数据并覆盖当日记录（取更新值） |
| 本县收购价聚合 | `0 30 9 * * ?` | 每日上午 9:30 执行（在行情采集之后），从 `pig_sales_record` 聚合前一日本县生猪销售均价，写入 `pig_market_price_daily`（`price_region = 'local'`） |
| 省份猪肉价格采集（周一） | `0 0 10 ? * MON` | 每周一上午 10:00 执行，调用 DeepSeek API 采集全国各省猪肉批发价格，写入 `pig_market_price_province` |
| 省份猪肉价格采集（周四） | `0 0 10 ? * THU` | 每周四上午 10:00 执行，同上 |

### F. DeepSeek API 对接方案

#### F.1 对接概述

| 项目 | 说明 |
|------|------|
| **服务商** | DeepSeek（深度求索） |
| **API 格式** | OpenAI 兼容格式，可使用 OpenAI SDK 调用 |
| **API Base URL** | `https://api.deepseek.com`（兼容路径 `https://api.deepseek.com/v1`） |
| **调用端点** | `POST /chat/completions` |
| **使用模型** | `deepseek-chat`（DeepSeek-V3.2，128K 上下文长度，非思考模式） |
| **API Key 获取** | 在 `platform.deepseek.com/api_keys` 创建 |
| **认证方式** | HTTP Header `Authorization: Bearer {API_KEY}` |
| **输出格式** | 启用 JSON Output 模式（`response_format: { type: "json_object" }`），确保大模型返回可解析的结构化 JSON |
| **官方文档** | `https://api-docs.deepseek.com/zh-cn/` |

#### F.2 日报价格采集 — Prompt 设计

每次定时任务调用 DeepSeek API 时，发送以下结构的请求：

**请求参数**：

```json
{
  "model": "deepseek-chat",
  "messages": [
    {
      "role": "system",
      "content": "你是一个农产品市场行情数据助手。请通过联网搜索获取最新的中国农产品市场价格数据，以严格的JSON格式返回，不要添加任何多余的解释文字。"
    },
    {
      "role": "user",
      "content": "请搜索并返回今日（{当前日期}）中国以下农产品的最新市场价格数据：\n1. 生猪（外三元）全国均价和四川省均价（单位：元/公斤）\n2. 玉米（14%水分）全国均价和四川省均价（单位：元/吨）\n3. 豆粕（43%蛋白）全国均价和四川省均价（单位：元/吨）\n4. 白条猪肉批发全国均价和四川省均价（单位：元/公斤）\n\n请以JSON格式返回，结构如下：\n{\"date\":\"YYYY-MM-DD\",\"items\":[{\"product_code\":\"pig_outer\",\"product_name\":\"生猪（外三元）\",\"national_price\":数值,\"sichuan_price\":数值,\"price_unit\":\"元/公斤\"},{\"product_code\":\"corn\",...},{\"product_code\":\"soybean_meal\",...},{\"product_code\":\"pork\",...}]}\n\n如果某个品种的四川省数据无法获取，sichuan_price填null。价格保留2位小数。"
    }
  ],
  "response_format": { "type": "json_object" },
  "temperature": 0.1,
  "max_tokens": 2000
}
```

> `temperature` 设为 0.1（极低随机性），确保返回的价格数据尽量准确和稳定。

**预期响应 JSON 结构**：

```json
{
  "date": "2026-03-10",
  "items": [
    {
      "product_code": "pig_outer",
      "product_name": "生猪（外三元）",
      "national_price": 15.62,
      "sichuan_price": 15.28,
      "price_unit": "元/公斤"
    },
    {
      "product_code": "corn",
      "product_name": "玉米（14%水分）",
      "national_price": 2380.00,
      "sichuan_price": 2450.00,
      "price_unit": "元/吨"
    },
    {
      "product_code": "soybean_meal",
      "product_name": "豆粕（43%蛋白）",
      "national_price": 3150.00,
      "sichuan_price": 3200.00,
      "price_unit": "元/吨"
    },
    {
      "product_code": "pork",
      "product_name": "白条猪肉",
      "national_price": 21.35,
      "sichuan_price": 20.80,
      "price_unit": "元/公斤"
    }
  ]
}
```

#### F.3 省份价格采集 — Prompt 设计

**请求 user message**：

```
请搜索并返回最近一周中国各省份（共31个省级行政区）的猪肉批发均价数据。

请以JSON格式返回，结构如下：
{"date":"YYYY-MM-DD","items":[{"province_code":"110000","province_name":"北京","pork_price":数值},{"province_code":"120000","province_name":"天津","pork_price":数值},...]}

省份编码使用6位行政区划编码。价格单位为元/公斤，保留2位小数。如某省无数据，pork_price填null。
```

**省份编码对照表**（硬编码于后端，用于校验 DeepSeek 返回数据）：

| 编码 | 省份 | 编码 | 省份 | 编码 | 省份 |
|------|------|------|------|------|------|
| `110000` | 北京 | `120000` | 天津 | `130000` | 河北 |
| `140000` | 山西 | `150000` | 内蒙古 | `210000` | 辽宁 |
| `220000` | 吉林 | `230000` | 黑龙江 | `310000` | 上海 |
| `320000` | 江苏 | `330000` | 浙江 | `340000` | 安徽 |
| `350000` | 福建 | `360000` | 江西 | `370000` | 山东 |
| `410000` | 河南 | `420000` | 湖北 | `430000` | 湖南 |
| `440000` | 广东 | `450000` | 广西 | `460000` | 海南 |
| `500000` | 重庆 | `510000` | 四川 | `520000` | 贵州 |
| `530000` | 云南 | `540000` | 西藏 | `610000` | 陕西 |
| `620000` | 甘肃 | `630000` | 青海 | `640000` | 宁夏 |
| `650000` | 新疆 | — | — | — | — |

#### F.4 后端数据处理流程

```
定时任务触发（或手动触发）
    ↓
1. 从 sys_config 读取 DeepSeek API Key、Base URL、模型、超时等配置
    ↓
2. 构造 Prompt（填入当前日期），调用 DeepSeek API
    ↓
3. 接收响应，记录原始 response 到 pig_market_fetch_log
    ↓
4. 解析 JSON：
   - 校验 product_code 是否在 4 种枚举值内
   - 校验价格数值是否为正数且在合理范围内（生猪 5-30 元/公斤，玉米 1500-4000 元/吨，豆粕 2000-6000 元/吨，白条猪肉 10-50 元/公斤）
   - 超出合理范围的数据标记为异常，不入库，记录到日志
    ↓
5. 计算 daily_change（与前一日同品种同区域价格对比）和 weekly_change_rate（与 7 天前对比）
    ↓
6. 写入 pig_market_price_daily（INSERT ON DUPLICATE KEY UPDATE，同日同品种同区域覆盖更新）
    ↓
7. 更新 pig_market_fetch_log 的 parsed_count、fetch_status
    ↓
8. 若 API 调用失败，按 max_retry 配置重试；重试仍失败则记录失败日志，不影响历史数据
```

#### F.5 数据质量校验规则

| 品种 | 价格合理范围 | 单位 | 超出处理 |
|------|-----------|------|---------|
| 生猪（外三元） | 5.00 ~ 30.00 | 元/公斤 | 不入库，标记异常 |
| 玉米（14%水分） | 1500.00 ~ 4000.00 | 元/吨 | 不入库，标记异常 |
| 豆粕（43%蛋白） | 2000.00 ~ 6000.00 | 元/吨 | 不入库，标记异常 |
| 白条猪肉 | 10.00 ~ 50.00 | 元/公斤 | 不入库，标记异常 |
| 各省猪肉批发价 | 10.00 ~ 60.00 | 元/公斤 | 不入库，标记异常 |

> 合理范围参考近年市场价格波动区间，可通过系统参数配置调整。大模型返回的价格若偏离合理范围，视为搜索结果不可靠，丢弃该条数据并在采集日志中记录原因。

#### F.6 DeepSeek API 调用代码示例（Java / Spring Boot）

```java
// 使用 OpenAI 兼容的 Java SDK 调用 DeepSeek API
// Maven 依赖：com.theokanning.openai-gpt3-java:service

OpenAiService service = new OpenAiService(
    apiKey,                         // 从 sys_config 读取
    Duration.ofMillis(timeoutMs)    // 从 sys_config 读取
);

ChatCompletionRequest request = ChatCompletionRequest.builder()
    .model("deepseek-chat")
    .messages(Arrays.asList(
        new ChatMessage("system", SYSTEM_PROMPT),
        new ChatMessage("user", buildDailyPricePrompt(LocalDate.now()))
    ))
    .responseFormat(new ResponseFormat("json_object"))
    .temperature(0.1)
    .maxTokens(2000)
    .build();

// 注意：需将 baseUrl 指向 https://api.deepseek.com
ChatCompletionResult result = service.createChatCompletion(request);
String jsonContent = result.getChoices().get(0).getMessage().getContent();

// 解析 JSON 并校验数据
MarketPriceResponse priceData = objectMapper.readValue(jsonContent, MarketPriceResponse.class);
```

> 由于 DeepSeek API 完全兼容 OpenAI 格式，可复用市面上成熟的 OpenAI Java SDK，将 `base_url` 修改为 `https://api.deepseek.com` 即可。如后续需要切换到其他 OpenAI 兼容的大模型服务（如硅基流动、阿里通义等），只需修改 `sys_config` 中的 `base_url`、`api_key`、`model` 三个参数，代码无需变更。
