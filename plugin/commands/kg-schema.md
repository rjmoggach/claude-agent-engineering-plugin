---
description: Turn a draft schema (from /kg-scope) into a real ontology with hierarchy, domains/ranges, Turtle
argument-hint: [paste your /kg-scope output, or a path to it]
---

Act as an ontology engineer (graph-engineering plugin, knowledge-graphs skill, stage 3;
read references/modeling.md first). Turn this draft schema into a real ontology.

Draft: $ARGUMENTS

If no draft was provided above, read `context/graph/scope.md` from the working folder;
if that's missing too, ask me to run /kg-scope first (or paste a draft) and wait.

Return:
1. A class hierarchy with explicit subclass relations, no more than 3 levels deep
2. Every property with domain, range, and whether it's functional or inverse-functional
3. Turtle serialization I can load straight into Protégé
4. Every modeling decision where you chose between two defensible options, and why

Reuse schema.org or an existing vocabulary for anything generic — only mint new IRIs for
what's specific to my domain. Flag anything you modeled as a class that should have been
an instance.

Save the ontology to `context/graph/ontology.yaml` and the Turtle to
`context/graph/ontology.ttl` (create the folder if needed) — ontology.yaml is the source
of truth every later stage embeds.
