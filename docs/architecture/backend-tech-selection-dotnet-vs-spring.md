# Backend 技术选型：.NET 还是 Spring Cloud

## 交付 framing

- scope: 基于当前三端需求、SaaS 目标和 nursing-backend-services 的目标形态，对 .NET 与 Spring Cloud 做专业选型评估并给出建议。
- affected audience: 技术负责人、架构师、后端负责人、招聘与交付团队。
- validation: 结论需和前端输入分析、总体架构、服务设计保持一致，并通过 `npm run docs:build`。
- rollback: 回退本文与索引更新，恢复到未明确技术结论的状态。

## 评估前提

当前项目特征如下：

- greenfield backend，可自由设计服务边界。
- 前端为 Next.js + Flutter，多端并行。
- 业务强 workflow、强审计、强对象状态机，不是单纯内容平台。
- 后续明确要做 SaaS，需要租户、套餐、配置和隔离策略。
- AI、消息、报表和通知会成为一等能力。

## 评估维度

| 维度 | .NET 10 | Spring Cloud |
| --- | --- | --- |
| greenfield 起步效率 | 高 | 中 |
| 中小团队一致性 | 高 | 中 |
| 多服务性能与内存占用 | 优 | 中到优 |
| BFF 与 Web API 开发速度 | 高 | 中 |
| 云原生生态成熟度 | 高 | 高 |
| 工作流与事件驱动生态 | 高 | 高 |
| 招聘与 Java 存量团队兼容 | 中 | 高 |
| 复杂治理组件成熟度 | 高 | 很高 |
| 学习曲线与样板复杂度 | 低到中 | 中到高 |
| SaaS 多租户实现难度 | 中 | 中 |

## .NET 方案分析

### 优势

- ASP.NET Core 在 API、BFF、gRPC、认证中间件和 OpenTelemetry 接入上非常直接。
- 对当前规模的团队和 greenfield 项目，代码风格更统一，样板更少。
- 运行时性能和资源占用更适合在 SaaS 早期控制成本。
- 与 PostgreSQL、Redis、RabbitMQ、OpenTelemetry 的集成成熟。
- 对 BFF 场景非常合适，Admin / Family / Nani 三套边缘服务可以快速一致化。

### 风险

- 如果未来团队主要由 Java / Spring 工程师构成，组织适配成本会上升。
- 若后续极度依赖 Spring 生态特有组件，迁移成本会增加。

## Spring Cloud 方案分析

### 优势

- 在超大规模微服务治理、组织级 Java 存量复用和企业中台经验方面更成熟。
- Spring Cloud Alibaba、Spring Security、Spring Data 等生态在大型企业中覆盖面广。
- 若已有强 Java 团队和大量复用中间件规范，Spring Cloud 可以更快接入现有基建。

### 风险

- 对当前项目阶段，样板代码、配置复杂度和运行资源消耗更高。
- 对三套 BFF + 多领域服务的早期建设，开发效率通常不如 .NET 轻量。
- 如果团队还没有稳定的 Java 微服务治理经验，容易一开始就引入过重平台负担。

## 结合本项目的专业结论

### 推荐结论

推荐优先选择 `.NET 10 + ASP.NET Core` 作为 nursing-backend-services 的主技术栈。

### 原因

1. 当前项目是 greenfield，最重要的是先把三端协同、工作流闭环和 SaaS 基线做对，而不是先引入最重的治理平台。
2. Admin、Family、Nani 三端都天然需要 BFF，.NET 在这类轻量边缘层和高性能 API 场景下更合适。
3. 养老护理业务的难点在 workflow、租户、审计和事件链路，不在 Java 特有的生态能力本身。
4. 早期 SaaS 更关注单位租户成本、部署密度和研发交付速度，.NET 在这几点更占优。
5. 如果后续服务数量和组织规模显著扩大，仍可在 .NET 生态内继续引入 service mesh、Dapr、Temporal、Kafka 等平台能力，而不必一开始押注最重框架。

## 推荐技术组合

### 核心框架

- runtime: .NET 10
- web: ASP.NET Core Minimal API / Web API
- edge: API Gateway + BFF pattern
- data: PostgreSQL
- cache: Redis
- message bus: RabbitMQ 起步，必要时升级 Kafka
- auth: OpenIddict 或 Keycloak
- observability: OpenTelemetry + Prometheus + Grafana + Loki / Seq

### 架构模式

- BFF for frontend
- DDD bounded context
- CQRS for read-heavy and task-heavy views
- outbox/inbox for domain events
- saga or workflow orchestration for long-running processes

### SaaS 必备能力

- tenant context propagation
- row-level or schema-level isolation
- feature flag per tenant
- audit trail per tenant
- billing and entitlement model per tenant

## 何时改选 Spring Cloud

只有在以下条件同时成立时，Spring Cloud 才应优先于 .NET：

- 团队核心研发力量明确以 Java 为主。
- 组织内部已有成熟 Spring Cloud 基建、规范和中间件平台。
- 项目短期内就会进入非常大规模服务治理，而不是先做 10 到 20 个高价值服务。

## 最终建议

本项目建议采用 `.NET 10 + ASP.NET Core + PostgreSQL + Redis + RabbitMQ + OpenTelemetry`，以轻量、强边缘层、强工作流、强 SaaS 基线为第一优先级。Spring Cloud 不是不能做，而是在当前阶段不够经济，也不够贴合项目的起步目标。