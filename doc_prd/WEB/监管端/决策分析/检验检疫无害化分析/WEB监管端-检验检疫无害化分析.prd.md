# WEB监管端 - 决策分析 - 检验检疫无害化分析

> **文档版本**：v1.0
> **创建时间**：2026-03-10
> **最后更新**：2026-03-10
> **文档状态**：待评审
> **关联企业端PRD**：`doc_prd/WEB/企业端/企业信息管理/养殖信息管理/WEB企业端-养殖信息管理-畜禽疾病诊疗记录.prd.md`、`doc_prd/WEB/企业端/企业信息管理/养殖信息管理/WEB企业端-养殖信息管理-畜禽免疫记录.prd.md`、`doc_prd/WEB/企业端/企业信息管理/养殖信息管理/WEB企业端-养殖信息管理-防疫监测记录.prd.md`、`doc_prd/WEB/企业端/企业信息管理/养殖信息管理/WEB企业端-养殖信息管理-病死畜禽无害化处理记录.prd.md`
> **关联监管端PRD**：`doc_prd/WEB/监管端/决策分析/养殖分析/WEB监管端-养殖分析.prd.md`、`doc_prd/WEB/监管端/决策分析/市场贸易分析/WEB监管端-市场贸易分析.prd.md`
> **关联需求文档**：`需求梳理/第4阶段-WEB监管端数据分析与大屏/WEB监管端-检验检疫无害化分析.md`
> **角色与权限基准**：`需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md`

---

## 3.1 基本信息

- **功能名称**：检验检疫无害化分析
- **关联现有模块/功能**：
  - 用户认证：复用现有 `/login`、`/getInfo`、`/getRouters` 接口与 JWT 机制
  - 权限管理：复用现有角色权限体系，新增"检验检疫无害化分析"权限标识 `admin:analysis:quarantine`
  - 企业端养殖信息管理：`pig_disease_treatment_record`（畜禽疾病诊疗记录）、`pig_immunity_record`（畜禽免疫记录）、`pig_epidemic_monitor_record`（防疫监测记录）、`pig_dead_animal_record`（病死畜禽无害化处理记录）为本模块分析数据来源
  - 养殖企业信息管理：`pig_enterprise` 表提供企业基本信息、所属部门及存栏数据关联
  - 养殖信息管理：`pig_breeding_info` 表提供存栏总量数据（用于计算病死率、免疫覆盖率）
  - 部门树：`sys_dept` 表提供行政区划数据，用于乡镇下拉筛选与数据权限过滤
  - 系统参数配置：`sys_config` 表存储预警阈值参数（病死率阈值、免疫覆盖率阈值）
  - 字典管理：处理方法等字典项
  - 养殖分析模块：养殖分析中的疫病预警和死亡预警与本模块数据来源有交叉，本模块侧重检验检疫和无害化处理领域的专项分析
  - 图表组件：ECharts（与大屏及其他分析模块保持技术栈统一）
  - 乡镇地图：蓬溪县行政区划 GeoJSON 数据，用于疫病分布和病死猪分布的热力图展示
  - 参考文档：`doc_spec/现有业务说明.spec.md`
- **目标用户/场景**：

| 用户角色 | 角色编码（role_key） | 使用场景 | 主要需求 |
|---------|---------------------|---------|---------|
| 县级管理员 | `county_admin` | 防疫决策参考、全县检验检疫态势分析 | 查看全部分析数据、切换筛选条件、导入外部数据、导出分析报告 |
| 县级业务人员 | `county_staff` | 日常检疫监管分析、数据报告 | 查看全部分析数据、切换筛选条件、导出分析报告 |
| 乡镇政府业务人员 | `town_gov_staff` | 辖区防疫情况分析 | 查看本乡镇分析数据（不可导入、不可导出） |
| 乡镇畜牧兽医站 | `town_vet_station` | 辖区防疫技术分析 | 查看本乡镇分析数据（不可导入、不可导出） |

> 角色编码引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1 角色总览。

## 3.2 目标与范围

**目标**：

1. 对全县检验检疫和无害化处理数据进行 8 个维度的多维度分析，以可视化图表和数据报表形式呈现防疫态势。
2. 提供仪表盘式布局，包含全局筛选区 + 8 个分析卡片，辅助监管部门掌握疫病防控态势和无害化处理情况。
3. 支持检疫合格率、疫病分布热力图、病死猪趋势与分布、免疫覆盖率、生猪保险统计等深度分析指标。
4. 整合省级平台检疫数据（通过 API 或手动导入）与本县企业填报数据，形成完整的检验检疫分析视图。
5. 支持外部数据导入（检疫数据 Excel 导入、保险数据 Excel 导入/逐条录入），解决省级平台和保险系统短期内无法 API 对接的过渡需求。
6. 支持分析报告导出（Excel）。
7. 数据范围按角色自动过滤：县级角色看全县数据，乡镇角色看本乡镇数据（省级平台导入的检疫汇总数据不做乡镇过滤）。

**非目标（Non-Goals）**：

- ❌ 不做与四川省智慧监督管理平台的实时 API 自动对接（初期通过管理员手动 Excel 导入过渡，API 对接在技术方案中另行确认）。
- ❌ 不做实时疫病预警推送（实时预警由养殖分析模块负责），本模块为分析统计视角。
- ❌ 不提供自定义统计维度或拖拽式报表功能。
- ❌ 不做产业总览大屏（属于第4阶段后续独立模块，大屏可引用本模块核心指标）。
- ❌ 不导出图表图片，仅导出数据表格（Excel 格式）。
- ❌ 不做检疫证明的电子签发或验证功能（属省级平台职责）。

## 3.3 现状与复用

**现状简述**：

- 企业端已有畜禽疾病诊疗记录（`pig_disease_treatment_record`）、畜禽免疫记录（`pig_immunity_record`）、防疫监测记录（`pig_epidemic_monitor_record`）、病死畜禽无害化处理记录（`pig_dead_animal_record`）4 张业务表，每条记录有 `status` 字段标识状态（`status = '4'` 表示已入库）。
- 省级平台（四川省智慧监督管理平台）的检疫数据目前无直接 API 对接，需新建检疫记录表 `pig_quarantine_record`，支持手动 Excel 导入和未来 API 自动对接两种数据入库方式。
- 生猪保险数据目前无来源，需新建保险统计月报表 `pig_insurance_monthly`，支持 Excel 导入和逐条录入。
- 存栏总量来自 `pig_breeding_info` 表（用于计算病死率和免疫覆盖率的分母）。
- 目前系统无检验检疫分析功能，需全部新建分析接口。
- 分析数据从已有业务表实时聚合（`status = '4'` 且 `del_flag = '0'`），不额外建分析快照表。

**复用检查结论**：

**1. 接口复用检查（对照 `doc_spec/现有业务说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户认证（登录） | POST `/login` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取用户信息 | GET `/getInfo` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取路由菜单 | GET `/getRouters` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 部门树下拉接口 | GET `/system/dept/treeselect` | ✅ 直接复用 | 用于乡镇筛选下拉，见现有业务说明.spec.md §三 |
| 字典数据接口 | GET `/system/dict/data/type/{dictType}` | ✅ 直接复用 | 处理方法等字典下拉，见现有业务说明.spec.md §三 |
| 系统参数接口 | GET `/system/config/configKey/{configKey}` | ✅ 直接复用 | 获取预警阈值参数配置，见现有业务说明.spec.md §三 |
| 检疫年度核心指标接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/overview` | 无现有接口满足 |
| 检疫类目分布接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/category` | 无现有接口满足 |
| 检疫趋势接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/trend` | 无现有接口满足 |
| 疫病数量及分布接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/disease` | 无现有接口满足 |
| 疫病乡镇下钻明细接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/disease/detail` | 无现有接口满足 |
| 病死猪数量及分布接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/dead-animal` | 无现有接口满足 |
| 病死猪乡镇下钻明细接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/dead-animal/detail` | 无现有接口满足 |
| 病死猪处理情况分析接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/disposal` | 无现有接口满足 |
| 免疫防疫覆盖分析接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/immunity` | 无现有接口满足 |
| 生猪保险统计接口 | 无 | 🆕 新增 `/admin/analysis/quarantine/insurance` | 无现有接口满足 |
| 导入检疫数据接口 | 无 | 🆕 新增 POST `/admin/analysis/quarantine/import` | 无现有接口满足 |
| 检疫导入模板下载接口 | 无 | 🆕 新增 POST `/admin/analysis/quarantine/import-template` | 无现有接口满足 |
| 导入保险数据接口 | 无 | 🆕 新增 POST `/admin/analysis/quarantine/insurance/import` | 无现有接口满足 |
| 保险导入模板下载接口 | 无 | 🆕 新增 POST `/admin/analysis/quarantine/insurance/import-template` | 无现有接口满足 |
| 新增保险月报接口 | 无 | 🆕 新增 POST `/admin/analysis/quarantine/insurance/data` | 逐条录入保险数据 |
| 导出分析报告接口 | 无 | 🆕 新增 POST `/admin/analysis/quarantine/export` | 无现有接口满足 |

