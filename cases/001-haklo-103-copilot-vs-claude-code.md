# Case 001: Copilot SWE Agent vs. Claude Code

**Date:** 2026-04-07  
**Project:** Haklo (family culture app)  
**Issue:** [ClayNelson/haklo#103](https://github.com/ClayNelson/haklo/issues/103) — "Coaching philosophy: individual nudges vs. showing up for each other"  
**Context:** A philosophical + architectural issue about coaching visibility, privacy, and whether coaching should serve individuals or the family unit. Written collaboratively with Claude Code (Opus 4.6), then assigned to GitHub Copilot's SWE agent with explicit strategic instructions.

---

## The Experiment

The issue was a discussion-first design problem with clear implementation instructions added as a comment:

1. **RLS fix first** — coaching private by default, do immediately (debt cleanup)
2. **Family pulse for facilitator** — Saturday meeting prep, ship within Plus tier
3. **Defer connective lane** — wait for Culture tier consent infrastructure (#54)
4. **Remove parent-triggered coaching entirely** — anti-surveillance principle

The instruction to Copilot included "use Opus," but Copilot's SWE agent doesn't support model selection — it uses its own routing (likely GPT-4o or Sonnet). The instruction was read as context but couldn't be acted on.

---

## What Copilot Produced

**PR #104** — a single PR, single feature commit (after an "Initial plan" commit), touching 6 files:

- `019_coaching_privacy.sql` — new RLS migration
- `generate-coaching/index.ts` — edge function ownership validation
- `coaching.tsx` — removed member selector, added FamilyPulseCard
- `coaching-service.ts` — added `generateFamilyPulse`
- `ai-coach.ts` — added family pulse prompt template
- `types.ts` — extended CoachingType union

Shipped in ~27 minutes from issue assignment to PR creation.

---

## What Went Right

- **RLS policy structure is correct.** Owner-only by default, parent/guardian read access for minors, separate INSERT/UPDATE/DELETE policies.
- **Edge function security gap closed.** Added caller ownership validation that was genuinely missing.
- **Member selector cleanly removed.** `currentMember` correctly derived from `session.user.id`.
- **The code works.** Mechanically, everything connects — types, service layer, UI, migration.

---

## What Went Wrong (or Was Missed)

### 1. Ignored stated sequencing

Instructions said "ship in this sequence." Copilot shipped everything in one PR. A security fix should be small, fast to review, and independently rollback-able. Bundling a new feature with a privacy fix muddies review scope.

**Claude Code behavior:** Would have produced two PRs — security fix first, feature second.

### 2. Architectural decision made silently

The codebase has two coaching tables: `coaching_interactions` (legacy) and `coaching_nudges` (v2, migration 017, designed for family-wide nudges with `member_id = NULL`). Copilot bolted `family_pulse` onto the legacy table under the facilitator's personal `member_id`. A meaningful architectural choice, never surfaced for review.

**Claude Code behavior:** Would have flagged this as a decision point before writing code.

### 3. `awayMembers` hardcoded to `[]`

The family pulse context builder has `awayMembers: []`. The gathering data model has presence status. The entire point of Saturday facilitator prep is orienting toward who's here and who isn't. Copilot shipped the stub without comment.

**Claude Code behavior:** Would have wired to real data or explicitly noted the gap with a TODO and rationale.

### 4. No tests for the security fix

The repo has pgTAP tests for RLS policies. The "do immediately" security fix shipped with zero test coverage.

**Claude Code behavior:** Would have added pgTAP tests for the new policies.

### 5. Saturday-only is too rigid

`new Date().getDay() === 6` — the family pulse card is invisible on any other day. Facilitators prepping Friday evening see nothing.

### 6. Design system drift

`FamilyPulseCard` introduces hardcoded hex colors (`#F0F7FF`, `#4A90D9`) not in the app's `C.*` design tokens.

### 7. Dead code path

A `family_pulse` case added to a switch with a comment saying "not reachable here."

---

## The Deeper Observation: Compliance vs. Comprehension

Copilot **complied** with the instructions. Every checkbox was checked.

But it didn't **comprehend** the instructions. "Ship in this sequence" meant separate, ordered deliverables — not one bundled PR. "Remove parent-triggered coaching" had a philosophical rationale about anti-surveillance — Copilot removed the UI element but didn't consider whether storing family pulse under a personal `member_id` creates a softer version of the same problem. The issue referenced #54's consent infrastructure as a dependency — Copilot didn't check what #54 contains.

The result is code that passes a functional review but fails a design review.

---

## Model Selection Matters Less Than Workflow

Copilot's SWE agent operates in a **single-shot execution loop**: read issue → plan → code → PR. It doesn't:

- Ask clarifying questions before committing to an approach
- Consult specialized agents (the repo has a Chief Therapy Officer agent for exactly these philosophical questions)
- Split work into sequenced deliverables matching stated priorities
- Surface architectural decisions for human review before implementing them
- Distinguish between "security debt to fix now" and "feature to build carefully"

The model powering the work matters, but the agentic workflow — when to stop and think vs. when to keep coding — matters more.

---

## Implications for Agent Pattern Research

1. **Instruction fidelity ≠ instruction comprehension.** An agent that follows every directive literally can still miss the intent.
2. **Architectural decisions embedded in code are invisible decisions.** The human reviewer must notice the absence of a choice never presented.
3. **The value of "refusing to proceed" is underrated.** 5 minutes of human attention before coding saves hours of migration work after.
4. **Agent review panels partially compensate but don't close the gap.** Reviews happened on the issue; the coding agent treated them as informational context, not binding constraints.
5. **Model routing opacity is a real problem.** If the platform silently ignores a model directive, you can't attribute output quality to model capability.

---

## Variables Snapshot

| Variable | This Case |
|----------|----------|
| Agent type | Copilot SWE agent (inside GitHub) |
| Execution model | Single-shot |
| Human checkpoints before PR | 0 |
| Sequencing followed | No (1 PR instead of 2) |
| Architectural decisions surfaced | 0 of 1 identified |
| Test coverage added | 0 (gap on security fix) |
| Design system compliance | No (hardcoded colors) |
| Time to PR | ~27 minutes |

---

*Issue #103 authored collaboratively with Claude Code (Opus 4.6), assigned to GitHub Copilot SWE agent, reviewed by Claude Code post-merge.*
