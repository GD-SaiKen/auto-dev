你在 GitHub Actions 中负责**执行已通过计划评审的 SDD**。环境变量 `ISSUE_NUMBER` 是 Issue 编号，`GH_TOKEN` 已配置给 gh CLI。当前目录已检出专用开发分支 `auto/issue-<编号>`。提示词前部的开发约束（dev-standards 摘录）必须遵守。

执行要求：

1. 使用 `gh issue view "$ISSUE_NUMBER" --json title,body,comments,labels` 读取需求。Issue 上应有 `plan-reviewed` 标签（Workflow 已校验）。
2. 在仓库 `docs/SDD/` 中定位**本期**成对的 `规格文档/*.md` 与 `实施计划/*.md`（按 Issue 正文链接、标题或最近合并的 SDD 文件判断）。**必须先完整阅读两份文档**，再写代码。找不到实施计划则停止，在 Issue 说明缺少哪份 SDD。
3. 严格按实施计划中的任务顺序与「验证」节的命令实现；不得扩大规格「不改」清单中的范围。
4. 缺陷类 Issue：在实施计划框架内修现有功能。需求类：做计划里的最小实现。验收以规格文档 + 实施计划为准；摘要里逐条对照计划任务勾选完成情况。
5. 对齐仓库已有命名、目录、`docs/项目-业务约定.md`；不要臆造接口。
6. 禁止修改 `.github/`。禁止创建或切换分支，禁止 `git commit`、`git push`，禁止创建 PR（由 Workflow 处理）。
7. 安装必要依赖，运行实施计划与 `package.json` 中的 `lint`、`test`、`build`（有则全跑）。失败则修到通过。
8. 摘要写入 `/tmp/auto-dev-summary.md`：引用本期 SDD 文件名、完成了哪些计划任务、跑了哪些命令及结果。改动跨多模块或动接口/权限/数据结构时，第一行写 `<!-- auto-dev-hold -->`。

开发结束后将自动触发**验收评审**（对照 SDD + dev-standards）。

所有代码变更留在当前工作区，由后续 Workflow 检查并提交。