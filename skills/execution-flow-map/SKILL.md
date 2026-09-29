---
name: execution-flow-map
description: >-
  Trace a specific operation, function, or page/screen interaction from its entry point
  through actual code, visualizing execution order, branches, exits, and state changes
  (setState, store setters, cache invalidation) as a readable ASCII Execution Flow Map.
  Show execution flow and state changes together, beyond a function call tree, including
  return/early return, throw/reject, catch, retry, cleanup/finally, and each path's final
  exit. For pages, trace individual mount/click/change/unmount interactions and
  background/periodic flows such as timers, polling, and WebSocket heartbeats.
  Do not use for bug detection, code quality assessment, refactoring, or solution proposals.
  Use for "Show this function/operation's execution flow", "Draw the control flow",
  "Draw an execution tree", "Show this page's flow", or "Trace how this runs".
---

# Execution Flow Map

## Purpose and prohibitions

This is a purely structural skill. Its sole responsibility is to trace the actual execution flow of the operation/function/interaction identified by the user and display execution order, branches, exits, and state changes together in an ASCII Execution Flow Map. Beyond a function call tree, the same tree shows "which state changes when during this flow".

Do not do the following (takes precedence over all other instructions):

- Diagnose bugs or design flaws
- Propose code deletion, refactoring, or solutions
- Label wrappers/callbacks/repeated validation as code smells
- Claim runtime behavior was reproduced without actual execution-based verification
- Omit tree syntax by replacing the flow with arrow-linked natural-language prose merely because the request was written in natural language

Even if the user asks "Is this a bug?" or "Fix it" during/after the flow report, that is outside this skill's responsibility. Show the structure, then briefly explain that the request needs separate handling.

## Procedure

1. **Establish the entry point.** Find the actual definition of the operation/function identified by the user. If several definitions share a name or the target is unclear, ask rather than guess.

   **For a page/screen flow request**, entry points are the page's individual interactions/lifecycles/background flows, not a single function. First inspect the code, then briefly list discovered entry points in these three categories. If the user has not specified scope, ask which to trace. Do not arbitrarily select "main" flows and omit the rest:
   - **User Interaction**: triggered by user actions such as click, submit, or change
   - **Lifecycle**: triggered by component/page lifetime events such as mount or unmount
   - **Background / Periodic**: repeated `setInterval`/`setTimeout`, WebSocket ping/pong/heartbeat, subscription callbacks, watchers, queue consumers, and other autonomous repeating/background execution

   Example: `On this page I found [User Interaction] delete click / filter change, [Lifecycle] mount / unmount, [Background] status polling every 3 seconds and WebSocket heartbeat. Which should we inspect, or all of them?`

   Background/Periodic entry points are **discovered directly while inspecting the same file/component**, so unlike step 8 (Reverse Consumer Lookup), they require no separate global search; they belong to ordinary entry-point discovery.

2. **Identify the Execution Mechanism.** Before tracing actual code order, establish "Which execution/communication/synchronization mechanism makes this feature work?" This differs from Execution Flow (steps 3–6), which answers "In what actual order and branches does code execute inside it?"

   **When to show it.** Show it if either applies:
   - The requested scope covers several entry points, such as a page/feature/subsystem. Display by default when there are multiple entry points rather than one function.
   - Even with a single entry point, execution relies on **an external trigger rather than an ordinary direct function call** (message/event arrival, timer, delegation to another process, etc.). Even one mechanism conveys important execution information not obvious from its name (e.g. a handler registered through `queue.on('message', ...)`, or a CLI delegating through IPC rather than finishing locally).

   Conversely, omit it for **a request directly calling one function whose mechanism is ordinary top-down execution** (e.g. a simple parser of local values), because it adds nothing beyond repeating code order.

   When several mechanisms form one causal pipeline (e.g. local preparation → IPC delegation), use a short arrow chain if useful. **Keep only boundaries where the mechanism itself changes**. Omit preparation within the same mechanism (e.g. argv parsing, config reading, validation under local direct calls); retain actual transitions, such as local execution → IPC delegation to another process. Preparation and internal processing order belong in Execution Flow. For a CLI restart:

   ```
   [Execution Mechanism]
   daemon restart → CLI command → IPC → execution delegated to daemon process
   ```

   Do not list `parse argv → prepare local config → validation`, which all share "local direct call" and add no mechanism information. "Where execution actually occurs" differs from the Overview's "which task stages it follows", so their content differs even when both use arrow chains. If the chain descends into detailed calls/branches/state changes, it belongs in Execution Flow.

   Format each item as **"feature/action → mechanism"**. The left side names an action with a code-observable purpose, not just an object/data/component (e.g. `render terminal list`, not `terminal list`; `check Worker status`, not `Worker`). Base the label on actual observable purpose — who/what consumes the data or what the call does — without inferring business intent absent from code. On the right, combine mechanisms as needed (`+`, "initially: A / afterward: B", or a causal chain), using only code-established evidence such as `useQuery`, `new WebSocket(...)`, `EventSource(...)`, `setInterval`, or message queue consumer registration. Do not descend into detailed calls/branches/state changes; those belong in Execution Flow Map and Expansion. Example:

   ```
   [Execution Mechanism]
   Render terminal list             → TanStack Query + HTTP polling (3-second refetch)
   Stream terminal screen           → WebSocket
   Check connection status          → WebSocket ping/pong (5-second interval)
   Handle terminal creation/closure → HTTP mutation
   ```

   Single-entry-point examples:

   ```
   [Execution Mechanism]
   Handle job results → Message Queue subscription (handler runs on message arrival)
   ```

   ```
   [Execution Mechanism]
   daemon restart → CLI command → IPC → execution delegated to daemon process
   ```

