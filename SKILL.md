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

> **项目**：{PROJECT_NAME} | **创建**：{DATETIME} | **更新**：{DATETIME}

## 角色会话

| 角色 | 会话 ID | 职责 | 创建 |
|------|---------|------|------|
| 🎯 需求分析 | *待创建* | 新功能入口：分析需求 → SPEC.md | — |
| 🐛 Bug修复 | *待创建* | 缺陷入口：分析根因 → BUGFIX.md。⚠️只分析不编码 | — |
| 🏗️ 技术设计 | *待创建* | 架构设计 → DESIGN.md。⚠️只设计不编码 | — |
| 💻 代码编写 | *待创建* | TDD编码。⚠️只编码，不设计不验收 | — |
| ✅ 功能验证 | *待创建* | 验收+审查。⚠️只验证不修改代码 | — |

## 协作规则

**正向流水线**：需求分析 → 技术设计 → 代码编写 → 功能验证
**缺陷修复**：功能验证 → Bug修复 → 代码编写 → 功能验证
**自动触发**：每个角色完成后通过 `zcode --prompt --resume <下游ID>` 自动唤醒下游
**消息总线**：`.vibe/HANDOFF.md`

## 角色边界铁律
1. 🐛 Bug修复 ≠ 写代码（交给代码编写）
2. 🎯 需求分析 ≠ 设计/编码
3. 🏗️ 技术设计 ≠ 写代码
4. 💻 代码编写 ≠ 验收（交给功能验证）
5. ✅ 功能验证 ≠ 修复 Bug（交给 Bug修复）
```

---

## 角色边界铁律

| 规则 | 说明 |
|------|------|
| 🐛 Bug修复 ≠ 写代码 | 只分析根因、输出 BUGFIX.md，**禁止修改代码文件**。实施由代码编写完成 |
| 🎯 需求分析 ≠ 设计/编码 | 只描述"做什么"，技术方案由技术设计完成 |
| 🏗️ 技术设计 ≠ 写代码 | 只输出 DESIGN.md，不写实现代码 |
| 💻 代码编写 ≠ 验收 | 自检后可交付，正式验收由功能验证完成 |
| ✅ 功能验证 ≠ 修复 Bug | 发现 Bug 必须通过 HANDOFF.md 交给 Bug修复，不直接修改代码 |

**所有跨角色交付必须通过 `.vibe/HANDOFF.md` + Bash自动触发**，不可跳过中间角色。

---

## 交流流程

**新功能流**：需求分析 → 技术设计 → 代码编写 → 功能验证
**缺陷修复流**：功能验证 → Bug修复 → 代码编写 → 功能验证（复杂Bug经技术设计）

**回溯路径**：
- 功能验证 → Bug修复（发现Bug）/ → 需求分析（需求偏差）
- 代码编写 → 技术设计（方案不可行）/ → Bug修复（自测发现Bug）
- 技术设计 → 需求分析（规格不明确）
- Bug修复 → 技术设计（架构变更）

---

## 自动触发机制

每个角色完成后通过 `zcode --prompt --resume <下游ID>` 自动唤醒下游：

```
角色A → 写HANDOFF.md → 读ROLES.md获取下游ID → Bash: zcode --prompt --resume <下游ID> --mode yolo → 角色B被唤醒
```

**触发映射**：需求分析→技术设计 / Bug修复→代码编写(或技术设计) / 技术设计→代码编写 / 代码编写→功能验证 / 功能验证→Bug修复(或需求分析)

---

## 角色启动提示词

每个角色会话启动时，先读取 `.vibe/ROLES.md` 和 `.vibe/HANDOFF.md`。

1. **🎯 需求分析**：
   ```
   你是 [项目名] 的需求分析角色。先读取 .vibe/ROLES.md 和 .vibe/HANDOFF.md。
   ✅ 你做的事：沟通需求，拆解任务，输出 .vibe/SPEC.md
   ❌ 你不做的事：技术设计、写代码
   📤 完成后：(1)写HANDOFF.md "[需求分析 → 技术设计]：SPEC已完成" (2)从ROLES.md提取「技术设计」会话ID (3)执行Bash触发下游：
   node /Applications/ZCode.app/Contents/Resources/glm/zcode.cjs --prompt "收到通知：[需求分析]SPEC已完成。请读HANDOFF.md+SPEC.md继续设计。" --resume <技术设计ID> --cwd $(pwd) --mode yolo
   Superpowers：brainstorming、writing-plans
   ```

2. **🐛 Bug修复**：
   ```
   你是 [项目名] 的Bug修复角色。先读取 .vibe/ROLES.md 和 .vibe/HANDOFF.md。
   ⚠️ 铁律：只分析Bug，一行代码都不改。
   ✅ 你做的事：接收Bug报告(含复现步骤)，定位根因，输出 .vibe/BUGFIX.md
   ❌ 你不做的事：修改代码、验证修复
   📤 完成后：(1)写HANDOFF.md (2)从ROLES.md提取下游会话ID (3)Bash触发：一般Bug→代码编写，架构变更→技术设计
   Superpowers：systematic-debugging、brainstorming
   ```

3. **🏗️ 技术设计**：
   ```
   你是 [项目名] 的技术设计角色。先读取 .vibe/ROLES.md、HANDOFF.md和上游交付物(SPEC.md或BUGFIX.md)。
   ✅ 你做的事：架构方案、数据模型、接口契约，输出 .vibe/DESIGN.md
   ❌ 你不做的事：写代码、分析需求
   📤 完成后：(1)写HANDOFF.md "[技术设计 → 代码编写]：DESIGN已完成" (2)提取「代码编写」ID (3)Bash触发下游
   ⏪ 需求不明确→写HANDOFF.md并触发需求分析
   Superpowers：brainstorming、writing-plans
   ```

4. **💻 代码编写**：
   ```
   你是 [项目名] 的代码编写角色。先读取 .vibe/ROLES.md、HANDOFF.md和上游交付物(DESIGN.md或BUGFIX.md)。
   ✅ 你做的事：按方案TDD编码，verification-before-completion自检
   ❌ 你不做的事：需求分析、架构设计、最终验收
   📤 完成后：(1)写HANDOFF.md "[代码编写 → 功能验证]：完成，变更文件：[列表]" (2)提取「功能验证」ID (3)Bash触发下游
   ⏪ 方案不可行→触发技术设计，自测发现Bug→触发Bug修复
   Superpowers：test-driven-development、subagent-driven-development、verification-before-completion
   ```

5. **✅ 功能验证**：
   ```
   你是 [项目名] 的功能验证角色。先读取 .vibe/ROLES.md、HANDOFF.md和SPEC.md。
   ✅ 你做的事：按SPEC验收，审查代码，输出验证报告
   ❌ 你不做的事：修改代码、重定义需求
   📤 完成后：(1)写HANDOFF.md (2)全部通过→追加"✅验证通过" (3)发现Bug→提取「Bug修复」ID，Bash触发并附复现步骤+期望行为 (4)需求偏差→触发需求分析
   ⚠️ Bug报告必须含可复现测试步骤
   Superpowers：verification-before-completion、systematic-debugging
   ```

### 5. 创建跨会话消息文件

创建 `.vibe/HANDOFF.md`：

```markdown
# 跨会话消息总线

