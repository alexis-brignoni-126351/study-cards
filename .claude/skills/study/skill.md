---
name: study
description: Complete learning workflow orchestrator. Handles full lifecycle - coverage audit, card creation, spaced repetition drilling, card evolution. Enforces Card Inclusion Bar, tracks mastery, auto-updates Coverage Maps, integrates with O'Reilly MCP.
allowed-tools: Read, Edit, Write, Bash, Glob, Grep
---

# /study - Learning Workflow Orchestrator

Complete system for active learning with spaced repetition. Twelve subcommands covering session audit, card creation, spaced repetition, and card evolution.

**Paths (auto-detected or configurable):**
```
STUDY_DIR  = study/                  # Top-level study directory
CARDS_DIR  = study/concepts/         # Concept cards
INDEX      = study/Study Index.md    # Master index
SESSIONS   = study/sessions/         # Session notes (optional)
```

**Default detection logic:**
- If current directory has `study/` folder, use it
- If `.claude/settings.json` has `studyDir` config, use that
- Otherwise, prompt user to create `study/` in current directory

**Prerequisites:**
- O'Reilly Read MCP server configured (see SETUP.md for installation)
- Study directory initialized with basic structure

---

## Subcommands

### `/study` or `/study drill` (default)

Active recall quiz on cards **due today** based on spaced repetition schedule.

**Logic:**
1. Read all cards in `CARDS_DIR`, parse frontmatter
2. Calculate `next-due` for each card:
   - If no `last-drilled:` - due now
   - If `mastery: new` - `last-drilled + 2 days`
   - If `mastery: learning` - `last-drilled + 7 days`
   - If `mastery: known` - `last-drilled + 30 days`
3. Filter to cards where `next-due <= today` (or no `next-due` field)
4. Sort by: `new` first, then `learning`, then `known`
5. **Interactive tutor mode:**
   - One card at a time: pose Q, let user attempt, reveal A
   - Close gaps interactively (on-ramp, re-ask differently, Feynman explain-back)
   - Self-grade and update `mastery:` (correct=advance, incorrect=regress, partial=stay)
6. After each card: update frontmatter with Edit:
   - `mastery:` (new/learning/known based on performance)
   - `last-drilled: <today's date>`
   - Calculate and set `next-due: <date>` based on new mastery level
7. End session: short scoreboard, log to `study/logs/drill-log.md`

**Flags:**
- `--all` - drill all cards (ignore due dates)
- `--session <N>` - drill only Session N cards (filter by tag `#session/N`)
- `--topic <name>` - drill only cards in this topic (filter by `deck: <name>`; e.g., `--topic rag`, `--topic ml-fundamentals`)
- `--mastery <level>` - drill only new/learning/known cards
- `--limit <N>` - drill max N cards

**Example:**
```
/study drill
→ Calculates due cards (2 new, 3 learning)
→ Runs interactive tutor on each
→ Updates frontmatter after each card
→ Logs session to drill-log.md

/study drill --topic rag
→ Drills only RAG topic cards

/study drill --limit 5
→ Drills max 5 cards
```

**Tutor behavior:**
- Encouraging, peer-level tone
- Lower the floor when stuck (on-ramp, analogies)
- Productive repetition (re-ask differently after mistakes)
- Feynman explain-backs
- Connect to real-world applications
- Desirable difficulty (let them struggle a bit, then help)

---

### `/study audit <session>`

Coverage audit: find concepts in completed session material that cross the Card Inclusion Bar but have no cards yet.

**Logic:**
1. Read `SESSIONS/Session <N> - <Topic>.md` (or prompt for session note path if not found)
2. Read session's Coverage Map to see what's already mapped
3. Identify assigned O'Reilly readings from session note (look for book chapter references)
4. **Pull O'Reilly chapter content:**
   - Use `oreilly-read` MCP's `read_chapter` tool (primary method)
   - Fallback: use `oreilly` discovery + prompt user to read chapter, paste content
5. Review session content + O'Reilly content against **Card Inclusion Bar**:
   - ✓ Mental models, failure modes, decision frameworks, non-obvious mechanics, production so-whats
   - ✗ History, trivia, vocab-only, program meta
6. Cross-check against existing cards (by title/topic)
7. Present gaps: "Found N concepts that cross the bar but have no cards: [list]"
8. Offer to create cards (confirm selection first)
9. Create cards with:
   - Frontmatter: `type: concept-card`, `deck: session-N`, `mastery: new`, `created: <date>`, `last-revised: <date>`, `version: 1.0`, `status: active`
   - Body: Q/A/On-ramp/Common pitfalls/Connects to/Source
10. **Auto-update Coverage Map** in session note:
    - Add new cards to "Post-Session Additions:" section (or create it)
    - Update audit timestamp: `**Audit:** ✅ Complete (YYYY-MM-DD) - X cards created, Y gaps found`
11. Update Session Coverage Status table in `INDEX`
12. Log to `study/logs/audit-log.md`

**Flags:**
- `--all` - audit all sessions
- `--create-auto` - auto-create all gap cards without prompting
- `--dry-run` - show gaps, don't create cards or update maps

