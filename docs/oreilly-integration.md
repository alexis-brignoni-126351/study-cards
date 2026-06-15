# O'Reilly Integration

How the framework integrates with O'Reilly Learning to pull chapter content and create cards.

---

## Why This Matters

The AI Upskilling Program assigns O'Reilly readings alongside sessions. These readings go deeper than session material and often contain failure modes, tradeoffs, and production patterns worth cardifying.

**Without this integration:** You'd manually read, take notes, create cards. Slow, error-prone.

**With this integration:** Pull full chapter text, review against Card Inclusion Bar, create cards in one step.

---

## Two MCP Servers

The framework uses **two** O'Reilly MCP servers:

### 1. `oreilly-read` (Primary)

**Repository:** [barclayneira/oreilly-mcp](https://github.com/barclayneira/oreilly-mcp)  
**What it does:** Pulls **full chapter text** from O'Reilly books  
**Tools:**
- `read_chapter` - returns complete chapter content
- `get_table_of_contents` - lists book chapters
- `get_book_info` - book metadata

**Used by:**
- `/study audit` - pulls assigned chapters to find concepts
- `/study deep` - pulls chapters for deep-dives

**Critical:** This is the server that makes coverage audits work. Without it, you can't pull chapter text.

### 2. `oreilly` (Official, Fallback)

**Repository:** [@modelcontextprotocol/server-oreilly](https://github.com/modelcontextprotocol/servers/tree/main/src/oreilly)  
**What it does:** **Discovery only** - searches catalog, returns metadata  
**Tools:**
- `search_oreilly_content` - search books/chapters/videos

**Used by:**
- `/study deep` - find chapters matching a topic
- Fallback when `oreilly-read` unavailable

**Limitation:** Can't pull chapter text. Returns URLs and metadata only.

---

## Setup (Quick Reference)

**Full setup in [SETUP.md](../SETUP.md). Summary:**

### Configure

The repo includes the config at `.claude/settings.json` — no separate install step needed. Claude Code downloads and runs the servers automatically via `npx`.

To make it available globally, copy the MCP block to `~/.claude/settings.json`:

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

### Authenticate

Just be logged in to O'Reilly in your browser. The MCP server picks up your session automatically — no cookie extraction needed.

---

## How `/study audit` Uses O'Reilly

When you run `/study audit session-N`, the system:

1. **Reads your session note** to find assigned O'Reilly readings
   
   Example from session note:
   ```markdown
   **O'Reilly Reading:** *Managing AI Projects*, Ch. 2 - Model Training
   ```

2. **Parses the book URN and chapter**
   
   From the session note or Study Index:
   ```
   urn:orm:book:9798341641006 - Ch. 2
   ```

3. **Pulls full chapter text** via `oreilly-read` MCP:
   
   ```
   Tool: read_chapter
   Input: { urn: "9798341641006", chapter: "2" }
   Output: [full chapter text]
   ```

4. **Reviews chapter against Card Inclusion Bar**
   
   Identifies mental models, failure modes, decision frameworks.

5. **Cross-checks existing cards**
   
   Avoids creating duplicates.

6. **Presents gaps and offers to create cards**

**Result:** You don't miss concepts from assigned readings.

---

## Filtering Book Searches (Optional)

By default, `/study deep` and `/study audit` search the **entire O'Reilly catalog**. You can limit searches to specific books.

### Three ways to filter:

**1. Config file** (applies to all searches):

Create `.claude/study-config.json` in repo root:
```json
{
  "oreilly_books": [
    "Managing AI Projects",
    "Generative AI on Microsoft Azure"
  ]
}
```

Or use URNs:
```json
{
  "oreilly_books": [
    "urn:orm:book:9798341641006",
    "urn:orm:book:9798341623279"
  ]
}
```

**2. Command flag** (one-time override):
```bash
/study deep "RAG" --books "Managing AI Projects,Generative AI on Azure"
```

**3. No filter** (default):
No config + no flag = searches everything.

**Override config temporarily:**
```bash
/study deep "RAG" --all-books   # Ignores config, searches entire catalog
```

---

## How `/study deep` Uses O'Reilly

When you run `/study deep <topic>`, the system:

1. **Checks for book filters:**
   - Reads `.claude/study-config.json` (if exists)
   - Or uses `--books` flag
   - Or searches entire catalog (default)

2. **Searches O'Reilly catalog** via `oreilly` MCP:
   
   ```
   Tool: search_oreilly_content
   Input: { query: "RAG retrieval strategies" }
   Output: [list of matching chapters with metadata]
   ```
   
   Filters results to configured books if specified.

3. **Shows you matching chapters:**
   
   ```
   Found 3 chapters:
   1. Generative AI on Azure, Ch. 8 - Vector Search
   2. Managing AI Projects, Ch. 5 - RAG Patterns
   3. LangChain in Practice, Ch. 4 - Retrieval
   ```

3. **You select which chapter to read**

4. **Pulls full chapter text** via `oreilly-read` MCP:
   
   ```
   Tool: read_chapter
   Input: { urn: "9798341623279", chapter: "8" }
   Output: [full chapter text]
   ```

5. **Displays content** (or saves to `study/deep-dives/`)

6. **Asks if any concepts are worth cardifying**

7. **Creates cards** if you say yes

**Result:** Turn curiosity into drillable concepts in one step.

---

## Fallback Strategy

**What if `oreilly-read` is unavailable?**

The system falls back to manual workflow:

1. Uses `oreilly` to find the chapter (discovery only)
2. Shows you the URL
3. Prompts: "Read this chapter in O'Reilly and paste key concepts here"
4. You paste content
5. System reviews against Card Inclusion Bar
6. Creates cards

**Works, but slower.** Prefer `oreilly-read` when available.

---

## Common Issues

### "Authentication failed"

**Cause:** O'Reilly browser session expired

**Fix:**
1. Log out and back in to https://learning.oreilly.com/ in your browser
2. Restart Claude Code session

### "MCP server not available"

**Cause:** Config incorrect or Claude Code needs a restart

**Fix:**
1. Check `.claude/settings.json` syntax (valid JSON)
2. Restart Claude Code session

### "Chapter not found"

**Cause:** Chapter number or URN incorrect

**Fix:**
1. Verify chapter number in O'Reilly (chapters may be numbered differently)
2. Use `get_table_of_contents` to list all chapters
3. Try chapter title instead of number

### "Rate limited"

**Cause:** Too many chapter pulls in short time

**Fix:**
- Wait 1-2 minutes
- Pull fewer chapters at once
- Use `--save` flag to cache chapters locally

---

## Pro Tips

### Cache Chapters Locally

Use `--save` flag to save pulled chapters:

```bash
/study deep "RAG retrieval" --save
```

Saves to `study/deep-dives/rag-retrieval-2026-06-14.md`. Re-read locally without re-pulling.

### List All Chapters

Before pulling, list chapters to verify numbering:

```
Can you get the table of contents for urn:orm:book:9798341641006?
```

Helps avoid "chapter not found" errors.

### Batch Audits

Audit multiple sessions at once:

```bash
/study audit --all
```

Pulls all assigned readings, creates all gap cards. Good for catching up.

### Limit to Specific Books

When deep-diving, limit search to cohort texts:

```bash
/study deep "fine-tuning" --book urn:orm:book:9798341641006
```

Avoids non-cohort results.

---

## O'Reilly Books Used in AIUP

**Cohort texts (add your program's books here):**

- *Managing AI Projects* - Adrián González Sánchez & Malini Jain Runtasewee
  - URN: `urn:orm:book:9798341641006`
  - URL: https://learning.oreilly.com/library/view/-/9798341641006/

- *Generative AI on Microsoft Azure* - Adrián González Sánchez et al.
  - URN: `urn:orm:book:9798341623279`
  - URL: https://learning.oreilly.com/library/view/-/9798341623279/

**How to find URNs:**
- Open book in O'Reilly
- Copy from URL (the long numeric ID)
- Or use `/study deep` to search by title

---

## References

- [SETUP.md](../SETUP.md) - Complete installation instructions
- [barclayneira/oreilly-mcp](https://github.com/barclayneira/oreilly-mcp) - O'Reilly Read MCP repo
- `/study audit` - Coverage audits with chapter pulling
- `/study deep` - Deep-dives with chapter pulling

---

## Security Note

The MCP server reads your O'Reilly session from the browser — no credentials are stored in this repo or passed through environment variables. Don't hardcode any tokens or cookies in `settings.json`.
