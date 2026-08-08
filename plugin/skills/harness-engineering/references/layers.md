# The Harness Layers — Audit Surface, Taxonomy, Evidence

*(Read when auditing a harness end to end, or when making the case that the harness
is the constraint. The six-layer model is the practical audit surface; ETCLOVG is the
deeper reference taxonomy from the field's survey.)*

## The six-layer audit model

Walk every layer; score present / partial / absent. Most production agents are
missing at least two — find the missing ones before tuning any single layer.

1. **Context management** — what the model sees each turn: window composition,
   placement, degradation risk, compression triggers. Audit with `/context-audit`
   (context-engineering skill).
2. **Tool system** — what the agent can do and how reliably it selects: tool count
   vs. leverage, naming and namespacing, description quality, return design, error
   design. Checklist in the context-engineering skill's tool-design reference.
3. **Execution orchestration** — how work is structured: topology (task-graphs),
   iteration within a run (context-loops), recurring schedules (loop-engineering).
   Audit with `/graph-audit` and `/loop-audit`.
4. **State and memory** — how state survives: handoff shape between contexts, reset
   semantics, checkpoint/resume, durable memory files, persistent knowledge
   (knowledge-graphs skill for the durable end).
5. **Evaluation and observation** — how the system knows it worked: traces kept,
   generator/critic separation, artifact-based judgment, evals built from real
   failures.
6. **Constraints and recovery** — how the system fails safely: boundary validators
   and hooks, denylists, backpressure, loop detection, attempt caps, kill switches.

## ETCLOVG — the seven-layer reference taxonomy

From the Agent Harness Engineering survey, which defines harness engineering as
coordinating "the closed-loop system around both" prompts and context, integrating
"execution environments, tool interfaces, persistent state, lifecycle control,
observability, verification, and governance." Per-layer scope, as the survey states
it:

| Layer | Scope |
|---|---|
| **E**xecution | "Determines where agent code runs and what sandbox constraints bound it" |
| **T**ooling | "Specifies how external capabilities are described, discovered, and invoked" |
| **C**ontext | "Controls what the model can see across short-term, session-level, and persistent horizons" |
| **L**ifecycle | "Organizes the control flow that reads and writes state" |
| **O**bservability | "Captures traces, costs, failures, and reliability signals" |
| **V**erification | "Turns tasks and traces into evaluation, failure attribution, and regression feedback" |
| **G**overnance | "Constrains behavior across model-level, system-level, and organizational-level sub-layers" |

ETCLOVG splits observability and governance out of the six-layer model because in
production each has its own tooling stack and a different owner. The survey's corpus
finding (as of its 2026 catalog): Execution, Tooling, Lifecycle, and Verification
have the densest open-source coverage (20, 12, 47, and 21 primary projects
respectively); Observability and Governance are thinner in open source and more
often live inside commercial platforms and SDK features — operational control
matured later than runtime and benchmark infrastructure.

Mapping between the two: six-layer "execution orchestration" ≈ E + L; "evaluation
and observation" ≈ O + V; "constraints and recovery" spans L (control flow) and G
(limits); the rest map one to one.

## The harness-only evidence

Results where the model was held fixed and only the harness changed. All figures are
as reported by their sources, collected 2026-08; leaderboard-dependent numbers drift —
re-check against the live leaderboard before repeating them, and never round or
extrapolate.

| Result | What changed | Source |
|---|---|---|
| Terminal-Bench 2.0: 52.8% → 66.5% (+13.7 pts) | Harness only | LangChain (via explainx.ai write-up) |
| Agent success rate 80% → 100% | Deleted ~80% of the agent's tools | Vercel (via MongoDB write-up) |
| Top 30 → Top 5 on Terminal-Bench 2.0 | Harness only | Viv Trivedy / HumanLayer (via Addy Osmani) |
| 76.4% on Terminal-Bench-2 | Automated harness optimization, no weight changes | Agent Harness Engineering survey |
| Legal-agent accuracy more than doubled | Harness optimization alone | Harvey (via MongoDB write-up) |
| SWE-bench Verified error rates materially reduced | Precise tool-description refinements | Anthropic, "Writing effective tools for agents" |

## The mechanism — post-training coupling

Models are post-trained inside a specific harness: particular tools, prompt shapes,
and feedback conventions. Deployed into a differently fitted harness — tools matched
to the actual codebase, a tighter prompt, sharper backpressure — the same weights can
express capability the original harness never elicited. This is why the same
frontier model scores substantially lower inside a general-purpose agent product
than inside a task-fitted custom harness on the same benchmark, and why the
reliability ceiling you observe is a property of the model–harness pair, never of
the model alone. It is also why harness work compounds: every layer you fit better
recovers headroom the model already has.

## Sources

Primary:

- Anthropic, *Writing effective tools for agents, with agents* —
  https://www.anthropic.com/engineering/writing-tools-for-agents
- Anthropic, *Effective context engineering for AI agents* —
  https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- *Agent Harness Engineering: A Survey* (ETCLOVG) —
  https://openreview.net/pdf?id=eONq7FdiHa — catalog:
  https://picrew.github.io/LLM-Harness/

Practitioner write-ups (verify figures against primary sources before repeating):

- Addy Osmani, *Agent Harness Engineering* —
  https://addyosmani.com/blog/agent-harness-engineering/
- MongoDB, *The Agent Harness* — Vercel and Harvey results.
- explainx.ai — the LangChain Terminal-Bench 2.0 result.
