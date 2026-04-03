# LTCI Assessment Agency Product Solution

## Scope

- Source repo: nursing-admin-v2
- Entry scope: 现有 admin 中与老人档案、资料导入、评估认定、规则配置、定点机构、任务排期、财务结算、报表稽核相关的模块
- Affected users: 评估员、评估主管、规则配置专员、质控复核人员、财务结算人员、协同评估机构联系人
- Rollout stage: 长护险产品化重构第二阶段，先纠正系统定位，再完成关键页面和共享配置层接入

## Corrected Product Position

当前系统不应再被定义为“长护险经办与服务监管后台”，也不应再以“护理服务套餐”和“服务计划执行”作为主叙事。

根据最新业务要求，系统的正确定位是：

- 长护险失能等级评定机构运营平台
- 以灵活护理项和灵活评定标准为核心治理对象
- 以个案评定、复评复核、抽检回访、质控留痕和评估费结算为核心业务闭环

这一定义意味着：

1. 你们的核心产出是认定意见、结论依据和服务建议，不是护理服务执行本身。
2. 护理服务机构、排班执行、居家服务回执属于下游协同对象，不应继续主导系统结构。
3. 系统必须支持规则版本化、模板化和配置化，避免把长护险评定逻辑写死在页面里。

## Product Principles

### 1. 规则先行

- 所有个案评定都必须消费一套明确的规则版本。
- 规则版本要能复核、生效、停用，并能追溯到结论与结算。

### 2. 护理项灵活配置

- 护理项不再是“套餐项”，而是标准化的评定颗粒。
- 每个护理项都应配置适用场景、适用等级、证据要求和标准时长。

### 3. 模板作为认定输出骨架

- 模板不是服务计划。
- 模板负责把规则集、护理项、证据要求、结论摘要和后续动作组合起来，直接用于个案认定。

### 4. 个案评定只消费配置，不反向定义配置

- 个案页负责匹配、展示、确认和留痕。
- 个案页不应继续承担“临时拼配置”的职责。

### 5. 原系统可复用，但不能再产生 workflow 冲突

- 现有老人档案、资料导入、组织协同、任务中心、排期、财务和报表模块继续复用。
- 但这些模块的长护险语义必须统一切到评定机构视角。

## Target Information Architecture

建议保留现有路由，但统一按以下语义承接：

| 路由 | 新语义 | 说明 |
| --- | --- | --- |
| /nursing/services | 评定机构总览 | 展示规则配置、模板、个案、排期、结算和健康信号 |
| /nursing/packages | 评定标准配置 | 承接护理项库和评定规则集，不再表达为服务包 |
| /nursing/plans | 认定方案模板 | 承接首评、复评、抽检模板，不再表达为服务计划 |
| /elderly/checkin | 个案评定中心 | 承接受理、AI 辅助、人工认定、结论输出和协同机构视图 |
| /staff/tasks | 现场评定任务 | 承接首评、复评、抽检和整改任务 |
| /staff/schedule | 派案排期 | 承接评估员排期和协同机构派案 |
| /financial | 评定服务结算与质控 | 承接评估费结算、资料门禁、风险单和抽检整改 |
| /organizations/partners | 协同评估机构治理 | 承接评估机构、复评复核机构和抽检协作对象 |

## Core Domain Model

### A. 护理项库

- 字段：名称、分类、适用场景、适用等级、标准时长、证据要求、说明、启停状态
- 目标：支撑灵活评定，不把具体护理动作写死在个案页面中

### B. 评定规则集

- 字段：名称、版本、场景、适用等级、评分区间、阈值口径、证据要求、关联护理项、质控门禁、状态
- 目标：把长护险规则转成可治理、可生效的标准版本

### C. 认定方案模板

- 字段：模板名称、场景、目标等级、关联规则集、关联护理项、结论摘要、后续动作、状态
- 目标：输出结构化认定结论，而不是输出服务执行计划

### D. 个案评定

- 字段：申请信息、结构化输入、AI 建议、人工认定结果、匹配规则集、匹配模板、协同机构、认定结论、生效状态
- 目标：让每个个案都能追溯自己用了哪一套规则、模板和证据

### E. 结算与质控

