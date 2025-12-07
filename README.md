# MindflowVibeCodingBootstrapStep

## 1. GitHub 仓库 & 基础文件

### 1.1 在 GitHub 上新建仓库

1. GitHub 网页 → `New repository`

2. 选：

   * **名字**：随意，比如 `my-project`
   * **可见性**：Public / Private
   * README 可以选自动生成，也可以后面自己写。

3. 本地克隆：

```bash
git clone git@github.com:YOU/my-project.git
cd my-project
```

---

### 1.2 添加 LICENSE（通用）

在本地新建一个 `LICENSE` 文件，内容可以通过 GitHub 的 “Add file → Add license” 生成，也可以自己复制 MIT/Apache-2.0 文本。

---

### 1.3 添加 `.gitignore`（包含 `.DS_Store`）

在仓库根创建 `.gitignore`，放一些通用规则，例如：

```gitignore
# macOS
.DS_Store

# Node
node_modules/
npm-debug.log*
yarn.lock
pnpm-lock.yaml

# Python (uv / venv)
.venv/
__pycache__/
*.py[cod]
*.pyo

# Rust
target/

# CMake / build artifacts
build/
CMakeCache.txt
CMakeFiles/

# IDE
.vscode/
.idea/
*.iml
```

按你的技术栈再加就行。

首个提交：

```bash
git add .
git commit -m "chore: initial repo with license and gitignore"
```

---

## 2. 语言 / 构建工具初始化（通用时机）

在这一步，你只按**需要的语言**选择执行——顺序上，建议在接 Codex / Spec 工具之前先大致建好语言骨架，这样 AI 扫目录有东西可读。

### 2.1 Node / JS（npm）

如果项目会用 JS/TS：

```bash
npm init -y   # 创建 package.json
# 之后按需：
# npm install <依赖>
```

---

### 2.2 Python（uv）

如果会用 Python + uv：

```bash
uv init  # 或者：uv venv .venv && uv add <依赖>
```

（这里不展开具体 layout，只是说明：**在开始写 Python 前执行即可**。）

---

### 2.3 Rust（cargo）

如果会用 Rust：

```bash
cargo init         # 当前目录初始化
# 或者：cargo new my-project
```

---

### 2.4 C/C++（CMake）

如果会有 C/C++：

1. 在根创建 `CMakeLists.txt`（可先留空骨架）。
2. 需要构建时再：

```bash
cmake -S . -B build
cmake --build build
```

---

## 3. 安装 & 初始化 Codex CLI

### 3.1 全局安装 Codex CLI

任选其一：

```bash
# npm（官网推荐）
npm i -g @openai/codex

# 或者 Homebrew（macOS）
brew install --cask codex
```

### 3.2 登录 & 为项目生成 `.codex` 配置

在终端执行：

```bash
codex
```

* 第一次会提示用 ChatGPT 账号登录并授权。
* 登录后，在**项目根目录**再次执行：

```bash
cd path/to/my-project
codex
```

Codex 会以当前目录为工作区，生成基础配置（`.codex/config.json` 等）。

---

## 4. 初始化 GitHub Spec Kit（speckit）+ Codex 集成

> 这里指的就是 GitHub 官方的 **Spec Kit**（Specify CLI），支持 Codex 作为 AI Agent。

### 4.1 安装 Specify CLI

推荐用 uv 的 “tool install” 模式：

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

（如果只想一次性用，也可以：`uvx --from git+https://github.com/github/spec-kit.git specify init ...`。）

### 4.2 在当前项目里初始化 Spec Kit（绑定 Codex）

在你的项目根目录：

```bash
specify init . --ai codex
# 或：
specify init --here --ai codex
```

效果（通用地说）：

* 创建 `.specify/` 目录，包括：

  * `specs/`：规格
  * `templates/`：模板
  * `scripts/` / `memory/` / `out/` 等（视版本而定）
