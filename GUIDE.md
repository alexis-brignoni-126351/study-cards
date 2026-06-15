## Usage Guide

How to use Study Cards for active learning with spaced repetition.

---

## The Complete Workflow

```
Session → Coverage Audit → Create Cards → Drill → Deep-Dive → Revise → Repeat
```

**Weekly cycle:**
1. Attend session (or complete learning material)
2. Run `/study audit session-N` to create cards from concepts that cross the bar
3. Drill new cards with `/study` (2 days later, then weekly, then monthly)
4. Deep-dive on topics with `/study deep <topic>` when needed
5. Revise cards based on drilling or new insights with `/study revise`

---

## Daily: Running Drills

**Command:** `/study` or `/study drill`

This is your daily active-recall practice. The system calculates which cards are due based on spaced repetition and quizzes you interactively.

### How it works

1. System finds cards due today (based on `mastery` level and `last-drilled` date)
2. Presents one card at a time: shows the Question
3. You attempt to answer (out loud or in your head)
4. System reveals the Answer
5. You self-grade: correct / incorrect / partial
6. System updates card mastery and schedules next drill

### Example session

```
/study drill

→ You have 5 cards due today (2 new, 3 learning)

Card 1/5: Bias-Variance Tradeoff

Q: Your model performs great on training data (98% accuracy) but terrible 
   on test data (65% accuracy). Walk through the bias-variance tradeoff...

[You attempt to answer]

Press enter to see the answer.

A: Your model is overfitting. High variance (too sensitive to training 
   data noise), low bias (fits training data well)...

Did you get it right?
1. Correct - advance to next mastery level
2. Partial - stay at current level
3. Incorrect - move back one level

→ [You select 1]

Card updated: mastery new → learning, next drill in 7 days

Card 2/5: RAG Chunking Strategy
...
```

### Tutor mode features

The system is an **interactive tutor**, not just flashcards:

- **Stuck?** Ask for the on-ramp (simpler explanation)
- **Need an example?** Ask for a concrete case
- **Want to go deeper?** Ask follow-up questions
- **Made a mistake?** It re-asks differently (productive repetition)

### Drill filters

Focus your practice:

```bash
/study drill --topic rag              # Only RAG cards
/study drill --session 1               # Only Session 1 cards
/study drill --mastery new             # Only new cards
/study drill --limit 3                 # Max 3 cards
/study drill --all                     # All cards (ignore due dates)
```

**Pro tip:** Use `/study new` as shorthand for drilling only new cards.

---

## After a Session: Coverage Audit

**Command:** `/study audit session-N`

This finds concepts in your session material that cross the Card Inclusion Bar but don't have cards yet.

### How it works

1. Reads your session note (from `study/sessions/`)
2. Reads the session's Coverage Map to see what's already covered
3. Pulls assigned O'Reilly chapter content (via `oreilly-read` MCP)
4. Reviews everything against the **Card Inclusion Bar**:
   - **Include:** Mental models, failure modes, decision frameworks, non-obvious mechanics, production so-whats
   - **Exclude:** History, trivia, vocab-only, program meta
5. Cross-checks against existing cards
6. Shows you the gaps and offers to create cards
7. Creates cards (after you confirm)
8. Auto-updates the session's Coverage Map
9. Logs the audit

### Example

```
/study audit session-2

→ Reading Session 2 note...
→ Found Coverage Map: 2 cards already created
→ Pulling O'Reilly chapters: Managing AI Projects Ch. 3...
→ Analyzing against Card Inclusion Bar...
→ 
→ Found 3 concepts that cross the bar:
→ 1. Context Engineering Scopes (mental model)
→ 2. RAG Question Vectorization (non-obvious mechanic)
→ 3. MCP Ping-Pong Pattern (failure mode)
→ 
→ Create these 3 cards? [y/n]

→ [You type: y]

Creating cards...
✓ Context Engineering Scopes (V1.0, mastery: new)
✓ RAG Question Vectorization (V1.0, mastery: new)
✓ MCP Ping-Pong Pattern (V1.0, mastery: new)

Updating Session 2 Coverage Map...
✓ Added to "Post-Session Additions"
✓ Audit timestamp updated

Session 2 audit complete: 3 cards created
```

### Best practice

Run the audit **1-2 days after the session** while the material is fresh. This gives you time to review session notes but not so long that you forget context.

---

## Ingest Any Content Source

**Command:** `/study ingest <mode>`

Use this to create cards from transcripts, articles, PDFs, or any text source. **No MCP needed** - just paste text or provide file path.

### Three modes

**1. Transcript** - paste meeting/session transcript
**2. Text** - paste any text (blog, article, paper)
**3. File** - read from file path

### How it works

1. You paste content (or provide file path)
2. System reviews against Card Inclusion Bar
3. Extracts concepts worth cardifying
4. Creates cards with minimal human work

### Example: Session Transcript

