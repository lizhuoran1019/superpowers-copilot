---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Output constraint:** When presenting the plan, code examples, or any large generated content to the user, split it into sequential chunks of no more than 200 lines per message.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

<HARD-GATE>
## User Interaction

**Use the `#vscode/askQuestions` tool to ask the user questions.** Present a carousel UI rather than plain text options.

For each question:
- Use askQuestions to present 2-4 options
- Ask only one question at a time
- Prefer multiple-choice over open-ended questions - they are easier to answer

This requirement applies to ALL interactive pauses, not just explicit "questions", including:
- Scope decomposition gates
- Plan approval or revision gates
- Execution handoff gates
- Any "continue / proceed / confirm" moment

Never wait for bare text responses (e.g., "ok", "continue", "yes"). If the user provides freeform text anyway, immediately follow up with `vscode/askQuestions` to collect an explicit choice before proceeding.
</HARD-GATE>

**Execution Flow (Non-blocking)**

This skill shall not stop and wait for bare text confirmation at each step during execution. The agent shall continue to advance according to the process; interactive pauses shall only be initiated when user decision, confirmation, or input is genuinely required. All such pauses must be implemented via `vscode/askQuestions` (or an equivalent tool specified in the documentation) - after the tool is used to display a selection or input interface, the agent may resume executing subsequent steps upon receiving the user's response. Do not use modes that rely solely on textual prompts such as "Please continue" or passive waiting.

When delivering long plan content, present it in ordered segments rather than one large dump. Each segment must stay within the 200-line limit.

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

If user input is needed to decide whether to split scope, collect that decision via `vscode/askQuestions` with clear options such as "Split into multiple plans", "Keep a single plan", or "Need more explanation".

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## Remember
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- Reference relevant skills with @ syntax
- Split long plan output into sequential sections, each no more than 200 lines
- DRY, YAGNI, TDD, frequent commits

## Plan Review Loop

After completing each chunk of the plan:

1. Dispatch plan-document-reviewer subagent (see plan-document-reviewer-prompt.md) with precisely crafted review context — never your session history. This keeps the reviewer focused on the plan, not your thought process.
   - Provide: chunk content, path to spec document
2. If ❌ Issues Found:
   - Fix the issues in the chunk
   - Re-dispatch reviewer for that chunk
   - Repeat until ✅ Approved
3. If ✅ Approved: proceed to next chunk (or execution handoff if last chunk)

**Chunk boundaries:** Use `## Chunk N: <name>` headings to delimit chunks. Each chunk should be ≤1000 lines and logically self-contained.

**Review loop guidance:**
- Same agent that wrote the plan fixes it (preserves context)
- If loop exceeds 5 iterations, surface to human for guidance
- Reviewers are advisory - explain disagreements if you believe feedback is incorrect

If a reviewer disagreement or scope tradeoff needs user input, present the decision via `vscode/askQuestions` rather than waiting for a freeform reply.

## Execution Handoff

After saving the plan:

Ask the user to choose the next step via `vscode/askQuestions` rather than a bare text prompt.

Example prompt text:

> "Plan complete and saved to `docs/superpowers/plans/<filename>.md`. What do you want to do next?"

Provide 2-4 options such as:
- "Proceed with execution"
- "Request plan changes"
- "Not ready to execute yet"

**Execution path depends on harness capabilities:**

**If harness has subagents (Claude Code, etc.):**
- **REQUIRED:** Use superpowers:subagent-driven-development
- Do NOT offer a choice - subagent-driven is the standard approach
- Fresh subagent per task + two-stage review

**If harness does NOT have subagents:**
- Execute plan in current session using superpowers:executing-plans
- Batch execution with checkpoints for review
