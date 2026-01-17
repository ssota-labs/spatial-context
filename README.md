# Spatial Context

**A Spatial Context Engineering SDK for AI Agents on 2D Canvas**

[![npm version](https://img.shields.io/npm/v/@spatial-context/core)](https://www.npmjs.com/package/@spatial-context/core)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-blue.svg)](https://www.typescriptlang.org/)

> **"Context engineering is the delicate art and science of filling the context window with just the right information for the next step."** — Andrej Karpathy

```
User Intent → Spatial Context Engineering → Relevant Context → AI Agent Action
```

## Why This Repository Exists

When building AI agents for canvas-based applications (whiteboards, mind maps, design tools), we faced a fundamental challenge:

**How do AI agents understand spatial relationships in a 2D canvas the way humans do?**

This repository provides a first-principles approach to **Spatial Context Engineering**—the discipline of providing AI agents with structured, relevant spatial context from 2D canvases.

```
          Flat Data Dump          │      Spatial Context Engineering
                ↓                 │                 ↓                      
    "Here's everything on canvas" │  "Here's what you need to know"
         (All 999 blocks)         │    (Selected, Connected, Nearby,
                                  │     Relevant, Historical, Actions)
```

---

## The Four Pillars of Spatial Context

Inspired by how humans collaborate around a whiteboard, we identified four types of contextual understanding that AI agents need:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SPATIAL CONTEXT ENGINEERING                            │
├──────────────────┬──────────────────┬──────────────────┬────────────────────┤
│  FOCUS CONTEXT   │ SEMANTIC CONTEXT │  WORK CONTEXT    │  ACTION CONTEXT    │
│                  │                  │                  │                    │
│  "What are we    │  "What's         │  "What happened  │  "What can I       │
│   looking at?"   │   relevant?"     │   before?"       │   do here?"        │
├──────────────────┼──────────────────┼──────────────────┼────────────────────┤
│                  │                  │                  │                    │
│  ┌───┐           │    ╭───╮         │  t-3 ──────────  │  [Action A]        │
│  │ ● │ Selected  │   ╱     ╲        │  t-2 ──────────  │  [Action B]        │
│  └─┬─┘           │  ╱ Query ╲       │  t-1 ──────────  │  [Action C]        │
│    │             │  ╲  ~~~  ╱       │  t-0 ● NOW       │       ↓            │
│  ┌─┴─┐ ┌───┐     │   ╲     ╱        │                  │  Parameters +      │
│  │   │ │   │     │    ╰───╯         │  Event History   │  Conditions        │
│  └───┘ └───┘     │       │          │                  │                    │
│  Connected +     │       ▼          │  Short-term +    │  Registry +        │
│  Nearby          │  Similar Blocks  │  Long-term       │  LLM Tools         │
│                  │                  │                  │                    │
└──────────────────┴──────────────────┴──────────────────┴────────────────────┘
```

---

## Understanding the Four Contexts

### 1. Focus Context (초점 맥락)

> "What are we looking at right now?"

**The Principle**: Humans naturally place semantically related information physically close together and connect them with edges or lines. This spatial organization reflects our mental models—what we think is related, we place near each other and explicitly link. AI agents should leverage this spatial intuition.

```
                    Focus Context Assembly
                    
    ┌─────────────────────────────────────────────────┐
    │                                                 │
    │      ┌───┐                 ┌───┐                │
    │      │   │ ─ ─ nearby ─ ─  │   │                │
    │      └───┘                 └───┘                │
    │         ╲                   ╱                   │
    │          ╲    ┌───────┐   ╱                     │
    │           ╲   │       │  ╱                      │
    │            ╲  │   ●   │ ╱  ← Selected Block     │
    │             ╲ │       │╱                        │
    │              ╲└───┬───┘                         │
    │               ╲   │edge                         │
    │                ╲  ▼                             │
    │                 ┌───┐                           │
    │                 │   │ ← Connected Block         │
    │                 └───┘                           │
    │                                                 │
    │   ┌─────────────────────────────┐              │
    │   │ ┌───┐  ┌───┐  ┌───┐         │              │
    │   │ │   │  │   │  │   │         │ ← Clustered │
    │   │ └───┘  └───┘  └───┘         │   Group     │
    │   └─────────────────────────────┘              │
    │                                                 │
    └─────────────────────────────────────────────────┘
    
    Priority: Selected > Edge-Connected > Clustered Group > Nearby (by distance)
```

**What It Extracts**:
1. **Selected Block**: The block the user is currently focused on
2. **Edge-Connected Blocks**: Blocks explicitly linked via edges (BFS traversal)
3. **Clustered/Grouped Blocks**: Blocks that belong to the same group or cluster as the selected block
4. **Nearby Blocks**: Blocks within a distance threshold (Euclidean distance)

**How It Works**:
- **Edge Traversal**: Uses breadth-first search (BFS) to find all blocks connected via edges, starting from the selected block. You can limit the search depth to avoid including blocks that are too far removed.
- **Group/Cluster Detection**: If the selected block belongs to a group or cluster, all other blocks in that same group are included in the context. This captures the user's mental grouping of related concepts.
- **Distance Calculation**: Uses Euclidean distance to find blocks that are physically close to the selected block, even if they're not explicitly connected. Closer blocks are given higher priority.

---

### 2. Semantic Context (의미 맥락)

> "What's relevant to the current intent and topic?"

**The Principle**: Sometimes the most relevant information is physically distant but semantically connected. AI agents need to bridge this gap.

```
                    Semantic Search Flow
                    
    ┌───────────────────────────────────────────────────────┐
    │                                                       │
    │   User Query: "authentication flow"                   │
    │         │                                             │
    │         ▼                                             │
    │   ┌─────────────┐                                     │
    │   │  Embedding  │  → [0.23, -0.45, 0.12, ...]         │
    │   │  Provider   │                                     │
    │   └─────────────┘                                     │
    │         │                                             │
    │         ├────────────────┬─────────────────┐          │
    │         ▼                ▼                 ▼          │
    │   ┌──────────┐    ┌──────────┐     ┌──────────┐       │
    │   │  Vector  │    │   BM25   │     │  Hybrid  │       │
    │   │  Search  │    │  Search  │     │  Search  │       │
    │   └──────────┘    └──────────┘     └──────────┘       │
    │         │                │                 │          │
    │         └────────────────┴─────────────────┘          │
    │                          │                            │
    │                          ▼                            │
    │                   Ranked Results                      │
    │                                                       │
    └───────────────────────────────────────────────────────┘
```

**Search Strategies**:

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Vector** | Embedding similarity (cosine) | Semantic meaning matching |
| **BM25** | Term frequency scoring | Keyword-based search |
| **Hybrid** | Weighted combination | Best of both worlds |

**Key Insight**: Combine vector embeddings with BM25 for robust semantic search that handles both concept matching and keyword matching.

---

### 3. Work Context (작업 맥락)

> "What happened before this moment?"

**The Principle**: Humans understand their work context not just through memory, but by tracing communication, finding relevant materials, and understanding the flow of how things came to be. AI agents should have access to the history of canvas operations to understand the evolution of ideas in context.

```
                    Work Context Timeline
                    
    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │  Long-term Memory                    Short-term Memory     │
    │  (Relevance + Time Weighted)         (Recent N Events)     │
    │                                                            │
    │  ────────────────────────────────────────────────────►     │
    │  Past                                              Now     │
    │                                                            │
    │  ┌─────┐ ┌─────┐ ┌─────┐      ┌─────┐ ┌─────┐ ┌─────┐      │
    │  │ E1  │ │ E2  │ │ E3  │ ···  │ En-2│ │ En-1│ │ En  │      │
    │  │ ░░░ │ │ ░░░ │ │ ▒▒▒ │      │ ▓▓▓ │ │ ▓▓▓ │ │ ███ │      │
    │  └─────┘ └─────┘ └─────┘      └─────┘ └─────┘ └─────┘      │
    │                                                            │
    │  Relevance Score = BM25_score × exp(-t/τ)                  │
    │                                                            │
    │  where τ = time_weight_factor × 86400 (seconds)            │
    │                                                            │
    └────────────────────────────────────────────────────────────┘
```

**Event Type Examples**:
- `block.created` - A new block was added
- `block.updated` - Block content/properties changed
- `block.deleted` - Block was removed
- `edge.created` - Blocks were connected
- `tool.executed` - An AI tool was invoked

**Key Insight**: Time-weighted relevance scoring ensures recent events have higher importance while still surfacing historically relevant context.

---

### 4. Action Context (액션 맥락)

> "What can I do here?"

**The Principle**: AI agents need to know what actions are available for specific data types, what parameters they require, and under what conditions they can be executed. Just as humans know what they can do in Excel or other tools—what operations are possible with numbers, text, or images—and learn new capabilities as they explore, AI agents should understand the capabilities of each block type and discover new actions over time.

```
                    Action Registry → LLM Tools
                    
    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │  BLOCK_ACTIONS_REGISTRY                                    │
    │  ┌──────────────────────────────────────────────────────┐  │
    │  │ BlockType: "youtube"                                 │  │
    │  │ ┌────────────────────────────────────────────────┐   │  │
    │  │ │ Action: "extractScript"                        │   │  │
    │  │ │ Description: "Extract transcript from video"   │   │  │
    │  │ │ InputSchema: {                                 │   │  │
    │  │ │   language: { type: "string" },                │   │  │
    │  │ │   format: { type: "string" }                   │   │  │
    │  │ │ }                                              │   │  │
    │  │ └────────────────────────────────────────────────┘   │  │
    │  │ ┌────────────────────────────────────────────────┐   │  │
    │  │ │ Action: "summarizeVideo"                       │   │  │
    │  │ │ Description: "Generate video summary"          │   │  │
    │  │ │ InputSchema: { length: { type: "string" } }    │   │  │
    │  │ └────────────────────────────────────────────────┘   │  │
    │  └──────────────────────────────────────────────────────┘  │
    │                           │                                │
    │                           ▼                                │
    │  ┌──────────────────────────────────────────────────────┐  │
    │  │ LLM Tool Definition (OpenAI Function Calling)        │  │
    │  │ {                                                    │  │
    │  │   "name": "youtube_extractScript",                   │  │
    │  │   "description": "Extract transcript from video",    │  │
    │  │   "parameters": {                                    │  │
    │  │     "type": "object",                                │  │
    │  │     "properties": {                                  │  │
    │  │       "blockId": { "type": "string" },               │  │
    │  │       "language": { "type": "string" },              │  │
    │  │       "format": { "type": "string" }                 │  │
    │  │     }                                                │  │
    │  │   }                                                  │  │
    │  │ }                                                    │  │
    │  └──────────────────────────────────────────────────────┘  │
    │                                                            │
    └────────────────────────────────────────────────────────────┘
```

**Key Features**:
- **JSON Schema Validation**: Parameters are validated before execution
- **Dynamic Registration**: Register custom actions for your block types
- **LLM Tool Generation**: Automatically generate OpenAI-compatible tool definitions

---

## Why Not Just Dump Everything?

Providing all blocks at once seems simple, but it has significant drawbacks:

- **Context Overload**: Too much irrelevant information dilutes what actually matters
- **Token Waste**: Every unnecessary token costs money and slows responses
- **Noise**: Relevant signals get lost in the noise of irrelevant data

Spatial Context Engineering solves this by providing only the right information:

- **Focus Context**: What the user is actively looking at
- **Semantic Context**: What's meaningfully related to their intent
- **Work Context**: What happened before (the flow and history)
- **Action Context**: What's possible to do next

The key insight: **Less is more when it's the right context.** By filtering and prioritizing information based on spatial relationships, temporal history, and semantic meaning, we provide AI agents with precisely what they need to understand and act.

---

## Architecture Overview

```
┌───────────────────────────────────────────────────────────────────────────┐
│                        @spatial-context/core                              │
├────────────────┬────────────────┬────────────────┬────────────────────────┤
│                │                │                │                        │
│   focus/       │   semantic/    │   work/        │   action/              │
│   ┌─────────┐  │   ┌─────────┐  │   ┌─────────┐  │   ┌─────────┐          │
│   │ Graph   │  │   │Embedding│  │   │ Event   │  │   │ Action  │          │
│   │Traversal│  │   │Provider │  │   │ Store   │  │   │Registry │          │
│   ├─────────┤  │   ├─────────┤  │   ├─────────┤  │   ├─────────┤          │
│   │Proximity│  │   │ Vector  │  │   │ Event   │  │   │ LLM     │          │
│   │ Search  │  │   │ Index   │  │   │ Filter  │  │   │ Tools   │          │
│   └─────────┘  │   ├─────────┤  │   └─────────┘  │   └─────────┘          │
│                │   │  BM25   │  │                │                        │
│                │   └─────────┘  │                │                        │
├────────────────┴────────────────┴────────────────┴────────────────────────┤
│                                                                           │
│   composer/                              llm/                             │
│   ┌────────────────────────────┐         ┌────────────────────────────┐   │
│   │   Context Composer         │         │   Prompt Generator         │   │
│   │   (Combine all contexts)   │────────▶│   Tool Converter           │   │
│   └────────────────────────────┘         └────────────────────────────┘   │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                      @spatial-context/react-flow                           │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│   adapters/                    hooks/                  providers/          │
│   ┌─────────────────┐          ┌─────────────────┐     ┌───────────────┐   │
│   │ toSpatialGraph  │          │useSpatialContext│     │SpatialContext │   │
│   │ fromSpatialGraph│          │                 │     │Provider       │   │
│   └─────────────────┘          └─────────────────┘     └───────────────┘   │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## Package Structure

```
packages/spatial-context/
├── README.md                      # You are here
├── docs/                          # Documentation (analysis, concepts)
│   ├── concepts/                  # Conceptual explanations
│   │   ├── focus-context.md
│   │   ├── semantic-context.md
│   │   ├── work-context.md
│   │   └── action-context.md
│   ├── analysis/                  # Code analysis from SSOTA
│   │   ├── code-analysis.md
│   │   ├── focus-context.md
│   │   ├── semantic-context.md
│   │   ├── work-context.md
│   │   └── action-context.md
│   └── architecture.md            # Architecture design
│
├── core/                          # @spatial-context/core (planned)
└── react-flow/                    # @spatial-context/react-flow (planned)
```

---

## Roadmap

### Phase 1: Core Concepts & Implementation 🚧

- [x] Core concept definition
- [x] API interface design
- [x] README and documentation
- [ ] Focus Context module
- [ ] Semantic Context module
- [ ] Work Context module
- [ ] Action Context module
- [ ] Context Composer

### Phase 2: Adapters & Integration 📋

- [ ] React Flow adapter
- [ ] Vercel AI SDK integration
- [ ] Example applications

### Future Experiments 🔬

- [ ] 3D space context experiments
- [ ] Real-time voice input integration (seamlessly integrated with other contexts)
- [ ] Multi-modal sensory context processing (eye tracking, hand gestures, etc.)

---

## Learning Path

```
┌─────────────────┐     ┌──────────────────┐     ┌────────────────┐
│  docs/concepts/ │     │  docs/analysis/  │     │ docs/          │
│                 │────▶│                  │────▶│ architecture   │
│  Why & What     │     │  How it works    │     │ How to build   │
│  (Principles)   │     │  (SSOTA impl)    │     │ (SDK design)   │
└─────────────────┘     └──────────────────┘     └────────────────┘
         │                                                │
         │                                                │
         ▼                                                ▼
┌─────────────────┐                             ┌────────────────┐
│  README.md      │◀───────────────────────────▶│  Examples      │
│                 │                             │  (coming soon) │
│  Quick overview │                             │  Real usage    │
└─────────────────┘                             └────────────────┘
```

---

## Key Concepts Reference

| Concept | Description | Key Insight |
|---------|-------------|-------------|
| **Spatial Proximity** | Nearby blocks are often semantically related | Humans organize by meaning |
| **Graph Traversal** | BFS to find edge-connected blocks | Explicit relationships matter |
| **Vector Embeddings** | Semantic similarity via embeddings | Meaning > Keywords |
| **BM25 Search** | Term frequency-based scoring | Keywords still matter |
| **Hybrid Search** | Combine vector + BM25 | Best of both approaches |
| **Time Weighting** | Recent events score higher | Recency as relevance signal |
| **Event Sourcing** | Track all canvas operations | History provides context |
| **Action Registry** | Block-type specific actions | Know what's possible |
| **LLM Tools** | JSON Schema → Function Calling | Enable AI execution |

---

## Research & Inspiration

This SDK is built on principles from:

- **Context Engineering** — [davidkimai/Context-Engineering](https://github.com/davidkimai/Context-Engineering)
- **Cognitive Tools (IBM Zurich)** — Structured prompts as reasoning tools
- **Memory Systems** — Short-term and long-term memory for AI agents
- **Semantic Search** — Vector embeddings + traditional search

> "Providing 'cognitive tools' to GPT-4.1 increases its pass@1 performance on AIME2024 from 26.7% to 43.3%." — IBM Zurich

---

## Contributing

We welcome contributions! This project is in active development.

**Current Focus**: Documentation and analysis phase. Code contributions will be accepted starting Sprint 029.

---

## License

MIT License - see [LICENSE](./LICENSE) for details.

---

## Related Projects

- [Canvasdown](../canvasdown) — DSL for AI canvas manipulation
- [React Flow](https://reactflow.dev/) — Canvas framework we integrate with
- [Context Engineering](https://github.com/davidkimai/Context-Engineering) — Context engineering principles

---

<p align="center">
  Built with care by the <a href="https://github.com/ssota-labs">SSOTA Labs</a> team
</p>
