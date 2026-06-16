# Study Cards

Spaced repetition + active recall system for technical learning. Built for the AI Upskilling Program cohort, designed to work anywhere.

**Core engine + optional modules.** Use as-is or extend with your own sources, formats, and organization.

---

## What This Is

A flexible learning workflow that turns passive reading into active mastery:

- **Spaced repetition** - automatic scheduling (drill new concepts in 2 days, learning in 7 days, known in 30 days)
- **Interactive tutor** - not flashcards, actual teaching (re-ask differently, explain simpler, connect to practice)
- **Session modes** - Deep (open-ended, rich feedback) or Rapid (quick-fire, tight feedback) — chosen at the start of every drill
- **Learner profile** - run `/study setup` once to tell the system your background and goals; questions and feedback adapt to your context from that point on
- **Card versioning** - cards improve as you learn (V1.0 post-session → V2.0 post-application)
- **Coverage audits** - automatically find concepts in session material worth cardifying
- **O'Reilly integration** - pull full chapter text, create cards from deep-dives
- **Quality filter** - Card Inclusion Bar enforces "mental models and failure modes," not trivia

**Works standalone** - no Obsidian required. Plain markdown files + Claude Code.  
**Obsidian optional** - adds clickable wikilinks and graph view.

---

## Why This Exists

The NBS AI Upskilling Program (June 2026) runs 9 sessions across 9 weeks. Each Friday session covers dense material, assigned O'Reilly readings go deeper, and by Session 9 you're presenting a capstone to leadership.

**The problem:** That pace leaves no room for passive reading. You finish a chapter, feel like you understood it, then can't recall it two weeks later when the capstone needs it.

**This framework:** Active recall with spaced repetition. You build cards for concepts that actually matter — mental models, failure modes, decision frameworks — drill them on a schedule, and revise them as your understanding deepens. The goal isn't to memorize everything. It's to have the right concepts ready when the capstone and your career actually need them.

Everyone in the cohort comes from a different job function and has different ambitions. The learner profile (`/study setup`) accounts for that — questions and feedback adapt to your specific context, not a generic "AI student" profile.

---

## Quick Start

**3 steps to first drill session:**

1. **Clone and configure**
   ```bash
   git clone https://github.com/alexis-brignoni-126351/study-cards.git
   cd study-cards
   ```
   Configure O'Reilly MCP in `.claude/settings.json` (already included) and log in to O'Reilly in your browser. See [SETUP.md](SETUP.md).

2. **Set up your learner profile**
   ```bash
   claude
   ```
   Then inside Claude Code:
   ```
   /study setup
   ```
   Takes 2 minutes. Tells the system your background and goals so questions and feedback adapt to your context.

3. **Run your first drill**
   ```
   /study drill --limit 1
   ```

**Full setup guide:** [SETUP.md](SETUP.md)  
**Usage guide:** [GUIDE.md](GUIDE.md)

---

## Features

### Book Filtering (Optional)

Limit O'Reilly searches to specific books instead of the entire catalog.

**Three ways:**
1. **Config file** - create `.claude/study-config.json` with your book list (persistent)
2. **Command flag** - `--books "Book1,Book2"` (one-time)
3. **No filter** - search everything (default)

See [README-BOOK-FILTERING.md](README-BOOK-FILTERING.md) for complete guide.

### Automatic Spaced Repetition

Cards track `mastery` level (new / learning / known) and calculate next drill date automatically:

- **New** - drill in 2 days
- **Learning** - drill in 7 days
- **Known** - drill in 30 days

Performance on each drill updates mastery:
- Correct → advance (new → learning → known)
- Incorrect → regress (learning → new, known → learning)
- Partial → stay at current level

**You never manually track "when to review."** Run `/study` and the system shows you what's due.

### Interactive Tutor (Not Flashcards)

Traditional flashcards: Q → flip → A.

This system: **conversational tutoring.**