```
/study ingest transcript

→ Paste transcript content:

→ [You paste Zoom auto-transcript from session]

Analyzing transcript against Card Inclusion Bar...

Found 4 concepts:
1. Agentic loop pattern (mental model)
2. Context window management (failure mode)
3. Prompt vs RAG decision framework (decision framework)
4. Token budgeting (production so-what)

Create these 4 cards? [y/n]

→ [You type: y]

Creating cards...
✓ Agentic Loop Pattern (V1.0, deck: agents)
✓ Context Window Management (V1.0, deck: context-engineering)
✓ Prompt vs RAG Decision (V1.0, deck: architecture)
✓ Token Budgeting (V1.0, deck: optimization)

Cards created.
```

### Example: Blog Post

```
/study ingest text --save

→ Paste content:

→ [You copy/paste blog post about RAG retrieval strategies]

Analyzing...

Found 2 concepts:
1. Semantic vs keyword search (decision framework)
2. Re-ranking impact on precision (production so-what)

Saving raw content to study/sources/...
Creating cards...

Cards created.
```

### Example: Research Paper

```
# First, extract text from PDF
pdftotext paper.pdf study/sources/paper.txt

# Then ingest
/study ingest file study/sources/paper.txt --session 3

→ Reading file...
→ Analyzing...

Found 3 concepts:
1. Self-attention mechanism (non-obvious mechanic)
2. Positional encoding trade-offs (decision framework)

Tagging with #session/3...
Create these 2 cards? [y/n]
```

### When to use `/study ingest`

- **Transcripts** - Teams, Zoom, Otter auto-transcripts from sessions
- **Articles** - blog posts, documentation, wiki pages
- **Papers** - research papers (extract key concepts, not everything)
- **Your notes** - distill your own notes into cards
- **Any text source** - paste it, extract concepts

**Pro tip:** Don't try to cardify everything. Let the Card Inclusion Bar filter. Only mental models, failure modes, decision frameworks, and production so-whats.

---

## Deep-Dive: Pull O'Reilly Content

**Command:** `/study deep <topic>`

Use this when you want to go deeper on a topic beyond what the session covered.

### How it works

1. Searches O'Reilly catalog for the topic
2. Shows matching chapters
3. You select which chapter to read
4. Pulls full chapter text (via `oreilly-read` MCP)
5. Displays content (or saves to `study/deep-dives/`)
6. Asks if any concepts are worth cardifying
7. Creates cards if you say yes

### Example

```
/study deep "RAG retrieval strategies"

→ Searching O'Reilly...
→ 
→ Found 3 chapters:
→ 1. Generative AI on Azure, Ch. 8 - Vector Search
→ 2. Managing AI Projects, Ch. 5 - RAG Patterns
→ 3. LangChain in Practice, Ch. 4 - Retrieval
→ 
→ Which chapter? [1/2/3]

→ [You type: 1]

Pulling Ch. 8...
[Full chapter content displayed]

Did this reveal any concepts worth cardifying? [y/n]

→ [You type: y]

Which concepts?
- Hybrid search re-ranking (adds precision to semantic search)
- Metadata filtering pre-retrieval (narrows search space)

Creating 2 cards...
✓ Hybrid Search Re-ranking (V1.0, deck: rag)
✓ Metadata Filtering (V1.0, deck: rag)

Added to Session 2 Coverage Map as "Post-Session Addition"
```

### Flags

```bash
/study deep <topic> --save         # Save chapter to study/deep-dives/
/study deep <topic> --no-cards     # Just read, don't create cards
/study deep <topic> --book <urn>   # Limit to specific book
```

---

## Check What's Due

**Command:** `/study due`

See which cards are coming up for review.

### Example

```
/study due

Cards due today: 4
- Session 1: 2 new, 1 learning
- Session 2: 1 learning

Overdue: 2
- Bias-Variance Tradeoff (learning, due 2026-06-12)
- Feature Engineering (known, due 2026-06-10)

Upcoming (next 7 days): 6
- 2026-06-16: 3 cards
- 2026-06-18: 2 cards
- 2026-06-20: 1 card
```

**Filters:**

```bash
/study due --overdue              # Only overdue cards
/study due --upcoming 14          # Next 14 days
```

---

## Check Your Progress

**Command:** `/study status`

Get an overview of where you are in the learning cycle.

### Example

```
/study status

Current: Session 2 completed (2026-06-12)
Next: Session 3 upcoming

Cards: 11 total
- 4 new (not yet drilled)
- 5 learning (1-2 weeks old)
- 2 known (mastered)

Due cards:
- 3 overdue
- 4 due today
- 2 upcoming (next 7 days)

Sessions needing audit:
- Session 2 (audit incomplete, 2 gaps found)

Suggested next action: /study drill (4 cards due today)
```

This tells you what needs attention and what to do next.

---

## Revise a Card

**Command:** `/study revise <card-name>`

Use this when:
- A card's Q/A doesn't land after drilling (version 1.1)
- A later session deepens the concept (version 1.2+)
- You applied it in practice and learned what really matters (version 2.0)

### Example: Post-drill revision

