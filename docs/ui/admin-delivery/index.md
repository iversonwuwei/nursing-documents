# Admin Route Delivery Archive

该目录统一归档 nursing-admin-v2 的页面级与路由级交付文档，用于跨工程集中管理、长期记录和站点内检索。

## Scope

- Source repo: nursing-admin-v2
- Audience: admin frontend maintainers, reviewers, release owners, and cross-project document readers
- Rollout stage: admin 路由交付文档集中归档
- Validation gates:
  - nursing-documents: `npm run docs:build`
  - nursing-admin-v2: `npm run lint`, `npm run build`, `npm run test:smoke`, `npm run verify:smoke`
- Rollback path: revert the moved docs here together with the thin local index updates in nursing-admin-v2

## Current Status

- Canonical route delivery notes for nursing-admin-v2 now live in this folder.
- The admin repo keeps only the local delivery template, local route inventory, verification checklist, and a thin DELIVERY_INDEX entry.
- Route verification automation is still incremental; the current smoke slice covers root redirect, login, equipment and devices status compatibility, AI assistant tracking-context flow, health compatibility plus metric routing, analytics alias plus report period switching, and the governed elderly create or import loops.

## Cross-cutting Design

- [Admin Create Flows Solution](./CREATE_FLOWS_SOLUTION.md)

## Route Delivery Notes

### Batch 1

- [Dashboard](./DASHBOARD_DELIVERY.md)
- [Alerts](./ALERTS_DELIVERY.md)
- [Elderly](./ELDERLY_DELIVERY.md)

### Batch 2

- [Elderly Detail](./ELDERLY_DETAIL_DELIVERY.md)
- [Health Monitoring](./HEALTH_MONITORING_DELIVERY.md)
- [Alerts History](./ALERTS_HISTORY_DELIVERY.md)
- [Incidents](./INCIDENTS_DELIVERY.md)

### Batch 3

- [Incident Detail](./INCIDENT_DETAIL_DELIVERY.md)
- [Activities](./ACTIVITIES_DELIVERY.md)
- [AI Logs](./AI_LOGS_DELIVERY.md)
- [AI Rules](./AI_RULES_DELIVERY.md)

### Batch 4

- [Activities Detail](./ACTIVITIES_DETAIL_DELIVERY.md)
- [AI Inference](./AI_INFERENCE_DELIVERY.md)
- [AI Family App](./AI_FAMILY_APP_DELIVERY.md)
- [AI Staff App](./AI_STAFF_APP_DELIVERY.md)

### Batch 5

- [Staff](./STAFF_DELIVERY.md)
- [Rooms](./ROOMS_DELIVERY.md)
- [Equipment](./EQUIPMENT_DELIVERY.md)
- [Financial](./FINANCIAL_DELIVERY.md)

### Batch 6

- [Devices](./DEVICES_DELIVERY.md)
- [Supplies](./SUPPLIES_DELIVERY.md)
- [Organization Detail](./ORGANIZATION_DETAIL_DELIVERY.md)
- [Staff Detail](./STAFF_DETAIL_DELIVERY.md)
- [Staff Tasks](./STAFF_TASKS_DELIVERY.md)
- [Staff Schedule](./STAFF_SCHEDULE_DELIVERY.md)

### Batch 7

- [Devices Detail](./DEVICES_DETAIL_DELIVERY.md)
- [Devices Assets](./DEVICES_ASSETS_DELIVERY.md)
- [Devices Realtime](./DEVICES_REALTIME_DELIVERY.md)
- [Devices Stats](./DEVICES_STATS_DELIVERY.md)
- [Devices Status](./DEVICES_STATUS_DELIVERY.md)

### Batch 8

- [Supplies Detail](./SUPPLIES_DETAIL_DELIVERY.md)
- [Rooms Detail](./ROOMS_DETAIL_DELIVERY.md)
- [Organizations](./ORGANIZATIONS_DELIVERY.md)
- [Settings](./SETTINGS_DELIVERY.md)
- [Notifications](./NOTIFICATIONS_DELIVERY.md)

### Batch 9

- [Equipment Detail](./EQUIPMENT_DETAIL_DELIVERY.md)
- [Equipment Monitor](./EQUIPMENT_MONITOR_DELIVERY.md)
- [Equipment Stats](./EQUIPMENT_STATS_DELIVERY.md)
- [Equipment Status](./EQUIPMENT_STATUS_DELIVERY.md)

### Batch 10

- [Root](./ROOT_DELIVERY.md)
- [Login](./LOGIN_DELIVERY.md)
- [Branch](./BRANCH_DELIVERY.md)
- [Analytics](./ANALYTICS_DELIVERY.md)
- [Data Dashboard](./DATA_DASHBOARD_DELIVERY.md)
- [Settings Roles](./SETTINGS_ROLES_DELIVERY.md)
- [Analytics Report](./ANALYTICS_REPORT_DELIVERY.md)

### Batch 11

- [Elderly New](./ELDERLY_NEW_DELIVERY.md)
- [Elderly Edit](./ELDERLY_EDIT_DELIVERY.md)
- [Elderly Health](./ELDERLY_HEALTH_DELIVERY.md)
- [Elderly Vitals](./ELDERLY_VITALS_DELIVERY.md)
- [Elderly Visits](./ELDERLY_VISITS_DELIVERY.md)
- [Elderly Checkin](./ELDERLY_CHECKIN_DELIVERY.md)

### Batch 12

- [Health Root](./HEALTH_ROOT_DELIVERY.md)
- [Health Metric](./HEALTH_METRIC_DELIVERY.md)

### Batch 13

- [AI Assistant Root](./AI_ASSISTANT_DELIVERY.md)

### Batch 14

- [Organizations New](./ORGANIZATIONS_NEW_DELIVERY.md)
- [Rooms New](./ROOMS_NEW_DELIVERY.md)

### Batch 15

- [Activities](./ACTIVITIES_DELIVERY.md)
- [Activities Detail](./ACTIVITIES_DETAIL_DELIVERY.md)
- [Activities New](./ACTIVITIES_NEW_DELIVERY.md)
- [Incidents](./INCIDENTS_DELIVERY.md)
- [Incident Detail](./INCIDENT_DETAIL_DELIVERY.md)
- [Incidents New](./INCIDENTS_NEW_DELIVERY.md)

### Batch 16

- [Staff New](./STAFF_NEW_DELIVERY.md)
- [Equipment New](./EQUIPMENT_NEW_DELIVERY.md)
- [Supplies New](./SUPPLIES_NEW_DELIVERY.md)
- [Elderly Health New](./ELDERLY_HEALTH_NEW_DELIVERY.md)
- [Elderly Vitals New](./ELDERLY_VITALS_NEW_DELIVERY.md)
- [Elderly Visits New](./ELDERLY_VISITS_NEW_DELIVERY.md)

### Batch 17

- [Elderly Import](./ELDERLY_IMPORT_DELIVERY.md)

### Batch 18

- [Elderly Face](./ELDERLY_FACE_DELIVERY.md)

## Next Steps

- Expand automated route verification beyond the current smoke slice, prioritizing the new resource and health-service create-loop paths after implementation lands.
- Keep route-level delivery notes here as the canonical archive, while local validation and execution guidance stays in nursing-admin-v2.
