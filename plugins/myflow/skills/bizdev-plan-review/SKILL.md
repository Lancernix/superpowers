---
name: bizdev-plan-review
description: 业务开发流程步骤④：派 plan-reviewer 评审 plan.md，通过则交回编排层；不通过则打回 writing-plans 重做（自动闸门，≤2 轮）
---

# BizDev Plan Review（计划评审）

读取 `~/.bizdev/<project-key>/plan.md` 与 `requirements.md`，派 `bizdev-workflow:plan-reviewer` 评审。

## 步骤

1. 派 `bizdev-workflow:plan-reviewer` agent，传入 plan.md 与 requirements.md 路径。
2. 读评审结论：
   - `APPROVE` → 控制权交回 `bizdev` 编排层（进入闸门2）。
   - `REJECT` → 把问题清单交回 `bizdev-workflow:writing-plans` 重做计划（自动闸门）。**不直接改 plan.md**，由 writing-plans 重写。
3. 编排层维护 `review_rounds`：`REJECT` 后 `review_rounds += 1`；达到上限 2 仍 `REJECT` → 由编排层升为 🔴人工闸门裁决，本 skill 不参与死循环。

本 skill 只做"派评审 + 传递结论"，不自行判断计划优劣。