```
/study revise "Bias-Variance Tradeoff"

→ Reading card...
→ What triggered this revision?
→ 1. Post-drill (Q/A doesn't land)
→ 2. Later session deepened the concept
→ 3. Real-world application
→ 
→ [You select: 1]
→ 
→ Current Q: "Your model performs great on training data..."
→ What's missing or unclear?

→ [You type: "The Q doesn't emphasize train-test gap as the diagnostic"]

Revising to emphasize train-test gap as key diagnostic...
Sharpening production so-what...

Card updated:
- Version: 1.0 → 1.1
- Last revised: 2026-06-14

Logged to study/logs/revision-log.md
```

### The version ladder

- **V1.0** - Post-session: "Here's the concept"
- **V1.1** - Post-drill: "Here's the concept + the gotcha I missed"
- **V1.2+** - Later session: "Here's the concept + the tradeoff"
- **V2.0** - Real-world: "Here's what you actually need to know to apply it"

---

## Retire a Bad Card

**Command:** `/study retire <card-name>`

Use this when a card doesn't actually meet the Card Inclusion Bar.

### Example

```
/study retire "Narrow vs General vs Super AI"

→ Reading card...
→ Why retire this card?
→ 1. Doesn't meet Card Inclusion Bar (reference-only)
→ 2. Superseded by better card
→ 3. No longer relevant
→ 
→ [You select: 1]

Marking as retired...
Updating Session 1 Coverage Map (moved to Retired section)...
Moving to study/retired/...

Card retired. No longer in active drill rotation.
```

Retired cards don't show up in drills but stay in the vault for reference.

---

## Housekeeping: Sync Everything

**Command:** `/study sync`

Validates consistency across all cards, Coverage Maps, and the Study Index. Run this weekly or when things feel out of sync.

### What it checks

- All Coverage Map references point to existing cards
- All cards are listed in the right session's Coverage Map
- Due dates are calculated correctly
- Prerequisites Map is valid

### Example

```
/study sync

→ Scanning sessions...
→ Session 1: 7 cards mapped ✓
→ Session 2: 4 cards created, Coverage Map shows 3 → inconsistency
→ 
→ Found issues:
→ - "Context Engineering Scopes" exists but not in Session 2 Coverage Map
→ 
→ Auto-fix? [y/n]

→ [You type: y]

Fixing...
✓ Added to Coverage Map
✓ Recalculated all due dates
✓ Updated Session Coverage Status table

Sync complete.
```

Use `--fix` flag to auto-fix without prompting: `/study sync --fix`

---

## Card Inclusion Bar Reference

**What gets a card (include):**
- Mental models (frameworks for decomposing problems)
- Failure modes (how things break, how to diagnose)
- Decision frameworks (when to use X vs Y)
- Non-obvious mechanics (how it actually works under the hood)
- Production so-whats (impacts how you build/deploy)

**What stays reference-only (exclude):**
- Historical context, timelines, who invented what
- Program logistics, session schedules
- Vocabulary definitions without tradeoffs
- Trivia (dates, company names)
- Examples/case studies unless they generalize

**Ambiguous?** Log the decision in Study Index > Coverage Decisions with rationale.

---

## Tips for Success

### Daily habit

Run `/study` every morning. 5-10 minutes. Spaced repetition only works if you actually do it.

### Quality over coverage

Better to have 20 high-quality cards you drill consistently than 100 mediocre cards you avoid.

### First version doesn't have to be perfect

Cards improve over time (V1.0 → V2.0). Create cards early, refine as you learn.

### Use the on-ramp

If a card's answer doesn't land, the on-ramp section has a simpler explanation. Use it.

### Connect concepts

When you create a new card, link it to related cards in the "Connects to" section. These connections build your mental model.

### Audit after every session

Don't let gaps pile up. Run `/study audit session-N` within 1-2 days of the session.

### Revise after application

When you use a concept in practice, revisit the card and upgrade it to V2.0 based on what you learned.

---

## Workflow Examples

### Weekly cycle

**Monday (after session):**
```
/study audit session-1
→ Creates 5 new cards
```

**Tuesday-Friday (daily):**
```
/study
→ Drills due cards (5-10 minutes)
```

**Weekend (deep-dive):**
```
/study deep "embedding strategies"
→ Pulls O'Reilly chapter
→ Creates 2 additional cards
```

### Pre-session check

Before starting a new session, check prerequisites:

```
/study check-prereqs session-3

→ Session 3 requires:
   ✓ Context Engineering (known)
   ✗ RAG Question Vectorization (new)
   
→ Drill prerequisite card first? [y/n]
```

Ensures you're ready for new material.

---

## Troubleshooting

### Cards not showing up in `/study`

**Check:**
- Card has `mastery: new` in frontmatter
- Card has `status: active` (not `retired`)
- Card file is in `study/concepts/` directory

### `/study audit` can't find O'Reilly chapters

**Fix:**
- Verify O'Reilly Read MCP is configured (see SETUP.md)
- Check `OREILLY_COOKIE` environment variable is set
- Try searching manually: "Can you search O'Reilly for X?"

### Coverage Map out of sync

**Fix:**
```
/study sync --fix
```

Auto-repairs inconsistencies.

---

## Next Steps

- Create your first card from a real session
- Set up a daily drill habit
- Run `/study status` to see where you are
- Explore `/study deep` on topics you want to understand better

For setup help, see [SETUP.md](SETUP.md).