* 在 `.github/` 下加上一些用于 `/speckit.*` 的提示/配置（不同 Agent 会略有差异，但对 Codex 来说主要是**启用 `/speckit.*` 指令**）。

> 此时，Spec Kit 已经把：
>
> * `/speckit.constitution` / `/speckit.specify` / `/speckit.plan` / `/speckit.tasks` / `/speckit.implement` 等命令挂到 Codex 上。

---

## 5. 初始化 OpenSpec（与 Codex 绑定）

> OpenSpec 是另一个 spec-driven 工具，可以通过 `openspec init` 配置你的仓库，并写自己的一份 `AGENTS.md`。

### 5.1 安装 OpenSpec CLI

```bash
npm install -g @fission-ai/openspec@latest
```

### 5.2 在项目中初始化

```bash
cd path/to/my-project
openspec init
```

* CLI 会让你选使用的 AI 工具，选 **Codex**。
* 初始化后通常会：

  * 创建 `openspec/` 目录，里面有：

    * `openspec/specs/`：当前真相（source-of-truth specs）
    * `openspec/changes/`：变更 / proposal 文件夹
  * 在项目根生成一份 `AGENTS.md`，里面写 OpenSpec 的工作流说明，供 Codex 等 AGENTS.md 兼容工具读取。
  * 对 Codex，OpenSpec 会在 Codex 的 prompts 目录注册 `/openspec-proposal`、`/openspec-apply`、`/openspec-archive` 命令。
好，那就专门把「统一 AGENTS」这块做细——不是抽象讲法，而是**基于 OpenSpec / GitHub Spec Kit 这两个官方仓库的真实行为**，给你一个可以直接照抄的结构和改造步骤。

---

## 1. 这两个工具实际会碰到哪些 AGENT 相关文件？

### 1.1 OpenSpec 会动哪些文件

来自官方 README：

**初始化 `openspec init` 时：**

1. 在项目根写一个 **托管的 `AGENTS.md` stub**

   * 里面是给所有 AI coding agent 的简短指令，大意是：

     * 当对话提到「proposal / spec / change / plan」之类，就去看 `@/openspec/AGENTS.md`
     * 那里有「如何创建/应用变更」、「spec 格式」、「项目结构指南」
   * OpenSpec 自己把这个叫做「managed AGENTS.md hand-off」。

2. 创建 `openspec/` 目录（source of truth）

   * `openspec/specs/**`：当前真相 spec
   * `openspec/changes/**`：proposal / tasks / spec delta
   * 很多示例仓库还会有 `openspec/AGENTS.md` / `openspec/project.md` 作为更长的说明和约定。

> 结论：
>
> * **OpenSpec 一定会写根目录的 `AGENTS.md`（stub）**；
> * 推荐再在 `openspec/AGENTS.md` 里写详细规则，stub 只做跳转。

---

### 1.2 Spec Kit（speckit）会动哪些文件

从官方 README + `update-agent-context.sh` 可以看出来：

**初始化 `specify init` 后：**

1. 创建 Spec Kit 自己的状态目录：

   * `.specify/memory/constitution.md`

     * `/speckit.constitution` 会生成/更新这份文件
     * 这里是 **「项目宪法 / 原则」**，对所有 agent 生效。

   * `.specify/templates/agent-file-template.md`

     * 模板内容大概是：项目名、Active Technologies、Project Structure、Commands、Code Style、Recent Changes（用占位符表示）
     * 之后所有 agent context 文件（包括 `AGENTS.md`）都从这里长出来。