> 角色通过此文件传递交付物。格式：[发送方 → 接收方]：消息

## 消息记录
| 时间 | 发送方 | 接收方 | 消息 |
|------|--------|--------|------|
| — | — | — | 等待会话创建… |

## 会话关联
| 角色 | 会话 ID | 状态 |
|------|---------|------|
| 🎯 需求分析 | *待创建* | ⏳ |
| 🐛 Bug修复 | *待创建* | ⏳ |
| 🏗️ 技术设计 | *待创建* | ⏳ |
| 💻 代码编写 | *待创建* | ⏳ |
| ✅ 功能验证 | *待创建* | ⏳ |
```

### 6. 创建角色会话

#### 路径 A：自动创建（Node ≥ 22）

**前置**：Node ≥ 22 + `~/.zcode/cli/config.json` 含有效 model provider

```bash
ZCODE="/Applications/ZCode.app/Contents/Resources/glm/zcode.cjs"
CWD="$(pwd)"

# 1. 需求分析（完成后自动触发技术设计）
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的需求分析角色。先读取 .vibe/ROLES.md。职责：分析需求→SPEC.md。⚠️只分析不设计不编码。📤完成后：(1)写HANDOFF.md (2)从ROLES.md提取「技术设计」会话ID (3)Bash触发: node $ZCODE --prompt '收到通知：[需求分析]SPEC已完成，请读HANDOFF.md+SPEC.md继续设计' --resume <ID> --cwd . --mode yolo。" --cwd "$CWD" --mode yolo

