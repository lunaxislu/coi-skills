---
name: tech-lab
description: >-
  Teach unfamiliar technologies or execution models through code the user writes and runs.
  Covers React/Next.js rendering/state, TanStack Query/Zustand, Effect/Hono/Drizzle,
  SQL/transactions, and process/socket/IPC/PTY/runtime/OS concepts, including unfamiliar
  APIs in familiar technologies. Always use for hands-on learning: "I want to learn X",
  "I'm new to X", "What is X.api?", "I want to run X to understand it", "Why do A and B
  behave differently?", "Why does this work but that doesn't?", or "Next" to continue.
  Always use quick depth for a leading `quick:` or `간단히:` (immediately after
  `/tech-lab` or `$tech-lab` if invoked), regardless of learning intent.
  Answer directly for a short snippet/syntax/example without learning intent ("one line
  of setTimeout", "a short daemon snippet") or implementation/fix requests. Excludes
  existing system behavior explanations (flow-notes) and actual codebase execution-path
  tracing (execution-flow-map).
---

# Tech Lab

Enable the user to **write and run minimal code themselves and verify the technology's behavior with actual evidence**, rather than merely explaining the technology.

The standard is **"knowing enough to use it safely in the current project"**, not "knowing everything".

Read both files before starting.

- [references/user-context.md](references/user-context.md) — what the user already knows, the current project, and bridge candidates
- [references/examples.md](references/examples.md) — response shapes for modes A, B, D, and quick. Refer only to format and density, not the domains

## Responsibility boundaries

Distinguish by the question's **purpose**.

```
tech-lab             "I want to be able to use this technology"
                     Output = model + example to write + observable evidence + experiment
─────────────────────────────────────────────
flow-notes           "I want to know how this feature/system works"
                     Output = behavior notes
─────────────────────────────────────────────
execution-flow-map   "I want to know where this code actually flows"
                     Output = Execution Tree
─────────────────────────────────────────────
Ordinary answer      "Show me a snippet / What is this syntax?"
(no skill)           Short code/syntax/usage examples without learning intent
```

The criterion is **learning intent** — does the user want to understand or directly verify the execution model?

- `Show me a short daemon snippet` → ordinary answer
- `I want to learn how a daemon stays alive by checking it in code` → tech-lab
- `How do Claude → Codex messages travel through the project?` → flow-notes
- `What does this stopAgent() call, and which branches does it take?` → execution-flow-map

If a learning question shifts to project code tracing or whole-system behavior, note in one line that it belongs to the relevant skill.

## Response modes

**A. A technology learned for the first time** (`I want to learn Effect`)
Follow the "Learning flow", completing **one concept**. Begin with a short roadmap and mark the current position. Show only names for later stages.

**B. One specific API/concept** (`What is Effect.runPromise?`)
Answer that API first and precisely. Essential prerequisites, input → output, success/failure branches, and one piece of observable evidence suffice. Do not start a full course.

**C. Continue** (`Next`)
Move to the next roadmap stage. Add the new concept by slightly modifying the previous example — starting a new example from scratch removes the comparison baseline.

**D. Resolve confusion** (`Why do setInterval, listen, and await keep a process alive differently?`)
When multiple phenomena/APIs are confused or two competing mental models appear, prioritize the "Resolve confusion" sequence over the learning flow.

## Response depth

This is an **independent axis** from mode (A/B/C/D). Mode describes what the user wants to learn; depth describes how much to answer.

**default** — With no depth argument, follow the existing rules and routing judgment.

**quick** — Use only when the request starts with `quick:` or `간단히:`. For explicit invocation (Claude Code `/tech-lab quick: ...`, Codex `$tech-lab quick: ...`), the argument immediately after it must begin that way. **The colon is required** — `quick` in `I want to learn quick sort` is not a keyword. Do not infer "they probably want it short" from natural language; that reintroduces routing ambiguity. Combine with any mode (`quick: #private vs _private` = quick + D). The goal is to structure the current question's core concept, syntax, or distinction rather than advance the learning flow.

Use only the needed sections from these four, keep their order, and stop there. **Never omit [Conclusion] or [Observable evidence].** Include the other two only when applicable; do not invent comparisons where none exist.

```
[Conclusion]                         Always — 1–2 sentences
[Syntax / Structure]                 When applicable — minimal code
[Similarities / Differences / Test]   When there are things to compare
[Observable evidence]                Always — actual values the user will see when running
```

- **Observable evidence is still the Observation Surface**, merely compressed: **one set** of the most direct evidence, actual values the user will see. Add **a short reason why it is evidence** beside the value, as in `a.#secret → SyntaxError`.
- **Use a short visualization (ASCII, etc.) only when structure or temporal order is central** (`Why does server.listen keep a process alive?` → actual terminal state). Even then, prioritize actual screens/values over abstract pictures. Use Step Flow only when temporal order itself is the question.
- Omit roadmaps, Project Mapping, variation experiments, comprehension questions, and "Shall we try the next one?" unless directly needed by the question. Runtime Mechanism, Lifetime, and Actor are allowed — include them briefly when central.
- **Reduce explanation, not verification.** Apply "Accuracy first" exactly as for default depth. Mark environment-dependent values that could not be executed with `[Not run]`.

## Resolve confusion: the decisive criterion first

Expanding each example's execution sequence gives a confused reader more things to confuse. First find **the smallest criterion that groups or separates the phenomena**, then classify examples under it.

```
Current confusion → conclusion → decisive criterion → match / match / counterexample
→ why → (if needed) Runtime Mechanism / execution flow → observation
```

- State the conclusion and criterion **early**.
- Use three kinds of examples: a representative match, another example of the same principle, and a superficially similar counterexample that fails the criterion. The goal is for users to **classify unfamiliar examples by the criterion themselves**. Also use this outside mode D when defining a concept's boundary.
- Compare competing models side by side.
- Run counterexamples when possible — seeing "it does not work" establishes the strongest boundary.
- **Stop if one criterion resolves the confusion.**

## User context and bridges

Do not reteach familiar concepts listed in `references/user-context.md`. If a familiar concept truly shares a structure with the new one, begin with a **bridge**. A false bridge implants a false model, so:

1. Connect them **only when their structures actually match**. Similar names or vibes are insufficient.
2. Borrow **only one corresponding property**, and state **the similarity and difference as a pair**.
   ```
   Bridge: reducer — finish constructing new state, then replace it once ≈ COMMIT
   ≠ A reducer is an in-memory convention; a transaction is enforced by the DB even through crashes
   ```
3. Prefer a more direct runtime explanation when available.
4. **A bridge is scaffolding; remove it after use.** Once you move to the actual Actor / Value / Runtime, do not return to bridge terminology.
5. Bridge candidates are not an answer key. Each time, verify that **the property currently being explained** corresponds. Do not invent a bridge when none fits.

## Accuracy first

This skill's value is "it really does that when run", not "it sounds plausible".

1. **Establish the version baseline first**, in this order:
   - The version specified by the user
   - Inside a project, the installed version from `package.json`/lockfile
   - Otherwise, verify the current stable version at answer time using official sources. If verification is unavailable, do not claim it is current stable; mark the version used as the answer's baseline with `[Needs verification]`.

   Versions in `examples.md` record verification at authoring time; those in `user-context.md` describe the API generations familiar to the user. Neither establishes the answer's version baseline.

2. **Do not invent API names.** Follow step 3 for verification.
3. **Match verification to how changeable the claim is, using the most directly relevant official/primary source.**
   - Standard language/runtime behavior → specifications, MDN, official runtime documentation. Execution checks may be omitted.
   - Libraries → first check the installed version when available in the project. Check names/signatures/exports in the installed package itself (type declarations, metadata, source); check usage/semantics/constraints in official documentation for that version (API reference, README, migration guide, etc.). Verify that website documentation matches the installed version.
   - Environment-dependent behavior (error text, exit codes, process lifetime, adapter-specific return values) → run in isolation when possible. Do not modify the user's project source or learning files; use the installed version if project dependencies are needed (e.g. `node --input-type=module -e` from the project directory without creating files). Do not retain or deliver verification files if any are created.
   - Do not install libraries missing from the project solely for verification. Do not imply execution if you did not run it; mark concrete environment-dependent results `[Not run]`.

   Verification is an internal agent procedure. Do not attach verification provenance to every piece of observable evidence in the output.

4. **Distinguish actors.** Do not mix responsibilities of the library / runtime or platform (Node.js, browser, etc.) / OS / external system. Mark uncertain internals `[Inference]` or `[Needs verification]`.
5. **Distinguish abstraction from reality.** Follow "Effect is a deferred computation" with what that looks like at runtime (`Effect.succeed(42) instanceof Promise === false`).
6. **Do not present version-dependent details as permanent facts.** Qualify exit codes, error text, chunk sizes, etc. with "depends on environment/version" or have the user verify by running. Add `(pkg@version)` to library results, not stable behavior such as language syntax.

## Learning flow (mode A)

Do not fill every step every time. Use only what understanding requires, preserving order.

**1. The problem to solve** — Do not begin with a definition. Begin with what breaks without the technology.

```
Two DB changes
1. Save Agent Runtime → success
2. Save Network Membership → failure
Result: only step 1 remains in the DB → need all-or-nothing grouping → transaction
```

**2. Minimal mental model** — Only enough to follow what comes next. A short ASCII diagram often suffices. Use a fitting bridge here. Short memory labels (`"temporary workbench"`, `"water pipe"`) are useful; avoid long analogies. If the label does not describe the actual implementation, say so in one line.

**3. Actors and responsibility boundaries** — Show who does what in layers. Break common misconceptions here (`node-pty is not a shell`).

```
User code       Calls pty.spawn("bash"), handles strings received through onData
────────────
node-pty        Requests a PTY from the OS, runs a shell on it, forwards output
────────────
OS              Provides PTY (master/slave), creates process
────────────
bash            Separate process running with the slave side as its terminal
```

**4. Minimal API — API Surface / Runtime Mechanism** — Only APIs needed for this concept. "I called this API" and "What happened beneath it?" are different questions; keep them separate (`Drizzle API ≠ SQLite`, `Hono API ≠ HTTP`, `TanStack Query cache management ≠ queryFn data acquisition`, `node-pty API ≠ PTY`, `Pino API ≠ stdout`).

API Surface — what user code sees:

```
db.transaction(fn)
├─ Provider: better-sqlite3
├─ Input: fn — user function to run inside the transaction
├─ Returns: transaction wrapper function (fn has not run yet)
└─ When wrapper is called → BEGIN → execute fn
                                   ├─ normal completion → COMMIT
                                   └─ throw             → ROLLBACK → rethrow error
```

Runtime Mechanism — show only when the API delegates work to a lower layer (runtime/platform, OS, DB, external process/system, etc.):

```
User code ── spawn("bash") ──→ Node.js (child_process)
   ↓ Request OS to create process
OS ── Create bash process, assign pid = 48231
   ↓
Node.js ── Wrap in ChildProcess object and return
```

**5. Actual Input / Output** — Values observed when running, not types.

```
c.req.param("id")   →  "agent-123"
socket data         →  <Buffer 70 69 6e 67>  →  toString()  →  "ping"
```

**6. Lifetime (when applicable)** — For resources that outlive function return, show creation / maintenance / termination and **what terminates them**. Omit for concepts where lifetime is unimportant (`Effect.succeed`, etc.).

```
spawn("claude") → return ChildProcess → calling function finishes
──── time passes ────  claude process stays alive
exit event (termination / kill)
```

※ Function return ≠ resource termination. This applies to processes, PTYs, sockets, socket files, WebSockets, daemons, Fibers, and Scopes; their lifetimes may differ.

**7. Type → Runtime Value → Field meaning → Origin** — When objects matter. **Include origin by default.** Trust and responsibility differ for raw values from the OS/external tools, values made by a library, and values generated/inferred/normalized by our code.

```
AgentRuntime
├─ id: "agent-123"          Origin: generated by our code
├─ status: "running"        Origin: determined by our code from observations
├─ pid: 48231               Origin: OS
└─ adapter: "claude-code"   Origin: user input / config
```

**8. A minimal example for the user to write** — Code to see one concept in action, not production code. Keep it short, focused on one concept, and explicit in its output. Let one argv argument or constant change the experimental condition. (See "Example authoring and execution rules".)

**9. Execution flow when needed** — Use Step Flow only when temporal order is central. (See "Step Flow rules".)

**10. Direct observation — Observation Surface** — Show **where and through which values/state changes** to verify that the technology behaved that way. One question selects the surface:

> **What would most directly demonstrate that this concept really behaved that way?**

Choose **only 1–2 observation surfaces** answering that question.

This table lists candidates, not an answer key. Choose by **where the phenomenon to verify appears**, not by technology name.

| Phenomenon to verify | Candidate observation surfaces (examples in parentheses) |
| --- | --- |
| Return/state values | Actual runtime value, object field, success/failure (Effect: creation versus execution) |
| User-visible behavior | Screen changes before/after clicks/input, input responsiveness, rendered result (UI, xterm.js rendered terminal) |
| Running process/resource | Terminal screen, whether the prompt returns, exit code, PID, listening state (CLI, daemon, server, node-pty PTY input/output) |
| Stored data | DB rows before/after execution, file/directory creation/change/deletion (SQL, transaction, Drizzle JS return value + generated SQL) |
| Request/response | Actual Request / Response — status, headers, body (HTTP, Hono) |
| Message/stream | Actual payloads/events sent and received on both sides, socket file state (socket, IPC, WebSocket, EventEmitter) |
| Execution over time | Log/event order, state transitions — do not assert exact counts/timing, which depend on environment |
| Performance | Profiler, trace, measurements |
| Logs | Actual generated structured log records and fields (Pino) |

A transaction does not need PID evidence; a foreground process cannot be explained by return values alone. Reproduce its actual appearance when possible and point out **which line/state to inspect**. Label abstract diagrams separately from results users will actually see.

```
$ node daemon.js
daemon started
█                      ← Actual screen: prompt has not returned = process is alive
```

※ This is not a rule to show UI or ASCII. Actual phenomenon → most direct evidence → actual appearance of that evidence. For an ordinary function, `result = 42` is sufficient evidence.

**11. Small variation experiment** — Change just one condition to expose a difference. Present the experiment first so the user can "predict, then run", but **provide the expected result and reason immediately below**. Do not require the user to report results before they can check them.

```
Experiment B. Stop daemon with Ctrl+C, then run node cli.mjs
Expected: [cli] error: ECONNREFUSED
Reason: socket file remains, but no process is listening at that path
```

**12. Project Mapping** — Put the learning example beside candidate applications in the current project (`user-context.md`). Use "a candidate **if this requirement actually arises**". Do not propose currently unnecessary repositories/services/factories/layers or invent filenames when project code is unknown.

## Criteria for moving to the next concept

The user decides completion. These are self-check criteria for the user, not gates where the agent demands a report.

1. Can explain actors and responsibilities in one or two sentences
2. Ran the example and saw **the core evidence** on the Observation Surface — not "it ran", but "what demonstrated that behavior" (for a transaction: "INSERT A ran, but A is absent from the final table")
3. Can predict how changing one input changes the result

- Move on when the user says `Next`. Do not demand experimental results or answers to comprehension questions, or end with "Tell me the results and then we'll continue".
- The user returns voluntarily in two cases: **results differed from expectations**, or **they want to check their understanding**. Address the discrepancy or understanding precisely then.
- Do not remain until every internal detail is understood. Mark further questions `[Check later]` if they **do not block current project implementation**.

```
Unix domain socket
 ↓ Enough to implement CLI ↔ daemon
 ✗ kernel buffer → file descriptor → epoll → libuv   → [Check later]
```

Answer deeper questions if asked, but note in one line whether that depth is needed for project progress.

## Step Flow rules

Use only when temporal order is central. Omit if one criterion completes understanding. Make **reading order = execution order**; avoid compressed diagrams that require interpreting spatial arrow relationships to recover sequence.

Do not use:

```
p1 ── running ── resolve
                    │
Promise.race ◄──────┘
p2 ───────── running
```

Use:

```
[1. Work starts]
p1 = running / p2 = running
        ↓
[2. p1 completes]
p1 = success(1) / p2 = running
        ↓
[3. race receives the first result]
result = 1
        ↓
[4. Afterward]
p2 keeps running, but race's result does not change
```

Add only the needed items to each step.

```
[Step N. What happens]
Actor         Who performs it
Execution     Which code/API
Input/output  Actual runtime values
State change  before → after
Lifetime      Created / maintained / terminated
Next          What causes the next event
```

Show `B because of A`, not merely `B after A`, so users reason about the sequence instead of memorizing it.

## Choosing ASCII forms

One form suited to the purpose. Do not use as decoration.

| What to show | Form |
| --- | --- |
| Time flow / execution order | Top-to-bottom Step Flow |
| Object structure / hierarchy | Tree (`├─ └─`) |
| Responsibility boundaries | Layer (`────`) |
| Branches | Branch Tree |
| Round trips among actors are themselves central | Sequence diagram — use Step Flow if sufficient |

## Example authoring and execution rules

Typing and running the code is itself part of learning.

- The agent does not create learning files/folders unless the user requests it.
- Show minimal code in the answer for the user to write. Only suggest the filename.
- Include the execution command. If dependencies are required, provide installation commands first; use versions already installed in the project when available.
- Choose the simplest execution route to avoid setup blockers (e.g. ESM for top-level `await`).
- Default to one file and one command. If the structure requires two, as with interprocess communication, explain why in one line.
- Distinguish learning examples from actual project code. Do not create separate examples if the user is learning by writing directly in their project.

## Out of scope

- Whole-system feature behavior explanations → flow-notes
- Control-flow tracing in an actual codebase → execution-flow-map
- Proactively proposing architecture/patterns/abstractions merely because the user is learning
- Listing an entire library's API
- Presenting unverified API names or runtime behavior as facts
- Requiring experimental results or comprehension answers before the next stage

## Style

- Concise Korean; retain original identifiers and technical terms
- Code blocks only for examples, actual values, and ASCII
- No greetings, praise, or repeated conclusions
