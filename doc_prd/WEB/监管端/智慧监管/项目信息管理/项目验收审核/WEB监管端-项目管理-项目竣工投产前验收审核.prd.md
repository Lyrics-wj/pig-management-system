# WEB监管端 - 项目管理 - 项目竣工投产前验收审核

> **文档版本**：v1.1  
> **创建时间**：2026-03-06  
> **最后更新**：2026-03-06（v1.1：对齐企业端 PRD v2.0 内容——详情页补充审核流程进度轴展示逻辑与节点定义；实施信息照片展示区对齐企业端折叠卡片分组格式；验收补充材料照片展示区移除 `photo_stage` 字段对齐；功能四 PDF 内容补充县农业农村局审核意见区域；各功能详情页补充历史审核意见可查看多条退回记录说明；新增附录：验收申请字段对照表、验收状态与申报状态联动关系表）  
> **文档状态**：待评审  
> **关联企业端PRD**：`doc_prd/WEB/企业端/项目信息管理/竣工验收申请/WEB企业端-项目管理-项目竣工投产前验收申请.prd.md`（v2.0）  
> **前置依赖**：`doc_prd/WEB/企业端/项目信息管理/项目申报/WEB企业端-项目申报-畜牧项目建设申报.prd.md`（申报流程县审核通过后企业端方可发起验收申请，管理端方可审核）  
> **角色与权限基准**：`需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md`

---

> **⚠️ 多功能 PRD 说明**
>
> 本文档包含 **4 个独立功能**，按照 PRD 设计规范 v1.8「多功能 PRD 编写规范」完整编写 3.1～3.6 全部章节。
>
> | 功能编号 | 功能名称 | 所属角色（role_key） |
> |---------|---------|---------|
> | 功能一 | 乡镇畜牧兽医站验收核实 | 乡镇畜牧兽医站（`town_vet_station`） |
> | 功能二 | 乡镇政府验收审核 | 乡镇政府业务人员（`town_gov_staff`） |
> | 功能三 | 县农业农村局验收审核 | 县级业务人员（`county_staff`） |
> | 功能四 | 验收申请表PDF生成与下载 | 乡镇畜牧兽医站（`town_vet_station`） |

---

## 功能一：乡镇畜牧兽医站验收核实

### 1.1 基本信息

- **功能名称**：乡镇畜牧兽医站验收核实
- **关联现有模块/功能**：
  - 用户认证：复用现有 `/login`、`/getInfo`、`/getRouters` 接口与 JWT 机制
  - 权限管理：复用现有角色权限体系，新增"乡镇畜牧兽医站验收核实"权限标识 `project:acceptance-audit:station`
  - 站内消息：审核不通过时向企业端发送站内消息
  - 关联企业端PRD：`doc_prd/WEB/企业端/项目信息管理/竣工验收申请/WEB企业端-项目管理-项目竣工投产前验收申请.prd.md` 功能二、功能三
  - 参考文档：`doc_spec/现有业务说明.spec.md`
- **目标用户/场景**：

| 用户角色 | 角色编码（role_key） | 使用场景 | 主要需求 |
|---------|---------------------|---------|---------|
| 乡镇畜牧兽医站 | `town_vet_station` | 对辖区内养殖场提交的验收申请进行现场核实、填写核实意见 | 查看待核实验收申请列表、查看申请详情（含申报回显信息、实施信息照片、验收补充材料照片）、填写核实意见并通过或退回 |

> 角色编码引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1 角色总览。乡镇畜牧兽医站侧重技术层面的"核实"（见全局权限文档 §1.3）。

### 1.2 目标与范围

**目标**：
1. 提供待核实验收申请列表视图，展示辖区内所有处于待站核实阶段（`PENDING_STATION_REVIEW`）的验收申请记录。
2. 查看企业端提交的验收申请完整信息，包括申报基本信息回显、建设内容明细、实施信息照片/视频（只读引用）、验收补充材料照片。
3. 核实通过后，验收申请进入下一流程节点（乡镇政府审核）。
4. 核实不通过时必须填写退回原因，系统自动推送站内消息通知企业端用户。

**非目标（Non-Goals）**：
- ❌ 不支持批量审核（每条验收申请须独立核实）。
- ❌ 不支持直接修改企业端填报内容或上传的照片。
- ❌ 不提供已核实记录的撤销功能。

### 1.3 现状与复用

**现状简述**：
- 现有管理端无乡镇畜牧兽医站验收核实功能，需新增页面与前端路由。
- 验收申请数据来源于企业端提交的 `project_acceptance` 表，本功能通过 `acceptance_status` 字段过滤展示待核实记录。
- 实施信息照片数据来源于 `project_implementation_record` + `project_implementation_media` 表（申报PRD功能四），管理端以只读方式引用展示。

**复用检查结论**：

**1. 接口复用检查（对照 `doc_spec/现有业务说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户认证（登录） | POST `/login` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取用户信息 | GET `/getInfo` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取路由菜单 | GET `/getRouters` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 验收申请主记录表 | `project_acceptance`（企业端PRD功能一/二新建） | ✅ 直接复用 | 本功能更新 `acceptance_status` 字段 |
| 验收补充材料照片表 | `project_acceptance_photo`（企业端PRD功能二新建） | ✅ 直接复用 | 本功能只读展示照片 |
| 验收审核日志表 | `project_acceptance_audit_log`（企业端PRD功能三新建） | ✅ 直接复用 | 本功能写入站核实审核记录 |
| 申报详情接口 | GET `/project/application/{id}`（申报PRD功能二新增） | ✅ 直接复用 | 获取关联申报信息用于详情页展示 |
| 验收申请详情接口 | GET `/project/acceptance/{id}`（企业端PRD功能二新增） | ✅ 直接复用 | 获取验收申请完整信息 |
| 实施信息总览接口 | GET `/project/implementation/{applicationId}/overview`（申报PRD功能四新增） | ✅ 直接复用 | 获取建设内容明细及实施信息填报状态 |
| 实施记录详情接口 | GET `/project/implementation/record`（申报PRD功能四新增） | ✅ 直接复用 | 获取每项建设内容每个阶段的照片/视频详情 |
| 站内消息发送接口 | `sys_notice` 通知公告模块 | 🔧 改造复用 | 见现有业务说明.spec.md §3.9 通知公告，审核退回时复用通知公告模块推送消息，需扩展消息类型与跳转链接字段（与项目申报审核管理端PRD功能二共用改造点） |
| 站核实验收申请列表接口 | 无 | 🆕 新增 `/project/acceptance/station/list` | 无现有接口满足需求 |
| 站核实操作接口 | 无 | 🆕 新增 `/project/acceptance/station-audit/{id}` | 无现有接口满足需求 |

**2. 数据库表复用检查（对照 `doc_spec/现有数据库说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户表 | `sys_user` | ✅ 直接复用 | 获取当前登录用户信息，见现有数据库说明.spec.md 系统表部分 |
| 部门表 | `sys_dept` | ✅ 直接复用 | 按乡镇部门树过滤辖区数据，见现有数据库说明.spec.md 系统表部分 |
| 验收申请主记录表 | `project_acceptance`（企业端PRD功能一新建） | ✅ 直接复用 | 本功能更新 `acceptance_status` 字段 |
| 验收补充材料照片表 | `project_acceptance_photo`（企业端PRD功能二新建） | ✅ 直接复用 | 本功能只读展示 |
| 验收审核日志表 | `project_acceptance_audit_log`（企业端PRD功能三新建） | ✅ 直接复用 | 本功能写入 `audit_node=STATION_AUDIT` 审核记录 |
| 申报主记录表 | `project_application`（申报PRD功能二新建） | ✅ 直接复用 | 通过 `application_id` 获取关联申报信息 |
| 建设内容明细表 | `project_application_item`（申报PRD功能二新建） | ✅ 直接复用 | 获取建设内容明细 |
| 资质附件表 | `project_application_attachment`（申报PRD功能二新建） | ✅ 直接复用 | 详情页展示资质附件 |
| 实施记录表 | `project_implementation_record`（申报PRD功能四新建） | ✅ 直接复用 | 获取实施过程记录 |
| 实施媒体资源表 | `project_implementation_media`（申报PRD功能四新建） | ✅ 直接复用 | 获取实施过程照片/视频 |
| 通知公告表 | `sys_notice` | 🔧 扩展复用 | 需新增消息类型、跳转链接字段（与项目申报审核管理端PRD功能二共用改造点） |

### 1.4 功能需求

**功能列表**：

| 功能点 | 优先级 | 说明 |
|-------|-------|------|
| 待核实验收申请列表 | P0 | 展示辖区内处于 `PENDING_STATION_REVIEW` 状态的验收申请记录 |
| 状态Tab筛选 | P1 | 待核实 / 核实通过 / 核实退回，切换立即筛选 |
| 验收申请详情查看 | P0 | 查看企业端提交的验收申请完整内容（含申报回显信息、实施信息照片/视频、验收补充材料照片），均只读 |
| 核实通过 | P0 | `PENDING_STATION_REVIEW` 状态下，填写核实意见（选填）并点击"核实通过"，状态变为 `STATION_APPROVED`，自动流转至乡镇政府审核（`PENDING_TOWN_REVIEW`） |
| 核实退回 | P0 | `PENDING_STATION_REVIEW` 状态下，填写退回原因（必填）并点击"核实退回"，状态变为 `STATION_REJECTED`，系统推送消息给企业端 |

**关键流程**：

```
验收申请列表（过滤：PENDING_STATION_REVIEW）
        ↓
点击验收申请记录 → 查看验收申请详情（只读）
含：申报回显信息 + 建设内容明细 + 实施信息照片/视频 + 验收补充材料照片
        ↓
仅 PENDING_STATION_REVIEW 状态展示核实操作区：
        ↓
      ┌────────────────┐
      ↓                ↓
  核实通过           核实退回
  填写意见（选填）   填写退回原因（必填）
      ↓                ↓
  状态：STATION_APPROVED  状态：STATION_REJECTED
  系统自动流转至          推送消息给企业端用户
  PENDING_TOWN_REVIEW
```