**Example:**
```
/study audit session-1
→ Reads Session 1 note + Coverage Map
→ Pulls assigned O'Reilly chapters via oreilly-read MCP
→ Finds 2 gaps: Overfitting Diagnostics, Train/Val/Test Split
→ Confirms with user
→ Creates 2 cards (V1.0, mastery: new)
→ Updates Session 1 Coverage Map automatically
→ Updates Session Coverage Status table in Study Index
→ Logs to audit-log.md
```

---

### `/study due`

Show which cards are due for drilling based on spaced repetition schedule.

**Logic:**
1. Read all cards, parse frontmatter
2. Calculate `next-due` for each (same logic as drill)
3. Group by:
   - Overdue (next-due < today)
   - Due today (next-due = today)
   - Upcoming (next-due in next 7 days)
4. Show counts by session and mastery level

**Flags:**
- `--upcoming <N>` - show cards due in next N days
- `--overdue` - show only overdue cards

---

### `/study check-prereqs <session>`

Check if prerequisite concepts from earlier sessions are at required mastery level.

**Logic:**
1. Read Prerequisites Map from `INDEX` (Session N requires concepts from Session M)
2. For each prerequisite concept, find the card and check `mastery:` level
3. Report which are met (`known`) vs need refresh (`learning`/`new`)
4. Offer to drill prerequisite cards first

---

### `/study revise <card-name>`

Guided card evolution workflow. Updates card based on trigger (post-drill / later-session / real-world application).

