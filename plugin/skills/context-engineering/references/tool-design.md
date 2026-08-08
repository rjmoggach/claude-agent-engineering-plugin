# Tool Design — Tools as Context Contracts

*(Read when creating or auditing an agent's tool surface. A tool definition is not
documentation for humans — it loads into the agent's context and steers behavior.
Every word helps or hurts selection accuracy.)*

## Subtraction first

Tool count is a cost before it is a capability: every definition consumes attention
and widens the selection distribution. The field's headline result is a production
team that raised agent success from 80% to 100% by deleting roughly 80% of the
agent's tools. Before adding a tool, ask which existing ones can go; prefer a few
high-leverage tools that genuinely expand capability over thin wrappers around
existing APIs. Selection is probabilistic — no compiler confirms the right tool
fired, so the authoring choices below decide whether it fires at all.

## The consolidation principle

If a human engineer cannot say definitively which tool applies in a situation, the
agent cannot either. Agents select tools by comparing descriptions; any overlap
introduces selection errors, and every description consumes attention budget.

- Prefer one comprehensive tool over several narrow ones: `schedule_event` that finds
  availability and books in one call beats `list_users` + `list_events` +
  `create_event` chained in the right order by the agent.
- **Don't over-consolidate**: keep tools separate when behaviors genuinely differ or
  a mega-tool's parameter surface becomes its own ambiguity.
- As the collection grows, namespace by domain (`db_*`, `web_*`) — hierarchical
  routing beats evaluating a flat list. With MCP, always reference tools by fully
  qualified name (`ServerName:tool_name`) so multi-server setups resolve correctly.

## Description engineering

Structure every description to answer four questions:

1. **What does it do** — exactly; never "helps with".
2. **When to use it** — direct triggers and indirect signals.
3. **What it accepts** — each parameter with type, constraints, defaults, and a
   format example. Defaults should make the common case zero-thought.
4. **What it returns** — output shape, a success example, and the error conditions.

Agents cannot ask clarifying questions before a call — the description must carry the
whole contract.

## Response formats and errors

- Offer concise vs detailed response options and document when to use each; response
  size is context spend.
- Return semantic, human-readable fields over raw internal IDs — the agent reasons
  over what it can read.
- Build in pagination, range selection, filtering, and truncation with sensible
  defaults. When truncating, say so in the response and steer the agent toward many
  small targeted calls; silent cuts read as complete data.
- **Errors must be actionable for the agent**: what went wrong + how to correct —
  retry guidance for transients, a corrected format example for input errors, the
  specific missing fields for incomplete requests. "Failed" is zero recovery signal.

## Schema consistency

Verb-noun names (`get_customer`, `create_order`); the same parameter name for the
same concept everywhere (always `customer_id`, never a mix of `id`/`identifier`);
consistent return field names. Consistency is cross-tool generalization.

## Architectural reduction

The consolidation principle at its extreme: replace most specialized tools with
primitives — filesystem access, shell, standard utilities — plus good documentation
in files. Models understand these abstractions deeply, can chain them flexibly, and
the approach *improves* as models improve, where specialized "guardrail" tools lock
in current limitations.

Choose reduction when the data layer is well-documented and consistent, the model is
capable, and scaffold maintenance outweighs its value. Avoid it when data is messy,
domain knowledge is missing, or safety genuinely requires constrained actions.

## Improving tools from evidence

Feed observed failures back: collect the calls that went wrong, have an agent
diagnose the description gaps that caused them, patch the descriptions, and re-test
on the same tasks. Treat reported gains as workload-specific until reproduced on your
own catalog.

## Tool-surface audit checklist

1. Any two tools a human would hesitate between? → consolidate or sharpen boundaries.
2. Any description that doesn't answer all four questions? → rewrite.
3. Any error path returning "failed" without recovery guidance? → redesign.
4. Inconsistent parameter/return naming across tools? → normalize.
5. Tool count growing without namespacing? → group by domain.
6. Specialized tools an agent with filesystem + shell + docs wouldn't need? →
   candidates for reduction.