#### 1.4.1 功能流程逻辑

**审核规则**：
- 仅 `PENDING_STATION_REVIEW` 状态下可执行核实操作。
- 核实退回时，退回原因为必填字段，不填写则无法提交退回操作，提示"请填写退回原因"。
- 核实通过时，核实意见为选填。
- 核实操作执行后，在 `project_acceptance_audit_log` 表写入审核记录（`audit_node=STATION_AUDIT`）。

**状态自动流转**：
- 站核实通过后，系统在同一事务内将 `acceptance_status` 依次经 `STATION_APPROVED` → `PENDING_TOWN_REVIEW` 完成流转（或直接置为 `PENDING_TOWN_REVIEW`，审核日志记录 `STATION_AUDIT` + `APPROVED` 意见）。
- 建议实现：写入 `STATION_AUDIT` 的核实通过日志后，立即将主记录状态置为 `PENDING_TOWN_REVIEW`，保证原子性。

**退回后企业端重新提交规则**（与企业端 PRD §2.4.1 一致）：
- 站核实退回（`STATION_REJECTED`）后，企业端用户可修改验收补充材料照片和申报单位意见并重新提交。
- 重新提交后，验收申请状态重置为 `PENDING_STATION_REVIEW`（待站核实），重新进入审核流程。
- 管理端审核列表中该验收申请记录将重新出现在"待核实"Tab 下，此前的审核日志保留（可在详情页审核流程区查看历史退回记录）。

#### 1.4.2 数据模型与字段定义

本功能无新建表，所有数据均写入 `project_acceptance`（更新 `acceptance_status` 字段）和 `project_acceptance_audit_log`（写入审核记录）。

复用字段详见：
- `project_acceptance` 表：见企业端PRD 1.4.2
- `project_acceptance_audit_log` 表：见企业端PRD 3.4.2

#### 1.4.3 界面交互逻辑

**待核实验收申请列表页**：
- 页面标题：项目验收审核（乡镇畜牧兽医站）。
- 状态 Tab：待核实 / 核实通过 / 核实退回，默认激活"待核实"Tab。
- **Tab 切换为即时响应**：点击 Tab 标签后立即筛选并刷新下方列表内容，无需额外点击「查询」按钮。切换过程中展示骨架屏加载效果，防止内容空白闪烁。当前激活标签以主色底色+白色文字标识选中态，非激活标签为浅灰底色。
- 表格列：序号、申请单位名称、关联项目名称（来自关联申报的 `project_application` 中的申报关联通知的 `project_notice.project_name`）、验收申请时间（格式：`YYYY-MM-DD`）、场址、当前状态标签（待核实-蓝色、核实通过-绿色、核实退回-红色）、操作列。
- 分页，每页展示 10 条，按创建时间倒序。
- 操作列按钮：
  - 待核实状态：「核实」蓝色文字按钮（点击跳转详情页并定位到审核操作区）、「查看」灰色文字按钮
  - 已核实状态：「查看」灰色文字按钮

**验收申请详情页（只读，含操作区）**：
- 面包屑：项目管理 > 验收审核 > 验收申请详情。
- 详情页内容分区如下（所有内容均为只读展示）：

  **详情页完整字段清单**（与企业端 PRD 功能二 §2.4.3 页面布局对齐，功能三 §3.4.1 流程进度轴对齐）：

  1. **审核流程进度轴**（横向步骤条，4个节点，与企业端 PRD 功能三 §3.4.1 流程进度轴定义完全一致）：
     - 每个节点显示名称、状态图标（待处理灰圆圈、进行中蓝色、已通过绿色勾、已退回红色叉）、完成时间（已完成/已退回节点显示对应 `project_acceptance_audit_log` 记录的 `audit_time`）
     - 审核退回时，对应节点下方展示退回意见文字（红色字体，来自对应审核日志的 `audit_opinion`）
     - 节点定义：

     | 步骤序号 | 节点名称 | 对应状态区间 | 展示逻辑 |
     |---------|---------|------------|---------|
     | 步骤1 | 填报提交 | 任意非 `DRAFT` 状态均为已完成 | 完成态（绿色勾） |
     | 步骤2 | 乡镇畜牧兽医站核实 | `PENDING_STATION_REVIEW` 为进行中（蓝色）；`STATION_APPROVED` 为已完成（绿色勾）；`STATION_REJECTED` 为退回（红色叉） | — |
     | 步骤3 | 乡镇政府审核 | `PENDING_TOWN_REVIEW` 为进行中（蓝色）；`TOWN_APPROVED` 为已完成（绿色勾）；`TOWN_REJECTED` 为退回（红色叉）；`STATION_APPROVED` 之前为未开始（灰色） | — |
     | 步骤4 | 县农业农村局审核 | `PENDING_COUNTY_REVIEW` 为进行中（蓝色）；`ACCEPTANCE_APPROVED` 为已完成（绿色勾）；`COUNTY_REJECTED` 为退回（红色叉）；`TOWN_APPROVED` 之前为未开始（灰色） | — |

     > 若存在多次退回-重提历史（如站核实退回→企业重提→站核实通过→镇审核退回→企业重提……），进度轴仅展示最新一轮的节点状态。历史审核日志可在下方"历史审核意见区"查看全部记录。

  2. **申报信息回显区**（灰色背景只读）：申请单位名称（`company_name`）、申请时间（`apply_date`）、场址-镇（`address_town`）、场址-村（`address_village`）、场址-社（`address_group`）、养殖场类型（`farm_type`，展示中文名称）、联系人（`contact_name`）、联系电话（`contact_phone`）、建设性质（`construction_type`，展示中文名称）、建设期限（`build_start_date` 至 `build_end_date`）、设计存栏规模（`design_stock`）、设计年出栏规模（`design_annual_output`）、建成后新增存栏（`new_stock_after_build`）、建成后新增出栏（`new_output_after_build`）、建成后饲养量（`total_raise_after_build`）、现存栏（`current_stock`）、养殖场资质附件（按6类附件类型分组展示文件名列表，点击可在线预览（PDF/图片）或下载（DOC/DOCX））
  3. **建设内容明细区**：明细表格展示所有行（子项目、建设内容、单位、数量、单价、金额、型号、备注），底部展示合计投资金额（`total_investment`）
  4. **实施信息照片展示区**（折叠卡片，默认折叠，与企业端 PRD 功能二 §2.4.3 实施信息照片展示区格式完全对齐）：
     - 区域顶部展示提示文字（蓝色信息框）："以下为项目实施过程中企业端已填报的照片和视频，仅供审核参考。"
     - 按建设内容明细分组展示，每组为一个折叠卡片：
       - 卡片标题行：`[子项目名称] - [建设内容] [数量][单位]`
       - 展开后按三个阶段（前期/中期/后期）分区展示：
         - 照片：缩略图网格展示（120×120px），点击可放大预览（Lightbox 模式）
         - 视频：封面帧缩略图 + 视频时长 + 文件名，点击可在线播放
         - 文字说明：只读文本展示
       - 某阶段未填报时展示灰色占位文字"该阶段未填报实施信息"
     - 默认展开第一项，其余折叠
  5. **验收补充材料照片区**（折叠卡片，默认折叠）：按子项目类别分组展示补充材料照片缩略图（与企业端 PRD 功能二 §2.4.1 验收补充材料照片上传分类规则对齐：每个类别下一个展示区，不区分建设前/中/后阶段），点击放大预览
  6. **申报单位意见区**：展示申报单位意见文字（`applicant_opinion`），只读
  7. **历史审核意见区**（仅当存在审核日志时展示）：按时间倒序展示所有审核记录（来自 `project_acceptance_audit_log`），每条记录展示：审核节点名称、审核结果（通过/退回）、审核意见、审核人姓名、审核时间。若存在多轮退回-重提历史，全部记录均展示，审核人员可查看完整审核历程

- **底部审核操作区**（仅 `PENDING_STATION_REVIEW` 状态显示）：
  - 展示"核实意见"文本域（选填，最大500字符）
  - 展示"核实退回"按钮（次要按钮，红色边框）和"核实通过"按钮（主色按钮）
  - 点击"核实退回"时，若核实意见为空则弹窗要求填写退回原因，确认退回前弹窗提示"核实退回后将通知企业端用户修改重提，是否确认？"
  - 点击"核实通过"前弹窗确认"确认核实通过？核实通过后将进入乡镇政府审核流程。"

**校验规则**：

> **校验时机**：提交核实操作前完成前端校验。
>
> **校验提示位置**：错误提示紧邻核实意见文本域下方展示（红色文字）。

| 校验字段 | 校验类型 | 校验规则 | 错误提示文案 |
|---------|---------|---------|------------|
| 核实意见（通过时） | 长度 | 选填，最大500字符 | 「核实意见不能超过500个字符」 |
| 核实意见（退回时） | 必填 + 长度 | 不能为空，最大500字符 | 「请填写退回原因」/「退回原因不能超过500个字符」 |

#### 1.4.4 接口路径定义