**Logic:**
1. Read the card
2. Ask: "What triggered this revision?"
   - 1. Post-drill (Q/A doesn't land)
   - 2. Later session deepened the concept
   - 3. Real-world application revealed production reality
3. **Based on trigger:**

   **Post-drill (V1.1):**
   - Ask: "What's missing or unclear in the Q/A?"
   - Guide revision: sharpen Q, improve A's production so-what, enhance on-ramp
   - Update frontmatter: `last-revised: <today>`, `version: 1.1`

   **Later session (V1.2+):**
   - Ask: "What did the later session add or change?"
   - Add to `## Connects to:` (link to new concept card)
   - Update `## Common pitfalls` with new nuance
   - Update frontmatter: `last-revised: <today>`, bump minor version

   **Real-world application (V2.0):**
   - Ask: "What did building this teach you that theory didn't?"
   - Rewrite production so-what based on lived experience
   - Update pitfalls with real failure modes encountered
   - Update frontmatter: `last-revised: <today>`, `version: 2.0` (major bump)

4. Log to `study/logs/revision-log.md`

---

### `/study deep <topic>`

Deep-dive on a topic using O'Reilly content. Pull chapter text, surface related material, optionally create cards.

**Logic:**
1. Search O'Reilly catalog via `oreilly` MCP: `search_oreilly_content`
2. Show matching chapters with descriptions
3. User selects which to read
4. Pull full chapter content via `oreilly-read` MCP: `read_chapter`
5. Display content (or save to `study/deep-dives/<topic>-<date>.md`)
6. Ask: "Did this reveal any concepts worth cardifying?"
7. If yes: mini-audit against Card Inclusion Bar, create cards (V1.0, mastery: new)
8. Wire new cards into originating session's Coverage Map as "Post-Session Addition"

**Flags:**
- `--book <urn>` - limit to specific book
- `--save` - auto-save chapter to deep-dives/
- `--no-cards` - just pull content, don't offer cards

---

### `/study ingest <mode>`

Ingest content from any source (transcript, article, PDF, paper) and create cards.

**Three modes:**

**1. `/study ingest transcript`** - Paste meeting/session transcript
**2. `/study ingest text`** - Paste any text (blog post, paper, PDF extract)
**3. `/study ingest file <path>`** - Read from file

**Logic:**
1. Prompt user to paste content (or read from file if path provided)
2. Review content against **Card Inclusion Bar**:
   - Extract mental models, failure modes, decision frameworks
   - Ignore historical context, trivia, vocabulary-only
3. Cross-check against existing cards (avoid duplicates)
4. Present concepts found: "Found N concepts crossing the bar: [list]"
5. Ask which to cardify (or use `--create-auto` to skip confirmation)
6. Create cards with:
   - Source: "Transcript: <topic>" or "Article: <title>" or file path
   - Deck: inferred from content or user specifies
   - Session tag: optional, user can assign
7. Optionally save raw content to `study/sources/<slug>-<date>.md`
8. Log to `study/logs/ingest-log.md`

**Flags:**
- `--save` - save raw content to study/sources/
- `--create-auto` - create all found cards without prompting
- `--session <N>` - tag cards with #session/N
- `--deck <name>` - assign cards to specific deck

**Example 1: Transcript**
```
/study ingest transcript

→ Paste transcript content (or type path to file):

→ [User pastes Teams/Zoom transcript]

Analyzing transcript against Card Inclusion Bar...

Found 4 concepts:
1. Agentic loop pattern (mental model)
2. Context window management (failure mode)
3. Prompt vs RAG decision framework (decision framework)
4. Token budgeting (production so-what)

Create these 4 cards? [y/n]

→ [User: y]

Creating cards...
✓ Agentic Loop Pattern (V1.0, deck: agents, source: Transcript)
✓ Context Window Management (V1.0, deck: context-engineering)
✓ Prompt vs RAG Decision (V1.0, deck: architecture)
✓ Token Budgeting (V1.0, deck: optimization)

Cards created. Run /study sync to update Study Index.
```

**Example 2: Blog Post**
```
/study ingest text --save

→ Paste content:

→ [User pastes blog post about RAG chunking strategies]

Analyzing...

Found 2 concepts:
1. Semantic chunking vs fixed-size (decision framework)
2. Chunk boundary handling (failure mode)

Create these 2 cards? [y/n]

→ [User: y]

Saving raw content to study/sources/chunking-strategies-2026-06-14.md...
Creating cards...

Cards created.
```

**Example 3: Research Paper (from file)**
```
/study ingest file research-papers/attention-is-all-you-need.pdf.txt --session 3

→ Reading file...
→ Analyzing against Card Inclusion Bar...

Found 3 concepts:
1. Self-attention mechanism (non-obvious mechanic)
2. Positional encoding trade-offs (decision framework)
3. Multi-head attention scaling (failure mode)

Tagging with #session/3...
Create these 3 cards? [y/n]
```

**Use cases:**
- Transcripts from Teams, Zoom, Otter (paste text)
- Blog posts, articles (copy/paste or save as file)
- Research papers (copy text from PDF)
- Documentation (pull relevant sections)
- Your own notes (distill into cards)

**No MCP required.** Works with plain text input.

---

### `/study connect <new-concept> <existing-card>`

Link new concept to existing card, trigger revision if it deepens/contradicts.

---

### `/study retire <card-name>`

Deprecate a card that doesn't meet the Card Inclusion Bar.

---

### `/study sync`

Housekeeping: update all Coverage Maps, Session Coverage Status table, recalculate due dates, validate consistency.

**Flags:**
- `--fix` - auto-fix inconsistencies

---

### `/study status`

Learning cycle overview: where am I, what needs attention, suggested next action.

---

### `/study new`

Drill only cards with `mastery: new`. Shorthand for `/study drill --mastery new`.

---

## Card Inclusion Bar

Enforced by `/study audit`, `/study deep`, and when minting new cards.

**Include (concepts that earn a card):**
- **Mental models** - frameworks for decomposing problems
- **Failure modes & diagnostics** - how things break, how to recognize
- **Decision frameworks** - when to use X vs Y, tradeoff analysis
- **Non-obvious mechanics** - how it actually works under the hood
- **Production so-whats** - concepts that impact how you build/deploy

**Exclude (reference-only, no card):**
- Historical context, trivia, program meta
- Vocabulary definitions alone (without decision/tradeoff)
- Examples/case studies (unless generalizable pattern)

**Ambiguous? Ask user, log decision in Study Index "Coverage Decisions" section with rationale.**

---

## Spaced Repetition Schedule

| Mastery Level | Drill Interval | next-due Calculation |
|---------------|----------------|----------------------|
| `new` | 2 days | `last-drilled + 2 days` |
| `learning` | 7 days | `last-drilled + 7 days` |
| `known` | 30 days | `last-drilled + 30 days` |

**Advancement logic (during drill):**
- Correct answer → advance (new→learning, learning→known, known stays known)
- Incorrect answer → regress (learning→new, known→learning, new stays new)
- Partial answer → stay at current level

---

## Card Frontmatter

```yaml
type: concept-card
date: YYYY-MM-DD              # original creation
tags: [#type/concept, #session/N]
source: "..."                  # book + chapter OR session section
deck: topic-name               # which topic deck
mastery: new | learning | known
created: YYYY-MM-DD            # initial creation
last-revised: YYYY-MM-DD       # most recent edit
version: X.Y                   # semantic versioning
last-drilled: YYYY-MM-DD       # last drill session (optional, added by drill)
next-due: YYYY-MM-DD           # calculated by drill/sync (optional)
status: active | retired       # active by default
```

---

## O'Reilly MCP Integration

**Required MCP servers:**

1. **`oreilly-read`** (primary) - full chapter text
   - Repository: `barclayneira/oreilly-mcp`
   - Tool: `read_chapter` (returns actual chapter content)
   - **Critical for `/study audit` and `/study deep` to work**

2. **`oreilly`** (official, fallback) - discovery only
   - Tool: `search_oreilly_content` (returns metadata + URL)
   - Use when oreilly-read unavailable

**Setup:** See SETUP.md for complete O'Reilly Read MCP installation and configuration.

---

## Quality Ratchet (card version progression)

- **V1.0** (post-session): "Here's the concept"
- **V1.1** (post-drill): "Here's the concept + the gotcha I missed"
- **V1.2+** (post-deep-dive / later session): "Here's the concept + the tradeoff"
- **V2.0** (post-application): "Here's what you actually need to know to apply it"
