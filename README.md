# ShinCode Dev Skills

A curated collection of Claude Code skills for modern web development.

## Available Skills

| Skill | Description | Command |
|-------|-------------|---------|
| [new-webapp](./skills/new-webapp/) | Interactive web project setup with framework, BaaS, auth, and payment selection | `/new-webapp` |

## Installation

### Option 1: Marketplace (Recommended)

```bash
# 1. Add marketplace
/plugin marketplace add Shin-sibainu/shincode-dev-skills

# 2. Install skill
/plugin install new-webapp@shincode-dev-skills
```

### Option 2: Download .skill file

```bash
# macOS/Linux
curl -L https://github.com/Shin-sibainu/shincode-dev-skills/raw/master/new-webapp.skill -o ~/.claude/skills/new-webapp.skill

# Windows (PowerShell)
Invoke-WebRequest -Uri "https://github.com/Shin-sibainu/shincode-dev-skills/raw/master/new-webapp.skill" -OutFile "$env:USERPROFILE\.claude\skills\new-webapp.skill"
```

### Option 3: Manual copy

```bash
git clone https://github.com/Shin-sibainu/shincode-dev-skills.git
cp -r shincode-dev-skills/skills/new-webapp ~/.claude/skills/
```

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
