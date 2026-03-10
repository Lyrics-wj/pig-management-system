# WEB监管端 - 决策分析 - 养殖分析

> **文档版本**：v1.0
> **创建时间**：2026-03-10
> **最后更新**：2026-03-10
> **文档状态**：待评审
> **关联企业端PRD**：`doc_prd/WEB/企业端/企业信息管理/养殖信息管理/` 目录下全部 18 个子模块 PRD
> **关联监管端PRD**：`doc_prd/WEB/监管端/智慧监管/养殖企业管理/WEB监管端-养殖档案监管.prd.md`、`doc_prd/WEB/监管端/智慧监管/电子档案管理/WEB监管端-档案统计分析.prd.md`
> **关联视频监管PRD**：`doc_prd/WEB/监管端/视频监管/` 目录下视频设备信息管理、视频监控告警管理相关 PRD
> **关联需求文档**：`需求梳理/第4阶段-WEB监管端数据分析与大屏/WEB监管端-养殖分析.md`
> **角色与权限基准**：`需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md`

---

## 3.1 基本信息

- **功能名称**：养殖分析
- **关联现有模块/功能**：
  - 用户认证：复用现有 `/login`、`/getInfo`、`/getRouters` 接口与 JWT 机制
  - 权限管理：复用现有角色权限体系，新增"养殖分析"权限标识 `admin:analysis:breeding`
  - 企业端养殖信息管理：18 个子模块的业务数据表为统计数据来源
  - 养殖企业信息管理：`pig_enterprise` 表提供企业基本信息及状态数据
  - 部门树：`sys_dept` 表提供行政区划数据，用于乡镇下拉筛选与数据权限过滤
  - 视频监管管理：`pig_video_device`、`pig_video_alarm` 表提供视频设备和告警数据
  - 工作通知管理：`pig_work_task_feedback`、`pig_work_collect_data` 表提供超期预警数据
  - 系统参数配置：`sys_config` 表存储预警阈值参数
  - 字典管理：企业类型 `pig_enterprise_type` 字典
  - 档案统计分析：与档案统计分析模块定位互补，本模块侧重产业发展视角
  - 图表组件：ECharts（与大屏及其他分析模块保持技术栈统一）
  - 参考文档：`doc_spec/现有业务说明.spec.md`
- **目标用户/场景**：

| 用户角色 | 角色编码（role_key） | 使用场景 | 主要需求 |
|---------|---------------------|---------|---------|
| 县级管理员 | `county_admin` | 产业决策参考、全县养殖态势分析 | 查看全部分析数据、切换筛选条件、处理预警、导出分析报告 |
| 县级业务人员 | `county_staff` | 日常监管分析、数据报告 | 查看全部分析数据、切换筛选条件、处理预警、导出分析报告 |
| 乡镇政府业务人员 | `town_gov_staff` | 辖区养殖情况分析 | 查看本乡镇分析数据（不可处理预警、不可导出） |
| 乡镇畜牧兽医站 | `town_vet_station` | 辖区养殖技术分析 | 查看本乡镇分析数据（不可处理预警、不可导出） |

> 角色编码引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1 角色总览。

## 3.2 目标与范围

**目标**：

1. 对全县养殖数据进行 10 个维度的多维度汇总分析，以可视化图表和数据报表形式呈现养殖产业整体态势。
2. 提供仪表盘式布局，包含全局筛选区 + 10 个分析卡片，辅助监管部门进行科学决策。
3. 支持 PSY 排行、养殖规模排行、成本排行、淘汰率、预警事件统计等深度产业分析指标。
4. 建立养殖预警机制，由后端定时任务自动生成预警记录，支持预警处理流程。
5. 支持分析报告导出（Excel）。
6. 数据范围按角色自动过滤：县级角色看全县数据，乡镇角色看本乡镇数据。

**非目标（Non-Goals）**：

- ❌ 不做实时告警推送（实时告警由视频监管模块负责），预警事件由后端定时任务定时扫描生成。
- ❌ 不提供自定义统计维度或拖拽式报表功能。
- ❌ 不做产业总览大屏（属于第4阶段后续独立模块）。
- ❌ 不导出图表图片，仅导出数据表格（Excel 格式）。
- ❌ 不在本模块内做养殖档案的完整性统计（该职责属于"档案统计分析"模块）。

## 3.3 现状与复用

**现状简述**：

- 18 类养殖档案数据存储在各自独立的业务表中，每条记录有 `status` 字段标识状态（`status = '4'` 表示已入库）。
- 企业基本信息存储在 `pig_enterprise` 表中，包含企业类型、养殖规模、所属部门、状态等。
- 视频设备和告警数据分别存储在 `pig_video_device` 和 `pig_video_alarm` 表中。
- 工作任务和信息收集反馈数据存储在 `pig_work_task_feedback` 和 `pig_work_collect_data` 表中。
- 目前系统无养殖产业分析功能，需全部新建分析接口。
- 需新建预警记录表 `pig_breeding_warning`，用于存储后端定时任务生成的预警记录。
- 分析数据全部从已有业务表实时聚合（`status = '4'` 且 `del_flag = '0'`），不额外建分析快照表。

**复用检查结论**：

**1. 接口复用检查（对照 `doc_spec/现有业务说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户认证（登录） | POST `/login` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取用户信息 | GET `/getInfo` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取路由菜单 | GET `/getRouters` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 部门树下拉接口 | GET `/system/dept/treeselect` | ✅ 直接复用 | 用于乡镇筛选下拉，见现有业务说明.spec.md §三 |
| 字典数据接口 | GET `/system/dict/data/type/{dictType}` | ✅ 直接复用 | 企业类型下拉使用 `pig_enterprise_type` 字典，见现有业务说明.spec.md §三 |
| 系统参数接口 | GET `/system/config/configKey/{configKey}` | ✅ 直接复用 | 获取预警阈值参数配置，见现有业务说明.spec.md §三 |
| 养殖结构概览接口 | 无 | 🆕 新增 `/admin/analysis/breeding/overview` | 无现有接口满足 |
| 养殖结构分布接口 | 无 | 🆕 新增 `/admin/analysis/breeding/structure` | 无现有接口满足 |
| 存栏出栏量趋势接口 | 无 | 🆕 新增 `/admin/analysis/breeding/stock-trend` | 无现有接口满足 |
| 能繁母猪存栏占比接口 | 无 | 🆕 新增 `/admin/analysis/breeding/sow-ratio` | 无现有接口满足 |
| PSY排行榜接口 | 无 | 🆕 新增 `/admin/analysis/breeding/psy-ranking` | 无现有接口满足 |
| 养殖规模排行榜接口 | 无 | 🆕 新增 `/admin/analysis/breeding/scale-ranking` | 无现有接口满足 |
| 养殖成本排行接口 | 无 | 🆕 新增 `/admin/analysis/breeding/cost-ranking` | 无现有接口满足 |
| 企业淘汰率接口 | 无 | 🆕 新增 `/admin/analysis/breeding/elimination-rate` | 无现有接口满足 |
| 预警事件统计接口 | 无 | 🆕 新增 `/admin/analysis/breeding/warning/stats` | 无现有接口满足 |
| 预警事件列表接口 | 无 | 🆕 新增 `/admin/analysis/breeding/warning/list` | 无现有接口满足 |
| 处理预警接口 | 无 | 🆕 新增 PUT `/admin/analysis/breeding/warning/handle` | 无现有接口满足 |
| 视频监控概况接口 | 无 | 🆕 新增 `/admin/analysis/breeding/video-overview` | 虽视频驾驶舱有类似统计，但养殖分析需独立接口以适配全局筛选参数 |
| 导出分析报告接口 | 无 | 🆕 新增 POST `/admin/analysis/breeding/export` | 无现有接口满足 |

**2. 数据库表复用检查（对照 `doc_spec/现有数据库说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 养殖企业表 | `pig_enterprise` | ✅ 直接复用 | 企业数、企业类型、养殖规模、状态等数据来源 |
| 养殖信息表 | `pig_breeding_info` | ✅ 直接复用 | 存栏数据来源 |
| 繁育信息表 | `pig_breeding_record` | ✅ 直接复用 | 能繁母猪数据来源 |
| 断奶信息表 | `pig_weaning_record` | ✅ 直接复用 | 断奶仔猪数（PSY计算）数据来源 |
| 出栏信息表 | `pig_outbound_record` | ✅ 直接复用 | 出栏量数据来源 |
| 饲料购进记录表 | `pig_feed_purchase_record` | ✅ 直接复用 | 饲料采购金额数据来源 |
| 兽药购进记录表 | `pig_vet_drug_purchase_record` | ✅ 直接复用 | 兽药采购金额数据来源 |
| 疾病诊疗记录表 | `pig_disease_treatment_record` | ✅ 直接复用 | 疫病预警数据来源 |
| 病死畜禽无害化处理记录表 | `pig_dead_animal_record` | ✅ 直接复用 | 死亡预警数据来源 |
| 视频设备表 | `pig_video_device` | ✅ 直接复用 | 视频监控概况数据来源 |
| 视频告警表 | `pig_video_alarm` | ✅ 直接复用 | 视频告警预警数据来源 |
| 工作任务反馈表 | `pig_work_task_feedback` | ✅ 直接复用 | 超期预警数据来源 |
| 信息收集数据表 | `pig_work_collect_data` | ✅ 直接复用 | 超期预警数据来源 |
| 部门表 | `sys_dept` | ✅ 直接复用 | 乡镇维度统计与数据权限过滤，见现有数据库说明.spec.md |
| 系统参数配置表 | `sys_config` | ✅ 直接复用 | 存储预警阈值参数，见现有数据库说明.spec.md |
| 定时任务表 | `sys_job` | ✅ 直接复用 | 注册预警扫描定时任务，见现有数据库说明.spec.md |
| 养殖预警记录表 | 无 | 🆕 新建表 `pig_breeding_warning` | 无现有表满足，需新建 |

