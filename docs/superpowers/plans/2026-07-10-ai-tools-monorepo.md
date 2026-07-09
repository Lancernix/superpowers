# AI Tools Monorepo Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use subagent-driven-development (recommended) or executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 superpowers fork 的技能、myflow plugin 整合到一个 git 仓库中，支持 `npx skills` 安装独立 skills，支持 Claude Code marketplace 分发 myflow plugin。

**Architecture:** 单仓库双体系：根目录 `skills/` 放独立可安装 skills，`plugins/myflow/` 放 Claude plugin，通过 `.claude-plugin/marketplace.json` 声明 marketplace 源。两个体系通过不同工具链安装，互不干扰。

**Tech Stack:** Git, `npx skills` (agentskills CLI), Claude Code plugin system, `git-subdir` marketplace source

## Global Constraints

- 保持现有 skills 内容不变，仅调整目录结构
- myflow plugin 从 `~/code/myflow/bizdev-workflow/` 迁移到 `plugins/myflow/`
- 独立 skills 命名空间保持原样，不添加前缀
- plugin skills 通过 plugin 命名空间自动隔离
- 所有路径使用相对路径，以 `./` 开头
- `npx skills` 只处理根目录 `skills/`，不进入 `plugins/`

---

### Task 1: 创建新目录结构

**Files:**
- Create: `plugins/myflow/.claude-plugin/plugin.json`
- Create: `plugins/myflow/skills/`
- Create: `plugins/myflow/agents/`
- Create: `plugins/myflow/hooks/`
- Create: `.claude-plugin/marketplace.json`

**Interfaces:**
- Consumes: 无
- Produces: 新目录结构，为后续迁移做准备

- [ ] **Step 1: 创建 plugins/myflow 目录结构**

```bash
mkdir -p plugins/myflow/.claude-plugin
mkdir -p plugins/myflow/skills
mkdir -p plugins/myflow/agents
mkdir -p plugins/myflow/hooks
mkdir -p .claude-plugin
```

- [ ] **Step 2: 创建 myflow plugin.json**

```json
{
  "name": "myflow",
  "description": "myflow bizdev workflow plugin",
  "version": "1.0.0"
}
```

- [ ] **Step 3: 创建 marketplace.json**

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

- [ ] **Step 4: 验证 JSON 格式**

```bash
python3 -c "import json; json.load(open('plugins/myflow/.claude-plugin/plugin.json')); json.load(open('.claude-plugin/marketplace.json')); print('ok')"
```

- [ ] **Step 5: 提交目录结构**

```bash
git add plugins/myflow/.claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "chore: create ai-tools monorepo structure with myflow plugin scaffold"
```

### Task 2: 迁移 myflow plugin 内容

**Files:**
- Copy: `~/code/myflow/bizdev-workflow/skills/*` → `plugins/myflow/skills/`
- Copy: `~/code/myflow/bizdev-workflow/agents/*` → `plugins/myflow/agents/`
- Copy: `~/code/myflow/bizdev-workflow/hooks/*` → `plugins/myflow/hooks/`
- Copy: `~/code/myflow/bizdev-workflow/.bizdev-build-ledger.md` → `plugins/myflow/`
- Copy: `~/code/myflow/bizdev-workflow/README.md` → `plugins/myflow/`

**Interfaces:**
- Consumes: Task 1 创建的目录结构
- Produces: 完整的 myflow plugin 内容

- [ ] **Step 1: 复制 skills 目录**

```bash
cp -r ~/code/myflow/bizdev-workflow/skills/* plugins/myflow/skills/
```

- [ ] **Step 2: 复制 agents 目录**

```bash
cp -r ~/code/myflow/bizdev-workflow/agents/* plugins/myflow/agents/
```

- [ ] **Step 3: 复制 hooks 目录**

```bash
cp -r ~/code/myflow/bizdev-workflow/hooks/* plugins/myflow/hooks/
```

- [ ] **Step 4: 复制其他文件**

```bash
cp ~/code/myflow/bizdev-workflow/.bizdev-build-ledger.md plugins/myflow/
cp ~/code/myflow/bizdev-workflow/README.md plugins/myflow/
```

- [ ] **Step 5: 验证 myflow plugin 结构**

```bash
ls -la plugins/myflow/
ls -la plugins/myflow/skills/
ls -la plugins/myflow/agents/
```

- [ ] **Step 6: 提交 myflow 内容**

```bash
git add plugins/myflow/
git commit -m "feat: migrate myflow plugin into ai-tools monorepo"
```

### Task 3: 更新根目录 package.json

**Files:**
- Modify: `package.json`

**Interfaces:**
- Consumes: 无
- Produces: 更新后的 package.json，反映新仓库身份

- [ ] **Step 1: 更新 package.json**

```json
{
  "name": "ai-tools",
  "version": "1.0.0",
  "description": "AI tools: skills and plugins for coding agents",
  "private": true,
  "keywords": ["skills", "plugins", "ai", "claude-code"],
  "pi": {
    "extensions": [],
    "skills": ["./skills"]
  }
}
```

- [ ] **Step 2: 验证 package.json**

```bash
python3 -c "import json; json.load(open('package.json')); print('ok')"
```

- [ ] **Step 3: 提交更新**

```bash
git add package.json
git commit -m "chore: update package.json for ai-tools monorepo"
```

### Task 4: 创建 README.md

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: 无
- Produces: 仓库 README，说明结构和安装方式

- [ ] **Step 1: 创建 README.md**

```markdown
# AI Tools

AI tools: skills and plugins for coding agents.

## Structure

- `skills/` - Independent skills installable via `npx skills`
- `plugins/myflow/` - Claude Code plugin for bizdev workflow

## Installation

### Independent Skills

```bash
npx skills add https://github.com/you/ai-tools.git
```

### myflow Plugin

```bash
/plugin marketplace add https://github.com/you/ai-tools.git
/plugin install myflow@my-tools
```

## Development

This is a monorepo containing multiple AI tools.

- Skills are in `skills/`
- Plugins are in `plugins/`
```

- [ ] **Step 2: 提交 README**

```bash
git add README.md
git commit -m "docs: add README for ai-tools monorepo"
```

### Task 5: 验证安装流程

**Files:**
- 无文件变更，仅验证

**Interfaces:**
- Consumes: Task 1-4 完成的结构
- Produces: 验证报告

- [ ] **Step 1: 验证 npx skills 发现**

```bash
npx skills add . --list
```

Expected: 列出根目录 `skills/` 下的所有 skills

- [ ] **Step 2: 验证 myflow plugin 结构**

```bash
ls -la plugins/myflow/.claude-plugin/
cat plugins/myflow/.claude-plugin/plugin.json
```

Expected: `plugin.json` 存在且有效

- [ ] **Step 3: 验证 marketplace 配置**

```bash
cat .claude-plugin/marketplace.json | python3 -m json.tool
```

Expected: JSON 格式正确，`git-subdir` 源指向 `plugins/myflow`

- [ ] **Step 4: 提交验证结果**

```bash
git add -A
git commit -m "chore: verify ai-tools monorepo installation flow"
```
