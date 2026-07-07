# 确认测验 · confirm-quiz

> 一个在「写代码之前」逼你证明「你真确定你懂了」的实现门禁技能。

- 中文名 **确认测验**，英文名 **confirm-quiz**（确认 confirm + 测验 quiz）。
- 核心一句话：**你是否确定？→ 用测验确认你确实确定了 → 否则不许提交 / 合并。**

本技能的方法论直接取自 Anthropic 工程师 Thariq Shihipar 的文章
*[A Field Guide to Claude Fable 5: Finding your Unknowns](https://claude.com/blog/a-field-guide-to-claude-fable-finding-your-unknowns)*
（首发于 [X](https://x.com/trq212/status/2073100352921215386)）。下面只节选与「确认测验」相关的部分。

---

## 设计来源：地图不是疆域

文章开篇的核心判断，正是本技能的存在理由（译文，来源同上）：

> 和 Claude Fable 5 一起工作，总会反复提醒我一个老道理：**地图不是疆域**。
> 地图，是对待完成工作的表征，也就是我的提示词、技能和上下文，是我交给 Claude 的东西。
> 疆域，才是工作真正发生的地方：代码库、现实世界，以及其中真实存在的约束。
> 地图和疆域之间的差异，就是我所说的**未知项**。当 Claude 遇到未知项时，它需要根据自己对我意图的最佳猜测来做决定。要做的事情越多，Claude 可能遇到的未知项就越多。

换句话说：**提示词 / 计划 / 技能只是「地图」，真正跑起来的是「疆域」**。Agent 一旦撞上未知项，就会替你猜。猜错了还照常 `merge`，就是盲合并。`确认测验` 要做的，就是在「地图」和「疆域」之间架一道确认门——尤其是文章强调的：**提前规划并不总是足够，未知项可能藏在实现深处**，所以验证必须贯穿实现前、中、后。

---

## 它是如何被设计的（对应原文三阶段）

文章把与 Agent 协作描述为一个「实现前 → 实现中 → 实现后」不断发现未知项的迭代循环。`确认测验` 把其中三段直接做成了技能的固定动作。

### 1. 实现前 — 实现计划（Pre-implementation: Implementation Plan）

> 当我觉得自己已经准备好实现时，我通常会请 Claude 写一份实现计划供我审阅，重点放在最可能需要改动的部分，比如数据模型、类型接口或 UX 流程。这能让 Claude 提前暴露那些我可能确实需要调整的东西。
> ……用 HTML 写一份实现计划，但**开头先放我最可能想调整的决策**：数据模型变更、新的类型接口，以及所有面向用户的内容。**机械性的重构细节放在底部**就好，那部分我信任你。

→ 这正是技能的「实现前」步骤：生成 HTML 实现计划，**优先暴露最关键的决策点**，机械重构沉底。

![实现计划 artifact 示例](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a4be4919e159adcdfa3ec1c_94358c3c.png)
*（配图来自 Thariq 原文，展示该方法下生成的实现计划 artifact）*

### 2. 实现中 — 实现笔记（During implementation: Implementation notes）

> 我会让 Claude Code 保留一个临时的 `implementation-notes.md`（或 `.html`）文件，在里面记录它做出的决定，这样我们就能从下一次尝试中学习。
> ……如果你遇到某个边界情况，导致你必须偏离计划，**就选择保守方案，把它记录在 `Deviations` 下面，然后继续往前做**。

→ 这正是技能的「实现中」步骤：强制维护 `implementation-notes.md`，遇偏离选保守方案并记入 `Deviations` 表。

![实现笔记 artifact 示例](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a4be4919e159adcdfa3ec52_cc0ee2de.png)
*（配图来自 Thariq 原文，展示该方法下生成的实现笔记 artifact）*

### 3. 实现后 — 测验门禁（Post implementation: Quizzes + gate）

> 经过一段很长的工作会话之后，Claude 可能完成了很多我都没意识到的事情。只读代码 diff 只能让我大致理解发生了什么，因为很多行为取决于既有代码路径。
> 让 Claude 在给我一大段上下文之后再来**测验我**，能帮助我理解到底发生了什么。**只有当我完美通过测验之后，我才会 merge。**
> ……给我一份 HTML 报告，解释这些改动，包含上下文、直觉、做了什么等等，并在底部放一个测验；**我必须通过这个测验才能继续**。

→ 这正是技能的「实现后」步骤 + **硬门禁**：生成 HTML 测验报告（上下文 / 直觉 / 具体改动 / 底部测验题），**你没全答对就不许 `commit / push / merge`**。

![测验报告 artifact 示例](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a4be4919e159adcdfa3ec55_c4646783.png)
*（配图来自 Thariq 原文，展示该方法下生成的测验报告 artifact）*

### 4. 反借口表（anti-rationalization）

文章点明一个纪律问题：**「提前规划并不总是足够」**——Agent 很容易在途中给自己找理由跳过计划、跳过笔记、跳过测验。
`确认测验` 为此附了一张 `anti-rationalization.md`（反借口表）：Agent 每次动作前自查是否在找借口跳过步骤，例如：

- 「我手动试过没问题，不用写测试」→ 反驳：没有自动化证据不能说完成，必须出测验报告。
- 「用户急着要，我直接帮他合并」→ 反驳：未经理解确认的代码是定时炸弹，必须等你通过测验。

---

## 它解决什么问题

信息差导致的「盲合并」：

- Agent 没吃透需求就敢 `merge`；
- 你没搞清改动逻辑，代码就进了主干；
- 事后才发现「原来这个字段是这么算的」。

`确认测验` 把「理解确认」作为 `commit / push / merge` 的**硬性前置条件**，而不是可跳过的流程。

---

## 它做什么（三阶段工作流）

| 阶段 | 强制动作 | 对应原文 |
| --- | --- | --- |
| **实现前** | 生成 HTML 实现计划，优先暴露最关键的决策点（数据模型、类型接口、UX 流程），机械性重构放最底部 | Implementation Plan |
| **实现中** | 强制维护 `implementation-notes.md`，遇到偏离计划时选保守方案，并在 `Deviations` 表记录原因 | Implementation notes |
| **实现后** | 生成 HTML 测验报告（上下文 / 直觉解释 / 具体改动 / 底部测验题），**硬门禁：你没全答对就不许提交 / 合并** | Quizzes + gate |

---

## 文件结构

```
confirm-quiz/
├── SKILL.md                      # 技能元信息与触发规则
├── anti-rationalization.md       # 反借口表
└── templates/
    ├── implementation-notes.md   # 实现中偏差记录模板
    └── quiz-report.html          # 实现后测验报告模板
```

---

## 安装方式

### 方式 A：从 GitHub 导入（推荐）

```bash
# 1. 克隆仓库
git clone https://github.com/mamawe/confirm-quiz.git

# 2. 压缩技能目录
cd confirm-quiz && zip -r confirm-quiz.zip .
```

然后在你的 AI 编程环境（WorkBuddy / Marvis / SOLO / TRAE / Claude Code 等）的技能市场点击「上传技能」，导入 `confirm-quiz.zip` 即可。

### 方式 B：直接放入技能目录（绕过 UI 导入）

把整个 `confirm-quiz/` 文件夹复制到对应环境的技能目录：

- **Marvis（macOS）**：
  ```
  ~/Library/Application Support/com.tencent.mac.marvis/MarvisData/User/default_user/skills/custom/
  ```
- **WorkBuddy（用户级）**：
  ```
  ~/.workbuddy/skills/
  ```
- **Claude Code / 类 SKILL.md 环境**：放到项目的 `.claude/skills/` 或用户级 skills 目录。

复制后**重启应用**即可生效。

---

## 使用触发

以下场景会自动触发本技能：

- 你准备开始编写代码
- 你请求代码审查
- 你准备合并代码
- 你提到「实现计划」「实现笔记」「确认测验」

触发后，它会依次产出：实现计划 → 实现笔记（含偏差） → 测验报告，并在你通过测验前拒绝执行 `git commit / push / merge`。

---

## 出处 / 参考

- Thariq Shihipar（Anthropic），*[A Field Guide to Claude Fable 5: Finding your Unknowns](https://claude.com/blog/a-field-guide-to-claude-fable-finding-your-unknowns)*，Anthropic Blog / [X 首发](https://x.com/trq212/status/2073100352921215386)。
- 本文 README 仅节选与该技能直接相关的文字与配图，版权归原作者所有。

---

## 许可证

MIT
