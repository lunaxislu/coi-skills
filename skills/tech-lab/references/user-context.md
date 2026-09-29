# User Context

This file describes **this user and the current project**, not the learning method. Change only this file when the user or project changes.

## Already familiar — do not reteach

The user is a frontend developer. Familiarity extends only to the listed scope. Even within a familiar technology, do not assume knowledge of unlisted APIs/execution models (e.g. React useTransition, Server Components) if the user says they are learning them for the first time.

```
React v19          lifecycle, useEffect, cleanup, Context API
Immutability / reducer  Create new state and replace it once, (state, action) → state
Next.js v15        SSR
TypeScript
Tailwind CSS v4    JIT
TanStack Query v5  cache, queryKey, polling/refetch
Docker v29         docker run, docker compose
JavaScript
  runtime          call stack, execution context
  scope            lexical scope, scope chain, closure
  function         this, arrow function this, call/apply/bind
  async            Promise, async/await, browser event loop,
                   task/microtask, setTimeout
  DOM event        addEventListener, bubbling/capturing
```

## Currently unfamiliar areas — build from the ground up

Backend/OS areas such as process, PTY, socket, IPC, SQL, transaction, and server runtimes. Explain terminology first. The user often gets stuck on **long-lived resources** (daemon, socket, PTY, shell, WebSocket, Fiber), where the frontend intuition "function return = work finished" does not apply.

## Current project: AIOX

A project being rebuilt, and the target for Project Mapping. Components encountered while learning:

- CLI ↔ daemon (IPC, Unix domain socket)
- Running and observing agent processes (child_process, node-pty, xterm.js)
- Persisting agent runtime / membership state (SQLite, Drizzle, transaction)
- HTTP / WebSocket (Hono, WebSocket), logging (Pino), Effect

Do not infer AIOX internals not listed here. Describe candidate applications only at the requirements level.

## Bridge candidates

Not an answer key. Each time, follow SKILL.md's bridge rules to recheck that **the property being explained now** corresponds. For example, Effect ↔ JSX fits only when explaining "creating ≠ executing"; it says nothing about Effect's error channel, required context, or composition.

| New concept | Familiar concept | Similarity | Difference |
| --- | --- | --- | --- |
| transaction | Immutability / reducer | Commit once without exposing intermediate state | DB enforces durability; reducer is a convention |
| Effect value | JSX element | A description object of "what to do", not execution | Executed by a `run*` call, not React |
| Effect Scope / finalizer | useEffect cleanup | Pair resource acquisition with release | Scope termination determines when release happens |
| AbortController | Canceling fetch in cleanup | Send a cancellation signal to ongoing work | — (same Web API) |
| WebSocket | TanStack Query polling | Keep server data up to date | Polling pulls; WebSocket lets the server push |
| DB (SQL) | TanStack Query cache | Retrieve the same data by key | Cache is a copy; DB is the source |
| Hono handler | Next.js Route Handler | Web-standard `Request` → `Response` | Hono provides routing/middleware |
| child_process | `docker run` | Start a process and receive stdout/exit code | Container is isolated; child_process is a child on the same OS |
| node-pty | `docker run -t` | Allocate a pseudo-TTY and run a program on it | node-pty directly controls the PTY master in code |
| Unix socket / IPC | `docker` CLI ↔ `/var/run/docker.sock` | CLI process requests the daemon through a socket file | — (same structure; AIOX CLI ↔ daemon also follows it) |
| Pino | `console.log` | Output logs | One JSON object per line, searchable by level/fields |
| Node.js event loop | Browser event loop, task/microtask | Callbacks execute through the event loop, separately from current synchronous execution | Node scheduling differs from the browser and has dedicated APIs such as `process.nextTick` and `setImmediate` |
| Node process lifetime | Browser event loop | Async work such as timers/I/O executes callbacks later | Node exits without ref'ed work/resources to wait for (timer, `server.listen`, socket, etc.); listener registration or a pending Promise alone does not keep it alive |
| EventEmitter | DOM `addEventListener` | Register listeners for named events and execute them when events occur | `emit()` calls listeners synchronously; no capture/bubble propagation |
