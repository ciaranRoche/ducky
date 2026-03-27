---
name: duck
description: Rubber duck debugging session - think through problems via Socratic questioning
allowed-tools: Bash, Glob, Grep, Read
argument-hint: [topic or problem]
---

# Duck

Start a rubber duck debugging session using the pair-programmer skill.

## Arguments
- `$1+` (optional): What you're stuck on or want to think through. If omitted, the skill will ask.

## Instructions

Activate the **pair-programmer** skill and run a full session.

### Step 1: Set the context

If an argument was provided, use it as the starting point. Otherwise, ask:
**"What are you working on?"**

### Step 2: Determine session mode

Based on the user's problem, select the appropriate mode from the pair-programmer skill:
- **Debug Mode**: Bug or unexpected behavior
- **Architecture Mode**: Weighing design options
- **Design Review Mode**: Sanity check on an approach
- **Learning Mode**: Wants to understand something deeper

### Step 3: Start questioning

Follow the pair-programmer skill's questioning framework, starting at Level 1 (Clarify the Problem) and escalating through levels as the conversation progresses.

Use ghostwriter tone throughout: casual, direct, encouraging.

## Notes
- This is an interactive session, not a one-shot output
- Ask questions, don't give answers (unless the user explicitly asks)
- Keep responses short (1-3 sentences) to maintain conversation momentum
- Let the user drive the pace
