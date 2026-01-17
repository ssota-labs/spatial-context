# Action Context (액션 맥락)

> "What can I do here?"

## The Principle

**AI agents need to know their capabilities.**

When you give someone a set of tools, they need to know:
1. What tools are available
2. How to use each tool
3. When each tool is appropriate

Action Context provides AI agents with this awareness.

---

## The Action Registry Pattern

Every block type can have associated actions:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   ACTION REGISTRY                                                       │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │ Block Type: "shape"                                             │   │
│   │                                                                 │   │
│   │   Actions:                                                      │   │
│   │   ┌─────────────────────────────────────────────────────────┐   │   │
│   │   │ • updateColor: Change the shape's fill color            │   │   │
│   │   │ • resize: Change width and height                       │   │   │
│   │   │ • rotate: Rotate by degrees                             │   │   │
│   │   │ • duplicate: Create a copy                              │   │   │
│   │   └─────────────────────────────────────────────────────────┘   │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │ Block Type: "markdown"                                          │   │
│   │                                                                 │   │
│   │   Actions:                                                      │   │
│   │   ┌─────────────────────────────────────────────────────────┐   │   │
│   │   │ • updateContent: Change the markdown text               │   │   │
│   │   │ • formatHeading: Apply heading level                    │   │   │
│   │   │ • insertLink: Add a hyperlink                           │   │   │
│   │   │ • convertToCode: Change to code block                   │   │   │
│   │   └─────────────────────────────────────────────────────────┘   │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │ Block Type: "image"                                             │   │
│   │                                                                 │   │
│   │   Actions:                                                      │   │
│   │   ┌─────────────────────────────────────────────────────────┐   │   │
│   │   │ • updateSource: Change image URL                        │   │   │
│   │   │ • addCaption: Set image caption                         │   │   │
│   │   │ • crop: Crop to specific dimensions                     │   │   │
│   │   │ • addAltText: Set accessibility text                    │   │   │
│   │   └─────────────────────────────────────────────────────────┘   │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Action Definition Structure

Each action is defined with:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   ACTION DEFINITION                                                     │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                                                                 │   │
│   │   name: "updateColor"                                           │   │
│   │   ━━━━━━━━━━━━━━━━━━━                                           │   │
│   │   A unique identifier for the action                            │   │
│   │                                                                 │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │                                                                 │   │
│   │   description: "Change the shape's fill color"                  │   │
│   │   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━                    │   │
│   │   Human-readable explanation for the AI                         │   │
│   │                                                                 │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │                                                                 │   │
│   │   inputSchema: (JSON Schema)                                    │   │
│   │   ━━━━━━━━━━━━━━━━━━━━━━━━━                                     │   │
│   │   {                                                             │   │
│   │     type: "object",                                             │   │
│   │     properties: {                                               │   │
│   │       color: {                                                  │   │
│   │         type: "string",                                         │   │
│   │         description: "Hex color code (e.g., #ff0000)"           │   │
│   │       }                                                         │   │
│   │     },                                                          │   │
│   │     required: ["color"]                                         │   │
│   │   }                                                             │   │
│   │                                                                 │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │                                                                 │   │
│   │   examples: (optional)                                          │   │
│   │   ━━━━━━━━━━━━━━━━━━━━                                          │   │
│   │   - "Make it red"                                               │   │
│   │   - "Change color to blue"                                      │   │
│   │                                                                 │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │                                                                 │   │
│   │   category: (optional)                                          │   │
│   │   ━━━━━━━━━━━━━━━━━━━                                           │   │
│   │   "styling" | "content" | "layout" | "data"                     │   │
│   │                                                                 │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## From Action to LLM Tool

