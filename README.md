# My Agent Skills

Personal Agent Skills collection for reusable development workflows, database patterns, and coding standards across projects.

## 📦 Available Skills

| Skill | Description | Use Case |
|-------|-------------|----------|
| `postgres-client` | High-performance PostgreSQL client patterns | psycopg3 + ConnectorX + Polars |
| `git-workflow` | Git workflow conventions | Conventional Commits, PR workflow |
| `coding-standards` | Coding standards and interaction preferences | Code style, debugging, review priorities |

## 🚀 Quick Start (Cursor + openskills)

### 1. Install openskills

```bash
npm i -g openskills
```

### 2. Install Skills to Your Project

```bash
cd your-project

# Install from this repo
openskills install Mirac-Le/my-agent-skills

# Or install official Anthropic skills
openskills install anthropics/skills
```

### 3. Sync to AGENTS.md

```bash
# Update AGENTS.md with installed skills
openskills sync
```

Now Cursor's Claude can use the installed skills automatically!

## 📁 Directory Structure

```
my-agent-skills/
├── .claude-plugin/
│   └── plugin.json          # Claude Code Plugin config
├── .github/
│   └── workflows/
│       └── sync-submodules.yml  # Auto-sync anthropic/skills daily
├── skills/
│   ├── anthropic/           # Official Anthropic skills (submodule, auto-synced)
│   │   └── skills/
│   │       ├── docx/
│   │       ├── pdf/
│   │       ├── pptx/
│   │       ├── xlsx/
│   │       └── ...
│   ├── postgres-client/     # Custom: PostgreSQL client patterns
│   │   ├── SKILL.md
│   │   └── references/
│   ├── git-workflow/        # Custom: Git workflow conventions
│   │   └── SKILL.md
│   └── coding-standards/    # Custom: Coding standards
│       └── SKILL.md
├── template/
│   └── SKILL.md
└── README.md
```

## 🔄 Alternative Installation Methods

### Option 1: Git Submodule

```bash
cd your-project

# Add as submodule
git submodule add git@github.com:Mirac-Le/my-agent-skills.git .claude/vendor/my-skills

# Link specific skills
ln -s vendor/my-skills/skills/postgres-client .claude/skills/postgres-client
```

### Option 2: Direct Copy

```bash
# Clone and copy what you need
git clone git@github.com:Mirac-Le/my-agent-skills.git /tmp/my-skills
cp -r /tmp/my-skills/skills/postgres-client your-project/.claude/skills/
```

### Option 3: Claude Code Plugin

```bash
/plugin marketplace add Mirac-Le/my-agent-skills
/plugin install my-agent-skills@Mirac-Le
```

## ✨ Creating a New Skill

1. Copy template:
   ```bash
   cp -r template skills/my-new-skill
   ```

2. Edit `skills/my-new-skill/SKILL.md`:
   ```yaml
   ---
   name: my-new-skill
   description: Describe what this skill does and when to use it.
   ---

   # My New Skill

   ## Instructions
   ...
   ```

3. Commit and push:
   ```bash
   git add .
   git commit -m "feat: add my-new-skill"
   git push
   ```

## 📋 Skill Format Specification

Each Skill must contain:

```yaml
---
name: skill-name              # Required: lowercase, hyphen-separated
description: ...              # Required: describe purpose and trigger conditions
allowed-tools:                # Optional: restrict available tools
  - Read
  - Write
  - Bash
---

# Skill Title

## Instructions
...
```

## 🔄 Auto-Sync with Anthropic Skills

This repo includes `skills/anthropic` as a Git submodule pointing to [anthropic/skills](https://github.com/anthropics/skills).

- **Auto-sync**: GitHub Actions runs daily to pull latest updates
- **Manual sync**: 
  ```bash
  git submodule update --remote skills/anthropic
  git add skills/anthropic
  git commit -m "chore: sync anthropic skills"
  git push
  ```

## 🤝 Contributing

1. Fork this repository
2. Create a new branch: `git checkout -b feat/new-skill`
3. Commit changes: `git commit -m "feat: add new skill"`
4. Push: `git push origin feat/new-skill`
5. Create Pull Request

## 📜 License

MIT License
