---
name: execution-flow-map
description: >-
  Trace an operation, function, or page interaction from its entry point into an ASCII
  Execution Flow Map of actual code, showing execution order, branches, exits, state
  mutations, and side effects. Use for requests to show execution flow, control flow,
  an execution tree, or page flows. Include return/throw/reject, catch, retry,
  finally/cleanup, and final termination. Do not use for bug detection, code quality
  assessment, refactoring, or solution proposals.
---

# Execution Flow Map

## Purpose and boundaries

Trace actual code into a human-readable ASCII Execution Flow Map. Show execution order,
branches, exits, and when and how state changes in the same tree, beyond a function call tree.
Apply the same principles to functions, CLI commands, API requests, daemon operations,
workers/jobs, event handlers, page interactions, and lifecycles.

This skill's responsibility is structural mapping only. Do not:

- Diagnose bugs or design flaws, or assess code quality.
- Propose deletion, refactoring, or solutions.
- Label wrappers, callbacks, or repeated validation as code smells.
- Claim to have reproduced runtime behavior without execution-based verification.

If asked “Is this a bug?” or “Fix it” afterward, briefly distinguish that request from this
skill's structural mapping and handle it separately. This boundary defines the skill's
responsibility; it does not override higher-priority instructions.

## Procedure

### 1. Establish the entry point

Find its actual definition. If names collide or the target is unclear, present candidates
and ask which one. Trace immediately when the conversation and code identify a clear target.

For a page/screen, separate individual interactions/lifecycles such as mount, click, change,
submit, and unmount. Inspect the code first and briefly list those found. If the user has
not specified scope, ask which interactions to trace, or whether to trace all. Do not select
only the “main” interactions yourself. Do not connect independent interactions as if they
were one sequential tree.

### 2. Build the high-level flow

Read the entry point body and arrange its calls in order as top-level nodes. Start shallow;
do not apply one uniform depth to the entire call tree. Expand only the necessary nodes.

### 3. Shallow-inspect each node

Open actual definitions of called functions. Check major call order, conditions, return/early
return, throw/reject, catch, await, retry/loops, cleanup/finally, side effects, and state
mutations. Mutations include local state setters, global store setters, cache invalidation
and manipulation, ref writes, and changes to DB, memory, and runtime state. Note state that
persists over time as well.

Inspection is internal investigation, distinct from expansion. Reading implementation does
not imply displaying all of it.

### 4. Expand recursively, per node

Collapse to the node name when that name sufficiently communicates the flow and exits.
Expand that node if its name hides any of:

- Important execution order.
- Distinct exit paths and final termination points.
- Side effects already performed before failure.
- Meaningful state mutations. Even with one exit, expand a name such as `handleSuccess()`
  when it hides changes to multiple stores, caches, or local states.

Apply inspect → expand/collapse recursively to each child. Stop when further expansion does
not change execution order, exit paths, or side-effect boundaries.

For follow-ups such as “Expand this” or “Show more,” inspect the requested node again as a
new entry point and apply the same rules. Output only that expanded portion, without
reprinting other nodes or the whole tree. Put its path from the original tree at the top:

```text
[Expanded: restartDaemon > startDaemon]
```

Maintain the path across successive expansions. Reuse step 1's candidate clarification when
several nodes share the name or the referent of “here” is unclear.

### 5. Render the execution tree

Follow the notation and Special Flow rules below. Terminal leaves identify actual exits;
non-terminal leaves identify the actual next step.

### 6. Explore overlapping flows

Report a separate **Overlap Candidate** only when the code establishes all three:

1. A shared mutable lifetime that persists over time.
2. An independent entry point accessing that state.
3. A write or potentially competing access whose outcome can depend on execution order.

Promises, DB records, resource/process/runtime state, membership, and global stores all
qualify as possible state. Mere read/subscribe access, or callback/wrapper/delayed-call syntax,
is insufficient. Accessibility does not prove actual concurrency or a race. Without runtime
confirmation, report only a Candidate and mark actual concurrent execution `[Needs verification]`.
Do not diagnose it as a bug or flaw.

If one interaction explicitly and deterministically triggers another flow through a state
change in the code, that is **forward propagation**. For example: successful deletion → cache
invalidation → refetch. Draw the triggered flow directly beneath the mutation node; do not
label it Overlap. Apply candidate criteria to independent triggers, such as polling timers
and clicks that do not call each other, accessing the same mutable lifetime without a
code-guaranteed order.

