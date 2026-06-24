---
name: vibe-coding-init
description: 初始化 vibe coding 项目——初始化 git 仓库、创建角色分配文件、自动创建 5 个 ZCode 角色会话（需求分析/Bug修复/技术设计/代码编写/功能验证）、建立跨会话消息传递机制。当用户提到"初始化项目"、"开始 vibe coding"、"设置开发角色"、"创建角色分配"、"vibe 项目初始化"、"初始化协作结构"时使用。
---

# Vibe Coding 项目初始化

此技能为 vibe coding 项目建立多角色协作结构。核心思路是将开发流程拆分为五个独立角色会话——需求分析（新功能入口）、Bug修复（缺陷修复入口）、技术设计、代码编写、功能验证——通过明确的流水线交接、HANDOFF.md 消息总线和回溯机制保证协作效率。

## 执行流程

### 1. 确认项目信息

- 如果用户未提供项目名称，询问项目名称
- 确认当前工作目录为项目根目录

### 2. 初始化 Git 仓库

检查 `.git` 目录是否存在。若不存在则执行：

```bash
git init
```

若已存在则跳过并告知用户。

### 3. 创建 .vibe 目录

```bash
mkdir -p .vibe
```

该目录存放与 vibe coding 协作流程相关的所有配置文件。

### 4. 创建角色分配文件

创建 `.vibe/ROLES.md`，填入以下模板（将 `{PROJECT_NAME}` 和 `{DATETIME}` 替换为实际值）：

```markdown
# 角色分配表

> **项目**：{PROJECT_NAME}
> **创建时间**：{DATETIME}
> **最后更新**：{DATETIME}

---

## 角色会话

| 角色 | 会话 ID | 职责说明 | 创建时间 |
|------|---------|----------|----------|
| 🎯 需求分析 | *待创建* | 分析用户需求、拆解开发任务、输出产品规格说明（做什么）**— 新功能入口** | — |
| 🐛 Bug修复 | *待创建* | 接收缺陷报告、分析根因、编写修复方案 → `.vibe/BUGFIX.md`。**禁止直接修改代码，只分析不编码。** | — |
| 🏗️ 技术设计 | *待创建* | 架构方案设计、数据模型、接口契约、技术选型（怎么做） | — |
| 💻 代码编写 | *待创建* | 根据技术方案/Bug修复方案编写实现代码，遵循现有代码风格和项目规范 | — |
| ✅ 功能验证 | *待创建* | 从需求视角验收功能 + 审查代码质量，发现缺陷反馈给 Bug修复 | — |

---

## ⚠️ 角色边界铁律

> **各角色只做自己该做的事，严禁越界。**

| 规则 | 说明 |
|------|------|
| 🐛 Bug修复 ≠ 写代码 | Bug修复只分析根因、输出修复方案到 `.vibe/BUGFIX.md`，**禁止直接修改代码文件**。修复实施由 💻 代码编写完成。 |
| 🎯 需求分析 ≠ 设计 | 需求分析只描述"做什么"，不涉及技术实现方案。技术方案由 🏗️ 技术设计完成。 |
| 🏗️ 技术设计 ≠ 写代码 | 技术设计只输出设计文档 `.vibe/DESIGN.md`，不写实现代码。 |
| 💻 代码编写 ≠ 验证 | 代码编写完成后自检（verification-before-completion），但正式验收由 ✅ 功能验证完成。 |
| ✅ 功能验证 ≠ 修复 | 发现 Bug 必须通过 HANDOFF.md 交给 🐛 Bug修复分析，不直接修改代码。 |

**所有跨角色交付必须通过 `.vibe/HANDOFF.md` 消息总线**，不可跳过中间角色。

---

## 交流流程

### 正向流水线（开发推进）

项目有两个入口，根据任务类型选择：

**新功能流**：
```
需求分析 ──规格说明──▶ 技术设计 ──技术方案──▶ 代码编写 ──实现报告──▶ 功能验证 ──✅ 完成
```

**缺陷修复流**：
```
功能验证 ──Bug报告──▶ Bug修复 ──修复方案──▶ 代码编写 ──修复报告──▶ 功能验证 ──✅ 关闭
                         │
                         └──(复杂Bug需架构变更)──▶ 技术设计 ──▶ 代码编写
