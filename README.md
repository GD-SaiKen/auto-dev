# GD-SaiKen Auto Dev

组织级 AI 自动开发流水线：开 Issue 即开发 → Kimi Code 改代码 → 自动评审 → 检查通过则合并。全流程说明见 **[docs/流水线全流程.md](docs/流水线全流程.md)**（含业务示例与 Agent 工作说明）。

本仓库是唯一需要维护完整实现的地方。其他仓库只需复制 5 个薄 Workflow，即可复用中心的 Prompt、Kimi Code CLI 安装 Action、开发/评审逻辑和安全闸口。

## 仓库内容

- `.github/workflows/auto-*.yml`：`workflow_call` 可复用实现；
- `.github/prompts/`：方案、开发、评审、修复 Prompt；
- `.github/actions/setup-kimi/`：Node.js 22 与 Kimi Code CLI 安装；
- `.github/actions/run-prompt/`：通过 `github.action_path` 读取中心 Prompt 并执行；
- `examples/consumer/.github/workflows/`：业务仓库接入模板。

## 业务仓库接入

1. 将 `examples/consumer/.github/workflows/` 下全部 `auto-*.yml`（含 `auto-fix-ci.yml`）原样复制到目标仓库的 `.github/workflows/`。**文件名必须保持不变**，阶段间通过文件名显式调度。
2. 在目标仓库 **Settings → Secrets and variables → Actions** 配置 Secret：
   - `LLM_API_KEY`：Kimi Code API Key，必需。
3. 可选 Variables：
   - `LLM_MODEL`：默认 `kimi-for-coding`；
   - `LLM_BASE_URL`：默认 `https://api.kimi.com/coding/v1`；
   - `KIMI_CLI_VERSION`：默认 `latest`，生产建议固定版本。
4. 可选标签：`proposal`、`plan-ready`、`approved`。日常提交不用打这些标签。
5. 直接新建 Issue，或推送 `proposals/*.md`，即可启动流程。写法见 [docs/怎么提Issue.md](docs/怎么提Issue.md)。Issue 模板在 `examples/consumer/.github/ISSUE_TEMPLATE/`。

调用方通过显式 Secret 映射授权中心 Workflow 使用模型 Key；Key 不保存在本中心仓库。

## 使用流程

1. 新建 Issue 即开始开发。缺陷和建议写法见 [docs/怎么提Issue.md](docs/怎么提Issue.md)，写不全也照样开工。
2. 推送 `proposals/*.md` 时先生成方案评论，再调度开发。
3. 开发推送 `auto/issue-<n>` 分支并创建 PR。改动跨多个功能，或动到对外接口、权限、数据结构时，PR 保持草稿，不自动评审、不自动合并。
4. 其余 PR 自动评审。Approve 且该仓库已有的检查通过后，squash 合并并删除分支。仓库没有检查项时仍会合并。
5. 评审打回后自动修复；`ci` 在 `auto/` 分支 PR 上失败时也会调度自动修复（读失败日志）。合计最多 3 次 `auto: fix` 提交，随后转人工。
6. 写方案、写代码、修代码和评审都会带上 `.github/prompts/agent-contract.md`（dev-standards 摘录，同步日期写在文件头）。仓库外的 Agent 在写方案或代码前读同一份，说明见 [docs/写方案和代码.md](docs/写方案和代码.md)。

## 安全控制

- 业务仓库 Token 和 Secret 不转移到中心仓库；
- 禁止提交对 `.github/` 的 Agent 修改；
- fork PR 不运行带模型凭据的评审；
- 评审 verdict 与 findings 严重级别一致；
- 仅 `auto/` 分支可自动修复/合并；
- 仅机器人对当前 head SHA 的 Approved 评审可合并，合并 API 再次校验 SHA。

## 版本引用

模板默认引用 `@main`，便于组织内直接接入和统一更新。生产环境建议将业务仓库中的引用改为固定 release tag 或 commit SHA，以获得不可变版本；GitHub 官方建议 commit SHA 的稳定性和安全性最高。

## 验证边界

Workflow 的静态结构可以在本地验证；真实方案生成、开发和评审需要目标仓库配置有效的 `LLM_API_KEY`。
