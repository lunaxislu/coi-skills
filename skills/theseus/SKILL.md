---
name: theseus
description:
  Use when an agent needs to create, delete, structurally review, design, or refine durable documents,
  documentation rules, authoring guides, templates, skeletons, or review feedback. Recovers the document's
  identity — what it represents — from the document and its parent and sibling relationships, then derives
  content, representation, structure, rule-altitude, and document-relation judgments from that identity
  through a review loop that propagates only along a change's blast radius.
license: MIT
---

# Theseus

## Definition And Identity

A document is a durable carrier of information, instructions, and history. Every judgment this
skill makes rests on the target document's **identity — "what this document represents."** Recover
identity from the roles that the document itself, its parent, and its relevant siblings define,
then derive content, representation, structure, rule-altitude, and document-to-document
relationship judgments from that identity. For every change that alters existence, location,
boundary, state, or content, check whether the existing identity is still valid and whether
related documents' role definitions and references must change along with it. Every axis, loop,
gate, and reference below is a tool that executes this foundation for a given situation; when
another rule conflicts with an identity judgment, the identity judgment wins.

The settled authority of a parent or sibling is a default constraint, not an axiom. On conflict,
revise or reject the subordinate proposal, or — with sufficient evidence and the owning party's
change process — escalate it as a candidate change to the higher-level document.

**Master invariant — correctness is restoration, not form.** There is no universal correct
compression level, representation, or structural relationship for the same content. Identity
decides what is right. There is exactly one criterion that holds across every form: **does the
chosen form let the reader restore the judgment identity requires?** Only among forms that pass
this test do you pick the one with the lowest reader cost. "Shortest" is a tiebreaker after the
restoration test passes, not the primary criterion.

Scope:

- A portable procedure for judging the creation, deletion, and review of durable documents;
  owner/scope placement; concept location; authoring-guideline and template structure; and
  information representation form.
- It does not own a specific project's taxonomy, document formats, current work, record locations,
  or closeout/owner handoff. It judges under whatever contracts and change processes the project
  specifies.
- User intent is important evidence but not an automatic answer. When it conflicts with a
  confirmed contract or corpus, report it with facts, inferences, opinions, and proposals
  distinguished from each other.

## Derivation Axes

Five axes derive from identity. No axis fixes a correct form in advance; each judges whether a
given form restores the judgment identity requires. Read each axis's detailed cases, smells, and
format lists via `Reference Routing`, and only when needed — before adding an item to a list,
first check whether it is a recurring pattern that the existing axes cannot restore. Do not add
simple variants.

