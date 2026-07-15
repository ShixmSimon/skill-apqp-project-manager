---
name: APQP技能助手
slug: apqp-project-manager
displayName: APQP技能助手
description: 辅助APQP项目管理全流程；支持项目阶段定义、文档生成（FMEA/PPAP/控制计划）、甘特图可视化、风险分析与知识库查询；适用于制造业产品质量规划全流程
version: 1.1.0
category: quality
author: org-jaxjwo0r
---
# APQP项目管理助手

## 任务目标
- 本Skill用于:制造业产品质量规划与项目管理，辅助用户完成从概念设计到量产的全流程质量管理
- 能力包含:APQP五阶段指导、标准文档生成、进度可视化、风险分析、知识库查询
- 触发条件:用户提到"APQP"、"产品质量规划"、"FMEA"、"PPAP"、"控制计划"、"项目阶段"或需要项目进度追踪

## 前置准备
- 依赖说明:jinja2==3.1.2（用于文档模板渲染）
- 非标准文件准备:需用户提供产品基本信息、供应商清单、设计参数等原始数据

## APQP五阶段概览

| 阶段 | 名称 | 核心输出 | 典型时长 |
|------|------|----------|----------|
| 阶段1 | 策划与定义 | 项目章程、QFD矩阵、初始BOM | 4-8周 |
| 阶段2 | 产品设计 | DFMEA、设计验证报告、样件 | 8-16周 |
| 阶段3 | 过程设计 | PFMEA、控制计划、PFMEA | 6-12周 |
| 阶段4 | 产品与过程验证 | PPAP、试生产、尺寸报告 | 4-8周 |
| 阶段5 | 反馈与改进 | 量产移交、持续改进计划 | 持续 |

## 操作步骤

### 一、项目初始化

1. **收集项目基本信息**
   - 产品名称、型号、目标市场
   - 预计年产量、单位成本目标
   - 关键客户要求（特殊特性）
   - 项目时间线（启动SOP目标日期）

2. **确定项目团队与职责**
   - 项目经理、质量工程师、设计工程师
   - 工艺工程师、采购、供应商
   - 明确RACI矩阵

### 二、阶段执行指导

**阶段1-策划与定义**
- 创建项目章程文档
- 开展多方论证会议
- 建立初始过程流程图
- 定义特殊产品特性

**阶段2-产品设计开发**
- 调用FMEA生成工具创建DFMEA
- 执行设计评审（DR1/DR2/DR3）
- 进行设计验证（DVP&R）
- 样件制作与认可

**阶段3-过程设计开发**
- 调用FMEA生成工具创建PFMEA
- 编制控制计划
- 开发工艺流程图
- 完成PFMEA与控制计划联动

**阶段4-产品与过程验证**
- 执行试生产（PPAP）
- 收集PPAP文件包
- 调用PPAP生成工具生成文件清单
- 客户提交与认可

**阶段5-反馈评定与纠正措施**
- 总结项目经验教训
- 建立持续改进机制
- 移交到量产阶段

### 三、文档生成（脚本调用）

#### 3.1 生成FMEA文档

```bash
# 最简用法：生成带一个占位行的 DFMEA 模板
python scripts/generate_fmea.py \
  --type dfmea \
  --project "产品A项目" \
  --output output/fmea_report.md

# 带入真实数据（data 为 FMEA 条目 JSON 数组）并指定负责人与 RPN 阈值
python scripts/generate_fmea.py \
  --type dfmea \
  --project "产品A项目" \
  --data data/fmea_items.json \
  --owner "张三" \
  --rpn-threshold 100 \
  --output output/fmea_report.md
```

