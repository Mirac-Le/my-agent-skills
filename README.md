# My Agent Skills

Personal Agent Skills collection for reusable development workflows, database patterns, and coding standards across projects.

## 📦 Available Skills

| Skill | Description | Use Case |
|-------|-------------|----------|
| `postgres-client` | High-performance PostgreSQL client patterns | psycopg3 + ConnectorX + Polars |
| `git-workflow` | Git workflow conventions | Conventional Commits, PR workflow |
| `coding-standards` | Coding standards and interaction preferences | Code style, debugging, review priorities |

## 🚀 Installation

### Option 1: Claude Code Plugin (Recommended)

```bash
# Add as Plugin Marketplace
/plugin marketplace add Mirac-Le/my-agent-skills

# Install
/plugin install my-agent-skills@Mirac-Le
```

### Option 2: Copy to Project

```bash
# Clone repository
git clone https://github.com/Mirac-Le/my-agent-skills.git

# Copy required skills to your project
cp -r my-agent-skills/skills/postgres-client your-project/.claude/skills/
```

### Option 3: Git Submodule

```bash
# Add as submodule in your project
git submodule add https://github.com/Mirac-Le/my-agent-skills.git .claude/vendor/my-skills

# Reference in AGENTS.md
```

## 📁 Directory Structure

```
my-agent-skills/
├── .claude-plugin/
│   └── plugin.json          # Claude Code Plugin config
├── skills/
│   ├── postgres-client/     # PostgreSQL client patterns
│   │   ├── SKILL.md
│   │   └── references/
│   ├── git-workflow/        # Git workflow
│   │   └── SKILL.md
│   └── coding-standards/    # Coding standards
│       └── SKILL.md
├── vendor/                  # Third-party skills (optional)
│   └── anthropic/           # anthropic/skills submodule
├── template/                # Skill template
│   └── SKILL.md
└── README.md
```

## 🔄 Sync with Anthropic Official Skills

To use skills from [anthropic/skills](https://github.com/anthropics/skills):

```bash
# Add as submodule
git submodule add https://github.com/anthropics/skills.git vendor/anthropic

# Update to latest
git submodule update --remote vendor/anthropic
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

3. Update `.claude-plugin/plugin.json` to add the new skill path

4. Commit and push

## 🔧 Usage in Projects

### Reference in AGENTS.md

```xml
<skill>
<name>postgres-client</name>
<description>High-performance PostgreSQL client patterns...</description>
<location>https://github.com/Mirac-Le/my-agent-skills</location>
</skill>
```

### Direct Copy to .claude/skills/

Copy required skills to your project's `.claude/skills/` directory. They will be automatically recognized.

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

## 🤝 Contributing

1. Fork this repository
2. Create a new branch: `git checkout -b feat/new-skill`
3. Commit changes: `git commit -m "feat: add new skill"`
4. Push: `git push origin feat/new-skill`
5. Create Pull Request

## 📜 License

MIT License
