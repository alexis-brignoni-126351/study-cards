# O'Reilly Integration

How the framework integrates with O'Reilly Learning to pull chapter content and create cards.

---

## Why This Matters

The AI Upskilling Program assigns O'Reilly readings alongside sessions. These readings go deeper than session material and often contain failure modes, tradeoffs, and production patterns worth cardifying.

**Without this integration:** You'd manually read, take notes, create cards. Slow, error-prone.

**With this integration:** Pull full chapter text, review against Card Inclusion Bar, create cards in one step.

---

## The MCP Server

The framework uses one O'Reilly MCP server:

**Repository:** [barclayneira/oreilly-mcp](https://github.com/barclayneira/oreilly-mcp)  
**What it does:** Searches the catalog **and** pulls full chapter text  
**Tools:**
- `search_content` - search books, chapters, and articles by query, format, or topic
- `get_book_info` - book metadata, description, and chapter list
- `get_table_of_contents` - detailed chapter hierarchy for a book
- `read_chapter` - returns complete chapter content
- `get_annotations` - your personal O'Reilly highlights and notes

**Used by:**
- `/study audit` - pulls assigned chapters to find concepts
- `/study deep` - searches for chapters, then pulls the one you pick

**Critical:** `read_chapter` is what makes coverage audits work. Without the server, you're back to manual copy-paste.

---

## Setup (Quick Reference)

**Full setup in [SETUP.md](../SETUP.md). Summary:**

1. Get your O'Reilly API token from https://learning.oreilly.com/apidocs/mcp/content/ (log in first)
2. Clone the server, install dependencies, and register it with Claude Code:

```bash
git clone https://github.com/barclayneira/oreilly-mcp.git
cd oreilly-mcp
uv sync

claude mcp add oreilly \
  -s user \
  -e ORM_JWT=YOUR_TOKEN_HERE \
  -- "$(pwd)/.venv/bin/python" "$(pwd)/stdio_server.py"
```

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

3. **Pulls full chapter text** via the `read_chapter` tool:
   
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

2. **Searches O'Reilly catalog** via the `search_content` tool:
   
   ```
   Tool: search_content
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

4. **You select which chapter to read**

5. **Pulls full chapter text** via the `read_chapter` tool:
   
   ```
   Tool: read_chapter
   Input: { urn: "9798341623279", chapter: "8" }
   Output: [full chapter text]
   ```

6. **Displays content** (or saves to `study/deep-dives/`)

7. **Asks if any concepts are worth cardifying**

8. **Creates cards** if you say yes

**Result:** Turn curiosity into drillable concepts in one step.

---

## Fallback Strategy

**What if the MCP server is unavailable?**

The system falls back to a manual workflow:

1. Helps you locate the chapter (title, book, URL if known)
2. Prompts: "Read this chapter in O'Reilly and paste key concepts here"
3. You paste content
4. System reviews against Card Inclusion Bar
5. Creates cards

**Works, but slower.** Prefer the MCP server when available.

---

## Common Issues

### "Authentication failed"

**Cause:** O'Reilly API token expired

**Fix:**
1. Get a fresh token from https://learning.oreilly.com/apidocs/mcp/content/
2. Re-run `claude mcp add oreilly` with the new `ORM_JWT` value
3. Restart Claude Code session

### "MCP server not available"

**Cause:** Server not registered or Claude Code needs a restart

**Fix:**
1. Run `claude mcp list` and confirm `oreilly` is listed and healthy
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
- [barclayneira/oreilly-mcp](https://github.com/barclayneira/oreilly-mcp) - O'Reilly MCP server repo
- `/study audit` - Coverage audits with chapter pulling
- `/study deep` - Deep-dives with chapter pulling

---

## Security Note

Your O'Reilly API token lives in your local Claude Code MCP registration (the `ORM_JWT` env var passed to `claude mcp add`) — it is never stored in this repo. Don't hardcode tokens in any committed file.