**2. 数据库表复用检查（对照 `doc_spec/现有数据库说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 畜禽疾病诊疗记录表 | `pig_disease_treatment_record` | ✅ 直接复用 | 疫病数量及分布分析的数据来源，按 `disease_name` 字段分组统计 |
| 畜禽免疫记录表 | `pig_immunity_record` | ✅ 直接复用 | 免疫防疫覆盖分析的数据来源，按疫苗类型分组统计 |
| 防疫监测记录表 | `pig_epidemic_monitor_record` | ✅ 直接复用 | 防疫监测数据来源 |
| 病死畜禽无害化处理记录表 | `pig_dead_animal_record` | ✅ 直接复用 | 病死猪数量/分布/处理分析的数据来源，按 `disposal_method` 字段分组统计 |
| 养殖企业表 | `pig_enterprise` | ✅ 直接复用 | 企业信息关联、乡镇归属 |
| 养殖信息表 | `pig_breeding_info` | ✅ 直接复用 | 存栏总量数据来源（用于计算病死率和免疫覆盖率分母） |
| 部门表 | `sys_dept` | ✅ 直接复用 | 乡镇维度统计与数据权限过滤，见现有数据库说明.spec.md |
| 系统参数配置表 | `sys_config` | ✅ 直接复用 | 存储预警阈值参数，见现有数据库说明.spec.md |
| 检疫记录表 | 无 | 🆕 新建表 `pig_quarantine_record` | 无现有表满足，用于存储省级平台或手动导入的检疫数据 |
| 保险统计月报表 | 无 | 🆕 新建表 `pig_insurance_monthly` | 无现有表满足，用于存储生猪保险统计数据 |

## 3.4 功能需求

**功能列表**：

| 功能点 | 优先级 | 说明 |
|-------|-------|------|
| 全局筛选条件 | P0 | 统计年度、所属乡镇、时间粒度 |
| 检疫年度核心指标（统计卡片） | P0 | 6 个核心指标数字卡片 |
| 检疫类目分布 | P0 | 饼图 + 数据表格 |
| 检验检疫趋势 | P0 | 组合图（堆叠柱状 + 折线） |
| 疫病数量及分布 | P0 | 柱状图 + 乡镇热力图 + 可折叠数据表格 |
| 病死猪数量及分布 | P0 | 折线趋势图 + 乡镇热力图 + 可折叠数据表格 |
| 病死猪处理情况分析 | P0 | 饼图 + 堆叠柱状图 + 汇总表格 |
| 免疫防疫覆盖分析 | P0 | 柱状图 + 数据表格 |
| 生猪保险统计 | P1 | 统计卡片 + 组合图 + 可折叠明细表格 |
| 检疫数据 Excel 导入 | P1 | 仅县级管理员可操作 |
| 保险数据导入/录入 | P1 | Excel 导入 + 逐条录入，仅县级管理员可操作 |
| 数据权限自动过滤 | P0 | 县级看全县，乡镇看本乡镇（省级检疫数据不按乡镇过滤） |
| 分析报告导出 | P1 | Excel，包含所有卡片数据表格 |

**关键流程**：

```
进入检验检疫无害化分析页面
    ↓
系统按默认筛选条件（当前年度、当前角色数据范围、按月粒度）加载所有分析卡片
    ↓
用户可调整全局筛选条件（统计年度、乡镇、时间粒度）→ 所有分析卡片同步刷新
    ↓
用户可查看各分析卡片的图表（hover 查看数值），可展开可折叠的数据表格
    ↓
疫病分布/病死猪分布的热力图可点击乡镇下钻查看明细
    ↓
县级管理员可通过「数据管理」菜单导入检疫数据和保险数据
    ↓
县级角色可导出分析报告（Excel）
```

### 3.4.1 功能流程逻辑

#### 全局筛选条件

| 序号 | 筛选条件 | 控件类型 | 是否必填 | 默认值 | 说明 |
|------|---------|---------|---------|-------|------|
| 1 | 统计年度 | 年份选择器 | 选填 | 当前年度（如 2026） | 可切换往年对比，年份范围：2020 至当前年度，仅此范围可选 |
| 2 | 所属乡镇 | 下拉选择（单选） | 选填 | 全县 | 含"全县"选项；乡镇角色不展示此筛选项（自动按本乡镇过滤）；数据来自 `GET /system/dept/treeselect` |
| 3 | 时间粒度 | 按钮切换组 | 选填 | 按月 | 枚举值：`month` 按月、`quarter` 按季、`year` 按年，仅此 3 种取值；用于检疫趋势、病死猪趋势、处理趋势、保险趋势等时间序列图表 |

全局筛选条件变更后，页面所有分析卡片**即时同步刷新**数据，无需额外点击"查询"按钮。

---

#### 卡片一：检疫年度核心指标

**展示形式**：页面顶部统计数字卡片组（6 个水平排列，两行每行 3 个）

| 序号 | 指标名称 | 计算逻辑 | 数据来源表 | 展示方式 |
|------|---------|---------|----------|---------|
| 1 | 检验检疫总量（头次） | SUM `pig_quarantine_record` 中所选年度的 `quantity` | `pig_quarantine_record` | 整数数字 + 单位"头次" + 较去年同期变化（↑X.X% 或 ↓X.X%） |
| 2 | 产地检疫数量（头次） | SUM `pig_quarantine_record` 中 `quarantine_type = 'production'` 的 `quantity` | `pig_quarantine_record` | 整数数字 + 单位"头次" + 同比变化 |
| 3 | 屠宰检疫数量（头次） | SUM `pig_quarantine_record` 中 `quarantine_type = 'slaughter'` 的 `quantity` | `pig_quarantine_record` | 整数数字 + 单位"头次" + 同比变化 |
| 4 | 检验检疫合格率（%） | SUM(`qualified_quantity`) / SUM(`quantity`) × 100% | `pig_quarantine_record` | 百分比数字（保留 1 位小数） + 颜色指示 |
| 5 | 疫病发现数量（例） | COUNT `pig_disease_treatment_record` 中所选年度已入库记录 | `pig_disease_treatment_record`（`status = '4'`） | 整数数字 + 单位"例" + 同比变化 |
| 6 | 病死猪处理数量（头） | SUM `pig_dead_animal_record` 中所选年度已入库记录的 `dead_quantity` | `pig_dead_animal_record`（`status = '4'`） | 整数数字 + 单位"头" + 同比变化 |

**同比变化计算规则**：
- 同比周期 = 当前筛选年度的上一年度同期。例如筛选 2026 年度时，同比对象为 2025 年度。
- 变化百分比 = (本年度值 - 去年同期值) / 去年同期值 × 100%，保留 1 位小数。
- 去年同期值为 0 时，变化值显示"—"（不计算百分比）。
- 正增长显示绿色 ↑，负增长显示红色 ↓，无变化显示灰色"—"。

**合格率颜色指示规则**：
- 合格率 ≥ 99%：绿色 `#52c41a`
- 95% ≤ 合格率 < 99%：黄色 `#faad14`
- 合格率 < 95%：红色 `#f5222d`
- 检疫总量为 0 时，合格率显示"—"

---

#### 卡片二：检疫类目分布

**展示形式**：左侧饼图（40% 宽度）+ 右侧数据表格（60% 宽度）

**饼图**：按检疫类目展示各类目数量占比分布。各扇形颜色固定：产地检疫 `#1890ff`（蓝色）、屠宰检疫 `#52c41a`（绿色）、调运检疫 `#faad14`（黄色）。

**数据表格**：

| 列名 | 数据来源 | 说明 |
|------|---------|------|
| 检疫类目 | `quarantine_type` 枚举 | 固定 3 行：产地检疫、屠宰检疫、调运检疫，末行为"合计" |
| 数量（头次） | SUM `pig_quarantine_record.quantity` 按 `quarantine_type` 分组 | 整数 |
| 占比 | 各类目数量 / 检疫总量 × 100% | 百分比，保留 1 位小数 |
| 合格数 | SUM `pig_quarantine_record.qualified_quantity` 按 `quarantine_type` 分组 | 整数 |
| 合格率 | 合格数 / 数量 × 100% | 百分比，保留 1 位小数；数量为 0 时显示"—" |
| 同比变化 | 本年度数量与上一年度同类目数量对比 | 变化百分比，保留 1 位小数 |

---

#### 卡片三：检验检疫趋势

**展示形式**：组合图（堆叠柱状图 + 折线图）

**图表配置**：
- X 轴：时间（按月/季/年，与全局时间粒度联动）
- 左 Y 轴：检疫数量（头次），堆叠柱状图
  - 产地检疫 `#1890ff`（蓝色柱）
  - 屠宰检疫 `#52c41a`（绿色柱）
  - 调运检疫 `#faad14`（黄色柱）