- **Content and compression** — carry only as much as restores the judgment identity requires.
  Identity sets the compression level (whether it stops at the action level, the event level, or
  reaches down to values), and no single level is universally correct. Invariant: if removing an
  element leaves the judgment unchanged, it is excess; if removing it lets a reasonable agent land
  on a different target, judgment, or action, it is deficient. A sentence is not independently
  sufficient or excessive. First restore the judgment unit that the surrounding context and the
  document's identity require, then check whether the sentence is enough to make that judgment.
  The fix is never simply compressing further — it's matching the density that keeps the required
  judgment intact. A sentence that hands off to action (routing, a gate's terminal step) is
  deficient if it stops at a state or label instead of naming the required action, since the reader
  would then have to infer the action. If another section owns the action, name that owner too.
- **Representation** — choose the form based on "what kind of information this is." When reader
  behavior requires repetition, re-entry, verification, or termination — as with a procedure,
  workflow, or review flow — consider a gate/flow/loop form instead of a plain list. It is not a
  well-formed loop unless it also restores a path back and a termination condition. Concrete form
  mapping and loop-component judgments are owned by `Information Format` and `Rule Shape`.
- **Structural relationship** — identity determines whether the relationship is a
  document>section>sentence>word hierarchy, a document<>section cross-reference, linear, or
  asymmetric. There is no universal form, so compare symmetric/hierarchical structure as the
  initial candidate, but use an asymmetric or reciprocal relationship when identity requires it.
- **Rule altitude** — every rule, restriction, or "thing to know" has an altitude: it applies
  globally across the whole document, or only locally within a specific section. Place each rule at
  its altitude, and let the reader restore which altitude a given rule sits at. Do not place a
  local rule as if it were global, or a global rule as if it were local.
- **Document-to-document relationship** — the coupling between two documents typically takes forms
  such as separation of concerns, delegation, or coordinated complementarity (each holds the part
  it needs, with neither subordinate to the other). This relationship determines whether content
  stays in the current document, gets linked/delegated to the owner, or gets supplemented through a
  shared section. Do not assume ownership or inheritance from physical nesting, path, or a shared
  template alone.

## Review Loop

Document review and authoring is a loop that starts from the identity anchor and travels along the
change's blast radius. The frontier starts as the single target document. If it's unclear which
document that is — for example, choosing between an index and one of its slices — start from
whichever one the user pointed to or the one closest to the change; step 4 can still relocate the
change to a sibling.

### Node Operation (for each document in the frontier)

1. Recover the target document's intended identity and its grounds — including who currently owns
   it, as part of that identity. Find grounds in the user prompt, project/root contracts,
   parent/child guidelines, the owner map, relevant siblings, and the actual corpus. For a new
   document, design identity from user intent and the parent/owner map/siblings.
2. Gather the change surface from two inputs. One is an explicit change to existence, location,
   boundary, state, or content. The other is a point that surfaces when the document directs a
   reader's judgment or action and you actually execute or dry-run (simulate execution of) it. In
   the latter, if the reader cannot restore something identity requires — the next action, a
   branch landing point, a re-entry point, a termination condition, escalation, an owner, an
   evidentiary standard, or where a piece of content belongs and at what altitude — mark that point
   as a **restoration gap**. The core test isn't a list of examples but a single question: "if you
   follow it through, is the next judgment restored?" Judge the surface of both inputs across the
   five axes (content/compression / representation / structural relationship / rule altitude /
   document-to-document relationship), evaluating structure and content together rather than one
   before the other, and when one defect spans multiple axes, don't pin it to just one — route it
   to every axis it touches. Don't invent a new smell or structure model for a gap like this first;
   only promote it to a candidate portable pattern once the same gap recurs and the existing
   axes/references can't restore it.
3. Read only the related documents that could change the judgment. Do not expand to every
   ancestor, sibling, child, or the whole corpus out of plain uncertainty.
4. Decide which party is responsible for this change — the current owner recovered in step 1, or
   an ancestor if the change actually belongs there — and the change location, comparing
   local-only, sibling-only, ancestor-only, both, and no-change candidates. Do not assume the
   document under review is always the one to change.
5. Apply the `Change Gates` that match the change type.
6. Among candidates that satisfy both role fit and the necessary propagation, make the change with
   the smallest blast radius. Decide the content's location first, then choose the form via the
   representation axis, and among candidates that pass the restoration test, pick the one with the
   lowest reader cost. Brevity is only one factor in that judgment.
7. Verify whether this document's and its directly related documents' identity and references are
   still valid, and branch on the result:
   - **pass** → close this document's node and go to `Propagation`.
   - **local verify fail** (the recovered identity/owner still holds, but the change location,
     form, or owner candidate fails verification) → go back to 4 and re-derive only this document's
     change.
   - **identity-invalidation fail** (new evidence or a propagation result overturns this document's
     original identity/owner recovery itself) → go back to 1 and redo identity recovery from
     scratch.

### Propagation

- Compute the blast radius only from relationships the passed change could break: parent role
  definitions, the owner map/taxonomy, inbound references, sibling boundaries, and child
  skeletons/invariants. This is not an exhaustive review — surface only the relationships that
  could change a judgment.
- Add each document in the blast radius to the frontier and reapply Node Operation. If a frontier
  document changes again, add its blast radius too.

### Termination

- Convergence is when the frontier is empty and the final verify passes. The blast radius is
  usually small, so convergence typically happens in one pass.
- Every re-entry in the loop — the N7 back-edges (→4, →1) and Propagation's frontier re-addition —
  is only justified when it consumes an input that changes the next judgment, such as new
  evidence, a propagation result, or a verification-failure cause. If a re-entry point would
  reproduce the state it already produced (step 1 = identity/owner, step 4 = change
  location/form/owner candidate, Propagation = the same frontier document with no new blast)
  without new input, that's a cycle — hand it to escalation instead of taking the back-edge or
  re-addition.
- On non-convergence (cycling or divergence), a destructive change, or uncertain identity,
  boundary, or authority, check with the user instead of guessing.
- Do not turn "every review" into "traverse the whole document set." That itself is
  over-engineering.

### Meta Ring (skill self-improvement, conditional, non-blocking)

- Fires only when this review exposes a recurring pattern that the current axes, gates, or the
  smell/case catalogs (see Reference Routing below) cannot name. Otherwise, it does not fire.
- If it isn't recurring, park it. If it is, strip domain-specific terms (a specific repo, team, or
  log name) and check whether it's portable. If it isn't portable, it's a candidate project rule,
  not a skill change.