| 接口名称 | HTTP 方法 | 路径 | 请求参数摘要 | 响应数据摘要 | 备注 |
|----------|-----------|------|-------------|-------------|------|
| 分页查询验收申请列表（站核实） | GET | `/project/acceptance/station/list` | pageNum、pageSize、statusTab（PENDING/APPROVED/REJECTED） | 分页列表+总数 | 🆕 新增接口，按辖区（当前登录用户所属乡镇）过滤 |
| 查询验收申请详情 | GET | `/project/acceptance/{id}` | — | 验收完整信息+补充材料照片列表+审核日志 | ✅ 复用企业端PRD功能二已定义接口 |
| 查询关联申报详情 | GET | `/project/application/{id}` | — | 申报完整信息+明细行+附件列表 | ✅ 复用申报PRD功能二已定义接口 |
| 查询实施信息总览 | GET | `/project/implementation/{applicationId}/overview` | — | 建设内容明细列表+每项的三个阶段填报状态 | ✅ 复用申报PRD功能四已定义接口 |
| 查询单项阶段实施详情 | GET | `/project/implementation/record` | itemId、stage | 实施记录详情（文字说明+照片列表+视频列表） | ✅ 复用申报PRD功能四已定义接口 |
| 站核实（通过/退回） | PUT | `/project/acceptance/station-audit/{id}` | auditResult（`APPROVED`通过/`REJECTED`退回，仅此2种取值）、auditOpinion（退回时必填，通过时选填，最大500字符） | 操作结果 | 🆕 新增接口，状态流转+写入审核日志+退回时推送消息 |

### 1.5 非功能需求

- **安全与权限**：需具备 `project:acceptance-audit:station` 权限标识的用户方可操作，仅绑定乡镇畜牧兽医站（`town_vet_station`）角色。数据权限按 `data_scope = '4'`（本部门及以下）控制，只能查看和核实本乡镇辖区内企业的验收申请。角色定义引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1，数据范围见全局权限文档 §2.1。
- **兼容性**：桌面端主流分辨率（1280px 及以上）正常显示。

### 1.6 验收标准

- [ ] 乡镇畜牧兽医站（`town_vet_station`）角色用户登录后，"验收审核"菜单下只能看到本乡镇辖区内的验收申请记录。
- [ ] 状态Tab切换即时筛选，待核实/核实通过/核实退回分组正确。
- [ ] 验收申请详情页展示审核流程进度轴（横向步骤条，4个节点：填报提交/乡镇畜牧兽医站核实/乡镇政府审核/县农业农村局审核），当前节点以蓝色进行中态标识。
- [ ] 详情页完整展示：申报回显信息（只读）、建设内容明细表（只读）、实施信息照片/视频（只读引用，按建设内容分项+阶段展示，含折叠卡片标题行、阶段分区、未填报阶段占位文字）、验收补充材料照片（只读，按子项目类别分组展示）、申报单位意见（只读）。
- [ ] 详情页历史审核意见区按时间倒序展示全部审核记录（若存在多轮退回-重提历史，全部记录均展示）。
- [ ] 实施信息照片支持点击缩略图放大预览（Lightbox 模式），视频支持点击在线播放。
- [ ] 待核实状态下可执行核实操作；核实通过后状态变为 `STATION_APPROVED` 并自动流转至 `PENDING_TOWN_REVIEW`（同一事务）；核实退回时退回原因必填，退回后状态变为 `STATION_REJECTED`，企业端收到站内消息通知（含退回原因和跳转链接）。
- [ ] 见可交互 HTML 原型。

---

## 功能二：乡镇政府验收审核

### 2.1 基本信息

- **功能名称**：乡镇政府验收审核
- **关联现有模块/功能**：
  - 用户认证：复用现有 `/login`、`/getInfo`、`/getRouters` 接口与 JWT 机制
  - 权限管理：复用现有角色权限体系，新增"乡镇政府验收审核"权限标识 `project:acceptance-audit:town`
  - 站内消息：审核不通过时向企业端发送站内消息
  - 关联企业端PRD：`doc_prd/WEB/企业端/项目信息管理/竣工验收申请/WEB企业端-项目管理-项目竣工投产前验收申请.prd.md`
  - 参考文档：`doc_spec/现有业务说明.spec.md`
- **目标用户/场景**：

| 用户角色 | 角色编码（role_key） | 使用场景 | 主要需求 |
|---------|---------------------|---------|---------|
| 乡镇政府业务人员 | `town_gov_staff` | 对乡镇畜牧兽医站核实通过后的验收申请进行行政审核 | 查看待审验收申请列表、查看验收详情（含站核实意见）、填写审核意见并通过或退回 |

> 角色编码引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1 角色总览。乡镇政府业务人员侧重行政层面的"审核"（见全局权限文档 §1.3）。

### 2.2 目标与范围

**目标**：
1. 提供待乡镇政府审核的验收申请列表（状态为 `PENDING_TOWN_REVIEW`）。
2. 查看验收申请详情及乡镇畜牧兽医站的核实意见。
3. 审核通过后验收申请进入县农业农村局审核流程；审核退回时向企业端推送站内消息通知。

**非目标（Non-Goals）**：
- ❌ 不支持批量审核。
- ❌ 不提供直接修改验收申请内容的功能。
- ❌ 不提供已审核记录的撤销功能。

### 2.3 现状与复用

**现状简述**：
- 现有管理端无乡镇政府验收审核功能，需新增页面与前端路由。
- 验收申请数据来源于 `project_acceptance` 表，乡镇畜牧兽医站核实通过后系统自动将状态从 `STATION_APPROVED` 置为 `PENDING_TOWN_REVIEW`。

**复用检查结论**：

**1. 接口复用检查（对照 `doc_spec/现有业务说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户认证（登录） | POST `/login` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取用户信息 | GET `/getInfo` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取路由菜单 | GET `/getRouters` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 验收申请主记录表 | `project_acceptance`（企业端PRD功能一新建） | ✅ 直接复用 | 本功能更新 `acceptance_status` 字段 |
| 验收审核日志表 | `project_acceptance_audit_log`（企业端PRD功能三新建） | ✅ 直接复用 | 本功能写入镇审核节点记录（`audit_node=TOWN_AUDIT`） |
| 验收申请详情接口 | GET `/project/acceptance/{id}`（企业端PRD功能二新增） | ✅ 直接复用 | 获取验收完整信息+审核日志 |
| 申报详情接口 | GET `/project/application/{id}`（申报PRD功能二新增） | ✅ 直接复用 | 获取关联申报信息 |
| 实施信息总览接口 | GET `/project/implementation/{applicationId}/overview`（申报PRD功能四新增） | ✅ 直接复用 | 获取实施信息填报状态 |
| 实施记录详情接口 | GET `/project/implementation/record`（申报PRD功能四新增） | ✅ 直接复用 | 获取实施过程照片/视频详情 |
| 镇政府验收审核接口 | 无 | 🆕 新增 `/project/acceptance/town-audit/{id}` | 无现有接口满足需求 |
| 镇政府验收审核列表接口 | 无 | 🆕 新增 `/project/acceptance/town/list` | 无现有接口满足需求 |

**2. 数据库表复用检查（对照 `doc_spec/现有数据库说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户表 | `sys_user` | ✅ 直接复用 | 获取当前登录用户信息，见现有数据库说明.spec.md 系统表部分 |
| 部门表 | `sys_dept` | ✅ 直接复用 | 按乡镇部门树过滤辖区数据 |
| 验收申请主记录表 | `project_acceptance`（企业端PRD功能一新建） | ✅ 直接复用 | 更新 `acceptance_status` |
| 验收审核日志表 | `project_acceptance_audit_log`（企业端PRD功能三新建） | ✅ 直接复用 | 写入 `audit_node=TOWN_AUDIT` 记录 |
| 申报主记录表 | `project_application`（申报PRD功能二新建） | ✅ 直接复用 | 获取关联申报信息 |

### 2.4 功能需求

**功能列表**：

| 功能点 | 优先级 | 说明 |
|-------|-------|------|
| 待审验收申请列表 | P0 | 展示辖区内 `PENDING_TOWN_REVIEW` 状态的验收申请记录 |
| 历史已审记录 | P1 | 状态Tab支持查看历史已通过（`TOWN_APPROVED`）和已退回（`TOWN_REJECTED`）记录 |
| 验收申请详情查看 | P0 | 查看完整验收申请内容（只读）及乡镇畜牧兽医站核实意见 |
| 审核通过 | P0 | 填写审核意见（选填）并通过，状态变为 `TOWN_APPROVED`，自动进入县审核流程（状态置为 `PENDING_COUNTY_REVIEW`） |
| 审核退回 | P0 | 填写退回原因（必填）并退回，状态变为 `TOWN_REJECTED`，推送站内消息给企业端 |

**关键流程**：

```
待审验收申请列表（PENDING_TOWN_REVIEW）
        ↓
点击验收申请记录 → 查看验收详情（只读）
含：完整验收内容 + 实施信息照片 + 补充材料照片 + 站核实意见
        ↓
      ┌────────────────┐
      ↓                ↓
  审核通过           审核退回
  填写意见（选填）   填写退回原因（必填）
      ↓                ↓
  状态：TOWN_APPROVED  状态：TOWN_REJECTED
  系统自动置为       推送消息给企业端
  PENDING_COUNTY_REVIEW
```

#### 2.4.1 功能流程逻辑

**状态自动流转**：
- 乡镇政府审核通过后，系统在同一事务内将 `acceptance_status` 依次经 `TOWN_APPROVED` → `PENDING_COUNTY_REVIEW` 完成流转（或直接置为 `PENDING_COUNTY_REVIEW`，审核日志记录 `TOWN_AUDIT` + `APPROVED` 意见）。
- 建议实现：写入 `TOWN_AUDIT` 的审核通过日志后，立即将主记录状态置为 `PENDING_COUNTY_REVIEW`，保证原子性。

**审核规则**：
- 仅 `PENDING_TOWN_REVIEW` 状态下可执行审核操作。
- 审核退回时，退回原因必填，不填则无法提交，提示"请填写退回原因"。
- 审核通过时，审核意见选填，最大长度500字符。
- 审核操作后在 `project_acceptance_audit_log` 写入 `audit_node=TOWN_AUDIT` 记录。