- Stuck? Ask for the on-ramp (simpler explanation)
- Made a mistake? It re-asks differently (productive repetition)
- Want an example? Ask for one
- Need to go deeper? Follow-up questions work

**Example:**

```
Q: Your model performs great on training data but terrible on test data.
   Walk through the bias-variance tradeoff...

[You attempt]

A: Your model is overfitting. High variance, low bias...

Didn't land? Here's the on-ramp:
Think of it like studying for an exam. Overfitting = memorizing 
practice test answers word-for-word. You ace the practice test but 
fail the real exam...
```

### Card Versioning (Quality Ratchet)

Cards improve as your understanding deepens:

- **V1.0** (post-session) - "Here's the concept"
- **V1.1** (post-drill) - "Here's the concept + the gotcha I missed"
- **V1.2+** (later session) - "Here's the concept + the tradeoff"
- **V2.0** (post-application) - "Here's what you actually need to know to apply it"

Use `/study revise <card-name>` to evolve a card based on new insights.

### Coverage Audits

After completing a session or chapter, run:

```
/study audit session-1
```

The system:
1. Reads your session note
2. Pulls assigned O'Reilly chapter content (full text via `oreilly-read` MCP)
3. Reviews against the **Card Inclusion Bar** (mental models, failure modes, decision frameworks)
4. Cross-checks existing cards
5. Shows gaps and offers to create cards
6. Auto-updates Coverage Map

**You never miss a concept worth learning.**

### O'Reilly Deep-Dives

Go deeper on a topic:

```
/study deep "RAG retrieval strategies"
```

The system:
1. Searches O'Reilly catalog
2. You pick a chapter
3. Pulls full chapter text
4. Asks which concepts are worth cardifying
5. Creates cards, wires them into Coverage Map

**Turns reading into drilling in one step.**

### Card Inclusion Bar (Quality Filter)

Not everything deserves a card. Only concepts that build engineering judgment:

**Include:**
- Mental models (frameworks for decomposing problems)
- Failure modes (how things break, diagnostics)
- Decision frameworks (when to use X vs Y)
- Non-obvious mechanics (how it actually works)
- Production so-whats (impacts how you build/deploy)

**Exclude:**
- Historical context, timelines, trivia
- Vocabulary without tradeoffs
- Program logistics
- Examples/case studies (unless they generalize)

**Result:** A small, high-quality card library you actually use, not a graveyard of forgotten flashcards.

---

## Tech Stack

