# Admin 机构养老委托入住工作流设计

## Scope

- scope: 设计机构养老委托入住的数据模型、共享 store 扩展和四个前端入口页的职责边界。
- boundaries: 仅覆盖 admin 前端共享 workflow store 与页面；不改 backend BFF、数据库或微服务契约。
- dependencies:
  - `src/lib/mock/assessment-workflow.ts`
  - `src/app/elderly/new/page.tsx`
  - `src/app/elderly/[id]/edit/page.tsx`
  - `src/app/elderly/checkin/page.tsx`
  - `src/app/elderly/entrustment/page.tsx`
- rollback: 回退新增字段、页面和入口链接，恢复当前机构入住 workflow。

## Entry Points

- 新建入口: `/elderly/new`
- 编辑入口: `/elderly/[id]/edit`
- 机构认定入口: `/elderly/checkin?scene=institutional`
- 机构委托工作台: `/elderly/entrustment`

## Core Model

在现有 `AdmissionFormState` / `AdmissionApplication` 上增加机构入住专属字段：

- `entrustmentType`: `政府委托 | 企业委托`
- `entrustmentOrganization`: 委托主体名称
- `monthlySubsidy`: 固定月补贴金额
- `serviceItems`: 固定服务项目数组

设计原则：

- 不引入新的独立 store，继续复用 `assessment-workflow` 作为机构入住和个案评定的共享源。
- 新建和编辑都写回同一个 store，避免建档页、编辑页、认定页、工作台读到不同对象快照。
- 对历史缺字段个案允许读取，但在工作台中明确标为“待补录委托信息”。

## Workflow Mapping

工作台阶段不新建后端状态字段，而在前端按对象完整性和认定状态推导：

1. `待补录委托信息`
   - 委托来源、委托单位、月补贴或服务项目任一缺失。
2. `待认定确认`
   - 委托信息完整，且认定状态为 `待人工确认`。
3. `待补贴服务生效`
   - 委托信息完整，且认定状态为 `计划已生成`。
4. `服务中`
   - 委托信息完整，且认定状态为 `已入住`。

这样可以在不扩大状态机 blast radius 的前提下，补齐运营视角工作台。

## Page Responsibilities

### `/elderly/new`

- 继续承担机构入住建档入口。
- 在现有“基础信息”和“评估输入”之间增加“委托与补贴”“固定服务项目”区块。
- 提交校验时，机构入口必须要求委托来源、委托单位、月补贴和至少一个服务项目。

### `/elderly/[id]/edit`

- 从静态假页改为读取共享 workflow store 的真实编辑页。
- 预填已有委托与补贴字段。
- 保存时回写共享对象，而不是只做假延时跳转。

### `/elderly/checkin?scene=institutional`

- 保持个案评定中心定位。
- 在选中个案详情区增加委托来源、月补贴和服务项目摘要卡片。
- 对委托字段不完整的对象给出显式提示，并提供跳转到编辑页或工作台的动作。

### `/elderly/entrustment`

- 新增机构养老委托入住工作台。
- 展示 KPI：政府委托数、企业委托数、月补贴总额、待补录数。
- 展示按推导阶段分组的对象列表，并提供 `去认定`、`去编辑`、`查看档案` 操作。

## Observability

- 新对象提交后，工作台和认定页都应立即反映同一条委托数据。
- 工作台需要显式展示哪些对象因为缺字段被卡在待补录阶段。
- 编辑页保存后，若字段补齐成功，工作台阶段应同步迁移。

## Verification

- docs: `npm run docs:build`
- frontend: `npm run lint && npm run build`
- manual:
  1. 从 `/elderly/new` 创建一位政府委托老人，确认跳转到机构认定页时可见委托摘要。
  2. 打开 `/elderly/entrustment`，确认该对象出现在正确阶段。
  3. 打开 `/elderly/[id]/edit`，修改为企业委托并更新补贴，确认工作台同步刷新。

## Rollback

- 回退共享 store 新字段、编辑逻辑和新工作台路由。
- 若只发现编辑页回归问题，可保留新建和工作台，单独回退编辑写回逻辑。