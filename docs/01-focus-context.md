# Focus Context (초점 맥락)

> "What am I looking at right now?"

## The Principle

**Humans naturally place semantically related information physically close together.**

When you work on a whiteboard, you don't scatter related ideas randomly across the board. You cluster them. You draw lines between connected concepts. You organize spatially because it helps you think.

AI agents should leverage this spatial intuition.

---

## What Focus Context Captures

Focus Context extracts three types of blocks based on the user's current attention:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│     Priority 1: SELECTED BLOCK                              │
│     ┌─────────────────────────────────────────────────┐     │
│     │                                                 │     │
│     │   The block the user explicitly selected        │     │
│     │   → Highest priority, full properties           │     │
│     │                                                 │     │
│     └─────────────────────────────────────────────────┘     │
│                           │                                 │
│                           ▼                                 │
│     Priority 2: EDGE-CONNECTED BLOCKS                       │
│     ┌─────────────────────────────────────────────────┐     │
│     │                                                 │     │
│     │   Blocks linked via edges (arrows/lines)        │     │
│     │   → User explicitly defined these relationships │     │
│     │   → Traverse using BFS with depth limit         │     │
│     │                                                 │     │
│     └─────────────────────────────────────────────────┘     │
│                           │                                 │
│                           ▼                                 │
│     Priority 3: NEARBY BLOCKS                               │
│     ┌─────────────────────────────────────────────────┐     │
│     │                                                 │     │
│     │   Blocks within a distance threshold            │     │
│     │   → Spatial proximity implies relevance         │     │
│     │   → Sorted by distance (closest first)          │     │
│     │                                                 │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## The Algorithms

### 1. Edge-Based Graph Traversal (BFS)

When users draw edges between blocks, they're explicitly saying "these are related." We use Breadth-First Search to find all blocks within N hops of the selected block.

**Key Parameters:**
- **Depth (N)**: How many edges to traverse. Default is 1 (direct connections only).
- **Direction**: We traverse both directions (source → target and target → source).

**Why BFS?**
- Explores all blocks at distance 1 before distance 2
- Respects the "closeness" of relationships
- Easy to limit depth

### 2. Distance-Based Proximity Search (Euclidean)

Even without explicit edges, physically close blocks are often related. We use Euclidean distance to find nearby blocks.

**Key Parameters:**
- **Radius**: Maximum distance in pixels (e.g., 1000px)
- **Max Count**: Limit the number of nearby blocks (e.g., 10)
- **Reference Point**: Center of selected block(s) or viewport center

**Formula:**
```
distance = √((x₂ - x₁)² + (y₂ - y₁)²)
```

---

## Why This Matters

### Before Focus Context

```
AI receives: "Here are all 500 blocks on the canvas..."

AI thinks: "Which of these 500 blocks matter for this task? 
           I have no idea what the user is looking at."
```

### With Focus Context

```
AI receives: "The user is looking at Block A.
             Block A is connected to Block B and Block C.
             Block D and Block E are nearby.
             Here's the content of these 5 blocks..."

AI thinks: "The user is working on Block A and its related context.
           I know exactly what's relevant."
```

---

## Use Cases

| Scenario | How Focus Context Helps |
|----------|------------------------|
| **Editing a design** | AI understands the selected component and its relationships |
| **Writing in a mind map** | AI sees the current node and connected ideas |
| **Reviewing a workflow** | AI follows the edges to understand the process |
| **Collaborating** | AI knows what the user is focused on |

---

## Key Insights

1. **Explicit > Implicit**: Edge-connected blocks are more reliable than spatially nearby blocks
2. **Limit Depth**: Going too deep (3+ hops) often brings irrelevant context
3. **Limit Count**: Too many nearby blocks dilute the focus
4. **Cache Traversals**: Graph structure doesn't change often; cache results

---

## Related Concepts

- **Graph Theory**: BFS/DFS traversal algorithms
- **Spatial Indexing**: R-trees, k-d trees for efficient proximity search
- **Attention Mechanisms**: What the model should "pay attention to"