- 右 Y 轴：检疫合格率（%），红色折线 `#f5222d`
- 合格率低于 95% 的数据点以实心圆放大标注（`symbolSize: 12`），正常数据点 `symbolSize: 6`
- hover Tooltip："时间: XXX \n 产地检疫: XXX 头次 \n 屠宰检疫: XXX 头次 \n 调运检疫: XXX 头次 \n 合格率: XX.X%"
- 支持图例（Legend）点击隐藏/显示数据系列

**数据来源**：`pig_quarantine_record`，按 `quarantine_date` 和 `quarantine_type` 聚合。

---

#### 卡片四：疫病数量及分布

**展示形式**：左侧柱状图（50% 宽度）+ 右侧乡镇热力图（50% 宽度）+ 下方可折叠数据表格

**柱状图（疫病类型 Top10）**：
- 纵向柱状图，Y 轴为疫病名称，X 轴为发病数量（例），按数量从多到少降序排列
- 展示 Top10 疫病类型
- 柱形颜色 `#f5222d`（红色渐变至 `#ff7a45`）
- hover Tooltip："疫病名称: XXX \n 发病数量: XXX 例 \n 涉及企业数: XXX 家"

**乡镇分布热力图**：
- 蓬溪县行政区划地图（使用自定义 GeoJSON 数据），按乡镇着色
- 颜色梯度：白色（0 例）→ 浅红 `#fff1f0` → 中红 `#ff7875` → 深红 `#cf1322`（数量越多颜色越深）
- hover 某乡镇 Tooltip："乡镇名称: XXX \n 疫病数量: XXX 例 \n 涉及企业数: XXX 家"
- 点击某乡镇触发下钻，弹出该乡镇疫病明细列表弹窗

**乡镇下钻明细列表弹窗**：

| 列名 | 说明 |
|------|------|
| 企业名称 | `pig_enterprise.enterprise_name` |
| 疫病名称 | `pig_disease_treatment_record.disease_name` |
| 发病数量（例） | 该企业该疫病的记录数 |
| 最近发病日期 | 该企业该疫病最近一条诊疗记录的日期，格式 YYYY-MM-DD |

弹窗支持分页，每页 10 条。

**可折叠数据表格**（默认收起）：

| 列名 | 数据来源 | 说明 |
|------|---------|------|
| 疫病名称 | `pig_disease_treatment_record.disease_name` | 按疫病名称分组 |
| 发病数量（例） | COUNT 该疫病的诊疗记录 | 整数 |
| 涉及企业数 | COUNT DISTINCT `enterprise_id` | 整数 |
| 涉及乡镇 | 通过 `enterprise_id` → `pig_enterprise.dept_id` → `sys_dept.dept_name`，取去重后的乡镇名称 | 逗号分隔展示，最多展示 3 个，超出显示"+N个" |
| 高发月份 | 该疫病发病数量最多的月份 | 格式"X月" |
| 同比变化 | 本年度发病数量与上一年度对比 | 变化百分比，保留 1 位小数 |

**数据来源**：`pig_disease_treatment_record`（`status = '4'` 且 `del_flag = '0'`），按 `disease_name` 字段分组统计。

---

#### 卡片五：病死猪数量及分布

**展示形式**：左侧折线趋势图（50% 宽度）+ 右侧乡镇热力图（50% 宽度）+ 下方可折叠数据表格

**折线趋势图**：
- X 轴：时间（按月），展示所选年度 12 个月
- Y 轴：病死猪数量（头）
- 本年度蓝色实线 `#1890ff`，去年同期灰色虚线 `#bfbfbf`（`lineStyle: { type: 'dashed' }`）
- hover Tooltip："月份: YYYY-MM \n 本年度: XXX 头 \n 去年同期: XXX 头"
- 支持图例（Legend）点击隐藏/显示数据系列

**乡镇分布热力图**：
- 蓬溪县行政区划地图（复用卡片四的 GeoJSON），按乡镇着色
- 颜色梯度：白色（0 头）→ 浅橙 `#fff7e6` → 中橙 `#ffa940` → 深红 `#cf1322`
- hover Tooltip："乡镇名称: XXX \n 病死数量: XXX 头 \n 涉及企业数: XXX 家 \n 病死率: X.X%"
- 病死率 = 病死数量 / 该乡镇存栏总量 × 100%，保留 1 位小数；存栏总量为 0 时显示"—"
- **预警标识**：乡镇病死率超过阈值（系统参数 `quarantine.dead_rate_threshold`，默认 5）时，在地图上以红色三角图标 `▲` 标注
- 点击某乡镇触发下钻，弹出该乡镇病死猪明细列表弹窗

**乡镇下钻明细列表弹窗**：

| 列名 | 说明 |
|------|------|
| 企业名称 | `pig_enterprise.enterprise_name` |
| 病死数量（头） | SUM `pig_dead_animal_record.dead_quantity` |
| 死因 | `pig_dead_animal_record.death_cause` |
| 处理方法 | `pig_dead_animal_record.disposal_method` |
| 记录日期 | 格式 YYYY-MM-DD |

弹窗支持分页，每页 10 条。

**可折叠数据表格**（默认收起）：

| 列名 | 数据来源 | 说明 |
|------|---------|------|
| 乡镇 | `sys_dept.dept_name` | 按乡镇分组 |
| 病死数量（头） | SUM `pig_dead_animal_record.dead_quantity` | 整数 |
| 存栏总量（头） | SUM 各企业最新已入库 `pig_breeding_info.stock_quantity` | 整数 |
| 病死率 | 病死数量 / 存栏总量 × 100% | 百分比，保留 1 位小数；存栏总量为 0 时显示"—" |
| 涉及企业数 | COUNT DISTINCT `enterprise_id` | 整数 |
| 主要死因 | 该乡镇出现次数最多的死因 | 取 `death_cause` 出现频率最高的值 |
| 同比变化 | 本年度病死数量与上一年度对比 | 变化百分比，保留 1 位小数 |

**病死率配色规则**：
- 病死率 < 3%：黑色（正常）
- 3% ≤ 病死率 < 5%：橙色 `#faad14`
- 病死率 ≥ 5%（达到预警阈值）：红色 `#f5222d`

**数据来源**：`pig_dead_animal_record`（`status = '4'` 且 `del_flag = '0'`）。

---

#### 卡片六：病死猪处理情况分析

**展示形式**：左侧饼图（40% 宽度）+ 右侧堆叠柱状图（60% 宽度）+ 下方汇总表格

**饼图（处理方式占比）**：
- 按处理方法展示占比分布
- 各扇形颜色固定：焚烧 `#f5222d`（红色）、化尸 `#fa8c16`（橙色）、掩埋 `#a0d911`（黄绿）、发酵 `#52c41a`（绿色）、委托集中处理 `#1890ff`（蓝色）
- hover Tooltip："处理方法: XXX \n 处理数量: XXX 头 \n 占比: X.X%"

**堆叠柱状图（月度处理趋势）**：
- X 轴：时间（按月，与全局时间粒度联动），展示所选年度的时间序列
- Y 轴：处理数量（头）
- 按处理方法分色堆叠，颜色与饼图一致
- hover Tooltip："月份: YYYY-MM \n 焚烧: XXX 头 \n 化尸: XXX 头 \n 掩埋: XXX 头 \n 发酵: XXX 头 \n 委托集中处理: XXX 头 \n 合计: XXX 头"

**下方汇总表格**（始终展示）：

| 列名 | 数据来源 | 说明 |
|------|---------|------|
| 处理方法 | `pig_dead_animal_record.disposal_method` 枚举 | 固定 5 行：焚烧、化尸、掩埋、发酵、委托集中处理，末行为"合计" |
| 处理数量（头） | SUM `dead_quantity` 按 `disposal_method` 分组 | 整数 |
| 占比 | 各处理方法数量 / 总处理数量 × 100% | 百分比，保留 1 位小数 |
| 涉及企业数 | COUNT DISTINCT `enterprise_id` 按 `disposal_method` 分组 | 整数 |
| 同比变化 | 本年度处理数量与上一年度同处理方法对比 | 变化百分比，保留 1 位小数 |

**数据来源**：`pig_dead_animal_record`（`status = '4'` 且 `del_flag = '0'`），按 `disposal_method` 字段分组统计。`disposal_method` 枚举值：`burn` 焚烧、`dissolve` 化尸、`bury` 掩埋、`ferment` 发酵、`entrust` 委托集中处理，仅此 5 种取值。

---

#### 卡片七：免疫防疫覆盖分析

**展示形式**：左侧柱状图（50% 宽度）+ 右侧数据表格（50% 宽度）

**柱状图（按疫苗类型统计）**：
- 纵向柱状图，X 轴为疫苗名称，Y 轴为免疫数量（头次）
- 柱形颜色 `#1890ff`（蓝色），强制免疫病种柱形颜色 `#f5222d`（红色）
- 强制免疫病种：猪瘟、口蹄疫、高致病性猪蓝耳病，仅此 3 种
- hover Tooltip："疫苗名称: XXX \n 免疫数量: XXX 头次 \n 免疫企业数: XXX 家 \n 免疫覆盖率: X.X%"

**数据表格**：

