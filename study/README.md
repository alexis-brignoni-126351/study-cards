# Study Directory

This is your active learning workspace.

## Structure

```
study/
├── Study Index.md       # Master index (copy from templates/)
├── concepts/            # Concept cards (drill these)
├── sessions/            # Session notes (optional)
├── logs/                # Auto-generated drill/audit logs
└── deep-dives/          # Saved O'Reilly chapters (optional)
```

## Getting Started

1. **Copy Study Index template:**
   ```bash
   cp ../templates/study-index-template.md Study\ Index.md
   ```

2. **Create your first card:**
   - Copy a template from `../templates/concept-card-template.md`
   - Or run `/study audit session-1` to create cards from session material

3. **Start drilling:**
   ```bash
   claude
   /study drill
   ```

## What Goes Where

**concepts/** - Concept cards you drill  
- Named by concept (e.g., `bias-variance-tradeoff.md`)
- Track mastery level (new/learning/known)
- Automatically scheduled for review

**sessions/** (optional) - Session notes with Coverage Maps  
- Named `Session N - Topic.md`
- Track which concepts have cards
- Used by `/study audit` to find gaps

**logs/** - Auto-generated logs  
- `drill-log.md` - drill session history
- `audit-log.md` - coverage audit history  
- `revision-log.md` - card revision history

**deep-dives/** (optional) - Saved O'Reilly chapters  
- Created when you use `/study deep --save`
- Reference material for creating additional cards

## Example Cards

See `concepts/` for three example cards:
- `bias-variance-tradeoff.md` - ML fundamentals
- `rag-chunking-strategy.md` - RAG concepts
- `model-selection-framework.md` - Decision framework

Study these to understand card format before creating your own.
