# Graph Engineering Plugin

Design the structures agents work through — task graphs for orchestration, context loops
for loop/context management, knowledge graphs for memory.

- **Agent**: `graph-engineer` — designs and audits orchestration, loops, and memory
  architecture; returns drawn graphs, state-object specs, guardrails, ontology drafts.
- **Skills**: `task-graphs` (topology patterns), `context-loops` (state objects,
  isolation, convergence, checkpoints), `knowledge-graphs` (the 9-stage pipeline with
  references and teaching mode).
- **Commands**: `/graph-audit`, `/kg-tutor`, and the chained build steps `/kg-scope`,
  `/kg-schema`, `/kg-extract`, `/kg-relations`, `/kg-events`, `/kg-fuse`, `/kg-eval`,
  `/kg-rag`.

Install from the repo root marketplace:

```
/plugin marketplace add rjmoggach/claude-graph-engineering-plugin
/plugin install graph-engineering@claude-graph-engineering-plugin
```

See the repository README for full documentation and credits.
