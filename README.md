# Ralph Fleet

A Ralph Loop that installs Ralph Loops.

Fleet orchestrator for managing Ralph infrastructure across all your GitHub repos.

## The Concept

```
┌─────────────────────────────────────────────────────────────┐
│                     ralph-fleet (this repo)                  │
│                                                              │
│   AGENTS.md + prd.json + GitHub API access                  │
│                                                              │
│   Commands:                                                  │
│   - scan     → list repos, detect Ralph status              │
│   - status   → fleet overview dashboard                     │
│   - install  → add Ralph infrastructure to repo             │
│   - sync     → push config updates across fleet             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
        ┌──────────┬──────────┬──────────┬──────────┐
        │  repo-a  │  repo-b  │  repo-c  │  repo-d  │
        │          │          │          │          │
        │ AGENTS.md│ AGENTS.md│ (none)   │ AGENTS.md│
        │ prd.json │ prd.json │          │ prd.json │
        └──────────┴──────────┴──────────┴──────────┘
                                   │
                                   ▼
                            `install repo-c`
                                   │
                                   ▼
                              repo-c now has
                              Ralph installed
```

## Why?

1. **Standardize** - Same Ralph infrastructure across all repos
2. **Orchestrate** - One place to manage fleet configuration
3. **Scale** - Install Ralph on 50 repos in one command
4. **Dogfood** - This repo itself runs as a Ralph Loop

## Quick Start

```bash
# Clone
git clone https://github.com/YOUR_USERNAME/ralph-fleet
cd ralph-fleet

# Setup
bun install
cp .env.example .env
# Add your GITHUB_TOKEN

# Scan your repos
bun run scan --days=30

# Install Ralph on a repo
bun run install owner/repo-name

# Install on all repos updated in last 30 days
bun run install --all --days=30
```

## What Gets Installed

When you run `install`, target repos receive:

```
repo/
├── AGENTS.md                    # Ralph instructions
└── scripts/
    └── ralph/
        ├── prd.json            # Task tracking
        └── progress.txt        # Learnings log
```

## This Repo is a Ralph Loop

ralph-fleet itself uses the Ralph methodology:

- `AGENTS.md` - Instructions for working on this repo
- `scripts/ralph/prd.json` - Stories to build the orchestrator
- `scripts/ralph/progress.txt` - What we've learned

To contribute or extend, just pick the next pending story from `prd.json`.

## Philosophy

> "As model quality, context, and speed race to commodity, the only moat left is the harness - your verification gates."

The loop compresses to nothing. The harness stays. Ralph Fleet helps you install that harness everywhere.

## License

MIT