## 3.4 功能需求

**功能列表**：

| 功能点 | 优先级 | 说明 |
|-------|-------|------|
| 全局筛选条件 | P0 | 时间范围、所属乡镇、企业类型 |
| 养殖结构概览（统计卡片） | P0 | 4 个核心指标数字卡片 |
| 养殖结构分布 | P0 | 饼图 + 数据表格 |
| 生猪存栏出栏量趋势 | P0 | 双Y轴折线图 + 数据表格 |
| 能繁母猪存栏占比 | P0 | 环形图 + 指标说明 |
| PSY排行榜 | P1 | 横向柱状图 + 排行表格 |
| 养殖规模排行榜 | P1 | 横向柱状图 + 排行表格 |
| 投入品成本排行 | P1 | 排行表格 |
| 养殖企业淘汰率 | P1 | 数字指标 + 折线趋势图 |
| 养殖预警事件统计 | P0 | 柱状图 + 事件列表（含处理功能） |
| 规模猪场视频监控概况 | P1 | 统计数字 + 状态分布饼图 |
| 预警事件处理 | P0 | 处理弹窗（仅县级角色） |
| 数据权限自动过滤 | P0 | 县级看全县，乡镇看本乡镇 |
| 分析报告导出 | P1 | Excel，包含所有卡片数据表格 |

**关键流程**：

```
进入养殖分析页面
    ↓
系统按默认筛选条件（近一年、当前角色数据范围、全部企业类型）加载所有分析卡片
    ↓
用户可调整全局筛选条件（时间范围、乡镇、企业类型）→ 所有分析卡片同步刷新
    ↓
用户可查看各分析卡片的图表（hover 查看数值），可展开排行表格的完整数据
    ↓
用户可在预警事件卡片中筛选和查看预警事件 → 点击"处理"对预警进行处置（仅县级角色）
    ↓
用户可导出分析报告（Excel，仅县级角色可操作）
```

### 3.4.1 功能流程逻辑

#### 全局筛选条件

| 序号 | 筛选条件 | 控件类型 | 是否必填 | 默认值 | 说明 |
|------|---------|---------|---------|-------|------|
| 1 | 时间范围 | 日期区间选择器 | 选填 | 近一年（当前日期往前推 12 个月） | 筛选各业务表中数据的时间范围；支持快捷切换：近一月、近一季、近半年、近一年 |
| 2 | 所属乡镇 | 下拉选择（单选） | 选填 | 全县 | 含"全县"选项；乡镇角色不展示此筛选项（自动按本乡镇过滤）；数据来自 `GET /system/dept/treeselect` |
| 3 | 企业类型 | 下拉选择（单选） | 选填 | 全部 | 枚举值来自字典 `pig_enterprise_type`：`1` 规模养殖场、`2` 家庭农场、`3` 散养户、`4` 合作社，另加"全部"选项；数据来自 `GET /system/dict/data/type/pig_enterprise_type` |

全局筛选条件变更后，页面所有分析卡片**即时同步刷新**数据，无需额外点击"查询"按钮。

---

#### 卡片一：养殖结构概览

**展示形式**：页面顶部统计数字卡片组（4 个水平排列的指标卡）

| 序号 | 指标名称 | 计算逻辑 | 数据来源表 | 筛选条件 | 展示方式 |
|------|---------|---------|----------|---------|---------|
| 1 | 养殖企业总数 | COUNT `pig_enterprise` 中 `status = '0'`（正常状态）且 `del_flag = '0'` 的记录 | `pig_enterprise` | 乡镇（按 `dept_id` 关联 `sys_dept`）、企业类型（按 `enterprise_type`） | 整数数字 + 单位"家" + 较上一统计周期变化值（↑X 或 ↓X） |
| 2 | 生猪存栏总量 | SUM 各企业最新一条已入库养殖信息的 `stock_quantity` | `pig_breeding_info`（`status = '4'`） | 时间范围、乡镇、企业类型 | 整数数字 + 单位"头" + 较上一统计周期变化百分比（↑X.X% 或 ↓X.X%） |
| 3 | 能繁母猪存栏量 | SUM 各企业最新已入库繁育信息中的 `fertile_sow_count`（能繁母猪数） | `pig_breeding_record`（`status = '4'`） | 时间范围、乡镇、企业类型 | 整数数字 + 单位"头" + 较上一统计周期变化百分比 |
| 4 | 本期出栏总量 | SUM 所选时间范围内已入库出栏记录的 `outbound_quantity` | `pig_outbound_record`（`status = '4'`） | 时间范围、乡镇、企业类型 | 整数数字 + 单位"头" + 较上一统计周期变化百分比 |

**"较上期变化"计算规则**：
- "上一统计周期"等于当前筛选时间范围的等长前一区间。例如筛选 2025-01 至 2025-12 时，上一统计周期为 2024-01 至 2024-12。
- 变化百分比 = (本期值 - 上期值) / 上期值 × 100%，保留 1 位小数。
- 上期值为 0 时，变化值显示"—"（不计算百分比）。
- 正增长显示绿色 ↑，负增长显示红色 ↓，无变化显示灰色"—"。

---

#### 卡片二：养殖结构分布

**展示形式**：左侧饼图（40% 宽度）+ 右侧数据表格（60% 宽度）

**饼图**：按企业类型（规模养殖场、家庭农场、散养户、合作社）展示企业数量占比分布。各扇形颜色固定：规模养殖场 `#1890ff`（蓝色）、家庭农场 `#52c41a`（绿色）、散养户 `#faad14`（黄色）、合作社 `#722ed1`（紫色）。

**数据表格**：

