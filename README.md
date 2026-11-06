# GD-SaiKen Auto Dev

组织级 AI 自动开发流水线：Issue/方案文档 → 人工审核 → Kimi Code 开发 → 自动评审 → 自动修复或合并。

本仓库是唯一需要维护完整实现的地方。其他仓库只需复制 5 个薄 Workflow，即可复用中心的 Prompt、Kimi Code CLI 安装 Action、开发/评审逻辑和安全闸口。

## 仓库内容

- `.github/workflows/auto-*.yml`：`workflow_call` 可复用实现；
- `.github/prompts/`：方案、开发、评审、修复 Prompt；
- `.github/actions/setup-kimi/`：Node.js 22 与 Kimi Code CLI 安装；
- `.github/actions/run-prompt/`：通过 `github.action_path` 读取中心 Prompt 并执行；
- `examples/consumer/.github/workflows/`：业务仓库接入模板。

## 业务仓库接入

1. 将 `examples/consumer/.github/workflows/` 下 5 个 `auto-*.yml` 原样复制到目标仓库的 `.github/workflows/`。**文件名必须保持不变**，阶段间通过文件名显式调度。
2. 在目标仓库 **Settings → Secrets and variables → Actions** 配置 Secret：
   - `LLM_API_KEY`：Kimi Code API Key，必需。
3. 可选 Variables：
   - `LLM_MODEL`：默认 `kimi-for-coding`；
   - `LLM_BASE_URL`：默认 `https://api.kimi.com/coding/v1`；
   - `KIMI_CLI_VERSION`：默认 `latest`，生产建议固定版本。
4. 创建标签：`proposal`、`plan-ready`、`approved`。
5. 新建带 `proposal` 标签的 Issue，或推送 `proposals/*.md`，即可启动流程。

调用方通过显式 Secret 映射授权中心 Workflow 使用模型 Key；Key 不保存在本中心仓库。

## 使用流程

1. `proposal` Issue 或方案文档触发自动方案，结果发布为机器人评论；
2. 人工审核后添加 `approved`；
3. 自动开发并推送 `auto/issue-<n>` 分支，创建 PR；
4. 自动评审，Approve 后自动 squash 合并并删除分支；
5. Request changes 后自动修复，最多 3 轮，随后转人工。

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
