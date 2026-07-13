---
name: writing-plans
description: 在拥有多步任务的 spec 或 requirements，且动手改代码之前使用
---

# Writing Plans

## 概述

编写全面的 implementation plan。假设工程师对我们的 codebase 零上下文，且品味一般。把他们需要知道的一切都写进文档里：每个 task 要改哪些文件、可能涉及的 code、tests、docs、如何测试。把整个 plan 拆成可一口吞下的小 task。DRY。YAGNI。TDD。频繁 commit。

假设他们是熟练的开发者，但对我们的 toolset 或问题领域几乎一无所知。假设他们不太懂好的 test design。

**起始时声明：** "I'm using the writing-plans skill to create the implementation plan."


**保存计划到：** `~/.bizdev/<project-key>/plan.md`（由 bizdev 编排层传入 project-key；不进仓库）
-（用户对 plan 位置的偏好会覆盖此默认值）

## 范围检查

如果 spec 覆盖多个独立 subsystem，在 brainstorming 阶段就该拆成 sub-project spec。如果没拆，建议拆成多个 plan —— 每个 subsystem 一个。每个 plan 应独立产出可工作、可测试的软件。

## 文件结构

在定义 tasks 之前，先梳理会创建或修改哪些文件，以及每个文件的职责。这里确定 decomposition 决策。

- 设计单元要有清晰的边界和定义明确的接口。每个文件应只有一个明确的职责。
- 你对能一次握在上下文里的 code 推理最清楚，当文件聚焦时你的编辑也更可靠。优先选择更小、更聚焦的文件，而不是做太多事的大文件。
- 一起变化的文件应放在一起。按职责拆分，而不是按技术层拆分。
- 在已有 codebase 中，遵循既有 pattern。如果 codebase 使用大文件，不要单方面重构 —— 但如果你正在修改的文件已经膨胀到难以维护，在 plan 中加入拆分是合理的。

这个结构指导 task decomposition。每个 task 应产出独立成章的变更，且本身可独立成立。

## Task 粒度

一个 task 是能携带自己的 test cycle 且值得一个 fresh reviewer's gate 的最小单位。在划 task 边界时：把 setup、configuration、scaffolding 和 documentation 步骤并入需要它们的 deliverable task；只有在 reviewer 可以有意义地拒绝一个 task 同时批准其相邻 task 时才拆分。每个 task 以独立可测试的 deliverable 结束。

## 小粒度 Task

**每一步是一个动作（2-5 分钟）：**
- "Write the failing test" — step
- "Run it to make sure it fails" — step
- "Implement the minimal code to make the test pass" — step
- "Run the tests and make sure they pass" — step
- "Commit" — step

## Plan 文档头部

**每个 plan 必须以这个 header 开头：**

```markdown
# [Feature Name] Implementation Plan:

> **For agentic workers:** REQUIRED SUB-SKILL: Use bizdev-workflow:subagent-driven-development to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

---
```

## Task 结构

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

- [ ] **Step 1: Write the failing test:**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 不允许占位符

每一步必须包含工程师需要的实际内容。这些是 **plan failures** —— 永远不要写：
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above"（没有实际 test code）
- "Similar to Task N"（重复 code —— 工程师可能乱序阅读 tasks）
- 只描述做什么但不展示如何做的 steps（code step 必须提供 code blocks）
- 引用在任何 task 中都未定义的 types、functions 或 methods

## 记住
- 总是精确文件路径
- 每一步都要完整的 code —— 如果 step 修改 code，展示 code
- 带预期输出的精确命令
- DRY, YAGNI, TDD, 频繁 commit

## 自检

写完完整 plan 后，带着 fresh eyes 对照 spec 检查 plan。这是你自己跑的一个 checklist —— 不是 subagent dispatch。

**1. Spec 覆盖：** 快速浏览 spec 的每个 section/requirement。你能指出哪个 task 实现它吗？列出任何缺口。

**2. 占位符扫描：** 在你的 plan 中搜索红旗 —— 以上 "No Placeholders" section 中的任何模式。修复它们。

**3. 类型一致性：** 你在后续 task 中使用的 types、method signatures 和 property names 与你在前面 task 中定义的一致吗？Task 3 中叫 `clearLayers()` 但 Task 7 中叫 `clearFullLayers()` 就是一个 bug。

如果发现问题，inline 修复。无需重新 review —— 只需修复然后继续。如果你发现 spec requirement 没有对应的 task，添加 task。