| 列名 | 数据来源 | 说明 |
|------|---------|------|
| 疫苗名称 | `pig_immunity_record.vaccine_name` | 按疫苗名称分组 |
| 免疫数量（头次） | SUM `pig_immunity_record.immune_quantity`（推测字段） | 整数；如实际字段为记录条数则按 COUNT 统计 |
| 免疫企业数 | COUNT DISTINCT `enterprise_id` | 整数 |
| 应免数量（头） | 各企业最新已入库 `pig_breeding_info.stock_quantity` 之和（即存栏总量） | 整数 |
| 免疫覆盖率 | 免疫数量 / 应免数量 × 100% | 百分比，保留 1 位小数；应免数量为 0 时显示"—" |
| 同比变化 | 本年度免疫数量与上一年度同疫苗类型对比 | 变化百分比，保留 1 位小数 |

**预警标识规则**：
- 强制免疫病种（猪瘟、口蹄疫、高致病性猪蓝耳病）覆盖率低于阈值（系统参数 `quarantine.immunity_threshold`，默认 90）时，该行免疫覆盖率数字以红色 `#f5222d` 加粗显示，并在数字后附红色警示图标 `⚠`
- 非强制免疫病种不做预警标识

**数据来源**：`pig_immunity_record`（`status = '4'` 且 `del_flag = '0'`），按疫苗名称字段分组统计。

---

#### 卡片八：生猪保险统计

**展示形式**：上方统计卡片组（3 个水平排列）+ 中间组合图 + 下方可折叠明细表格

**统计卡片组**（3 个水平排列）：

| 序号 | 指标名称 | 计算逻辑 | 展示方式 |
|------|---------|---------|---------|
| 1 | 本年度投保总数（头） | SUM `pig_insurance_monthly.insured_quantity` | 整数数字 + 单位"头" + 较去年同期变化 |
| 2 | 本年度理赔总数（头） | SUM `pig_insurance_monthly.claimed_quantity` | 整数数字 + 单位"头" + 较去年同期变化 |
| 3 | 本年度理赔金额（万元） | SUM `pig_insurance_monthly.claimed_amount` / 10000 | 数字（保留 2 位小数）+ 单位"万元" + 较去年同期变化 |

**组合图（月度投保/理赔趋势）**：
- X 轴：月份（1月～12月）
- 左 Y 轴：数量（头），柱状图
  - 投保数量蓝色柱 `#1890ff`
  - 理赔数量橙色柱 `#fa8c16`
- 右 Y 轴：理赔金额（元），红色折线 `#f5222d`
- hover Tooltip："月份: X月 \n 投保数量: XXX 头 \n 理赔数量: XXX 头 \n 理赔金额: XXX 元"
- 支持图例（Legend）点击隐藏/显示数据系列

**可折叠明细表格**（默认收起）：

| 列名 | 数据来源 | 说明 |
|------|---------|------|
| 月份 | `pig_insurance_monthly.stat_month` | 格式"X月" |
| 投保数量（头） | `insured_quantity` | 整数 |
| 理赔数量（头） | `claimed_quantity` | 整数 |
| 理赔金额（元） | `claimed_amount` | 保留 2 位小数 |
| 理赔率 | 理赔数量 / 投保数量 × 100% | 百分比，保留 1 位小数；投保数量为 0 时显示"—" |

**数据来源**：`pig_insurance_monthly`，按 `stat_month` 筛选所选年度数据。

> 保险数据的获取方式为管理员通过 Excel 导入或逐条手动录入。如后续与保险公司完成 API 对接，可自动入库（`data_source = 'api'`），但页面展示逻辑不变。

---

#### 外部数据导入功能

**入口位置**：分析页面右上角「数据管理」下拉按钮（仅 `county_admin` 角色展示），包含 2 个子项：
- 「导入检疫数据」
- 「导入保险数据」

**检疫数据导入流程**：

```
点击「导入检疫数据」
    ↓
弹出导入弹窗，提供「下载导入模板」链接
    ↓
用户下载 Excel 模板，按模板格式填写数据
    ↓
上传 Excel 文件
    ↓
系统校验数据格式（必填项、类型、枚举值），校验通过后入库
    ↓
导入成功提示"导入成功，共导入 X 条数据"，页面分析数据自动刷新
    ↓
校验失败提示具体错误行号和错误原因
```

**检疫数据导入模板字段**：

| 列名 | 对应字段 | 必填 | 校验规则 |
|------|---------|------|---------|
| 检疫类目 | `quarantine_type` | 是 | 必须为"产地检疫""屠宰检疫""调运检疫"之一 |
| 检疫日期 | `quarantine_date` | 是 | 日期格式 YYYY-MM-DD |
| 检疫数量（头次） | `quantity` | 是 | 正整数，最小值 1，最大值 999999 |
| 合格数量（头次） | `qualified_quantity` | 是 | 正整数，最小值 0，最大值不超过检疫数量 |
| 检疫单位 | `quarantine_org` | 否 | 最大长度 100 字符 |
| 养殖企业名称 | `enterprise_name` | 否 | 最大长度 200 字符 |
| 所属乡镇 | `area_name` | 否 | 最大长度 100 字符 |
| 检出问题 | `issue_desc` | 否 | 最大长度 500 字符 |

导入后数据自动标记 `data_source = 'manual_import'`。

**保险数据导入流程**：

支持两种录入方式：

**方式一：Excel 导入**

| 列名 | 对应字段 | 必填 | 校验规则 |
|------|---------|------|---------|
| 统计年月 | `stat_month` | 是 | 格式 YYYY-MM |
| 所属乡镇 | `area_name` | 否 | 最大长度 100 字符；为空视为全县汇总 |
| 投保数量（头） | `insured_quantity` | 是 | 非负整数，最小值 0，最大值 9999999 |
| 理赔数量（头） | `claimed_quantity` | 是 | 非负整数，最小值 0，最大值不超过投保数量 |
| 理赔金额（元） | `claimed_amount` | 是 | 非负数，最小值 0，最大值 99999999999.99，保留 2 位小数 |

导入后数据自动标记 `data_source = 'manual'`。

**方式二：逐条录入**

点击「导入保险数据」弹窗中的「手动录入」标签页，展示表单：

| 字段名 | 控件类型 | 必填 | 校验规则 |
|-------|---------|------|---------|
| 统计年月 | 月份选择器 | 是 | 格式 YYYY-MM |
| 所属乡镇 | 下拉选择 | 否 | 含"全县汇总"选项；数据来自 `GET /system/dept/treeselect` |
| 投保数量（头） | 数字输入框 | 是 | 非负整数，最小值 0，最大值 9999999 |
| 理赔数量（头） | 数字输入框 | 是 | 非负整数，最小值 0，最大值不超过投保数量 |
| 理赔金额（元） | 数字输入框 | 是 | 非负数，最小值 0，最大值 99999999999.99，保留 2 位小数 |

提交后数据标记 `data_source = 'manual'`。如同一 `stat_month` + `area_name` 已存在记录，提示"该月份该乡镇已有数据，是否覆盖？"，用户确认后覆盖更新。

### 3.4.2 数据模型与字段定义

#### 新建表一：检疫记录表 `pig_quarantine_record`

| 字段名（中文） | 字段名（英文） | 类型 | 必填 | 默认值 | 取值范围/枚举值 | 说明 |
|--------------|--------------|------|------|-------|---------------|------|
| 记录ID | `id` | bigint(20) | 是 | 自增 | — | 主键，自增 |
| 检疫类目 | `quarantine_type` | varchar(20) | 是 | — | `production` 产地检疫、`slaughter` 屠宰检疫、`transport` 调运检疫，仅此 3 种取值 | — |
| 检疫日期 | `quarantine_date` | date | 是 | — | 合法日期，不得晚于当前日期 | — |
| 检疫数量 | `quantity` | int | 是 | — | 正整数，最小值 1，最大值 999999 | 单位：头次 |
| 合格数量 | `qualified_quantity` | int | 是 | — | 非负整数，最小值 0，最大值不超过 `quantity` 的值 | 单位：头次 |
| 检疫单位 | `quarantine_org` | varchar(100) | 否 | NULL | 最大长度 100 字符 | 实施检疫的机构名称 |
| 养殖企业名称 | `enterprise_name` | varchar(200) | 否 | NULL | 最大长度 200 字符 | 被检疫的企业名称 |
| 企业ID | `enterprise_id` | bigint(20) | 否 | NULL | — | 关联本县 `pig_enterprise.enterprise_id`；省级数据可能为 NULL |
| 所属乡镇 | `area_name` | varchar(100) | 否 | NULL | 最大长度 100 字符 | — |
| 检出问题 | `issue_desc` | varchar(500) | 否 | NULL | 最大长度 500 字符 | 检疫中发现的问题描述 |
| 数据来源 | `data_source` | varchar(50) | 是 | — | `province_platform` 省级平台自动对接、`manual_import` 手动 Excel 导入，仅此 2 种取值 | — |
| 创建者 | `create_by` | varchar(64) | 否 | NULL | 最大长度 64 字符 | 手动导入时为操作用户名，API 对接时为系统标识 |
| 创建时间 | `create_time` | datetime | 是 | CURRENT_TIMESTAMP | — | 数据入库时间 |
| 更新者 | `update_by` | varchar(64) | 否 | NULL | 最大长度 64 字符 | — |
| 更新时间 | `update_time` | datetime | 否 | NULL | — | — |
| 删除标志 | `del_flag` | char(1) | 是 | `0` | `0` 存在、`2` 已删除，仅此 2 种取值 | 逻辑删除标志 |