3. **Build the high-level flow.** Read the entry-point body and list its calls in order as top-level nodes. Start shallow instead of expanding the entire call tree to a uniform depth.

4. **Shallow-inspect each node.** Open the actual definitions of called functions and check at least: major call order, conditional branches, return/early return, throw/reject, catch, await, retry/loops, cleanup/finally, side effects, waits for external responses/events, and **state changes (setState, store setters, query invalidation/cache manipulation, ref writes, etc.)**. This is internal investigation for step 5's expansion decision, not information to print every time merely because it was checked. Note state persisting over time (step 7), but decide separately under step 7 whether to represent it.

5. **Expansion — apply recursively.**

   ```
   inspect node
     ↓
   expansion needed?
     ├─ no  → collapse (name only)
     └─ yes → show children
                ↓
           apply same rules recursively to each child
   ```

   Keep a node collapsed if its name accurately expresses its actual exit paths. Expand only that node if its parent name hides any of:
   - Important execution order
   - Distinct exit paths (e.g. different failure sites lead to different exits)
   - Side effects already performed before failure
   - Meaningful state mutations hidden by the parent name (e.g. `handleSuccess()` changes several stores/caches/local states internally; even one exit path justifies expansion because state-change tracing is central to this skill)
   - A meaningful `[WAITING]` → continuation resumption boundary hidden by the parent (e.g. `awaitApproval()` hides request creation → waiting → response-dependent resumption; applies even without an explicit waiting-state mutation)
   - A lifecycle boundary requiring distinction between child/sub-operation termination and the parent operation continuing (e.g. one request resolves while its parent operation continues)

   Apply the same decision recursively to expanded children. Stop when further expansion does not change execution order / exit paths / side-effect boundaries. Inspecting (reading) and expanding (outputting) differ; reading does not require displaying every internal detail.

   **For follow-up requests to expand a node** ("Show more", "Expand the internals", "Detail this part"), shallow-inspect it again as a new entry point and apply the Expansion rules recursively from that node. Do not re-expand other nodes or reprint the entire Execution Tree; show only the requested expanded portion. This makes the skill a tool for incremental exploration through the tree, rather than one-time tree generation.

   At the top, leave a one-line path from the original tree to the node, such as `[Expanded: restartDaemon > startDaemon]`, so successive drill-downs retain their location in the whole flow.

   If the requested node is ambiguous (same name at several locations, unclear "here"), **reuse step 1's entry-point clarification rule**: ask about candidates rather than arbitrarily expanding one.

6. **Render as an ASCII Execution Tree.** Follow "Notation" and "Special Flow" below. Identify where leaves ultimately terminate (SUCCESS/ERROR/concrete return or reject value).

