## Setup Guide

Complete installation and configuration for Study Cards.

---

## Prerequisites

- **Claude Code CLI** installed and authenticated
- **O'Reilly account** (corporate or personal subscription)
- **Node.js** 18+ (for O'Reilly MCP server)
- **Git** (for cloning this repo)

**Optional:**
- **Obsidian** (for vault-based workflow)

---

## Quick Start (3 steps)

1. Clone this repo and navigate into it
2. Install and configure O'Reilly Read MCP server
3. Initialize your study directory

**Time:** 10-15 minutes

---

## Step 1: Clone Repository

```bash
git clone https://github.com/alexis-brignoni-126351/study-cards.git
cd study-cards
```

---

## Step 2: Install O'Reilly Read MCP Server

**This is required for `/study audit` and `/study deep` to work.** The O'Reilly Read MCP server pulls full chapter text from O'Reilly books.

### Install the server

```bash
npm install -g @barclayneira/oreilly-mcp
```

### Configure in Claude Code

Edit your Claude Code settings file:

**Location:**
- Project-specific: `study-cards/.claude/settings.json`
- Global (all projects): `~/.claude/settings.json`

**Add this MCP server config:**

```json
{
  "mcpServers": {
    "oreilly-read": {
      "command": "npx",
      "args": ["-y", "@barclayneira/oreilly-mcp"],
      "description": "O'Reilly full chapter text reader"
    },
    "oreilly": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-oreilly"],
      "description": "O'Reilly official search (fallback)"
    }
  }
}
```

**Why two servers?**
- `oreilly-read` (primary) - pulls full chapter text (required for audit/deep-dive)
- `oreilly` (official, fallback) - search and discovery only

### Authenticate O'Reilly

The server uses your O'Reilly session cookies for authentication.

**Steps:**

1. **Log in to O'Reilly** in your browser: https://learning.oreilly.com/
2. **Get your session cookie:**
   - Open browser DevTools (F12)
   - Go to Application > Cookies > https://learning.oreilly.com
   - Find cookie named `BrowserCookie` or `orm-jwt`
   - Copy the value
3. **Set environment variable:**

   **macOS/Linux:**
   ```bash
   export OREILLY_COOKIE="your-cookie-value-here"
   ```

   **Add to your shell profile** (~/.zshrc or ~/.bashrc) to persist:
   ```bash
   echo 'export OREILLY_COOKIE="your-cookie-value-here"' >> ~/.zshrc
   source ~/.zshrc
   ```

   **Windows (PowerShell):**
   ```powershell
   $env:OREILLY_COOKIE = "your-cookie-value-here"
   ```

   **Note:** Cookies expire. If `/study deep` stops working, refresh your cookie (repeat steps 1-3).

### Test the connection

Start a Claude Code session in this directory and run:

```bash
claude
```

Then test:
```
Can you search O'Reilly for "Managing AI Projects"?
```

If the server is working, Claude will find and list matching books/chapters.

**Troubleshooting:**
- If you get "MCP server not available": check settings.json syntax and server command
- If you get "authentication failed": refresh your O'Reilly cookie
- If chapters won't load: verify `oreilly-read` is installed globally (`npm list -g @barclayneira/oreilly-mcp`)

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

## Step 4: Copy Study Index Template

Initialize your Study Index from the template:

```bash
cp templates/study-index-template.md study/Study\ Index.md
```

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

This lets Claude Code read/write to your Obsidian vault directly.

```bash
npm install -g obsidian-mcp
```

Add to `.claude/settings.json`:

```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": ["-y", "obsidian-mcp", "/path/to/study-cards"],
      "description": "Obsidian vault integration"
    }
  }
}
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
│   ├── settings.json              # O'Reilly MCP config
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

- [ ] O'Reilly Read MCP server installed (`npm list -g @barclayneira/oreilly-mcp`)
- [ ] O'Reilly cookie environment variable set (`echo $OREILLY_COOKIE`)
- [ ] Claude Code can search O'Reilly (test in a session)
- [ ] `/study status` runs without errors
- [ ] `/study drill --limit 1` runs interactive tutor
- [ ] Study Index exists at `study/Study Index.md`

If all checks pass, you're ready to start building your card library.

---

## Common Issues

### "MCP server oreilly-read not available"

**Fix:** Check `.claude/settings.json` syntax. Make sure the server config is valid JSON. Restart Claude Code session.

### "Authentication failed" when pulling chapters

**Fix:** Your O'Reilly cookie expired. Log in to O'Reilly in browser, get fresh cookie, update `OREILLY_COOKIE` env var.

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
