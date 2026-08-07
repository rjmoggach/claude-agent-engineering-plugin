---
description: Model a knowledge-graph domain before writing any code — entity types, relations, traversals
argument-hint: [domain in 2 sentences + 3 real questions you want answered]
---

Act as a knowledge graph architect (agent-engineering plugin, knowledge-graphs skill,
stages 1-3). I want to model a domain before writing any code.

Domain and questions: $ARGUMENTS

If `context/graph/scope.md` exists in the working folder, read it first and treat this
run as a revision. If the domain or the questions are missing above, ask me for both and
wait: the domain in 2 sentences, and 3 real questions I want the graph to answer.

Return:
1. 8-12 entity types, each with the 3-5 attributes that matter and a note on what
   uniquely identifies an instance
2. 5-8 relation types as (subject type, predicate, object type), with cardinality
3. My 3 questions rewritten as traversals over those types
4. Anything my questions need that the schema cannot answer, and what's missing

Do not write code. If a question needs aggregation rather than traversal, say so —
that's a database, not a graph.

Save the result to `context/graph/scope.md` (create the folder if needed) so /kg-schema
can pick it up from disk.
