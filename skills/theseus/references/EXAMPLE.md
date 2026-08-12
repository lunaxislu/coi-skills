# Theseus Examples

This document is a portable collection of cases showing the theseus skill's judgment patterns.
It does not preserve specific project names, repo paths, team names, or original logs. Cases are
abstracted just enough that the next author can restore the same structural judgment.

## Contents

1. [Misfiled Owner Map](#1-misfiled-owner-map)
2. [Evidence And Sources Are Siblings](#2-evidence-and-sources-are-siblings)
3. [Tool Guidance Belongs In Document Identity](#3-tool-guidance-belongs-in-document-identity)
4. [Index And Slice Need Different Rules](#4-index-and-slice-need-different-rules)
5. [Same Section Name, Different Drift Meaning](#5-same-section-name-different-drift-meaning)
6. [Example Leaks A Hidden Rule](#6-example-leaks-a-hidden-rule)
7. [Repeated Writing Problem Becomes Guideline Improvement](#7-repeated-writing-problem-becomes-guideline-improvement)
8. [Destructive Change Needs A Gate](#8-destructive-change-needs-a-gate)
9. [Ancestor Contract May Be The Defect](#9-ancestor-contract-may-be-the-defect)
10. [Over-provisioned Skeleton Section](#10-over-provisioned-skeleton-section)
11. [New Content Outgrows Document Scope](#11-new-content-outgrows-document-scope)
12. [New Document Needs An Independent Identity](#12-new-document-needs-an-independent-identity)
13. [Reader Path And Loop Are Misaligned](#13-reader-path-and-loop-are-misaligned)
14. [Gate Branch Ends In A Label](#14-gate-branch-ends-in-a-label)

## 1. Misfiled Owner Map

Situation:

A document that explains a document system's taxonomy and owner map sits under a specific leaf
document type. That leaf type's authoring rules, in turn, depend back on the taxonomy this
document describes.

Structure smell:

- `owner or altitude mismatch`
- `reverse or circular dependency`

Review:

The document's physical location doesn't match its conceptual hierarchy. A taxonomy or owner map
is a meta-level document that classifies and connects multiple leaf owners — it isn't a detail
document of one leaf type. Rather than adding an explanatory note to justify a structure that
looks circular, it's better to move the document to the appropriate parent scope and simplify the
dependency direction.

Preferred response:

- Judge the altitude of the knowledge the document actually owns.
- Decide whether it belongs under the leaf-type rules or under a parent/common scope.
- If a move is needed, repoint only the active inbound references and leave the historical record
  as history.
- Treat a plain explanatory note or excessive document splitting as a rejected alternative if it
  fails to remove the smell.

## 2. Evidence And Sources Are Siblings

Situation:

A draft put code/test evidence, provenance sources, and historical logs all under a single
Evidence section. As a result, provenance reads as if it were a subordinate concept of code/test
evidence.

Structure smell:

- `owner or altitude mismatch`

Review:

Evidence and Sources are close in that both belong to the traceability family, but neither is the
other's parent/child. Evidence can be code/test grounds that verify a current/as-built claim, while
Sources can be inputs for restoring the reasoning behind a decision or its provenance.

Preferred response:

- Instead of narrowing the section name to one concept, adjust it so the sibling relationship
  between the two is visible.
- Restrict the code/test-owner rule to the Evidence side's meaning.
- Do not copy provenance-owner or log-lifecycle rules at length into this section.
- Delete explanatory sentences that the rules and examples already say.

## 3. Tool Guidance Belongs In Document Identity

Situation:

An authoring-guide draft placed a separate "authoring tools" section explaining which syntax or
helper skill to use when. The content itself is necessary, but the tool-usage section stands out
against the document's overall flow.

Structure smell:

- `owner or altitude mismatch`

Review:

Even content that looks like tool usage is more natural to absorb into the introduction than to
keep as an independent section, if its real role is explaining the document's identity or edit
conditions. First state what the document family is and what format it's read and edited in, then
briefly link the tool reference that's needed under those conditions.

Preferred response:

- Check whether the standalone section is creating a wrong higher-level concept for the document.
- Merge the flow between document-family identity, editor/reader context, and tool reference into
  the introduction.
- Keep tool instructions from reading like authority content.
- If project rules and tool guidance conflict, state explicitly which one takes precedence.

## 4. Index And Slice Need Different Rules

Situation:

A large owner document was split into an index/hub and several slices. The authoring guide's
skeleton looks like it applies to every document, and it's ambiguous whether new body content
should go in the index or in a slice.

Structure smell:

- `hidden sub-kind`
- `duplicated ownership`

Review:

Even within one document family, there can be structural sub-kinds. An index/hub can own
navigation, the document map, stable anchors, vocabulary, and an overview, but full contracts or
body text are better owned by the owning slice.

Preferred response:

- State explicitly which sub-kind the skeleton targets.
- Keep only navigation entries and stubs in the index/hub, and send full body text to the slice.
- Preserve existing inbound anchors or numbered sections unless there's an explicit migration.
- Let split/index rules be owned by the relevant owner guideline rather than a generic parent
  guide.

## 5. Same Section Name, Different Drift Meaning

Situation:

Sibling authoring guides both cover handling drift. One document type describes the current
implementation; the other normatively defines an external contract or target behavior.

Structure smell:

- `copied section meaning`

Review:

Symmetry is only a baseline for section order and the review lens — the same heading doesn't
guarantee the same meaning. A descriptive/as-built document may be stale if it differs from the
code. A normative/target contract must not be overwritten with the current implementation just
because it differs from the code — the difference may be an implementation gap.

Preferred response:

- Compare sibling structure, but classify document role first.
- Even when the same drift heading is used, re-derive the staleness criteria to fit each document
  type.
- Do not apply the same update rule to current/as-built claims and normative/target claims.
- Where a difference is needed, name the content property that requires it.

## 6. Example Leaks A Hidden Rule

Situation:

An authoring guide's skeleton example shows formatting — link form, evidence indentation, status
placement — but the body rules never state that format's owner or scope of application.

Structure smell:

- `hidden or implicit authority`
- `orphan local rule`

Review:

An example is meant to support a rule. If a needed rule hides only in the example, the next author
will find it hard to restore whether it's a required form or an incidental detail. Conversely, if a
child guide redefines a format the parent guide already owns, that becomes duplicated ownership.

Preferred response:

- Check whether the example is quietly creating a new rule.
- If a rule is needed, surface it as a body rule in the owner document.
- Don't let a child redefine a format the parent owns — delegate instead, or make only the example
  conform.
- Wrap skeleton examples in a fenced block so they don't pollute the host document's outline.

## 7. Repeated Writing Problem Becomes Guideline Improvement

Situation:

The same owner-boundary confusion, section-placement problem, or skeleton-application problem
recurs every time an individual document is authored.

Review signal:

- Recurring owner-boundary/section-placement/skeleton-application confusion becomes a candidate
  for Guideline Improvement/the recurrence gate, beyond a local fix.

Review:

A recurring problem may not be a wording problem in one document — it may be a problem in the
authoring guide, template, or skeleton. If it's a one-off misapplication, fixing only the current
document can be enough. If the next author is likely to make the same mistake, treat it as a
candidate for a guideline improvement.

Preferred response:

- Distinguish a recurring problem from a one-off misapplication.
- Before changing the guideline, check whether relocation, condensing, or clarifying the owner
  boundary can resolve it.
- If a new rule is needed, pick the smallest rule shape among scope rule, identity rule, decision
  table, gate-driven flow, feedback loop, invariant, negative rule, must rule, and
  destructive-change gate.
- Leave project-specific record locations or workflow to that project's own rules.

## 8. Destructive Change Needs A Gate

Situation:

An authoring-guide draft explains the conditions for deleting an old owner document using only
"no longer important" and "no references." This document doesn't own the current state itself, but
it holds the trade-offs and rejected alternatives behind a past decision, so deleting it could make
it hard to restore why the current direction still holds. Search results mix references from the
active owner document with provenance references from historical logs.

Structure smell:

- `abstract destructive gate`
- `hidden or implicit authority`

Review:

Hard-to-reverse changes — deletion, replacement, archiving, status transitions — must not be
allowed through with abstract judgment words alone. First confirm what would be lost: does
knowledge restorable only through this document — rationale, trade-offs, rejected alternatives,
re-evaluation conditions — survive elsewhere, in another owner or in code? Next, classify the
nature of each reference: is it a reference that affects current judgment — the current owner,
active work, an entry/contract document, a code/test comment — or is it a provenance-only
log/archive reference?

Preferred response:

- Turn abstractions like "not important" into a knowledge-loss question.
- Distinguish active/current-impact references from provenance-only references.
- If a current-impact reference exists, require owner reinforcement or reference repointing before
  deletion.
- If anything is uncertain, don't delete — route it to a non-destructive fallback such as holding,
  user confirmation, owner reinforcement, or writing a new document.
- Let the relevant project's guideline fill in project-specific log status, plan status, or archive
  location names.

## 9. Ancestor Contract May Be The Defect

Situation:

The same owner-boundary problem keeps recurring every time a downstream authoring guide is fixed.
Fixing only the local document tidies the sentence for now, but the next agent follows the same
parent taxonomy or ancestor owner map and makes the same misjudgment again. The ancestor contract
has a large blast radius since multiple documents depend on it, so changing it right away is risky.

Structure smell:

- `owner or altitude mismatch`
- `orphan local rule`
- `contradiction`

Review:

A parent/ancestor contract is a default constraint, not an axiom. When a recurring structural smell
originates at an ancestor boundary, don't assume the local document is the one to change. Instead,
separate out `local-only`, `ancestor-only`, `both`, and `no-change` candidates. An
ancestor-change candidate affects downstream dependents, so it needs a higher evidentiary bar and
the owning party's change process.

Preferred response:

- First distinguish whether the same smell is a one-off or recurring.
- Compare how each candidate changes agent behavior in the next authoring scenario.
- You may suggest the ancestor is the problem, but don't let a downstream document quietly override
  the parent rule.
- If an ancestor change is needed, surface the affected dependents, the conflicting evidence, and
  whether approval/review is required.
- If the evidence is insufficient, choose a non-destructive fallback such as a local boundary note
  or a recorded no-change decision.

## 10. Over-provisioned Skeleton Section

Situation:

An authoring-guide draft provides a skeleton for new documents and includes every default section
— `Deferred`, `Non-Goals`, `Boundaries`, `Links`. The actual corpus has no recurring subtype that
always needs those sections filled in, and the user says "it seems like this section could be
dropped."

Structure smell:

- `over-provisioned structure`
- `copied section meaning`

Review:

A default section is a cost, not a neutral. If a section exists, an agent will try to fill the
blank, and that can pull in future content or content another owner should own into the current
document. The user's "it seems like this could be dropped" isn't mere preference — it can be a
signal that the skeleton is larger than actual authoring needs.

Preferred response:

- Check whether an actually recurring subtype exists.
- Compare whether the agent omits a required section without the skeleton, versus fills an
  unnecessary section with it.
- Evaluate each default section as a candidate for delete, absorb, optional, subtype-only, or
  invariant-only.
- Absorb any needed boundary into an existing section such as a higher-level `Scope`, and send
  deferred work or target detail to the document type that owns it.
- If a section contract, invariant, or golden document is a smaller fix than the whole skeleton,
  choose that instead.

## 11. New Content Outgrows Document Scope

Situation:

While authoring a document narrowly declared to cover topic A, content for a separate topic B
becomes necessary. The document's identity is A-only, so B gets awkwardly wedged in.

Structure smell:

- `owner or altitude mismatch`
- `under-scoped identity`
- `hidden sub-kind`

Review:

Before wedging B locally into the A document, first work upward to understand the relationship and
flow between A and B. The problem isn't where one sentence sits — it's whether the document's
identity can hold the new content at all. If B belongs to another owner, delegation is right; if A
and B are siblings under a common higher-level topic, redefine the document as the higher-level
concept with A and B underneath, or split it into two; if B is a natural extension of A, widen the
identity statement to include it.

Preferred response:

- Before a local fix, judge the relationship between the new and existing content (same owner,
  sibling, subordinate, or unrelated).
- Compare which of delegation, scope expansion, splitting, and promotion to a higher-level concept
  is the smallest fix.
- If it's low-frequency, one-off content, first check whether absorbing it into the body or
  dropping it is enough.
- If identity or scope changes, check the impact on inbound references and downstream documents,
  and preserve existing anchors.

## 12. New Document Needs An Independent Identity

Situation:

While trying to add a new topic to an existing owner document, the content is expected to grow
long, so a separate document is proposed. But the new topic shares the same owner and readers as
the existing document, changes together with it, and the parent taxonomy defines no separate
document type or sibling role for it.

Structure smell:

- `over-provisioned structure`
- `duplicated ownership`

Review:

Length alone does not create an independent identity. First check whether it can be absorbed into
existing context, and create a new document only when an actual difference in owner, scope,
reader, lifecycle, or change coupling requires an independent role. If a new document is needed,
since it has no body yet, design its intended identity from user intent, the parent, and relevant
siblings.

Preferred response:

- Check whether an existing owner or section already owns the same responsibility.
- Don't create a new document based on length, writing convenience, or future possibility alone.
- If an independent identity is needed, first decide the owner/scope and the parent/sibling
  relationships and placement.
- After creation, update the parent index or taxonomy, inbound references, and sibling boundaries
  together.
- If the content boundary can't be restored, don't create the document — check with the user.

## 13. Reader Path And Loop Are Misaligned

Situation:

An entry document exposes a working loop such as `Enter`, `Record`, `Reconcile`, `Closeout`, but the
target reference places its detailed rules under different-language titles, a different order, and
narrower headings. The reader either bounces across sections to finish one phase, or misses
follow-up actions — handoff, cleanup, verification — because a heading doesn't cover its actual
scope.

Structure smell:

- `under-scoped identity`
- `misaligned reader path`
- `excessive read path`

Review:

The problem may not be the absence of one more anchor. If the entry document sets the working loop
or reader path, the target reference's headings, order, and section scope need to be able to
receive that same unit of work. A title isn't just style — it can be the interface through which the
reader finds the next action. Before adding a local rule, check whether renaming a section,
reordering, or adjusting the loop map or routing surface can align the document boundaries instead.

Preferred response:

- Check the phases, steps, commands, or template units the entry document exposes.
- Cross-check whether the target document's headings and order provide the same units of work and
  scan path.
- If a section name doesn't cover its actual scope, either shrink the content or widen the name.
- Don't force a non-phase file skeleton or common substrate into a phase — instead explain in the
  loop map or introduction which phase references it.
- Before adding a new cross-reference rule, compare whether adjusting document boundaries,
  headings, order, or the routing surface is the smaller fix.

## 14. Gate Branch Ends In A Label

Situation:

An entry document judges cases through an ordered gate, and some branch outcomes end with a
state/label statement such as "this case is superseded." The actual transition/record/verification
action is owned by another section, but the branch doesn't name it, so the reader has to infer which
section to apply. A different draft does the opposite — it repeats a concrete procedure at every
branch, creating a downstream section with dual ownership.

Structure smell / axis signal:

- Content/compression restoration failure — a branch outcome stops at a state/label instead of the
  required action (formerly `inferred action`; now owned by the SKILL body's content/compression
  axis and the Output Contract)
- `duplicated ownership`

Review:

A branch outcome is a sentence that hands off to action. If it states only a state or label, the
reader has to infer the follow-up action; if a concrete procedure is copied into every branch, the
same knowledge ends up with two owners. Write each branch as an instruction naming the required
action and the section that owns it, and let the owner section be the single owner of the concrete
transition/verification steps.

Preferred response:

- Classify whether a branch outcome sentence is an action instruction or a state/label statement.
- Rewrite each branch as an instruction naming the required action and its owner section, parallel
  in form across branches.
- Give sole ownership of concrete procedures and verification steps to the owner section; don't
  repeat them in the branches.
- Even a terminal branch that does nothing should state that termination explicitly, so nothing is
  left to inference.
