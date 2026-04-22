# Case 002: Verified ≠ Durable

**Date:** 2026-04-22
**Project:** Haklo (family culture app)
**Context:** A late-night Claude Code session shipped and locally verified six features over the course of a working session — Apple Sign In, password reset flow, peer nudges, push tokens, feedback modal, plus a small meeting-history bug fix. None of them were committed. All six sat untracked on a stale `feat/privacy-implementation` branch. A single hardware failure would have lost the entire night's work.

Status: **stub — to be expanded with full post-mortem.**

---

## The Pattern in One Sentence

The agent treated *"works locally"* as a terminal state when it is in fact an intermediate checkpoint that should trigger the next pipeline stage: commit.

---

## The Observation

Across a multi-hour session, each feature followed this loop:

1. Propose implementation
2. Write code
3. User tests locally
4. User confirms it works
5. Move to next feature

Step 6 — *"commit this"* — never fired. The agent's internal definition of done was "verified," not "durable."

When the user finally flagged the pile, the agent discovered:

- Six features' worth of work, untracked, on one laptop
- The branch holding them was stale against `master` (which had already deleted files the agent was "fixing")
- Several "fixes" made during the session were solving problems `master` had already solved — work done against a phantom baseline
- The only genuinely net-new work from the night was a 3-line status-filter fix and a date-formatting tweak

---

## Why This Is Case 001 in Different Clothes

Case 001 identified **compliance ≠ comprehension**: the agent followed every instruction literally and still missed the intent.

Case 002 is the same shape on a different axis: **verified ≠ durable**. The agent optimized for the visible success signal ("it works") and never ran the loop that converts a local success into a durable artifact. Both cases share a root structure:

> The agent treats a proximate success state as a terminal state, when it should be a trigger for the next pipeline stage.

In Case 001, "code compiles and PR is green" was terminal; architectural decisions never got surfaced.
In Case 002, "feature works in dev" was terminal; artifacts never got committed.

Neither failure is about model capability. Both are about **workflow contract**.

---

## The Economics

Through the four-plane lens:

| Plane | Cost of the failure mode | Cost of the fix (2-min WIP commit) |
|-------|---------------------------|------------------------------------|
| Compute | 0 | ~0 |
| Task | Re-implement 6 features on hardware loss | ~1 commit turn |
| Project | Stale branch fighting master, phantom fixes, mounting confusion | Clean artifact history, external witness |
| Organization | Knowledge provenance broken — no record of what was tried | Full audit trail in git |

Reversibility asymmetry is stark: the reversal cost of an unnecessary WIP commit is approximately zero (rebase or squash later). The reversal cost of lost work is catastrophic and often non-recoverable (you can't reconstruct implementation detail you didn't write down).

An agent running even a crude decision-economics check at the end of each "verified" state would always recommend committing. The question isn't "should we commit?" — it's "why aren't we defaulting to it?"

---

## The Fragility Frame (Taleb)

What the session produced was maximally fragile state:

- **Single point of failure:** one laptop, one filesystem, no redundancy
- **No external witness:** no other system, human, or tool had visibility into the work
- **Silent drift:** the baseline (`master`) kept moving while the work sat unshipped, so "done" was being measured against a stale reference

A WIP commit — even one labeled `WIP: untriaged in-flight features, not for merge` — converts this into an antifragile position. The work survives hardware loss. It becomes reviewable by other tools, other models, other humans. It participates in the knowledge provenance chain.

Cheap external witness beats expensive internal memory. This is the same principle that moves decisions out of local agent memory and into GitHub issues.

---

## The Fix Is a Contract, Not a Reminder

Adding "remember to commit" to an agent's instructions is the wrong altitude. Memory-based fixes assume the agent will, at each decision point, recall and prioritize the rule against whatever else is in context. That's the abstraction Haklo design doc #91 is designed to avoid at a different layer: don't trust good behavior inside a boundary, enforce the contract *at* the boundary.

The contractual version:

> **"Verified" is not a valid terminal state for a code change. The only valid terminal states are: (a) committed to a named branch, or (b) explicitly discarded.**

Under this contract, the agent's self-check after any successful test run is not *"did that work?"* but *"is this change now durable?"* The failure mode becomes syntactically visible rather than requiring vigilance.

---

## Implications for Agent Pattern Research

1. **Definition-of-done is a workflow artifact, not a disposition.** Agents don't need better memory of when to commit; they need a contract that makes "verified but uncommitted" an inadmissible state.
2. **Proximate success signals are dangerous terminal states.** Any signal that feels like completion ("tests pass," "code compiles," "PR is green," "it works") should be checked against a durability predicate before being treated as done.
3. **Artifact durability belongs in the economic model.** The four-plane view already captures reversal cost; an agent that reasons about reversal cost should refuse to end a turn holding fragile uncommitted work.
4. **External witnesses beat internal memory.** Git commits, GitHub issues, filed artifacts — each one converts fragile local state into durable shared state that other agents and humans can review.
5. **Case 001 + Case 002 share a general form.** Both are *terminal-state misclassification* failures. Worth testing whether other agent pattern failures reduce to the same shape.

---

## Variables Snapshot

| Variable | This Case |
|----------|-----------|
| Agent type | Claude Code (Opus) in interactive pair-programming mode |
| Execution model | Conversational, feature-by-feature |
| Commit checkpoints enforced | 0 |
| Features verified locally | 6 |
| Features committed | 0 |
| Baseline drift noticed mid-session | No |
| Reversal cost if hardware failed | Catastrophic (hours of re-implementation) |
| Reversal cost of 2-min WIP commit | ~0 |

---

## Open Questions (to expand in full write-up)

- What's the right trigger? Per-feature commit? Per-verified-change commit? Time-boxed?
- How does this interact with Case 001's sequencing failure? (If Copilot had been forced to commit after the security fix, would it have felt less pressure to bundle the feature?)
- Is there a generalizable "terminal-state predicate" that covers both cases + future ones?
- Does the Decision Economics Agent spec need an artifact-durability field alongside reversibility and coupling delta?

---

*Session captured from Claude Code conversation on 2026-04-22. Full transcript and diff summary to be added when writing up the expanded post-mortem.*
