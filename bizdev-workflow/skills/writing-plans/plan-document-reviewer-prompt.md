# 计划文档评审者 Prompt 模板

在派发 plan document reviewer subagent 时使用此模板。

**目的：** 验证 plan 完整、与 spec 对齐、且有正确的 task decomposition。

**派发时机：** 完整 plan 已写好后。

```
Subagent (general-purpose):
  description: "Review plan document"
  prompt: |
    你是一位计划文档评审者。验证此 plan 完整且可实现。

    **待评审的 plan:** [PLAN_FILE_PATH]
    **参考 spec:** [SPEC_FILE_PATH]

    ## 检查内容

    | 类别 | 检查点 |
    |------|--------|
    | 完整性 | TODO、占位符、不完整的 task、缺失的 step |
    | Spec 对齐 | plan 覆盖 spec 需求，无重大 scope creep |
    | Task 分解 | task 有清晰边界，step 可执行 |
    | 可构建性 | 工程师能否按此 plan 工作而不卡住？ |

    ## 校准

    **只标记会在实现中造成真实问题的缺陷。**
    实现者构建了错误的东西或卡住了才算问题。
    措辞微调、风格偏好和"nice to have"建议不算。

    除非有严重缺口——spec 需求缺失、步骤矛盾、占位符内容、或 task 模糊到
    无法执行——否则应 Approved。

    ## 输出格式

    ## Plan Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Task X, Step Y]: [具体问题] - [为什么对实现重要]

    **Recommendations (建议性，不阻止 approval):**
    - [改进建议]
```

**评审者返回:** Status、Issues（如有）、Recommendations