| 列名 | 数据来源 | 说明 |
|------|---------|------|
| 企业类型 | 字典 `pig_enterprise_type` | 固定 4 行：规模养殖场、家庭农场、散养户、合作社，末行为"合计" |
| 企业数（家） | COUNT `pig_enterprise`（`status = '0'`）按 `enterprise_type` 分组 | 整数 |
| 企业占比 | 各类型企业数 / 企业总数 × 100% | 百分比，保留 1 位小数 |
| 存栏量（头） | SUM 各类型企业最新已入库养殖信息的 `stock_quantity` | 整数 |
| 存栏占比 | 各类型存栏量 / 存栏总量 × 100% | 百分比，保留 1 位小数 |
| 出栏量（头） | SUM 各类型企业已入库出栏记录的 `outbound_quantity` | 整数 |
| 出栏占比 | 各类型出栏量 / 出栏总量 × 100% | 百分比，保留 1 位小数 |

---

#### 卡片三：生猪存栏出栏量趋势

**展示形式**：双 Y 轴折线图 + 可折叠数据表格

**折线图配置**：
- 左 Y 轴：存栏量（头），蓝色折线 `#1890ff`
- 右 Y 轴：出栏量（头），绿色折线 `#52c41a`
- X 轴：时间（按月/季/年，与时间粒度联动）
- 数据点 hover 展示 Tooltip："时间: XXX \n 存栏量: XXX 头 \n 出栏量: XXX 头"
- 支持图例（Legend）点击隐藏/显示数据系列

**时间粒度切换**：图表区域右上角单选按钮组，枚举值：`month` 按月、`quarter` 按季、`year` 按年，仅此 3 种取值，默认 `month`。切换后图表自动重新加载。

**可折叠数据表格**（默认收起）：

| 列名 | 说明 |
|------|------|
| 时间周期 | 按时间粒度展示：YYYY-MM（月）、YYYY-Q1/Q2/Q3/Q4（季）、YYYY（年） |
| 存栏量（头） | 该周期内的存栏数量，整数 |
| 出栏量（头） | 该周期内的出栏数量总和，整数 |

**数据来源**：存栏 `pig_breeding_info`（`stock_quantity`，按 `statistics_date` 聚合）、出栏 `pig_outbound_record`（`outbound_quantity`，按 `outbound_date` 聚合），均取 `status = '4'` 且 `del_flag = '0'`。

---

#### 卡片四：能繁母猪存栏占比

**展示形式**：环形图 + 指标说明文字

**环形图配置**：
- 展示比例：能繁母猪存栏量 / 生猪存栏总量
- 环形图中心大字体显示占比百分比数字（保留 1 位小数）
- 内环颜色 `#ff4d4f`（红色），外环颜色 `#f0f0f0`（灰色背景）
- 右侧行业参考线区域：标注行业合理区间 4% ～ 5%（橙色虚线标注），当前占比低于 4% 或高于 5% 时以黄色背景高亮提示"偏离合理区间"

**指标说明文字**（环形图下方）：
- "能繁母猪存栏量 X 头 / 生猪存栏总量 X 头"
- "同比变化：↑X.X%（较去年同期）"或"同比变化：↓X.X%（较去年同期）"

**同比计算规则**：
- 同比周期 = 当前筛选时间范围往前推 12 个月的等长区间
- 变化百分比 = (本期占比 - 去年同期占比) / 去年同期占比 × 100%，保留 1 位小数
- 去年同期无数据时显示"—"

**数据来源**：能繁母猪 `pig_breeding_record`（`fertile_sow_count`），生猪存栏 `pig_breeding_info`（`stock_quantity`），均取 `status = '4'`。

---

#### 卡片五：PSY 排行榜

**展示形式**：横向柱状图（上方）+ 排行表格（下方）

**PSY 计算方式**：
- PSY = 年断奶仔猪总数 / 能繁母猪平均存栏数
- 断奶仔猪总数来源：`pig_weaning_record` 表 SUM `weaning_count`（`status = '4'`）
- 能繁母猪平均存栏数来源：`pig_breeding_record` 表按月取 `fertile_sow_count` 的平均值（`status = '4'`）
- 能繁母猪平均存栏数为 0 时，该企业 PSY 无法计算，不参与排行
- 企业未填报繁育或断奶数据时，不参与排行，在排行列表外以灰色文字标注"数据不足"

**横向柱状图**：
- 展示 Top10 企业的 PSY 值，柱形从高到低排列
- 柱形颜色：渐变蓝色 `#1890ff` → `#69c0ff`
- 全县平均 PSY 值以红色虚线标注在柱状图中，虚线旁标注"全县平均: X.X"

**排行表格**：

| 列名 | 说明 |
|------|------|
| 排名 | 序号，从 1 开始 |
| 企业名称 | `pig_enterprise.enterprise_name` |
| 所属乡镇 | 通过 `dept_id` 关联 `sys_dept` 获取乡镇名称 |
| 能繁母猪数（头） | 该企业筛选期内能繁母猪平均存栏数，整数 |
| 断奶仔猪数（头） | 该企业筛选期内断奶仔猪总数，整数 |
| PSY | 计算值，保留 1 位小数 |
| 较上期变化 | 与上一统计周期的 PSY 差值，保留 1 位小数，正值绿色 ↑，负值红色 ↓ |

- 默认展示 Top10，点击「查看全部」展开完整排行（分页，每页 20 条）
- 按 PSY 值降序排列

---

#### 卡片六：养殖规模排行榜

**展示形式**：横向柱状图（上方）+ 排行表格（下方）

**横向柱状图**：展示 Top10 企业的当前存栏量排名，柱形从高到低排列，颜色 `#52c41a`

**排行表格**：

| 列名 | 说明 |
|------|------|
| 排名 | 序号，从 1 开始 |
| 企业名称 | `pig_enterprise.enterprise_name` |
| 所属乡镇 | 通过 `dept_id` 关联 `sys_dept` 获取乡镇名称 |
| 企业类型 | 字典 `pig_enterprise_type` 转换展示 |
| 养殖规模-设计（头） | `pig_enterprise.breeding_scale`，整数 |
| 当前存栏（头） | 该企业最新已入库养殖信息的 `stock_quantity`，整数 |
| 产能利用率 | 当前存栏 / 设计养殖规模 × 100%，保留 1 位小数 |

**产能利用率配色规则**：
- 产能利用率 ≥ 50%：黑色（正常）
- 30% ≤ 产能利用率 < 50%：橙色 `#faad14`
- 产能利用率 < 30%：红色 `#f5222d`
- 设计养殖规模为 0 时，显示"—"

- 默认展示 Top10，点击「查看全部」展开完整排行（分页，每页 20 条）
- 按当前存栏量降序排列

---

#### 卡片七：投入品成本排行

**展示形式**：排行表格

> 说明：当前平台仅有饲料和兽药的采购记录，缺少人工、水电等成本数据。本排行以"投入品成本"命名，明确为饲料 + 兽药采购成本，非完整养殖总成本，避免误导。

**排行表格**：

| 列名 | 说明 |
|------|------|
| 排名 | 序号，从 1 开始 |
| 企业名称 | `pig_enterprise.enterprise_name` |
| 所属乡镇 | 通过 `dept_id` 关联 `sys_dept` 获取乡镇名称 |
| 饲料采购金额（元） | SUM `pig_feed_purchase_record.purchase_amount`（`status = '4'`），保留 2 位小数 |
| 兽药采购金额（元） | SUM `pig_vet_drug_purchase_record.purchase_amount`（`status = '4'`），保留 2 位小数 |
| 投入品合计（元） | 饲料采购金额 + 兽药采购金额，保留 2 位小数 |
| 出栏头数（头） | SUM `pig_outbound_record.outbound_quantity`（`status = '4'`），整数 |
| 头均投入品成本（元/头） | 投入品合计 / 出栏头数，保留 2 位小数 |

