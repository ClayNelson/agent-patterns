# Spec: Decision Economics Agent

**Status**: Ready for implementation  
**Target repo**: `ClayNelson/haklo`  
**Owner**: Clay Nelson  
**Research context**: `agent-patterns` — studying how workflow architecture shapes AI output quality

---

## Problem

Haklo's GitHub issue workflow currently produces qualitative analysis from specialized reviewer agents (architect, security, UX). These produce opinion and risk flags but no quantitative data. As Haklo moves toward production, architectural decisions have increasing reversal cost — but the human decision-maker has no structured economic metadata to calibrate how much weight to give any particular qualitative signal.

The gap: there is no prospective economic scoring layer at the moment of deliberation. DORA metrics are retrospective. Architecture Decision Records are narrative-only. Nothing quantifies "what does it cost to be wrong about this decision?"

This spec defines a **Decision Economics Agent** — a structured-output agent that runs alongside existing qualitative reviewers and attaches economic metadata to every GitHub issue evaluation.

---

## Scope (v1)

Build a GitHub Action in the Haklo repo that:

1. Triggers on any issue labeled `decision-eval`
2. Reads the issue title, body, and any existing comments from reviewer agents
3. Calls the Anthropic API with a structured prompt (defined below)
4. Posts a formatted comment to the issue containing:
   - A human-readable economic summary (4–6 sentences)
   - A machine-readable JSON block with the full economic payload
5. Adds a `economics-scored` label to the issue after posting

**Out of scope for v1:**
- Reading the codebase to count actual files (estimates from issue text only)
- Tracking predicted vs. actual outcomes (the calibration loop — v2)
- Integration with any external metrics platform
- Any UI or dashboard

---

## Architecture

```
haklo/
  .github/
    workflows/
      decision-economics.yml   ← GitHub Action definition
    prompts/
      decision-economics.md    ← System prompt for the economics agent
  scripts/
    decision-economics.js      ← Core logic (called by the Action)
```

The Action uses Node.js 20. The Anthropic API key is stored as a GitHub Actions secret: `ANTHROPIC_API_KEY`.

---

## GitHub Action: `decision-economics.yml`

Trigger conditions:
- `issues` event, types: `[labeled]`
- Only runs when the label added is exactly `decision-eval`

Steps:
1. Checkout repo (needed to read the system prompt file)
2. Set up Node.js 20
3. Install `@anthropic-ai/sdk` (no other dependencies)
4. Run `node scripts/decision-economics.js` with environment variables:
   - `ISSUE_NUMBER` from `${{ github.event.issue.number }}`
   - `ISSUE_TITLE` from `${{ github.event.issue.title }}`
   - `ISSUE_BODY` from `${{ github.event.issue.body }}`
   - `GITHUB_TOKEN` from `${{ secrets.GITHUB_TOKEN }}`
   - `ANTHROPIC_API_KEY` from `${{ secrets.ANTHROPIC_API_KEY }}`
   - `REPO_FULL_NAME` from `${{ github.repository }}`

After the script succeeds, add label `economics-scored` using the GitHub Actions API.

---

## Script: `decision-economics.js`

The script should:

1. Read the system prompt from `.github/prompts/decision-economics.md`
2. Fetch all existing comments on the issue via GitHub REST API (`GET /repos/{owner}/{repo}/issues/{issue_number}/comments`) and concatenate them as additional context
3. Call `claude-sonnet-4-20250514` with:
   - The system prompt
   - A user message containing: issue title, issue body, and existing comments (labeled clearly)
   - `max_tokens: 1200`
4. Parse the response to extract:
   - The narrative summary (everything before the JSON block)
   - The JSON payload (the fenced code block tagged `json`)
5. Validate that all required JSON fields are present (see schema below). If validation fails, post an error comment and exit with code 1.
6. Post a single comment to the issue with the formatted output (see output format below)
7. Exit 0 on success

Error handling: wrap the Anthropic API call in try/catch. On failure, post a comment: `⚠️ Decision economics agent failed: [error message]` and exit 1.

---

## Economic Payload Schema

The agent must return a JSON object with exactly these fields:

```json
{
  "reversibility": 3,
  "reversibility_rationale": "One-line explanation of the score",
  "files_touched_estimate": 8,
  "coupling_delta": "+3 edges",
  "coupling_delta_rationale": "One-line explanation",
  "implementation_turns_estimate": 10,
  "token_cost_estimate_usd": "$0.08",
  "reversal_cost_estimate": "2 sprints",
  "fragility_impact": "increases",
  "fragility_rationale": "One-line explanation",
  "four_plane_impact": {
    "compute": "low",
    "task": "high",
    "project": "moderate",
    "organization": "low"
  },
  "confidence": "moderate",
  "confidence_note": "One-line explanation of confidence level and key uncertainty"
}
```

Field definitions:

| Field | Type | Values | Definition |
|-------|------|--------|------------|
| `reversibility` | int | 1–5 | 1 = structural change, very hard to undo; 5 = easily reverted with no downstream impact |
| `reversibility_rationale` | string | — | Why this score was assigned |
| `files_touched_estimate` | int | ≥1 | Estimated number of files this change will touch |
| `coupling_delta` | string | e.g. `"+3 edges"`, `"-1 edge"`, `"neutral"` | Net change in system coupling implied by this change |
| `coupling_delta_rationale` | string | — | What new dependencies or removals are implied |
| `implementation_turns_estimate` | int | ≥1 | Estimated conversation turns required for clean AI-assisted implementation |
| `token_cost_estimate_usd` | string | e.g. `"$0.06"` | Using quadratic turn cost model: N(N+1)/2 turns × estimated tokens/turn × $3/1M input tokens |
| `reversal_cost_estimate` | string | e.g. `"3 sprints"`, `"1 day"` | Human time cost to undo this decision if it proves wrong |
| `fragility_impact` | string | `"increases"`, `"neutral"`, `"decreases"` | Does this make the system more or less brittle? |
| `fragility_rationale` | string | — | Why |
| `four_plane_impact` | object | see below | Impact assessment across the four economic planes |
| `confidence` | string | `"high"`, `"moderate"`, `"low"` | Agent's confidence in this scoring, given available information |
| `confidence_note` | string | — | Key uncertainty driving the confidence level |

