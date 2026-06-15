# Concept Cards

This directory contains your drillable concept cards.

## What's a Concept Card?

A card with:
- **Reasoning-level question** (not "what is X?" but "walk through how X works and when it breaks")
- **Answer ending in production so-what** (why this matters when building)
- **On-ramp** (simpler explanation for when stuck)
- **Common pitfalls** (failure modes and mistakes)
- **Connects to** (related concepts)
- **Source** (book chapter or session)

## Card Format

Each card has frontmatter tracking:
- `mastery` (new/learning/known) - automatically updated by `/study drill`
- `deck` (topic grouping) - for focused drilling
- `version` (semantic versioning) - evolves as you learn
- `last-drilled` - last drill date
- `next-due` - next scheduled review

See the three example cards in this directory for reference.

**Note:** Example cards contain wikilinks to other cards (like `[[Feature Engineering]]`, `[[Hybrid Search and Re-ranking]]`) that don't exist yet. These show the card format - how to link related concepts. The links won't resolve until you create those cards.

## Creating Cards

**Method 1: Manual (from template)**
```bash
cp ../../templates/concept-card-template.md my-concept.md
# Edit to fill in Q/A
```

**Method 2: Audit (from session material)**
```bash
claude
/study audit session-1
# System finds gaps and creates cards
```

**Method 3: Deep-dive (from O'Reilly)**
```bash
claude
/study deep "topic name"
# Pull chapter, extract concepts, create cards
```

## Drilling Cards

```bash
claude
/study                    # Drill cards due today
/study --topic rag        # Drill only RAG cards
/study --session 1        # Drill only Session 1 cards
/study new                # Drill only new cards
```

## Organizing Cards

Cards are organized by **topic** (`deck: topic-name`) for drilling and by **session** (`#session/N` tag) for coverage tracking.

**Example:**
```yaml
deck: rag
tags: [#type/concept, #session/2]
```

Allows you to drill all RAG cards (`/study --topic rag`) while tracking which session introduced the concept.
