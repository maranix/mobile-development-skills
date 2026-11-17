# Planning and project documents

Four documents record a project. Each answers a different question; link between them instead of copying. Decisions are the only doc that keeps a chronological trail; architecture, design, plan, and README are current-state.

| File | Answers | Holds |
| --- | --- | --- |
| `docs/decisions.md` | Why is it like this? | Durable choices, one short entry each: date, status, choice, reason, rejected alternative. |
| `docs/architecture.md` | How is it built? | Current module and feature tree, seams, dependency direction, libraries and versions, platform split, commands. |
| `docs/plan.md` | What are we building next? | Only active and next work; one short entry per open item. Completed and dropped work is removed. |
| `README.md` | How do I use it? | Current purpose, platforms, prerequisites, setup, commands, layout, and license. |

A `docs/design.md` or per-feature design doc follows the architecture rule: current state, edited in place. Do not create one for a small change.

## Prune before adding

- Start every update by deleting what the change made false: removed features, renamed modules, dropped plan items, superseded designs.
- Edit the affected section in place. Do not append a new section below the old one; a diff that is only additions is a log, not documentation.
- Do not create a new section, table, or doc for a one-line change. Change the existing line; add it once if it is missing.
- Never duplicate a fact across docs. Link to the owner.
- Git history is the archive. Do not keep full `cancelled`, `done`, or `superseded` entries; one line is the maximum.

## Decisions

- One entry per durable choice, newest last, at most six lines: date, status (proposed, agreed, superseded), the choice, one-sentence reason, and the rejected alternative.
- Record only choices that are expensive to reverse: architecture, contracts, dependencies, toolchain, data ownership. A rename or local implementation detail is not a decision.
- When a choice changes, replace the old entry with one line (`Superseded by <date>: <new choice>`) or delete it if it has no lasting value. Keep the current entry full; do not keep two full entries that conflict.
- Do not record implementation steps, file lists, or the diff.
- Keep architecture to a paragraph and link to `docs/architecture.md`.

## Architecture

- One page for most apps. Describe the current shape only: feature and module tree, the seam each layer crosses, dependency direction, and where the composition root sits.
- List libraries with versions and a one-line reason. Delete a library's entry when the dependency is removed.
- For KMP and Flutter, show what is shared and what stays native.
- Record the build, run, lint, and test commands.
- No change history, `## Implementation`, step lists, or per-feature narrative. Rename in place; do not add a note that it was renamed.
- Update it in the same change that moves the code.

## Plan

The plan is a queue, not a history. Use `## Next`, `## In progress`, and optionally `## Done this release`. Do not keep older releases or a full archive.

Each open entry is at most twelve lines:

```markdown
### Add offline workout sync

- **Status:** proposed | agreed | in progress
- **Release:** 1.2
- **Goal:** Sync completed workouts when online.
- **Requirements:** up to three bullets: must do, must not change, constraint.
- **Approach:** one or two sentences; link a decision for alternatives.
- **Language level:** the level and forms it enables, or `unchanged`.
- **Sub-steps:** up to five checkboxes, only when the work spans commits.
- **Acceptance:** how to verify it is done.
```

- No file lists, commit messages, per-step log, or full diff description.
- On completion, replace the entry with one line under `## Done this release`; clear that list at the release.
- On drop, delete the entry and its architecture, design, and README traces in the same change.
- A rename, dependency bump, or internal refactor gets no entry. A bug fix gets a one-line entry only when it needs reproduction and acceptance or spans commits.

## README

- Write for someone who has never seen the project: purpose, platforms, prerequisites, configuration, commands, layout, and license.
- Keep it current. Update only when behavior, setup, commands, or structure change.
- Link to `docs/architecture.md` for module detail instead of duplicating it.
- Remove instructions and features that no longer exist; a stale README is worse than a short one.
- No changelog, decision log, or plan. Git history and `docs/` own those.

## Keeping documents in sync

Docs move with the code. The plan entry is only for work that needs tracking:

| Change | `docs/plan.md` | `docs/architecture.md` and design | `README.md` | `docs/decisions.md` |
| --- | --- | --- | --- | --- |
| Feature added or removed | Short entry while open; one line when done; delete on drop | Edit the current tree and dependencies; remove the old feature | Update behavior, setup, commands, layout | Add only if it changes a durable choice |
| Bug fixed | Only if it needs reproduction and acceptance, or spans commits | Only if the fix moves a seam or dependency | Only if behavior, setup, or commands change | Only if it changes a durable choice |
| Rename or internal refactor | None | Rename in place and delete the old name | Update the name where it appears | None |
| Dependency added or removed | Only if it is feature work | Update the version and one-line reason | Update prerequisites and setup | Only if the choice is not obvious |
| Command or tooling change | Only if it is feature work | Update the command | Update the command | Only if the choice is not obvious |

- **Delete first:** every doc update removes or rewrites the affected lines. An all-additive doc diff is a defect.
- **Budget:** a rename or small fix touches at most about ten doc lines total; a feature at most about thirty; a scaffold one page per doc. Exceed it only for a real design change and say why.
- **Check the weight:** `git diff --stat -- README.md docs/` should be proportional to the code diff. If the docs grew faster, cut them.
- **Definition of done:** the plan contains only current work and the current release's one-line done list; architecture, design, and README match the code; decisions record the durable choices; no document contradicts another; no document describes removed work.
- **Prose-only changes need no code gate.** Check them against the code and links.
- **Stale docs are a bug.** Fix them in the same change or open a task before doing anything else.

## Feature and bug intake

Before writing an entry or code:

1. **Use a requirements skill if one is available.** Invoke it for the feature or bug. Otherwise ask the questions below.
2. **Understand the request.** What problem does it solve, who hits it, what happens today, what must not change, and how will success be measured?
3. **Look for a smaller path.** If an existing feature, a configuration change, or a different approach meets the need, propose it and say what it saves. If the user agrees, take that path and record it; otherwise keep gathering requirements and continue with the requested work.
4. **Agree before building.** Put a short entry in `docs/plan.md`: goal, up to three requirements, and acceptance. Add sub-steps only when the work spans commits. Present it and get approval. A rename or internal refactor goes straight to the code and the affected docs.
5. **Break it down into atomic commits.** Each sub-step is an atomic commit (under ±200 lines ideally, ±500 max). Commit often; split larger work as `summary (1/x)` unless tightly coupled or an initial scaffold. Record only durable choices in `docs/decisions.md`.
6. **Work the list and prune.** Check off sub-steps as they land. Update architecture, design, and README in place whenever the shape, dependencies, commands, or user-facing behavior move. On completion, collapse the entry; on drop, delete it and its traces.

Do not start implementation while the goal or acceptance is unclear; ask instead of guessing.