Actions are converted to LLM-compatible tool definitions:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   ACTION DEFINITION                    LLM TOOL (OpenAI Format)         │
│                                                                         │
│   ┌─────────────────────┐              ┌─────────────────────────────┐  │
│   │ name: "updateColor" │              │ {                           │  │
│   │                     │              │   "type": "function",       │  │
│   │ description:        │    ────▶     │   "function": {             │  │
│   │   "Change color"    │              │     "name": "shape_update   │  │
│   │                     │              │       Color",               │  │
│   │ inputSchema: {      │              │     "description": "Change  │  │
│   │   type: "object",   │              │       the shape's fill      │  │
│   │   properties: {     │              │       color",               │  │
│   │     color: {...}    │              │     "parameters": {         │  │
│   │   }                 │              │       "type": "object",     │  │
│   │ }                   │              │       "properties": {       │  │
│   └─────────────────────┘              │         "blockId": {...},   │  │
│                                        │         "color": {...}      │  │
│                                        │       },                    │  │
│                                        │       "required": [...]     │  │
│                                        │     }                       │  │
│                                        │   }                         │  │
│                                        │ }                           │  │
│                                        └─────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Key Transformation:**
- `name` becomes `function.name` (prefixed with block type)
- `inputSchema` becomes `function.parameters`
- `blockId` is automatically added as a required parameter

---

## The Execution Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   1. USER REQUEST                                                       │
│      "Make this shape red"                                              │
│                           │                                             │
│                           ▼                                             │
│   2. AI DECIDES TO USE TOOL                                             │
│      Tool: shape_updateColor                                            │
│      Args: { blockId: "shape-1", color: "#ff0000" }                     │
│                           │                                             │
│                           ▼                                             │
│   3. PARAMETER VALIDATION (JSON Schema)                                 │
│      ✓ blockId is a string                                              │
│      ✓ color is a valid hex code                                        │
│                           │                                             │
│                           ▼                                             │
│   4. ACTION EXECUTION                                                   │
│      Find shape-1 → Update color property → Save                        │
│                           │                                             │
│                           ▼                                             │
│   5. RESULT RETURNED                                                    │
│      { success: true, message: "Color updated to #ff0000" }             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Context-Aware Action Filtering

Not all actions are always available. Action Context filters based on:

### 1. Block Type Filtering

```
Selected Block: markdown

Available Actions:
  ✓ updateContent
  ✓ formatHeading
  ✓ insertLink
  ✗ updateColor (not for markdown)
  ✗ crop (not for markdown)
```

### 2. Relevance Filtering

```
User Query: "make this bigger"

Relevant Actions (by query match):
  ✓ resize (high relevance)
  ○ duplicate (low relevance)
  ✗ updateColor (no relevance)
```

### 3. Condition Filtering

```
Block State: { locked: true }

Available Actions:
  ✗ updateContent (locked blocks can't be edited)
  ✓ duplicate (can still copy)
  ✓ unlock (special action to unlock)
```

---

## Use Cases

| Scenario | How Action Context Helps |
|----------|-------------------------|
| **AI Tool Generation** | Automatically generate LLM tools from action registry |
| **Command Palette** | Show available actions for selected block |
| **Contextual Menus** | Right-click menu based on block type |
| **AI Suggestions** | Recommend relevant actions based on user intent |
| **Automation** | Chain multiple actions together |

---

## Key Insights

1. **Schema is Truth**: JSON Schema validates before execution
2. **Namespace Actions**: Prefix with block type to avoid collisions
3. **Rich Descriptions**: Better descriptions = better AI decisions
4. **Examples Help**: Provide examples for complex actions
5. **Filter Aggressively**: Don't overwhelm AI with irrelevant actions

---

## Security Considerations

- **Validate Inputs**: Always validate against schema before execution
- **Permission Checks**: Verify user can perform the action
- **Rate Limiting**: Prevent action spam
- **Audit Logging**: Log all action executions (→ Work Context)

---

## Related Concepts

- **JSON Schema**: Standard for describing JSON structure
- **Function Calling**: LLM capability to invoke functions
- **Command Pattern**: Encapsulate actions as objects
- **Plugin Systems**: Extensible action registration