```

| 步骤 | 发起方 | 接收方 | 传递内容 |
|------|--------|--------|----------|
| 1a | 🎯 需求分析 | 🏗️ 技术设计 | 产品规格说明（`.vibe/SPEC.md`） |
| 1b | ✅ 功能验证 | 🐛 Bug修复 | Bug 报告（`.vibe/BUGFIX.md` 中的问题描述） |
| 2a | 🏗️ 技术设计 | 💻 代码编写 | 技术设计方案（`.vibe/DESIGN.md`） |
| 2b | 🐛 Bug修复 | 💻 代码编写 | 修复方案（`.vibe/BUGFIX.md` 中的根因分析和修复步骤） |
| 3 | 💻 代码编写 | ✅ 功能验证 | 实现/修复完成通知 + 变更文件列表 |
| 4 | ✅ 功能验证 | — | 验证报告（通过 / 问题清单） |

### 反向回溯（问题驱动）

```
功能验证 ──Bug报告──▶ Bug修复 ──修复方案──▶ 代码编写
功能验证 ──需求偏差──▶ 需求分析
代码编写 ──设计质疑──▶ 技术设计
技术设计 ──需求质疑──▶ 需求分析
Bug修复 ──架构变更──▶ 技术设计
```

| 场景 | 发起角色 | 目标角色 | 触发条件 | 传递内容 |
|------|----------|----------|----------|----------|
| 🐛 发现 Bug | ✅ 功能验证 | 🐛 Bug修复 | 验证发现功能异常或回归问题 | Bug 描述 + 复现步骤 + 期望行为 |
| 🔧 修复不完整 | ✅ 功能验证 | 🐛 Bug修复 | 修复后验证仍存在问题 | 残留问题描述 |
| 🐛 实现缺陷 | ✅ 功能验证 | 💻 代码编写 | 验证发现实现与修复方案不符 | 具体问题描述 + 复现步骤 |
| 🔄 需求偏差 | ✅ 功能验证 | 🎯 需求分析 | 实现虽符合技术方案但偏离原始需求意图 | 偏差说明 + 影响分析 |
| 🚫 设计不可行 | 💻 代码编写 | 🏗️ 技术设计 | 技术方案在当前约束下无法落地 | 技术分析 + 替代建议 |
| ❓ 规格不明确 | 🏗️ 技术设计 | 🎯 需求分析 | 设计时发现需求有歧义或缺失关键约束 | 具体疑问点列表 |
| 🏗️ Bug需架构变更 | 🐛 Bug修复 | 🏗️ 技术设计 | 修复需要涉及架构层面的调整 | 根因分析 + 影响范围 |

---

## Superpowers 集成

> **核心原则**：Superpowers 是角色的武器，不是角色的替代。每个角色在自己的职责范围内使用对应的 Superpower 提升效率。多角色多会话体系不受影响——Superpowers 在每个会话内独立生效。

### 角色 × Superpower 映射

| Superpower | 🎯 需求分析 | 🐛 Bug修复 | 🏗️ 技术设计 | 💻 代码编写 | ✅ 功能验证 | 使用场景 |
|---|---|---|---|---|---|---|
| `brainstorming` | ✅ | ✅ | ✅ | — | — | 需求探索、根因分析、方案头脑风暴 |
| `writing-plans` | ✅ | ✅ | ✅ | — | — | 输出可执行的规格/修复/设计计划 |
| `systematic-debugging` | — | ✅ | — | ✅ | ✅ | Bug 根因定位、验证失败时分析 |
| `test-driven-development` | — | — | — | ✅ | — | 编写功能代码/修复代码前，先写测试 |
| `subagent-driven-development` | — | — | — | ✅ | — | 执行实施计划中的独立任务 |
| `dispatching-parallel-agents` | — | — | — | ✅ | — | 多个无依赖任务并行推进 |
| `using-git-worktrees` | — | — | — | ✅ | — | 隔离 feature/fix 开发环境 |
| `verification-before-completion` | — | ✅ | — | ✅ | ✅ | 修复验证 / 提交前验证 / 验证结论前确认 |
| `requesting-code-review` | — | — | — | ✅ | — | 功能完成后提交代码审查 |
| `receiving-code-review` | — | — | — | ✅ | — | 收到审查反馈后实施修改 |
| `finishing-a-development-branch` | — | — | — | ✅ | — | 合并 / PR / 清理分支 |

### 正向流水线中的触发点

```
新功能流：
需求分析                       技术设计                       代码编写                       功能验证
   │                              │                              │                              │
   ├─ brainstorming               ├─ brainstorming               ├─ TDD                        ├─ verification
   ├─ writing-plans               ├─ writing-plans               ├─ subagent-driven            └─ systematic-debugging
   └─ 产出 .vibe/SPEC.md          └─ 产出 .vibe/DESIGN.md        ├─ dispatching-parallel
                                                                 ├─ git-worktrees
                                                                 ├─ code-review
                                                                 └─ finishing-branch