**退回后企业端重新提交规则**（与企业端 PRD §2.4.1 一致）：
- 镇政府审核退回（`TOWN_REJECTED`）后，企业端用户可修改验收补充材料照片和申报单位意见并重新提交。
- 重新提交后，验收申请状态重置为 `PENDING_STATION_REVIEW`（待站核实），重新进入完整审核流程（从乡镇畜牧兽医站核实开始）。
- 管理端各级审核列表中该验收申请记录将按新状态重新出现，此前的审核日志保留。

#### 2.4.2 数据模型与字段定义

本功能无新建表，所有数据均写入 `project_acceptance`（更新 `acceptance_status` 字段）和 `project_acceptance_audit_log`（写入审核记录）。

复用字段详见：
- `project_acceptance` 表：见企业端PRD 1.4.2
- `project_acceptance_audit_log` 表：见企业端PRD 3.4.2

#### 2.4.3 界面交互逻辑

**待审验收申请列表页**：
- 页面标题：项目验收审核（乡镇政府）。
- 状态 Tab：待审核 / 已通过 / 已退回，默认激活"待审核"Tab。
- **Tab 切换为即时响应**：点击 Tab 标签后立即筛选并刷新下方列表内容，无需额外点击「查询」按钮。切换过程中展示骨架屏加载效果。当前激活标签以主色底色+白色文字标识选中态，非激活标签为浅灰底色。
- 表格列：序号、申请单位名称、关联项目名称（来源同功能一列表）、验收申请时间（格式：`YYYY-MM-DD`）、场址、站核实时间（格式：`YYYY-MM-DD`，来自 `project_acceptance_audit_log` 中 `audit_node=STATION_AUDIT` 且 `audit_result=APPROVED` 的最新记录的 `audit_time`）、当前状态标签（待审核-蓝色、已通过-绿色、已退回-红色）、操作列。
- 分页，每页展示 10 条，按创建时间倒序。
- 操作列：待审核状态展示「审核」蓝色文字按钮（点击跳转详情页并定位到审核操作区）+ 「查看」灰色文字按钮；已通过/已退回状态展示「查看」灰色文字按钮。

**验收申请详情页（只读，含操作区）**：
- 面包屑：项目管理 > 验收审核 > 验收申请详情。
- 详情内容分区与功能一详情页完全一致（见功能一 §1.4.3「详情页完整字段清单」，须与企业端 PRD 2.4.3 页面布局全量对齐），均为只读展示，包含：
  1. **审核流程进度轴**（横向步骤条，4个节点，节点定义及展示逻辑与功能一 §1.4.3 完全一致）
  2. **申报信息回显区**（灰色背景只读，字段清单同功能一 §1.4.3）
  3. **建设内容明细区**（只读表格，同功能一 §1.4.3）
  4. **实施信息照片展示区**（折叠卡片，格式同功能一 §1.4.3）
  5. **验收补充材料照片区**（折叠卡片，按子项目类别分组，格式同功能一 §1.4.3）
  6. **申报单位意见区**（只读，同功能一 §1.4.3）
  7. **历史审核意见区**：按时间倒序展示所有审核记录（同功能一 §1.4.3）。在 `PENDING_TOWN_REVIEW` 状态下，至少包含乡镇畜牧兽医站的核实意见（`audit_node=STATION_AUDIT` 记录的 `audit_opinion`）及核实时间。若存在多轮退回-重提历史，全部历史记录均展示
- 底部审核操作区（仅 `PENDING_TOWN_REVIEW` 状态显示）：
  - 展示"审核意见"文本域（选填，最大500字符）
  - 展示"审核退回"（次要按钮，红色边框）和"审核通过"（主色按钮）
  - 点击"审核退回"时，若审核意见为空则要求填写退回原因；确认前弹窗提示"审核退回后将通知企业端用户修改重提，是否确认？"
  - 点击"审核通过"前弹窗确认"确认审核通过？审核通过后将进入县农业农村局审核流程。"

**校验规则**：

> **校验时机**：提交审核操作前完成前端校验。
>
> **校验提示位置**：错误提示紧邻审核意见文本域下方展示（红色文字）。

| 校验字段 | 校验类型 | 校验规则 | 错误提示文案 |
|---------|---------|---------|------------|
| 审核意见（通过时） | 长度 | 选填，最大500字符 | 「审核意见不能超过500个字符」 |
| 审核意见（退回时） | 必填 + 长度 | 不能为空，最大500字符 | 「请填写退回原因」/「退回原因不能超过500个字符」 |

#### 2.4.4 接口路径定义

| 接口名称 | HTTP 方法 | 路径 | 请求参数摘要 | 响应数据摘要 | 备注 |
|----------|-----------|------|-------------|-------------|------|
| 分页查询验收申请列表（镇政府审核） | GET | `/project/acceptance/town/list` | pageNum、pageSize、statusTab（PENDING/APPROVED/REJECTED） | 分页列表+总数 | 🆕 新增接口，按辖区（当前登录用户所属乡镇）过滤 |
| 查询验收申请详情 | GET | `/project/acceptance/{id}` | — | 验收完整信息+审核日志 | ✅ 复用企业端PRD功能二已定义接口 |
| 查询关联申报详情 | GET | `/project/application/{id}` | — | 申报完整信息+明细行+附件列表 | ✅ 复用申报PRD功能二已定义接口 |
| 查询实施信息总览 | GET | `/project/implementation/{applicationId}/overview` | — | 建设内容明细列表+每项的三个阶段填报状态 | ✅ 复用申报PRD功能四已定义接口 |
| 查询单项阶段实施详情 | GET | `/project/implementation/record` | itemId、stage | 实施记录详情（文字说明+照片列表+视频列表） | ✅ 复用申报PRD功能四已定义接口 |
| 镇政府验收审核（通过/退回） | PUT | `/project/acceptance/town-audit/{id}` | auditResult（`APPROVED`通过/`REJECTED`退回，仅此2种取值）、auditOpinion（退回时必填，通过时选填，最大500字符） | 操作结果 | 🆕 新增接口，状态流转+写入审核日志+退回时推送消息 |

### 2.5 非功能需求

- **安全与权限**：需具备 `project:acceptance-audit:town` 权限标识的用户方可操作，仅绑定乡镇政府业务人员（`town_gov_staff`）角色。数据权限按 `data_scope = '4'`（本部门及以下）控制，只能查看和审核本乡镇辖区内的验收申请。角色定义引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1，数据范围见全局权限文档 §2.1。
- **兼容性**：桌面端主流分辨率（1280px 及以上）正常显示。

### 2.6 验收标准

- [ ] 乡镇政府业务人员（`town_gov_staff`）角色用户登录后，只能看到本乡镇辖区内的待审验收申请列表。
- [ ] 状态Tab切换即时筛选，待审核/已通过/已退回分组正确。
- [ ] 验收申请详情页展示审核流程进度轴（4个节点，显示当前流程位置）和完整验收内容（含申报回显信息、实施信息照片/视频、补充材料照片、申报单位意见），均为只读。
- [ ] 详情页历史审核意见区展示乡镇畜牧兽医站核实意见及操作时间（只读），若存在多轮退回-重提历史，全部记录均展示。
- [ ] 审核通过后状态变为 `TOWN_APPROVED` 并自动流转至 `PENDING_COUNTY_REVIEW`（同一事务），进入县农业农村局审核流程。
- [ ] 审核退回时退回原因必填，退回后企业端收到站内消息通知，消息内含退回原因和跳转链接。退回后企业端重新提交回到 `PENDING_STATION_REVIEW` 状态，重新进入完整审核流程。
- [ ] 见可交互 HTML 原型。

---

## 功能三：县农业农村局验收审核

### 3.1 基本信息

- **功能名称**：县农业农村局验收审核
- **关联现有模块/功能**：
  - 用户认证：复用现有 `/login`、`/getInfo`、`/getRouters` 接口与 JWT 机制
  - 权限管理：复用现有角色权限体系，新增"县农业农村局验收审核"权限标识 `project:acceptance-audit:county`
  - 站内消息：审核不通过时向企业端发送站内消息
  - 关联企业端PRD：`doc_prd/WEB/企业端/项目信息管理/竣工验收申请/WEB企业端-项目管理-项目竣工投产前验收申请.prd.md`
  - 功能四：PDF生成与下载（验收通过后可触发PDF生成）
  - 参考文档：`doc_spec/现有业务说明.spec.md`
- **目标用户/场景**：

| 用户角色 | 角色编码（role_key） | 使用场景 | 主要需求 |
|---------|---------------------|---------|---------|
| 县级业务人员 | `county_staff` | 对乡镇政府审核通过后的验收申请进行最终审核 | 查看所有乡镇的待审验收申请、查看验收详情（含全流程审核意见）、进行最终审核，审核通过后验收完成并联动更新申报状态 |

> 角色编码引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1 角色总览。

### 3.2 目标与范围

**目标**：
1. 提供所有乡镇的待县审核验收申请列表（状态为 `PENDING_COUNTY_REVIEW`）。
2. 查看验收申请完整详情，含乡镇畜牧兽医站核实意见和乡镇政府审核意见。
3. 审核通过后验收申请状态变为 `ACCEPTANCE_APPROVED`（验收完成）；**系统自动将关联申报主记录 `project_application.application_status` 更新为 `ACCEPTED`**；审核退回时向企业端推送站内消息。
4. 审核通过后乡镇畜牧兽医站可生成PDF验收申请表（功能四），企业端也可下载。

**非目标（Non-Goals）**：
- ❌ 不支持批量审核。
- ❌ 不提供直接修改验收申请内容的功能。
- ❌ `ACCEPTANCE_APPROVED` 状态不可再被撤销或修改。

### 3.3 现状与复用

**现状简述**：
- 现有管理端无县农业农村局验收审核功能，需新增页面与前端路由。
- 本功能是验收流程的最终审核节点，审核通过后验收完成，关联申报状态联动更新。

