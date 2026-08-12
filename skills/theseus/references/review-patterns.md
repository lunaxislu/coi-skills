# Review Patterns

Read this when classifying a structural problem, or when a document relationship's read path,
concept placement, owner, or change location is ambiguous. Each section spells out the SKILL
body's five axes (content/compression / representation / structural relationship / rule altitude /
document-to-document relationship) and the Review Loop as concrete instances; add an item only when
it passes the recurrence gate (it actually recurs, and the existing axes cannot restore it).

## Contents

- [Context Read Path](#context-read-path)
- [Concept Placement](#concept-placement)
- [Structure Smells](#structure-smells)
- [Owner And Scope Decisions](#owner-and-scope-decisions)
- [Change Location](#change-location)

## Context Read Path

Follow only the relationships that could change the judgment, to confirm what contract the
document sits under and how it divides roles.

- Start from the direct parent contract, and follow only as far up the ancestors it delegates to as
  affects the current owner/scope judgment.
- If a guideline hierarchy exists, check the derivation relationships among the project/root
  contract, parent guideline, child guideline, and the actual corpus, to the extent needed.
- Check the project's document authoring/editing flow, taxonomy, owner map, index, or entry
  document if it determines the current document's role.
- Read a sibling only when its symmetry or difference affects the judgment; read a child only to
  determine whether the current document owns a skeleton, an invariant, an index, or routing.
- Read the corpus or past cases only to confirm a rule's applicability or the chance of
  misunderstanding.
- Stop once you reach the root or a scope boundary, or once further reading no longer changes the
  judgment.

## Concept Placement

- First judge whether something is an independent section or content to absorb into the
  introduction or existing context.
- Check whether A is a subordinate concept of B, a sibling at the same level, or whether the title
  is creating a wrong higher-level concept.
- Keep items at the same level even within the same family if their purposes differ. Direct
  evidence and a provenance source are both related to traceability, but neither needs to be a
  subordinate concept of the other.
- Even something that looks like tool usage can be absorbed into the introduction if its real role
  is document identity or an edit condition.
- Before adding a new section or document, check whether renaming, absorbing, moving, or delegating
  can resolve it.

## Structure Smells

Each smell is a static surface where the identity anchor or one of the five axes' restoration tests
fails. It's a restoration gap that recurs often enough to have earned a name — it shares the same
test ("does the reader restore this judgment?") but has hardened into a form recognizable without a
dry-run. A dynamic gap that only surfaces through a dry-run is gathered by Node Operation instead.
The axes are an exploration aid, not an exclusive classification, so when one defect breaks
restoration across multiple axes, route it to every axis it touches. Keep a catalog entry only when
it passes the recurrence gate (it actually recurs, and the existing axes cannot restore it); an
entry restorable from axis principles alone is a candidate for removal.

**Content/compression restoration failure** — content doesn't carry enough to make the required
judgment, or it carries content that leaves the judgment unchanged if removed, creating reader
cost.

- `over-provisioned structure` — a section premised on a low-frequency exception or future content
  invites filling it with content that leaves the judgment unchanged if removed.

  The problem of a sentence that hands off to action stopping at a state/label instead of the
  required action (formerly `inferred action`) is owned directly by this axis's SKILL-body rule and
  the Output Contract, so it isn't kept as a separate smell.

**Representation restoration failure** — the form doesn't match the nature of the information, so
reader behavior (re-entry, verification, termination) isn't restored.

- `misaligned reader path` — the heading, order, or loop doesn't match the actual unit of work, so
  the reader misses a transition, cleanup, or verification needed to finish one task. (Also breaks
  structural-relationship restoration.)
- `abstract destructive gate` — a hard-to-reverse change is allowed through with abstract judgment
  words alone, without surfacing knowledge-loss, reference scope, or a fallback, so the reader
  can't restore the inputs needed for the judgment. (Also breaks content/compression restoration
  and the Destructive Change gate.)

**Structural-relationship restoration failure** — the hierarchy, sibling, or sub-kind relationship
among documents, sections, or concepts isn't restored.

- `hidden sub-kind` — a single structure hides a subtype difference such as index, slice, router, or
  standalone.
- `mixed concept roles` — one section mixes different conceptual roles — evidence, provenance,
  workflow, style — blurring each role's boundary and meaning.

**Rule-altitude restoration failure** — the altitude a rule applies at (global/local) and its
affiliation aren't restored.

- `hidden or implicit authority` — a needed rule, scope, or exception hides only in an example,
  title, or convention, so the reader can't restore even whether it's a rule, let alone its
  altitude.
- `orphan local rule` — a local rule disconnected from the parent contract or owner map, so its
  scope and precedence can't be restored. (Also breaks document-to-document-relationship
  restoration.)
- `owner or altitude mismatch` — content sits at the wrong owner or conceptual level, so which
  altitude or ownership the rule belongs to isn't restored. (Also touches
  document-to-document-relationship and structural-relationship restoration.)

**Document-to-document-relationship restoration failure** — the coupling between two documents
(delegation, complementarity, boundary) isn't restored.

- `duplicated ownership` — the same knowledge lives in two or more owners, so which side owns it and
  which delegates isn't restored.
- `reverse or circular dependency` — a downstream document defines or governs the upstream rule it
  depends on, so the delegation direction isn't restored.
- `contradiction` — two or more owners make incompatible claims.
- `copied section meaning` — a sibling's heading or order was copied without re-judging its role, so
  the actual difference in role between the two documents isn't restored.
- `excessive read path` — restoring a single judgment requires following more documents than
  necessary.

**Identity-anchor restoration failure (pre-axis anchor)** — the identity the axes derive from is
itself under-expressed.

- `under-scoped identity` — a document-identity statement or a section/scope label's name is tied to
  one subtype, so it can't hold the concept that should also cover the case where that subtype is
  absent. This often surfaces as an altitude mismatch or a missing reader path, but the cause is an
  under-expressed identity.

## Owner And Scope Decisions

- Judge ownership from the explicit contract, actual scope of application, authority, dependency,
  and lifecycle/change coupling.
- Don't copy content another document owns — link or delegate to the owner. If the current document
  needs a boundary, keep only the owner and how it's consumed, not the detailed procedure.
- Classify the relationship between new and existing content as same-owner, sibling, subordinate,
  or unrelated. If it exceeds the scope, compare delegation, scope expansion, splitting, or
  promotion to a higher-level concept.
- A restructuring that changes identity or scope has a large blast radius, so it requires a higher
  evidentiary bar.
- In a portable skill or general guideline, don't hardcode a specific project, repo, team name, or
  path — turn cases into abstract patterns.

## Change Location

- Check whether a structural smell occurs only in the local document, or recurs at a
  parent/ancestor boundary.
- Compare how `local-only`, `ancestor-only`, `both`, and `no-change` each change the next author's
  behavior and read path.
- Revise or reject a downstream change that conflicts with settled higher-level authority. If the
  ancestor itself is stale, overbroad, or misplaced, present the affected dependents and evidence
  and escalate the change through the owner's process.
- If the evidence for an ancestor change is insufficient, keep a non-destructive choice such as a
  local boundary note or `no-change`.