**索引设计**：
- 主键索引：`id`
- 普通索引：`idx_quarantine_type`（`quarantine_type`）
- 普通索引：`idx_quarantine_date`（`quarantine_date`）
- 普通索引：`idx_enterprise_id`（`enterprise_id`）
- 普通索引：`idx_data_source`（`data_source`）
- 组合索引：`idx_type_date`（`quarantine_type`, `quarantine_date`）

#### 新建表二：保险统计月报表 `pig_insurance_monthly`

| 字段名（中文） | 字段名（英文） | 类型 | 必填 | 默认值 | 取值范围/枚举值 | 说明 |
|--------------|--------------|------|------|-------|---------------|------|
| 记录ID | `id` | bigint(20) | 是 | 自增 | — | 主键，自增 |
| 统计年月 | `stat_month` | varchar(7) | 是 | — | 格式 YYYY-MM，如 `2026-03`，最大长度 7 字符 | — |
| 所属乡镇 | `area_name` | varchar(100) | 否 | NULL | 最大长度 100 字符；NULL 表示全县汇总 | — |
| 投保数量 | `insured_quantity` | int | 是 | — | 非负整数，最小值 0，最大值 9999999 | 单位：头 |
| 理赔数量 | `claimed_quantity` | int | 是 | — | 非负整数，最小值 0，最大值不超过 `insured_quantity` 的值 | 单位：头 |
| 理赔金额 | `claimed_amount` | decimal(12,2) | 是 | — | 非负数，最小值 0，最大值 99999999999.99 | 单位：元 |
| 数据来源 | `data_source` | varchar(50) | 是 | — | `api` 外部接口自动对接、`manual` 手动录入或 Excel 导入，仅此 2 种取值 | — |
| 创建者 | `create_by` | varchar(64) | 否 | NULL | 最大长度 64 字符 | — |
| 创建时间 | `create_time` | datetime | 是 | CURRENT_TIMESTAMP | — | — |
| 更新者 | `update_by` | varchar(64) | 否 | NULL | 最大长度 64 字符 | — |
| 更新时间 | `update_time` | datetime | 是 | CURRENT_TIMESTAMP | — | — |
| 删除标志 | `del_flag` | char(1) | 是 | `0` | `0` 存在、`2` 已删除，仅此 2 种取值 | 逻辑删除标志 |

**索引设计**：
- 主键索引：`id`
- 普通索引：`idx_stat_month`（`stat_month`）
- 普通索引：`idx_area_name`（`area_name`）
- 唯一索引：`uk_month_area`（`stat_month`, `area_name`）—— 同一年月+乡镇仅允许一条记录（用于逐条录入时的覆盖判断）

#### 复用现有表（统计数据来源）

| 分析内容 | 数据来源表 | 关键聚合字段 | 复用说明 |
|---------|----------|-----------|---------|
| 疫病数量及分布 | `pig_disease_treatment_record` | `disease_name`、`enterprise_id`、`create_time` | 复用现有表，取 `status = '4'`，按疫病名称分组统计 |
| 免疫防疫覆盖 | `pig_immunity_record` | `vaccine_name`、`enterprise_id`、`create_time` | 复用现有表，取 `status = '4'`，按疫苗类型分组统计 |
| 防疫监测 | `pig_epidemic_monitor_record` | `enterprise_id`、`create_time` | 复用现有表，取 `status = '4'` |
| 病死猪数量/分布/处理 | `pig_dead_animal_record` | `dead_quantity`、`disposal_method`、`death_cause`、`enterprise_id`、`create_time` | 复用现有表，取 `status = '4'`，按处理方法和乡镇分组统计 |
| 存栏总量（病死率/免疫覆盖率分母） | `pig_breeding_info` | `stock_quantity`、`enterprise_id` | 复用现有表，取 `status = '4'`，各企业最新记录的存栏量 |
| 企业信息关联 | `pig_enterprise` | `enterprise_id`、`enterprise_name`、`dept_id` | 复用现有表，关联企业名称和所属乡镇 |
| 乡镇维度 | `sys_dept` | `dept_id`、`dept_name` | 复用现有表，乡镇筛选与分组 |
| 预警阈值参数 | `sys_config` | `config_key`、`config_value` | 复用现有表，存储病死率和免疫覆盖率阈值 |

#### 各分析接口通用请求参数

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `year` | int | 选填 | 统计年度，默认当前年度，范围 2020 至当前年度 |
| `areaCode` | string | 选填 | 乡镇编码（`sys_dept.dept_id`），为空时按角色数据范围自动过滤 |
| `granularity` | string | 选填 | 时间粒度，枚举值：`month` 按月、`quarter` 按季、`year` 按年，默认 `month`，仅此 3 种取值 |

#### 检疫年度核心指标响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `quarantineTotal` | int | 检验检疫总量（头次），最小值 0 |
| `quarantineChange` | decimal(5,1) | 较去年同期变化百分比，正值增长负值下降，去年为 0 时返回 null |
| `productionTotal` | int | 产地检疫数量（头次），最小值 0 |
| `productionChange` | decimal(5,1) | 同比变化百分比 |
| `slaughterTotal` | int | 屠宰检疫数量（头次），最小值 0 |
| `slaughterChange` | decimal(5,1) | 同比变化百分比 |
| `qualifiedRate` | decimal(4,1) | 检验检疫合格率（%），保留 1 位小数；检疫总量为 0 时返回 null |
| `diseaseTotal` | int | 疫病发现数量（例），最小值 0 |
| `diseaseChange` | decimal(5,1) | 同比变化百分比 |
| `deadAnimalTotal` | int | 病死猪处理数量（头），最小值 0 |
| `deadAnimalChange` | decimal(5,1) | 同比变化百分比 |

#### 检疫类目分布响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 分布数据数组（固定 3 个元素 + 合计行） |
| `items[].quarantineType` | string | 检疫类目编码，`production`/`slaughter`/`transport` |
| `items[].quarantineTypeName` | string | 检疫类目名称 |
| `items[].quantity` | int | 数量（头次），最小值 0 |
| `items[].ratio` | decimal(4,1) | 占比（%），保留 1 位小数 |
| `items[].qualifiedQuantity` | int | 合格数（头次），最小值 0 |
| `items[].qualifiedRate` | decimal(4,1) | 合格率（%），保留 1 位小数；数量为 0 时返回 null |
| `items[].yoyChange` | decimal(5,1) | 同比变化百分比 |

#### 检验检疫趋势响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 趋势数据数组 |
| `items[].period` | string | 时间周期标识：YYYY-MM（月）、YYYY-Q1/Q2/Q3/Q4（季）、YYYY（年） |
| `items[].productionQuantity` | int | 产地检疫数量（头次），最小值 0 |
| `items[].slaughterQuantity` | int | 屠宰检疫数量（头次），最小值 0 |
| `items[].transportQuantity` | int | 调运检疫数量（头次），最小值 0 |
| `items[].qualifiedRate` | decimal(4,1) | 该周期合格率（%），保留 1 位小数 |

#### 疫病数量及分布响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `diseaseRanking` | array | 疫病类型 Top10 排行数组 |
| `diseaseRanking[].diseaseName` | string | 疫病名称 |
| `diseaseRanking[].count` | int | 发病数量（例），最小值 0 |
| `diseaseRanking[].enterpriseCount` | int | 涉及企业数 |
| `townDistribution` | array | 乡镇分布数组（用于热力图） |
| `townDistribution[].townCode` | string | 乡镇编码（`sys_dept.dept_id`） |
| `townDistribution[].townName` | string | 乡镇名称 |
| `townDistribution[].diseaseCount` | int | 疫病数量（例） |
| `townDistribution[].enterpriseCount` | int | 涉及企业数 |
| `tableData` | array | 可折叠表格数据数组 |
| `tableData[].diseaseName` | string | 疫病名称 |
| `tableData[].count` | int | 发病数量（例） |
| `tableData[].enterpriseCount` | int | 涉及企业数 |
| `tableData[].towns` | string | 涉及乡镇名称，逗号分隔 |
| `tableData[].peakMonth` | string | 高发月份，格式"X月" |
| `tableData[].yoyChange` | decimal(5,1) | 同比变化百分比 |

#### 疫病乡镇下钻明细响应数据结构

额外请求参数：

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `townCode` | string | 是 | 乡镇编码 |
| `pageNum` | int | 选填 | 页码，默认 1，最小值 1 |
| `pageSize` | int | 选填 | 每页条数，默认 10，最小值 1，最大值 100 |

