# Tech Lab Examples

Refer only to format and density, not the domains. Example versions record verification at authoring time; choose the actual answer's version baseline using SKILL.md's version rules.

## Example 1 — Mode A: transaction (better-sqlite3)

````
Problem
Save Agent Runtime succeeds → save Membership fails
→ Only Runtime remains in DB. Both must be handled as one unit

Minimal model
BEGIN ──── Changes here are on a "temporary workbench" ──── COMMIT → finalized
                                                        └── ROLLBACK → discard all
※ "Temporary workbench" is a mental model for atomicity.
  It does not mean SQLite actually collects data in a separate temporary table.

Bridge: reducer — finish constructing new state, then replace once ≈ COMMIT
        If it throws midway, state is not replaced ≈ ROLLBACK
≠ Reducer is an in-memory convention; SQLite enforces transactions even through crashes

Responsibilities
User code      Decides which queries belong together, signals failure with throw
better-sqlite3  BEGIN / COMMIT around fn; on throw, ROLLBACK and rethrow error
SQLite         Guarantees actual atomicity

API
db.transaction(fn) → transaction wrapper function (fn has not run yet)
Call wrapper → BEGIN → fn → COMMIT / on throw, ROLLBACK and rethrow error

Write it yourself — tx.ts (user creates the file)
```ts
import Database from "better-sqlite3";
const db = new Database(":memory:");
db.exec("CREATE TABLE users (name TEXT)");
const insert = db.prepare("INSERT INTO users (name) VALUES (?)");
const all = () => db.prepare("SELECT name FROM users").all();

const FAIL_SECOND = process.argv[2] === "fail";

const createTwo = db.transaction(() => {
  insert.run("A");
  console.log("insert A");
  if (FAIL_SECOND) throw new Error("insert B failed");
  insert.run("B");
  console.log("insert B");
});

console.log("before:", all());
try {
  createTwo();
} catch (e) {
  console.log("caught:", (e as Error).message);
}
console.log("after:", all());
```

Execution order (fail case)
[1. Create wrapper]
db.transaction(fn) → createTwo (transaction wrapper function)
※ fn ≠ createTwo. No queries have run yet
        ↓
[2. Call wrapper]
createTwo()
Actor: better-sqlite3 → BEGIN → execute fn
        ↓
[3. insert A]
Changes inside transaction (mental model): [] → ["A"]   (not finalized yet)
        ↓
[4. throw]
fn throws Error
        ↓
[5. ROLLBACK]
Actor: better-sqlite3
Changes inside transaction (mental model): ["A"] → []
Next: rethrow error to caller
        ↓
[6. User code catches]
caught: insert B failed

Observation — surface: DB state before/after transaction (prompt/PID unnecessary)
$ pnpm tsx tx.ts fail
before: []
insert A              ← A's INSERT actually ran
caught: insert B failed
after: []             ← Yet it did not remain. This is rollback

Experiment — predict, then run. Expected results immediately below
A. pnpm tsx tx.ts
   Expected: after: [ { name: 'A' }, { name: 'B' } ]
   Reason: fn completes normally → COMMIT
B. Remove db.transaction(...), call the same function directly, then pnpm tsx tx.ts fail
   Expected: after: [ { name: 'A' } ]
   Reason: without transaction, each INSERT commits immediately → A remains despite failure

AIOX application candidates
Learning example      AIOX
├─ insert A            ├─ Save Agent Runtime
└─ insert B            └─ Save Network Membership
→ Transaction is a candidate if both must actually succeed together
````

※ Do not end with "Tell me the results and then we'll continue". Users return only when results differ or they want to check their understanding.

## Example 2 — Mode B: `What is Effect.runPromise?`

````
Effect.runPromise(effect) = Actually execute an Effect (execution plan) and return its result as a Promise.

One prerequisite
Effect.succeed(42) is neither 42 nor a Promise. It is a plan before execution.
  Effect.succeed(42) instanceof Promise  →  false
Bridge: JSX element — <App /> is also a description object of "what to render", not execution
≠ React renders JSX itself, whereas Effect executes only when runPromise is called

Input → output
Effect.runPromise(Effect.succeed(42))  →  Promise, resolve(42)
Effect.runPromise(Effect.fail("boom")) →  Promise, reject(FiberFailure)

