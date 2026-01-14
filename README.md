# My Agent Skills

All-in-one Agent Skills collection: custom development workflows + official Anthropic skills.

**One install, all skills.**

## 🚀 Quick Start (Cursor + openskills)

```bash
# 1. Install openskills (once)
npm i -g openskills

# 2. Install all skills to your project
cd your-project
openskills install Mirac-Le/my-agent-skills

# 3. Sync to AGENTS.md
openskills sync
```

Done! Cursor's Claude can now use all skills.

## 📦 Available Skills

### Custom Skills

| Skill | Description |
|-------|-------------|
| `postgres-client` | High-performance PostgreSQL patterns (psycopg3 + ConnectorX + Polars) |
| `git-workflow` | Git conventions (Conventional Commits, PR workflow) |
| `coding-standards` | Coding standards and interaction preferences |

### Official Anthropic Skills (in `skills/anthropic/`, auto-synced daily)

| Skill | Description |
|-------|-------------|
| `docx` | Word document creation and editing |
| `pdf` | PDF manipulation and form filling |
| `pptx` | PowerPoint presentation creation |
| `xlsx` | Excel spreadsheet operations |
| `frontend-design` | Production-grade frontend interfaces |
| `mcp-builder` | MCP server development guide |
| `webapp-testing` | Web app testing with Playwright |
| `skill-creator` | Guide for creating new skills |
| ... | And more! |

## 📁 Directory Structure

```
my-agent-skills/
├── .anthropic-source/       # Submodule (hidden, for CI sync)
├── skills/
│   ├── postgres-client/     # Custom (marked with .custom)
│   ├── git-workflow/        # Custom
│   ├── coding-standards/    # Custom
│   └── anthropic/           # Official Anthropic skills
│       ├── docx/
│       ├── pdf/
│       ├── pptx/
│       ├── xlsx/
│       └── ...
└── .github/workflows/
    └── sync-submodules.yml  # Auto-sync daily
```

## 🔄 Auto-Sync

GitHub Actions automatically syncs official Anthropic skills daily:
- Updates `.anthropic-source` submodule
- Copies new/updated skills to `skills/anthropic/` directory

## ✨ Creating a New Custom Skill

1. Create skill directory:
   ```bash
   mkdir skills/my-new-skill
   touch skills/my-new-skill/.custom  # Mark as custom (won't be overwritten)
   ```

2. Create `skills/my-new-skill/SKILL.md`:
   ```yaml
   ---
   name: my-new-skill
   description: What this skill does and when to use it.
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

## 🔧 Alternative Installation

### Git Submodule

```bash
git submodule add git@github.com:Mirac-Le/my-agent-skills.git .claude/vendor/my-skills
```

### Direct Copy

```bash
git clone git@github.com:Mirac-Le/my-agent-skills.git /tmp/skills
cp -r /tmp/skills/skills/postgres-client your-project/.claude/skills/
```

## 📜 License

- Custom skills: MIT License
- Anthropic skills: See individual LICENSE files
