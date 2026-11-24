# GD-SaiKen Auto Dev

组织级 **SDD 先行**自动开发：**Issue → 规格 → 评审 → 实施计划 → 计划评审 → 手动 auto-develop** → 验收评审 → 合并。全程 **dev-standards**；CI 失败自动修复。详见 **[docs/流水线全流程.md](docs/流水线全流程.md)**。

本仓库是唯一需要维护完整实现的地方。其他仓库只需复制 5 个薄 Workflow，即可复用中心的 Prompt、`dsh` 安装 Action、开发/评审逻辑和安全闸口。

## 仓库内容

- `.github/workflows/auto-*.yml`：`workflow_call` 可复用实现；
- `.github/prompts/`：方案、开发、评审、修复 Prompt；
- `.github/actions/setup-dsh/`：Node.js 22 与 `@deepseek-ai/dsh` 安装；
- `.github/actions/run-prompt/`：读取中心 Prompt，经 `dsh --profile headless` 执行；
- `.github/dsh/globalsino.patch.yml`：默认第三方模型商（Globalsino）与 `deepseek-flash`；
- `examples/consumer/.github/workflows/`：业务仓库接入模板。

## 业务仓库接入

1. 将 `examples/consumer/.github/workflows/` 下全部 `auto-*.yml`（含 `auto-fix-ci.yml`）及 `ci.yml` 复制到目标仓库的 `.github/workflows/`。**文件名必须保持不变**，阶段间通过文件名显式调度。`ci` 仅在 **pull_request** 上运行，不向 `main` 直推触发，避免误调度 LLM 修复。
2. 在目标仓库 **Settings → Secrets and variables → Actions** 配置 Secret：
   - `LLM_API_KEY`：**Globalsino（或兼容）API Key**，运行时映射为 `GLOBALSINO_API_KEY`。
3. 可选 Variables：
   - `LLM_MODEL`：默认 `deepseek-flash`；
   - `LLM_BASE_URL`：默认 `https://aikey.globalsino.com.cn`；
   - `DSH_VERSION`：`@deepseek-ai/dsh` 版本，默认 `latest`，生产建议固定版本。
4. SDD 标签（评审自动打）：`spec-reviewed`、`plan-reviewed`；`auto-develop` 要求 Issue 已有 `plan-reviewed`。
5. 新建 Issue 立项；`proposals/*.md` 可选；**开发**仅在 Actions 手动跑 `auto-develop`。写法见 [docs/怎么提Issue.md](docs/怎么提Issue.md)。

调用方通过显式 Secret 映射授权中心 Workflow 使用模型 Key；Key 不保存在本中心仓库。

## 使用流程（八步）

1. **Issue**（不触发开发）
2. PR：`docs/SDD/规格文档/` → `ci` + `auto-review`（规格评审）→ `spec-reviewed`
3. PR：`docs/SDD/实施计划/` → `ci` + `auto-review`（计划评审）→ `plan-reviewed`
4. 手动 **auto-develop**（带 Issue 号）
5. `auto/issue-<n>` PR → `ci` + **验收评审**（对照 SDD）
6. **auto-merge**（`auto/` 且 Approve 且检查绿）
7. **auto-fix-ci**：仅 `auto/*` 开放 PR 上失败的 `ci` → **auto-fix**（`source=ci`）；评审打回在 `auto/` 上同样自动修，最多 3 轮
8. Prompt 均前置 [agent-contract.md](.github/prompts/agent-contract.md)（dev-standards 摘录）

## 安全控制

- 业务仓库 Token 和 Secret 不转移到中心仓库；
- 禁止提交对 `.github/` 的 Agent 修改；
- fork PR 不运行带模型凭据的评审；
- 评审 verdict 与 findings 严重级别一致；
- CI 自动修复仅适用于 **`auto/*` + pull_request** 触发的失败 `ci`；SDD / 人工 PR 的 CI 红需人工处理；评审触发的修复与自动合并仍限 `auto/`；
- 仅机器人对当前 head SHA 的 Approved 评审可合并，合并 API 再次校验 SHA。

## 版本引用

模板默认引用 `@main`，便于组织内直接接入和统一更新。生产环境建议将业务仓库中的引用改为固定 release tag 或 commit SHA，以获得不可变版本；GitHub 官方建议 commit SHA 的稳定性和安全性最高。

## 验证边界

Workflow 的静态结构可以在本地验证；真实方案生成、开发和评审需要目标仓库配置有效的 `LLM_API_KEY`（Globalsino Key）。