- **Claude Code** - skill framework + interactive tutor
- **O'Reilly Read MCP** - full chapter text from O'Reilly books ([barclayneira/oreilly-mcp](https://github.com/barclayneira/oreilly-mcp))
- **Markdown files** - cards are plain text, no database
- **Optional: Obsidian** - for wikilink navigation and graph view

---

## Example Card

```markdown
---
type: concept-card
deck: rag
mastery: learning
version: 1.1
---

# RAG Chunking Strategy

**Q:** You're building a RAG system for technical docs. Walk through how 
chunk size impacts retrieval quality: what happens when chunks are too 
small vs too large, and what's your decision framework?

**A:**
- Chunk size is a precision-vs-context tradeoff
- Too small (50-100 tokens): high precision, loses context
- Too large (1000+ tokens): more context, lower precision
- Just right (200-500 tokens): balances both
- Decision framework: query type, document structure, retrieval top-K
- Use 10-20% overlap so concepts spanning boundaries don't split
- So-what: Start with 300-token chunks, 50-token overlap, tune based on 
  retrieval quality

## On-ramp
Think of chunking like cutting up a textbook for search...

## Common pitfalls
- Using same chunk size for all document types
- No overlap (concepts split across boundaries)
- Not measuring retrieval quality after tuning

## Connects to
[[Hybrid Search and Re-ranking]] · [[Embeddings Types]]
```

---

## Commands Reference

| Command | What it does |
|---------|--------------|
| `/study setup` | One-time learner profile setup — personalizes questions and feedback |
| `/study` | Drill cards due today — prompts for Deep or Rapid mode first |
| `/study audit session-N` | Find gaps in session material, create cards |
| `/study ingest <mode>` | Ingest transcript, article, PDF (paste text or file path) |
| `/study deep <topic>` | Pull O'Reilly chapter, create cards from it |
| `/study due` | Show upcoming cards (overdue, today, next 7 days) |
| `/study status` | Learning progress overview + suggested action |
| `/study revise <card>` | Evolve a card (V1.0 → V2.0) |
| `/study sync` | Validate consistency, fix Coverage Maps |
| `/study new <name>` | Create a new card interactively |
| `/study retire <card>` | Archive a card that's no longer useful |
| `/study check-prereqs session-N` | Verify prerequisite concepts are at required mastery |
| `/study connect <card1> <card2>` | Link two cards as related concepts |

**Full command reference:** [GUIDE.md](GUIDE.md)  
**Extensibility guide:** [docs/extending.md](docs/extending.md)

---

## Installation

**Prerequisites:**
- Claude Code CLI
- Node.js 18+
- O'Reilly account (Nelnet corporate access)

**Steps:**

1. Clone repo:
   ```bash
   git clone https://github.com/alexis-brignoni-126351/study-cards.git
   cd study-cards
   ```

2. Configure O'Reilly MCP in `.claude/settings.json` (already included in the repo):
   ```json
   {
     "mcpServers": {
       "oreilly-read": {
         "command": "npx",
         "args": ["-y", "@barclayneira/oreilly-mcp"]
       }
     }
   }
   ```

3. Log in to O'Reilly in your browser — the MCP picks up your session automatically, no extra steps needed.

4. Test:
   ```bash
   claude
   /study status
   ```

**Full setup instructions:** [SETUP.md](SETUP.md)

---

## Usage Example

**Weekly workflow:**

**Monday (after session):**
```
/study audit session-2
→ Finds 4 concepts crossing the bar
→ Creates 4 cards (V1.0, mastery: new)
```

**Tuesday-Friday (daily drill, 5-10 min):**
```
/study
→ 2 cards due today
→ Interactive tutor, self-grade
→ Cards advance to "learning"
```

**Weekend (deep-dive):**
```
/study deep "RAG hybrid search"
→ Pulls chapter from O'Reilly
→ Creates 2 more cards
→ Wires into Session 2 Coverage Map
```

**Next week:**
```
/study
→ Original cards due again (7 days later)
→ One you got wrong regressed to "new"
→ Drill until it sticks
```

---

## Project Structure

```
study-cards/
├── .claude/skills/study/       # The /study skill
├── study/
│   ├── Study Index.md          # Master index
│   ├── concepts/               # Concept cards
│   ├── sessions/               # Session notes (optional)
│   └── logs/                   # Auto-generated logs
├── templates/                  # Card templates
├── docs/                       # Reference docs
├── SETUP.md                    # Installation
├── GUIDE.md                    # Usage
└── README.md                   # This file
```

---

## Documentation

- [SETUP.md](SETUP.md) - Installation and configuration
- [GUIDE.md](GUIDE.md) - Complete usage guide with examples
- [docs/card-inclusion-bar.md](docs/card-inclusion-bar.md) - Quality filter principles
- [docs/spaced-repetition.md](docs/spaced-repetition.md) - How the SR system works
- [docs/oreilly-integration.md](docs/oreilly-integration.md) - O'Reilly MCP setup details

---

## Using This Framework

Built for the AIUP cohort and shared as-is.

- **Fork it** - make it yours
- **Modify it** - adapt to your workflow  
- **Share improvements** - PRs welcome but not monitored actively

---

## License

MIT License - use it, modify it, share it.

---

## Acknowledgments

Built on:
- Claude Code skill framework (Anthropic)
- O'Reilly Read MCP ([@barclayneira](https://github.com/barclayneira))
- AIUP cohort feedback and testing

Created for the AI Upskilling Program cohort to turn passive learning into active mastery.
