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
