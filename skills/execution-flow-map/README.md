# Execution Flow Map usage guide

The trees below are hypothetical syntax examples. Actual output follows paths established in code.

### 1. Basic request

`Show the execution flow of restartDaemon`

```text
restartDaemon
├─ validateConfig()
├─ stopDaemon()
├─ startDaemon()
│  ├─ startApi()
│  ├─ startWeb()
│  └─ writeState()
└─ SUCCESS
```

The skill starts shallow and expands only nodes hiding meaningful order, exits, or state changes,
rather than expanding every function to the same depth. This example assumes hypothetical code
with no failure paths.

### 2. Reading the tree

```text
Top → bottom   Execution order
├─ / └─        Sibling steps or labeled alternatives
Indentation    Parent action's internals/results
→              Next step / state mutation / final result
↻              Retry / loop
```

Conditions split beneath a decision node.

```text
conn.isOpen?
├─ true → proceed to conn.write
└─ false → conn.open()
   ├─ conn.open succeeds → proceed to conn.write
   └─ conn.open fails → RetryableError → retry handling
```

### 3. Expansion is per node

There is no tree-wide `depth=3`. A can remain collapsed while B's children expand recursively
as needed. Inspecting code and expanding it in the output are different actions.

### 4. Follow-up expansion and paths

`Show more of startDaemon` outputs only that portion.

```text
[Expanded: restartDaemon > startDaemon]
startDaemon
├─ createApiServer()
├─ createWebServer()
└─ writeDaemonState() → SUCCESS
```

Then `Show cleanup after createWebServer fails` can produce:

```text
[Expanded: restartDaemon > startDaemon > createWebServer > failure > cleanup]
cleanup
├─ webServer.close()
├─ apiServer.close()
├─ runtimeState → cleared
└─ cleanup completes → rethrow original error → ERROR
```

`[Expanded: ...]` locates the current detail within the original map. The whole tree is not repeated.

### 5. Ambiguous node requests

If `Show more of cleanup` matches several nodes, the skill presents path candidates and asks
which one. If the preceding conversation and tree identify one clear target, it expands immediately.

### 6. Overview for complex trees

With at least five major top-level steps or at least two expanded nodes, normal progression
appears first.

```text
[Overview] restartDaemon → validate config → stop old daemon → start new daemon → write state → SUCCESS
```

The detailed tree always follows. An overview does not replace branches and exception paths.

### 7. State mutations are included by default

`Show the delete button click flow` includes actual changes to setters, global stores, query
caches, refs, DB, memory, and runtime state in the branch and at the point where they happen.

```text
deleteRecord(id)
├─ deleteRecord fails
│  ├─ errorState → store deletion error
│  └─ rethrow deleteRecord error → ERROR
└─ deleteRecord succeeds
   ├─ cache ['records'] → invalidate → refetchRecords
   ├─ recordStore → selection = null
   ├─ localState → modalOpen = false
   └─ SUCCESS
```

Code that returns normally after catching an error shows its actual return value. A failure does
not always mean that the operation terminates with ERROR.

### 8. Impact scope is Reverse Consumer Lookup

An explicit request such as `Also show what is affected when selection becomes null` searches
backward from state to consumers. Discovered files, roles, read/write access, and established
subsequent behavior appear below the mutation. The output always states the search scope and
`Only consumers found within the current search scope — not exhaustive`. Default forward tracing
does not perform an exhaustive search for global consumers.

### 9. Readers and writers are distinguished

```text
recordStore.selection = null
├─ Consumer: RecordDetail
│  ├─ File: pages/RecordDetail.tsx
│  ├─ read: selection
│  └─ change → rerender [Confirmed in code]
└─ Consumer: RecordActions
   ├─ File: components/RecordActions.tsx
   ├─ read: selection
   ├─ write: setSelection(...)
   └─ subscribes to selection (rerender [Needs verification])
```

Mere read/subscribe access is not an Overlap Candidate. Writers receive `[See Overlap Candidate]`
only after separately meeting all three candidate criteria. Subscription alone does not prove rerender.

### 10. Overlap Candidates between independent flows

A shared mutable lifetime + independent entry point + write or order-dependent access must all
be established. Candidates encountered during default tracing are reported too.
`Find other independent flows accessing this state` explicitly requests a reverse search.

```text
Flow A: polling          Flow B: user edit
fetch                    click
  ↓                        ↓
recordState write        recordState write

[Overlap Candidate] Independent writes within the same mutable lifetime [Possible in code]
Actual concurrent execution [Needs verification]
```

Accessibility alone does not establish a race or bug.

### 11. Propagation versus overlap

`Successful deletion → invalidate → refetch` is a causal chain in which one flow directly triggers
another; it belongs under the mutation node. Independent polling and clicks modifying the same
state are evaluated against the candidate criteria.

### 12. Pages are split into interactions

`Show the RecordListPage flow` first inspects code and lists discovered interactions such as mount,
delete click, filter change, pagination change, and unmount. Select `delete click`, `mount and
pagination`, or `all`. Independent interactions are not drawn as one giant sequential tree.

### 13. Retry and loops

Ask `Show the sendWithRetry flow` or `Expand the retry path`.

```text
retry handling
└─ failure type decision
   ├─ FatalError → throw immediately → ERROR
   └─ RetryableError → attempt count < 3?
      ├─ true → backoff → ↻ attempt
      └─ false → throw retry exhausted error → ERROR
```

The map distinguishes attempt limits, termination conditions, and non-retryable failures.

### 14. IPC/process/module boundaries

Ask `Trace from the CLI into the daemon`. Boundaries are labeled and tracing continues when the
repository contains the implementation. Otherwise it stops, for example:
`[IPC] daemon.start → [Needs verification] handler definition unavailable`.

### 15. External boundaries

Standard/third-party libraries and framework internals stop at `[external]`. Tracing continues
through project code handling their results/errors or executed through callbacks/effects.
Registration and execution times are distinguished.

### 16. Useful prompts

| Goal | Example request |
|---|---|
| Overall structure | Show the restartDaemon flow |
| Specific function | Show the execution flow of sendWithRetry |
| Page | Show the RecordListPage flow |
| Event | Show the delete button click flow |
| Expand a node | Expand startDaemon |
| Deeper expansion | Show the Web failure cleanup within that |
| State mutations | Include the state changes in this flow |
| Impact scope | Show what is affected by changes to selection |
| Consumer search | Find who reads/writes this store |
| Overlapping flows | Find other independent flows accessing the same state |
| Retry | Expand the retry path |
| Cleanup | Show failure cleanup in detail |
| Process boundary | Follow IPC into the daemon implementation |

Explore through a flow map → node expansion → explicitly requested state impact search → independent
flow candidates. The map structures facts. Reviewing whether order, completion conditions, or
failure recovery are appropriate is a separate task that can use this map as input.
