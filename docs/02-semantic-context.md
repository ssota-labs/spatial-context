# Semantic Context (의미 맥락)

> "What's relevant to the user's question?"

## The Principle

**Sometimes the most relevant information is physically distant but semantically connected.**

A user might ask "How do we handle authentication?" while looking at a block about "User Dashboard." The authentication logic might be on the other side of the canvas, but it's exactly what they need.

Semantic Context bridges the gap between physical distance and meaning.

---

## The Challenge

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              CANVAS                                     │
│                                                                         │
│   ┌──────────────┐                               ┌──────────────┐       │
│   │ User         │                               │ Auth         │       │
│   │ Dashboard    │                               │ Flow         │       │
│   │   ●          │                               │              │       │
│   │ (selected)   │                               │  (relevant!) │       │
│   └──────────────┘                               └──────────────┘       │
│                                                                         │
│                     Focus Context can't reach this                      │
│                     (too far, no edges)                                 │
│                                                                         │
│                     But Semantic Context can!                           │
│                     (matches the query meaning)                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Two Approaches to Semantic Search

### 1. Vector Embeddings (Semantic Similarity)

Transform text into high-dimensional vectors where similar meanings are close together.

```
"authentication flow"  →  [0.23, -0.45, 0.12, 0.89, ...]
"login process"        →  [0.21, -0.42, 0.15, 0.87, ...]  ← Similar!
"color scheme"         →  [-0.67, 0.33, 0.45, -0.12, ...]  ← Different

Similarity = cosine(query_vector, block_vector)
```

**Pros:**
- Understands synonyms and paraphrases
- Captures semantic meaning beyond keywords
- Works with natural language queries

**Cons:**
- Requires an embedding provider (API cost)
- May miss exact keyword matches
- Embedding quality varies by model

### 2. BM25 (Term Frequency Scoring)

A classic information retrieval algorithm that scores documents based on term frequency.

```
Query: "authentication flow"

Block A: "The authentication flow starts with..."  → High score (exact match)
Block B: "User login requires authentication..."   → Medium score (partial match)
Block C: "The color scheme is blue..."             → Low score (no match)
```

**Pros:**
- No external API needed
- Exact keyword matches score high
- Fast and efficient

**Cons:**
- Misses semantic equivalents ("auth" vs "authentication")
- Sensitive to wording differences
- No understanding of meaning

### 3. Hybrid Search (Best of Both)

Combine vector and BM25 for robust results.

```
Final Score = (vectorWeight × vectorScore) + ((1 - vectorWeight) × bm25Score)

Example with vectorWeight = 0.7:
Final = 0.7 × cosine_similarity + 0.3 × bm25_score
```

**Why Hybrid Works:**
- Catches semantic matches (vector)
- Doesn't miss exact keywords (BM25)
- Tunable based on use case

---

## Semantic Context Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   Step 1: QUERY PROCESSING                                              │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                               │     │
│   │   User Query: "How do we handle user authentication?"         │     │
│   │                                                               │     │
│   │   → Extract key concepts                                      │     │
│   │   → Generate embedding (if using vector search)               │     │
│   │   → Tokenize for BM25                                         │     │
│   │                                                               │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                         │
│   Step 2: SEARCH EXECUTION                                              │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                               │     │
│   │   Vector Search          BM25 Search                          │     │
│   │   ┌─────────────┐        ┌─────────────┐                      │     │
│   │   │ Compare     │        │ Match       │                      │     │
│   │   │ embeddings  │        │ keywords    │                      │     │
│   │   │ (cosine)    │        │ (tf-idf)    │                      │     │
│   │   └─────────────┘        └─────────────┘                      │     │
│   │          │                      │                             │     │
│   │          └──────────┬───────────┘                             │     │
│   │                     ▼                                         │     │
│   │              Hybrid Scoring                                   │     │
│   │                                                               │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                         │
│   Step 3: RESULT RANKING                                                │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                               │     │
│   │   1. Auth Flow Block        (score: 0.95)                     │     │
│   │   2. Login Component        (score: 0.87)                     │     │
│   │   3. Session Management     (score: 0.82)                     │     │
│   │   4. User Settings          (score: 0.45) ← Below threshold   │     │
│   │                                                               │     │
│   │   → Return top-K results above minScore                       │     │
│   │                                                               │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Key Parameters

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| **strategy** | Search approach | `'hybrid'` |
| **vectorWeight** | Weight for vector vs BM25 | `0.7` |
| **topK** | Maximum results to return | `10` |
| **minScore** | Minimum relevance threshold | `0.5` |

---

## Use Cases

| Scenario | How Semantic Context Helps |
|----------|---------------------------|
| **Question Answering** | Find blocks that answer the user's question |
| **Knowledge Discovery** | Surface related concepts across the canvas |
| **Smart Search** | Natural language search beyond keywords |
| **Contextual Suggestions** | Recommend relevant blocks to explore |

---

## Key Insights

1. **Hybrid > Pure**: Neither vector nor BM25 alone is sufficient
2. **Index Once, Search Many**: Pre-compute embeddings for all blocks
3. **Threshold Matters**: Too low = noise, too high = missed results
4. **Context Length**: Don't overload with too many results

---

## Embedding Provider Considerations

When choosing an embedding provider:

| Provider | Dimensions | Quality | Cost |
|----------|------------|---------|------|
| OpenAI text-embedding-3 | 256-3072 | Excellent | Moderate |
| Voyage AI | 1024 | Excellent | Moderate |
| Local (e.g., Sentence Transformers) | 384-768 | Good | Free |

---

## Related Concepts

- **Vector Databases**: Pinecone, Qdrant, Weaviate
- **Information Retrieval**: BM25, TF-IDF, Okapi
- **Embeddings**: Word2Vec, BERT, Modern LLM embeddings
- **RAG**: Retrieval-Augmented Generation