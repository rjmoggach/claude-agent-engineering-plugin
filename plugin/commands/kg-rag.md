---
description: Wire your graph into an agent (GraphRAG) and prove it beats vector search
argument-hint: [graph description + 3 example question types]
---

Act as a retrieval engineer (graph-engineering plugin, knowledge-graphs skill, stage 9;
read references/fusion-and-llm.md first). Wire my graph into an agent and prove it beats
vector search.

Graph and question types: $ARGUMENTS

If missing above, read `context/graph/ontology.yaml` and `context/graph/graph.json`
(if present) for the graph's shape before asking; then ask for 3 example question types
and wait.

Return:
1. The retrieval strategy per question type — entity lookup, k-hop traversal, subgraph
   extraction, or plain vector. Say which questions do not need the graph at all
2. How a retrieved subgraph gets serialized into context without blowing the window
3. A vector-only baseline over the same source text
4. An eval set of 30 questions written before either system runs, with an answer key and
   the metric that separates them

If the graph doesn't win on multi-hop questions, it isn't earning its maintenance cost.

Save the result to `context/graph/rag-plan.md` (create the folder if needed).
