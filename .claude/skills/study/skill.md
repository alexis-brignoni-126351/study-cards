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
CONFIG     = .claude/study-config.json  # Optional learner profile + book filters
```

**Default detection logic:**
- If current directory has `study/` folder, use it
- If `.claude/settings.json` has `studyDir` config, use that
- Otherwise, prompt user to create `study/` in current directory

**Prerequisites:**
- O'Reilly Read MCP server configured (see SETUP.md for installation)
- Study directory initialized with basic structure

**Learner profile (optional):**
If `.claude/study-config.json` exists and has a `learner` block, all subcommands read it to personalize question framing, feedback tone, and card creation. See `.claude/study-config.json.example` for the format.

---

## Subcommands

### `/study setup`

One-time learner profile setup. Asks structured questions and writes `.claude/study-config.json`. Run before first drill session; re-run anytime context changes.

**Logic:**
1. Check if `.claude/study-config.json` already exists — if so, ask "Update existing profile or start fresh?"
2. Ask the following questions in sequence, presenting options where possible:

   **Q1: What's your current role?**
   - Software / data engineer
   - Product manager or analyst
   - Technical lead or architect
   - Student or career changer
   - Other (type it)

   **Q2: What's your technical background with AI/ML?**
   - New to it — building foundational understanding
   - Familiar with concepts, limited hands-on
   - Have built or shipped AI features
   - Working in AI/ML professionally

   **Q3: What's your primary learning goal?**
   - Perform well in technical interviews
   - Apply AI concepts in my current role
   - Build toward an AI engineering career
   - Lead or evaluate AI projects and teams
   - General literacy and fluency

   **Q4: How do you want concepts framed?**
   - Practical — connect to real systems, tradeoffs, and decisions
   - Conceptual — build deep mental models first, application second
   - Both — alternate depending on the concept

   **Q5: Anything you want the tutor to avoid?**
   - Pure theory without application
   - Over-simplified analogies
   - Excessive jargon without explanation
   - Nothing — no preference

3. Write responses to `.claude/study-config.json` under a `learner` key:
   ```json
   {
     "learner": {
       "role": "...",
       "ai_background": "...",
       "learning_goal": "...",
       "framing_preference": "...",
       "avoid": "..."
     }
   }
   ```
   Preserve any existing `oreilly_books` config if updating.

4. Confirm: "Profile saved. Your drills and card creation will now adapt to your context. Run `/study` to start drilling."

---

### `/study` or `/study drill` (default)

Active recall quiz on cards **due today** based on spaced repetition schedule.

**Logic:**
1. Read `.claude/study-config.json` if it exists — load `learner` profile and note it for the session
2. **Ask session mode:**
   ```
   How do you want to drill today?
   [1] Deep — open-ended questions, full answers, rich feedback
   [2] Rapid — quick-fire, short answers, move fast
   ```
   Default to Deep if no response. Carry the chosen mode through the entire session.
3. Read all cards in `CARDS_DIR`, parse frontmatter
4. Calculate `next-due` for each card:
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
- Desirable difficulty (let them struggle a bit, then help)

**Session mode behavior:**
- **Deep mode:** pose the full card question as-is, expect articulated answers, give rich feedback with follow-ups, push back on gaps, connect to adjacent concepts
- **Rapid mode:** distill the question to its core, accept bullet-point answers, give tight feedback (correct/incorrect + one key insight), no follow-ups unless user asks, move immediately to next card

**Learner profile behavior (if loaded):**
- Before posing each question, add one sentence: "Here's why this matters for you: [connect concept to learner's role/aspirations]"
- Frame feedback through their lens — connect gaps to real stakes in their context
- When writing card Q/A (audit, deep, ingest): ground the question in a realistic scenario relevant to the learner's role, not a generic textbook prompt
- Respect `avoid` field — don't give pure theory responses if learner wants application focus

---

### `/study audit <session>`

Coverage audit: find concepts in completed session material that cross the Card Inclusion Bar but have no cards yet.

**Logic:**
1. Read `SESSIONS/Session <N> - <Topic>.md` (or prompt for session note path if not found)
2. Read session's Coverage Map to see what's already mapped
3. Identify assigned O'Reilly readings from session note (look for book chapter references)
4. **Pull O'Reilly chapter content:**
   - Use the O'Reilly MCP's `read_chapter` tool (primary method)
   - Fallback: locate the chapter with `search_content`, share the URL, prompt user to read chapter and paste content
5. Review session content + O'Reilly content against **Card Inclusion Bar**:
   - ✓ Mental models, failure modes, decision frameworks, non-obvious mechanics, production so-whats
   - ✗ History, trivia, vocab-only, program meta
6. Cross-check against existing cards (by title/topic)
7. Present gaps: "Found N concepts that cross the bar but have no cards: [list]"
8. Offer to create cards (confirm selection first)
9. Create cards with:
   - Frontmatter: `type: concept-card`, `deck: session-N`, `mastery: new`, `created: <date>`, `last-revised: <date>`, `version: 1.0`, `status: active`
   - Body: Q/A/On-ramp/Common pitfalls/Connects to/Source
   - **If learner profile loaded:** frame the Q as a realistic scenario grounded in their role/context, not a generic definition prompt. The A should emphasize the production so-what and tradeoffs relevant to their lens.
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
→ Pulls assigned O'Reilly chapters via the O'Reilly MCP
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
1. **Check for book filters:**
   - Read `.claude/study-config.json` if exists (get `oreilly_books` array)
   - Or use `--books` flag value
   - If neither, search entire catalog
2. Search O'Reilly catalog via the O'Reilly MCP: `search_content`
   - Filter results to configured books if specified
3. Show matching chapters with descriptions
4. User selects which to read
5. Pull full chapter content via the O'Reilly MCP: `read_chapter`
6. Display content (or save to `study/deep-dives/<topic>-<date>.md`)
7. Ask: "Did this reveal any concepts worth cardifying?"
8. If yes: mini-audit against Card Inclusion Bar, create cards (V1.0, mastery: new)
9. Wire new cards into originating session's Coverage Map as "Post-Session Addition"

**Flags:**
- `--books <list>` - limit search to these books (comma-separated titles or URNs)
- `--book <urn>` - limit to specific book URN (legacy, same as --books)
- `--save` - auto-save chapter to deep-dives/
- `--no-cards` - just pull content, don't offer cards
- `--all-books` - ignore config, search entire catalog

**Book filtering (optional):**

You can limit O'Reilly searches to specific books in three ways:

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
```
/study deep "RAG" --books "Managing AI Projects,Generative AI on Azure"
```

**3. No filtering** (default):
Don't create config, don't use flag → searches entire O'Reilly catalog.

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

### `/study connect <card1> <card2>`

Link two cards as related concepts. Updates `## Connects to` in both cards bidirectionally.