2. 当你运行 `scripts/bash/update-agent-context.sh` 时（很多教程会让你跑）：

   这个脚本会根据 `plan.md` 自动更新一堆文件：

   * 根目录：

     * `AGENTS.md`（`AGENTS_FILE="$REPO_ROOT/AGENTS.md"`）

       * 对于 `codex` / `opencode` / `q` / `amp`，都用这一个文件。
       * 如果不存在，就用 `.specify/templates/agent-file-template.md` 生成；
       * 如果已经存在，只会维护：

         * `## Active Technologies`
         * `## Recent Changes`
           这两个 section，其它内容保持不动。

   * 其它 agent 的 context 文件（不叫 AGENT，但会被各自工具看见）：

     * `CLAUDE.md`
     * `GEMINI.md`
     * `QWEN.md`
     * `CODEBUDDY.md`
     * `.github/copilot-instructions.md`
     * `.cursor/rules/specify-rules.mdc`
     * `.windsurf/rules/specify-rules.md`
     * `.kilocode/rules/specify-rules.md`
     * `.augment/rules/specify-rules.md`
     * `.roo/rules/specify-rules.md`
       ……统统都是从 `agent-file-template.md` 生的。

> 结论：
>
> * **Spec Kit 也会用「根目录 `AGENTS.md`」作为 Codex / opencode 等的 Agent 文件**；
> * 但它对已有文件是「**只改两个 section**（Active Technologies / Recent Changes），不动其它内容」。

---

## 2. 统一策略（核心想法）

我们利用这两点行为：

1. OpenSpec 的根 `AGENTS.md` 只是一段短 stub，可以被当作「OpenSpec 子段落」；
2. Spec Kit 的脚本对已有 `AGENTS.md` **只维护两个 section**，剩下内容你可以完全接管。

**目标：**

* 仓库里只有 **一个顶层 `AGENTS.md`**，你说了算；
* OpenSpec 的 stub 挂在这个文件的某个 section 里，保持原样，方便以后 `openspec update`；
* Spec Kit 的脚本只在这个文件里维护 `## Active Technologies` / `## Recent Changes` 这两块；
* OpenSpec 的详细规则放在 `openspec/AGENTS.md`，Spec Kit 的父规则放在 `.specify/memory/constitution.md`。

---

## 3. 顶层 `AGENTS.md` 的具体结构（可以直接拿去改）

下面是一个**结构化的 AGENTS.md 草稿**，你可以在项目根创建 / 重写为这个结构，然后把 OpenSpec 初始化生成的那段 stub 粘到指定位置。

> 注意：中括号的内容你要自己填，比如项目名。