- Self-review (dogfood) any proposed skill change using this skill's own Review Loop and Change
  Gates.
- The Meta Ring stops at proposing a skill-improvement candidate. Since the skill is an owner
  document, do not auto-edit it — get user approval.

## Change Gates

Apply only the gate that matches the change type, at Node Operation step 5. A gate is an instance
of an axis applied to a specific kind of change.

### New Document

- First check whether an existing owner document or section already owns the same responsibility,
  and whether it can be absorbed into existing context.
- Create a new document only when an actual difference in owner, scope, reader, lifecycle, or
  change coupling requires an independent identity.
- Before creating it, decide the intended identity, owner/scope, and its parent/sibling
  relationships and placement. After creating it, confirm that the parent taxonomy, index, inbound
  references, and sibling boundaries correctly describe the new role.

### Content Or Identity Change

- If new content shares the existing identity's owner and scope, absorb it; if it's a subordinate
  concept, place it in the section it needs.
- If it exceeds the identity/scope, don't wedge it in locally. Compare delegating to an owner,
  expanding scope, splitting the document, and promoting it to a higher-level concept.
- Raise the evidentiary bar for moves, renames, splits, merges, and identity/scope changes, and do
  not break existing anchors or inbound references without an explicit migration.

### Destructive Change

- Before deleting, replacing, archiving, or transitioning status, check whether knowledge
  restorable only through this document — rationale, trade-offs, rejected alternatives,
  re-evaluation conditions — survives elsewhere, in another owner document or in code.
- Classify each reference as serving current authority/active work/implementation, or as
  provenance-only. When preservation or reference impact is uncertain, choose a non-destructive
  fallback — hold, reinforce, create a new document, or confirm with the user.

### Guideline Or Template Change

- Do not treat a default section or template as neutral. Add one only when an actually recurring
  subtype and a real omission risk are confirmed, and first check whether absorbing it into
  existing context or an invariant is the smaller fix.
- Check that a section or scope named more narrowly than its concept doesn't omit other subtypes
  or the no-subtype case. Before adding a new section, align a narrow identity with its concept
  through renaming, generalization, or scope redefinition.
- Cases and the corpus are inputs that verify a rule, not an automatic template.

## Reference Routing

Every reference is an optional tool. Read one only when it meets a condition that would change the
current judgment. Each reference's list is an instance of the five axes; add an item only when it
passes the recurrence gate (it actually recurs, and the existing axes cannot restore it).

- For classifying a structural problem, the context read path, concept placement, or an ambiguous
  owner/scope/change location, see `references/review-patterns.md`.
- For rule shape, wording, representation form, guideline/template structure, or comparing sibling
  guides, see `references/authoring-patterns.md`.
- When it's unclear how to apply the above criteria to a concrete scenario, see only the relevant
  case in `references/EXAMPLE.md`.

## Output Contract

Apply only the relevant items to the final proposal or revision.

- Did you distinguish confirmed facts, inferences, opinions, and proposals?
- Did you recover the target document's identity and its grounds?
- Did you derive the content, representation, structure, rule-altitude, and document-relationship
  judgments from that identity?
- Does every sentence that hands off to action let the reader restore the required action — and,
  if another section owns it, that owner too?
- When you dry-ran a document that directs a reader's judgment or action, did no restoration gap
  remain?
- Does the chosen form restore the judgment identity requires, and only then is reader cost the
  lowest? (Did you avoid shortening at the cost of restoration, and avoid leaving in content that
  doesn't change the judgment?)
- Is the owner, scope, and change location appropriate, and did you apply the necessary gate? Do
  the parent role definition, owner map, inbound references, sibling boundaries, and child
  skeletons/invariants all remain valid?
- Did you avoid guessing your way past uncertain identity, boundary, or authority — or a
  destructive change?
- Did the loop converge, or — if it didn't — did you escalate instead of closing out the
  non-convergence with a guess?