响应数据：

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `total` | int | 总记录数 |
| `rows` | array | 明细数组 |
| `rows[].enterpriseName` | string | 企业名称 |
| `rows[].diseaseName` | string | 疫病名称 |
| `rows[].count` | int | 发病数量（例） |
| `rows[].latestDate` | string | 最近发病日期，格式 YYYY-MM-DD |

#### 病死猪数量及分布响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `currentYearTrend` | array | 本年度月度趋势数组 |
| `currentYearTrend[].month` | string | 月份，格式 YYYY-MM |
| `currentYearTrend[].quantity` | int | 病死猪数量（头），最小值 0 |
| `lastYearTrend` | array | 去年同期月度趋势数组 |
| `lastYearTrend[].month` | string | 月份，格式 YYYY-MM |
| `lastYearTrend[].quantity` | int | 病死猪数量（头），最小值 0 |
| `townDistribution` | array | 乡镇分布数组（用于热力图） |
| `townDistribution[].townCode` | string | 乡镇编码 |
| `townDistribution[].townName` | string | 乡镇名称 |
| `townDistribution[].deadQuantity` | int | 病死数量（头） |
| `townDistribution[].stockTotal` | int | 存栏总量（头） |
| `townDistribution[].deadRate` | decimal(4,1) | 病死率（%），保留 1 位小数；存栏为 0 时返回 null |
| `townDistribution[].enterpriseCount` | int | 涉及企业数 |
| `townDistribution[].overThreshold` | boolean | 是否超过病死率预警阈值，true/false |
| `tableData` | array | 可折叠表格数据数组 |
| `tableData[].townName` | string | 乡镇名称 |
| `tableData[].deadQuantity` | int | 病死数量（头） |
| `tableData[].stockTotal` | int | 存栏总量（头） |
| `tableData[].deadRate` | decimal(4,1) | 病死率（%） |
| `tableData[].enterpriseCount` | int | 涉及企业数 |
| `tableData[].mainCause` | string | 主要死因 |
| `tableData[].yoyChange` | decimal(5,1) | 同比变化百分比 |

#### 病死猪乡镇下钻明细响应数据结构

额外请求参数：

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `townCode` | string | 是 | 乡镇编码 |
| `pageNum` | int | 选填 | 页码，默认 1，最小值 1 |
| `pageSize` | int | 选填 | 每页条数，默认 10，最小值 1，最大值 100 |

响应数据：

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `total` | int | 总记录数 |
| `rows` | array | 明细数组 |
| `rows[].enterpriseName` | string | 企业名称 |
| `rows[].deadQuantity` | int | 病死数量（头） |
| `rows[].deathCause` | string | 死因 |
| `rows[].disposalMethod` | string | 处理方法名称 |
| `rows[].recordDate` | string | 记录日期，格式 YYYY-MM-DD |

#### 病死猪处理情况分析响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `pieData` | array | 饼图数据数组（固定 5 个元素） |
| `pieData[].disposalMethod` | string | 处理方法编码：`burn`/`dissolve`/`bury`/`ferment`/`entrust` |
| `pieData[].disposalMethodName` | string | 处理方法名称 |
| `pieData[].quantity` | int | 处理数量（头），最小值 0 |
| `pieData[].ratio` | decimal(4,1) | 占比（%），保留 1 位小数 |
| `trendData` | array | 堆叠柱状图月度趋势数组 |
| `trendData[].period` | string | 时间周期标识 |
| `trendData[].burnQuantity` | int | 焚烧数量（头），最小值 0 |
| `trendData[].dissolveQuantity` | int | 化尸数量（头），最小值 0 |
| `trendData[].buryQuantity` | int | 掩埋数量（头），最小值 0 |
| `trendData[].fermentQuantity` | int | 发酵数量（头），最小值 0 |
| `trendData[].entrustQuantity` | int | 委托集中处理数量（头），最小值 0 |
| `tableData` | array | 汇总表格数据数组（固定 5 个元素 + 合计行） |
| `tableData[].disposalMethodName` | string | 处理方法名称 |
| `tableData[].quantity` | int | 处理数量（头） |
| `tableData[].ratio` | decimal(4,1) | 占比（%） |
| `tableData[].enterpriseCount` | int | 涉及企业数 |
| `tableData[].yoyChange` | decimal(5,1) | 同比变化百分比 |

#### 免疫防疫覆盖分析响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `items` | array | 疫苗类型统计数组 |
| `items[].vaccineName` | string | 疫苗名称 |
| `items[].isMandatory` | boolean | 是否为强制免疫病种，true/false |
| `items[].immuneQuantity` | int | 免疫数量（头次），最小值 0 |
| `items[].enterpriseCount` | int | 免疫企业数 |
| `items[].requiredQuantity` | int | 应免数量（头）= 存栏总量，最小值 0 |
| `items[].coverageRate` | decimal(4,1) | 免疫覆盖率（%），保留 1 位小数；应免数量为 0 时返回 null |
| `items[].belowThreshold` | boolean | 是否低于免疫覆盖率阈值（仅强制免疫病种有效），true/false |
| `items[].yoyChange` | decimal(5,1) | 同比变化百分比 |

#### 生猪保险统计响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `insuredTotal` | int | 本年度投保总数（头），最小值 0 |
| `insuredChange` | decimal(5,1) | 较去年同期变化百分比 |
| `claimedTotal` | int | 本年度理赔总数（头），最小值 0 |
| `claimedChange` | decimal(5,1) | 较去年同期变化百分比 |
| `claimedAmountTotal` | decimal(12,2) | 本年度理赔金额总计（元） |
| `claimedAmountChange` | decimal(5,1) | 较去年同期变化百分比 |
| `monthlyData` | array | 月度明细数组 |
| `monthlyData[].month` | string | 月份，格式 YYYY-MM |
| `monthlyData[].insuredQuantity` | int | 投保数量（头） |
| `monthlyData[].claimedQuantity` | int | 理赔数量（头） |
| `monthlyData[].claimedAmount` | decimal(12,2) | 理赔金额（元） |
| `monthlyData[].claimRate` | decimal(4,1) | 理赔率（%），保留 1 位小数；投保数量为 0 时返回 null |

#### 导入检疫数据响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `successCount` | int | 成功导入条数 |
| `failCount` | int | 失败条数 |
| `errors` | array | 错误明细数组（仅 failCount > 0 时有值） |
| `errors[].rowNum` | int | 错误行号 |
| `errors[].message` | string | 错误原因 |

#### 导入保险数据响应数据结构

| 字段名（英文） | 类型 | 说明 |
|-------------|------|------|
| `successCount` | int | 成功导入条数 |
| `failCount` | int | 失败条数 |
| `errors` | array | 错误明细数组 |
| `errors[].rowNum` | int | 错误行号 |
| `errors[].message` | string | 错误原因 |

#### 新增保险月报请求数据结构

| 参数名（英文） | 类型 | 必填 | 说明 |
|-------------|------|------|------|
| `statMonth` | string | 是 | 统计年月，格式 YYYY-MM |
| `areaName` | string | 否 | 所属乡镇，为空视为全县汇总 |
| `insuredQuantity` | int | 是 | 投保数量（头），非负整数，最小值 0，最大值 9999999 |
| `claimedQuantity` | int | 是 | 理赔数量（头），非负整数，最小值 0，最大值不超过 insuredQuantity |
| `claimedAmount` | decimal(12,2) | 是 | 理赔金额（元），非负数，最小值 0，最大值 99999999999.99 |

### 3.4.3 界面交互逻辑

**检验检疫无害化分析页面整体布局**：

- 页面标题：检验检疫无害化分析。
- 页面顶部全局筛选栏（固定，不随页面滚动）：
  - 统计年度（年份选择器，默认当前年度）
  - 所属乡镇（下拉选择，单选，含"全县"选项；乡镇角色不展示此筛选项）
  - 时间粒度（按钮切换组：按月/按季/按年，默认按月）
  - 右侧按钮组：「数据管理」下拉按钮（仅 `county_admin` 角色展示）+ 「导出分析报告」按钮（蓝色主按钮，仅 `county_admin` 和 `county_staff` 角色展示）
- 分析卡片区域（筛选栏下方，可滚动）：
  - 第一行：检疫年度核心指标（6 个指标卡，通栏，两行每行 3 个）
  - 第二行：检疫类目分布（左半栏）+ 检验检疫趋势（右半栏）
  - 第三行：疫病数量及分布（通栏，左半柱状图 + 右半热力图 + 下方可折叠表格）
  - 第四行：病死猪数量及分布（通栏，左半折线图 + 右半热力图 + 下方可折叠表格）
  - 第五行：病死猪处理情况分析（通栏，左饼图 + 右堆叠柱状图 + 下方汇总表格）
  - 第六行：免疫防疫覆盖分析（通栏，左柱状图 + 右数据表格）
  - 第七行：生猪保险统计（通栏，上方统计卡片 + 中间组合图 + 下方可折叠表格）

