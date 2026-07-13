---
name: bizdev-clarify
description: 业务开发流程步骤②：分析 requirements-raw.md，提炼待澄清问题清单，经人工闸门澄清后定稿 requirements.md
---

# BizDev Clarify（需求澄清）

读取 `~/.bizdev/<project-key>/requirements-raw.md`，分析并产出待澄清清单，经人工闸门后定稿 `requirements.md`。

## 步骤

1. 分析原始需求，识别：歧义点、缺失信息、冲突需求、需确认的边界/非功能要求。
2. 产出 **待澄清清单**（markdown，每条：问题 + 为什么需要澄清 + 建议选项）。
3. 🔴 **闸门1**：`ask your human partner` 逐条澄清（可一次抛出清单，等人回答）。
4. 把人回答合并进需求，定稿 `~/.bizdev/<project-key>/requirements.md`（结构化，含背景/功能点/验收标准/不确定项已闭合）。
5. 控制权交回 `bizdev` 编排层（阶段置为 plan）。

若 requirements-raw.md 不存在：报告 bizdev 编排层，停在 intake 之后、闸门1 之前。