During default forward tracing, do not search the whole codebase for other global-store
references. Investigate only access points encountered while tracing, and state that scope.
Reverse searches for other consumers/writers require the explicit request in step 7. Reuse
the exact state names from the tree in candidate reports.

### 7. Reverse Consumer Lookup — optional, disabled by default

Perform only when explicitly asked to search backward, for example “Show how far this state
change affects things,” “Find who reads/writes this,” or “Find other independent flows
accessing the same state.” Actually search the accessible repository/requested scope and
attach discovered consumers beneath the mutation node.

- Distinguish read/subscribe from write. Being a writer alone does not establish an Overlap
  Candidate. Evaluate all three criteria in step 6 separately; add `[See Overlap Candidate]`
  only when they hold.
- Distinguish subscription from “change → subsequent action.” Write labels such as
  `change → rerender [Confirmed in code]` only when project code such as selectors or
  dependencies establishes it. Otherwise report subscription only.
- Include file paths and roles such as Component/Page/Modal/API/Worker/Scheduler/Service only
  when actually established. This is not limited to frontend consumers.
- Always state the search scope and `Only consumers found within the current search scope —
  not exhaustive`. Dynamic references, other repositories, and lazy imports may be missed.

Consumer lookup investigates propagation targets; Overlap Candidates identify independent,
potentially competing access. Keep them distinct.

### 8. Mark uncertainty

Do not invent paths that the code does not establish.

- `[Needs verification]`: Requires more code inspection or execution to establish.
- `[Possible in code]`: Structurally reachable, but not verified through execution.
- `[Confirmed in code]`: Evidence found in project code; not a claim of runtime reproduction.
- `[Overlap Candidate]`: Meets all three criteria in step 6.
- `[external]`: Implementation not owned by this project.
- `[IPC]` / `[process]` / `[module]`: Boundary crossed by a call.

## Output principles and notation

Prioritize the tree and avoid long introductory or trailing explanations. Complete each
operation separately when several are requested. Omit a default caption. If there are at
least five major top-level steps or at least two expanded nodes, precede the detailed tree
with a one-line `[Overview]` showing only the normal progression. The overview never replaces
the detailed tree. Follow-up expansion shows only the requested subtree, retaining
`[Expanded: A > B]`.

```text
Top → bottom   Execution order
├─ / └─        Siblings: sequential steps or explicitly labeled alternatives
Indentation    Parent action and its internals/results
→              Next flow / state mutation / final destination
↻              Loop / retry
```

- Terminal leaves end with `SUCCESS`, `ERROR`, or a concrete return/reject value. Non-terminal
  leaves name the actual next step. Collapsed calls retain only their names.
- Branch/result leaves specify **target + result + next step**. Do not use standalone labels
  or placeholders such as `success`, `failure`, `continue`, or `and then`. Each label should be
  understandable without searching upward for its parent.
- When branches converge, repeat the same next-step name on each path. When the same kind of
  result routes to retry handling from several locations, repeat that destination consistently
  at every occurrence.
- Always draw conditions and value decisions as explicit sibling alternatives beneath one
  decision node, using `true`/`false` or actual conditions/values. Do not fold conditions into
  prose nodes such as “open if closed.”
- When an operation terminates within a branch, close its exit inside that branch. Do not
  place a common `SUCCESS` outside already-terminated branches, implying that those paths
  might also continue there.

## Special Flow

- Show mutations in the branch and at the point where they happen, not in a separate summary
  table/index. Name the target and result, and connect subsequent steps when present. For
  example: `store → selection = null`, `cache ['records'] → invalidate → refetchRecords`.
- State “no state change” only when the difference between sibling paths, such as success
  and failure, is central. Do not mechanically repeat it for all remaining branches.
- Use `↻` for retries/loops. Identify attempt limits/termination conditions, non-retryable
  failures, and exhaustion.
- Stop tracing internals at `[external]` for third-party/standard libraries and shared
  libraries outside this repository. Continue tracing how project code handles their returns
  and exceptions.
- Framework scheduler/reconciliation/effect scheduling internals are also `[external]`.
  Continue into project code triggered by registered effects/callbacks, invalidation, or
  state changes. Do not assert unverified subsequent execution.
- Mark IPC/process/module boundaries and keep tracing when the repository contains the
  implementation. Stop with `[Needs verification]` only when it cannot be established.
- Represent callback/event handler/wrapper/delayed invocation registration and execution as
  deferred control flow. Do not imply execution occurs immediately at registration. Syntax
  alone does not establish an Overlap Candidate.

Read [references/examples.md](references/examples.md) when rendering examples are needed for
complex branches, retry, mutations, expansion, consumers, IPC, or overlap.
