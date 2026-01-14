# ShinCode Dev Skills

A curated collection of Claude Code skills for modern web development.

## Available Skills

| Skill | Description | Command |
|-------|-------------|---------|
| [new-webapp](./skills/new-webapp/) | Interactive web project setup with framework, BaaS, auth, and payment selection | `/new-webapp` |

## Installation

### Option 1: Manual Installation (Single Skill)

1. Copy the desired skill folder to your Claude Code skills directory:

```bash
# macOS/Linux
cp -r skills/new-webapp ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse skills/new-webapp $env:USERPROFILE/.claude/skills/
```

2. Restart Claude Code

### Option 2: Clone Entire Repository

```bash
# Clone to your global skills directory
git clone https://github.com/YOUR_USERNAME/shincode-dev-skills.git ~/.claude/skills/shincode-dev-skills
```

### Option 3: Project-Specific Installation

Copy skills to your project's `.claude/skills/` directory for project-specific use.

## Usage

After installation, invoke skills using their command:

```
/new-webapp
```

Claude will guide you through an interactive setup process.

## Contributing

1. Fork this repository
2. Create your skill in `skills/your-skill-name/SKILL.md`
3. Add your skill to `registry.json`
4. Submit a pull request

## Skill Structure

Each skill follows this structure:

```
skills/
└── skill-name/
    └── SKILL.md    # Required: Skill definition with YAML metadata
```

### SKILL.md Format

```markdown
---
name: skill-name
description: Brief description of what this skill does
trigger: When should Claude use this skill
---

# Instructions for Claude

Your skill instructions here...
```

## License

MIT