**排行规则**：
- 默认按头均投入品成本**升序**排列（成本越低排名越前）
- 出栏头数为 0 时，头均投入品成本显示"—"，排在列表末尾
- 表头上方显示提示文字："仅统计饲料和兽药采购成本，不含人工、水电等其他成本"（灰色小字）
- 默认展示 Top10，点击「查看全部」展开完整排行（分页，每页 20 条）

---

#### 卡片八：养殖企业淘汰率

**展示形式**：左侧数字指标区（30% 宽度）+ 右侧折线趋势图（70% 宽度）

**淘汰率计算**：
- 淘汰率 = 统计周期内状态变为"停用"（`status` 从 `'0'` 变为非 `'0'`）的企业数 / 期初正常企业总数 × 100%
- 期初正常企业总数 = 统计周期开始日期时 `pig_enterprise.status = '0'` 的企业数
- 保留 1 位小数

**数字指标区**：
- 当期淘汰率：大字体百分比数字（如 3.2%），淘汰率 > 5% 时红色，≤ 5% 时绿色
- 淘汰企业数 / 期初企业数：如"12 / 375 家"

**折线趋势图**：
- 展示近 12 个月的淘汰率逐月变化，X 轴为月份（YYYY-MM），Y 轴为淘汰率（%）
- 折线颜色 `#ff4d4f`（红色）
- 数据点 hover 展示 Tooltip："月份: YYYY-MM \n 淘汰率: X.X% \n 淘汰企业数: X 家"

**可折叠淘汰企业明细列表**（默认收起）：

| 列名 | 说明 |
|------|------|
| 企业名称 | `pig_enterprise.enterprise_name` |
| 企业类型 | 字典 `pig_enterprise_type` 转换展示 |
| 所属乡镇 | 通过 `dept_id` 关联 `sys_dept` |
| 停用时间 | `pig_enterprise.update_time`（状态变更时间），格式 YYYY-MM-DD |

---

#### 卡片九：养殖预警事件统计

**展示形式**：上方柱状图 + 下方事件列表

**预警事件来源与触发条件**：

| 预警类别 | 预警编码 | 触发条件 | 数据来源 | 扫描频率 |
|---------|---------|---------|---------|---------|
| 疫病预警 | `1` | 疾病诊疗记录中 `disease_name` 包含高致死率疫病关键词（非洲猪瘟、口蹄疫、猪瘟、蓝耳病、伪狂犬、猪流感，共 6 种） | `pig_disease_treatment_record` | 每小时 |
| 死亡预警 | `2` | 单企业单月病死数量 / 该企业当月存栏量 > 病死率阈值（系统参数 `breeding.warning.death_rate_threshold`，默认 5） | `pig_dead_animal_record`、`pig_breeding_info` | 每小时 |
| 视频告警 | `3` | 视频监控产生人员入侵告警 | `pig_video_alarm`（`alarm_type = '1'`） | 每小时 |
| 产能预警 | `4` | 企业当月存栏量较上月存栏量环比下降超过产能下降阈值（系统参数 `breeding.warning.capacity_drop_threshold`，默认 30）% | `pig_breeding_info` 月度存栏数据 | 每日 |
| 超期预警 | `5` | 工作任务反馈超期（`is_overdue = '1'`）或信息收集超期未提交 | `pig_work_task_feedback`、`pig_work_collect_data` | 每日 |

**预警阈值系统参数**：

| config_key | config_name | config_value（默认） | 说明 |
|-----------|------------|---------------------|------|
| `breeding.warning.death_rate_threshold` | 病死率预警阈值(%) | `5` | 单企业单月病死率超过此值触发死亡预警，单位 %，最小值 1，最大值 100，整数 |
| `breeding.warning.capacity_drop_threshold` | 产能下降预警阈值(%) | `30` | 存栏量环比下降超过此值触发产能预警，单位 %，最小值 1，最大值 100，整数 |
| `breeding.warning.high_risk_diseases` | 高致死率疫病关键词 | `非洲猪瘟,口蹄疫,猪瘟,蓝耳病,伪狂犬,猪流感` | 疫病预警匹配关键词，英文逗号分隔 |

**柱状图**：
- X 轴：5 个预警类别（疫病预警、死亡预警、视频告警、产能预警、超期预警）
- Y 轴：事件数量
- 各类别颜色固定：疫病 `#f5222d`（红色）、死亡 `#fa541c`（橙红）、视频 `#faad14`（黄色）、产能 `#1890ff`（蓝色）、超期 `#722ed1`（紫色）
- hover 展示 Tooltip："类别: XXX \n 事件数: XXX 件 \n 其中未处理: XXX 件"

**事件列表**（柱状图下方，默认展示）：

| 列名 | 说明 |
|------|------|
| 预警时间 | `pig_breeding_warning.create_time`，格式 YYYY-MM-DD HH:mm |
| 预警类别 | Tag 标签展示，颜色与柱状图一致。`1` 疫病预警（红色Tag）、`2` 死亡预警（橙红Tag）、`3` 视频告警（黄色Tag）、`4` 产能预警（蓝色Tag）、`5` 超期预警（紫色Tag） |
| 预警内容摘要 | `pig_breeding_warning.warning_content`，最多显示 80 字符，超出截断加"..." |
| 涉及企业 | 通过 `enterprise_id` 关联 `pig_enterprise.enterprise_name`；无特定企业时显示"—" |
| 所属乡镇 | 通过 `enterprise_id` → `pig_enterprise.dept_id` → `sys_dept.dept_name` |
| 处理状态 | `0` 未处理（红色文字）、`1` 已处理（绿色文字）、`2` 已忽略（灰色文字） |
| 操作 | 「查看详情」链接 + 「处理」按钮（仅 `handle_status = '0'` 且用户有 `admin:analysis:breeding:warning:handle` 权限时展示） |

**事件列表内置筛选**：

| 筛选条件 | 控件类型 | 说明 |
|---------|---------|------|
| 预警类别 | 下拉选择（单选） | 枚举值：全部、疫病预警、死亡预警、视频告警、产能预警、超期预警 |
| 处理状态 | 下拉选择（单选） | 枚举值：全部、未处理、已处理、已忽略 |
| 时间范围 | 日期区间选择器 | 默认与全局筛选的时间范围一致 |

- 未处理预警行以浅红色背景 `#fff1f0` 高亮
- 分页展示，每页 10 条
- 点击「查看详情」：根据 `ref_type` 跳转到对应来源模块的详情页（新窗口打开）

**预警处理弹窗**：

点击「处理」按钮弹出处理弹窗：

| 字段名 | 控件类型 | 必填 | 说明 |
|-------|---------|------|------|
| 预警内容 | 纯文本展示（只读） | — | 展示 `warning_content` 全文 |
| 处理方式 | 单选按钮组 | 是 | 枚举值：`1` 已处理、`2` 已忽略，仅此 2 种取值 |
| 处理说明 | 文本域 | 选填 | 最大长度 500 字符 |

弹窗按钮：「确认」「取消」。确认后调用 `PUT /admin/analysis/breeding/warning/handle` 接口，成功后刷新事件列表。

---

#### 卡片十：规模猪场视频监控概况

**展示形式**：左侧统计数字卡片组（40% 宽度）+ 右侧饼图（60% 宽度）

**统计数字卡片组**（5 个指标竖向排列）：

| 序号 | 指标名称 | 计算逻辑 | 说明 |
|------|---------|---------|------|
| 1 | 接入摄像头总数 | COUNT `pig_video_device`（`del_flag = '0'`） | 整数 + 单位"台" |
| 2 | 当前在线数 | COUNT `pig_video_device`（`device_status = '0'`） | 整数 + 绿色文字 |
| 3 | 当前离线数 | COUNT `pig_video_device`（`device_status = '1'`） | 整数 + 灰色文字 |
| 4 | 今日告警数 | COUNT `pig_video_alarm`（`alarm_time` 为今日，`del_flag = '0'`） | 整数 + 红色文字（若 > 0） |
| 5 | 告警覆盖企业数 | COUNT DISTINCT `pig_video_alarm_rule`（`status = '0'`）关联的 `pig_video_device.enterprise_id` | 整数 |