**筛选交互**：
- 全局筛选条件变更后，所有分析卡片**即时同步刷新**，无需额外点击"查询"按钮。
- 时间粒度切换后，涉及时间序列的图表（检疫趋势、处理趋势等）自动重新加载。
- 乡镇角色进入页面时，乡镇筛选项不可见，系统按登录用户的 `dept_id` 自动过滤。
- 省级平台导入的检疫汇总数据（卡片一、二、三）不受乡镇筛选过滤（省级数据可能无法精确到乡镇粒度）；本县企业数据（卡片四～八）按乡镇筛选过滤。

**图表通用交互**：
- 所有图表使用 ECharts 渲染，最小高度 300px。
- 所有图表支持 hover Tooltip 交互。
- 折线图和柱状图支持图例（Legend）点击隐藏/显示数据系列。
- 乡镇热力图使用蓬溪县自定义 GeoJSON 行政区划地图数据。

**热力图交互**：
- 疫病分布和病死猪分布的乡镇热力图支持 hover 查看详情和点击下钻。
- 点击乡镇后弹出下钻明细列表弹窗（模态弹窗），弹窗标题为"XXX镇 - 疫病明细"或"XXX镇 - 病死猪明细"。
- 弹窗支持分页，每页 10 条，弹窗底部有「关闭」按钮。

**可折叠数据表格**：
- 疫病分布、病死猪分布、保险统计的数据表格默认收起。
- 病死猪处理的汇总表格始终展示。
- 卡片标题栏右侧有「展开数据」/「收起数据」切换图标按钮。

**数据管理交互**：
- 「数据管理」下拉按钮仅 `county_admin` 角色可见。
- 点击展开下拉菜单：「导入检疫数据」「导入保险数据」。
- 导入弹窗包含：上传区域（拖拽或点击上传 Excel 文件，仅接受 `.xlsx` 和 `.xls` 格式）+ 「下载导入模板」链接 + 导入结果展示区。
- 保险数据导入弹窗有两个标签页：「Excel 导入」和「手动录入」。标签页切换为即时响应，内容区域自动切换。
- 导入成功后页面分析数据自动刷新。

**导出交互**：
- 页面顶部「导出分析报告」按钮仅县级角色（`county_admin` 和 `county_staff`）可见。
- 点击后导出包含所有分析卡片数据表格的完整 Excel 报告。
- 每个分析卡片的数据占一个 Sheet 页。
- 文件名格式：`检验检疫无害化分析_{年度}_YYYYMMDD_HHmmss.xlsx`，如 `检验检疫无害化分析_2026_20260310_143000.xlsx`。
- 导出仅包含数据表格，不含图表图片。

**加载状态**：
- 页面进入和筛选条件变更后，各分析卡片独立展示 loading 状态（旋转加载图标 + "加载中..."文字）。
- 某个接口加载失败时，对应卡片展示错误提示"数据加载失败，请重试"和「重试」按钮，不影响其他卡片正常展示。

**空状态处理**：
- 检疫数据为空时（无检疫记录），卡片一～三展示"暂无检疫数据，请先导入检疫数据"。
- 疫病分布无数据时，展示"暂无疫病诊疗记录"。
- 病死猪分布无数据时，展示"暂无病死畜禽处理记录"。
- 免疫覆盖分析无数据时，展示"暂无免疫记录"。
- 保险统计无数据时，展示"暂无保险统计数据，请先导入保险数据"。
- 热力图无数据时，地图以全灰色展示，中间标注"暂无数据"。

**校验规则**：

| 校验字段 | 校验类型 | 校验规则 | 错误提示文案 |
|---------|---------|---------|------------|
| 统计年度 | 范围 | 2020 至当前年度 | 「请选择有效的统计年度」 |
| 导入文件-文件格式 | 格式 | 仅接受 `.xlsx` 和 `.xls` 文件 | 「仅支持 Excel 文件（.xlsx、.xls）」 |
| 导入文件-文件大小 | 范围 | 最大 10MB | 「文件大小不能超过 10MB」 |
| 检疫导入-检疫类目 | 枚举 | 必须为"产地检疫""屠宰检疫""调运检疫"之一 | 「第X行：检疫类目无效，必须为产地检疫、屠宰检疫或调运检疫」 |
| 检疫导入-检疫日期 | 格式 | 日期格式 YYYY-MM-DD | 「第X行：检疫日期格式不正确，应为 YYYY-MM-DD」 |
| 检疫导入-检疫数量 | 范围 | 正整数，1 ～ 999999 | 「第X行：检疫数量必须为 1 到 999999 之间的正整数」 |
| 检疫导入-合格数量 | 逻辑 | 非负整数，0 ～ 检疫数量 | 「第X行：合格数量不能超过检疫数量」 |
| 保险录入-统计年月 | 必填+格式 | 不能为空，格式 YYYY-MM | 「统计年月不能为空」/「统计年月格式不正确」 |
| 保险录入-投保数量 | 必填+范围 | 非负整数，0 ～ 9999999 | 「投保数量不能为空」/「投保数量必须为 0 到 9999999 之间的整数」 |
| 保险录入-理赔数量 | 逻辑 | 非负整数，不超过投保数量 | 「理赔数量不能超过投保数量」 |
| 保险录入-理赔金额 | 必填+范围 | 非负数，0 ～ 99999999999.99 | 「理赔金额不能为空」/「理赔金额超出允许范围」 |

### 3.4.4 接口路径定义

| 接口名称 | HTTP 方法 | 路径 | 请求参数摘要 | 响应数据摘要 | 备注 |
|----------|-----------|------|-------------|-------------|------|
| 检疫年度核心指标 | GET | `/admin/analysis/quarantine/overview` | year、areaCode | 6 个指标值 + 同比变化百分比 | 🆕 新增接口 |
| 检疫类目分布 | GET | `/admin/analysis/quarantine/category` | year、areaCode | 3 类检疫类目的数量、占比、合格率 | 🆕 新增接口 |
| 检验检疫趋势 | GET | `/admin/analysis/quarantine/trend` | year、areaCode、granularity | 时间序列检疫数量和合格率 | 🆕 新增接口 |
| 疫病数量及分布 | GET | `/admin/analysis/quarantine/disease` | year、areaCode | 疫病 Top10 排行 + 乡镇分布 + 表格数据 | 🆕 新增接口 |
| 疫病乡镇下钻明细 | GET | `/admin/analysis/quarantine/disease/detail` | year、townCode、pageNum、pageSize | 分页疫病明细列表 | 🆕 新增接口 |
| 病死猪数量及分布 | GET | `/admin/analysis/quarantine/dead-animal` | year、areaCode | 月度趋势 + 去年对比 + 乡镇分布 + 表格数据 | 🆕 新增接口 |
| 病死猪乡镇下钻明细 | GET | `/admin/analysis/quarantine/dead-animal/detail` | year、townCode、pageNum、pageSize | 分页病死猪明细列表 | 🆕 新增接口 |
| 病死猪处理情况分析 | GET | `/admin/analysis/quarantine/disposal` | year、areaCode、granularity | 饼图 + 趋势 + 汇总表格数据 | 🆕 新增接口 |
| 免疫防疫覆盖分析 | GET | `/admin/analysis/quarantine/immunity` | year、areaCode | 按疫苗类型分组的免疫统计 | 🆕 新增接口 |
| 生猪保险统计 | GET | `/admin/analysis/quarantine/insurance` | year、areaCode | 投保/理赔汇总 + 月度明细 | 🆕 新增接口 |
| 导入检疫数据 | POST | `/admin/analysis/quarantine/import` | Excel 文件（multipart/form-data） | 导入结果（成功数/失败数/错误明细） | 🆕 新增接口 |
| 检疫导入模板下载 | POST | `/admin/analysis/quarantine/import-template` | — | Excel 模板文件流 | 🆕 新增接口 |
| 导入保险数据 | POST | `/admin/analysis/quarantine/insurance/import` | Excel 文件（multipart/form-data） | 导入结果（成功数/失败数/错误明细） | 🆕 新增接口 |
| 保险导入模板下载 | POST | `/admin/analysis/quarantine/insurance/import-template` | — | Excel 模板文件流 | 🆕 新增接口 |
| 新增保险月报 | POST | `/admin/analysis/quarantine/insurance/data` | 见 3.4.2 新增保险月报请求数据结构 | 操作结果 | 🆕 新增接口 |
| 导出分析报告 | POST | `/admin/analysis/quarantine/export` | year、areaCode | Excel 文件流（多 Sheet） | 🆕 新增接口 |
| 部门树下拉（乡镇筛选） | GET | `/system/dept/treeselect` | — | 部门树结构 | ✅ 复用现有接口，见现有业务说明.spec.md §三 |
| 字典数据 | GET | `/system/dict/data/type/{dictType}` | — | 字典枚举列表 | ✅ 复用现有接口，见现有业务说明.spec.md §三 |
| 系统参数查询 | GET | `/system/config/configKey/{configKey}` | configKey | 参数值 | ✅ 复用现有接口，见现有业务说明.spec.md §三 |

## 3.5 非功能需求