7. **Explore Overlapping Flow.** Separately find flows that cannot fit one tree. If state persists over time and another entry point can access it during that lifetime in the code, mark an "Overlapping Flow Candidate" separately. State candidates include Promises, memory, DB records, runtime state, resource references, process state, membership, **global stores such as zustand/Redux/Context API**, and other shared state. **Syntax alone (callback, wrapper, deferred execution, etc.) does not qualify**; shared lifetime plus another entry point must be established in code. **Mere read/subscribe access by another entry point is insufficient.** All three criteria must hold: a shared mutable lifetime; an independent entry point that can access it; and access that **changes the mutable state (write) or can yield different results depending on execution order (potentially competing access)**. Accessibility alone proves neither concurrency nor a race; report only a Candidate unless confirmed.

   **Do not mark Overlap merely because state is shared.** If one interaction's mutation explicitly and deterministically triggers another flow in code (e.g. successful deletion calls `invalidate`, causing that query to refetch), this is a direct causal relationship rather than incidental temporal overlap. It is forward propagation, not Overlap; draw it directly beneath the mutation leaf (e.g. `['patients'] → invalidate → refetch`). Apply Overlap criteria only when independent entry points have separate triggers (timer, click, etc.), do not call each other in code, and access the same lifetime without a code-guaranteed execution order (e.g. polling and a user action sharing display state).

   **State the access-point search scope for global stores.** Query cache/component state access often emerges within the same page/component, but zustand/Redux/Context global stores can be touched from entirely different pages/components. **Default forward tracing (steps 1–7) does not search the entire codebase for global references**; report only access points encountered while tracing. Actually locating other consumers/writers belongs to reverse search only when step 8 (Reverse Consumer Lookup) is explicitly requested.

8. **Reverse Consumer Lookup (optional, disabled by default).** This is not part of the default procedure. Forward tracing (steps 1–7) follows "what the entry point calls" by opening definitions; consumer lookup reverses direction to find "who reads this state". Perform it **only when explicitly requested**, e.g. "Also show how far this state change affects things". Actually search backward within the accessible repository/search scope. Because completeness is not guaranteed (dynamic references, other repositories, lazy imports, etc. may be missed), always label results as "not guaranteed exhaustive".

   List discovered consumers beneath the mutation node from step 6, distinguishing:
   - **Read/subscribe only versus write access as well.** Mark writers, but do not automatically classify them as Overlap Candidates. Separately apply step 7's shared lifetime + independent entry point + access criteria and cross-reference `[See Overlap Candidate]` when they qualify.
   - **Subscription versus "change → subsequent action (rerender, etc.)".** Write `change → rerender [Confirmed in code]` only when actual code such as selectors/dependency arrays establishes re-execution. Otherwise report subscription alone; do not assume rerendering.
   - Include file paths and roles (Component/Page/Modal/API/Worker/Scheduler/Service, etc.) when verifiable, labeling only what actual project structure/code establishes. These are concrete examples of general "consumers affected by state changes", not a frontend-only model; schedulers/workers/services play the same role on the backend.
   - Always state "Only those found in the current search scope — not exhaustive".

   Consumer lookup and Overlap Candidate are distinct: the former traces ordinary data propagation; the latter flags independent writers whose execution may overlap. Do not conflate them.

9. **Mark uncertainty.** Do not fill in paths absent from verified code. Use as needed: `[Needs verification]` (more code inspection needed), `[Possible in code]` (structurally reachable but not executed), `[Overlap Candidate]` (meets step 7), `[external]` (implementation not owned by project), `[IPC]`/`[process]`/`[module]` (cross-boundary calls).

## Output principles

- Prioritize the tree itself over explanation. Do not surround it with long explanatory paragraphs.
- **Even for natural-language requests ("Explain by user actions", "How does this page work?"), express the flow in Execution Flow Map (ASCII tree) syntax.** Prose does not replace the tree; add at most one or two supporting sentences before/after it if needed. Do not replace the entire flow with arrow-linked natural-language sequencing such as "click new shell → disable create button → ...", which obscures branches/exits/state changes without tree syntax.
- Do not add a one-line caption by default. Tree syntax (order/branches/indentation/arrows/retry symbols) should be self-contained, assuming the user becomes familiar with it. Add a one-line High-level Overview before the detailed Execution Tree if either applies (thresholds may be adjusted after more fixtures):
  - At least five major top-level execution steps
  - At least two expanded nodes

  Example:

  ```
  [Overview]
  restartDaemon
  → validate config
  → stop existing daemon
  → start new daemon
  → record state
  → SUCCESS
  ```

  Overview does not replace the detailed Tree. It shows only normal progression, omitting detailed branches and exceptions, and always accompanies the detailed Tree.

- Distinguish established facts from inference using step 9's markings.
- When multiple operations are requested, complete each tree first and minimize shared explanation.
- For follow-up expansion, output only that node's expanded portion, not the entire previous tree. Retain a one-line path from the original tree as `[Expanded: A > B]`.

## Special Flow