Branches
runPromise(effect)
 ├─ success → Promise resolve(value)
 └─ failure → Promise reject — error comes wrapped in FiberFailure
              (e.g. String(e) === "(FiberFailure) Error: boom" — wording depends on version)

Observation — surface: actual runtime value
```ts
// obs.mts — uses top-level await, so run as ESM (.mts or "type": "module")
import { Effect } from "effect"
const eff = Effect.succeed(42)
console.log(eff instanceof Promise)          // false — only a plan created, no 42 yet
console.log(await Effect.runPromise(eff))    // 42
```

Run it yourself
$ pnpm tsx obs.mts
false
42
$ █

Observation
false  ← eff itself is not a Promise — only a plan was created
42     ← Actual success value arrives only after runPromise executes
(Verified at authoring time: effect@x.y.z)

Evidence to check: user code receives the actual result 42 only after running runPromise.

Constraint
An Effect with remaining required context (R) causes a type error — supply required services before execution

When to use
The boundary from the Effect world to the Promise world (e.g. await inside a Hono handler)
````

## Example 3 — Mode D: `Don't debounce and useTransition both delay rendering search results?`

````
Conclusion
Both help avoid heavy rendering blocking input, but debounce only postpones when rendering starts,
while useTransition starts rendering immediately and interrupts that render when new input arrives.

Decisive criterion
→ If new input arrives during result rendering, can that render be interrupted to handle input first?

Classification
startTransition(() => setQuery(v))  → transition render is interruptible                  → input first (representative)
useDeferredValue(query)             → background render using deferred value interruptible → input first (same principle)
setTimeout(() => setQuery(v), 300)  → renders as ordinary update after 300ms              → blocks input once started
startTransition(() => {             → callback runs synchronously immediately            → blocks input (counterexample)
  const r = heavyFilter(v)            Only React rendering becomes interruptible,
  setResults(r)                       not computation inside the callback
})

Competing models
"transition = execute later"
→ Starts immediately rather than postponing. Appears late because it yielded to new input
"transition = make heavy code asynchronous"
→ Only rendering becomes interruptible. Ordinary JS computation still blocks the main thread

Observation — surface: actual input responsiveness
```tsx
// Add as one component to an existing React project and run (pnpm dev)
import { useState, useTransition } from "react";

const MODE: "plain" | "debounce" | "transition" = "plain"; // ← Change only this to compare
let timer: number | undefined;

function SlowItem({ text }: { text: string }) {
  const start = performance.now();
  while (performance.now() - start < 1) {} // Deliberately block for 1ms
  return <li>{text}</li>;
}

export default function App() {
  const [query, setQuery] = useState("");
  const [, startTransition] = useTransition();

  return (
    <>
      <input
        onChange={(e) => {
          const v = e.target.value;
          if (MODE === "transition") startTransition(() => setQuery(v));
          else if (MODE === "debounce") {
            clearTimeout(timer);
            timer = window.setTimeout(() => setQuery(v), 300);
          } else setQuery(v);
        }}
      />
      {/* Deliberately slow rendering so the difference can be observed */}
      <ul>{Array.from({ length: 300 }, (_, i) => <SlowItem key={i} text={`${query} ${i}`} />)}</ul>
    </>
  );
}
```

Type "abcdef" quickly

[plain]
Input: characters appear late, several together     ← Input handling blocked until rendering finishes

[debounce]
While typing: characters appear immediately         ← Rendering has not started yet
Type again after a brief pause: character is late   ← Render started after 300ms blocks input

[transition]
Input: each character appears as pressed            ← Ongoing render discarded; input handled first
List updates to the final value after typing stops

(Verified at authoring time: react@x.y.z — perceived speed depends on device; change 1ms to 3ms if difference is invisible)
````

※ No per-example execution sequences were expanded. One criterion resolved the confusion, so Step Flow is omitted.

## Example 4 — quick + mode D: `quick: What's the difference between #private and _private?`

```
[Conclusion]
#secret is private with language-enforced access restrictions; _weak is an ordinary property with a naming convention.

[Syntax / Structure]
class Sample {
  #secret = 1;
  _weak = 2;
}
const a = new Sample();

[Similarities / Differences / Test]
Similarity  Both can be read/written with this inside the class
Difference  #secret → JavaScript private element (not a property), inaccessible outside the class
            _weak   → ordinary property; `_` is a naming convention
Test        → Does the language actually block access?

[Observable evidence]
a._weak         → 2                 ← Readable from outside
a["#secret"]    → undefined         ← Looks up an ordinary string property named "#secret".
                                      This is not a failed attempt to read the private field
Object.keys(a)  → [ '_weak' ]       ← #secret is a private element, not a property; excluded
a.#secret       → SyntaxError       ← Private name outside the class → rejected during parsing
                                      (Even a preceding line printing "start" does not execute)
```

