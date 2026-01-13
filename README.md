# 23blocks AI Agents Marketplace

Official Claude Code marketplace for 23blocks platform plugins. One plugin per block.

## Structure

```
ai-agents-mono/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace definition
├── plugins/
│   ├── forms-block/              # Forms Block Plugin
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── agents/
│   │   │   └── forms.md
│   │   ├── skills/
│   │   │   ├── schemas-api/
│   │   │   │   └── SKILL.md
│   │   │   ├── schemas-sdk/
│   │   │   │   └── SKILL.md
│   │   │   └── validation-api/
│   │   │       └── SKILL.md
│   │   ├── commands/
│   │   └── hooks/
│   ├── auth-block/               # Auth Block Plugin
│   ├── data-block/               # Data Block Plugin
│   └── storage-block/            # Storage Block Plugin
├── pro-plugins/                  # Pro-only plugins (not in community)
├── docs/
│   └── agent-feature-guide.md    # Guide for block teams
└── scripts/
    └── build.sh
```

## Installation

```bash
# Add 23blocks marketplace
claude plugin marketplace add https://github.com/23blocks-OS/ai-agents

# List available plugins
claude plugin marketplace list

# Install the blocks you need
claude plugin install forms-block
claude plugin install auth-block
claude plugin install data-block
```

## For Block Teams

See [docs/agent-feature-guide.md](docs/agent-feature-guide.md) for complete instructions on creating your block's plugin.

**Quick Start:**
1. Create your plugin directory: `plugins/your-block/`
2. Add plugin manifest: `.claude-plugin/plugin.json`
3. Add your agent: `agents/your-block.md`
4. Add skills: `skills/feature-api/SKILL.md`, `skills/feature-sdk/SKILL.md`
5. Submit PR

## Distribution

| Repo | Visibility | Content |
|------|------------|---------|
| [23blocks-OS/ai-agents](https://github.com/23blocks-OS/ai-agents) | Public | All standard plugins |
| [23blocks/ai-agents-pro](https://github.com/23blocks/ai-agents-pro) | Private | All plugins + pro-only |

## Development

### Local Build
```bash
./scripts/build.sh
```

### CI/CD
On push to `main`, GitHub Actions:
1. Builds community and pro distributions
2. Syncs to respective distribution repos

## License

- Standard plugins: MIT
- Pro plugins: Proprietary
