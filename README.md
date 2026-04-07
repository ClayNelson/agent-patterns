# Agent Patterns

A research log studying usage patterns between **agents operating inside GitHub** (Copilot SWE agent, Copilot workspace) and **Claude operating outside GitHub** (Claude Code, Claude.ai with MCP).

The central question: how do agentic workflow structure, context access, and human-in-the-loop dynamics shape output quality — independent of model capability?

---

## Core Hypothesis

Model selection matters less than **workflow architecture**. The key variables are:

- **When does the agent stop and ask?** — Single-shot execution vs. iterative clarification
- **What context is visible?** — Issue thread, repo structure, related issues, specialized agents
- **What can the agent refuse to do?** — Surfacing architectural decisions vs. silently implementing them
- **How is work sequenced?** — Bundled PRs vs. ordered, independently reviewable deliverables

This maps to a compounding dynamics principle: the value of early human intervention is front-loaded. A question asked before code is written costs minutes; the same question asked after a PR is merged costs hours or architectural debt. See the [token cost compounding model](https://claynelson.github.io) for the economic framing.

---

## Repository Structure

```
cases/          # Individual experiment write-ups
framework/      # Evolving analytical framework
notes/          # Raw observations, fragments
```

---

## Cases

| # | Date | Project | Issue | Agent Compared | Key Finding |
|---|------|---------|-------|----------------|-------------|
| 001 | 2026-04-07 | Haklo | [#103](https://github.com/ClayNelson/haklo/issues/103) | Copilot SWE Agent vs. Claude Code | Compliance ≠ comprehension; single-shot execution misses strategic sequencing |

---

## Related Work

- [Token cost compounding model](https://claynelson.github.io) — the economic argument for early human intervention
- [Haklo](https://github.com/ClayNelson/haklo) — the primary development context generating these observations