- **安全与权限**：
  - 需具备 `admin:analysis:quarantine` 权限标识方可访问检验检疫无害化分析页面。
  - 需具备 `admin:analysis:quarantine:import` 权限标识方可执行检疫/保险数据导入操作。
  - 需具备 `admin:analysis:quarantine:export` 权限标识方可执行导出分析报告操作。
  - 权限分配：

  | 权限标识 | 说明 | 拥有角色 |
  |---------|------|---------|
  | `admin:analysis:quarantine` | 检验检疫无害化分析查看 | `county_admin`、`county_staff`、`town_gov_staff`、`town_vet_station` |
  | `admin:analysis:quarantine:import` | 导入检疫/保险数据 | `county_admin` |
  | `admin:analysis:quarantine:export` | 导出分析报告 | `county_admin`、`county_staff` |

  - 数据权限按 `data_scope` + 部门树自动过滤：县级角色（`county_admin`、`county_staff`）看全县数据（`data_scope=1`），乡镇角色（`town_gov_staff`、`town_vet_station`）看本乡镇数据（`data_scope=4`）。
  - 省级平台导入的检疫数据（`pig_quarantine_record`）为全县级别汇总，不做乡镇级数据权限过滤。
  - 角色定义引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1。

- **性能**：
  - 各分析接口独立返回，单个接口响应时间不超过 5 秒。
  - 页面采用异步并行加载策略：所有分析接口并行请求，各卡片独立渲染，不因某一个慢的接口阻塞整个页面。
  - 乡镇热力图的 GeoJSON 地图数据建议前端缓存，避免重复加载。
  - Excel 导入接口支持最大 5000 行数据，超过 5000 行时提示"单次导入不能超过 5000 行"。
  - 如后续数据量增大导致聚合查询性能不佳，可引入定时任务生成统计快照表进行优化，初期不做。

- **兼容性**：桌面端主流分辨率（1280px 及以上）正常显示。图表组件使用 ECharts，乡镇地图使用自定义蓬溪县行政区划 GeoJSON 数据，与大屏及其他分析模块保持技术栈统一。

## 3.6 验收标准

- [ ] 检验检疫无害化分析页面展示全局筛选栏和 8 个分析卡片，布局与 3.4.3 定义一致。
- [ ] 全局筛选条件（年度、乡镇、时间粒度）变更后，所有分析卡片即时同步刷新，无需额外点击"查询"按钮。
- [ ] 本县企业数据（卡片四～八）仅基于已入库（`status = '4'`）的业务数据，草稿和审批中的数据不纳入统计。
- [ ] 县级角色看全县分析数据，乡镇角色仅看本乡镇分析数据（省级检疫数据不受乡镇过滤）。
- [ ] 乡镇角色不展示乡镇筛选项、不展示「数据管理」按钮、不展示「导出分析报告」按钮。
- [ ] **检疫年度核心指标**：6 个指标数字卡片正确展示检疫总量、产地检疫、屠宰检疫、合格率、疫病数量、病死猪处理数量，各指标附有同比变化百分比。合格率按阈值显示对应颜色（≥99%绿色，95%-99%黄色，<95%红色）。
- [ ] **检疫类目分布**：饼图正确展示产地检疫/屠宰检疫/调运检疫的数量占比，数据表格正确展示各类目的数量、占比、合格数、合格率。
- [ ] **检验检疫趋势**：组合图正确展示按类目堆叠的检疫数量（柱状图）和合格率（折线图），合格率低于 95% 的数据点放大标注。
- [ ] **疫病数量及分布**：柱状图展示 Top10 疫病类型；乡镇热力图正确着色展示各乡镇疫病数量；点击乡镇可下钻查看明细列表。
- [ ] **病死猪数量及分布**：折线图展示本年度 vs 去年同期趋势对比；乡镇热力图正确着色，病死率超阈值的乡镇标注红色三角预警图标；点击乡镇可下钻查看明细。
- [ ] **病死猪处理情况分析**：饼图正确展示 5 种处理方法占比；堆叠柱状图展示月度各处理方法数量构成；汇总表格数据准确。
- [ ] **免疫防疫覆盖分析**：柱状图按疫苗类型展示免疫数量，强制免疫病种柱形红色；数据表格中强制免疫病种覆盖率低于阈值时红色加粗并附警示图标。
- [ ] **生猪保险统计**：3 个统计卡片展示投保/理赔汇总；组合图展示月度投保、理赔数量和金额趋势；明细表格可折叠展开。
- [ ] 县级管理员可通过「数据管理」→「导入检疫数据」上传 Excel 文件导入检疫数据，导入后分析数据自动刷新。
- [ ] 县级管理员可通过「数据管理」→「导入保险数据」上传 Excel 或手动录入保险月报数据，同一月份+乡镇重复录入时提示覆盖确认。
- [ ] 导入模板可正常下载，导入校验失败时显示具体错误行号和错误原因。
- [ ] 「导出分析报告」按钮仅县级角色可见，点击导出多 Sheet Excel 文件。
- [ ] 所有图表 hover 时展示具体数值 Tooltip。
- [ ] 各分析卡片独立加载，某个加载失败不影响其他卡片展示，失败卡片展示重试按钮。
- [ ] 各卡片在无数据时展示对应空状态提示。
- [ ] 预警阈值参数（病死率阈值、免疫覆盖率阈值）可通过系统管理→参数设置（`sys_config`）调整，无需修改代码。
- [ ] 见可交互 HTML 原型。

---

## 附录

### A. 分析卡片与数据表对照总表

| 分析卡片 | 涉及数据表 | 关键聚合字段 |
|---------|----------|-----------|
| 检疫年度核心指标 | `pig_quarantine_record`、`pig_disease_treatment_record`、`pig_dead_animal_record` | quantity、qualified_quantity、disease_name、dead_quantity |
| 检疫类目分布 | `pig_quarantine_record` | quarantine_type、quantity、qualified_quantity |
| 检验检疫趋势 | `pig_quarantine_record` | quarantine_type、quarantine_date、quantity、qualified_quantity |
| 疫病数量及分布 | `pig_disease_treatment_record`、`pig_enterprise`、`sys_dept` | disease_name、enterprise_id、dept_id |
| 病死猪数量及分布 | `pig_dead_animal_record`、`pig_breeding_info`、`pig_enterprise`、`sys_dept` | dead_quantity、death_cause、stock_quantity、enterprise_id、dept_id |
| 病死猪处理情况分析 | `pig_dead_animal_record` | disposal_method、dead_quantity |
| 免疫防疫覆盖分析 | `pig_immunity_record`、`pig_breeding_info`、`pig_enterprise` | vaccine_name、immune_quantity、stock_quantity |
| 生猪保险统计 | `pig_insurance_monthly` | stat_month、insured_quantity、claimed_quantity、claimed_amount |

### B. 预警阈值系统参数配置

| config_key | config_name | config_type | config_value | 说明 |
|-----------|------------|------------|-------------|------|
| `quarantine.dead_rate_threshold` | 病死率预警阈值(%) | `Y` | `5` | 整数，1-100；乡镇病死率超过此值在热力图标注预警 |
| `quarantine.immunity_threshold` | 强制免疫覆盖率阈值(%) | `Y` | `90` | 整数，1-100；强制免疫病种覆盖率低于此值红色标注 |

### C. 与其他模块的定位区分

| 模块 | 定位 | 数据范围 | 实现阶段 |
|------|------|---------|---------|
| **检验检疫无害化分析（本模块）** | 动物卫生防疫和无害化处理领域的专项分析，关注检疫合格率、疫病分布、病死猪处理、免疫覆盖 | 省级平台检疫数据 + 本县已入库疫病/免疫/病死处理数据 + 保险数据 | 第4阶段 |
| 养殖分析 | 宏观产业分析视角，关注养殖态势、效率指标、排行榜、预警趋势 | 已入库养殖数据的聚合统计 | 第4阶段 |
| 市场贸易分析 | 外部市场价格行情分析，关注生猪及猪肉价格走势 | 外部市场价格数据 | 第4阶段 |
| 产业总览大屏 | 最高层展示视角，汇聚多个分析模块的核心指标到可视化大屏 | 全部分析数据的精选展示 | 第4阶段（后续） |

### D. 导出文件名汇总

| 导出类型 | 文件名格式 | 示例 |
|---------|----------|------|
| 整页分析报告 | `检验检疫无害化分析_{年度}_YYYYMMDD_HHmmss.xlsx` | `检验检疫无害化分析_2026_20260310_143000.xlsx` |
| 检疫数据导入模板 | `检疫数据导入模板.xlsx` | — |
| 保险数据导入模板 | `保险数据导入模板.xlsx` | — |

### E. 外部数据对接备忘

| 数据源 | 当前方案 | 未来方案 | 说明 |
|-------|---------|---------|------|
| 四川省智慧监督管理平台（检疫数据） | 管理员 Excel 手动导入 | API 接口或数据推送自动对接 | 具体对接方式待技术方案确认；`data_source` 字段用于区分来源 |
| 保险公司（生猪保险数据） | 管理员 Excel 导入 / 逐条手动录入 | 与保险公司 API 对接 | 待协调；`data_source` 字段用于区分来源 |
