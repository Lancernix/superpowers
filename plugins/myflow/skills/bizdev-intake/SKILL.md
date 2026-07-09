---
name: bizdev-intake
description: 业务开发流程步骤①：根据需求来源（钉钉链接/Confluence 链接/口头描述）路由读取，并调用 prd agent 产出结构化需求 requirements-raw.md
---

# BizDev Intake（需求获取）

你是需求获取的路由者。目标：产出 `~/.bizdev/<project-key>/requirements-raw.md`（原始需求采集）。

## 源路由

- **钉钉文档链接**（`*.dingtalk.com`）：通过钉钉 MCP（`dingtalk-doc.*`）读取；`sf-alidocs.dingtalk.com` 需替换为 `alidocs.dingtalk.com`。
- **Confluence 链接**（`confluence.sf-express.com`）：通过 Confluence MCP 读取。
- **口头描述**：用户直接在对话里给的需求，无需外部读取，直接整理进 requirements-raw.md。
- 其它内部系统（丰声/腾讯文档/zhi水）：本期未实现，提示人改用钉钉/Confluence 或口头。

## 模式感知

本 skill 读取编排层传入的 `mode`：
- `mode = full`：按上方「源路由」完整执行（MCP 路由 → prd agent 结构化解读）
- `mode = fast`：走下方「快速分支」，简化执行

## 步骤

1. 判定来源类型。
2. 若是链接：调用 `bizdev-workflow:prd` agent，把链接交给它读取并产出结构化需求（prd agent 负责图片/附件下载与委派 image-viewer）。**在派发 prompt 中显式传入输出路径** `~/.bizdev/<project-key>/requirements-raw.md`，要求 prd agent 把最终结构化需求写入该路径。
3. 若是口头：自行把对话需求整理为同结构文本。
4. 写入 `~/.bizdev/<project-key>/requirements-raw.md`。
5. 读取失败（链接无效 / 无对应 MCP 工具）：停在 bizdev 闸门1 前，向人报告并请求正确来源或配置 MCP，**不静默跳过**。

完成后把控制权交回 `bizdev` 编排层（`mode = fast` 时阶段置为 plan，`mode = full` 时置为 clarify）。

## 快速分支（mode = fast）

1. 跳过 MCP 路由判断（不区分钉钉/Confluence/口头）。
2. 若用户给了链接：仍调 `bizdev-workflow:prd` agent 读取（简单需求也可能有链接）。
3. 若用户直接给了文字描述：自行整理为 `requirements-raw.md`。
4. 写入 `~/.bizdev/<project-key>/requirements-raw.md`。
5. 完成后把控制权交回 `bizdev` 编排层（阶段置为 plan）。
