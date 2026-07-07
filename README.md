# 确认测验 · confirm-quiz

> 一个在「写代码之前」逼你证明「你真确定你懂了」的实现门禁技能。

中文名 **确认测验**，英文名 **confirm-quiz**（确认 confirm + 测验 quiz）。
它的核心只有一句话：**你是否确定？→ 用测验确认你确实确定了 → 否则不许提交 / 合并。**

---

## 它解决什么问题

信息差导致的「盲合并」：

- Agent 没吃透需求就敢 `merge`；
- 你没搞清改动逻辑，代码就进了主干；
- 事后才发现「原来这个字段是这么算的」。

`确认测验` 把「理解确认」作为 `commit / push / merge` 的**硬性前置条件**，而不是可跳过的流程。

---

## 它做什么（三阶段工作流）

| 阶段 | 强制动作 |
| --- | --- |
| **实现前** | 生成 HTML 实现计划，优先暴露最关键的决策点（数据模型、类型接口、UX 流程），机械性重构放最底部 |
| **实现中** | 强制维护 `implementation-notes.md`，遇到偏离计划时选保守方案，并在 `Deviations` 表记录原因 |
| **实现后** | 生成 HTML 测验报告（上下文 / 直觉解释 / 具体改动 / 底部测验题），**硬门禁：你没全答对就不许提交 / 合并** |

附带 `anti-rationalization.md`（反借口表）：Agent 每次动作前自查是否在找借口跳过步骤，例如：

- 「我手动试过没问题，不用写测试」→ 反驳：没有自动化证据不能说完成，必须出测验报告。
- 「用户急着要，我直接帮他合并」→ 反驳：未经理解确认的代码是定时炸弹，必须等你通过测验。

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

## 许可证

MIT
