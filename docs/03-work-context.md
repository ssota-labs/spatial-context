# Work Context (작업 맥락)

> "What happened before this moment?"

## The Principle

**Humans have memory. AI agents should too.**

When you return to a whiteboard after a break, you remember what you were working on. You recall the decisions you made, the changes you considered, the evolution of your ideas.

Work Context gives AI agents the same temporal awareness.

---

## The Two Types of Memory

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   WORK CONTEXT = SHORT-TERM MEMORY + LONG-TERM MEMORY                   │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                                                                 │   │
│   │   SHORT-TERM MEMORY                                             │   │
│   │   ━━━━━━━━━━━━━━━━━━                                            │   │
│   │   • Recent N events (time-ordered)                              │   │
│   │   • "What just happened?"                                       │   │
│   │   • Immediate context for current task                          │   │
│   │                                                                 │   │
│   │   Examples:                                                     │   │
│   │   - Last 20 block operations                                    │   │
│   │   - Recent edge connections                                     │   │
│   │   - Current session activity                                    │   │
│   │                                                                 │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                                                                 │   │
│   │   LONG-TERM MEMORY                                              │   │
│   │   ━━━━━━━━━━━━━━━━━                                             │   │
│   │   • All historical events (relevance-weighted)                  │   │
│   │   • "What's relevant from the past?"                            │   │
│   │   • Searchable by query with time decay                         │   │
│   │                                                                 │   │
│   │   Examples:                                                     │   │
│   │   - Past decisions about authentication                         │   │
│   │   - Historical changes to a specific block                      │   │
│   │   - Relevant past work sessions                                 │   │
│   │                                                                 │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Event Sourcing for Canvas

Every action on the canvas is captured as an event:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   EVENT LOG                                                             │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │ Timestamp          │ Event Type      │ Target    │ Data         │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │ 2024-01-15 10:23   │ block.created   │ block-1   │ {title:...}  │   │
│   │ 2024-01-15 10:25   │ block.updated   │ block-1   │ {content:..} │   │
│   │ 2024-01-15 10:28   │ edge.created    │ edge-1    │ {src,tgt}    │   │
│   │ 2024-01-15 10:30   │ block.created   │ block-2   │ {title:...}  │   │
│   │ 2024-01-15 10:32   │ tool.executed   │ block-1   │ {action:...} │   │
│   │ 2024-01-15 10:35   │ block.deleted   │ block-3   │ {}           │   │
│   │ ...                │ ...             │ ...       │ ...          │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Common Event Types:**
- `block.created` - A new block was added
- `block.updated` - Block content or properties changed
- `block.deleted` - Block was removed
- `block.moved` - Block position changed
- `edge.created` - Blocks were connected
- `edge.deleted` - Connection was removed
- `tool.executed` - An AI action was performed

---

## Time-Weighted Relevance

Not all past events are equally important. Recent events matter more.

```
                    TIME-WEIGHTED SCORING
                    
    Relevance
    Score
       │
    1.0┤ ●                          
       │  ●                         
       │   ●                        
    0.5┤    ●                       Formula:
       │      ●                     
       │        ●  ●                score = BM25_score × exp(-t/τ)
       │             ●  ●           
    0.0┤                  ●  ●  ●   where τ = time_weight_factor × 86400
       └──────────────────────────
         Now    1d    3d    7d    14d    Time ago
         
    τ = 7 days × 86400 seconds = 604800 seconds
    
    At t = 7 days:  exp(-604800/604800) = exp(-1) ≈ 0.37
    At t = 14 days: exp(-1209600/604800) = exp(-2) ≈ 0.14
```

**The Formula:**
```
Relevance = BM25_score × exp(-time_since_event / τ)
```

Where:
- `BM25_score` = How well the event matches the query
- `time_since_event` = Seconds since the event occurred
- `τ` = Time decay constant (e.g., 7 days in seconds)

---

## Short-Term vs Long-Term

| Aspect | Short-Term Memory | Long-Term Memory |
|--------|-------------------|------------------|
| **Query** | None (just recent) | Text query for relevance |
| **Ordering** | Time (most recent first) | Relevance × Time weight |
| **Limit** | Fixed N events | Top-K by score |
| **Use Case** | Immediate context | Historical search |
| **Example** | "What did I just do?" | "When did we discuss auth?" |

---

## Use Cases

### 1. Session Resumption

```
User: "What was I working on?"

AI uses Short-Term Memory:
  - Last 10 blocks you edited
  - Recent connections you made
  - Tools you executed

AI responds: "You were working on the User Authentication flow.
             You created 3 new blocks and connected them to the Login component."
```

### 2. Historical Context

```
User: "When did we decide on JWT for authentication?"

AI uses Long-Term Memory:
  - Searches events mentioning "JWT" and "authentication"
  - Weights by time (more recent = higher score)
  - Finds relevant past decisions

AI responds: "On January 10th, you updated the Auth Flow block 
             to specify JWT tokens. Here's what was changed..."
```

### 3. Change Tracking

```
User: "What's changed in this block since last week?"

AI uses Work Context:
  - Filters events by block ID
  - Filters by time range
  - Returns chronological history

AI responds: "This block was updated 3 times:
             - Jan 12: Added validation logic
             - Jan 14: Changed error handling
             - Jan 15: Updated description"
```

---

## Key Parameters

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| **maxEvents** | Limit for short-term memory | `20` |
| **timeRange** | Filter events by date range | `{ from: Date, to: Date }` |
| **eventTypes** | Filter specific event types | `['block.updated']` |
| **timeWeightFactor** | Days for time decay | `7` |
| **topK** | Max results for long-term search | `10` |

---

## Key Insights

1. **Event Sourcing is Powerful**: Capturing all events enables time-travel debugging
2. **Decay is Essential**: Without time weighting, old irrelevant events dominate
3. **Combine Both**: Short-term for immediate context, long-term for search
4. **Session Awareness**: Track session boundaries for better context

---

## Privacy & Storage Considerations

- **What to Store**: Balance detail vs. privacy
- **Retention Policy**: How long to keep events
- **User Control**: Allow users to clear history
- **Anonymization**: Consider stripping sensitive data

---

## Related Concepts

- **Event Sourcing**: Capture all changes as events
- **CQRS**: Command Query Responsibility Segregation
- **Time-Series Data**: Handling temporal data
- **Memory Systems in AI**: Short/Long-term memory architectures