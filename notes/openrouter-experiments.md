# OpenRouter Experiments

OpenRouter provides a unified API that routes to many model providers (Anthropic, OpenAI, Mistral, Meta, Google, etc.) with explicit model selection. This directly addresses the **model routing opacity** problem identified in Case 001 — where directing Copilot to "use Opus" had no effect because the platform silently overrides model selection.

With OpenRouter, you specify exactly what runs. That makes it possible to hold workflow constant and vary model, or hold model constant and vary workflow.

---

## Why This Matters for Agent Pattern Research

The central hypothesis of this repo is that **workflow architecture matters more than model selection**. But that claim requires evidence — and to generate evidence, you need controlled comparisons. Currently:

- **Copilot SWE agent**: unknown model, single-shot workflow
- **Claude Code**: known model (Claude), iterative workflow
- **Claude.ai + MCP**: known model, human-in-the-loop workflow

You can't isolate the variable of interest if model and workflow always change together.

OpenRouter lets you decouple them:
- Same task, same workflow structure, different model → isolate model contribution
- Same task, same model, different workflow → isolate workflow contribution

---

## Experiment Design Sketch

### Experiment A: Model constant, workflow varies
Route the same task through:
1. A single-shot prompt (simulating Copilot's execution model)
2. A multi-turn prompt with human checkpoints (simulating Claude Code's execution model)

Same model (e.g., `claude-opus-4-5` via OpenRouter) for both. Measure: sequencing fidelity, architectural decision surface rate, instruction comprehension vs. compliance.

### Experiment B: Workflow constant, model varies
Use a single-shot prompt structure for all. Route through:
- `anthropic/claude-opus-4-5`
- `openai/gpt-4o`
- `google/gemini-2.5-pro`
- `meta-llama/llama-4-maverick`

Same task. Measure the same variables. This tells you whether the compliance/comprehension gap is model-specific or structural.

### Experiment C: Context window as a variable
Route the same task with different amounts of repository context injected:
- Issue only
- Issue + repo structure summary
- Issue + repo structure + linked issue content
- Issue + full relevant file contents

Measure whether architectural decision surface rate correlates with context window utilization.

---

## OpenRouter API Notes

```
Endpoint: https://openrouter.ai/api/v1/chat/completions
Header:   Authorization: Bearer <OPENROUTER_API_KEY>
Header:   HTTP-Referer: https://github.com/ClayNelson/agent-patterns
Model:    Specified explicitly per request (e.g., "model": "anthropic/claude-opus-4-5")
```

OpenRouter supports streaming, tool use (for models that support it), and returns model metadata in the response — including which model actually ran, useful for verifying routing fidelity.

---

## Connection to the Broader Framework

The compliance/comprehension distinction from Case 001 may turn out to be:
- A **model capability** issue (some models comprehend intent better than others)
- A **workflow structure** issue (single-shot execution structurally prevents comprehension regardless of model)
- A **context access** issue (the model would comprehend if it had access to linked issues, v2 schema, etc.)
- Some combination

OpenRouter experiments are designed to disambiguate these. The prediction from the core hypothesis: workflow structure will explain more variance than model selection.

---

## Open Questions

- Does a prompted single-shot Claude outperform a prompted single-shot GPT-4o on comprehension metrics, or is the gap small?
- Is there a minimum context injection that recovers comprehension in single-shot execution?
- How does model latency interact with human checkpoint density? (Faster models might reduce the cost of iterative workflows, shifting the workflow vs. model tradeoff.)
