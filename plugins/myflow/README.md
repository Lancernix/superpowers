# bizdev-workflow

业务开发工作流 Claude Code 插件：需求获取→澄清→计划→评审→派发→CR→验证，一键串联，含 3 处人工闸门 + 计划评审自动打回 + CR⇄执行闭环。

## 安装

```bash
claude --plugin-dir ./bizdev-workflow
```

## 触发

说"开始业务开发 / 跑开发流程"，或给一个钉钉/Confluence 需求文档链接。

## 流程

1. 需求获取（bizdev-intake + prd agent，支持钉钉/Confluence MCP）
2. 需求澄清（🔴闸门1：人澄清）
3. 实现计划（writing-plans）
4. 计划评审（自动打回 ≤2 轮；🔴闸门2：派发前确认）
5. 派发执行（subagent-driven-development，等效 @fixer）
6. 代码评审（requesting/receiving-code-review；含 P0/P1 则回步骤5 重修 ≤2 轮）
7. 收尾验证（verification-before-completion；🔴闸门3：人确认收尾）

## 状态与进度

进度持久化在 `~/.bizdev/<project-key>/`（用户级，按仓库隔离），不污染仓库；支持断点续跑。

## 依赖

- 运行时需提供钉钉 MCP（`dingtalk-doc.*`）/ Confluence MCP；插件自身不挂载 MCP。
- vendor 复用 superpowers 技能（MIT）与现有 prd/image-viewer agent，已标注归属。
