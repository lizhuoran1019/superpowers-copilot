---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks end-to-end, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Tell your human partner that Superpowers works much better with access to subagents. The quality of its work will be significantly higher if run on a platform with subagent support (such as Claude Code or Codex). If subagents are available, use superpowers:subagent-driven-development instead of this skill, and carry forward the same execution constraints from this skill.

**Default execution stance:** Once execution starts, continue through the full task list without stopping to ask whether to continue. Pause only for a real blocker or a genuine user decision that cannot be inferred.

<HARD-GATE>
## User Interaction

**Use the `askQuestions` tool whenever execution requires a user decision, clarification, confirmation, approval, or consent.** Do not rely on bare text prompts.

For each interaction:
- Use askQuestions to present 2-4 options
- Ask only one question at a time
- Prefer multiple-choice over open-ended questions
- Present structured options instead of plain-text prompts

This applies to ALL interactive pauses, including:
- Plan review gates when concerns are found
- Blocker escalation
- Main/master branch consent
- Any proceed / continue / stop / revise / approve decision

Never wait for bare text responses such as "ok", "continue", or "yes". If the user replies in freeform text anyway, immediately follow up with `askQuestions` to collect an explicit choice before proceeding.
</HARD-GATE>

**Execution Flow (Non-blocking)**

Do not pause for plain-text confirmation. Do not ask whether to continue between tasks. Only stop when a real user decision is required, and implement that pause via `askQuestions`.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting, using `askQuestions` when a decision or approval is required
4. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. When writing or editing code, keep each generated chunk focused and under 200 lines; split larger implementations into sequential chunks
4. Add concise Chinese comments for non-obvious logic to improve readability
5. Run verifications as specified
6. Mark as completed

### Step 3: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

Do not stop for routine progress updates or to ask whether the human wants you to keep going.

**Ask for clarification rather than guessing.** Use `askQuestions` if the blocker requires a user decision.

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Keep executing until all tasks are complete unless a real blocker forces a pause
- Do not ask whether to continue between tasks or code chunks
- Keep code generation/edit batches under 200 lines
- Add concise Chinese comments to generated code where the logic is not self-evident
- Don't skip verifications
- Reference skills when plan says to
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent collected via `askQuestions`

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** - REQUIRED: Set up isolated workspace before starting
- **superpowers:writing-plans** - Creates the plan this skill executes
- **superpowers:finishing-a-development-branch** - Complete development after all tasks