- **Show state mutations in their actual branch and at the moment they occur**, not in a separate summary table/index. Place `setState`, store setters (zustand/redux, etc.), query invalidation/cache manipulation, ref writes, and other side effects at their actual leaf as "target + result + next step". Example:
  ```
  TanStack Query ['patients'] → invalidate → refetch patient list
  Zustand patientStore → selectedPatient = null
  Local State → deleteModalOpen = false
  ```
- **Say "no state change" only when differences between sibling branches matter to understanding.** Do not force it onto every branch. If only one of several siblings changes state, repeating it on all the others adds noise. Explicitly show it when the distinction itself matters, such as failure leaving cache unchanged versus success invalidating cache.
- **When a mutation target meets step 7's Overlapping Flow criteria, reuse its exact name in the Overlap Candidate.** State names (e.g. `patientStore`, `['patients']`) already appear in the tree, so cross-reference them without reintroducing them.
- **Use `↻` for retry/loops.** Always show its repetition condition: a count for **finite repetition** within one operation (e.g. `↻ attempt (maximum 3 attempts)`), or a termination condition for **unbounded/conditional repetition** independent of user actions, as in Background Flow below (e.g. `↻ polling (stop on disconnect)`).
- **Draw Background / Periodic Flow as separate entry points and separate trees.** Label autonomous repeated execution — timers, intervals, WebSocket ping/pong/heartbeat, subscription callbacks, watchers, queue consumers — `[Background Flow]`. Order it as start trigger (mount, connection open, etc.) → `↻` loop body → termination condition. **These flows need not end with SUCCESS**: an indefinite loop ending on unmount/disconnect is normal. Use the actual termination condition rather than forcing SUCCESS/ERROR. If it shares state with another flow, reuse the name in Overlap Candidate. Example:
  ```
  [Background Flow] terminal status polling
  Page active
  │
  └─ ↻ polling (stop on page unmount, 3-second interval)
     ├─ Fetch terminal/agent/notification lists
     ├─ Check response freshness
     │  ├─ stale → discard → no state change
     │  └─ current → update lists/execution state
     └─ Wait until next tick → ↻ polling
  ```
- **Waiting / Suspension: waiting is not an exit.** Mark a continuation paused for an external response/event/condition without ending its lifecycle as `[WAITING: what is awaited]` (e.g. user approval, IPC/worker response, lock acquisition, external process termination, connection establishment).
  - `[WAITING]` is not a terminal leaf, SUCCESS, ERROR, or operation termination. Continue tracing the continuation and result-dependent branches beneath it after the response/event arrives.
  - **Do not mark every `await` as `[WAITING]`.** Ordinary async calls such as `await fs.readFile()`, `await fetch()`, or `await db.query()` need only the existing notation. Use it **only when waiting itself matters to understanding operation state**, such as an explicit `waitingOnApproval` state or an external response/event/condition resuming the continuation.
  - Include the external response/event/condition that resumes execution (resume trigger) on the `[WAITING]` line. **A resume trigger is not necessarily an independent entry point**: an internal callback resolving a Promise from a worker response is merely a resume trigger. Show a separate entry point only if code establishes independence (e.g. a separate request handler for user responses), connecting it as `└─ [Event] approval response → resume continuation`. If it writes mutable state retained during the wait, separately evaluate step 7. The existence of `[WAITING]` alone does not establish an Overlap Candidate.
- **Lifecycle scope: identify whose lifecycle ended.** For ending states/events (`resolved`, `completed`, `closed`, etc.), include the target name. Use `SUCCESS`/`ERROR` for the final exit **of that operation's scope**. Expanded child operations can have their own SUCCESS/ERROR, but these terminate the child, not the parent. If a nested child exit could read as parent termination, qualify it (e.g. `Command → ERROR`). Likewise name the owner of lifecycle states such as resolved/completed/closed (e.g. `Approval Request → RESOLVED`, not `Turn → SUCCESS`). Continue drawing the parent if it runs on after the child lifecycle ends.
  Example:
  ```
  [Event] Request to execute a risky command
  │
  ├─ command approval required = false
  │  └─ Execute command → proceed with remaining turn execution
  │
  └─ command approval required = true
     ├─ activeFlags → ["waitingOnApproval"]
     ├─ requestApproval(id=101)
     └─ [WAITING: user approval response]
        ├─ Approved
        │  ├─ Approval Request(101) → RESOLVED
        │  ├─ activeFlags → []
        │  └─ Execute command → proceed with remaining turn execution
        │
        ├─ Denied
        │  ├─ Approval Request(101) → RESOLVED
        │  ├─ activeFlags → []
        │  └─ Do not execute command → proceed with remaining turn execution
        │
        └─ Turn cancelled
           ├─ Approval Request(101) → RESOLVED
           └─ Turn → interrupted (turn/completed) → terminated
  ```
  Here `Approval Request → RESOLVED` ends only one request's lifecycle; the Turn continues in both Approved and Denied branches. Mark Turn termination only where its own ending event is established, as in `Turn cancelled`.
