# Rendering examples

These are hypothetical examples. Populate real output from inspected code. No example requires
a particular library or tool.

## Mutations and branch-local exits

```text
[Event] Delete record click
└─ selectedId decision
   ├─ absent → return undefined
   └─ present → deleteRecord(id)
      ├─ deleteRecord fails
      │  ├─ errorState → store deletion error
      │  ├─ cache → unchanged
      │  └─ deletion handler → return undefined
      └─ deleteRecord succeeds
         ├─ cache ['records'] → invalidate
         │  └─ refetchRecords [Confirmed in code]
         │     ├─ refetchRecords succeeds → cache stores list → SUCCESS
         │     └─ refetchRecords fails → store query error → ERROR
         ├─ recordStore → selection = null
         ├─ localState → modalOpen = false
         └─ deletion handler → return undefined
```

Do not mechanically label a handler `ERROR` if it catches a failure and returns normally.
Indicate whether refetch is awaited or merely started asynchronously according to actual code.

## Node expansion

```text
[Expanded: restartDaemon > startDaemon > createServer > failure > cleanup]
cleanup
├─ closeServer()
├─ runtimeState → server = null
└─ cleanup completes → rethrow original startDaemon error → ERROR
```

```text
terminate
└─ inFlightCleanup exists?
   ├─ true → return existing cleanup Promise → exit with that Promise's result
   └─ false → owner.terminate()
      ├─ owner.terminate succeeds → SUCCESS
      └─ owner.terminate fails → ERROR
```

## Compound decisions and retry

```text
sendWithRetry
├─ ↻ attempt (at most 3 attempts total)
│  ├─ getConnection(payload.target)
│  │  └─ [Needs verification] pool implementation unavailable; below follows returned conn
│  ├─ conn.isOpen?
│  │  ├─ true → proceed to conn.write
│  │  └─ false → conn.open()
│  │     ├─ conn.open succeeds → proceed to conn.write
│  │     └─ conn.open fails → RetryableError('connection not open') → retry handling
│  ├─ conn.write(payload.data)
│  │  └─ conn.write returns → ack.status decision
│  └─ ack.status decision
│     ├─ 'rejected' → throw FatalError → ERROR
│     ├─ 'timeout' → RetryableError('ack timeout') → retry handling
│     └─ otherwise → return ack → SUCCESS
└─ retry handling (RetryableError)
   └─ attempt count < 3?
      ├─ true → sleep(backoff) [external] → ↻ attempt
      └─ false → throw Error('retry exhausted') → ERROR
```

## Reverse Consumer Lookup

Attach this information beneath the mutation node only when impact scope is explicitly requested.
Cross-reference a writer to Overlap only after separately establishing all three candidate criteria.

```text
recordStore.selection = null
├─ Consumer: RecordDetail
│  ├─ File: pages/RecordDetail.tsx
│  ├─ read: selection
│  └─ change → rerender [Confirmed in code]
└─ Consumer: RecordActions
   ├─ File: components/RecordActions.tsx
   ├─ read: selection
   ├─ write: setSelection(...) [See Overlap Candidate]
   └─ subscribes to selection (rerender [Needs verification])

Current search scope: pages/, components/, stores/
Only consumers found within the current search scope — not exhaustive
```

## IPC boundaries

```text
restart
├─ validateConfig()
└─ [IPC] daemon.stop
   └─ [process] daemon → stopDaemon()
      ├─ stopDaemon fails → error response → ERROR
      └─ stopDaemon succeeds → [IPC] daemon.start
         └─ [Needs verification] daemon-side start handler absent from repository
```

## Overlapping flows

```text
Flow A: cleanup
start → store cleanupPromise → await ─────────→ complete
                                ↑
Flow B: forceTerminate           │
independent call → write cleanupPromise / resource state
```

[Overlap Candidate]: During the mutable lifetime of `cleanupPromise`/resource, independent
`forceTerminate` writes the same state and order may affect the outcome [Possible in code].
Actual concurrent execution [Needs verification]. Search scope: the two entry points encountered
in forward tracing.

Mere read/subscribe access is not a candidate. Invalidation → refetch directly caused by deletion
is causal forward propagation; attach it to the mutation node.
