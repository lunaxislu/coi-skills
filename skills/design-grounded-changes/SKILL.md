---
name: design-grounded-changes
description: Design, implement, or review software and documentation changes from observed needs and existing system capabilities. Use for features, defects, hardening, refactors, operations, and documentation when Codex must establish the real outcome, trace current responsibilities and flows, distinguish evidence from assumptions, evaluate built-ins/frameworks/installed libraries before new machinery, keep scope YAGNI, and verify the resulting behavior.
license: MIT
---

# Design Grounded Changes

Ground every change in an observable outcome and the system that already exists. Only after
settling what must change and what current capabilities already provide do you pick the smallest
fitting mechanism.

Throughout, "contract" means any obligation the system already binds itself to — a documented API
or spec, a persisted schema, an established UI/UX behavior, a test that encodes expected behavior,
or an explicit product decision — not merely a convenient assumption.

## 1. Frame The Change

Classify the primary intent before picking implementation details:

- Feature: state the user or system scenario and the observable outcome.
- Defect: reproduce it, or otherwise establish the reachable path, expected behavior, and actual
  harm. A result that is merely large or unusual is not a defect unless it violates a contract or
  produces a harmful failure mode. Before calling something a defect, settle whether what you
  observed is state that persists after the task completes, or transient state that exists only
  while a single task is still in flight — the latter is not evidence of the former.
- Hardening: establish the reachable threat or failure path, the asset being protected, and the
  consequence. Unless a concrete failure path, privilege, or access condition surfaces, assume
  components and peers honor their own contracts. Mark hypothetical risk as a hypothesis until it
  is verified or an explicit boundary contract states otherwise.
- Refactor: show the current structural cost and name the behavior that must stay unchanged.
- Operations: state the transition, the resources involved, and the externally observable
  success/failure states.
- Documentation: state the reader judgment or action the document must support, and the
  authoritative source it represents.

Separate confirmed facts, inferences, proposals, and choices that are still undecided. Treat any
constraint, limit, storage format, API, or abstraction that no existing contract already fixes as
a candidate mechanism, not a requirement.

## 2. Reconstruct The Current System

Investigate the surrounding system enough to narrate the change start to finish:

1. Find the code or document that owns the responsibility.
2. Trace callers, producers, consumers, state transitions, cleanup paths, and tests across every
   affected boundary.
3. Check the specs, decisions, architecture, repository instructions, and dependency manifests
   that constrain this work.
4. Describe the current flow. For a feature, defect, hardening, operations, or documentation
   change, pinpoint exactly where the desired outcome and the actual outcome diverge. For a
   refactor, there is no such divergence by definition — instead pinpoint exactly where the
   current structure imposes the cost you're removing.

Before asking the user to choose between candidate designs, if the choice depends on how the
affected components currently interact, first show a small responsibility/composition map and a
short event or state flow, and connect each option to observable behavior.

## 3. Deliberately Reuse Existing Capability

Review implementation sources in this order, and record only findings that would change the
resulting design or implementation:

1. Existing project behavior, helpers, data models, conventions.
2. Primitives of the language, runtime, platform, or browser.
3. Framework primitives and supported extension points.
4. Already-installed packages/libraries — including whether the currently installed version
   actually provides the needed behavior.
5. The smallest local implementation.
6. A new dependency or abstraction, only when none of the above fit and the
   operational/maintenance cost is justified.

Do not pick a package merely because it exists, and do not reimplement a capability merely because
a local solution is possible. Compare fitness, semantic mismatch, failure behavior, ownership,
testability, and lifecycle cost.

## 4. Choose The Smallest Fitting Design

Define the behavior contract before fields and mechanisms. Keep only the work that an established
scenario, contract, or verified failure path actually requires.

- Prefer a narrow-scope change in the component that owns the responsibility.
- Preserve existing semantics unless changing them is part of the stated outcome. For a refactor,
  keeping the existing external facade stable while its internals change is exactly this — it is
  not backward-compatibility code.
- Avoid unverified extensions such as speculative limits, generalized event models, storage
  systems, or configuration surfaces.
- Do not add backward-compatibility or transition code — fallback paths, compatibility layers,
  aliases, dual-read/write — without an explicit request. This applies when you are bridging an
  intentionally changed behavior, not when you are simply leaving an unchanged facade alone. If
  you judge new compatibility code necessary, do not add it quietly; report the rationale and
  alternatives and ask for a decision.
- When a value or policy must be set, derive it from an existing contract, observed workload,
  platform constraint, or explicit product decision. If none of these determine it, leave the
  mechanism configurable only when configurability itself is justified. If neither applies, treat
  the value as an open decision and ask the user rather than picking one silently.
- Flag adjacent findings separately. Do not silently expand implementation scope.

Use observation targets that fit the domain. A UI change may observe events, state, rendering,
accessibility, and error display; an API change may observe request/state/response behavior; a CLI
may observe process output and exit status; a service may observe resource lifecycle and
diagnostics; a document may observe the reader's final judgment or action. Do not force one
domain's fields onto another.

## 5. Implement And Verify

Implement through the owning path, and separate structural cleanup out when it would obscure the
behavior change. If the code reveals a simpler built-in or installed-library path during
implementation, repeat the capability review and update the design from step 4 accordingly — do
not keep an already-chosen mechanism once a smaller fit is confirmed.

Verify proportional to risk:

- Run the real scenario, or the closest deterministic reproduction.
- Add or update focused regression tests for the changed contract.
- Confirm the success, expected-failure, and cleanup/state-transition paths this change actually
  affects.
- Run relevant broader checks when the blast radius crosses a boundary shared with other
  components, services, or consumers.
- Report what you verified, what remains inference, and the residual risk. Do not claim a
  hypothetical path is fixed without executing it or proving it through contract and control flow.

If the evidence does not justify a production change, stop at the finding level and recommend the
smallest next probe or decision instead of manufacturing implementation work.