**复用检查结论**：

**1. 接口复用检查（对照 `doc_spec/现有业务说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户认证（登录） | POST `/login` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取用户信息 | GET `/getInfo` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取路由菜单 | GET `/getRouters` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 验收申请主记录表 | `project_acceptance`（企业端PRD功能一新建） | ✅ 直接复用 | 本功能更新 `acceptance_status` 字段 |
| 验收审核日志表 | `project_acceptance_audit_log`（企业端PRD功能三新建） | ✅ 直接复用 | 本功能写入县审核节点记录（`audit_node=COUNTY_AUDIT`） |
| 申报主记录表 | `project_application`（申报PRD功能二新建） | ✅ 直接复用 | 验收通过后联动更新 `application_status` 为 `ACCEPTED` |
| 验收申请详情接口 | GET `/project/acceptance/{id}`（企业端PRD功能二新增） | ✅ 直接复用 | 获取验收完整信息+审核日志 |
| 申报详情接口 | GET `/project/application/{id}`（申报PRD功能二新增） | ✅ 直接复用 | 获取关联申报信息 |
| 实施信息总览接口 | GET `/project/implementation/{applicationId}/overview`（申报PRD功能四新增） | ✅ 直接复用 | 管理端以只读方式查看实施信息 |
| 实施记录详情接口 | GET `/project/implementation/record`（申报PRD功能四新增） | ✅ 直接复用 | 管理端以只读方式查看实施过程照片/视频 |
| 县局验收审核列表接口 | 无 | 🆕 新增 `/project/acceptance/county/list` | 无现有接口满足需求 |
| 县局验收审核操作接口 | 无 | 🆕 新增 `/project/acceptance/county-audit/{id}` | 无现有接口满足需求 |
| 验收数据导出接口 | 无 | 🆕 新增 `/project/acceptance/county/export` | P2功能，后续迭代 |

**2. 数据库表复用检查（对照 `doc_spec/现有数据库说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户表 | `sys_user` | ✅ 直接复用 | 获取当前登录用户信息 |
| 部门表 | `sys_dept` | ✅ 直接复用 | 乡镇下拉筛选数据来源 |
| 验收申请主记录表 | `project_acceptance`（企业端PRD功能一新建） | ✅ 直接复用 | 更新 `acceptance_status` |
| 验收审核日志表 | `project_acceptance_audit_log`（企业端PRD功能三新建） | ✅ 直接复用 | 写入 `audit_node=COUNTY_AUDIT` 记录 |
| 申报主记录表 | `project_application`（申报PRD功能二新建） | ✅ 直接复用 | 验收通过后联动更新 `application_status` 为 `ACCEPTED` |

### 3.4 功能需求

**功能列表**：

| 功能点 | 优先级 | 说明 |
|-------|-------|------|
| 待审验收申请列表 | P0 | 展示全县所有乡镇 `PENDING_COUNTY_REVIEW` 状态的验收申请记录 |
| 多维度筛选 | P1 | 支持按乡镇、申请时间范围、申请单位名称关键词筛选 |
| 历史已审记录 | P1 | 状态Tab支持查看历史已通过（`ACCEPTANCE_APPROVED`）和已退回（`COUNTY_REJECTED`）记录 |
| 验收申请详情查看 | P0 | 查看完整验收申请内容（只读）及全流程审核意见（站核实+镇政府审核意见） |
| 审核通过 | P0 | 填写审核意见（选填）并通过，状态变为 `ACCEPTANCE_APPROVED`，验收完成；**系统自动将关联申报 `application_status` 更新为 `ACCEPTED`** |
| 审核退回 | P0 | 填写退回原因（必填）并退回，状态变为 `COUNTY_REJECTED`，推送站内消息给企业端 |
| 验收数据汇总导出 | P2 | 导出当前筛选条件下的验收申请列表为Excel（含基本信息和汇总数据） |

**关键流程**：

```
待审验收申请列表（PENDING_COUNTY_REVIEW，来自全县所有乡镇）
        ↓
可按乡镇/时间/申请单位筛选
        ↓
点击验收申请记录 → 查看验收详情（只读）
含：完整验收内容 + 实施信息照片 + 补充材料照片 + 站核实意见 + 镇政府审核意见
        ↓
      ┌────────────────┐
      ↓                ↓
  审核通过           审核退回
  填写意见（选填）   填写退回原因（必填）
      ↓                ↓
  状态：ACCEPTANCE_APPROVED  状态：COUNTY_REJECTED
  验收流程完成         推送消息给企业端
  联动申报状态为ACCEPTED  企业修改重提→回到PENDING_STATION_REVIEW
  乡镇畜牧兽医站可生成PDF
  企业端可下载PDF
```

#### 3.4.1 功能流程逻辑

**审核规则**：
- 仅 `PENDING_COUNTY_REVIEW` 状态下可执行审核操作。
- 审核退回时，退回原因必填，最大500字符，不填则无法提交，提示"请填写退回原因"。
- 审核通过时，审核意见选填，最大500字符。
- 审核操作后在 `project_acceptance_audit_log` 写入 `audit_node=COUNTY_AUDIT` 记录。

**审核通过后触发**：
- 系统自动将 `acceptance_status` 置为 `ACCEPTANCE_APPROVED`。
- **系统自动将关联申报主记录 `project_application.application_status` 更新为 `ACCEPTED`**（在同一事务中完成，确保数据一致性）。
- 企业端验收申请详情页"下载验收申请书PDF"按钮变为可用状态（功能四提供PDF生成接口）。

**退回后企业端重新提交规则**（与企业端 PRD §2.4.1 一致）：
- 县局审核退回（`COUNTY_REJECTED`）后，企业端用户可修改验收补充材料照片和申报单位意见并重新提交。
- 重新提交后，验收申请状态重置为 `PENDING_STATION_REVIEW`（待站核实），重新进入完整审核流程（从乡镇畜牧兽医站核实开始）。
- 管理端各级审核列表中该验收申请记录将按新状态重新出现，此前的审核日志保留。

#### 3.4.2 数据模型与字段定义

本功能无新建表，所有数据均写入 `project_acceptance`（更新 `acceptance_status` 字段）和 `project_acceptance_audit_log`（写入审核记录），并联动更新 `project_application.application_status`。

复用字段详见：
- `project_acceptance` 表：见企业端PRD 1.4.2
- `project_acceptance_audit_log` 表：见企业端PRD 3.4.2
- `project_application` 表：见申报PRD 2.4.2（联动更新 `application_status` 字段）

#### 3.4.3 界面交互逻辑

**待审验收申请列表页**：
- 页面标题：项目验收审核（县农业农村局）。
- 顶部筛选栏：乡镇下拉选择（单选，含"全部"选项）、申请时间范围（日期范围选择器）、申请单位名称（关键词输入框）+ 查询按钮 + 重置按钮。
- 状态 Tab：待审核 / 已通过 / 已退回，默认激活"待审核"Tab。
- **Tab 切换为即时响应**：点击 Tab 标签后立即筛选并刷新下方列表内容。切换过程中展示骨架屏加载效果。当前激活标签以主色底色+白色文字标识选中态，非激活标签为浅灰底色。
- 表格列：序号、申请单位名称、所属乡镇、关联项目名称（来源同功能一列表）、验收申请时间（格式：`YYYY-MM-DD`）、场址、镇政府审核时间（格式：`YYYY-MM-DD`，来自 `project_acceptance_audit_log` 中 `audit_node=TOWN_AUDIT` 且 `audit_result=APPROVED` 的最新记录的 `audit_time`）、当前状态标签（待审核-蓝色、已通过-绿色、已退回-红色）、操作列。
- 分页，每页展示 10 条，按创建时间倒序。
- 操作列：待审核状态展示「审核」蓝色文字按钮（点击跳转详情页并定位到审核操作区）+ 「查看」灰色文字按钮；已通过/已退回状态展示「查看」灰色文字按钮。
- 表格右上方展示"导出"按钮（P2功能）。

**验收申请详情页（只读，含操作区）**：
- 面包屑：项目管理 > 验收审核 > 验收申请详情。
- 详情内容分区与功能一详情页完全一致（见功能一 §1.4.3「详情页完整字段清单」，须与企业端 PRD 2.4.3 页面布局全量对齐），均为只读展示，包含：
  1. **审核流程进度轴**（横向步骤条，4个节点，节点定义及展示逻辑与功能一 §1.4.3 完全一致）
  2. **申报信息回显区**（灰色背景只读，字段清单同功能一 §1.4.3）
  3. **建设内容明细区**（只读表格，同功能一 §1.4.3）
  4. **实施信息照片展示区**（折叠卡片，格式同功能一 §1.4.3）
  5. **验收补充材料照片区**（折叠卡片，按子项目类别分组，格式同功能一 §1.4.3）
  6. **申报单位意见区**（只读，同功能一 §1.4.3）
  7. **历史审核意见区**：按时间倒序展示所有审核记录（同功能一 §1.4.3）。在 `PENDING_COUNTY_REVIEW` 状态下，至少包含乡镇畜牧兽医站的核实意见（`audit_node=STATION_AUDIT`）和乡镇政府审核意见（`audit_node=TOWN_AUDIT`）及各自操作时间。若存在多轮退回-重提历史，全部历史记录均展示
- 底部审核操作区（仅 `PENDING_COUNTY_REVIEW` 状态显示）：
  - 展示"审核意见"文本域（选填，最大500字符）
  - 展示"审核退回"（次要按钮，红色边框）和"审核通过"（主色按钮）
  - 交互逻辑与功能一、功能二相同（弹窗确认）
  - 点击"审核退回"时，若审核意见为空则要求填写退回原因；确认前弹窗提示"审核退回后将通知企业端用户修改重提，是否确认？"
  - 点击"审核通过"前弹窗确认"确认审核通过？审核通过后验收流程完成，关联申报状态将更新为已验收。"

