# Extending the Framework

How to adapt the Study Cards to your workflow, sources, and preferences.

---

## Core Philosophy

**The framework is modular by design:**
- **Core engine** - spaced repetition, mastery tracking, drill scheduling (works as-is)
- **Optional modules** - O'Reilly integration, transcript ingestion, custom sources
- **Your organization** - topics, sessions, weeks, projects (you decide)
- **Your format** - concept cards, flashcards, mind maps (customize templates)

**Don't fight the system, extend it.** The `/study` skill handles the mechanics (scheduling, tracking, drilling). You control the content and organization.

---

## Adding New Content Sources

The framework comes with O'Reilly integration. Here's how to add others:

### Transcripts (Built-in)

Use `/study ingest transcript` to process meeting/session transcripts:

```bash
/study ingest transcript
# Paste Teams/Zoom/Otter transcript
# System extracts concepts, creates cards
```

**Works with:**
- Teams auto-transcripts
- Zoom transcripts
- Otter.ai transcripts
- Manual notes from recorded sessions

**No MCP needed** - just paste text.

### Blog Posts & Articles

Use `/study ingest text` for web content:

```bash
/study ingest text --save
# Paste article content
# System finds concepts, creates cards
# Saves raw content to study/sources/
```

**Workflow:**
1. Read article in browser
2. Copy relevant sections (or whole article)
3. Paste into `/study ingest text`
4. System extracts card-worthy concepts

### Research Papers

Two approaches:

**Option 1: Copy/paste text**
```bash
/study ingest text --session N
# Copy text from PDF (paper abstract + key sections)
# Tag with session number
```

**Option 2: Save as file, then ingest**
```bash
# Extract text: pdftotext paper.pdf paper.txt
/study ingest file paper.txt --deck research
```

**Pro tip:** Don't try to cardify the whole paper. Extract 3-5 key concepts that cross the Card Inclusion Bar.

### Documentation & Wikis

Same as articles - copy relevant sections:

```bash
/study ingest text
# Paste from internal wiki, docs site, README
# Focus on failure modes, decision frameworks, non-obvious mechanics
```

### Your Brain (Cross-Reference)

If you have the Claude Brain setup:

**Option 1: Export from Brain, ingest here**
```bash
# In Brain: read wiki/domain/concept.md
# Copy key sections
# In study framework: /study ingest text
```

**Option 2: Link cards to Brain pages**
Add Brain references in card's `## Source` section:
```markdown
## Source
Brain: [[wiki/systems/FACTS SIS#authentication-flow]]
O'Reilly: *Managing AI Projects*, Ch. 3
```

**Option 3: Brain skill integration** (advanced)
Create a Brain skill that calls `/study` in this directory to drill cards from Brain context.

### Custom MCP Sources

If you have other MCP servers (Notion, GitHub, etc.):

**Pattern:**
1. Use MCP to fetch content (via Claude Code)
2. Save content to file in `study/sources/`
3. Run `/study ingest file <path>`

**Example: GitHub Issues**
```bash
# In Claude Code session:
"Pull issue #42 from myrepo and save to study/sources/issue-42.md"
/study ingest file study/sources/issue-42.md
```

---

## Customizing Card Format

The default format is **concept cards** (reasoning-level Q, production so-what A). You can adapt:

### Flashcards (Simple Q/A)

Edit template to simplify:

```markdown
# Concept Name

**Q:** [Simple factual question]

**A:** [Direct answer]
```

Remove on-ramp, pitfalls, connects to. Still tracks mastery and schedules reviews.

### Cloze Deletion Cards

Add cloze format to template:

```markdown
# Concept Name

**Text:** Overfitting occurs when {{c1::training accuracy >> test accuracy}}, 
indicating {{c2::high variance}}.

**Reveals:**
- c1: training accuracy >> test accuracy
- c2: high variance
```

The drill flow stays the same (Q → attempt → reveal → grade).

### Visual Cards (Diagrams)

Add diagram section:

