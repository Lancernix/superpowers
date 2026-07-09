# AI Tools Monorepo 整合设计

## 1. 目标

把 superpowers fork 的技能、myflow plugin，以及后续个人技能统一在一个 git 仓库里管理，满足：
- 一个仓库维护所有 AI 工具
- 独立 skills 可通过 `npx skills` 安装
- myflow 作为 Claude plugin 可通过 marketplace 分发
- 结构清晰，互不干扰

## 2. 仓库结构

```
ai-tools/
├── skills/                          # 独立 skills，npx skills 安装
│   ├── brainstorming/
│   ├── writing-plans/
│   ├── systematic-debugging/
│   └── personal-my-custom/
├── plugins/
│   └── myflow/                      # Claude plugin
│       ├── .claude-plugin/
│       │   └── plugin.json          # name: "myflow"
│       ├── skills/                  # plugin 内部 skills
│       ├── agents/
│       └── hooks/
├── .claude-plugin/
│   └── marketplace.json             # 远程 marketplace 声明
├── package.json
└── README.md
```

## 3. 各层职责

| 目录 | 职责 | 安装方式 |
|------|------|----------|
| `skills/` | 独立可安装 skills | `npx skills add <repo-url>` |
| `plugins/myflow/` | Claude plugin | `--plugin-dir` 或 marketplace |
| `.claude-plugin/marketplace.json` | 声明仓库内 plugin 源 | `/plugin marketplace add <repo-url>` |

## 4. 安装流程

### 独立 skills

```bash
npx skills add https://github.com/you/ai-tools.git
```

`npx skills` 会扫描仓库根目录的 `skills/`，发现所有 `SKILL.md` 并安装。

### myflow plugin

```bash
# 1. 添加 marketplace（只需一次）
/plugin marketplace add https://github.com/you/ai-tools.git

# 2. 安装 myflow plugin（只需一次）
/plugin install myflow@my-tools
```

marketplace 使用 `git-subdir` 源类型，指向 `plugins/myflow` 子目录。

## 5. marketplace 配置

`.claude-plugin/marketplace.json`：

```json
{
  "name": "my-tools",
  "owner": {
    "name": "Your Name"
  },
  "plugins": [
    {
      "name": "myflow",
      "source": {
        "source": "git-subdir",
        "url": "https://github.com/you/ai-tools.git",
        "path": "plugins/myflow"
      },
      "description": "myflow bizdev workflow"
    }
  ]
}
```

关键点：
- `source` 使用 `git-subdir` 类型，支持从 monorepo 子目录安装
- `path` 指向 `plugins/myflow`
- `url` 是远程仓库地址

## 6. myflow plugin 配置

`plugins/myflow/.claude-plugin/plugin.json`：

```json
{
  "name": "myflow",
  "description": "myflow bizdev workflow plugin",
  "version": "1.0.0"
}
```

可选：如果 plugin 的 skills/agents/hooks 不在根目录，可通过字段声明：
```json
{
  "name": "myflow",
  "skills": "./skills/",
  "agents": "./agents/",
  "hooks": "./hooks/hooks.json"
}
```

## 7. 边界与约束

- `npx skills` 只处理根目录 `skills/`，不进入 `plugins/`
- Claude plugin 只处理 `plugins/myflow/` 及其声明路径，不读取根目录 `skills/`
- 两个体系通过不同工具链安装，互不干扰
- superpowers fork 的技能如需要更新，手动合并到 `skills/` 对应目录