缺陷修复流：
功能验证 ──Bug报告──▶ Bug修复 ──修复方案──▶ 代码编写 ──修复报告──▶ 功能验证
                          │
                          ├─ systematic-debugging
                          ├─ brainstorming
                          └─ 产出 .vibe/BUGFIX.md
```

### 反向回溯中的触发点

| 回溯场景 | 被调角色 | 应使用的 Superpower |
|---|---|---|
| 🐛 验证发现 Bug → Bug修复 | 🐛 Bug修复 | `systematic-debugging` 定位根因 |
| 🔧 修复后验证失败 → Bug修复 | 🐛 Bug修复 | `systematic-debugging` 重新分析 |
| 🐛 实现缺陷 → 代码编写 | 💻 代码编写 | `systematic-debugging` 定位根因 |
| 🔄 验证发现需求偏差 → 需求分析 | 🎯 需求分析 | `brainstorming` 重新对齐需求 |
| 🚫 编码发现设计不可行 → 技术设计 | 🏗️ 技术设计 | `brainstorming` 探索替代方案 |
| ❓ 设计发现规格不明确 → 需求分析 | 🎯 需求分析 | `brainstorming` 澄清约束条件 |
| 🏗️ Bug需架构变更 → 技术设计 | 🏗️ 技术设计 | `brainstorming` 评估架构影响 |

---

## 使用指南

### 如何启动角色会话

每个角色会话启动后，第一件事是读取协作文件：

> **启动时的标准操作**：
> 1. 读取 `.vibe/ROLES.md` — 了解自身职责和协作规则
> 2. 读取 `.vibe/HANDOFF.md` — 查看上游角色是否已交付
> 3. 如果有上游交付物，读取对应文件（如 `.vibe/SPEC.md`、`.vibe/DESIGN.md`）
> 4. 如果需要更多上下文，使用 `ReadSessionContext` 读取上游角色的会话记录

**自动创建时的提示词**：

1. **启动需求分析会话**：
   ```
   你是 [项目名] 的需求分析角色。先读取 .vibe/ROLES.md 和 .vibe/HANDOFF.md。
   
   ✅ 你做的事：
   - 与用户沟通需求，使用 brainstorming 探索意图
   - 拆解开发任务，编写可执行的产品规格说明 → .vibe/SPEC.md
   
   ❌ 你不做的事：
   - 不做技术设计（怎么实现交给技术设计）
   - 不写代码、不架构设计
   
   📤 完成后通知下游：
   在 .vibe/HANDOFF.md 追加：
   "[需求分析 → 技术设计]：SPEC.md 已完成，请接手设计。关键需求：[简述核心功能点]"
   
   Superpowers：brainstorming、writing-plans
   ```

2. **启动 Bug修复会话**：
   ```
   你是 [项目名] 的 Bug修复角色。先读取 .vibe/ROLES.md 和 .vibe/HANDOFF.md。
   
   ⚠️ 铁律：你只分析 Bug，一行代码都不改。
   
   ✅ 你做的事：
   1. 接收 Bug 报告（含复现步骤 + 期望行为）
   2. 使用 systematic-debugging 定位根因（哪个文件、哪个函数、哪行逻辑出错）
   3. 编写修复方案 → .vibe/BUGFIX.md（根因 + 修复建议 + 涉及文件清单）
   
   ❌ 你不做的事：
   - 不修改任何代码文件
   - 不验证修复效果（这是功能验证的事）
   
   📤 完成后通知下游：
   - 一般 Bug：追加 "[Bug修复 → 代码编写]：BUGFIX.md 已完成。涉及文件：[列表]"
   - 涉及架构变更：追加 "[Bug修复 → 技术设计]：此 Bug 需架构调整，详见 BUGFIX.md"
   
   Superpowers：systematic-debugging、brainstorming
   ```

3. **启动技术设计会话**：
   ```
   你是 [项目名] 的技术设计角色。先读取 .vibe/ROLES.md 和 .vibe/HANDOFF.md。
   新功能时读取 .vibe/SPEC.md；Bug 架构变更时读取 .vibe/BUGFIX.md。
   
   ✅ 你做的事：
   - 架构方案设计、数据模型定义、接口契约制定
   - 输出结构化技术方案 → .vibe/DESIGN.md
   
   ❌ 你不做的事：
   - 不写实现代码
   - 不分析需求（需求不明确时退回给需求分析）
   
   📤 完成后通知下游：
   追加 "[技术设计 → 代码编写]：DESIGN.md 已完成，核心方案：[简述]"
   
   ⏪ 遇到问题回溯：
   如需求不明确：追加 "[技术设计 → 需求分析]：请澄清 [具体问题]"
   
   Superpowers：brainstorming、writing-plans
   ```

4. **启动代码编写会话**：
   ```
   你是 [项目名] 的代码编写角色。先读取 .vibe/ROLES.md 和 .vibe/HANDOFF.md。
   新功能时读取 .vibe/DESIGN.md；Bug修复时读取 .vibe/BUGFIX.md。
   
   ✅ 你做的事：
   - 根据上游方案编写/修复代码
   - 使用 TDD 先写测试再编码
   - 使用 verification-before-completion 自检
   
   ❌ 你不做的事：
   - 不做需求分析（需求问题找需求分析）
   - 不做架构设计（方案问题找技术设计）
   - 不做最终验收（交给功能验证）
   
   📤 完成后通知下游：
   追加 "[代码编写 → 功能验证]：实现/修复完成。变更文件：[列表]。自检结果：[通过/有已知问题]"
   
   ⏪ 遇到问题回溯：
   方案不可行：追加 "[代码编写 → 技术设计]：[具体问题]，请调整方案"
   自测发现新 Bug：追加 "[代码编写 → Bug修复]：[描述]，请分析"
   
   Superpowers：test-driven-development、subagent-driven-development、dispatching-parallel-agents、verification-before-completion
   ```

5. **启动功能验证会话**：
   ```
   你是 [项目名] 的功能验证角色。先读取 .vibe/ROLES.md、.vibe/HANDOFF.md 和 .vibe/SPEC.md。
   
   ✅ 你做的事：
   - 按 SPEC.md 逐项验收功能
   - 审查代码质量
   - 输出验证报告
   
   ❌ 你不做的事：
   - 不修改代码（发现 Bug 交给 Bug修复，不是自己修）
   - 不重新定义需求（需求偏差交给需求分析）
   
   📤 完成后通知下游：
   - 全部通过：追加 "[功能验证 → 全员]：✅ 验证通过。项目可交付。"
   - 发现 Bug：追加 "[功能验证 → Bug修复]：Bug：[描述] | 复现步骤：[...] | 期望行为：[...]"
   - 需求偏差：追加 "[功能验证 → 需求分析]：[偏差描述]"
   
   ⚠️ Bug 报告必须包含可复现的测试步骤，方便 Bug修复 分析、代码编写 修复后回测。
   
   Superpowers：verification-before-completion、systematic-debugging
   ```

### 更新会话 ID

每次创建新的角色会话后，立即更新上方「角色会话」表格：

- 将对应角色的「会话 ID」列填写为实际会话 ID（可在 ZCode 会话信息中获取，格式如 `sess_xxx`）
- 更新「创建时间」列
- 更新文件顶部的「最后更新」时间

### 角色切换信号

在 `.vibe/HANDOFF.md` 中追加消息来实现跨会话通信。格式：

```markdown
| 时间 | 发送方 | 接收方 | 消息 |
|------|--------|--------|------|
| 17:00 | 🎯 需求分析 | 🏗️ 技术设计 | SPEC.md 已完成，请接手设计 |
| 17:30 | 🏗️ 技术设计 | 💻 代码编写 | DESIGN.md 已完成，请开始编码 |
| 18:00 | 💻 代码编写 | ✅ 功能验证 | 实现完成，文件：index.html, app.js, ... |
```

回溯场景的消息示例：
```markdown
| 18:30 | ✅ 功能验证 | 💻 代码编写 | 发现 bug：计时器暂停后无法恢复 |
| 18:45 | 💻 代码编写 | 🏗️ 技术设计 | 方案中接口定义与实际框架不兼容，请调整 |
| 19:00 | 🏗️ 技术设计 | 🎯 需求分析 | 需求第 3 点约束条件不明确，请澄清 |
```

**读取上游会话上下文**（当 HANDOFF.md 消息不够详细时）：
```
ReadSessionContext(sessionId="上游会话ID", query="关于 SPEC.md 的设计决策讨论")
```
- 此工具可拉取其他角色会话中的完整讨论记录
- 避免重复沟通，确保上下游信息一致

---

## 附录：文件结构

初始化完成后，项目目录结构如下：

```
.
├── .vibe/
│   ├── ROLES.md         # 角色分配与交流流程定义
│   ├── HANDOFF.md       # 跨会话消息总线
│   ├── SPEC.md           # 需求分析产出：产品规格说明
│   ├── DESIGN.md         # 技术设计产出：架构/技术方案
│   └── BUGFIX.md         # Bug修复产出：根因分析 + 修复方案
├── .git/                # Git 仓库
└── (项目文件)
```

### 5. 创建跨会话消息文件

创建 `.vibe/HANDOFF.md`，作为会话间的共享消息总线：

```markdown
# 跨会话消息总线

