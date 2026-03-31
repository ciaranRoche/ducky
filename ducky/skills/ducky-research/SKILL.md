---
name: ducky-research
description: Quick inline research on a technical topic with synthesized findings. For autonomous deep-dive research, use the researcher agent instead.
allowed-tools: Bash, WebSearch, WebFetch, Glob, Grep, Read, mcp__writing-samples__qdrant-find
argument-hint: [topic]
---

# Research

Deep research on a technical topic, producing a synthesized report with sources.

## Arguments
- `$1+`: Topic to research. Be as specific as possible.

## Instructions

### Step 1: Understand the research question

Parse the topic and identify:
- What specific information is needed
- Whether this relates to the current codebase or is a general topic
- What depth of research is appropriate

### Step 2: Multi-source research

**Web research (always):**
- Run 3-5 varied web searches with different phrasings
- Prioritize official documentation, reputable tech blogs, and primary sources
- Deep dive on the most relevant results using WebFetch
- Note conflicting information between sources

**Codebase research (if relevant):**
- Search the local codebase for related patterns, implementations, or configuration
- Use Glob and Grep to find relevant files
- Read key files to understand current implementation

**CLI tools (if relevant):**
- Use `gh` to check GitHub discussions, issues, or wikis
- Use other CLI tools as appropriate

### Step 3: Synthesize findings

Combine all sources into a clear, structured report.
Use ducky-ghostwriter style for the writeup: casual, precise, teaching-oriented.

### Step 4: Present the report

## Output Format

```
### Research: [Topic]

**TL;DR**: [1-2 sentence summary of the key finding]

---

#### Background
[Why this matters, context for the research]

#### Key Findings

**1. [Finding title]**
[Explanation with specifics]
Source: [URL or reference]

**2. [Finding title]**
[Explanation with specifics]
Source: [URL or reference]

**3. [Finding title]**
[Explanation with specifics]
Source: [URL or reference]

#### Practical Implications
- [What this means for your work]
- [How to apply this]
- [What to watch out for]

#### Tradeoffs & Considerations
- [Important caveat or limitation]
- [Conflicting information and how to think about it]

---

#### Sources
- [Source Title](URL)
- [Source Title](URL)

#### Further Reading
- [Resource for deeper exploration]
- [Related topic worth investigating]
```

## When to Use This vs the Researcher Agent

| | `/ducky-research` (this skill) | `researcher` agent |
|---|---|---|
| **Speed** | Fast, inline in conversation | Slower, runs autonomously |
| **Depth** | 3-5 web searches, quick synthesis | Multi-phase deep investigation |
| **Best for** | Quick lookups, API questions, "how does X work?" | Complex topics needing cross-referencing, trade-off analysis, multi-source synthesis |
| **Interaction** | You stay in the conversation loop | Agent works independently and returns a full report |
| **Use when** | You need an answer now to unblock your current task | You need thorough research and can wait for a comprehensive report |

**Rule of thumb:** If you can answer it with a few searches and a summary, use this skill. If the answer requires comparing multiple approaches, reading long docs, or synthesizing across many sources, use the researcher agent.

## Notes
- Always cite sources for factual claims
- Distinguish between established facts and opinions
- Highlight areas of uncertainty or conflicting information
- Focus on practical applicability, not just theory
- If the research reveals the question needs reframing, say so
- Use ducky-ghostwriter tone throughout
