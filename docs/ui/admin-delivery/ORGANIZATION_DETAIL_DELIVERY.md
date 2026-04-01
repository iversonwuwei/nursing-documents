# Organization Detail Delivery Unit

## Scope

- Entry route: src/app/organizations/[id]/page.tsx
- Affected users: 机构运营、床位协调、院长与人力协同用户
- Rollout stage: 第六批设备与组织扩展路由治理说明

## User Impact

- 机构详情页承担机构概览、床位管理、员工管理与对象级 AI 组织摘要的合流入口。
- 详情页现在也能读取新建待启用机构，不再只依赖本地硬编码对象。
- 保持床位和员工管理仍是只读原型展示，不自动改床位或调整人员配置。

## Data Source

- Route type: client detail route with local tab state and shared workflow subscription
- Primary sources: master-data-workflow merged organization、派生床位数据、员工名册和 AI organization helpers
- Downstream links: AI assistant context links for overview, beds, and staff roster

## UI States

- Loading state: 当前为本地同步 mock；后续接真实机构详情接口时需补 tab 内加载反馈。
- Empty state: 当前样本非空；未来若未命中组织或某 tab 无数据，需保持局部空态而不破坏详情骨架。
- Error state: 概览、床位、员工与 AI 摘要口径不一致时需局部暴露。
- Mobile impact: 多 tab、多表格和多 AI 卡并存，后续改动需验证窄屏堆叠和 tab 可达性。

## Health Signals

- Healthy signal: 机构概览、待启用状态、床位管理、员工管理和 AI 组织摘要围绕同一机构上下文保持一致。
- Failure signal: tab 内容错位、机构对象映射错误，或新建机构详情被回退到错误默认对象。
- Verification proxy: lint 通过；行为改动时加 build 与机构详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证机构概览、床位 tab、员工 tab 和 AI 链路

## Rollback

- Revert this delivery note and any future organization detail route changes together.
- If regressions appear, fallback is the previous local organization detail tabs and AI summary composition.