> 各角色会话通过此文件传递交付物和问题。
> 格式：`[发送方 → 接收方]：消息内容`

---

## 消息记录

| 时间 | 发送方 | 接收方 | 消息 |
|------|--------|--------|------|
| — | — | — | 等待需求分析会话启动… |

---

## 会话关联

| 角色 | 会话 ID | 状态 |
|------|---------|------|
| 🎯 需求分析 | *待创建* | ⏳ |
| 🐛 Bug修复 | *待创建* | ⏳ |
| 🏗️ 技术设计 | *待创建* | ⏳ |
| 💻 代码编写 | *待创建* | ⏳ |
| ✅ 功能验证 | *待创建* | ⏳ |
```

**注意**：HANDOFF.md 的「会话关联」表格将随会话创建自动更新，无需手动维护。

### 6. 创建角色会话

此步骤自动创建 5 个 ZCode 角色会话。根据环境选择路径：

#### 路径 A：自动创建（需要 Node.js ≥ 22）

**前置条件**：
1. Node.js ≥ 22（`brew install node@22` 然后 `export PATH="/usr/local/opt/node@22/bin:$PATH"`）
2. `~/.zcode/cli/config.json` 需包含有效的 model provider（从 `~/.zcode/v2/config.json` 复制当前使用的 provider）

**创建命令**（使用 `--mode yolo` 避免权限确认阻塞）：

```bash
ZCODE="/Applications/ZCode.app/Contents/Resources/glm/zcode.cjs"
CWD="$(pwd)"

