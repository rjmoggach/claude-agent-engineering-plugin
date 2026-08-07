# Agent Engineering Plugin

Design the structures agents work through: task graphs for orchestration, loops at
two timescales, context engineering for the attention budget, knowledge graphs for
memory.

- **Agent**: `agent-engineer` — designs and audits orchestration, loops, context, and
  memory architecture; returns drawn graphs, state-object specs, guardrails,
  operational-loop designs, context budgets, ontology drafts.
- **Skills**: `task-graphs` (topology patterns), `context-loops` (state objects,
  isolation, convergence, checkpoints — the inner loop), `loop-engineering` (recurring
  operational loops — autonomy ladder, budgets, safety, the seven patterns),
  `context-engineering` (attention budget, degradation patterns, optimization, tool
  design), `knowledge-graphs` (the 9-stage pipeline with references and teaching mode).
- **Commands**: `/graph-audit`, `/loop-design`, `/loop-audit`, `/context-audit`,
  `/task-brief`, `/kg-tutor`, and the chained build steps `/kg-scope`, `/kg-schema`,
  `/kg-extract`, `/kg-relations`, `/kg-events`, `/kg-fuse`, `/kg-eval`, `/kg-rag`.

Durable graph work lives in the user's project under `context/graph/` (created lazily;
`runs/` holds transient checkpoints and is gitignorable). The plugin is a design-time
optimizer for other plugins and pipelines — never a runtime dependency of them.

Install from the repo root marketplace:

```
/plugin marketplace add rjmoggach/claude-agent-engineering-plugin
/plugin install agent-engineering@claude-agent-engineering-plugin
```

See the repository README for full documentation and credits.