**饼图**：设备在线/离线/故障占比分布
- 在线 `#52c41a`（绿色）、离线 `#bfbfbf`（灰色）、故障 `#f5222d`（红色）
- `device_status` 枚举：`0` 在线、`1` 离线、`2` 故障，仅此 3 种取值

### 3.4.2 数据模型与字段定义

#### 新建表：养殖预警记录表 `pig_breeding_warning`

| 字段名（中文） | 字段名（英文） | 类型 | 必填 | 默认值 | 取值范围/枚举值 | 说明 |
|--------------|--------------|------|------|-------|---------------|------|
| 预警ID | `warning_id` | bigint(20) | 是 | 自增 | — | 主键，自增 |
| 预警类别 | `warning_type` | char(1) | 是 | — | `1` 疫病预警、`2` 死亡预警、`3` 视频告警、`4` 产能预警、`5` 超期预警，仅此 5 种取值 | — |
| 预警内容 | `warning_content` | varchar(500) | 是 | — | 最大长度 500 字符 | 预警描述文字 |
| 企业ID | `enterprise_id` | bigint(20) | 否 | NULL | — | 关联 `pig_enterprise.enterprise_id`；部分预警（如超期预警）可能无特定企业，此时为 NULL |
| 关联业务ID | `ref_id` | bigint(20) | 否 | NULL | — | 关联来源记录的主键ID |
| 关联业务类型 | `ref_type` | varchar(50) | 否 | NULL | `pig_disease_treatment_record`、`pig_dead_animal_record`、`pig_video_alarm`、`pig_breeding_info`、`pig_work_task_feedback`、`pig_work_collect_data`，仅此 6 种取值 | 来源表标识，用于跳转详情 |
| 处理状态 | `handle_status` | char(1) | 是 | `0` | `0` 未处理、`1` 已处理、`2` 已忽略，仅此 3 种取值 | — |
| 处理人 | `handle_by` | varchar(64) | 否 | NULL | 最大长度 64 字符 | 处理操作人用户名 |
| 处理时间 | `handle_time` | datetime | 否 | NULL | — | 执行处理操作的时间 |
| 处理说明 | `handle_remark` | varchar(500) | 否 | NULL | 最大长度 500 字符 | 处理时填写的说明文字 |
| 创建者 | `create_by` | varchar(64) | 否 | NULL | 最大长度 64 字符 | 系统自动填入（定时任务标识） |
| 创建时间 | `create_time` | datetime | 是 | CURRENT_TIMESTAMP | — | 预警生成时间 |
| 更新者 | `update_by` | varchar(64) | 否 | NULL | 最大长度 64 字符 | — |
| 更新时间 | `update_time` | datetime | 否 | NULL | — | — |
| 删除标志 | `del_flag` | char(1) | 是 | `0` | `0` 存在、`2` 已删除，仅此 2 种取值 | 逻辑删除标志 |

**索引设计**：
- 主键索引：`warning_id`
- 普通索引：`idx_warning_type`（`warning_type`）
- 普通索引：`idx_enterprise_id`（`enterprise_id`）
- 普通索引：`idx_handle_status`（`handle_status`）
- 普通索引：`idx_create_time`（`create_time`）
- 组合索引：`idx_type_status_time`（`warning_type`, `handle_status`, `create_time`）

#### 复用现有表（统计数据来源）

| 分析内容 | 数据来源表 | 关键聚合字段 | 复用说明 |
|---------|----------|-----------|---------|
| 企业数/企业状态 | `pig_enterprise` | `enterprise_type`、`status`、`dept_id`、`breeding_scale` | 复用现有表，WEB企业端-基础信息 PRD 定义 |
| 存栏数据 | `pig_breeding_info` | `stock_quantity`、`statistics_date` | 复用现有表，取 `status = '4'` |
| 出栏数据 | `pig_outbound_record` | `outbound_quantity`、`outbound_date` | 复用现有表，取 `status = '4'` |
| 能繁母猪数据 | `pig_breeding_record` | `fertile_sow_count` | 复用现有表，取 `status = '4'` |
| 断奶仔猪数据 | `pig_weaning_record` | `weaning_count` | 复用现有表，取 `status = '4'` |
| 饲料采购金额 | `pig_feed_purchase_record` | `purchase_amount` | 复用现有表，取 `status = '4'` |
| 兽药采购金额 | `pig_vet_drug_purchase_record` | `purchase_amount` | 复用现有表，取 `status = '4'` |
| 疾病诊疗 | `pig_disease_treatment_record` | `disease_name` | 复用现有表，取 `status = '4'` |
| 病死处理 | `pig_dead_animal_record` | `dead_quantity` | 复用现有表，取 `status = '4'` |
| 视频设备 | `pig_video_device` | `device_status` | 复用现有表 |
| 视频告警 | `pig_video_alarm` | `alarm_type`、`alarm_time` | 复用现有表 |
| 视频告警规则 | `pig_video_alarm_rule` | `status`（启用状态） | 复用现有表 |
| 工作任务反馈 | `pig_work_task_feedback` | `is_overdue` | 复用现有表 |
| 信息收集数据 | `pig_work_collect_data` | `status` | 复用现有表 |

#### 各分析接口通用请求参数

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `startTime` | string | 选填 | 时间范围开始，格式 YYYY-MM-DD，默认当前日期往前推 12 个月 |
| `endTime` | string | 选填 | 时间范围结束，格式 YYYY-MM-DD，默认当前日期 |
| `areaCode` | string | 选填 | 乡镇编码（`sys_dept.dept_id`），为空时按角色数据范围自动过滤 |
| `enterpriseType` | string | 选填 | 企业类型，枚举值：`1` 规模养殖场、`2` 家庭农场、`3` 散养户、`4` 合作社，为空时查询全部类型 |

#### 养殖结构概览响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `enterpriseCount` | int | 养殖企业总数，最小值 0 |
| `enterpriseChange` | decimal(5,1) | 较上期变化百分比，正值增长负值下降，上期为 0 时返回 null |
| `stockTotal` | int | 生猪存栏总量（头），最小值 0 |
| `stockChange` | decimal(5,1) | 较上期变化百分比 |
| `sowTotal` | int | 能繁母猪存栏量（头），最小值 0 |
| `sowChange` | decimal(5,1) | 较上期变化百分比 |
| `exitTotal` | int | 本期出栏总量（头），最小值 0 |
| `exitChange` | decimal(5,1) | 较上期变化百分比 |

#### 养殖结构分布响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 分布数据数组（固定 4 个元素 + 合计行） |
| `items[].enterpriseType` | string | 企业类型编码，`1`/`2`/`3`/`4` |
| `items[].enterpriseTypeName` | string | 企业类型名称 |
| `items[].enterpriseCount` | int | 企业数，最小值 0 |
| `items[].enterpriseRatio` | decimal(4,1) | 企业占比（%），保留 1 位小数 |
| `items[].stockQuantity` | int | 存栏量（头），最小值 0 |
| `items[].stockRatio` | decimal(4,1) | 存栏占比（%），保留 1 位小数 |
| `items[].exitQuantity` | int | 出栏量（头），最小值 0 |
| `items[].exitRatio` | decimal(4,1) | 出栏占比（%），保留 1 位小数 |

#### 存栏出栏量趋势接口额外请求参数

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `granularity` | string | 选填 | 时间粒度，枚举值：`month` 按月、`quarter` 按季、`year` 按年，默认 `month`，仅此 3 种取值 |