```md
# AGENTS.md — Global Agent Contract for [PROJECT_NAME]

> 所有在本仓库工作的 AI coding agents（Codex、Gemini CLI、Cursor 等）  
> MUST 把这个文件当作**唯一的顶层规则**。其它说明文件都是它的子配置。

---

## 1. 决策顺序（谁说了算）

当规则有冲突时，优先级依次为：

1. **本文件 `AGENTS.md`**（全局约束）
2. **OpenSpec 规格**  
   - `openspec/specs/**`：当前真相  
   - `openspec/changes/**`：proposal / tasks / deltas  
   - 详细工作流：`openspec/AGENTS.md`
3. **Spec Kit 规格**  
   - `.specify/memory/constitution.md`：项目宪法 / 原则  
   - `.specify/specs/**`、feature 目录下的 `spec.md` / `plan.md` / `tasks.md`
4. **工具自己的规则文件**  
   - `CLAUDE.md` / `GEMINI.md` / `.cursor/rules/specify-rules.mdc` 等  
   - 只能在不与 1–3 冲突的前提下补充细节。

---

## 2. 通用行为约束（所有 agent 必须遵守）

1. 在做**非 trivial 改动**之前（新特性、重构、风险操作）：
   - 先检查是否已有相关 spec（OpenSpec / Spec Kit）；
   - 找不到时，先帮我补 spec，而不是直接写代码。
2. 发现需求 /历史描述前后矛盾时：
   - **先质询**，给出冲突列表和建议解决方案，再继续实现。
3. 新能力 / 大改动：
   - 优先通过 **OpenSpec change workflow**（proposal → review → apply → archive）。
4. 任何自动生成的大块代码：
   - 同步更新相关 spec（OpenSpec 或 Spec Kit），否则当作不完整实现。

---

## 3. OpenSpec 集成（监督者）

1. 当对话满足下面任一情况时，必须先进入 OpenSpec 工作流：
   - 出现「proposal / change / spec / plan」等关键词；
   - 引入新的大能力、重大架构调整、性能 / 安全影响大的改动；
   - 场景描述模糊、你无法确定 expected behavior。
2. 具体动作：
   - 用 `/openspec-proposal` 创建 / 定位变更；
   - 用 `/openspec-apply` 按 `tasks.md` 实现；
   - 用 `/openspec-archive` 合并 spec delta。
3. 详细规则：
   - 当你需要更细节的 OpenSpec 说明时，打开 `@/openspec/AGENTS.md`。

> 总结：OpenSpec 负责「**这件事到底要怎么改、spec 怎么更新**」。

---

## 4. Spec Kit（speckit）集成（spec 辅助）

Spec Kit 只做「**帮我把想法整理成 spec / plan / tasks**」，不直接当顶层宪法。

1. 当我在对话里头脑风暴、描述未来愿景、架构设想时，Codex 应该优先调用：
   - `/speckit.constitution`：整理和更新项目原则（写入 `.specify/memory/constitution.md`）
   - `/speckit.specify`：把需求整理成 feature spec
   - `/speckit.plan`：根据 tech stack 出实现方案
   - `/speckit.tasks`：拆成任务（避免你自己手动管理依赖地狱）
2. Spec Kit 生成的 spec / plan / tasks：
   - 如果是新能力，最后必须通过 OpenSpec change（proposal / apply / archive）落到 `openspec/specs/**`。
3. 当两边信息冲突时：
   - 以 **OpenSpec specs** 为准，Spec Kit 负责被修正。

---

## 5. 文件映射（给 agent 的精确路径）

- **OpenSpec：**
  - 源：`openspec/specs/**`
  - 变更：`openspec/changes/**`
  - 项目约定：`openspec/project.md`
  - 详细工作流：`openspec/AGENTS.md`
- **Spec Kit：**
  - 宪法 / 原则：`.specify/memory/constitution.md`
  - agent 模板（不要乱删）：`.specify/templates/agent-file-template.md`
  - feature spec / plan / tasks：按 Spec Kit 默认目录。
- **工具级 context（只补充，不 override 本文件）：**
  - `CLAUDE.md` / `GEMINI.md` / `QWEN.md` / `CODEBUDDY.md`
  - `.github/copilot-instructions.md`
  - `.cursor/rules/specify-rules.mdc`
  - `.windsurf/rules/specify-rules.md`
  - `.kilocode/rules/specify-rules.md`
  - `.augment/rules/specify-rules.md`
  - `.roo/rules/specify-rules.md`

---

## 6. Active Technologies / Recent Changes（由 Spec Kit 自动维护）

> 这两个 section 专门留给 `scripts/bash/update-agent-context.sh` 来更新。  
> 你可以手动删掉单行，但不要删掉整个标题。

## Active Technologies

- （第一次可以留空，跑完 `update-agent-context.sh codex` 后这里会自动填）

## Recent Changes

- （同上，这里会被脚本写入最近几个 feature / 分支）

---

## 7. OpenSpec 托管块（原样保留）

> 下面这一段是 `openspec init` 在根目录生成的「OpenSpec Instructions」stub。  
> 你只需要把它**整体剪切到这个 section 下面**，不要改文字内容，方便以后 `openspec update` 刷新。

<!-- OPENSPEC-MANAGED: DO NOT EDIT MANUALLY -->

[在这里粘贴你当前项目的 OpenSpec AGENTS stub 原文]

<!-- END-OPENSPEC-MANAGED -->
```

这样：

* 对 Codex / 其它识别 `AGENTS.md` 的 agent 来说，这就是唯一顶层规则；
* Spec Kit 脚本只会看见 `## Active Technologies` / `## Recent Changes`，只改这两块；
* OpenSpec 的 stub 依然完整存在，不影响之后 CLI 的更新。

---

## 4. 实际改造步骤（新项目 & 旧项目都适用）

### 4.1 新项目建议顺序（保证不会互相覆盖）

1. **先跑 OpenSpec：**

```bash
openspec init
```

* 这一步会创建：

  * 根 `AGENTS.md`（stub）
  * `openspec/**` 目录树。

2. **再跑 Spec Kit：**

```bash
specify init . --ai codex
# 初始化 .specify/*
```

（这一步不会强制创建 AGENTS.md，真正写 AGENTS 的是下一步的脚本。）

3. **让 Spec Kit 生成 codex 的上下文 section：**

```bash
./scripts/bash/update-agent-context.sh codex
```

* 因为根 `AGENTS.md` 已经存在（来自 OpenSpec），脚本会走「更新已有文件」分支：

  * 如果文件里还没有 `## Active Technologies` / `## Recent Changes`，就在末尾新增这两个 section；
  * 不会改你的其它内容（包括你手动写的 1~5 章和 OpenSpec stub）。

4. **最后，按照上面的模板重写/整理 `AGENTS.md`：**

* 把 OpenSpec stub 整段剪下来，放入 `## 7. OpenSpec 托管块` 下面；
* 在前面补上 1~6 章的内容（可以先照抄模板，再慢慢改细节）。

---

### 4.2 旧项目（已经很脏、有多份 AGENTS）的清理流程

1. **先把所有 AGENT 文件找出来（本地执行）：**

```bash
# 找所有 AGENT(S).md
find . -maxdepth 4 -iname 'AGENT*.md'

# 看看有没有多份 OpenSpec / Spec Kit 的说明
ls openspec/AGENTS.md .specify/memory/constitution.md 2>/dev/null || true
```

典型情况你会看到：

* 根：`AGENTS.md`（可能被 OpenSpec / Spec Kit 都写过）
* `openspec/AGENTS.md`（如果你之前按示例项目建过）
* `.specify/memory/constitution.md`

2. **决定谁是「唯一的顶层」：**

* 顶层：**保留根 `AGENTS.md`**，并按上面的模板改写；
* OpenSpec / Spec Kit 各自的说明文件都变成「子配置」：

  * OpenSpec：`openspec/AGENTS.md` / `openspec/project.md`
  * Spec Kit：`.specify/memory/constitution.md`。

3. **如果你有其它乱七八糟的 AGENT/AGENTS 文件：**

* 比如：

  * `openspec/AGENTS.md` + 根也有一份自写 AGENTS；
  * 或者你自己另外建了 `AGENTS.speckit.md` 之类。

可以这样处理：

```bash
# 示例：把多余的说明收拢为子配置
mv AGENTS.speckit.md .specify/AGENTS.speckit.md   # 或者 docs/agents-speckit.md
mv openspec/AGENTS-old.md openspec/AGENTS.legacy.md
```

然后在顶层 `AGENTS.md` 的「5. 文件映射」里加一句：

```md
- Spec Kit 额外说明（可选）：`.specify/AGENTS.speckit.md`
- OpenSpec 历史说明（只读）：`openspec/AGENTS.legacy.md`
```

4. **确保 Spec Kit 只改它该管的两块：**

只要你：

* 保持 `AGENTS.md` 里 **存在** `## Active Technologies` / `## Recent Changes` 这两个标题；
* 不删掉这些标题本身；

`update-agent-context.sh` 就只会往这两个 section 里插 / 更新条目，不会碰你写的其它内容。

5. **以后新增/更新规则的习惯：**

* 所有「全局行为约束」都只写在根 `AGENTS.md`；
* OpenSpec 的细节写在 `openspec/AGENTS.md` / specs 内，保证它是「规格监督者」；
* Spec Kit 的流程和原则写在 `.specify/memory/constitution.md` / `.specify/specs/**`，只当「spec 生产工具」，不抢「全局话语权」。