**筛选交互**：
- 乡镇下拉切换立即筛选，状态Tab切换也立即筛选。
- 申请时间范围和申请单位名称关键词需点击"查询"按钮触发（非即时）。
- 点击"重置"清空所有筛选条件并重新加载列表。

**校验规则**：

> **校验时机**：提交审核操作前完成前端校验。
>
> **校验提示位置**：错误提示紧邻审核意见文本域下方展示（红色文字）。

| 校验字段 | 校验类型 | 校验规则 | 错误提示文案 |
|---------|---------|---------|------------|
| 审核意见（通过时） | 长度 | 选填，最大500字符 | 「审核意见不能超过500个字符」 |
| 审核意见（退回时） | 必填 + 长度 | 不能为空，最大500字符 | 「请填写退回原因」/「退回原因不能超过500个字符」 |
| 乡镇筛选 | — | 选填，单选（含"全部"选项） | — |
| 申请时间范围 | 日期范围 | 开始日期 ≤ 结束日期 | 「开始日期不能晚于结束日期」 |
| 申请单位名称关键词 | 长度 | 最大100字符 | 「关键词不能超过100个字符」 |

#### 3.4.4 接口路径定义

| 接口名称 | HTTP 方法 | 路径 | 请求参数摘要 | 响应数据摘要 | 备注 |
|----------|-----------|------|-------------|-------------|------|
| 分页查询验收申请列表（县局审核） | GET | `/project/acceptance/county/list` | pageNum、pageSize、statusTab（PENDING/APPROVED/REJECTED）、townId（乡镇ID）、applyDateStart、applyDateEnd、companyName | 分页列表+总数 | 🆕 新增接口，县局可查全县验收申请 |
| 查询验收申请详情 | GET | `/project/acceptance/{id}` | — | 验收完整信息+审核日志 | ✅ 复用企业端PRD功能二已定义接口 |
| 查询关联申报详情 | GET | `/project/application/{id}` | — | 申报完整信息+明细行+附件列表 | ✅ 复用申报PRD功能二已定义接口 |
| 查询实施信息总览 | GET | `/project/implementation/{applicationId}/overview` | — | 建设内容明细列表+每项的三个阶段填报状态 | ✅ 复用申报PRD功能四已定义接口 |
| 查询单项阶段实施详情 | GET | `/project/implementation/record` | itemId、stage | 实施记录详情（文字说明+照片列表+视频列表） | ✅ 复用申报PRD功能四已定义接口 |
| 县局验收审核（通过/退回） | PUT | `/project/acceptance/county-audit/{id}` | auditResult（`APPROVED`通过/`REJECTED`退回，仅此2种取值）、auditOpinion（退回时必填，通过时选填，最大500字符） | 操作结果 | 🆕 新增接口，状态流转+写入审核日志+退回时推送消息+通过时联动更新申报状态为 `ACCEPTED` |
| 导出验收申请列表Excel | POST | `/project/acceptance/county/export` | 同列表查询参数 | Excel文件流 | 🆕 新增接口，P2功能，后续迭代 |

### 3.5 非功能需求

- **安全与权限**：需具备 `project:acceptance-audit:county` 权限标识的用户方可操作，仅绑定县级业务人员（`county_staff`）角色。数据权限按 `data_scope = '1'`（全部数据）控制，可查看全县所有乡镇的验收申请数据。角色定义引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1，数据范围见全局权限文档 §2.1。
- **兼容性**：桌面端主流分辨率（1280px 及以上）正常显示。

### 3.6 验收标准

- [ ] 县级业务人员（`county_staff`）角色用户登录后，能看到全县所有乡镇的待审验收申请列表。
- [ ] 支持按乡镇（下拉单选含"全部"）、申请时间范围、申请单位名称关键词筛选，状态Tab切换即时筛选。
- [ ] 验收申请详情页展示审核流程进度轴（4个节点，显示当前流程位置）和完整验收内容（含申报回显信息、实施信息照片/视频、补充材料照片、申报单位意见）。
- [ ] 详情页历史审核意见区展示乡镇畜牧兽医站核实意见和乡镇政府审核意见及各自操作时间（只读），若存在多轮退回-重提历史，全部记录均展示。
- [ ] 审核通过后状态变为 `ACCEPTANCE_APPROVED`，验收流程完成。关联申报主记录 `project_application.application_status` 自动联动更新为 `ACCEPTED`（同一事务）。
- [ ] 审核通过后，乡镇畜牧兽医站（`town_vet_station`）可在验收详情页生成验收申请书PDF（功能四），企业端用户可下载PDF。
- [ ] 审核退回时退回原因必填，退回后企业端收到站内消息通知，消息内含退回原因和跳转链接。退回后企业端重新提交回到 `PENDING_STATION_REVIEW` 状态，重新进入完整审核流程。
- [ ] 见可交互 HTML 原型。

---

## 功能四：验收申请表PDF生成与下载

### 4.1 基本信息

- **功能名称**：验收申请表PDF生成与下载
- **关联现有模块/功能**：
  - 用户认证：复用现有 `/login`、`/getInfo`、`/getRouters` 接口与 JWT 机制
  - 权限管理：需具备 `project:acceptance-pdf:generate` 权限标识，绑定乡镇畜牧兽医站（`town_vet_station`）角色
  - 功能三：县农业农村局审核通过后触发PDF可生成状态
  - 关联企业端PRD：`doc_prd/WEB/企业端/项目信息管理/竣工验收申请/WEB企业端-项目管理-项目竣工投产前验收申请.prd.md` 功能三中企业端用户也可下载（仅下载，不可生成）
  - 参考文档：`doc_spec/现有业务说明.spec.md`
- **目标用户/场景**：

| 用户角色 | 角色编码（role_key） | 使用场景 | 主要需求 |
|---------|---------------------|---------|---------|
| 乡镇畜牧兽医站 | `town_vet_station` | 县农业农村局验收审核通过后，将线上验收申请内容生成标准格式的PDF验收申请书 | 一键生成符合《新/改/扩建项目竣工投产前验收申请书》格式的PDF文件，供存档和上报使用 |
| 企业管理员 | `ent_admin` | 验收通过后下载PDF验收申请书留存 | 下载已生成的PDF验收申请书 |

> 角色编码引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1 角色总览。

### 4.2 目标与范围

**目标**：
1. 县农业农村局审核通过（`ACCEPTANCE_APPROVED`）后，乡镇畜牧兽医站可在管理端触发PDF生成。
2. 生成的PDF格式严格按照《新/改/扩建项目竣工投产前验收申请书》原始表格格式，包含所有回显字段、建设内容明细表、各级审核意见签字区域（由系统填入审核意见文字，签字栏留空供线下签字）。
3. 生成成功后，乡镇畜牧兽医站可下载PDF，企业端用户也可在验收详情页下载。
4. 同一验收申请的PDF可多次生成（重新生成将覆盖上一次生成的文件）。

**非目标（Non-Goals）**：
- ❌ 不在 PDF 中自动添加电子签名或公章图片（签字栏留空，线下签字盖章后使用）。
- ❌ 不支持批量生成多份验收申请的PDF（每次只能对单条验收申请生成）。
- ❌ 生成的PDF不支持在线编辑修改。
- ❌ PDF中不包含实施信息照片和验收补充材料照片（仅包含表格化文字信息）。

### 4.3 现状与复用

**现状简述**：
- 现有系统PDF生成能力可复用项目申报审核管理端PRD功能五的技术方案（iText 或 Apache PDFBox 或 HTML模板转PDF），本功能新增验收申请书的PDF模板。

**复用检查结论**：

**1. 接口复用检查（对照 `doc_spec/现有业务说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 用户认证（登录） | POST `/login` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取用户信息 | GET `/getInfo` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 获取路由菜单 | GET `/getRouters` | ✅ 直接复用 | 见现有业务说明.spec.md §二 认证与通用 |
| 验收申请主记录表 | `project_acceptance`（企业端PRD功能一新建） | ✅ 直接复用 | 读取验收申请数据 |
| 申报主记录表 | `project_application`（申报PRD功能二新建） | ✅ 直接复用 | 读取关联申报信息 |
| 建设内容明细表 | `project_application_item`（申报PRD功能二新建） | ✅ 直接复用 | 读取明细行数据生成表格 |
| 验收审核日志表 | `project_acceptance_audit_log`（企业端PRD功能三新建） | ✅ 直接复用 | 读取各级审核意见填入PDF |
| PDF生成接口 | 无 | 🆕 新增 `/project/acceptance/{id}/pdf/generate` | 无现有接口满足需求 |
| PDF下载接口 | `/project/acceptance/{id}/pdf`（企业端PRD功能三已定义） | ✅ 直接复用 | 管理端和企业端共用同一下载接口 |

**2. 数据库表复用检查（对照 `doc_spec/现有数据库说明.spec.md`）**

| 检查项 | 现有资源 | 复用结论 | 说明 |
|--------|---------|---------|------|
| 验收申请主记录表 | `project_acceptance`（企业端PRD功能一新建） | ✅ 直接复用 | 读取验收数据，更新 `pdf_file_path` 和 `pdf_generate_time` |
| 申报主记录表 | `project_application`（申报PRD功能二新建） | ✅ 直接复用 | 读取关联申报信息 |
| 建设内容明细表 | `project_application_item`（申报PRD功能二新建） | ✅ 直接复用 | 读取明细行 |
| 验收审核日志表 | `project_acceptance_audit_log`（企业端PRD功能三新建） | ✅ 直接复用 | 读取审核意见 |

### 4.4 功能需求

**功能列表**：

| 功能点 | 优先级 | 说明 |
|-------|-------|------|
| 生成PDF验收申请书 | P0 | 乡镇畜牧兽医站在审核列表的已通过记录或验收详情页点击"生成验收申请书PDF"按钮，系统生成PDF文件 |
| 下载PDF验收申请书 | P0 | 管理端和企业端均可下载已生成的PDF文件 |
| 查看PDF生成状态 | P1 | 验收详情页展示PDF生成状态（未生成/已生成），已生成时展示生成时间和下载按钮 |

**关键流程**：

```
验收申请状态为 ACCEPTANCE_APPROVED
        ↓
