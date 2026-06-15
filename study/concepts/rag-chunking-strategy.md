---
type: concept-card
date: 2026-06-14
tags: [#type/concept, #session/2]
source: "Generative AI on Microsoft Azure, Ch. 1 - RAG Fundamentals"
deck: rag
mastery: new
created: 2026-06-14
last-revised: 2026-06-14
version: 1.0
status: active
---

# Chunking Strategy for RAG

**Q:** You're building a RAG system for technical documentation. Walk through how chunk size impacts retrieval quality: what happens when chunks are too small vs too large, and what's your decision framework for picking chunk size?

**A:**
- **Chunk size is a precision-vs-context tradeoff:**
  - **Too small (50-100 tokens):** High precision (retrieved chunk matches query closely) but loses context. You get the exact sentence but not the surrounding explanation. User question: "How do I configure auth?" → retrieves config syntax but misses the "why" and "when" from surrounding paragraphs.
  - **Too large (1000+ tokens):** More context (full page/section) but lower precision. Semantic similarity score diluted by irrelevant content in the chunk. Retrieval may miss the right chunk entirely because the query-relevant part is buried.
  - **Just right (200-500 tokens):** Balances precision and context. Chunk contains the answer plus enough surrounding text to be useful.

- **Decision framework:**
  - **Query type:** If users ask narrow factual questions ("What's the API endpoint?"), smaller chunks work. If they ask conceptual questions ("How does auth work?"), need larger chunks for context.
  - **Document structure:** If docs are well-structured (headings, clear sections), chunk by section (semantic chunking). If unstructured (narrative prose), use fixed size with overlap.
  - **Retrieval top-K:** If you retrieve top-3 chunks, each needs to be self-contained (larger). If top-10, smaller chunks can work (user sees more pieces).
  - **LLM context window:** Larger chunks mean fewer fit in the prompt. If you have limited context budget, smaller chunks let you retrieve more diversity.

- **Overlap matters:** Use 10-20% overlap between chunks so concepts spanning chunk boundaries don't get split. Without overlap, a query about "authentication flow" might miss the chunk where the flow starts at the boundary.

- **So-what:** Start with 300-token chunks with 50-token overlap, then tune based on retrieval quality. Too many irrelevant results? Go smaller. Missing context in answers? Go larger. Chunk size isn't set-and-forget, it's a lever you tune to your query distribution.

## On-ramp

Think of chunking like cutting up a textbook for a search index. **Too small** = one sentence per index card (you find the exact sentence but lose the explanation). **Too large** = whole chapter per card (hard to find the right card). **Just right** = one concept per card with enough context to understand it.

## Common pitfalls

- Using same chunk size for all document types (technical docs vs narrative need different sizes)
- No overlap (concepts split across chunk boundaries become unsearchable)
- Chunking by arbitrary token count without considering document structure (mid-sentence splits)
- Not measuring retrieval quality after changing chunk size (tune blind, stay blind)
- Forgetting that chunk size interacts with top-K (small chunks + top-3 = too narrow)

## Connects to

[[Hybrid Search and Re-ranking]] (re-ranking can rescue larger chunks by surfacing the relevant part) · [[Embeddings Types and Tradeoffs]] (sentence embeddings work better with smaller chunks, document embeddings with larger)

## Source

*Generative AI on Microsoft Azure*, Ch. 1 - [open](https://learning.oreilly.com/library/view/-/9798341623279/) · Section: "Chunking Strategies for Vector Search"