- **Stop at the `[external]` boundary for external libraries.** Do not trace implementations the project does not own (standard libraries, third-party packages, or internal shared libraries outside this repository); trace only how current code handles their results/returns/exceptions. **Framework/runtime internals (e.g. React scheduler, reconciliation, effect rescheduling) are also `[external]`.** Continue tracing what project code actually executes through `useEffect` callbacks, query invalidation, and state updates. The boundary stops at framework internals, not at project code they trigger.
- **Mark IPC/process/module boundaries but continue tracing the operation.** A boundary alone does not stop tracing. Expand actual implementations available in this repository; stop with `[Needs verification]` only when unavailable.
- **Do not treat callbacks/event handlers/wrappers/deferred binding/delayed invocation as separate problem categories.** Registration and execution being separate in code is deferred control flow, shown sequentially. Promote to Overlap Candidate only when step 7 (shared lifetime + another entry point) is actually established.

## Notation

```
operationName
│
├─ stepA
│  └─ failure → ERROR
│
├─ stepB
│  ├─ failure → ERROR
│  └─ success → proceed to stepC
│
├─ stepC
│
├─ stepD
│  ├─ substepD1
│  ├─ substepD2
│  └─ failure → cleanup → ERROR
│
└─ SUCCESS
```

- `├─` / `└─`: sibling nodes. Sequential calls and conditional branches use the same tree shape; labels (`failure →`, `success`) distinguish them.
- `→`: state transition/final destination. Examples: `failure → ERROR`, `failure → cleanup → ERROR`.
- `↻`: retry/loop. Example: `↻ attempt (maximum 3 attempts)`.
- **Terminal leaves that end the operation** end with `SUCCESS`, `ERROR`, or a concrete return/reject value confirmed in code. **Other leaves that do not end the operation** (e.g. `proceed to conn.write`) name the actual next step. Not every leaf must end with SUCCESS/ERROR.
- Collapsed nodes retain only their names. Expanded nodes indent internal branches one level using the same syntax; recursively assess each child for expansion/collapse under step 5.
- **Success/failure labels must identify what succeeded/failed from that line alone.** Do not make readers search upward for the parent to understand them. Include the subject if needed, e.g. `conn.open() failure →`.
- **Do not use placeholders such as "continue" or "and then".** When branches converge, repeat the actual next-step name at each leaf. Repetition is allowed: the goal is each line being self-contained, not minimizing duplication.
- **Branch/leaf labels include "target + result + next step".** Avoid standalone words such as `success`, `failure`, or `continue`. For example, `failure → RetryableError('connection not open') → go to retry handling` identifies what happened and where it goes on that line. If the same result type (e.g. RetryableError from multiple sites) leads to the same next step, repeat identical destination wording at every occurrence instead of omitting it elsewhere.
- **Always draw conditional branches explicitly.** Do not bury conditions inside prose node names such as "try conn.open() if conn is not open". Separate `true`/`false` or actual conditions as sibling branches, so branch-specific side effects (e.g. skipping versus calling `open()`) do not disappear into prose. A decision splitting by a value (e.g. status variants) uses the same syntax: sibling branches beneath one decision node. Do not mix decision nodes with flat lists of alternatives.
- **When a branch terminates the operation, close its final exit inside that branch.** If some branches already end with ERROR/SUCCESS internally, do not put a common exit (e.g. a separate final `SUCCESS` after the sibling list) outside them. An outside common exit needlessly suggests terminated branches may continue there. Put each exit as the final child where that branch itself terminates.

  Bad (failure already ends with ERROR, but an outside common SUCCESS makes the relationship ambiguous):

  ```
  deletePatient(id)
  ├─ failure
  │  └─ set error state → ERROR
  └─ success
     ├─ Query invalidate
     └─ Zustand clear
  └─ SUCCESS
  ```

  Good (the success branch itself ends with SUCCESS):

  ```
  deletePatient(id)
  ├─ failure
  │  └─ set error state → ERROR
  │
  └─ success
     ├─ Query ['patients'] → invalidate → refetch
     ├─ patientStore → selectedPatient = null
     ├─ deleteModalOpen = false
     └─ SUCCESS
  ```