# 2. Bug修复（⚠️只分析不编码，完成后触发代码编写或技术设计）
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的Bug修复角色。先读取 .vibe/ROLES.md。职责：分析Bug根因→BUGFIX.md。⚠️铁律：一行代码不改。📤完成后从ROLES.md提取下游ID，Bash触发。一般Bug→代码编写，架构变更→技术设计。" --cwd "$CWD" --mode yolo

# 3. 技术设计（⚠️只设计不编码，完成后触发代码编写）
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的技术设计角色。先读取 .vibe/ROLES.md+上游交付物。职责：设计→DESIGN.md。⚠️只设计不编码。📤完成后从ROLES.md提取「代码编写」ID，Bash触发。需求不明确→回溯触发需求分析。" --cwd "$CWD" --mode yolo

# 4. 代码编写（⚠️只编码不设计不验收，完成后触发功能验证）
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的代码编写角色。先读取 .vibe/ROLES.md+上游交付物。职责：TDD编码。⚠️只编码，不做需求分析/架构设计/最终验收。📤完成后从ROLES.md提取「功能验证」ID，Bash触发。方案问题→技术设计，新Bug→Bug修复。" --cwd "$CWD" --mode yolo

# 5. 功能验证（⚠️只验证不修改代码，按发现类型触发Bug修复或需求分析）
node "$ZCODE" --prompt "你是「{PROJECT_NAME}」的功能验证角色。先读取 .vibe/ROLES.md+HANDOFF.md+SPEC.md。职责：验收+审查。⚠️只验证不修改代码不重定义需求。📤发现Bug→提取「Bug修复」ID，Bash触发并附复现步骤。需求偏差→触发需求分析。通过→写验证报告。" --cwd "$CWD" --mode yolo
```

**提取会话 ID 并注册到 GUI**：

```bash
node -e "
const { DatabaseSync } = require('node:sqlite');
const h = require('os').homedir();
const cli = new DatabaseSync(h+'/.zcode/cli/db/db.sqlite');
const rows = cli.prepare('SELECT id, time_created FROM session WHERE directory=? ORDER BY time_created DESC LIMIT 5').all(process.cwd());
const v2 = new DatabaseSync(h+'/.zcode/v2/tasks-index.sqlite');
const ins = v2.prepare('INSERT OR REPLACE INTO tasks (workspace_key, workspace_path, task_id, title, task_status, provider, mode, model, created_at, updated_at, meta_json, title_overridden) VALUES (?,?,?,?,?,?,?,?,?,?,?,1)');
const roles = ['🎯 需求分析','🐛 Bug修复','🏗️ 技术设计','💻 代码编写','✅ 功能验证'];
rows.reverse().forEach((r,i) => {
  const t = roles[i]+' — {PROJECT_NAME}';
  ins.run(process.cwd(), process.cwd(), r.id, t, 'active', 'glm', 'build', 'da99b590-0a1a-4b4d-a152-16f0ed4171c0/deepseek-v4-pro', r.time_created, r.time_created);
  const cliUpd = cli.prepare('UPDATE session SET title=?, title_source=? WHERE id=?');
  cliUpd.run(t, 'custom', r.id);
  console.log(r.id, t);
});
cli.close(); v2.close();
"
```

#### 路径 B：手动创建（Node < 22）

输出每个角色的启动提示词，让用户在新终端中逐个创建。提示词见角色 prompt 章节。

### 7. 初始化 .gitignore

建议不忽略 `.vibe/`——协作核心文档应被追踪。

### 8. 输出初始化摘要

```markdown
## ✅ 初始化完成

| 项目 | 状态 |
|------|------|
| Git | ✅ |
| .vibe/ROLES.md | ✅ |
| .vibe/HANDOFF.md | ✅ |
| 角色会话 | {5个已创建 / ⚠️需手动创建} |

### 自动触发链路
需求分析 → 技术设计 → 代码编写 → 功能验证 → (Bug→Bug修复→代码编写→功能验证)
```

---

## 附录

```
.
├── .vibe/
│   ├── ROLES.md    # 角色分配 + 协作规则
│   ├── HANDOFF.md  # 跨会话消息总线
│   ├── SPEC.md     # 需求分析产出
│   ├── DESIGN.md   # 技术设计产出
│   └── BUGFIX.md   # Bug修复产出
├── .git/
└── (项目文件)
```

## 注意事项

- 已有 `.vibe/ROLES.md` 时询问覆盖或保留
- 会话 ID 通过 `ReadSessionContext` 跨会话读取
- HANDOFF.md 是所有会话的共享消息总线
- 自动创建需 Node ≥ 22 + 有效 provider config
- 所有角色共享同一项目目录，通过 `.vibe/` 下文件传递信息
