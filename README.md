# coi-skills

Reusable skills following the [Agent Skills](https://agentskills.io/specification) standard.
Claude Code, Codex, Cursor, and other compatible agents use the same `skills/<name>/SKILL.md`.
Claude/Codex plugins distribute the collection; `npx skills` also installs individual skills.

## Skills

### [design-grounded-changes](skills/design-grounded-changes/SKILL.md)

Design, implement, or review software and documentation changes from observed needs and existing
system capabilities. For features, defects, hardening, refactors, operations, and documentation,
establish the real outcome and current responsibilities/flows, distinguish evidence from assumptions,
and evaluate built-ins, frameworks, and installed libraries before new machinery. Keep scope YAGNI
and verify the resulting behavior.

### [theseus](skills/theseus/SKILL.md)

Create, delete, structurally review, design, or refine durable documents, documentation rules,
authoring guides, templates, skeletons, and review feedback. Recover identity from the document and
its parent/sibling relationships, then evaluate content, representation, structure, rule altitude,
and document relationships, reviewing along the change's blast radius.

### [execution-flow-map](skills/execution-flow-map/SKILL.md)

Map execution order, branches, exits, state mutations, and side effects from actual code into an
ASCII Execution Flow Map. Use for functions, CLI commands, APIs, daemons, workers/jobs, event
handlers, and page interactions/lifecycles. Bug diagnosis, quality assessment, and fixes are
separate tasks.

[Detailed usage guide](skills/execution-flow-map/README.md)

### [flow-notes](skills/flow-notes/SKILL.md)

Explain features, events, lifecycles, protocols, and architectures through engineering notes
centered on execution order, actors, data transfer, state changes, and causality. Also use to
organize established system behavior from supplied conversations, code, logs, and documents.

### [tech-lab](skills/tech-lab/SKILL.md)

Help users learn technology by writing and running minimal code and observing concrete evidence.
Supports first-time learning, individual APIs, continuation, confusion resolution, and explicit
`quick:` / `간단히:` depth. Keep user background and the current project in
`references/user-context.md`.

## Install

### Individual skills — `npx skills`

[Vercel Labs skills](https://github.com/vercel-labs/skills) discovers skills in this repository
and installs them into the selected agent's location. Plugin installation is not required.

```bash
npx skills add lunaxislu/coi-skills
npx skills add lunaxislu/coi-skills --skill execution-flow-map -a claude-code
npx skills add lunaxislu/coi-skills --skill execution-flow-map -a codex
npx skills add lunaxislu/coi-skills --skill design-grounded-changes -a claude-code
npx skills add lunaxislu/coi-skills --skill theseus -a codex
npx skills add lunaxislu/coi-skills --all
```

Installation is project-scoped by default. Add `-g` for global installation.

### Claude Code plugin — all skills

```bash
claude plugin marketplace add lunaxislu/coi-skills
claude plugin install coi-skills@coi-skills
```

Existing individual plugin paths `design-grounded-changes@coi-skills` and `theseus@coi-skills`
remain available. Choose either the collection or individual installation to avoid duplicate
skill entries. [Claude plugin documentation](https://code.claude.com/docs/en/plugins-reference)

### Codex plugin — all skills

```bash
codex plugin marketplace add lunaxislu/coi-skills
codex plugin add coi-skills@coi-skills
```

Use a Codex CLI supporting `plugin marketplace` and `plugin add`. Start a new conversation after
installation. This adds the repository's marketplace directly; it does not imply review or listing
in an official directory. [Codex plugin documentation](https://learn.chatgpt.com/docs/build-plugins)

## Maintenance layout

All three installation routes share `skills/`. `.claude-plugin/plugin.json` and
`.codex-plugin/plugin.json` define the root collection plugins. Their marketplaces are
`.claude-plugin/marketplace.json` and `.agents/plugins/marketplace.json`. No hooks, MCP, or build step
are needed.

Edit Korean `.ko.md` originals first, then translate and synchronize the English `.md` files.
Local Git exclusion rules keep `.ko.md` files out of distribution commits.

## License

[MIT](LICENSE)
