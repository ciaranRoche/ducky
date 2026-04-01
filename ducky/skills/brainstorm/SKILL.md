---
name: brainstorm
description: Structured brainstorming session with thinking frameworks
allowed-tools: Bash, WebSearch, WebFetch, mcp__writing-samples__qdrant-find
argument-hint: [topic]
---

# Brainstorm

Run a structured brainstorming session on a topic using proven thinking frameworks.

## Arguments
- `$1+` (optional): Topic to brainstorm. If omitted, ask the user.

## Instructions

### Step 1: Establish the topic

If no argument provided, ask: **"What do you want to brainstorm about?"**

### Step 2: Query writing style

If the `qdrant-find` MCP tool is available, query for style samples matching:
1. "brainstorming session" or "collaborative discussion"
2. "casual technical exploration"

Use retrieved samples to calibrate voice and tone for the session.

### Step 3: Select a framework

Ask the user which framework to use, or auto-select based on the topic:

- **Pros/Cons**: Simple tradeoff analysis. Best for binary decisions ("should we use X or Y?")
- **First Principles**: Break down to fundamentals, rebuild from scratch. Best for novel problems or challenging assumptions.
- **Impact/Effort Matrix**: Categorize ideas by impact vs effort. Best for prioritizing a list of options.
- **Six Thinking Hats**: Parallel thinking from 6 perspectives (facts, emotions, caution, benefits, creativity, process). Best for comprehensive analysis.
- **SCAMPER**: Substitute, Combine, Adapt, Modify, Put to other use, Eliminate, Reverse. Best for improving existing things.
- **Freeform**: Unstructured ideation. Best when you just want to explore.

### Step 4: Run the session

This is interactive. For each framework:

**Pros/Cons:**
1. List pros and cons for each option
2. Ask: "What am I missing on either side?"
3. Weight the factors by importance
4. Synthesize a recommendation

**First Principles:**
1. "What are the fundamental truths here? What do we know for certain?"
2. "What assumptions are we making that we could challenge?"
3. "If we started from scratch with just the fundamentals, what would we build?"
4. Build up from the base truths

**Impact/Effort Matrix:**
1. List all ideas/options
2. Rate each on impact (low/medium/high) and effort (low/medium/high)
3. Plot into quadrants: Quick Wins, Big Bets, Fill-ins, Avoid
4. Recommend the quick wins first

**Six Thinking Hats:**
1. White Hat (Facts): What do we know? What data do we have?
2. Red Hat (Feelings): What's your gut reaction? What feels right?
3. Black Hat (Caution): What could go wrong? What are the risks?
4. Yellow Hat (Benefits): What's the best case? What are the opportunities?
5. Green Hat (Creativity): What alternatives haven't we considered?
6. Blue Hat (Process): What's the decision? What are the next steps?

**SCAMPER:**
- Walk through each letter with the user's topic
- Generate concrete ideas for each

**Freeform:**
- Let ideas flow
- Build on each other's suggestions
- Group and theme ideas as they emerge

### Step 5: Keep it interactive

- After presenting initial ideas, ask probing questions
- Build on the user's responses
- Challenge weak ideas constructively
- Combine ideas that complement each other
- Use web search if the user needs data to inform decisions

### Step 6: Produce session summary

## Output Format

```
### Brainstorm: [Topic]
**Framework**: [Selected Framework]

---

[Interactive session content, framework-specific]

---

### Session Summary

**Key Ideas:**
1. [Most promising idea with brief rationale]
2. [Second idea]
3. [Third idea]

**Top 3 Next Steps:**
1. [Concrete, actionable step]
2. [Concrete, actionable step]
3. [Concrete, actionable step]

**Open Questions:**
- [Question that needs more research or input]
- [Question that surfaced during brainstorming]

**Parking Lot** (ideas worth revisiting later):
- [Idea that's interesting but not actionable now]
```

## Notes
- Use ghostwriter tone: casual, direct, collaborative
- Keep energy high and positive during ideation
- Don't shoot down ideas too early
- Push for specificity: "What would that actually look like?"
- If the user is stuck, throw out provocative ideas to get them going
