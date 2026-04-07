# Analytical Framework: Variables for Agent Pattern Studies

This document tracks the evolving set of variables used to compare agent behavior across cases.

---

## Execution Model

- **Single-shot**: Read context → plan → execute → deliver. No checkpoints.
- **Iterative**: Pause, ask, split, revise. Human intervention at multiple points.
- **Hybrid**: Single-shot with post-hoc review gates.

---

## Core Variables

### Context Access
What the agent can see when it starts: issue thread (comments, linked issues), repository structure (files, schemas, tests), related issues referenced in the thread, specialized agents/personas in the repo, prior conversation history with the human.

### Human Checkpoint Density
Number of human decision points *before* code is committed. Zero in single-shot agents; variable in iterative workflows.

Related to token cost compounding: the economic value of early intervention is front-loaded. A correction at turn 1 prevents quadratic cost growth across subsequent turns. An agent that stops to ask costs minutes; the same question after merge costs hours.

### Sequencing Fidelity
Did the agent honor stated delivery sequence? Sequencing carries: risk management (independent rollback), review scope clarity, dependency management. Treating a sequence directive as ordering preference misses the intent.

### Architectural Decision Surface Rate
Of the architectural decisions implicitly made by the agent, what fraction were surfaced for human review before implementation? Unannounced decisions are invisible technical debt — the reviewer must notice the absence of a choice never presented.

### Instruction Comprehension vs. Compliance
An agent can comply with every directive literally and still miss the intent. Comprehension is measured by what was *not* done that should have been, and what *was* done that the stated rationale should have precluded.

### Design System Compliance
Does generated UI code use the project's established design tokens, or introduce hardcoded values?

### Test Coverage Delta
Were tests added for new or modified behavior? Especially important for security-critical changes.

---

## Open Questions

- How do agent review panels (specialized reviewer personas) change the coding agent's behavior, if at all?
- Is there a measurable relationship between context window available to the agent and architectural decision surface rate?
- Does model routing opacity (not knowing which model is actually running) make controlled comparison impossible, or just harder?
- What is the minimum viable human checkpoint to recover comprehension in a single-shot agent?
