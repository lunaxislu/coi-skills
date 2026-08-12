# coi-skills

Portable [Agent Skills](https://agentskills.io/specification) — reusable across Claude Code,
Codex, Cursor, and any other tool that supports the `SKILL.md` spec.

## Skills

### [design-grounded-changes](skills/design-grounded-changes/SKILL.md)

Design, implement, or review software and documentation changes from observed needs and existing
system capabilities. Use for features, defects, hardening, refactors, operations, and
documentation when the agent must establish the real outcome, trace current responsibilities and
flows, distinguish evidence from assumptions, evaluate built-ins/frameworks/installed libraries
before new machinery, keep scope YAGNI, and verify the resulting behavior.

### [theseus](skills/theseus/SKILL.md)

Create, delete, structurally review, design, or refine durable documents, documentation rules,
authoring guides, templates, skeletons, or review feedback. Recovers a document's identity — what
it represents — from the document and its parent/sibling relationships, then derives content,
representation, structure, rule-altitude, and document-relation judgments from that identity
through a review loop that propagates only along a change's blast radius.

## Install

### Via `npx skills` (any supported agent)

[`npx skills`](https://github.com/vercel-labs/skills) fetches `SKILL.md` files directly from this
repo and installs them into whichever agent directory applies.

```bash
npx skills add lunaxislu/coi-skills                                     # pick interactively
npx skills add lunaxislu/coi-skills --skill design-grounded-changes -a claude-code
npx skills add lunaxislu/coi-skills --skill theseus -a claude-code
npx skills add lunaxislu/coi-skills --all                               # install everything detected
```

### Via Claude Code plugin marketplace

Each skill is also packaged as an independent single-skill Claude Code plugin under this repo's
marketplace.

```bash
claude plugin marketplace add lunaxislu/coi-skills
claude plugin install design-grounded-changes@coi-skills
claude plugin install theseus@coi-skills
```

## License

[MIT](LICENSE)