**Logic:**
1. Read both cards
2. Check if connection already exists in either card's `## Connects to` section
3. If not, add `[[card2]]` to card1's `## Connects to` and `[[card1]]` to card2's `## Connects to`
4. Ask: "Does this connection deepen or contradict either card?"
5. If yes → offer to trigger `/study revise` on the affected card

**Example:**
```
/study connect "RAG Chunking Strategy" "Hybrid Search and Re-ranking"

→ Reading both cards...
→ No existing connection found.

Adding connection in both cards...
→ RAG Chunking Strategy: added [[Hybrid Search and Re-ranking]]
→ Hybrid Search and Re-ranking: added [[RAG Chunking Strategy]]

Does this connection deepen either card? [Y/n]
→ [User: Y]
→ Which card needs revision? [1] RAG Chunking Strategy [2] Hybrid Search...
```

---

### `/study retire <card-name>`

Deprecate a card that doesn't meet the Card Inclusion Bar. Moves it out of the active drill rotation but keeps it in the vault.

**Logic:**
1. Read the card
2. Ask: "Why retire this card?"
   - 1. Doesn't meet Card Inclusion Bar (reference-only)
   - 2. Superseded by a better card
   - 3. No longer relevant to current learning goals
3. Set `status: retired` in frontmatter
4. Move file to `study/retired/`
5. Remove from any Coverage Map that references it (mark as "Retired")
6. Update Session Coverage Status table in `INDEX` if card count changes
7. Log to `study/logs/revision-log.md`

**Example:**
```
/study retire "Narrow vs General vs Super AI"

→ Reading card...
→ Why retire?
→ 1. Doesn't meet Card Inclusion Bar (reference-only)
→ 2. Superseded by better card
→ 3. No longer relevant

[User: 1]

→ Setting status: retired...
→ Moving to study/retired/...
→ Removing from Session 1 Coverage Map...

Card retired. No longer in active drill rotation.
```

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

**Required MCP server:**

**`oreilly`** - Repository: `barclayneira/oreilly-mcp`
- `search_content` - catalog search (books, chapters, articles)
- `get_book_info` / `get_table_of_contents` - book metadata and chapter lists
- `read_chapter` - full chapter text
- **Critical for `/study audit` and `/study deep` to work**

If the server is unavailable, fall back to the manual workflow: share the chapter URL and ask the user to paste key content.

**Setup:** See SETUP.md for installation and authentication (O'Reilly API token).

---

## Quality Ratchet (card version progression)

- **V1.0** (post-session): "Here's the concept"
- **V1.1** (post-drill): "Here's the concept + the gotcha I missed"
- **V1.2+** (post-deep-dive / later session): "Here's the concept + the tradeoff"
- **V2.0** (post-application): "Here's what you actually need to know to apply it"
