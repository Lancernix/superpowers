# AI 工具集

面向编码 agent 的技能（skills）与插件（plugins）集合。

## 目录结构

- `skills/` - 独立技能，可通过 `npx skills` 安装
- `bizdev-workflow/` - bizdev 业务开发工作流的 Claude Code 插件

## 已同步的开源技能

本节记录本仓库引入（vendored）的每个开源技能的同步版本，便于后续比对是否需要更新。重新同步时请同步更新下方的 commit SHA / 日期 与 备注。

| 技能 | 来源仓库 | 上游 commit | 同步日期 | 备注 |
|------|----------|-------------|----------|------|
| `ui-ux-pro-max` | [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | `0468898`（2026-07-13） | 2026-07-13 | 新增 `disable-model-invocation: true`；保留上游 `LICENSE`（MIT © 2024 Next Level Builder）；`scripts/search.py`、`data/motion.csv` 由 CRLF 转 LF |
| `brainstorming`、`writing-plans`、`writing-skills`、`using-superpowers`、`verification-before-completion`、`test-driven-development`、`using-git-worktrees`、`systematic-debugging`、`subagent-driven-development`、`executing-plans`、`requesting-code-review`、`receiving-code-review`、`dispatching-parallel-agents`、`finishing-a-development-branch`（共 14 个） | [obra/superpowers](https://github.com/obra/superpowers) | `d884ae0`（2026-07-02，Release v6.1.1） | 2026-07-13 | 与上游 `main` 正文内容逐文件比对一致；仅按本仓库约定在各 `SKILL.md` frontmatter 新增 `disable-model-invocation: true`，其余文件无改动 |

## 安装

### 独立技能

```bash
# 本地路径
npx skills add ./skills

# 远程（GitHub）
npx skills add https://github.com/Lancernix/agent-tools/tree/master/skills
```

### bizdev-workflow 插件

```bash
claude --plugin-dir ./bizdev-workflow
```

### 通过 Marketplace 分发安装

```bash
# 添加市场（一次性）
/plugin marketplace add git@github.com:Lancernix/agent-tools.git
# 安装插件（一次性）
/plugin install bizdev-workflow@agent-tools
```

## 开发

这是一个包含多个 AI 工具（skills / plugins）的仓库。

- 技能位于 `skills/`
- 插件位于 `bizdev-workflow/`
