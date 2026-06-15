---
type: index
date: YYYY-MM-DD
tags: [#type/index]
---

# Study Index

Your active-recall learning environment. This is where you **drill it until it sticks** and **read deeper** via O'Reilly texts.

**Mantra:** Attend → Reflect → Distill. This is the Distill.

---

## How to Use It

| Command | What it does |
|---------|--------------|
| `/study` | Quiz me on the concept cards (active recall). Tracks mastery per card. |
| `/study new` | Drill only cards not yet seen (`mastery: new`). |
| `/study audit session-N` | Find concepts in session material that cross the Card Inclusion Bar and create cards. |
| `/study deep <topic>` | Pull O'Reilly chapter content and optionally create cards from it. |
| `/study due` | Show which cards are due for review based on spaced repetition. |
| `/study status` | Overview of learning progress and suggested next action. |

---

## What Gets a Concept Card

**Include:**
- **Mental models** - frameworks for decomposing problems
- **Failure modes & diagnostics** - how things break and how to recognize it
- **Decision frameworks** - when to use X vs Y, tradeoff analysis
- **Non-obvious mechanics** - how something actually works under the hood
- **Production so-whats** - concepts that directly impact how you build or deploy

**Exclude (reference-only):**
- **Historical context** - timelines, who coined what, industry narratives
- **Program meta** - logistics, schedules, admin
- **Peer examples or case studies** - unless they generalize to a reusable pattern
- **Trivia** - specific dates, company names, researcher backgrounds
- **Vocabulary definitions alone** - if the term is just a label without a decision/tradeoff attached

**Ambiguous? Log it in Coverage Decisions section below with rationale.**

---

## Review Schedule Logic

**Spaced Repetition:**
- `new` - drill within 2 days
- `learning` - drill weekly (7 days)
- `known` - drill monthly (30 days)

**Curriculum Progression:**
- Check prerequisite mastery before starting new sessions
- If prerequisites at `learning` or below, refresh before diving in

**Prerequisites Map:**

| Session | Requires Mastery Of |
|---------|---------------------|
| Session 2 | Session 1 core concepts |
| Session 3 | Session 2 core concepts |
| Add as needed... | |

---

## Concept Card Lifecycle

Cards are **living documents** that sharpen over time.

### When to Create Cards

1. **Post-session initial capture** (within 1-2 days)
   - After session note + Coverage Map
   - Create cards for obvious mental models
   - First-draft quality (V1.0)

2. **Post-deep-dive additions** (whenever)
   - O'Reilly chapter reveals failure mode or tradeoff
   - Add to originating session's Coverage Map as "Post-Session Addition"

### When to Revise Cards

1. **After first drill cycle** - Q/A doesn't land → revise (V1.1)
2. **When later session deepens concept** - add connections, update pitfalls (V1.2+)
3. **When you apply it in practice** - rewrite based on lived experience (V2.0)

### The Quality Ratchet

- **V1.0** (post-session): "Here's the concept"
- **V1.1** (post-drill): "Here's the concept + the gotcha I missed"
- **V1.2+** (post-deep-dive): "Here's the concept + the tradeoff"
- **V2.0** (post-application): "Here's what you actually need to know to apply it"

---

## Session Coverage Status

| Session | Mapped Concepts | Ambiguous / Deferred | Coverage Audit |
|---------|-----------------|----------------------|----------------|
| Session 1 | N cards | Topics excluded (see Coverage Decisions) | ✅ YYYY-MM-DD |
| Session 2 | N cards | ... | ✅ YYYY-MM-DD |

---

## Coverage Decisions (Ambiguous Cases)

**Session 1:**
- **[Topic Name]:** Reference-only. Rationale: [why it doesn't cross the bar]

**Session 2:**
- **[Topic Name]:** Under review. Rationale: [what you're waiting to decide]

---

## Concept Cards

[Organize by topic for drilling, or by session for coverage tracking]

### By Topic

**[Topic Name]** (`deck: topic-slug`)
- [[Card Name 1]] - one-line summary
- [[Card Name 2]] - one-line summary

### By Session

**Session 1 - [Session Title]**
- [[Card Name]] - from session material
- [[Card Name]] - from O'Reilly Ch. N

**Session 2 - [Session Title]**
- [[Card Name]] - from session material