乡镇畜牧兽医站在验收详情页（或列表页）
点击"生成验收申请书PDF"
        ↓
系统读取验收申请数据（申报回显信息+建设内容明细+各级审核意见）
按照验收申请书模板格式生成PDF
        ↓
生成成功 → 可下载
  ↓
企业端用户也可在验收详情页点击"下载验收申请书PDF"
```

#### 4.4.1 功能流程逻辑

**PDF生成内容规范**（严格按照《新/改/扩建项目竣工投产前验收申请书》格式）：

生成的PDF必须包含以下内容，布局和字段顺序与原始纸质表格完全一致：

1. **标题**：`蓬溪县新/改/扩建项目竣工投产前验收申请书`（根据 `construction_type` 动态选择"新建"/"改建"/"扩建"）
2. **申请时间**：`{apply_date.year}年{apply_date.month}月{apply_date.day}日`
3. **表格第一部分（基本信息）**：
   - 申请单位（营业执照注册全名）：`company_name`（来自 `project_application.company_name`）
   - 场址：`{address_town}镇 {address_village}村 {address_group}社`（来自 `project_application`）
   - 养殖场类型：`farm_type` 的中文名称——`SELF_BREEDING`自繁自养场、`FATTENING`育肥场、`PIGLET_BREEDING`仔猪繁育场（来自 `project_application.farm_type`）
   - 联系人：`contact_name`（来自 `project_application`）
   - 联系电话：`contact_phone`（来自 `project_application`）
   - 建设性质：`construction_type` 的中文名称——`NEW`新建、`RENOVATION`改建、`EXPANSION`扩建（来自 `project_application`）
   - 建设期限：`{build_start_date}至{build_end_date}`（来自 `project_application`）
   - 设计存栏规模（头）：`design_stock`（来自 `project_application`）
   - 设计年出栏规模（头）：`design_annual_output`（来自 `project_application`）
   - 建成后新增存栏（头）：`new_stock_after_build`（来自 `project_application`）
   - 建成后新增出栏（头）：`new_output_after_build`（来自 `project_application`）
   - 建成后饲养量（头）：`total_raise_after_build`（来自 `project_application`）
   - 现存栏（头）：`current_stock`（来自 `project_application`）
   - 养殖场资质办理情况：列出已上传的6类附件名称（文字列举，不含附件文件本身）
4. **表格第二部分（申请建设内容）**：
   - 按顺序逐行填入所有建设内容明细：子项目、建设内容、单位、数量、单价（元）、金额（万元）、型号、备注
   - 最后一行为合计行：显示合计投资金额（万元）（来自 `project_application.total_investment`）
5. **申报单位意见区域**：
   - 内容：`{applicant_opinion}`（来自 `project_acceptance.applicant_opinion`，如为空则填入标准格式文字："本单位承诺以上填报内容真实、准确，如有虚假，愿承担相关法律责任。"）
   - 负责人签字栏（留空）、单位公章栏（留空）、日期：`{apply_date.year}年{apply_date.month}月{apply_date.day}日`
6. **畜牧兽医站意见区域**：
   - 内容：`{station_audit_opinion}`（来自 `project_acceptance_audit_log` 中 `audit_node=STATION_AUDIT` 且 `audit_result=APPROVED` 的最新记录的 `audit_opinion` 字段）
   - 负责人签字栏（留空）、单位公章栏（留空）、日期：`{station_audit_time}`（取对应日志记录的 `audit_time`）
7. **乡镇人民政府意见区域**：
   - 内容：`{town_audit_opinion}`（来自 `project_acceptance_audit_log` 中 `audit_node=TOWN_AUDIT` 且 `audit_result=APPROVED` 的最新记录的 `audit_opinion` 字段）
   - 签字栏（留空）、公章栏（留空）、日期：`{town_audit_time}`（取对应日志记录的 `audit_time`）
8. **县农业农村局意见区域**：
   - 内容：`{county_audit_opinion}`（来自 `project_acceptance_audit_log` 中 `audit_node=COUNTY_AUDIT` 且 `audit_result=APPROVED` 的最新记录的 `audit_opinion` 字段）
   - 签字栏（留空）、公章栏（留空）、日期：`{county_audit_time}`（取对应日志记录的 `audit_time`）

> **注意**：PDF 中必须完整包含申报单位、畜牧兽医站、乡镇人民政府、县农业农村局共4个意见区域，与企业端 PRD 附录「验收申请与申报表字段对照」中的字段对照保持一致。

**PDF文件命名规则**：`{company_name}-竣工投产前验收申请书-{generate_date}.pdf`，文件名中特殊字符（`/`、`\`、`:`、`*`、`?`、`"`、`<`、`>`、`|`）替换为下划线（`_`），总长度不超过200字符。

**生成状态说明**：
- `NOT_GENERATED`（未生成）：`ACCEPTANCE_APPROVED` 后，PDF尚未生成，`pdf_file_path` 为 NULL
- `GENERATED`（已生成）：PDF生成成功，存储路径记录在 `project_acceptance.pdf_file_path` 字段

#### 4.4.2 数据模型与字段定义

本功能使用企业端 PRD 功能一（1.4.2）中已定义的 `project_acceptance` 表 `pdf_file_path` 和 `pdf_generate_time` 两个字段，无新建表或新增字段。

复用字段详见：
- `project_acceptance.pdf_file_path`：见企业端PRD 1.4.2
- `project_acceptance.pdf_generate_time`：见企业端PRD 1.4.2

#### 4.4.3 界面交互逻辑

**管理端-验收详情页（已通过状态下）**：
- 在详情页底部或右上角操作区展示"生成验收申请书PDF"按钮（主色按钮，仅乡镇畜牧兽医站（`town_vet_station`）角色可见）。
- 点击"生成验收申请书PDF"后按钮变为加载状态（loading），生成完成后展示"下载验收申请书PDF"按钮（次要按钮）和生成时间文字提示（格式：`已于 YYYY-MM-DD HH:mm 生成`）。
- 若PDF已生成，直接展示"重新生成"按钮（警告色按钮，点击前弹窗确认"重新生成将覆盖之前的PDF文件，是否确认？"）和"下载验收申请书PDF"按钮。
- 点击"下载验收申请书PDF"按钮，浏览器触发文件下载，文件名见4.4.1命名规则。

**管理端-已通过验收列表页（可选）**：
- 已通过（`ACCEPTANCE_APPROVED`）状态记录的操作列可展示"生成PDF"快捷按钮（可选功能，优先在详情页实现）。

**校验规则**：
- "生成验收申请书PDF"按钮仅在 `acceptance_status = ACCEPTANCE_APPROVED` 时展示；其他状态下不展示该按钮。
- "下载验收申请书PDF"按钮仅在 `pdf_file_path` 不为 NULL 时展示。

#### 4.4.4 接口路径定义

| 接口名称 | HTTP 方法 | 路径 | 请求参数摘要 | 响应数据摘要 | 备注 |
|----------|-----------|------|-------------|-------------|------|
| 生成验收申请书PDF | POST | `/project/acceptance/{id}/pdf/generate` | — | 操作结果+pdf_file_path | 🆕 新增接口，仅 `ACCEPTANCE_APPROVED` 状态可调用；重新生成时覆盖旧文件 |
| 下载验收申请书PDF | GET | `/project/acceptance/{id}/pdf` | — | PDF文件流 | ✅ 复用企业端PRD功能三已定义接口，管理端和企业端共用 |

### 4.5 非功能需求

- **性能**：PDF生成须在 30 秒内完成（验收数据量较小，表格内容不超过3页A4）；超时须返回错误并提示"PDF生成超时，请稍后重试"。
- **安全与权限**：生成操作需具备 `project:acceptance-pdf:generate` 权限标识，仅绑定乡镇畜牧兽医站（`town_vet_station`）角色。下载操作企业端企业管理员（`ent_admin`，申报人本人）和管理端相关角色均可操作。角色定义引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1。
- **文件存储**：PDF文件存储在服务器文件系统或OSS，路径记录于 `project_acceptance.pdf_file_path`，30天内下载链接有效（若使用OSS签名URL）。

### 4.6 验收标准

- [ ] 验收审核通过（`ACCEPTANCE_APPROVED`）后，乡镇畜牧兽医站管理端验收详情页展示"生成验收申请书PDF"按钮。
- [ ] 点击"生成验收申请书PDF"，系统生成PDF，生成过程中按钮显示加载状态，完成后展示下载按钮和生成时间。
- [ ] 下载的PDF格式严格按照《新/改/扩建项目竣工投产前验收申请书》原始表格布局，包含全部回显字段、建设内容明细表（含合计行）、申报单位意见区域、畜牧兽医站核实意见区域（含核实意见文字）、乡镇人民政府审核意见区域（含审核意见文字）、县农业农村局审核意见区域（含审核意见文字）——共4个意见区域。
- [ ] PDF中签字栏和公章栏为空白，供线下手工签字盖章。
- [ ] 企业端验收详情页（`ACCEPTANCE_APPROVED` 状态且已生成PDF）展示"下载验收申请书PDF"按钮，点击可正常下载。
- [ ] 重新生成PDF前弹窗确认，确认后覆盖旧文件，下载的为最新生成的文件。
- [ ] 见可交互 HTML 原型。