※ End without a roadmap, Project Mapping, variation experiment, or comprehension question. As language syntax, include neither verification provenance nor a version label.

## Example 5 — quick + mode B (library): `quick: What does Drizzle insert return?`

```
[Conclusion]
insert(...).values(...) builds a query; the return value depends on the execution method attached.

[Syntax / Structure]
db.insert(users).values({ name: "Moon" }).run()
db.insert(users).values({ name: "Kim" }).returning().all()

[Observable evidence]
toSQL()            → insert into "users" ("id", "name") values (null, ?)  params: ['Moon']
run()              → { changes: 1, lastInsertRowid: 1 }
returning().all()  → [ { id: 2, name: 'Kim' } ]
(Verified at authoring time: drizzle-orm@0.45.3 / better-sqlite3 — APIs/return values may vary by version)
```

※ Include a version line because these are library results. Do not attach verification provenance such as "(executed)" to every value. If unavailable dependencies prevented execution, mark these values `[Not run]`.

## Example 6 — Mode D + JavaScript bridge: `Why do setInterval and server.listen keep a process alive, but await and emitter.on don't?`

````
Conclusion
A Node process stays alive when "ref'ed work/resources remain for the event loop to wait for".
Timers and listening sockets are representative examples. await and emitter.on() look like "code to run later",
but by themselves create no ref'ed work/resources that Node must keep waiting for.

Bridge: browser event loop — when a timer finishes, its callback enters the task queue and runs later ≈ Node timer
≠ Browser tabs stay alive without pending work. Node exits without ref'ed work/resources to wait for

Decisive criterion
→ Did this code delegate work/resources outside JS (to the Node runtime), and are they ref'ed?

Classification
setInterval(fn, 1000)             → delegates timer to runtime (ref)         → alive (representative)
net.createServer().listen(3000)   → delegates listening socket to runtime    → alive (same principle)
setInterval(fn, 1000).unref()     → delegated, but ref removed                → exits (directly disables criterion)
emitter.on("hello", fn)           → only stores a function in a JS object     → exits (counterexample)
await new Promise(() => {})       → Promise is a JS object; no delegated work → exits (counterexample)

Competing models
"Registering a callback keeps it alive"
→ emitter.on only puts a function in a JS object; the event loop does not know it exists
"Unfinished async work keeps it alive"
→ A pending Promise is just a JS object; without delegating work to resolve it to the runtime, it cannot hold the process alive

Observation — surface: terminal prompt return and exit code
```js
// alive.mjs — .mjs because it uses top-level await
import net from "node:net";
import { EventEmitter } from "node:events";

const MODE = process.argv[2]; // interval | listen | unref | emitter | await
console.log("start:", MODE);

if (MODE === "interval") setInterval(() => console.log("tick"), 1000);
if (MODE === "listen") net.createServer().listen(3000, () => console.log("listening :3000"));
if (MODE === "unref") setInterval(() => console.log("tick"), 1000).unref();
if (MODE === "emitter") new EventEmitter().on("hello", () => console.log("hello"));
if (MODE === "await") await new Promise(() => {});

console.log("end of script");
```

Run it yourself
$ node alive.mjs interval
start: interval
end of script          ← Script reached its end, yet
tick
tick
█                      ← No prompt = ref'ed timer holds process alive (Ctrl+C to stop)

$ node alive.mjs emitter
start: emitter
end of script
$ echo $?
0                      ← Exits immediately despite registered listener

$ node alive.mjs await
start: await
Warning: Detected unsettled top-level await at file:///.../alive.mjs:12
$ echo $?
13                     ← Exits without printing "end of script"
                         No runtime work exists to wake the awaited Promise

(Verified at authoring time: Node.js v22 — exit code 13 and warning text depend on version)
````

※ The bridge appears only at the beginning; subsequent explanation uses Node terms (ref'ed work/resources). Event loop phases and libuv internals are omitted because one criterion resolves the confusion.
