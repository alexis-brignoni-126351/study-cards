# Spaced Repetition System

How the framework schedules card reviews to maximize retention with minimum time.

---

## The Problem Spaced Repetition Solves

**Traditional studying:** Cram everything right before you need it. Retention drops fast after the exam.

**Spaced repetition:** Review concepts at increasing intervals right before you'd forget them. Builds long-term retention.

**The forgetting curve:** You forget ~50% of new information within 24 hours, ~70% within a week unless you review. Each review pushes the curve out further.

**This system automates the schedule.** You never manually track "when to review." Just run `/study` and it shows you what's due.

---

## How It Works

### Three Mastery Levels

Every card has a `mastery` level that determines review interval:

| Mastery Level | When to Drill | Typical State |
|---------------|---------------|---------------|
| `new` | 2 days | Just created, never drilled |
| `learning` | 7 days | Drilled once or twice, still shaky |
| `known` | 30 days | Solid understanding, just maintaining |

### Drill Intervals

After each drill, the system calculates when to show the card next:

```
If mastery = new:      next drill in 2 days
If mastery = learning: next drill in 7 days
If mastery = known:    next drill in 30 days
```

**Stored in frontmatter:**
```yaml
last-drilled: 2026-06-14
next-due: 2026-06-21       # last-drilled + 7 days (learning)
```

### Advancement Logic

After each drill, you self-grade:

**Correct** - advance one level:
- `new` → `learning`
- `learning` → `known`
- `known` stays `known`

**Incorrect** - regress one level:
- `learning` → `new`
- `known` → `learning`
- `new` stays `new`

**Partial** - stay at current level:
- Review again at the same interval

**Example progression:**
```
Day 1:  Create card (mastery: new, next-due: Day 3)
Day 3:  Drill, get it right → learning (next-due: Day 10)
Day 10: Drill, get it partially → learning (next-due: Day 17)
Day 17: Drill, get it right → known (next-due: Day 47)
Day 47: Drill, get it right → known (next-due: Day 77)
```

---

## Why These Intervals?

### 2 Days (New)

New concepts need quick reinforcement before they fade. First review at 2 days catches them before the forgetting curve drops too much.

**Goal:** Get the concept into short-term memory and start building connections.

### 7 Days (Learning)

Once you've seen it twice, you can stretch the interval. Weekly reviews build medium-term retention without overwhelming your daily drill load.

**Goal:** Move from "I think I know this" to "I definitely know this."

### 30 Days (Known)

Concepts you've mastered don't need frequent review. Monthly check-ins keep them fresh without wasting time on what you already know.

**Goal:** Maintain long-term retention with minimal time investment.

---

## The Drill Queue

When you run `/study`, the system:

1. **Finds all cards where `next-due <= today`**
   - Or cards with no `next-due` (never drilled)

2. **Sorts by mastery level:**
   - `new` first (most fragile, drill these first)
   - `learning` second
   - `known` last

3. **Presents one at a time:**
   - Show question
   - You attempt
   - Show answer
   - You self-grade
   - System updates `mastery`, `last-drilled`, `next-due`

4. **Logs the session:**
   - Cards drilled, performance, mastery changes
   - Saved to `study/logs/drill-log.md`

**Result:** You always drill what needs drilling, in the order that maximizes learning.

---

## Handling Overdue Cards

If you skip drills or take a break, cards pile up as "overdue."

**What happens:**
- Card was due 2026-06-10, today is 2026-06-14 → 4 days overdue
- Still shows up in `/study` (past due = due today)

**No penalty.** The system doesn't punish you for missing drills. Just drill what's due and get back on track.

**Pro tip:** Check `/study due --overdue` to see what piled up. Drill those first.

---

## Customizing Intervals

The default intervals (2, 7, 30 days) work for most concepts. But you can tune them:

**Harder concepts** - drill more often:
- Manually set `next-due` to a closer date
- Or let it regress to `new` / `learning` naturally

**Easier concepts** - drill less often:
- Once it hits `known`, you're already at 30 days
- Can retire the card if it's truly trivial

**Emergency prep** - drill everything:
```bash
/study drill --all
```
Ignores due dates, drills every card. Use before a capstone checkpoint or session where you need everything fresh.

---

## Integration with Card Lifecycle

Spaced repetition works with card evolution:

**New card created (V1.0):**
```yaml
mastery: new
created: 2026-06-14
next-due: 2026-06-16      # 2 days later
```

**After first drill (might revise to V1.1):**
```yaml
mastery: learning
last-drilled: 2026-06-16
next-due: 2026-06-23      # 7 days later
```

**After mastery (might revise to V2.0 post-application):**
```yaml
mastery: known
last-drilled: 2026-06-23
version: 2.0              # Upgraded based on real-world use
next-due: 2026-07-23      # 30 days later
```

**Cards improve while you drill them.** Version upgrades don't reset the schedule - a V2.0 card at `known` stays `known`.

---

## Daily Habit

Spaced repetition only works if you actually do it.

**Recommended:**
- Run `/study` every morning (5-10 minutes)
- Check `/study due` at week start to see what's coming
- Run `/study status` monthly to see progress

**What if I miss a day?**
- Cards pile up as overdue
- Just drill what's due the next day
- System catches up automatically

**What if I take a week off?**
- Many cards become overdue
- Drill them over a few days to catch up
- Or use `/study drill --limit 5` to pace it out

---

## Measuring Effectiveness

The system tracks performance in `study/logs/drill-log.md`:

**After each session:**
```markdown
## 2026-06-14 - Drill Session
- Cards drilled: 5 (Session 1: 2, Session 2: 3)
- Performance: 3 correct, 1 incorrect, 1 partial
- Mastery changes:
  - Bias-Variance Tradeoff: new → learning
  - RAG Chunking: learning → new (regressed)
  - Model Selection: learning → known
```

**Look for patterns:**
- Cards that keep regressing → need revision (Q/A doesn't land)
- Topics with high correct rate → you're mastering the material
- Sessions with many overdue → need more initial coverage (audit gaps)

---

## Advanced: Prerequisites and Sequencing

Use the Prerequisites Map in Study Index to enforce mastery before new sessions:

```markdown
## Prerequisites Map

| Session | Requires Mastery Of |
|---------|---------------------|
| Session 3 | Session 2 core: Context Engineering, RAG Mechanics |
```

Then run:
```bash
/study check-prereqs session-3
```

System checks if those cards are at `known`. If not, drills them before you start Session 3.

**Result:** You build on solid foundations, not shaky concepts.

---

## Why This Works

**Expanding intervals** - review right before you'd forget (optimal timing)  
**Active recall** - retrieving from memory strengthens it (testing effect)  
**Immediate feedback** - you see the answer right after attempting (reinforcement)  
**Self-grading** - metacognition (knowing what you don't know) improves learning

Combined, these produce **80% retention at 20% time investment** compared to passive re-reading.

---

## References

- `/study drill` - run the daily drill queue
- `/study due` - see upcoming cards
- `/study status` - progress overview
- Study logs in `study/logs/drill-log.md` - performance tracking

For examples of how cards track mastery, see the example cards in `study/concepts/`.
