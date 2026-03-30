# nursing-documents

用于集中存储养老护理项目的开发文档、设计说明、接口定义、运维手册和会议纪要。

当前工程已升级为 VitePress 文档站，可本地预览和构建静态站点。

文档索引主入口为 `docs/doc-index.md`，`docs/INDEX.md` 仅保留为迁移说明页。

## 目录结构

```text
docs/
  .vitepress/      VitePress 配置
  requirements/    需求文档
  architecture/    架构设计
  api/             接口文档
  ui/              UI/交互文档
  operations/      运维与发布文档
  meeting-notes/   会议纪要
  README.md        文档首页
  INDEX.md         文档索引
templates/         文档模板
```

## 本地启动

```bash
npm install
npm run docs:dev
```

默认会启动一个本地文档站，用于预览和持续补充开发文档。

## 构建站点

```bash
npm run docs:build
```

如需本地预览构建结果：

```bash
npm run docs:preview
```

## 使用约定

1. 新文档优先放到最贴近主题的目录中。
2. 文档命名建议采用 `YYYY-MM-DD-主题.md` 或 `模块名-主题.md`。
3. 涉及接口、架构、流程变更时，同步更新相关索引和历史说明。
4. 会议纪要建议记录结论、待办、负责人和截止时间。

## 推荐维护方式

- 需求先落在 `docs/requirements/`
- 方案设计落在 `docs/architecture/`
- API 协议落在 `docs/api/`
- 页面与交互说明落在 `docs/ui/`
- 发布、部署、巡检、回滚落在 `docs/operations/`
- 沟通记录落在 `docs/meeting-notes/`

## 初始化内容

当前工程已预置：

- VitePress 文档站配置
- 文档首页
- 文档索引
- 需求模板
- 架构设计模板
- API 模板
- 会议纪要模板
- 发布检查清单模板
