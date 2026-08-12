# Authoring Patterns

Read this after deciding document structure and owner placement, when choosing rule shape,
wording, information format, or guideline/template structure.

## Contents

- [Wording](#wording)
- [Rule Shape](#rule-shape)
- [Information Format](#information-format)
- [Guideline Structure Models](#guideline-structure-models)
- [Comparative Guideline Review](#comparative-guideline-review)
- [Guideline Improvement](#guideline-improvement)

## Wording

- Remove elaboration that blurs the core of the problem, and pick a more precise, broader concept
  word.
- If removing an element would let a reasonable agent choose a different target, judgment, action,
  or change scope, it isn't sufficient. If removing it leaves the judgment unchanged, treat it as a
  duplication candidate.
- Don't repeat as commentary what a rule and its example already say. If an example substitutes for
  a rule or creates a new exception, condense it or promote it into a body rule.
- Don't add an exception, field, or safety step based on abstract possibility alone. Ground it in
  an actual case and a real chance of recurrence.
- Avoid overly specific cases and abstractions that don't help execution judgment; generalize only
  as much as the current problem requires.

## Rule Shape

These forms are optional tools, not mandatory for every improvement. When a new rule is needed,
pick the shape from the categories below that fits the problem. Add a shape not listed here only
when it passes an actual-recurrence-plus-portable-grounds test (recurrence + cannot be restored by
an existing axis/shape).

**Scope and identity**

- `scope rule` — use when the applicable scope is unstable.
- `identity rule` — use when a document's or structure's identity is unstable.

**Conditional branching, ordering, and re-entry**

- `decision table` — use when conditions and outcomes split into multiple branches.

`gate-driven flow` and `feedback loop` are instances that judge the components the representation
axis proposes when it suggests a flow/loop form for a procedure, workflow, or review flow. Both
shapes are restored from whichever of the following elements each needs.

- `entry` — the entry condition.
- `state` — what's carried and cycled during repetition (a change candidate, an unresolved item,
  etc.).
- `operation` — what happens in one pass.
- `back-edge` — which outcome sends control back to which earlier step.
- `termination` — which goal, once met, exits the loop.
- `escalation` — where to hand off when the loop can't resolve itself.

Each shape is defined by whichever of these elements it needs.

- `gate-driven flow` — use when the same subject is judged through an ordered set of gates and each
  outcome changes the next action or exit path. State the entry, the gate order (operation), the
  branch outcomes, and the termination. Write each branch outcome as an instruction naming the
  required action, parallel in form across branches. If another section owns the action, name that
  owner section — don't leave the reader to infer the action from a state or label alone. This is a
  one-directional flow with no path back.
- `feedback loop` — use when an execution or verification result can re-trigger an earlier step's
  judgment. Add a back-edge and repeating state to the elements of gate-driven flow, and state the
  re-entry condition and the step to return to. It isn't a well-formed loop unless it also restores
  a path back and a termination. Each iteration must move the state closer to termination; if the
  state repeats without progress, hand off to escalation instead of taking the back-edge again.

**Constraints and safeguards**

- `invariant` or `negative rule` — use when a misinterpretation to forbid keeps recurring. If a
  negative instruction alone can't pin down what it applies to, promote it to a classification or
  decision table.
- `must rule` — use when the condition is clear but execution keeps getting skipped.
- `destructive-change gate` — use to state the knowledge-preservation, reference-impact, and
  non-destructive-fallback requirements for deletion, replacement, archiving, or a status
  transition.

## Information Format

Choose the form based on what kind of information it is. When multiple forms can restore the same
judgment, pick the one with the lowest reader cost among those that pass the restoration test.
Brevity is a tiebreaker after restoration, not the primary criterion. The goal isn't maximum
compression — it's the density that doesn't lose the required judgment.

**Table** — use when an item has two or more independent properties and both rows and columns carry
meaning, or when a condition-based outcome lookup is needed. For a single column, a simple pair of
two properties, long prose cells, a table with many empty cells, or a table in the middle of a
procedure, first check whether a bullet list, definition style, prose, or a separate section can
restore the same judgment at lower reader cost.

**Bullet list** — use for independent items where order doesn't affect the outcome. Include an
intro sentence that states the list's meaning, and don't use it for a single item. Past 7-8 items,
consider categorizing or another form.

**Numbered list** — use for a procedure, steps, or priorities where order affects the outcome or
meaning. Don't use it for a single item. If a procedure needs more than one-directional order —
re-entry, branching, or a termination judgment — don't force it into a numbered list; look at
`gate-driven flow` or `feedback loop` under `Rule Shape` instead.

**Prose paragraph** — use when a narrative flow is needed, as with rationale, context, why, or
conceptual relationships. Don't over-fragment relationships or the fact/inference/opinion
distinction into a list or table.

**Definition style** (`term — explanation`) — use when a term and its definition pair up, as with
vocabulary, smells, options, or parameters. If a single term carries three or more independent
properties, consider a table.

**Code block / fenced example** — use to separate a command, output, template, skeleton, or literal
example from the body rules. Separate the command from the output, and explain the meaning of any
placeholder and how to substitute it.

**Callout or highlighted note** — use only for information outside the main flow whose omission
would cause misunderstanding, loss, or risk. If it fits naturally into the body, use prose instead;
don't use it for consecutive callouts or to emphasize an ordinary rule.

After choosing a form, check whether bullet overuse, over-fragmented prose, excessive callouts,
prose compressed into a table, commands mixed with output, or a single numbered item is blocking
restoration of the judgment.

## Guideline Structure Models

If the responsible owner guideline has already settled on a structure model, don't override it with
the options below.

- `skeleton-primary` — use when a familiar form and preventing missing required sections matter.
  State the applicable subtype and the conditions for omitting an empty section together.
- `invariant-primary` — use when document forms vary but only the owner boundary or a semantic
  constraint is common across them.
- `decision-table` — use when the document family has clear subtypes or a condition-based
  structure.
- `composition blocks` — use when multiple document types share some sections but must not inherit
  the whole template.
- `golden document` — use when a concrete exemplar is clearer than an abstract template. State what
  it demonstrates, and don't let incidental detail read as a rule.

Run every default section through the delete/absorb/optional/subtype-only/invariant-only
candidates first. Weigh the omission risk without the skeleton against the pressure to fill
unnecessary sections with it.

## Comparative Guideline Review

- Symmetry is a baseline, not a requirement. For parallel roles, keep order and role parallel, but
  explain any needed difference through document role or a content property.
- Don't copy a section's meaning just because the heading matches.
- If a document family has sub-kinds, state explicitly which one the skeleton targets.
- A skeleton example must follow the link form, indentation, evidence shape, and fencing rules it's
  teaching.
- Before judging a claim as stale or conflicting, first classify the statement's document role —
  normative, descriptive, or provenance.

## Guideline Improvement

- Check whether recurring owner, placement, or skeleton confusion is a candidate for a guideline
  improvement rather than a problem in one document. If it's a one-off misapplication, a local fix
  is enough.
- For a new document family, first propose the owner, reader, required evidence, lifecycle, and
  sibling relationships.
- Don't redefine the same rule across multiple documents — check whether relocation, condensing, or
  clarifying the boundary is smaller than a new rule.
- When structural-change candidates diverge, compare the next author's read path and misjudgment
  risk across 1-3 authoring scenarios.
- A router or summary document should not repeat detailed procedures — write it so the starting
  point and what to check are restorable.
- After a change, confirm it covers the problem case, doesn't leave excluded cases open, and doesn't
  conflict with owner boundaries.
