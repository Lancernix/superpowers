# Repository Guidelines

## 这是什么仓库

个人维护的 Claude Code **技能（skills）与插件（plugin）收藏库**。本身不是编译型应用，交付物是 prompt/脚本文档。

- 把外部好用的开源 skill/plugin 拿过来做**轻量定制**（例如默认 `disable-model-invocation` 关闭 agent 自动感知），只保留真正有用的。
- 技能经 `npx skills` 分发；插件 `bizdev-workflow` 经 marketplace（`.claude-plugin/marketplace.json`，marketplace 名 `agent-tools`）分发。
- License: MIT。改/加他人 skill 时保留原作者归属。

## 目录结构

```
skills/<name>/          # 独立技能：每个目录一个，至少含 SKILL.md
  SKILL.md              # agent 提示词 + YAML 前置元数据（核心）
  scripts/              # 可选的确定性脚本（bash / node）
  references/           # 可选：跨平台工具映射等大段参考资料
  examples/             # 可选：验证样例 / 示例输出
bizdev-workflow/        # Claude Code 插件（bizdev 业务开发工作流）
  .claude-plugin/plugin.json
  agents/               # prd.md / plan-reviewer.md / image-viewer.md
  skills/               # 插件内 skill（bizdev 编排器 + 各阶段）
```

## 同步的开源技能版本（维护重点）

当前 vendored 进来的开源技能，其上游来源与同步版本统一记录在 **README.md 的「已同步的开源技能」章节**（表格列：技能、来源仓库、上游 commit、同步日期、备注）。

**如何查证与维护：**
- 要确认某技能同步自哪个上游、哪个 commit，去读 README.md 的「已同步的开源技能」章节，按技能名查表，**不要凭记忆判断**。
- 重新同步 / 更新某技能后，必须同步更新 README 该表格对应的 commit SHA、日期、备注，便于后续比对是否还需再更新。
- 从外部开源仓库引入新技能时，同样要在该表格登记。

- 顶层 `skills/` 与插件内 `bizdev-workflow/skills/` 是**两套**：前者可独立 `npx skills add`，后者随插件走。
- 插件不放 `hooks/`、`commands/`，复用顶层 `skills/` 模板。

## 怎么写一个技能（skill）

1. 新建 `skills/<kebab-name>/SKILL.md`，前置元数据：
   ```yaml
   ---
   name: <kebab-name>            # 必须与目录同名
   description: "[SP] Use when …" # 写触发场景，不是功能介绍
   disable-model-invocation: true # 默认关；除非需被 agent 自动感知
   ---
   ```
2. **默认 `disable-model-invocation: true`**——本仓库的基调是"不让 agent 自动感知"，显式调用才跑。需自动触发的 skill 才省略。
3. 正文只放导航与核心约束；大段资料拆到 `references/` 或 `examples/`，别塞进 `SKILL.md`。
4. 脚本放 `scripts/`，在 SKILL.md 里用**相对路径**调用，消费其输出（文件路径或 JSON），**不要内联脚本内容**。
5. 命名一律 kebab-case。

## 怎么改插件（bizdev-workflow）

- 流程编排在 `bizdev-workflow/skills/bizdev/SKILL.md`（状态机 + 模式路由）。
- **状态必须落用户级目录 `~/.bizdev/<project-key>/<session>.md`**（YAML frontmatter），**绝不写进仓库**；支持断点续跑、`full`/`fast` 模式。
- 执行派发 subagent，制品经 task brief / review package / ledger（`.superpowers/sdd/`）传递，避免上下文污染。
- 插件本身**不挂载 MCP**；钉钉/Confluence MCP 由运行时提供。
- 成员命名（marketplace 名 `agent-tools`、插件名 `bizdev-workflow`）若改动，需同步：marketplace.json 的 `name`、安装命令 `@x`、各文档引用。

## 必守约定

- **行尾 LF**：`.gitattributes` 强制 `eol=lf`（`*.sh`/`*.cmd`/`*.md`/`*.json`/`*.js`/`*.mjs`/`*.ts`）。`.cmd` 同时被 `cmd` 与 `bash` 解析，shell 脚本必须 LF——**禁止提交 CRLF**。
- Node 脚本 `.cjs`/`.js` **只用内置模块**，无 `package.json`、无 `npm install`。
- 改完技能/插件后，走一遍其触发流程做行为验证再交付（本仓库无自动化测试/CI）。
- 保留原作者归属；不要删他人 skill 的 license/copyright。