#### 存栏出栏量趋势响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 趋势数据数组 |
| `items[].period` | string | 时间周期标识：YYYY-MM（月）、YYYY-Q1/Q2/Q3/Q4（季）、YYYY（年） |
| `items[].stockQuantity` | int | 该周期存栏量，最小值 0 |
| `items[].exitQuantity` | int | 该周期出栏量，最小值 0 |

#### 能繁母猪存栏占比响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `sowTotal` | int | 能繁母猪存栏量（头），最小值 0 |
| `stockTotal` | int | 生猪存栏总量（头），最小值 0 |
| `ratio` | decimal(4,1) | 占比（%），保留 1 位小数 |
| `yoyChange` | decimal(5,1) | 同比变化百分比，正值增长负值下降，去年同期无数据时返回 null |
| `referenceMin` | decimal(3,1) | 行业参考区间下限（%），固定 4.0 |
| `referenceMax` | decimal(3,1) | 行业参考区间上限（%），固定 5.0 |

#### PSY 排行榜响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `avgPsy` | decimal(4,1) | 全县平均 PSY 值，保留 1 位小数 |
| `items` | array | 排行数据数组 |
| `items[].rank` | int | 排名，从 1 开始 |
| `items[].enterpriseId` | bigint | 企业ID |
| `items[].enterpriseName` | string | 企业名称 |
| `items[].townName` | string | 所属乡镇名称 |
| `items[].sowCount` | int | 能繁母猪平均存栏数（头），最小值 0 |
| `items[].weaningCount` | int | 断奶仔猪总数（头），最小值 0 |
| `items[].psy` | decimal(4,1) | PSY 值，保留 1 位小数 |
| `items[].psyChange` | decimal(4,1) | 较上期 PSY 变化值，正值增长负值下降 |
| `total` | int | 参与排行的企业总数 |

PSY 排行榜接口支持分页参数：`pageNum`（页码，默认 1）、`pageSize`（每页条数，默认 10，最大 100）。

#### 养殖规模排行榜响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 排行数据数组 |
| `items[].rank` | int | 排名 |
| `items[].enterpriseId` | bigint | 企业ID |
| `items[].enterpriseName` | string | 企业名称 |
| `items[].townName` | string | 所属乡镇名称 |
| `items[].enterpriseTypeName` | string | 企业类型名称 |
| `items[].designScale` | int | 养殖规模-设计（头） |
| `items[].currentStock` | int | 当前存栏（头），最小值 0 |
| `items[].capacityRate` | decimal(4,1) | 产能利用率（%），保留 1 位小数；设计规模为 0 时返回 null |
| `total` | int | 企业总数 |

支持分页参数：`pageNum`（页码，默认 1）、`pageSize`（每页条数，默认 10，最大 100）。

#### 投入品成本排行响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 排行数据数组 |
| `items[].rank` | int | 排名 |
| `items[].enterpriseId` | bigint | 企业ID |
| `items[].enterpriseName` | string | 企业名称 |
| `items[].townName` | string | 所属乡镇名称 |
| `items[].feedCost` | decimal(12,2) | 饲料采购金额（元） |
| `items[].drugCost` | decimal(12,2) | 兽药采购金额（元） |
| `items[].totalCost` | decimal(12,2) | 投入品合计（元） |
| `items[].exitCount` | int | 出栏头数（头），最小值 0 |
| `items[].unitCost` | decimal(10,2) | 头均投入品成本（元/头），出栏头数为 0 时返回 null |
| `total` | int | 企业总数 |

支持分页参数：`pageNum`（页码，默认 1）、`pageSize`（每页条数，默认 10，最大 100）。

#### 企业淘汰率响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `currentRate` | decimal(4,1) | 当期淘汰率（%），保留 1 位小数 |
| `eliminatedCount` | int | 淘汰企业数 |
| `baseCount` | int | 期初企业总数 |
| `monthlyTrend` | array | 近 12 个月淘汰率趋势数组 |
| `monthlyTrend[].month` | string | 月份，格式 YYYY-MM |
| `monthlyTrend[].rate` | decimal(4,1) | 该月淘汰率（%） |
| `monthlyTrend[].eliminatedCount` | int | 该月淘汰企业数 |
| `eliminatedList` | array | 淘汰企业明细列表 |
| `eliminatedList[].enterpriseName` | string | 企业名称 |
| `eliminatedList[].enterpriseTypeName` | string | 企业类型名称 |
| `eliminatedList[].townName` | string | 所属乡镇名称 |
| `eliminatedList[].eliminatedDate` | string | 停用时间，格式 YYYY-MM-DD |

#### 预警事件统计响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `stats` | array | 按类别统计数组（固定 5 个元素） |
| `stats[].warningType` | string | 预警类别编码，`1`/`2`/`3`/`4`/`5` |
| `stats[].warningTypeName` | string | 预警类别名称 |
| `stats[].totalCount` | int | 该类别总事件数 |
| `stats[].unhandledCount` | int | 该类别未处理事件数 |

#### 预警事件列表额外请求参数

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `warningType` | string | 选填 | 预警类别，枚举值：`1`/`2`/`3`/`4`/`5`，为空时查询全部 |
| `handleStatus` | string | 选填 | 处理状态，枚举值：`0` 未处理、`1` 已处理、`2` 已忽略，为空时查询全部 |
| `pageNum` | int | 选填 | 页码，默认 1，最小值 1 |
| `pageSize` | int | 选填 | 每页条数，默认 10，最小值 1，最大值 100 |

#### 预警事件列表响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `total` | int | 总记录数 |
| `rows` | array | 预警记录数组 |
| `rows[].warningId` | bigint | 预警ID |
| `rows[].warningType` | string | 预警类别编码 |
| `rows[].warningTypeName` | string | 预警类别名称 |
| `rows[].warningContent` | string | 预警内容 |
| `rows[].enterpriseId` | bigint | 企业ID，无特定企业时为 null |
| `rows[].enterpriseName` | string | 企业名称，无特定企业时为"—" |
| `rows[].townName` | string | 所属乡镇名称 |
| `rows[].handleStatus` | string | 处理状态编码 |
| `rows[].handleStatusName` | string | 处理状态名称 |
| `rows[].handleBy` | string | 处理人 |
| `rows[].handleTime` | string | 处理时间，格式 YYYY-MM-DD HH:mm |
| `rows[].handleRemark` | string | 处理说明 |
| `rows[].refId` | bigint | 关联业务ID |
| `rows[].refType` | string | 关联业务类型 |
| `rows[].createTime` | string | 预警生成时间，格式 YYYY-MM-DD HH:mm |

#### 预警处理请求数据结构

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `warningId` | bigint | 是 | 预警ID |
| `handleStatus` | string | 是 | 处理方式，枚举值：`1` 已处理、`2` 已忽略，仅此 2 种取值 |
| `handleRemark` | string | 否 | 处理说明，最大长度 500 字符 |

#### 视频监控概况响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `totalDevices` | int | 接入摄像头总数 |
| `onlineCount` | int | 当前在线数 |
| `offlineCount` | int | 当前离线数 |
| `faultCount` | int | 当前故障数 |
| `todayAlarmCount` | int | 今日告警数 |
| `alarmEnterpriseCount` | int | 告警覆盖企业数 |
| `statusDistribution` | array | 状态分布数组（固定 3 个元素） |
| `statusDistribution[].status` | string | 状态编码：`0` 在线、`1` 离线、`2` 故障 |
| `statusDistribution[].statusName` | string | 状态名称 |
| `statusDistribution[].count` | int | 数量 |
| `statusDistribution[].ratio` | decimal(4,1) | 占比（%），保留 1 位小数 |

### 3.4.3 界面交互逻辑

**养殖分析页面整体布局**：

- 页面标题：养殖分析。
- 页面顶部全局筛选栏（固定，不随页面滚动）：
  - 时间范围（日期区间选择器，默认近一年，左侧有快捷按钮组：近一月、近一季、近半年、近一年）
  - 所属乡镇（下拉选择，单选，含"全县"选项；乡镇角色不展示此筛选项）
  - 企业类型（下拉选择，单选，含"全部"选项）
  - 「导出分析报告」按钮（蓝色主按钮，靠右，仅 `county_admin` 和 `county_staff` 角色展示）
