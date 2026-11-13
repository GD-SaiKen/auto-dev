# 开发约束

来源：dev-standards。同步日期：2026-11-13。

冲突时按这个顺序：本期 `docs/SDD` > 仓库 `docs/项目-业务约定.md` > 本文件。没有 SDD 也继续做，不要停下来等规格。

开工前先读 Issue 原文。仓库里有 `AGENTS.md`、`docs/项目-业务约定.md` 就读。Issue 若点了某份 `docs/SDD`，以那份为准。

## 通用

- 最小改动。不改 Issue 没提到的文件，不做顺手重构。
- 禁止修改 `.github/`。
- 新的英文源文件用 kebab-case。Go、Dart 源文件用 snake_case。
- JSON 字段用 camelCase。
- 新单测跟该仓库已有测试放在一起。Vue 新单测放在 `src/__test__/`，路径镜像 `src/`。
- 只跑 `package.json`、`pubspec.yaml` 或实施计划里已经有的检查命令。有 `lint`、`test`、`build` 就跑。

## Vue

- 组件：`components/<feature-kebab>/<feature-kebab>.vue`
- 页面：`views/<page-kebab>/index.vue`
- 单文件组件顺序：`template`、`script`、`style`

## 有 HTTP 接口时

成功体为 `{ "code": 200, "data": {}, "msg": "success", "message": "操作成功" }`。列表的 `data` 为 `{ "list": [], "total": 0, "page": 1, "pageSize": 10 }`。业务代码只返回 `data` 里的内容，不要再套一层信封。

## 摘要

把实现摘要写到 `/tmp/auto-dev-summary.md`，包含：这是缺陷还是需求、验收（用原文，没有就补一条）、改了哪些文件、跑了哪些命令以及结果。

改动跨多个功能，或改了对外接口、权限、数据结构时，摘要第一行写 `<!-- auto-dev-hold -->`。
