# Watchlist: Companies and People to Track

Signals relevant to the agent-patterns research and GitHub competitive/product landscape.

---

## Entire — Thomas Dohmke

**Website:** https://entire.io  
**LinkedIn:** https://www.linkedin.com/company/entireio/posts/?feedView=all  
**First observed:** 2026-04-07  
**Signal level:** 🔴 High — directly addresses the substrate problems this repo is studying

### What it is

Founded by Thomas Dohmke (GitHub CEO 2021–August 2025) with a $60M seed round at $300M valuation, led by Felicis. Notably, Microsoft's own VC arm M12 participated — suggesting this isn't framed as competitive to GitHub but as a layer above it.

Entire's stated architecture is three layers:
1. A **Git-compatible database** — unifying code, intent, constraints, and reasoning in a single version-controlled system
2. A **universal semantic reasoning layer** — enabling multi-agent coordination through a "context graph"
3. An **AI-native SDLC interface** — reinventing agent-to-human collaboration

First shipped product: **Checkpoints** (open source CLI). Captures AI agent context — reasoning, prompts, decisions — on every Git commit. Supports Claude Code and Gemini CLI currently.

### Why it matters

Entire is building exactly the layer described in `notes/git-scaling-externality.md` as the potential Black Swan for GitHub: a semantic, agent-aware version control substrate that doesn't require Git's line-based merge algorithm. The framing is "alongside GitHub/GitLab, not instead of" — but that's a seed-stage positioning, not necessarily a permanent one.

The core diagnosis Dohmke is making is identical to the team-scale inflection argument in `framework/team-scale-inflection.md`:
> "Our manual system of software production — from issues, to git repositories, to pull requests, to deployment — was never designed for the era of AI in the first place."

Checkpoints is specifically solving the **decision half-life problem**: agent reasoning disappears when a session ends. Checkpoints pins it to the commit, making it durable and traceable. That's a direct answer to the Command Intent survival gap we identified as GitHub's moat — and Entire is building it as a portable, model-agnostic open-source layer.

### Tensions and open questions

- M12 participation makes this more interesting: is Microsoft incubating a GitHub successor, or hedging against one?
- Dohmke has said he spoke with Satya Nadella before founding and received support. The relationship is warm.
- Positioned as "platform at a higher level of the technology stack" — but a semantic DB + reasoning layer + new UI is a very complete stack.
- Open source model (à la GitLab) means the network effect builds outside GitHub's walls.

### Things to watch

- Adoption rate of Checkpoints among Claude Code and Gemini CLI users
- Whether Checkpoints integration gets added to Copilot workflows or remains competitive
- When Entire ships the semantic reasoning layer (multi-agent coordination)
- Enterprise deals — if they start selling to GitHub's installed base directly, the framing shifts
- GitHub's response: does Copilot gain a "reasoning capture" feature that mirrors Checkpoints?

---

## Atomic — Lee Faus

**Website:** https://atomic.dev  
**Newsletter:** Be Atomic (atomicdotdev.substack.com)  
**First observed:** 2026-04-07  
**Signal level:** 🟡 Medium — directionally relevant, worth tracking

### What it is

Lee Faus was Global Field CTO at GitLab after a prior stint at GitHub. He has launched **Atomic** (atomic.dev), described as covering "the infrastructure, primitives, and ideas behind agent-native software development."

The newsletter "Be Atomic" is both the publication arm and the brand. Product scope is unclear as of April 2026 — currently manifesting primarily as content/thought leadership.

### Notable content

March 2026 post **"The $750k Developer"** engages directly with the AI productivity ROI gap:
- Goldman Sachs: AI investment contributed "basically zero" to U.S. GDP growth in 2025
- NBER survey of ~6,000 CEOs/CFOs: 80–90% report no measurable productivity impact despite active AI use
- MIT studied $30–40B in enterprise GenAI investment: 95% of organizations got no return
- The frame: if agent-native tooling can unlock *measurable* developer output, the $750k developer thesis (one developer + agents = a team) is the real value prop

This is a complementary lens to the team-scale inflection work — Faus is interrogating whether individual productivity gains are real *before* the question of what breaks at team scale becomes relevant.

### Why it matters

Faus has deep DevOps and SDLC credibility from both GitHub and GitLab sides, plus a field CTO background (enterprise sales-adjacent). His framing of "agent-native primitives" is likely to resonate with practitioners and influence buyers. If Atomic coheres into a product, it would sit in the same space as Entire's tooling — focused on making agent workflows observable, governable, and composable.

### Things to watch

- What Atomic actually ships as product (currently unclear — newsletter + brand)
- Whether "Be Atomic" framework coheres into a product thesis or stays as content
- Relationship to Entire — complementary, overlapping, or competitive?
- Any fundraising or YC/accelerator activity

---

## Related

- `framework/team-scale-inflection.md` — the team-size inflection analysis these companies are directly responding to
- `notes/git-scaling-externality.md` — the substrate risk thesis Entire is building into
