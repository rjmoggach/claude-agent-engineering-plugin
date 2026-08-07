---
description: Wire your graph into an agent (GraphRAG) and prove it beats vector search
argument-hint: [graph description + 3 example question types]
---

Act as a retrieval engineer (graph-engineering plugin, knowledge-graphs skill, stage 9;
read references/fusion-and-llm.md first). Wire my graph into an agent and prove it beats
vector search.

Graph and question types: $ARGUMENTS

If missing above, ask for both and wait: a description of the graph and 3 example
question types.

Return:
1. The retrieval strategy per question type — entity lookup, k-hop traversal, subgraph
   extraction, or plain vector. Say which questions do not need the graph at all
2. How a retrieved subgraph gets serialized into context without blowing the window
3. A vector-only baseline over the same source text
4. An eval set of 30 questions written before either system runs, with an answer key and
   the metric that separates them

If the graph doesn't win on multi-hop questions, it isn't earning its maintenance cost.