- 分析卡片区域（筛选栏下方，可滚动）：
  - 第一行：养殖结构概览（4 个指标卡，通栏）
  - 第二行：养殖结构分布（左半栏）+ 能繁母猪存栏占比（右半栏）
  - 第三行：生猪存栏出栏量趋势（通栏）
  - 第四行：PSY 排行榜（左半栏）+ 养殖规模排行榜（右半栏）
  - 第五行：投入品成本排行（左半栏）+ 养殖企业淘汰率（右半栏）
  - 第六行：养殖预警事件统计（通栏）
  - 第七行：规模猪场视频监控概况（通栏）

**筛选交互**：
- 全局筛选条件变更后，所有分析卡片**即时同步刷新**，无需额外点击"查询"按钮。
- 快捷按钮（近一月/一季/半年/一年）点击后自动设置日期区间并刷新。
- 乡镇角色进入页面时，乡镇筛选项不可见，系统按登录用户的 `dept_id` 自动过滤。

**图表通用交互**：
- 所有图表使用 ECharts 渲染，最小高度 300px。
- 所有图表支持 hover Tooltip 交互。
- 折线图和柱状图支持图例（Legend）点击隐藏/显示数据系列。
- 横向柱状图的柱形从高到低排列（顶部最高）。

**排行榜交互**：
- PSY 排行榜、养殖规模排行榜、投入品成本排行默认展示 Top10。
- 表格底部有「查看全部」按钮，点击后展开完整排行，支持分页（每页 20 条）。
- 展开后「查看全部」变为「收起」按钮。

**可折叠数据表格**：
- 生猪存栏出栏量趋势、养殖企业淘汰率的数据表格/明细列表默认收起。
- 卡片标题栏右侧有「展开数据」/「收起数据」切换图标按钮。

**导出交互**：
- 页面顶部「导出分析报告」按钮仅县级角色可见。
- 点击后导出包含所有分析卡片数据表格的完整 Excel 报告。
- 每个分析卡片的数据占一个 Sheet 页。
- 文件名格式：`养殖分析_{起始日期}至{截止日期}_YYYYMMDD_HHmmss.xlsx`，如 `养殖分析_2025-01-01至2025-12-31_20260310_143000.xlsx`。
- 导出仅包含数据表格，不含图表图片。

**加载状态**：
- 页面进入和筛选条件变更后，各分析卡片独立展示 loading 状态（旋转加载图标 + "加载中..."文字）。
- 某个接口加载失败时，对应卡片展示错误提示"数据加载失败，请重试"和「重试」按钮，不影响其他卡片正常展示。

**空状态处理**：
- PSY 排行榜所有企业均数据不足时，图表区域展示"暂无足够数据生成 PSY 排行，请确保企业已填报繁育和断奶信息"。
- 投入品成本排行无数据时，展示"暂无投入品采购数据"。
- 预警事件列表无数据时，展示"暂无预警事件"。

**校验规则**：

| 校验字段 | 校验类型 | 校验规则 | 错误提示文案 |
|---------|---------|---------|------------|
| 时间范围-开始时间 | 逻辑 | 不得晚于结束时间 | 「开始时间不能晚于结束时间」 |
| 时间范围-结束时间 | 逻辑 | 不得早于开始时间 | 「结束时间不能早于开始时间」 |
| 处理说明（预警处理弹窗） | 长度 | 选填，最大 500 字符 | 「处理说明不能超过500个字符」 |
| 处理方式（预警处理弹窗） | 必填 | 必须选择一种处理方式 | 「请选择处理方式」 |

### 3.4.4 接口路径定义

| 接口名称 | HTTP 方法 | 路径 | 请求参数摘要 | 响应数据摘要 | 备注 |
|----------|-----------|------|-------------|-------------|------|
| 养殖结构概览 | GET | `/admin/analysis/breeding/overview` | startTime、endTime、areaCode、enterpriseType | 4 个指标值 + 变化百分比 | 🆕 新增接口 |
| 养殖结构分布 | GET | `/admin/analysis/breeding/structure` | startTime、endTime、areaCode、enterpriseType | 4 类企业的企业数、存栏量、出栏量及占比 | 🆕 新增接口 |
| 存栏出栏量趋势 | GET | `/admin/analysis/breeding/stock-trend` | startTime、endTime、areaCode、enterpriseType、granularity | 时间序列存栏量和出栏量 | 🆕 新增接口 |
| 能繁母猪存栏占比 | GET | `/admin/analysis/breeding/sow-ratio` | startTime、endTime、areaCode、enterpriseType | 占比值 + 同比变化 + 参考区间 | 🆕 新增接口 |
| PSY 排行榜 | GET | `/admin/analysis/breeding/psy-ranking` | startTime、endTime、areaCode、enterpriseType、pageNum、pageSize | 排行列表 + 全县平均 PSY + 总数 | 🆕 新增接口 |
| 养殖规模排行榜 | GET | `/admin/analysis/breeding/scale-ranking` | startTime、endTime、areaCode、enterpriseType、pageNum、pageSize | 排行列表 + 总数 | 🆕 新增接口 |
| 投入品成本排行 | GET | `/admin/analysis/breeding/cost-ranking` | startTime、endTime、areaCode、enterpriseType、pageNum、pageSize | 排行列表 + 总数 | 🆕 新增接口 |
| 企业淘汰率 | GET | `/admin/analysis/breeding/elimination-rate` | startTime、endTime、areaCode、enterpriseType | 淘汰率 + 月度趋势 + 明细列表 | 🆕 新增接口 |
| 预警事件统计 | GET | `/admin/analysis/breeding/warning/stats` | startTime、endTime、areaCode、enterpriseType | 5 类预警的总数和未处理数 | 🆕 新增接口 |
| 预警事件列表 | GET | `/admin/analysis/breeding/warning/list` | startTime、endTime、areaCode、enterpriseType、warningType、handleStatus、pageNum、pageSize | 分页预警记录列表 | 🆕 新增接口 |
| 处理预警 | PUT | `/admin/analysis/breeding/warning/handle` | warningId、handleStatus、handleRemark | 操作结果 | 🆕 新增接口 |
| 视频监控概况 | GET | `/admin/analysis/breeding/video-overview` | areaCode、enterpriseType | 设备统计 + 状态分布 | 🆕 新增接口 |
| 导出分析报告 | POST | `/admin/analysis/breeding/export` | startTime、endTime、areaCode、enterpriseType | Excel 文件流（多 Sheet） | 🆕 新增接口 |
| 部门树下拉（乡镇筛选） | GET | `/system/dept/treeselect` | — | 部门树结构 | ✅ 复用现有接口，见现有业务说明.spec.md §三 |
| 字典数据（企业类型） | GET | `/system/dict/data/type/pig_enterprise_type` | — | 企业类型枚举列表 | ✅ 复用现有接口，见现有业务说明.spec.md §三 |
| 系统参数查询 | GET | `/system/config/configKey/{configKey}` | configKey | 参数值 | ✅ 复用现有接口，见现有业务说明.spec.md §三 |

## 3.5 非功能需求

