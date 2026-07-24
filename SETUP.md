## Setup Guide

Complete installation and configuration for Study Cards.

---

## Prerequisites

- **Claude Code CLI** installed and authenticated
- **O'Reilly account** (corporate or personal subscription)
- **[uv](https://docs.astral.sh/uv/)** (runs the O'Reilly MCP server, which is Python-based)
- **Git** (for cloning this repo)

**Optional:**
- **Obsidian** (for vault-based workflow)

---

## Quick Start (3 steps)

1. Clone this repo and navigate into it
2. Install and register the O'Reilly MCP server
3. Initialize your study directory

**Time:** 10-15 minutes

---

## Step 1: Clone Repository

```bash
git clone https://github.com/alexis-brignoni-126351/study-cards.git
cd study-cards
```

---

## Step 2: Install the O'Reilly MCP Server

**This is required for `/study audit` and `/study deep` to work.** The [O'Reilly MCP server](https://github.com/barclayneira/oreilly-mcp) searches the catalog and pulls full chapter text from O'Reilly books.

### Get your O'Reilly API token

1. Log in to https://learning.oreilly.com/ in your browser
2. Copy your token from https://learning.oreilly.com/apidocs/mcp/content/

### Install and register with Claude Code

Clone the server somewhere outside this repo, install its dependencies, and register it at user scope (available in all your projects):

```bash
git clone https://github.com/barclayneira/oreilly-mcp.git
cd oreilly-mcp
uv sync

claude mcp add oreilly \
  -s user \
  -e ORM_JWT=YOUR_TOKEN_HERE \
  -- "$(pwd)/.venv/bin/python" "$(pwd)/stdio_server.py"
```

The server provides `search_content` (catalog search), `get_book_info`, `get_table_of_contents`, and `read_chapter` (full chapter text) — everything `/study audit` and `/study deep` need.

### Test the connection

Start a Claude Code session in this directory:

```bash
claude
```

Then test:
```
Can you search O'Reilly for "Managing AI Projects"?
```

If the server is working, Claude will find and list matching books/chapters.

**Troubleshooting:**
- If you get "MCP server not available": run `claude mcp list` to verify `oreilly` is registered, then restart Claude Code
- If chapters won't load: your token may have expired — grab a fresh one from the apidocs page and re-register the server

---

## Step 3: Initialize Study Directory

**Option A: Use existing structure (recommended)**

The repo already has the directory structure. Just start using it:

```bash
# You're already in study-cards/
# Ready to go!
```

**Option B: Custom location**

If you want to use a different location:

1. Copy the `study/` folder to your preferred location
2. Update `.claude/settings.json` with custom path:

```json
{
  "studyDir": "/path/to/your/study/directory"
}
```

---

## Step 4: Customize Your Study Index

The repo ships a ready-to-use Study Index at `study/Study Index.md` (a fresh copy of the template is in `templates/study-index-template.md` if you ever want to start over).

Edit `study/Study Index.md` and customize:
- Prerequisites Map (which sessions require which concepts)
- Your session schedule
- Your learning goals

---

## Step 5: (Optional) Configure Book Filters

If you want to limit O'Reilly searches to specific books (like program-assigned texts), create a config file:

```bash
cp .claude/study-config.json.example .claude/study-config.json
```

Edit `.claude/study-config.json`:
```json
{
  "oreilly_books": [
    "Managing AI Projects",
    "Generative AI on Microsoft Azure"
  ]
}
```

Now `/study deep` and `/study audit` only search these books.

**Skip this if you want to search the entire O'Reilly catalog.**

---

## Step 6: First Study Session

Test the system with the example cards:

```bash
claude
```

Inside Claude Code:
```
/study status
```

You should see 3 new cards ready to drill. Then try:
```
/study drill --limit 1
```

This runs the interactive tutor on one card. Follow the prompts.

**If this works, you're ready!**

---

## Optional: Obsidian Integration

If you use Obsidian, you can open this directory as a vault for better linking and navigation.

### Install Obsidian MCP (optional)

This lets Claude Code read/write to your Obsidian vault directly. Requires Node.js 18+.

```bash
claude mcp add obsidian -s user -- npx -y obsidian-mcp /path/to/study-cards
```

**Benefits:**
- Wikilink navigation (`[[Card Name]]` clickable)
- Graph view of concept connections
- Better markdown preview

**Not required.** The framework works with plain files.

---

## Directory Structure

After setup, your directory looks like this:

```
study-cards/
├── .claude/
│   ├── settings.json              # Optional skill settings (e.g. custom studyDir)
│   └── skills/study/
│       └── skill.md               # The /study skill
│
├── study/
│   ├── Study Index.md             # Your master index
│   ├── concepts/                  # Concept cards (drill these)
│   │   ├── bias-variance-tradeoff.md
│   │   ├── rag-chunking-strategy.md
│   │   └── model-selection-framework.md
│   ├── sessions/                  # Session notes (optional)
│   └── logs/                      # Auto-generated drill/audit logs
│       ├── drill-log.md
│       ├── audit-log.md
│       └── revision-log.md
│
├── templates/
│   ├── concept-card-template.md
│   ├── study-index-template.md
│   └── session-note-template.md
│
├── SETUP.md                       # This file
├── GUIDE.md                       # How to use the system
└── README.md                      # Overview
```

---

## Verification Checklist

Before your first real study session, verify:

- [ ] O'Reilly MCP server registered (`claude mcp list` shows `oreilly`)
- [ ] O'Reilly API token set (the `ORM_JWT` value passed to `claude mcp add`)
- [ ] Claude Code can search O'Reilly (test in a session)
- [ ] `/study status` runs without errors
- [ ] `/study drill --limit 1` runs interactive tutor
- [ ] Study Index exists at `study/Study Index.md`

If all checks pass, you're ready to start building your card library.

---

## Common Issues

### "MCP server oreilly not available"

**Fix:** Run `claude mcp list` and confirm `oreilly` is registered and healthy. If missing, re-run the `claude mcp add` command from Step 2. Restart Claude Code session.

### "Authentication failed" when pulling chapters

**Fix:** Your O'Reilly API token likely expired. Get a fresh one from https://learning.oreilly.com/apidocs/mcp/content/ and re-register the server with the new `ORM_JWT` value.

### Cards not showing up in `/study due`

**Fix:** Cards need `mastery: new` in frontmatter and `status: active`. Check example cards for correct format.

### `/study audit` can't find session notes

**Fix:** Session notes must be in `study/sessions/` and named `Session N - Topic.md`. Or provide full path when prompted.

---

## Next Steps

1. Read [GUIDE.md](GUIDE.md) - learn the full workflow
2. Create your first card from a real session/chapter
3. Run `/study audit session-1` to practice coverage audits
4. Set up a daily drill habit (`/study` every morning)

---

## Support

- **Issues:** [GitHub Issues](https://github.com/alexis-brignoni-126351/study-cards/issues)
- **Questions:** Ask in the AIUP cohort channel
- **Updates:** Pull latest changes with `git pull origin main`