- 字段：案件、场景、结算状态、资料完整性、风险标记、已确认金额、暂缓金额、质控动作
- 目标：围绕评估费而不是护理服务费建立结算闭环

## End-to-End Workflow

### 1. 标准配置闭环

- 配置护理项 -> 创建规则集草稿 -> 提交复核 -> 发布生效 -> 创建认定模板 -> 启用模板

### 2. 个案评定闭环

- 老人建档或资料导入 -> 进入个案评定中心 -> AI 给出建议 -> 系统匹配规则集和模板 -> 人工认定 -> 出具结论 -> 进入生效/回访/抽检

### 3. 协同复核闭环

- 指派协同评估机构 -> 首评回传 -> 复评复核 -> 抽检回访 -> 异常整改

### 4. 结算质控闭环

- 评定案件进入结算 -> 资料初审 -> 质控复核 -> 风险单处置 -> 评估费拨付 -> 留痕归档

## No-Conflict Integration Rules

为避免与现有系统产生 workflow 冲突，必须遵守以下接入规则：

1. `/nursing/packages` 只承接标准配置，不再承接服务套餐定价发布。
2. `/nursing/plans` 只承接认定模板，不再承接服务计划执行。
3. `/elderly/checkin` 的主要输出是认定结论和服务建议，不再以“入住审核”或“生成服务计划”作为页面主心智。
4. `/financial` 只讨论评估费和质控，不再继续沿用护理服务申报话术。
5. 任务中心和排期页可以保留，但其长护险来源语义应逐步切换为评定任务、复评任务和抽检任务。

## Development Plan

### Shared Store Layer

- New file: `src/lib/mock/assessment-config-workflow.ts`
- Responsibility:
  - 提供护理项库、规则集、认定模板三类共享配置
  - 提供本地持久化与订阅能力
  - 提供个案匹配规则和模板的派生函数

### Core UI Changes

- `src/components/nursing/NursingWorkflowPages.tsx`
  - `services` 路由改为评定机构总览
  - `packages` 路由改为评定标准配置
  - `plans` 路由改为认定方案模板
- `src/app/elderly/checkin/page.tsx`
  - 接入评定配置 store
  - 显示当前个案匹配到的规则集、模板和护理项
  - 把“服务计划”话术统一改成“认定结论与服务建议”
- `src/app/financial/page.tsx`
  - 统一改成评定服务结算与质控
  - 让结算页可见规则版本和模板上下文
- `src/components/layout/TopNavbar.tsx`
  - 导航统一改成评定机构模型
- `src/app/elderly/import/page.tsx`、`src/app/elderly/new/page.tsx`
  - 入口语义从“入住审核”切换为“个案评定”

### Route Documentation

- 更新 `ELDERLY_CHECKIN_DELIVERY.md`
- 更新 `FINANCIAL_DELIVERY.md`
- 更新 `PARTNER_SERVICE_WORKFLOWS_DELIVERY.md`

## Verification

- Admin minimum gate: `cd nursing-admin-v2 && npm run lint`
- Admin stronger gate: `cd nursing-admin-v2 && npm run build`
- Docs gate: `cd nursing-documents && npm run docs:build`
- Manual health signals:
  - `/nursing/services` 明确展示评定机构主线
  - `/nursing/packages` 可创建护理项和规则集
  - `/nursing/plans` 可创建认定模板
  - `/elderly/checkin` 可展示当前个案匹配到的规则集和模板
  - `/financial` 明确表达评估费结算与质控，而不是护理服务结算

## Rollback

- 回退本文档
- 回退 `assessment-config-workflow.ts`
- 回退 `NursingWorkflowPages.tsx`、`checkin/page.tsx`、`financial/page.tsx`、`TopNavbar.tsx` 和相关入口页文案
- 若需紧急回退，系统可恢复到上一版 demo 语义，但会重新暴露“评定机构定位错误”的产品风险

## Residual Risk

- 当前仍是前端 demo 配置和演示数据，尚未接真实规则引擎、BFF 和正式结算接口。
- 任务中心和排期页的下游任务语义还未完全切到评定任务模型，本轮先完成上游纠偏。
- `assessment-workflow.ts` 仍保留部分历史内部命名以兼容现有 store；后续可继续去掉 admission 兼容别名。