Expansion example — expand `terminate` because its name hides different exit paths depending on in-flight cleanup:

```
terminate
├─ in-flight cleanup exists = true
│  └─ return existing cleanup Promise → exit with that Promise's result
│
└─ in-flight cleanup exists = false
   └─ owner terminate
      ├─ owner terminate succeeds → SUCCESS
      └─ owner terminate fails → ERROR
```

Retry example:

```
send
├─ attempt
│  ├─ attempt succeeds → SUCCESS
│  └─ attempt fails
│      ├─ retry allowed → ↻ attempt (maximum 3 attempts)
│      └─ retries exhausted → ERROR
```

Combined example — when conditional and value decisions mix, use one consistent syntax (decision node → sibling branches) and include target + result + next step at every leaf. Repeat identical destination wording even when the same result type (RetryableError) arises at different sites:

```
sendWithRetry
│
├─ ↻ attempt (maximum 3 attempts)
│  │
│  ├─ getConnection(payload.target)
│  │  └─ [Needs verification] globalConnectionPool definition unavailable
│  │
│  ├─ conn.isOpen?
│  │  ├─ true
│  │  │  └─ proceed to conn.write
│  │  │
│  │  └─ false
│  │     └─ conn.open()
│  │        ├─ success → proceed to conn.write
│  │        └─ failure → RetryableError('connection not open') → go to retry handling
│  │
│  ├─ conn.write(payload.data)
│  │  └─ return ack
│  │
│  └─ evaluate ack.status
│     ├─ 'rejected'
│     │  └─ FatalError → throw immediately → ERROR
│     │
│     ├─ 'timeout'
│     │  └─ RetryableError('ack timeout') → go to retry handling
│     │
│     └─ otherwise
│        └─ return ack → SUCCESS
│
└─ retry handling (on RetryableError)
   ├─ retries remain → sleep(backoff) → ↻ attempt
   └─ retries exhausted → throw Error(...) → ERROR
```

Reading follows these four rules:

```
Top → bottom   Execution order
├─ / └─        Branches
Indentation    Which action produced the result
→              Next flow / final destination
↻              Repetition / retry
```

State-mutation example — show that the presence and kind of state changes differ between success/failure branches:

```
[Event] Click delete patient button
│
├─ Check selectedPatientId
│  ├─ absent → exit
│  └─ present → call deletePatient(id)
│
└─ deletePatient(id)
   ├─ failure
   │  └─ set error state → ERROR
   │
   └─ success
      ├─ TanStack Query ['patients'] → invalidate → refetch patient list
      ├─ Zustand patientStore → selectedPatient = null
      ├─ Local State → deleteModalOpen = false
      └─ SUCCESS
```

If `patientStore` also receives writes from another independent entry point unrelated to this flow (e.g. Header logout, which deletion does not call), reuse its name in the Overlap Candidate below. (`['patients']` invalidate → refetch is directly caused by this flow, not Overlap; see the rules above.)

Reverse Consumer Lookup example — absent from default output; append below a mutation only when impact scope is requested. Distinguish read/write and whether rerendering is confirmed:

```
Zustand patientStore.selectedPatient = null
│
├─ Consumer: PatientDetail
│  ├─ File: pages/patient/PatientDetailPage.tsx
│  ├─ read: selectedPatient
│  └─ change → rerender [Confirmed in code]
│
└─ Consumer: PatientActions
   ├─ File: components/patient/PatientActions.tsx
   ├─ read: selectedPatient
   ├─ write: setSelectedPatient(...)  [See Overlap Candidate]
   └─ subscribes to selectedPatient (rerender [Needs verification])

(Only consumers found in the current search scope — not exhaustive)
```

IPC boundary example — mark the boundary but continue if implementation is in the repository:

```
restart
├─ CLI validation
├─ [IPC] daemon.stop
│   └─ Daemon process
│       └─ stopDaemon()
│           └─ ... (continue tracing)
└─ [IPC] daemon.start [Needs verification]  ← daemon process implementation absent from repository
```

Overlapping Flow example — show separately when one tree cannot express it:

```
Flow A: cleanup
start
  ↓
Store Promise
  ↓
await ───────────────→ complete
        ↑
        │
Flow B  │
force terminate
```

[Overlap Candidate]: While `Promise` persists, the `force terminate` entry point can access the same state in code. Actual concurrent execution [Needs verification].
