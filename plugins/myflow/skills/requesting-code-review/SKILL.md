---
name: requesting-code-review
description: 业务开发流程步骤⑥：对派发执行产出发起代码评审，按 P0/P1/P2 分级；含 P0/P1 则触发 dispatch⇄CR 小闭环（≤2 轮）。也可独立用于完成任务后、合并前的工作验证。
---

# 请求代码评审

派发一个代码评审 subagent，在问题扩散前发现缺陷。评审者将获得精心准备的评估上下文——绝不是你的 session 历史。这使评审者专注于工作成果，而非你的思考过程，并保留你自己的上下文以便继续工作。

**核心原则：早评审，常评审。**

## 何时请求评审

**强制要求：**
- 在 subagent-driven development 中每个 task 之后
- 完成重大 feature 后
- 合并到 main 之前

**可选但有价值：**
- 遇到瓶颈时（获得新视角）
- 重构前（基线检查）
- 修复复杂 bug 后

## 如何请求

**1. 获取 git SHAs：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发代码评审 subagent：**

派发一个 `general-purpose` subagent，使用 [code-reviewer.md](code-reviewer.md) 中的模板。

**占位符：**
- `{DESCRIPTION}` - 所构建内容的简要概述
- `{PLAN_OR_REQUIREMENTS}` - 应实现的功能
- `{BASE_SHA}` - 起始 commit
- `{HEAD_SHA}` - 结束 commit

**3. 根据反馈行动：**
- 立即修复 Critical 问题
- 继续前修复 Important 问题
- 记录 Minor 问题留待后续处理
- 如果评审者错误则反驳（附带技术理由）

## 示例

```
[刚刚完成 Task 2：添加验证函数]

你：让我在继续前请求代码评审。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派发代码评审 subagent]
  DESCRIPTION: 添加了 verifyIndex() 和 repairIndex()，共 4 种问题类型
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md 中的 Task 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661

[Subagent 返回]：
  Strengths: 架构清晰，测试真实
  Issues:
    Important: 缺少进度指示器
    Minor: 报告间隔使用魔术数字 (100)
  Assessment: 可以继续

你：[修复进度指示器]
[继续 Task 3]
```

## 与工作流集成

**Subagent-Driven Development：**
- 每个 task 之后评审
- 在问题累积前发现
- 移动到下一个 task 前修复

**执行计划：**
- 每个 task 之后或在自然检查点评审
- 获得反馈，应用，继续

**Ad-Hoc 开发：**
- 合并前评审
- 遇到瓶颈时评审

## 警示信号

**绝不要：**
- 因"很简单"而跳过评审
- 忽略 Critical 问题
- 带着未修复的 Important 问题继续
- 与有效的技术反馈争辩

**如果评审者错误：**
- 附带技术理由反驳
- 展示证明其可行的代码/测试
- 请求澄清

参见模板：[code-reviewer.md](code-reviewer.md)