# 1. 需求分析
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的需求分析角色。首先读取 .vibe/ROLES.md 了解协作流程，读取 .vibe/HANDOFF.md 查看消息。你的职责：分析需求，输出 .vibe/SPEC.md。完成后在 .vibe/HANDOFF.md 追加 '[需求分析 → 技术设计]：SPEC.md 已完成。' Superpowers：brainstorming, writing-plans。" --cwd "$CWD" --mode yolo

# 2. Bug修复
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的 Bug修复角色。首先读取 .vibe/ROLES.md 了解协作流程，读取 .vibe/HANDOFF.md 查看 Bug 报告。你的职责：分析根因，编写修复方案到 .vibe/BUGFIX.md。完成后在 .vibe/HANDOFF.md 追加 '[Bug修复 → 代码编写]：BUGFIX.md 已完成。' Superpowers：systematic-debugging。" --cwd "$CWD" --mode yolo

# 3. 技术设计
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的技术设计角色。首先读取 .vibe/ROLES.md 了解协作流程，读取 .vibe/HANDOFF.md 查看上游消息。你的职责：设计技术方案，输出 .vibe/DESIGN.md。完成后在 .vibe/HANDOFF.md 追加 '[技术设计 → 代码编写]：DESIGN.md 已完成。' Superpowers：brainstorming, writing-plans。" --cwd "$CWD" --mode yolo

