# Card Inclusion Bar

The quality filter that determines what gets a concept card vs. what stays reference-only.

---

## Core Principle

**Not everything you learn deserves a card.**

Cards are for concepts that build **engineering judgment** - the ability to make good decisions when building systems. Everything else stays in session notes or books as reference material.

---

## Include (These Earn a Card)

### 1. Mental Models

Frameworks for decomposing problems into understandable parts.

**Examples:**
- Bias-variance tradeoff (overfitting vs underfitting diagnostic)
- AI capability taxonomy (decompose system by capabilities)
- Context engineering scopes (prompt vs context vs harness)

**Test:** Does this concept give me a way to break down a complex problem into parts I can reason about?

### 2. Failure Modes & Diagnostics

How things break and how to recognize when they're breaking.

**Examples:**
- Overfitting symptoms (train accuracy >> test accuracy)
- RAG retrieval failure modes (semantic drift, chunk boundaries)
- Data leakage in train/test splits

**Test:** Does this concept teach me how to diagnose when something is wrong and what specifically went wrong?

### 3. Decision Frameworks

When to use technique X vs. Y, with tradeoff analysis.

**Examples:**
- Classic ML vs. deep learning vs. GenAI (based on data type, volume, explainability needs)
- Prompting vs. RAG vs. fine-tuning vs. agents (based on task, budget, control)
- Chunk size selection (query type, document structure, top-K)

**Test:** Does this concept help me choose between options based on specific constraints or requirements?

### 4. Non-Obvious Mechanics

How something actually works under the hood (not just what it does).

**Examples:**
- Why both the question AND knowledge base get vectorized in RAG
- Attention mechanism (how transformers weigh input tokens)
- MCP two-phase ping-pong (capabilities discovery, then task execution)

**Test:** Does understanding this mechanic change how I'd use or debug the system?

### 5. Production So-Whats

Concepts that directly impact how you build, deploy, or operate systems.

**Examples:**
- Chunk overlap prevents boundary-split concepts
- Regularization reduces overfitting but adds training cost
- Hybrid search + re-ranking balances precision and recall

**Test:** Does this concept change what I'd do when building a real system?

---

## Exclude (These Stay Reference-Only)

### 1. Historical Context

Timelines, who invented what, industry narratives.

**Examples:**
- When the Dartmouth conference happened
- AI winters and summers timeline
- Who coined the term "artificial intelligence"

**Why excluded:** Interesting background, but doesn't help you build systems or make better engineering decisions.

### 2. Program Meta

Logistics, session schedules, admin details.

**Examples:**
- Session schedule and deadlines
- Capstone submission requirements
- Cohort feedback forms

**Why excluded:** Operational details, not technical concepts that transfer to other contexts.

### 3. Vocabulary Without Tradeoffs

Terms that are just labels without decision-making attached.

**Examples:**
- "Tokenization = cutting text into pieces" (definition only)
- List of Azure building blocks (shopping list)
- "Embeddings = numerical representations" (what, not when/why)

**Why excluded:** You can look up definitions. Cards are for concepts you need to **reason with**, not just recall.

**Exception:** If the vocabulary carries a tradeoff or failure mode, it earns a card. Example: "Tokenization breaks on out-of-vocab words" becomes a card because it's a failure mode.

### 4. Examples and Case Studies

Peer presentations, company-specific examples, one-off scenarios.

**Examples:**
- "Company X used RAG for customer support"
- Peer's capstone demo (unless it reveals a generalizable pattern)
- Specific Azure service names without broader pattern

**Why excluded:** These are illustrations of concepts, not concepts themselves.

**Exception:** If the example generalizes to a reusable pattern, extract the pattern and card that. Example: "Peer's critique-loop gate design" becomes "Evaluator pattern for quality gates" card.

### 5. Trivia

Specific dates, company names, researcher backgrounds.

**Examples:**
- Year GPT-4 was released
- Which researcher went to which lab
- Specific benchmark scores

**Why excluded:** Not actionable for engineering work. You can look it up when needed.

---

## Handling Ambiguous Cases

Some concepts sit on the line. When unsure:

### Ask These Questions

1. **Will I need to reason with this concept when building?**
   - Yes → probably a card
   - No → probably reference-only

2. **Does this involve a tradeoff, failure mode, or decision?**
   - Yes → probably a card
   - No → probably reference-only

3. **Is this a generalizable pattern or a specific instance?**
   - Generalizable → card
   - Specific → reference-only

4. **Would a smart engineer who doesn't know this make worse decisions?**
   - Yes → card
   - No → reference-only

### Log the Decision

If a concept is ambiguous, log it in your Study Index under "Coverage Decisions" with rationale:

**Example:**
```markdown
## Coverage Decisions

**Session 1:**
- **Narrow/General/Super AI taxonomy:** Reference-only. Rationale: 
  Philosophically interesting but not actionable for engineering 
  decisions. Doesn't change how you build systems.

**Session 2:**
- **Azure building blocks list:** Reference-only. Rationale: Shopping 
  list of services, not a decision framework. The architecture-thinking 
  method (cost after design) is the card-worthy concept, not the list.
```

This creates a record so you don't re-debate the same concept later.

---

## Quality Test (Before Creating a Card)

Before creating a card, ask:

1. **Can I write a reasoning-level question?**
   - Good: "Walk through how chunk size impacts retrieval quality..."
   - Bad: "What is chunking?"

2. **Does the answer end in a production so-what?**
   - Good: "...so start with 300-token chunks and tune based on retrieval quality"
   - Bad: "...so chunking is important" (vague, no action)

3. **Will drilling this concept make me better at building systems?**
   - Yes → create the card
   - No → leave it in session notes

---

## Examples: Include vs. Exclude

| Concept | Include or Exclude? | Why |
|---------|---------------------|-----|
| Bias-variance tradeoff | ✓ Include | Mental model + diagnostic framework |
| Year deep learning became popular | ✗ Exclude | Historical trivia |
| RAG question vectorization | ✓ Include | Non-obvious mechanic + failure mode |
| List of Azure AI services | ✗ Exclude | Vocabulary shopping list |
| When to use prompting vs. RAG | ✓ Include | Decision framework with tradeoffs |
| Peer's capstone demo | ✗ Exclude | Specific example (unless pattern generalizes) |
| Train/val/test split strategy | ✓ Include | Failure mode (data leakage) |
| "Tokenization definition" | ✗ Exclude | Vocabulary without tradeoff |
| "Why tokenization breaks on OOV words" | ✓ Include | Failure mode |

---

## The Bar Enforces Itself

As you drill cards:
- **Good cards** (cross the bar) - you use them, they improve via revision
- **Bad cards** (don't cross the bar) - you avoid them, eventually retire them

The system self-corrects. If a card never helps when building, it fails the bar and gets retired.

---

## References

- Study Index template has Coverage Decisions section for logging ambiguous cases
- `/study audit` enforces this bar when creating cards
- `/study retire` removes cards that don't meet the bar

For examples of cards that cross the bar, see the example cards in `study/concepts/`.
