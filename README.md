# Production Release

`production-release` 是一个面向 Codex 的端到端生产发布 Skill。用户只需发出一次发布请求，它会沿着目标仓库已有规则完成版本判断、发布准备、质量门禁、Git 审查流程、制品发布与消费端验证。

当前正式版本：`v0.1.0`

```text
发布请求
  → 发现仓库规则与发布边界
  → 解析版本、渠道和停止点
  → 更新版本、文档与发布说明
  → 执行可信质量门禁
  → branch / commit / PR / CI / merge
  → tag / registry / hosting release
  → 从真实消费端验证
```

## 正式交付形态

本项目交付为一个独立的 [Agent Skill](https://agentskills.io/specification) 目录：

- `SKILL.md` 是唯一执行入口，定义触发条件、发布流程、安全边界和完成标准。
- `agents/openai.yaml` 提供 Codex 界面名称、简述和默认调用提示。
- 项目不包含守护进程、运行时服务或需要编译的可执行文件，也没有自身的生产依赖。
- 版本以不可变 Git tag 和 GitHub Release 为准；源代码归档就是可分发制品。
- 实际发布时使用目标仓库已经配置的 Git、CI、包管理器、registry 和托管平台能力。

## 实际实现方式

Codex 启动时读取 Skill 的名称和描述；当发布请求匹配或用户显式调用 `$production-release` 时，再加载完整的 `SKILL.md`。Skill 本身不替代 GitHub、npm、PyPI、Cargo 或容器 registry，而是负责发现并编排这些系统已有的工具与规则。

对于 GitHub 交付：

- 分支、暂存、提交和推送由本地 Git 完成。
- PR 查询、创建和状态跟踪优先使用 GitHub connector，能力不足时才回退到 `gh`。
- 适合本次发布的现有非默认分支会被复用，否则从最新远程默认分支创建发布分支。
- 推送后核对远程分支指向已验证的 commit，并先查询同一 head/base 是否已经存在 PR。
- fork 和跨仓库 PR 使用能够明确表达源仓库、head 与 base 的工具，不依赖猜测。
- PR 默认先以 draft 创建，CI 通过后才进入 ready 和 merge；完整发布还会继续完成 tag、Release、制品和消费端验证。

## 安装

### 个人范围

Linux、macOS 或 WSL：

```bash
mkdir -p "$HOME/.agents/skills"
git clone https://github.com/CaoLiangqiang/Xiaocao-Production-Release.git \
  "$HOME/.agents/skills/production-release"
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force "$HOME\.agents\skills" | Out-Null
git clone https://github.com/CaoLiangqiang/Xiaocao-Production-Release.git `
  "$HOME\.agents\skills\production-release"
```

若要锁定可复现版本，在安装后执行：

```bash
git -C "$HOME/.agents/skills/production-release" checkout v0.1.0
```

### 仓库范围

团队也可以把目录放在目标仓库的 `.agents/skills/production-release` 下，使该 Skill 只对该仓库生效。若使用 Git submodule：

```bash
git submodule add \
  https://github.com/CaoLiangqiang/Xiaocao-Production-Release.git \
  .agents/skills/production-release
```

Codex 通常会自动发现 Skill；如果没有出现，重启 Codex。可以用 `/skills` 查看，或直接以 `$production-release` 显式调用。

## 运行要求

- 支持 Agent Skills 的 Codex 环境。
- 本地 Git，以及目标远程仓库所需的认证权限。
- 目标仓库声明的语言运行时、包管理器、构建和测试工具。
- 发布到 GitHub 时，建议配置 GitHub connector；需要回退到 CLI 时再安装并登录 `gh`。
- 发布 registry 制品时，需要目标项目已有的发布身份或可信发布配置。

Skill 不会要求所有目标项目安装同一套工具；它只使用仓库证据能够证明适用的检查和发布命令。

## 使用方式

完整发布下一个版本：

```text
$production-release 发布这个仓库的下一个生产版本。
```

只准备发布 PR：

```text
$production-release 准备下一个 patch 版本的 release PR，停在 draft PR。
```

恢复中断的发布：

```text
$production-release npm publish 刚才超时了。检查外部状态，安全地继续，不要重复上传。
```

明确说“发布、发版、publish、ship”时，Skill 会以完成交付为目标持续执行；只要求检查、准备或创建草稿时，它会停在指定阶段。

## 工作原则

- 先读取仓库说明、版本来源、CI 和发布自动化，再决定操作方式。
- 在第一次修改前说明已解析的版本、来源、渠道、检查项和停止点。
- 审查、暂存、打包和发布范围保持一致，不静默包含无关文件。
- 验证命令来自可信仓库证据；找不到时明确标记为未验证，不自行编造。
- 不绕过失败的必需检查、分支保护或强制人工审批。
- 不覆盖已有版本或标签；远端响应不明确时先查询状态，再决定是否重试。
- 上传成功不等于发布完成，必须从真实 registry 或全新消费环境再次验证。

## 与 git-ship 的取舍

本项目参考了 [`oil-oil/git-ship`](https://github.com/oil-oil/git-ship) 在触发边界、执行预览、验证命令发现和失败说明方面的设计：

| | `git-ship` | `production-release` |
| --- | --- | --- |
| 主要目标 | 把工作区改动通过分支、PR 和 squash merge 送进 `main` | 完成跨 Git、托管平台和制品仓库的生产发布 |
| 范围 | GitHub Git 交付 | npm、PyPI、Cargo、容器、GitHub Release 等 |
| 策略 | 固定、明确的 Git 工作流 | 优先遵循目标仓库已有发布策略和自动化 |
| 完成标准 | PR 已合并并回到最新 `main` | 制品已发布，并从真实消费者环境验证成功 |

项目吸收它的清晰度和安全提示，但不固定使用 `main`、`origin`、stash、squash merge 或 `git add -A`。它也吸收了 `github:yeet` 的本地 Git 与 GitHub connector 分工、draft PR、防重复和 PR 描述规范，但不会强制依赖 `gh`，也不会在 PR 创建后停止。

## 项目结构

```text
production-release/
├── .github/workflows/validate.yml # Agent Skill 结构 CI 校验
├── agents/openai.yaml             # Codex 界面信息和默认提示
├── .gitignore                     # 本地过程文件忽略规则
├── LICENSE                        # MIT 许可证
├── README.md                      # 用户与维护说明
└── SKILL.md                       # Agent 执行规范
```

## 验证与维护

每次 PR 和向 `main` 的推送都会执行 `Validate skill` 工作流，检查目录名称、`SKILL.md` 元数据、正文和 `agents/openai.yaml` 的关键字段。正式发布还会在临时目录中验证 Git 归档的文件清单和独立安装，不依赖当前工作区路径或缓存。

修改发布行为时，请保持改动聚焦，并同时说明成功路径、失败路径和恢复方式。不要为单一仓库硬编码分支名、合并策略、包管理器或托管平台；目标仓库的规则始终优先。

本项目采用 [MIT License](LICENSE)。