# 4. 代码编写
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的代码编写角色。首先读取 .vibe/ROLES.md 了解协作流程，读取 .vibe/HANDOFF.md 查看上游消息。你的职责：根据 DESIGN.md 或 BUGFIX.md 编写代码。完成后在 .vibe/HANDOFF.md 追加 '[代码编写 → 功能验证]：实现完成。' Superpowers：TDD, subagent-driven-development, verification-before-completion。" --cwd "$CWD" --mode yolo

# 5. 功能验证
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的功能验证角色。首先读取 .vibe/ROLES.md 了解协作流程，读取 .vibe/HANDOFF.md 查看上游消息。你的职责：验证功能，发现 Bug → Bug修复，发现偏差 → 需求分析。完成后在 .vibe/HANDOFF.md 追加验证报告。Superpowers：verification-before-completion, systematic-debugging。" --cwd "$CWD" --mode yolo
```

**提取会话 ID 并注册到 GUI**：创建完成后，从 CLI 数据库获取会话 ID，同时写入 GUI 任务索引（否则 ZCode 界面看不到）：

```bash
node -e "
const { DatabaseSync } = require('node:sqlite');
const h = require('os').homedir();

// 从 CLI 数据库获取最新 5 个会话
const cli = new DatabaseSync(h + '/.zcode/cli/db/db.sqlite');
const rows = cli.prepare('SELECT id, time_created FROM session WHERE directory = ? ORDER BY time_created DESC LIMIT 5').all(process.cwd());

	// 写入 GUI 任务索引（title_overridden=1 防止被第一条消息覆盖标题）
	const v2 = new DatabaseSync(h + '/.zcode/v2/tasks-index.sqlite');
	const ins = v2.prepare('INSERT OR REPLACE INTO tasks (workspace_key, workspace_path, task_id, title, task_status, provider, mode, model, created_at, updated_at, meta_json, title_overridden) VALUES (?, ?, ?, ?, \"active\", \"glm\", \"build\", ?, ?, ?, \"{}\", 1)');