参数说明:
- `--type`: dfmea(设计FMEA) 或 pfmea(过程FMEA)，必填
- `--project`: 项目名称，必填
- `--output`: 输出文件路径，必填
- `--data`: FMEA 数据 JSON 文件路径(可选)；不传则生成占位模板
- `--owner` / `--designer` / `--quality-engineer` / `--process-engineer`: 团队成员(可选)
- `--rpn-threshold`: 高 RPN 阈值，默认 100(可选)
- `--version` / `--date`: 文档版本与日期(可选)

生成内容包含:项目信息表、团队成员、严重度/频度/探测度评分准则、RPN计算逻辑、风险矩阵、改进建议

#### 3.2 生成控制计划

```bash
python scripts/generate_control_plan.py \
  --project "产品A项目" \
  --data data/control_plan_items.json \
  --customer "客户A" \
  --part-number "PN-12345" \
  --author "李四" \
  --reviewer "王五" \
  --output output/control_plan.md
```

参数说明:
- `--project`: 项目名称，必填
- `--output`: 输出文件路径，必填
- `--data`: 控制计划条目 JSON 文件路径(可选)；不传则生成占位模板
- `--customer` / `--part-number` / `--part-name` / `--company-name` / `--plan-number`: 表头信息(可选)
- `--author` / `--reviewer`: 编制人/审核人(可选)
- 注意:控制计划数据来自 `--data` JSON，**不会**自动读取 FMEA；特殊特性需在数据中用 `special_chars` 字段标明(SC/CC/KCC/KPC)

生成内容包含:控制计划表头、产品/过程特殊特性、关键工序控制参数、测量系统要求、不良处理方式

#### 3.3 生成PPAP文件包

```bash
python scripts/generate_ppap.py \
  --level 3 \
  --submission-type "首次提交" \
  --customer "客户A" \
  --part_number "PN-12345" \
  --output output/ppap_checklist.json
```

参数说明:
- `--level`: PPAP提交等级(1-5)，默认3
- `--submission-type`: 提交类型(首次/重新/设计变更)
- `--customer`: 客户名称
- `--part_number`: 零件号
- `--output`: 输出清单JSON路径

生成内容包含:19项PPAP要素清单、每项要求说明、状态追踪表(待完成/已完成/不适用)、提交日期追踪

#### 3.4 生成甘特图

```bash
# 方式一：不传 --phases，使用内置的 APQP 五阶段默认计划
python scripts/generate_gantt.py \
  --project "产品A项目" \
  --start-date 2024-01-01 \
  --sop-date 2024-06-01 \
  --output output/gantt.html

# 方式二：传入自定义阶段 JSON（结构见下）
python scripts/generate_gantt.py \
  --project "产品A项目" \
  --phases data/phases.json \
  --start-date 2024-01-01 \
  --sop-date 2024-06-01 \
  --output output/gantt.html
```

参数说明:
- `--project`: 项目名称，必填
- `--start-date`: 项目开始日期(YYYY-MM-DD)，必填
- `--sop-date`: 量产开始(SOP)目标日期，必填
- `--output`: 输出 HTML 甘特图路径，必填
- `--phases`: 阶段定义 **JSON 文件**(可选)；**注意是 JSON 不是 Markdown**，省略则使用内置默认五阶段

`--phases` JSON 结构示例:
```json
[
  {
    "name": "阶段1: 策划与定义",
    "tasks": [
      {"name": "项目立项", "start": "2024-01-01", "end": "2024-01-15"},
      {"name": "阶段1评审", "start": "2024-01-15", "end": "2024-01-15", "type": "milestone"}
    ]
  }
]
```

生成内容包含:交互式甘特图、阶段里程碑标记、进度百分比显示、关键路径标识

### 四、风险分析与优化

1. **风险识别方法**
   - 审查FMEA中的高RPN项(>100需优先处理)
   - 检查控制计划中的特殊特性标识
   - 回顾历史项目类似问题

2. **调用风险分析脚本**

```bash
python scripts/analyze_risk.py \
  --fmea-files "output/fmea_dfmea.md,output/fmea_pfmea.md" \
  --control-plan "output/control_plan.md" \
  --output output/risk_analysis.md
```