```markdown
# Architecture Pattern

**Q:** Walk through this system's data flow...

**A:** [Explanation]

**Diagram:**
```
[ASCII art or reference to image file]
```

Store images in `study/concepts/images/`, reference in card.

### Mind Map Cards

Focus on connections:

```markdown
# Central Concept

**Connects to:**
- [[Upstream Concept]] - how it flows in
- [[Downstream Concept]] - what it enables
- [[Contrasts with]] - different approach
- [[Composed of]] - building blocks
```

Use `## Connects to` section heavily.

### Code Snippet Cards

For programming concepts:

```markdown
# Pattern Name

**Q:** When do you use this pattern and what's the tradeoff?

**A:** [Explanation]

**Example:**
```python
# Code example here
```

**Anti-pattern:**
```python
# What NOT to do
```
```

---

## Changing Organization

Default is organize by **topic** (deck) and **session** (tag). Alternatives:

### By Week

Tag cards with week instead of session:

```yaml
tags: [#type/concept, #week/1]
deck: topic-name
```

Drill by week: `/study drill --week 1` (if you modify the skill's filter logic).

### By Chapter (Book-Centric)

Organize around book chapters:

```yaml
tags: [#type/concept, #book/managing-ai-projects]
deck: chapter-2
source: "Managing AI Projects, Ch. 2"
```

### By Project

If studying for specific projects:

```yaml
tags: [#type/concept, #project/capstone]
deck: rag
```

Drill project-relevant cards: `/study drill --project capstone`.

### By Difficulty

Add difficulty rating:

```yaml
difficulty: easy | medium | hard
```

Drill harder cards more often (custom scheduling logic).

### Flat (No Organization)

Skip deck/session entirely:

```yaml
tags: [#type/concept]
```

Just drill whatever's due. Simplest approach.

---

## Integration Patterns

### With Your Brain

**Pattern 1: Separate but linked**
- Brain = work knowledge (projects, meetings, people)
- Study framework = learning knowledge (concepts, theories)
- Link between them in card sources

**Pattern 2: Study cards in Brain**
- Move `study/` folder into Brain vault
- Brain index references study cards
- Drill from Brain context

**Pattern 3: Brain skill calls study**
Create Brain skill that:
1. Reads concept from Brain page
2. Creates study card in framework
3. Links back to Brain page

### With Obsidian

If you use Obsidian for other notes:

**Option 1: Open framework as vault**
```bash
# In Obsidian: Open folder as vault
# Select: /path/to/study-cards
```

Wikilinks become clickable, graph view shows connections.

**Option 2: Dual vaults**
- Main vault = general notes
- Study vault = this framework
- Link between them with file paths

### With Notion/Roam/Logseq

Export content from other tools:

```bash
# Export Notion page to markdown
# Save to study/sources/
/study ingest file study/sources/notion-export.md
```

Or paste directly via `/study ingest text`.

---

## Advanced Customization

### Custom Drill Logic

Want to drill cards in different order? Modify skill.md:

**Current sort:** `new` first, then `learning`, then `known`

**Alternative:** Random shuffle
```javascript
// In drill logic:
cards.sort(() => Math.random() - 0.5)
```

**Alternative:** Overdue first
```javascript
cards.sort((a, b) => a.next_due - b.next_due)
```

### Custom Mastery Levels

Want more granularity than new/learning/known?

```yaml
mastery: new | reviewing | familiar | confident | mastered
```

Update intervals:
- `new` → 1 day
- `reviewing` → 3 days
- `familiar` → 7 days
- `confident` → 21 days
- `mastered` → 60 days

### Custom Card Types

Add type field:

```yaml
type: concept-card | flashcard | diagram | code-snippet
```

Drill different types differently (e.g., diagrams need visual recall).

### Collaborative Cards

Study with peers:

**Pattern 1: Shared repo**
- Fork framework to team repo
- Everyone contributes cards
- Pull updates weekly
- Each person's mastery tracked locally (gitignored)

**Pattern 2: Card exchange**
- Each person maintains their own repo
- Export high-quality V2.0 cards
- Share as PR to team collection

**Pattern 3: Review each other's cards**
- Peer A creates card
- Peer B reviews and suggests improvements
- Iterate to V1.1+

---

## Extension Examples

### Example 1: PDF Research Paper Pipeline

**Goal:** Turn research papers into drillable concepts

**Setup:**
```bash
mkdir -p study/sources/papers
```

**Workflow:**
1. Download paper PDF
2. Extract text: `pdftotext paper.pdf study/sources/papers/paper.txt`
3. Ingest: `/study ingest file study/sources/papers/paper.txt --deck research`
4. Review concepts, create 3-5 cards
5. Tag with `#source/paper`

**Result:** Papers become card libraries, not read-once-forget.

### Example 2: Weekly Digest from Work Meetings

**Goal:** Capture key concepts from work meetings

**Workflow:**
1. Get meeting transcript (Teams, Zoom, Otter)
2. Paste into `/study ingest transcript --deck work-knowledge`
3. System extracts technical concepts
4. Create cards for patterns, failure modes, decisions
5. Tag with `#source/meeting`

**Result:** Work knowledge accumulates as drillable cards.

### Example 3: Cross-Reference with Brain

**Goal:** Link study cards to Brain projects

**Setup:** Add Brain path to card source:

```markdown
## Source
O'Reilly: *Managing AI Projects*, Ch. 2
Brain: [[projects/FACTS A&E/Architecture#context-engineering]]
Application: Used in BPO-77 POC
```

**Workflow:**
1. Learn concept via book (create card)
2. Apply in project (add Brain link to card)
3. Revise card to V2.0 with production learnings
4. Brain project page links back to study card

**Result:** Theory (study card) ↔ Practice (Brain project).

---

## Tips for Extending

### Start Simple

Don't customize everything at once. Use defaults first, then adapt what doesn't work.

### Track What Works

Note which customizations help vs. hurt. Complexity costs time.

### Respect the Card Inclusion Bar

Whatever source you add, apply the bar. Not everything deserves a card.

### Version Your Extensions

If you modify templates or skill logic, tag as V2.0 so you can roll back.

### Share Back

If you build a useful extension (new source integration, card format), share as PR. Others might benefit.

---

## Common Extension Questions

### "Can I use this for non-technical learning?"

Yes. The spaced repetition engine works for any domain. Adapt the Card Inclusion Bar:
- Language learning: phrases, grammar patterns, idioms
- History: causes, effects, connections (not dates)
- Business: frameworks, case patterns, failure modes

### "Can I integrate with Anki/SuperMemo?"

Export cards to Anki format:
```bash
# Script to convert cards to Anki CSV
# Front = Q section
# Back = A section
```

Or keep drilling in Claude Code (interactive tutor > flashcards).

### "Can I add video/audio sources?"

Yes, via transcript:
1. Transcribe video/audio (YouTube auto-captions, Whisper, Otter)
2. Save transcript to file
3. `/study ingest file transcript.txt`

Or manually note key concepts from video.

### "Can I organize by project instead of session?"

Yes. Change frontmatter:
```yaml
tags: [#type/concept, #project/capstone]
deck: rag
```

Drill: `/study drill --project capstone` (add filter to skill if needed).

### "Can I share cards but not my mastery progress?"

Yes. `.gitignore` already excludes mastery changes if you uncomment:
```gitignore
# Uncomment to keep mastery private:
# study/concepts/*.md
```

Or: commit initial cards (V1.0), gitignore updates (V1.1+, mastery changes).

---

## Reference

- [skill.md](../.claude/skills/study/skill.md) - skill implementation
- [Templates](../templates/) - starting points to customize
- [Card Inclusion Bar](card-inclusion-bar.md) - quality filter (adapt to your domain)
- [GUIDE.md](../GUIDE.md) - core usage patterns

The framework is yours. Extend it to fit your learning workflow.