const roles = ['🎯 需求分析', '🐛 Bug修复', '🏗️ 技术设计', '💻 代码编写', '✅ 功能验证'];
rows.reverse().forEach((r, i) => {
  const title = roles[i] + ' — {PROJECT_NAME}';
  ins.run(process.cwd(), process.cwd(), r.id, title, 'da99b590-0a1a-4b4d-a152-16f0ed4171c0/deepseek-v4-pro', r.time_created, r.time_created);
  console.log(r.id, title);
});

// 同时修复 CLI 数据库中的标题
const cliUpd = cli.prepare('UPDATE session SET title = ?, title_source = \"custom\" WHERE id = ?');
rows.reverse().forEach((r, i) => cliUpd.run(roles[i] + ' — {PROJECT_NAME}', r.id));

cli.close(); v2.close();
"
```

然后将所有会话 ID 填入 `.vibe/ROLES.md` 和 `.vibe/HANDOFF.md`。

#### 路径 B：手动创建（Node.js < 22）

如果 Node 版本不够，输出以下手动创建指南，让用户在新终端窗口中逐个创建：

```
⚠️ 当前 Node 版本过低（需要 ≥ 22），无法自动创建会话。

请在新终端窗口中逐个创建角色会话。每个会话启动后：
1. 粘贴对应角色的启动提示词
2. 记录会话 ID
3. 手动更新 .vibe/ROLES.md 中的「会话 ID」列
```

然后输出每个角色的启动提示词（见「使用指南 → 如何启动角色会话」）。

---

### 7. 初始化 .gitignore（可选但推荐）

向用户确认是否需要在 `.gitignore` 中添加 `.vibe/` 目录。说明：

- **建议不忽略**：`ROLES.md`、`SPEC.md` 和 `HANDOFF.md` 是项目协作的核心文档，团队成员应共享
- **建议忽略**：如果角色会话记录包含敏感内容

### 8. 输出初始化摘要

执行完以上步骤后，输出结构化的摘要：

```markdown
## ✅ Vibe Coding 项目初始化完成

| 项目 | 状态 |
|------|------|
| Git 仓库 | ✅ 已初始化 / ⏭️ 已存在 |
| .vibe/ROLES.md | ✅ 已创建 |
| .vibe/HANDOFF.md | ✅ 已创建 |
| 角色会话 | {自动创建完成 / ⚠️ 需手动创建（共 5 个）} |

### 会话状态

| 角色 | 会话 ID | 创建方式 |
|------|---------|----------|
| 🎯 需求分析 | {session_id / *待创建*} | {自动 / 手动} |
| 🐛 Bug修复 | {session_id / *待创建*} | {自动 / 手动} |
| 🏗️ 技术设计 | {session_id / *待创建*} | {自动 / 手动} |
| 💻 代码编写 | {session_id / *待创建*} | {自动 / 手动} |
| ✅ 功能验证 | {session_id / *待创建*} | {自动 / 手动} |

### 跨会话协作机制

- 📨 **消息总线**：`.vibe/HANDOFF.md` — 各角色在此写入交付物和问题
- 🔗 **会话关联**：`ReadSessionContext` — 任意角色可读取其他角色的完整上下文
- 📋 **角色定义**：`.vibe/ROLES.md` — 所有角色共享的协作规则

### 下一步

按正向流水线依次启动角色会话，每个会话启动后自动读取 HANDOFF.md 获取上游交付物。
```

---

## 注意事项

- 如果项目已有 `.vibe/ROLES.md`，询问用户是要覆盖还是保留现有配置
- 创建的角色会话引导语应根据实际项目名称调整
- 角色之间的会话 ID 可以通过 ZCode 的 `ReadSessionContext` 工具跨会话读取
- `.vibe/HANDOFF.md` 是所有会话的共享消息总线，追加消息时使用表格行格式
- 自动创建会话依赖 Node.js ≥ 22（支持 `node:sqlite`）。如果版本不够，技能会自动走手动创建路径
- 会话创建后，**所有角色共享同一个项目目录**，通过 `.vibe/` 下的文件实现信息传递