`four_plane_impact` sub-fields (each `"low"`, `"moderate"`, or `"high"`):
- `compute` — impact on computational resources (tokens, API calls, inference cost)
- `task` — impact on implementation effort (turns, developer time)
- `project` — impact on system architecture, coupling, and technical debt
- `organization` — impact on team coordination requirements

---

## System Prompt: `.github/prompts/decision-economics.md`

Write this file with the following content (preserve the framing exactly — it establishes the agent's role and scoring model):

```
You are a Decision Economics Agent. Your role is to produce structured economic metadata for GitHub issues — quantitative scoring that helps a human decision-maker calibrate how consequential a proposed change is and what it costs to be wrong.

You are NOT an architect. You are NOT a security reviewer. You are an economic analyst. Your job is to score the decision — not to evaluate the technical merits of the approach.

Scoring model:

Reversibility (1–5): How difficult is it to undo this decision after implementation?
  1 = Structural (data model changes, protocol changes, foundational abstractions) — reversal costs multiple sprints
  2 = Deep (significant refactoring required to undo, touches multiple systems)
  3 = Moderate (meaningful but bounded work to reverse)
  4 = Surface (contained change, reversal is straightforward)
  5 = Trivial (easily reverted, no downstream impact)

Coupling delta: Does this change add, remove, or preserve system coupling?
  Consider: new dependencies introduced, new integration points, changes to shared interfaces, changes to data flow.
  Express as: "+N edges", "-N edges", or "neutral"

Implementation turns estimate: How many back-and-forth conversation turns with an AI coding agent would clean implementation realistically require?
  Simple / well-scoped / one-file changes: 3–5 turns
  Moderate complexity / multiple files / some architectural judgment: 8–12 turns
  High complexity / cross-cutting concerns / unclear scope: 15–25 turns

Token cost estimate: Apply the quadratic turn cost model.
  Formula: estimated_turns × (estimated_turns + 1) / 2 × average_tokens_per_turn × $3 / 1,000,000
  Use average_tokens_per_turn = 2000 as default.
  Express as USD to two decimal places.

Fragility impact: Does implementing this change make the overall system more or less fragile?
  increases = tighter coupling, more failure modes, harder to test in isolation
  neutral = no material change to fragility
  decreases = decouples concerns, simplifies failure modes, improves testability

Four-plane impact:
  compute = effect on runtime resources, token usage, API costs
  task = effect on implementation effort and developer time
  project = effect on architecture, coupling, technical debt accumulation
  organization = effect on coordination requirements across people and teams

Confidence:
  high = issue is well-scoped, sufficient detail to score reliably
  moderate = scope is understood but some uncertainty about implementation path
  low = issue is vague, speculative, or missing key context

Output format:
First, write a 4–6 sentence narrative summary of the economic picture. Be direct: lead with the most important economic signal (usually reversibility combined with reversal cost). Do not repeat information that will appear in the JSON. Do not use headers in the narrative.

Then output a JSON code block containing the exact schema specified below. Output nothing after the JSON block.

IMPORTANT: You are scoring based on the issue as written. If the issue is underspecified, reflect that in a low confidence score and explain what information would sharpen it. Do not invent scope that isn't there.
```

---

## Comment Output Format

The script should post a comment structured as:

```
## Decision Economics

[4–6 sentence narrative summary from the agent]

<details>
<summary>Economic payload (JSON)</summary>

```json
[full JSON payload]
```

</details>

---
*Scored by Decision Economics Agent · Confidence: [confidence value] · [confidence_note]*
```

The `<details>` collapsible keeps the comment readable at a glance while preserving the machine-readable payload.

---

## Labels Required in Haklo

Before testing, ensure these labels exist in the Haklo repo:
- `decision-eval` (trigger label) — suggested color: `#E4B8F0`
- `economics-scored` (output label) — suggested color: `#0E8A16`

The script should create these labels via the GitHub API if they don't exist (use `PUT /repos/{owner}/{repo}/labels` — idempotent).

---

## Success Criteria

The build is complete when:

1. Adding label `decision-eval` to any Haklo issue triggers the Action
2. The Action posts a comment within 60 seconds containing the narrative summary and the collapsible JSON payload
3. All required JSON fields are present and correctly typed
4. The `economics-scored` label is applied after the comment posts
5. If the Anthropic API call fails, a fallback error comment is posted and the Action exits non-zero
6. The Action does not fail if `decision-eval` is already present on the issue (idempotent label creation)

---

## Test Issue

After deploying, test against a real Haklo issue by adding the `decision-eval` label. Suggested test: find an open architectural issue in Haklo and label it. Review the scoring output against intuition — does the reversibility score match your mental model of the change?

Capture the test result as a note in `agent-patterns/notes/` for calibration tracking.

---

## Research Context

This implementation is Case 002 in the agent-patterns research log. The core research question being tested:

> Can a structured-output agent produce economically meaningful metadata from issue text alone, without reading the codebase? And does attaching that metadata to issues measurably change the human decision-maker's behavior?

The calibration loop (tracking predicted vs. actual reversibility scores over time) is the v2 research objective. Build v1 to generate the data; analyze the data before designing v2.

See `framework/variables.md` for the broader analytical framework this experiment sits within.