---

## 附录

### 整体验收审核流程状态流转图

> 审批链路参照 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.4 典型审批链路：企业管理员（`ent_admin`）→ 乡镇畜牧兽医站（`town_vet_station`）→ 乡镇政府业务人员（`town_gov_staff`）→ 县级业务人员（`county_staff`）

```
企业端填报验收申请（ent_admin）
    ↓
  DRAFT（草稿）
    ↓ 企业提交
  PENDING_STATION_REVIEW（待站核实）
    ↓                            ↓
  STATION_APPROVED（站核实通过）  STATION_REJECTED（站核实退回）→ 消息通知企业→ 企业修改重提→ 回到PENDING_STATION_REVIEW
    ↓ 自动流转
  PENDING_TOWN_REVIEW（待镇政府审核）
    ↓                            ↓
  TOWN_APPROVED（镇政府审核通过） TOWN_REJECTED（镇政府审核退回）→ 消息通知企业→ 企业修改重提→ 回到PENDING_STATION_REVIEW
    ↓ 自动流转
  PENDING_COUNTY_REVIEW（待县局审核）
    ↓                            ↓
  ACCEPTANCE_APPROVED（验收通过）  COUNTY_REJECTED（县局审核退回）→ 消息通知企业→ 企业修改重提→ 回到PENDING_STATION_REVIEW
    ↓
  【验收完成】系统自动联动：project_application.application_status → ACCEPTED
    ↓
  乡镇畜牧兽医站（town_vet_station）可生成验收申请书PDF
  企业管理员（ent_admin）可下载PDF
```

### 各角色权限标识汇总

> 角色编码引用自 `需求梳理/第0阶段-多端权限架构/全局-角色与权限体系.md` §1.1 角色总览。

| 角色名称 | 角色编码（role_key） | 权限标识 | 功能范围 | 数据范围 |
|---------|---------------------|---------|---------|---------|
| 乡镇畜牧兽医站 | `town_vet_station` | `project:acceptance-audit:station` | 功能一：站核实 | 本部门及以下（`data_scope=4`） |
| 乡镇畜牧兽医站 | `town_vet_station` | `project:acceptance-pdf:generate` | 功能四：生成验收申请书PDF | 本部门及以下（`data_scope=4`） |
| 乡镇政府业务人员 | `town_gov_staff` | `project:acceptance-audit:town` | 功能二：镇政府审核 | 本部门及以下（`data_scope=4`） |
| 县级业务人员 | `county_staff` | `project:acceptance-audit:county` | 功能三：县局审核 | 全部数据（`data_scope=1`） |

### 数据字典（验收申请状态）

> 共11种取值，与企业端 PRD 中 `project_acceptance.acceptance_status` 定义保持一致。

| 状态编码 | 状态中文名 | 操作角色 | 说明 |
|---------|---------|---------|------|
| DRAFT | 草稿 | `ent_admin` | 企业端暂存，未提交 |
| PENDING_STATION_REVIEW | 待站核实 | — | 企业提交后，等待乡镇畜牧兽医站（`town_vet_station`）核实 |
| STATION_APPROVED | 站核实通过 | `town_vet_station` | 乡镇畜牧兽医站核实通过（系统自动流转至下一步） |
| STATION_REJECTED | 站核实退回 | `town_vet_station` | 乡镇畜牧兽医站核实退回，等待企业修改重提 |
| PENDING_TOWN_REVIEW | 待镇政府审核 | — | 站核实通过后系统自动流转至此 |
| TOWN_APPROVED | 镇政府审核通过 | `town_gov_staff` | 乡镇政府审核通过（系统自动流转至下一步） |
| TOWN_REJECTED | 镇政府审核退回 | `town_gov_staff` | 乡镇政府审核退回，等待企业修改重提 |
| PENDING_COUNTY_REVIEW | 待县局审核 | — | 镇政府审核通过后系统自动流转至此 |
| ACCEPTANCE_APPROVED | 验收通过 | `county_staff` | 县农业农村局审核通过，验收完成，联动申报状态更新为 `ACCEPTED` |
| COUNTY_REJECTED | 县局审核退回 | `county_staff` | 县农业农村局审核退回，等待企业修改重提 |
| WITHDRAWN | 已撤回 | `ent_admin` | 企业端用户主动撤回草稿，不可恢复 |

### 管理端与企业端数据表交叉引用

| 数据表 | 企业端PRD定义位置 | 管理端PRD定义位置 | 管理端操作方式 | 说明 |
|-------|-----------------|-----------------|-------------|------|
| `project_acceptance` | 企业端 §1.4.2 新建 | 管理端功能一～三复用 | 更新 `acceptance_status` 字段 + 功能四更新 `pdf_file_path`、`pdf_generate_time` | 管理端通过审核操作更新验收状态 |
| `project_acceptance_photo` | 企业端 §2.4.2 新建 | 管理端各功能只读展示 | 只读 | 管理端审核详情页展示验收补充材料照片 |
| `project_acceptance_audit_log` | 企业端 §3.4.2 新建 | 管理端功能一～三写入 | 写入审核记录 | 管理端写入各级审核日志 |
| `project_application` | 申报PRD §2.4.2 新建 | 管理端功能三联动更新 | 联动更新 `application_status` 为 `ACCEPTED` | 验收通过后联动 |
| `project_application_item` | 申报PRD §2.4.2 新建 | 管理端各功能只读展示 | 只读 | 管理端审核详情页展示建设内容明细 |
| `project_application_attachment` | 申报PRD §2.4.2 新建 | 管理端各功能只读展示 | 只读 | 管理端审核详情页展示资质附件 |
| `project_implementation_record` | 申报PRD §4.4.2 新建 | 管理端各功能只读展示 | 只读 | 管理端审核详情页展示实施过程记录 |
| `project_implementation_media` | 申报PRD §4.4.2 新建 | 管理端各功能只读展示 | 只读 | 管理端审核详情页展示实施过程照片/视频 |

### 验收申请与申报表字段对照（管理端审核详情页展示字段来源说明）

> 与企业端 PRD 附录「验收申请与申报表字段对照」保持一致，管理端全部字段均为只读展示。

| 纸质申请书字段 | 数据来源 | 所属表 | 管理端展示方式 |
|-------------|--------|-------|-------------|
| 申请单位（营业执照注册全名） | `company_name` | `project_application` | 只读文本 |
| 申请时间 | `apply_date` | `project_acceptance` | 只读文本 |
| 场址 | `address_town/village/group` | `project_application` | 只读文本 |
| 养殖场类型 | `farm_type` | `project_application` | 只读文本（展示中文名称） |
| 联系人 | `contact_name` | `project_application` | 只读文本 |
| 联系电话 | `contact_phone` | `project_application` | 只读文本 |
| 建设性质 | `construction_type` | `project_application` | 只读文本（展示中文名称） |
| 建设期限 | `build_start_date/end_date` | `project_application` | 只读文本 |
| 设计存栏规模（头） | `design_stock` | `project_application` | 只读文本 |
| 设计年出栏规模（头） | `design_annual_output` | `project_application` | 只读文本 |
| 建成后新增存栏（头） | `new_stock_after_build` | `project_application` | 只读文本 |
| 建成后新增出栏（头） | `new_output_after_build` | `project_application` | 只读文本 |
| 建成后饲养量（头） | `total_raise_after_build` | `project_application` | 只读文本 |
| 现存栏（头） | `current_stock` | `project_application` | 只读文本 |
| 养殖场资质办理情况 | 附件列表文字 | `project_application_attachment` | 只读文本（按6类分组展示文件名，支持预览/下载） |
| 建设内容明细 | 全部明细行 | `project_application_item` | 只读表格 |
| 合计投资金额 | `total_investment` | `project_application` | 只读文本 |
| 实施过程照片/视频 | 照片/视频文件 | `project_implementation_media` | 只读（折叠卡片，按建设内容分项+阶段展示，点击放大/播放） |
| 验收补充材料照片 | 照片文件 | `project_acceptance_photo` | 只读（折叠卡片，按子项目类别分组展示，点击放大） |
| 申报单位意见 | `applicant_opinion` | `project_acceptance` | 只读文本 |
| 畜牧兽医站意见 | `audit_opinion`（`STATION_AUDIT`节点） | `project_acceptance_audit_log` | 只读文本（审核流程进度轴+历史审核意见区展示） |
| 乡镇人民政府意见 | `audit_opinion`（`TOWN_AUDIT`节点） | `project_acceptance_audit_log` | 只读文本（审核流程进度轴+历史审核意见区展示） |
| 县农业农村局意见 | `audit_opinion`（`COUNTY_AUDIT`节点） | `project_acceptance_audit_log` | 只读文本（审核流程进度轴+历史审核意见区展示，仅验收通过后可见） |

### 验收状态与申报状态联动关系

> 与企业端 PRD 附录「验收状态与申报状态联动关系」保持一致。

| 验收申请状态 | 对应申报主记录状态 | 联动操作 | 管理端执行角色 |
|------------|----------------|---------|-------------|
| `DRAFT` ～ `PENDING_COUNTY_REVIEW`（审核中） | `COUNTY_APPROVED`（县审核通过，保持不变） | 无联动 | — |
| `ACCEPTANCE_APPROVED`（验收通过） | `ACCEPTED`（验收通过） | 验收通过时系统自动将 `project_application.application_status` 更新为 `ACCEPTED`（同一事务） | `county_staff` 审核通过后系统自动执行 |
| `STATION_REJECTED` / `TOWN_REJECTED` / `COUNTY_REJECTED`（退回） | `COUNTY_APPROVED`（县审核通过，保持不变） | 无联动，申报状态不受验收退回影响 | — |
| `WITHDRAWN`（已撤回） | `COUNTY_APPROVED`（县审核通过，保持不变） | 无联动 | — |
