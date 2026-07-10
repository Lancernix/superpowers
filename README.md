# AI Tools

AI tools: skills and plugins for coding agents.

## Structure

- `skills/` - Independent skills installable via `npx skills`
- `plugins/myflow/` - Claude Code plugin for bizdev workflow

## Installation

### Independent Skills

```bash
# Local path
npx skills add ./skills

# Remote (GitHub)
npx skills add https://github.com/Lancernix/agent-tools/tree/master/skills
```

### myflow Plugin

```bash
# Add marketplace (once)
/plugin marketplace add git@github.com:Lancernix/agent-tools.git

# Install plugin (once)
/plugin install myflow@my-tools
```

## Development

This is a monorepo containing multiple AI tools.

- Skills are in `skills/`
- Plugins are in `plugins/`
