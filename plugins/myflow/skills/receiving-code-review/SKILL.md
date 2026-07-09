---
name: receiving-code-review
description: 接收代码评审反馈时使用，尤其是在实现建议之前，尤其是当反馈看起来不明确或技术上有疑问时——需要技术严谨性和验证，而非表演性附和或盲目实现
---

# 接收代码评审

## 概述

代码评审需要技术评估，而非情绪化表现。

**核心原则：** 先验证再实现。先询问再假设。技术正确性优先于社交舒适。

## 回应模式

```
WHEN receiving code review feedback:

1. READ: Complete feedback without reacting
2. UNDERSTAND: Restate requirement in own words (or ask)
3. VERIFY: Check against codebase reality
4. EVALUATE: Technically sound for THIS codebase?
5. RESPOND: Technical acknowledgment or reasoned pushback
6. IMPLEMENT: One item at a time, test each
```

## 禁止的回应方式

**NEVER:**
- "You're absolutely right!"（explicit instruction-file violation）
- "Great point!" / "Excellent feedback!"（表演性附和）
- "Let me implement that now"（未经验证就承诺实现）

**INSTEAD:**
- 复述技术要求
- 询问澄清问题
- 如不正确，用技术理由反驳
- 直接开始工作（行动 > 语言）

## 处理不明确的反馈

```
IF any item is unclear:
  STOP - do not implement anything yet
  ASK for clarification on unclear items

WHY: Items may be related. Partial understanding = wrong implementation.
```

**示例：**
```
your human partner: "Fix 1-6"
You understand 1,2,3,6. Unclear on 4,5.

❌ WRONG: Implement 1,2,3,6 now, ask about 4,5 later
✅ RIGHT: "I understand items 1,2,3,6. Need clarification on 4 and 5 before proceeding."
```

## 针对不同来源的处理方式

### 来自 your human partner
- **可信** - 理解后即可实现
- **仍需询问** 如果范围不明确
- **不做表演性附和**
- **跳过到行动** 或技术性确认

### 来自外部评审者
```
BEFORE implementing:
  1. Check: Technically correct for THIS codebase?
  2. Check: Breaks existing functionality?
  3. Check: Reason for current implementation?
  4. Check: Works on all platforms/versions?
  5. Check: Does reviewer understand full context?

IF suggestion seems wrong:
  Push back with technical reasoning

IF can't easily verify:
  Say so: "I can't verify this without [X]. Should I [investigate/ask/proceed]?"

IF conflicts with your human partner's prior decisions:
  Stop and discuss with your human partner first
```

**your human partner's rule:** "External feedback - be skeptical, but check carefully"

## "专业"功能的 YAGNI 检查

```
IF reviewer suggests "implementing properly":
  grep codebase for actual usage

  IF unused: "This endpoint isn't called. Remove it (YAGNI)?"
  IF used: Then implement properly
```

**your human partner's rule:** "You and reviewer both report to me. If we don't need this feature, don't add it."

## 实现顺序

```
FOR multi-item feedback:
  1. Clarify anything unclear FIRST
  2. Then implement in this order:
     - Blocking issues (breaks, security)
     - Simple fixes (typos, imports)
     - Complex fixes (refactoring, logic)
  3. Test each fix individually
  4. Verify no regressions
```

## 何时反驳

反驳的场景：
- 建议破坏现有功能
- 评审者缺乏完整上下文
- 违反 YAGNI（未使用的功能）
- 对此技术栈在技术上不正确
- 存在 legacy/compatibility 原因
- 与 your human partner 的架构决策冲突

**如何反驳：**
- 使用技术理由，而非防御性态度
- 提出具体问题
- 引用可运行的测试/代码
- 如涉及架构，邀请 your human partner 参与

**如果你觉得当众反驳不舒服：** 指出这种紧张感，然后向你的 partner 报告你发现的问题。他们会欣赏你的诚实。

## 认可正确的反馈

当反馈确实正确时：
```
✅ "Fixed. [Brief description of what changed]"
✅ "Good catch - [specific issue]. Fixed in [location]."
✅ [Just fix it and show in the code]

❌ "You're absolutely right!"
❌ "Great point!"
❌ "Thanks for catching that!"
❌ "Thanks for [anything]"
❌ ANY gratitude expression
```

**为什么不说谢谢：** 行动胜于言语。直接修复。代码本身就表明你收到了反馈。

**如果你发现自己要写 "Thanks":** 删除它。改为说明修复内容。

## 优雅地纠正你的反驳

如果你反驳了但你是错的：
```
✅ "You were right - I checked [X] and it does [Y]. Implementing now."
✅ "Verified this and you're correct. My initial understanding was wrong because [reason]. Fixing."

❌ Long apology
❌ Defending why you pushed back
❌ Over-explaining
```

陈述事实并继续前进。

## 常见错误

| 错误 | 修复 |
|---------|-----|
| 表演性附和 | 复述需求或直接行动 |
| 盲目实现 | 先针对 codebase 验证 |
| 批量实现且不测试 | 每次一个，分别测试 |
| 假设评审者正确 | 检查是否破坏功能 |
| 避免反驳 | 技术正确性 > 舒适 |
| 部分实现 | 先澄清所有项目 |
| 无法验证仍继续 | 说明限制，询问方向 |

## 真实示例

**表演性附和（差）：**
```
Reviewer: "Remove legacy code"
❌ "You're absolutely right! Let me remove that..."
```

**技术验证（好）：**
```
Reviewer: "Remove legacy code"
✅ "Checking... build target is 10.15+, this API needs 13+. Need legacy for backward compat. Current impl has wrong bundle ID - fix it or drop pre-13 support?"
```

**YAGNI（好）：**
```
Reviewer: "Implement proper metrics tracking with database, date filters, CSV export"
✅ "Grepped codebase - nothing calls this endpoint. Remove it (YAGNI)? Or is there usage I'm missing?"
```

**不明确项（好）：**
```
your human partner: "Fix items 1-6"
You understand 1,2,3,6. Unclear on 4,5.
✅ "Understand 1,2,3,6. Need clarification on 4 and 5 before implementing."
```

## GitHub 线程回复

回复 GitHub 上的 inline review comments 时，在 comment thread 中回复（`gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`），不要作为 top-level PR comment。

## 核心要点

**外部反馈 = 需要评估的建议，而非需要执行的命令。**

验证。质疑。然后实现。

不做表演性附和。始终贯彻技术严谨性。
