# Code Table Specification

> 码表说明文档 | 版本 v1.0 | 创建时间：2026-02-25
>
> 本文档定义系统中所有公共码表的结构与模拟码值，后续业务确认后统一更新正式码值。

---

## 目录

1. [Campus Management - 校区管理](#1-campus-management---校区管理)
2. [Unit Management - 单位管理](#2-unit-management---单位管理)
3. [Semester Management - 学期管理](#3-semester-management---学期管理)
4. [Room Type Management - 房间类型管理](#4-room-type-management---房间类型管理)
5. [Staff Type Management - 职工类型管理](#5-staff-type-management---职工类型管理)
6. [Staff Title Management - 职工职称管理](#6-staff-title-management---职工职称管理)
7. [Education Level Management - 文化程度管理](#7-education-level-management---文化程度管理)
8. [Staff Position Management - 职工职务管理](#8-staff-position-management---职工职务管理)
9. [On-duty Status Management - 在岗状态管理](#9-on-duty-status-management---在岗状态管理)
10. [Lab Center Category Management - 实验中心类别](#10-lab-center-category-management---实验中心类别)
11. [Lab Center Type Management - 实验中心类型](#11-lab-center-type-management---实验中心类型)
12. [Project Planning - 项目规划](#12-project-planning---项目规划)
13. [Project Type - 项目类型](#13-project-type---项目类型)
14. [Project Level - 项目级别](#14-project-level---项目级别)
15. [Importance Level - 重要性](#15-importance-level---重要性)
16. [Personnel Type - 人员类型](#16-personnel-type---人员类型)
17. [Personnel Responsibility - 人员职责](#17-personnel-responsibility---人员职责)
18. [Purchase Method - 购买方式](#18-purchase-method---购买方式)
19. [Quantitative Target Type - 量化目标类型管理](#19-quantitative-target-type---量化目标类型管理)
20. [Lab Project Type - 实验室项目类型](#20-lab-project-type---实验室项目类型)
21. [Equipment Category Management - 设备类别管理](#21-equipment-category-management---设备类别管理)
22. [Equipment Type Management - 设备类型管理](#22-equipment-type-management---设备类型管理)
23. [General Equipment Management - 通用设备管理](#23-general-equipment-management---通用设备管理)
24. [Equipment Origin Management - 设备产地管理](#24-equipment-origin-management---设备产地管理)
25. [Integrated System Management - 集成系统管理](#25-integrated-system-management---集成系统管理)
26. [Equipment Status Management - 设备状态管理](#26-equipment-status-management---设备状态管理)
27. [Dependent Project Type Management - 依托项目类型管理](#27-dependent-project-type-management---依托项目类型管理)
28. [Achievement Application Type Management - 成果应用类型管理](#28-achievement-application-type-management---成果应用类型管理)
29. [Consumable Category Management - 耗材分类管理](#29-consumable-category-management---耗材分类管理)
30. [Consumable Type Management - 耗材类型管理](#30-consumable-type-management---耗材类型管理)
31. [Consumable Usage Management - 耗材用途管理](#31-consumable-usage-management---耗材用途管理)

---

## 码表通用字段说明

| 字段名 | 类型 | 说明 |
|--------|------|------|
| `code` | String | 码值（唯一标识） |
| `name` | String | 名称（中文显示） |
| `sort` | Integer | 排序序号 |
| `status` | Integer | 状态：`1`-启用，`0`-禁用 |
| `remark` | String | 备注说明 |

---

## 1. Campus Management - 校区管理

**表名：** `sys_campus`  
**描述：** 管理学校各校区信息，用于标识设备、实验室等资源所属校区。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `CAMPUS_001` | 本部校区 | 1 | 1 | 主校区 |
| `CAMPUS_002` | 东校区 | 2 | 1 | — |
| `CAMPUS_003` | 南校区 | 3 | 1 | — |
| `CAMPUS_004` | 新校区 | 4 | 1 | 待启用 |

> 注：正式码值待业务确认后更新。

---

## 2. Unit Management - 单位管理

**表名：** `sys_unit`  
**描述：** 管理院系、部门等组织单位，用于人员归属、设备归属等关联。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `UNIT_001` | 信息工程学院 | 1 | 1 | — |
| `UNIT_002` | 机械工程学院 | 2 | 1 | — |
| `UNIT_003` | 化学工程学院 | 3 | 1 | — |
| `UNIT_004` | 生命科学学院 | 4 | 1 | — |
| `UNIT_005` | 实验室管理处 | 5 | 1 | 管理部门 |

> 注：正式码值待业务确认后更新。

---

## 3. Semester Management - 学期管理

**表名：** `sys_semester`  
**描述：** 定义学年学期信息，用于实验课程排课、耗材申领等按学期统计。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `SEM_2024_1` | 2024-2025学年第一学期 | 1 | 1 | — |
| `SEM_2024_2` | 2024-2025学年第二学期 | 2 | 1 | — |
| `SEM_2025_1` | 2025-2026学年第一学期 | 3 | 1 | 当前学期 |
| `SEM_2025_2` | 2025-2026学年第二学期 | 4 | 0 | 未开始 |

> 注：正式码值待业务确认后更新。

---

## 4. Room Type Management - 房间类型管理

**表名：** `sys_room_type`  
**描述：** 定义实验室房间的用途类型，用于实验用房管理。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `ROOM_TYPE_01` | 实验室 | 1 | 1 | 教学实验用途 |
| `ROOM_TYPE_02` | 研究室 | 2 | 1 | 科研用途 |
| `ROOM_TYPE_03` | 仪器室 | 3 | 1 | 存放精密仪器 |
| `ROOM_TYPE_04` | 准备室 | 4 | 1 | 实验准备用途 |
| `ROOM_TYPE_05` | 办公室 | 5 | 1 | 行政办公用途 |
| `ROOM_TYPE_06` | 储藏室 | 6 | 1 | 物资储存用途 |

> 注：正式码值待业务确认后更新。

---

## 5. Staff Type Management - 职工类型管理

**表名：** `sys_staff_type`  
**描述：** 区分职工的用工性质类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `STAFF_TYPE_01` | 在编教职工 | 1 | 1 | — |
| `STAFF_TYPE_02` | 合同制职工 | 2 | 1 | — |
| `STAFF_TYPE_03` | 外聘人员 | 3 | 1 | — |
| `STAFF_TYPE_04` | 返聘人员 | 4 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 6. Staff Title Management - 职工职称管理

**表名：** `sys_staff_title`  
**描述：** 定义职工的专业技术职称等级。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `TITLE_01` | 正高级 | 1 | 1 | 教授/研究员 |
| `TITLE_02` | 副高级 | 2 | 1 | 副教授/副研究员 |
| `TITLE_03` | 中级 | 3 | 1 | 讲师/助理研究员 |
| `TITLE_04` | 初级 | 4 | 1 | 助教 |
| `TITLE_05` | 未评定 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 7. Education Level Management - 文化程度管理

**表名：** `sys_education_level`  
**描述：** 定义职工的最高学历层次。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `EDU_01` | 博士研究生 | 1 | 1 | — |
| `EDU_02` | 硕士研究生 | 2 | 1 | — |
| `EDU_03` | 本科 | 3 | 1 | — |
| `EDU_04` | 大专 | 4 | 1 | — |
| `EDU_05` | 高中及以下 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 8. Staff Position Management - 职工职务管理

**表名：** `sys_staff_position`  
**描述：** 定义职工在单位中担任的行政职务。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `POS_01` | 院长 | 1 | 1 | — |
| `POS_02` | 副院长 | 2 | 1 | — |
| `POS_03` | 实验室主任 | 3 | 1 | — |
| `POS_04` | 实验室副主任 | 4 | 1 | — |
| `POS_05` | 实验技术人员 | 5 | 1 | — |
| `POS_06` | 无职务 | 6 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 9. On-duty Status Management - 在岗状态管理

**表名：** `sys_on_duty_status`  
**描述：** 标识职工当前在岗情况。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `DUTY_01` | 在岗 | 1 | 1 | — |
| `DUTY_02` | 离职 | 2 | 1 | — |
| `DUTY_03` | 退休 | 3 | 1 | — |
| `DUTY_04` | 借调 | 4 | 1 | — |
| `DUTY_05` | 长期病假 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 10. Lab Center Category Management - 实验中心类别

**表名：** `sys_lab_center_category`  
**描述：** 实验中心的大类别划分，用于实验中心分类管理。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `LAB_CAT_01` | 教学实验中心 | 1 | 1 | — |
| `LAB_CAT_02` | 科研实验中心 | 2 | 1 | — |
| `LAB_CAT_03` | 共享实验平台 | 3 | 1 | — |
| `LAB_CAT_04` | 工程实践中心 | 4 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 11. Lab Center Type Management - 实验中心类型

**表名：** `sys_lab_center_type`  
**描述：** 实验中心的细分类型，与类别配合使用。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `LAB_TYPE_01` | 基础实验室 | 1 | 1 | — |
| `LAB_TYPE_02` | 专业实验室 | 2 | 1 | — |
| `LAB_TYPE_03` | 综合实验室 | 3 | 1 | — |
| `LAB_TYPE_04` | 创新实验室 | 4 | 1 | — |
| `LAB_TYPE_05` | 虚拟仿真实验室 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 12. Project Planning - 项目规划

**表名：** `sys_project_planning`  
**描述：** 定义项目所属的规划阶段或规划批次。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `PLAN_01` | "十四五"规划 | 1 | 1 | 2021-2025年 |
| `PLAN_02` | "十五五"规划 | 2 | 1 | 2026-2030年 |
| `PLAN_03` | 年度计划 | 3 | 1 | 当年度计划 |
| `PLAN_04` | 专项计划 | 4 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 13. Project Type - 项目类型

**表名：** `sys_project_type`  
**描述：** 区分实验室建设项目的性质类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `PROJ_TYPE_01` | 新建项目 | 1 | 1 | — |
| `PROJ_TYPE_02` | 改建项目 | 2 | 1 | — |
| `PROJ_TYPE_03` | 扩建项目 | 3 | 1 | — |
| `PROJ_TYPE_04` | 设备购置项目 | 4 | 1 | — |
| `PROJ_TYPE_05` | 软件采购项目 | 5 | 1 | — |
| `PROJ_TYPE_06` | 维修改造项目 | 6 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 14. Project Level - 项目级别

**表名：** `sys_project_level`  
**描述：** 定义项目的资金来源层级或建设层级。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `PROJ_LEVEL_01` | 国家级 | 1 | 1 | — |
| `PROJ_LEVEL_02` | 省级 | 2 | 1 | — |
| `PROJ_LEVEL_03` | 市级 | 3 | 1 | — |
| `PROJ_LEVEL_04` | 校级 | 4 | 1 | — |
| `PROJ_LEVEL_05` | 院级 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 15. Importance Level - 重要性

**表名：** `sys_importance_level`  
**描述：** 用于标记事项、设备、项目等的重要程度。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `IMP_01` | 非常重要 | 1 | 1 | — |
| `IMP_02` | 重要 | 2 | 1 | — |
| `IMP_03` | 一般 | 3 | 1 | — |
| `IMP_04` | 不重要 | 4 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 16. Personnel Type - 人员类型

**表名：** `sys_personnel_type`  
**描述：** 区分参与项目或实验室管理的人员身份类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `PER_TYPE_01` | 教师 | 1 | 1 | — |
| `PER_TYPE_02` | 实验技术人员 | 2 | 1 | — |
| `PER_TYPE_03` | 研究生 | 3 | 1 | — |
| `PER_TYPE_04` | 本科生 | 4 | 1 | — |
| `PER_TYPE_05` | 外聘专家 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 17. Personnel Responsibility - 人员职责

**表名：** `sys_personnel_responsibility`  
**描述：** 定义人员在项目或实验室中承担的职责角色。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `RESP_01` | 负责人 | 1 | 1 | — |
| `RESP_02` | 副负责人 | 2 | 1 | — |
| `RESP_03` | 参与人员 | 3 | 1 | — |
| `RESP_04` | 安全员 | 4 | 1 | — |
| `RESP_05` | 设备管理员 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 18. Purchase Method - 购买方式

**表名：** `sys_purchase_method`  
**描述：** 定义设备、耗材等采购所采用的采购方式。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `PURCHASE_01` | 公开招标 | 1 | 1 | 金额≥限额标准 |
| `PURCHASE_02` | 竞争性谈判 | 2 | 1 | — |
| `PURCHASE_03` | 询价采购 | 3 | 1 | — |
| `PURCHASE_04` | 单一来源采购 | 4 | 1 | — |
| `PURCHASE_05` | 自行采购 | 5 | 1 | 金额较小 |
| `PURCHASE_06` | 框架协议采购 | 6 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 19. Quantitative Target Type - 量化目标类型管理

**表名：** `sys_quantitative_target_type`  
**描述：** 定义实验室建设或项目验收中量化考核指标的类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `QT_TYPE_01` | 教学工作量 | 1 | 1 | 单位：学时 |
| `QT_TYPE_02` | 科研经费 | 2 | 1 | 单位：万元 |
| `QT_TYPE_03` | 论文发表数 | 3 | 1 | 单位：篇 |
| `QT_TYPE_04` | 专利授权数 | 4 | 1 | 单位：项 |
| `QT_TYPE_05` | 获奖数量 | 5 | 1 | 单位：项 |
| `QT_TYPE_06` | 开放实验人次 | 6 | 1 | 单位：人次 |

> 注：正式码值待业务确认后更新。

---

## 20. Lab Project Type - 实验室项目类型

**表名：** `sys_lab_project_type`  
**描述：** 区分实验室承接或开展的项目类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `LAB_PROJ_01` | 教学项目 | 1 | 1 | — |
| `LAB_PROJ_02` | 纵向科研项目 | 2 | 1 | — |
| `LAB_PROJ_03` | 横向科研项目 | 3 | 1 | — |
| `LAB_PROJ_04` | 开放实验项目 | 4 | 1 | — |
| `LAB_PROJ_05` | 创新创业项目 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 21. Equipment Category Management - 设备类别管理

**表名：** `sys_equipment_category`  
**描述：** 设备的顶层大类别，按学科或用途进行划分。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `EQ_CAT_01` | 教学仪器设备 | 1 | 1 | — |
| `EQ_CAT_02` | 科研仪器设备 | 2 | 1 | — |
| `EQ_CAT_03` | 计算机及网络设备 | 3 | 1 | — |
| `EQ_CAT_04` | 图书文献资料 | 4 | 1 | — |
| `EQ_CAT_05` | 家具及其他设备 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 22. Equipment Type Management - 设备类型管理

**表名：** `sys_equipment_type`  
**描述：** 设备类别下的细分类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `EQ_TYPE_01` | 分析仪器 | 1 | 1 | — |
| `EQ_TYPE_02` | 电子测量仪器 | 2 | 1 | — |
| `EQ_TYPE_03` | 光学仪器 | 3 | 1 | — |
| `EQ_TYPE_04` | 机械设备 | 4 | 1 | — |
| `EQ_TYPE_05` | 计算机设备 | 5 | 1 | — |
| `EQ_TYPE_06` | 网络设备 | 6 | 1 | — |
| `EQ_TYPE_07` | 多媒体设备 | 7 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 23. General Equipment Management - 通用设备管理

**表名：** `sys_general_equipment`  
**描述：** 定义跨学科通用设备的标准目录，用于设备共享管理。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `GEN_EQ_01` | 台式计算机 | 1 | 1 | — |
| `GEN_EQ_02` | 笔记本电脑 | 2 | 1 | — |
| `GEN_EQ_03` | 投影仪 | 3 | 1 | — |
| `GEN_EQ_04` | 打印机 | 4 | 1 | — |
| `GEN_EQ_05` | 扫描仪 | 5 | 1 | — |
| `GEN_EQ_06` | 示波器 | 6 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 24. Equipment Origin Management - 设备产地管理

**表名：** `sys_equipment_origin`  
**描述：** 标识设备的生产来源地，用于国产/进口统计。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `ORIGIN_01` | 国产 | 1 | 1 | — |
| `ORIGIN_02` | 进口（美国） | 2 | 1 | — |
| `ORIGIN_03` | 进口（德国） | 3 | 1 | — |
| `ORIGIN_04` | 进口（日本） | 4 | 1 | — |
| `ORIGIN_05` | 进口（其他） | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 25. Integrated System Management - 集成系统管理

**表名：** `sys_integrated_system`  
**描述：** 定义与本系统对接的外部集成系统或子系统类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `INT_SYS_01` | 资产管理系统 | 1 | 1 | — |
| `INT_SYS_02` | 教务管理系统 | 2 | 1 | — |
| `INT_SYS_03` | 财务管理系统 | 3 | 1 | — |
| `INT_SYS_04` | 人事管理系统 | 4 | 1 | — |
| `INT_SYS_05` | 采购管理系统 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 26. Equipment Status Management - 设备状态管理

**表名：** `sys_equipment_status`  
**描述：** 标识设备当前的使用状态。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `EQ_STATUS_01` | 正常使用 | 1 | 1 | — |
| `EQ_STATUS_02` | 维修中 | 2 | 1 | — |
| `EQ_STATUS_03` | 闲置 | 3 | 1 | — |
| `EQ_STATUS_04` | 报废 | 4 | 1 | — |
| `EQ_STATUS_05` | 借出 | 5 | 1 | — |
| `EQ_STATUS_06` | 待验收 | 6 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 27. Dependent Project Type Management - 依托项目类型管理

**表名：** `sys_dependent_project_type`  
**描述：** 定义设备或成果所依托的立项项目类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `DEP_PROJ_01` | 国家自然科学基金 | 1 | 1 | — |
| `DEP_PROJ_02` | 国家社会科学基金 | 2 | 1 | — |
| `DEP_PROJ_03` | 省部级科研项目 | 3 | 1 | — |
| `DEP_PROJ_04` | 横向合作项目 | 4 | 1 | — |
| `DEP_PROJ_05` | 校级科研项目 | 5 | 1 | — |
| `DEP_PROJ_06` | 教改项目 | 6 | 1 | — |
| `DEP_PROJ_07` | 无依托项目 | 7 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 28. Achievement Application Type Management - 成果应用类型管理

**表名：** `sys_achievement_application_type`  
**描述：** 定义科研或教学成果的应用转化类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `ACH_TYPE_01` | 教学应用 | 1 | 1 | — |
| `ACH_TYPE_02` | 科研转化 | 2 | 1 | — |
| `ACH_TYPE_03` | 产学合作 | 3 | 1 | — |
| `ACH_TYPE_04` | 社会服务 | 4 | 1 | — |
| `ACH_TYPE_05` | 专利转让 | 5 | 1 | — |
| `ACH_TYPE_06` | 技术入股 | 6 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 29. Consumable Category Management - 耗材分类管理

**表名：** `sys_consumable_category`  
**描述：** 耗材的大类别划分，用于耗材统计与预算管理。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `CON_CAT_01` | 化学试剂 | 1 | 1 | — |
| `CON_CAT_02` | 生物试剂 | 2 | 1 | — |
| `CON_CAT_03` | 实验耗材 | 3 | 1 | — |
| `CON_CAT_04` | 办公耗材 | 4 | 1 | — |
| `CON_CAT_05` | 防护用品 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 30. Consumable Type Management - 耗材类型管理

**表名：** `sys_consumable_type`  
**描述：** 耗材分类下的细分类型。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `CON_TYPE_01` | 无机试剂 | 1 | 1 | — |
| `CON_TYPE_02` | 有机试剂 | 2 | 1 | — |
| `CON_TYPE_03` | 标准品 | 3 | 1 | — |
| `CON_TYPE_04` | 玻璃器皿 | 4 | 1 | — |
| `CON_TYPE_05` | 一次性耗材 | 5 | 1 | — |
| `CON_TYPE_06` | 防护手套 | 6 | 1 | — |
| `CON_TYPE_07` | 墨盒/硒鼓 | 7 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 31. Consumable Usage Management - 耗材用途管理

**表名：** `sys_consumable_usage`  
**描述：** 定义耗材的使用用途，便于成本核算与分类统计。

| code | name | sort | status | remark |
|------|------|------|--------|--------|
| `CON_USAGE_01` | 教学实验 | 1 | 1 | — |
| `CON_USAGE_02` | 科学研究 | 2 | 1 | — |
| `CON_USAGE_03` | 设备维护 | 3 | 1 | — |
| `CON_USAGE_04` | 安全防护 | 4 | 1 | — |
| `CON_USAGE_05` | 日常办公 | 5 | 1 | — |

> 注：正式码值待业务确认后更新。

---

## 变更记录

| 版本 | 日期 | 变更内容 | 变更人 |
|------|------|----------|--------|
| v1.0 | 2026-02-25 | 初始版本，创建全部31个码表定义及模拟码值 | — |