- **安全与权限**：
  - 需具备 `admin:analysis:breeding` 权限标识方可访问养殖分析页面。
  - 需具备 `admin:analysis:breeding:warning:handle` 权限标识方可执行预警处理操作。
  - 需具备 `admin:analysis:breeding:export` 权限标识方可执行导出分析报告操作。
  - 权限分配：

  | 权限标识 | 说明 | 拥有角色 |
  |---------|------|---------|
  | `admin:analysis:breeding` | 养殖分析查看 | `county_admin`、`county_staff`、`town_gov_staff`、`town_vet_station` |
  | `admin:analysis:breeding:warning:handle` | 处理预警 | `county_admin`、`county_staff` |
  | `admin:analysis:breeding:export` | 导出分析报告 | `county_admin`、`county_staff` |

  - 数据权限按 `data_scope` + 部门树自动过滤：县级角色（`county_admin`、`county_staff`）看全县数据（`data_scope=1`），乡镇角色（`town_gov_staff`、`town_vet_station`）看本乡镇数据（`data_scope=4`）。
  - 角色定义引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1。

- **性能**：
  - 各分析接口独立返回，单个接口响应时间不超过 5 秒。
  - 页面采用异步并行加载策略：所有分析接口并行请求，各卡片独立渲染，不因某一个慢的接口阻塞整个页面。
  - 预警事件由后端定时任务生成（频率见 3.4.1 预警触发条件表），不做实时触发。
  - 如后续数据量增大导致聚合查询性能不佳，可引入定时任务生成统计快照表进行优化，初期不做。

- **兼容性**：桌面端主流分辨率（1280px 及以上）正常显示。图表组件使用 ECharts，与大屏及其他分析模块保持技术栈统一。

## 3.6 验收标准

- [ ] 养殖分析页面展示全局筛选栏和 10 个分析卡片，布局与 3.4.3 定义一致。
- [ ] 全局筛选条件变更后，所有分析卡片即时同步刷新，无需额外点击"查询"按钮。
- [ ] 所有分析数据仅基于已入库（`status = '4'`）的业务数据，草稿和审批中的数据不纳入统计。
- [ ] 县级角色看全县分析数据，乡镇角色仅看本乡镇分析数据。
- [ ] 乡镇角色不展示乡镇筛选项、不展示「导出分析报告」按钮、不展示预警「处理」按钮。
- [ ] **养殖结构概览**：4 个指标数字卡片正确展示企业总数、存栏总量、能繁母猪数、出栏总量，各指标附有较上期变化百分比。
- [ ] **养殖结构分布**：饼图正确展示企业类型占比，数据表格正确展示各类型的企业数、存栏量、出栏量及占比。
- [ ] **生猪存栏出栏量趋势**：双 Y 轴折线图正确展示存栏和出栏趋势，支持按月/季/年切换时间粒度。
- [ ] **能繁母猪存栏占比**：环形图正确展示占比，标注行业参考区间 4%-5%，展示同比变化。
- [ ] **PSY 排行榜**：横向柱状图展示 Top10，全县平均 PSY 以红色虚线标注；未填报繁育/断奶数据的企业不参与排行。
- [ ] **养殖规模排行榜**：横向柱状图展示 Top10，排行表格中产能利用率低于 50% 橙色标注、低于 30% 红色标注。
- [ ] **投入品成本排行**：排行表格按头均投入品成本升序排列，表头标注"仅统计饲料和兽药采购成本"；出栏头数为 0 的企业头均成本显示"—"。
- [ ] **养殖企业淘汰率**：正确展示当期淘汰率数字和近 12 个月趋势折线图，淘汰企业明细列表可折叠展开。
- [ ] **养殖预警事件统计**：柱状图按 5 个预警类别展示事件数量；事件列表支持按类别/状态/时间筛选，分页展示；未处理预警红色高亮。
- [ ] **预警处理**：县级角色点击「处理」弹出处理弹窗，提交后预警状态更新，事件列表刷新。
- [ ] **视频监控概况**：正确展示 5 个统计指标和设备状态饼图。
- [ ] 所有排行榜默认 Top10，点击「查看全部」可展开完整排行并分页。
- [ ] 所有图表 hover 时展示具体数值 Tooltip。
- [ ] 「导出分析报告」按钮仅县级角色可见，点击导出多 Sheet Excel 文件。
- [ ] 各分析卡片独立加载，某个加载失败不影响其他卡片展示，失败卡片展示重试按钮。
- [ ] PSY 排行榜、投入品成本排行、预警事件列表在无数据时展示对应空状态提示。
- [ ] 预警阈值参数可通过系统管理→参数设置（`sys_config`）调整，无需修改代码。
- [ ] 见可交互 HTML 原型。

---

## 附录

### A. 分析卡片与数据表对照总表

| 分析卡片 | 涉及数据表 | 关键聚合字段 |
|---------|----------|-----------|
| 养殖结构概览 | `pig_enterprise`、`pig_breeding_info`、`pig_breeding_record`、`pig_outbound_record` | status、stock_quantity、fertile_sow_count、outbound_quantity |
| 养殖结构分布 | `pig_enterprise`、`pig_breeding_info`、`pig_outbound_record` | enterprise_type、stock_quantity、outbound_quantity |
| 存栏出栏量趋势 | `pig_breeding_info`、`pig_outbound_record` | stock_quantity、statistics_date、outbound_quantity、outbound_date |
| 能繁母猪存栏占比 | `pig_breeding_record`、`pig_breeding_info` | fertile_sow_count、stock_quantity |
| PSY 排行榜 | `pig_weaning_record`、`pig_breeding_record` | weaning_count、fertile_sow_count |
| 养殖规模排行榜 | `pig_enterprise`、`pig_breeding_info` | breeding_scale、stock_quantity |
| 投入品成本排行 | `pig_feed_purchase_record`、`pig_vet_drug_purchase_record`、`pig_outbound_record` | purchase_amount、outbound_quantity |
| 养殖企业淘汰率 | `pig_enterprise` | status、update_time |
| 养殖预警事件统计 | `pig_breeding_warning` | warning_type、handle_status |
| 视频监控概况 | `pig_video_device`、`pig_video_alarm`、`pig_video_alarm_rule` | device_status、alarm_time |

### B. 预警阈值系统参数配置

| config_key | config_name | config_type | config_value | 说明 |
|-----------|------------|------------|-------------|------|
| `breeding.warning.death_rate_threshold` | 病死率预警阈值(%) | `Y` | `5` | 整数，1-100 |
| `breeding.warning.capacity_drop_threshold` | 产能下降预警阈值(%) | `Y` | `30` | 整数，1-100 |
| `breeding.warning.high_risk_diseases` | 高致死率疫病关键词 | `Y` | `非洲猪瘟,口蹄疫,猪瘟,蓝耳病,伪狂犬,猪流感` | 英文逗号分隔 |

### C. 与其他模块的定位区分

| 模块 | 定位 | 数据范围 | 实现阶段 |
|------|------|---------|---------|
| **养殖分析（本模块）** | 宏观产业分析视角，关注全县/乡镇级别的养殖态势、效率指标、排行榜、预警趋势 | 已入库数据的聚合统计 | 第4阶段 |
| 档案统计分析 | 档案管理视角，关注数据完整性、填报情况、归档台账 | 已入库数据的填报统计 | 第2阶段 |
| 产业总览大屏 | 最高层展示视角，将养殖分析等多个分析模块的核心指标汇聚到一张可视化大屏 | 全部分析数据的精选展示 | 第4阶段（后续） |
| 视频监管驾驶舱 | 视频监控实时态势 | 视频设备和告警的实时数据 | 第3阶段 |

### D. 导出文件名汇总

| 导出类型 | 文件名格式 | 示例 |
|---------|----------|------|
| 整页分析报告 | `养殖分析_{起始日期}至{截止日期}_YYYYMMDD_HHmmss.xlsx` | `养殖分析_2025-01-01至2025-12-31_20260310_143000.xlsx` |

### E. 定时任务配置

| 任务名称 | cron 表达式 | 说明 |
|---------|-----------|------|
| 养殖预警扫描（高频） | `0 0 * * * ?` | 每小时整点执行，扫描疫病、死亡、视频告警 |
| 养殖预警扫描（低频） | `0 0 6 * * ?` | 每日凌晨6点执行，扫描产能预警、超期预警 |
