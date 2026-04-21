# Branch Delivery Unit

## Scope

- Entry route: src/app/branch/page.tsx
- Affected users: 区域运营、院务管理、多分院统筹用户
- Rollout stage: 分院概览 Fluent 收敛与旧页替换

## User Impact

- 分院管理页作为多院区运营概览入口，首屏统计、优先分院队列与全部分院入口全部来自真实组织服务，而不是旧的本地硬编码分院数组。
- 首屏继续先帮助区域运营判断哪家分院需要先看，但分院总数、床位总数、在院人数、员工数、入住率等指标都按租户下真实机构列表汇总。
- 当前仍不引入新的写路径或编辑流；营收维度在 Billing 按机构聚合之前显式标注为待接入。

## Data Source

- Route type: live branch overview page
- Primary source: `fetchAdminOrganizationList()` in `src/lib/organizations/admin-organization-api.ts` -> Admin BFF `/api/admin/organizations` -> Organization + Rooms + Staff + Elder 聚合
- Aggregation reuse: 复用 Admin BFF 已有 `AdminOrganizationSummary` 的 `totalBeds / occupiedBeds / availableBeds / elderlyCount / staffCount / roomCount` 字段，不新增 BFF 或后端端点。
- Intentional gap: 营收字段尚未下沉到机构维度，页面显式标注“营收接入中”，等 Billing 按机构聚合后再补。

## UI States

- Loading state: 初次加载时显示总览骨架与“加载中”提示，而非空白卡片。
- Empty state: 当前租户无任何已建档机构时，显式提示“暂无分院记录”，并给出进入机构管理建档的入口。
- Error state: `/api/admin/organizations` 非 2xx 或字段异常时显式展示错误文案与重试按钮，不回退到任何本地 mock 列表。
- Mobile impact: 总览区、优先队列和分院入口卡在窄屏下继续保持纵向堆叠，不依赖横向指标条才可读。

## Health Signals

- Healthy signal: 登录后 `/branch` 首屏 KPI、优先分院卡与全部入口卡与 `/organizations` 列表口径完全一致（床位、在院、员工、入住率）。
- Failure signal: 出现“本地概览数据”字样、营收以外的字段以 `¥`/硬编码呈现，或页面回退到旧的 BRANCHES 静态数组。
- Verification proxy: lint + build 通过；行为改动时加分院页人工回归，确认与 `/organizations` 数据一致。

## Verification

- Minimum gate: `npm run lint`
- Required gate for this batch (behavior change): `npm run lint` + `npm run build`
- Manual path: 登录 admin -> 进入 `/branch`，确认 KPI 汇总与 `/organizations` 列表字段一致；停掉 Admin BFF 后应看到明确错误态而非静态数据。

## Rollback

- Revert this delivery note together with `src/app/branch/page.tsx` 前端 commit 即可回到之前的本地 BRANCHES 概览。
- 由于未引入新后端或 BFF 端点，不存在需要同步回滚的服务侧改动。