分析内容:RPN分布统计、高风险项清单、推荐的纠正措施、优先级排序

### 五、知识库查询

**何时读取参考文档**:
- 需要详细阶段输出清单 → 读取 [references/apqp_phases.md](references/apqp_phases.md)
- 需了解IATF 16949标准要求 → 读取 [references/iatf16949_knowledge.md](references/iatf16949_knowledge.md)
- 需要最佳实践指导 → 读取 [references/best_practices.md](references/best_practices.md)

## 使用示例

### 示例1: 新项目启动
- 场景/输入:用户说"我们要启动一个新产品的APQP项目，产品是汽车发动机支架"
- 预期产出:项目章程模板、APQP阶段计划、初步风险清单
- 关键要点:收集产品信息(材料、工艺要求、客户特殊要求)、确定SOP目标日期、建立项目团队

### 示例2: DFMEA编制
- 场景/输入:用户提供产品设计信息，要求生成DFMEA
- 预期产出:完整的DFMEA文档，包含失效模式、潜在原因、控制措施
- 关键要点:需要产品功能分解、接口分析、边界图定义

### 示例3: PPAP准备
- 场景/输入:项目进入试生产阶段，需要准备PPAP文件包
- 预期产出:PPAP 19项清单、每项状态追踪、缺失项提醒
- 关键要点:确认客户要求(提交等级)、检查各项文件完备性

## 资源索引

- 脚本:见 [scripts/generate_fmea.py](scripts/generate_fmea.py)（用途:生成DFMEA/PFMEA文档；参数:type/project/output，可选:data/owner/designer/quality-engineer/process-engineer/rpn-threshold/version/date）
- 脚本:见 [scripts/generate_control_plan.py](scripts/generate_control_plan.py)（用途:生成控制计划；参数:project/output，可选:data/customer/part-number/author/reviewer 等表头信息）
- 脚本:见 [scripts/generate_ppap.py](scripts/generate_ppap.py)（用途:生成PPAP文件清单；参数:level/submission-type/customer/part_number/output）
- 脚本:见 [scripts/generate_gantt.py](scripts/generate_gantt.py)（用途:生成甘特图HTML；参数:project/start-date/sop-date/output，可选:phases JSON）
- 脚本:见 [scripts/analyze_risk.py](scripts/analyze_risk.py)（用途:综合风险分析；参数:fmea-files/control-plan/output，可选:project）
- 参考:见 [references/apqp_phases.md](references/apqp_phases.md)（何时读取:需要详细阶段输出清单、节点检查表）
- 参考:见 [references/iatf16949_knowledge.md](references/iatf16949_knowledge.md)（何时读取:查询IATF 16949标准要求、审核要点）
- 参考:见 [references/best_practices.md](references/best_practices.md)（何时读取:获取行业最佳实践、避免常见错误）

## 注意事项

- 生成文档前需用户提供基本项目信息，无信息则生成模板占位符
- PPAP要求因客户而异，生成后需对照客户特殊要求调整
- FMEA需团队协作，不要仅依赖AI单独完成
- 甘特图生成后可根据实际进展调整日期
- 知识库内容仅供参考，最终以客户和标准要求为准

## TRACE 测评

| 维度 | 评分 | 说明 |
|------|------|------|
| T — 可信任度 | 10/10 | 纯文档/脚本技能，无外部依赖风险，支持中文交互；已声明安全注意事项 |
| R — 可靠性 | 9/10 | 有异常处理说明; 输出格式明确 |
| A — 适用性 | 9/10 | 有适用范围声明; 触发条件明确 |
| C — 规范性 | 10/10 | frontmatter 完整; 文档结构清晰; 内容充分 |
| E — 有效性 | 10/10 | 输出明确; 含使用示例; 文档详尽 |
| **总分** | **48/50** | 通过 |
