# Elderly Import Delivery Unit

## Scope

- Entry route: src/app/elderly/import/page.tsx
- Affected users: 档案补录、入住接待、护理主管、运营与质控用户
- Rollout stage: 第十七批长者资料导入治理说明

## User Impact

- 资料导入页为老人历史档案补录提供独立入口，允许从身份证、病历摘要和随访资料生成结构化草稿。
- 页面保持 AI 先抽取、人再确认的边界，不直接绕过入住审核或护理主管定级。
- 导入成功后统一跳转办理入住页，继续人工确认、计划生成和入册闭环。

## Data Source

- Route type: client page with local import state and shared admission workflow write path
- Primary source: elderly-document-intake mock extractor, template seeds, OCR 摘要文本和上传文件名
- Downstream dependency: addAdmissionApplication 以 document-import 来源写入 shared store，再跳转 `/elderly/checkin?entry=elderly-import`

## UI States

- Loading state: 当前识别与写入为本地同步流程；后续接真实 OCR/对象存储时需补充上传与识别中的进度反馈。
- Empty state: 初次进入时保持待识别状态，不应出现空白卡片或误导性的已入库提示。
- Error state: 缺少最小输入、必填字段缺失、身份证格式非法或识别后未复核直接提交时，必须显式报错。
- Mobile impact: 资料输入、识别结果和人工复核表单都较长，需保证窄屏滚动和提交按钮可达。

## Health Signals

- Healthy signal: 资料导入后可形成结构化草稿，进入入住审核页时保留 document-import 来源与资料摘要，并最终回流老人列表。
- Failure signal: 身份字段被占位符覆盖、导入来源丢失、审核页无法识别资料包，或列表无法追踪导入对象。
- Verification proxy: lint 和 build 通过；smoke 覆盖资料导入 -> 审核 -> 入住 -> 列表回流主路径。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Additional regression gate: npm run test:smoke or npm run verify:smoke
- Manual path: 选择模板或粘贴 OCR 摘要，确认 AI 识别结果、人工补齐字段、跳转办理入住页并回流老人列表

## Rollback

- Revert this delivery note together with the elderly import route, navigation entry, and smoke case.
- If regressions appear, fallback is the existing manual elderly new flow plus checkin review path, without document-import entry.