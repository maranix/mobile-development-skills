# Planning and project documents

A README and a plan are the baseline; each document answers a different question, so link between them instead of copying. Decisions are the only doc that keeps a chronological trail; architecture, design, plan, and README are current-state.

| File | Answers | Holds |
| --- | --- | --- |
| `docs/decisions.md` | Why is it like this? | Durable choices, one short entry each: date, status, choice, reason, rejected alternative. |
| `docs/architecture.md` | How is it built? | Current module and feature tree, seams, dependency direction, libraries and reasons, platform split, commands. |
| `docs/plan.md` | What are we building next? | Only active and next work; one short entry per open item. Completed and dropped work is removed. |
| `README.md` | How do I use it? | Current purpose, platforms, prerequisites, setup, commands, layout, and license. |

A small app may stop at the README and plan. Add `docs/decisions.md` at the first durable choice and `docs/architecture.md` when there is more than one module, a real seam, or a non-obvious dependency direction. Do not create a document with nothing durable to say; a scaffold usually has enough choices and shape to fill all four.

A `docs/design.md` or per-feature design doc follows the architecture rule: current state, edited in place. Do not create one for a small change.

## Monorepos and packages

- Root docs own the overall shape and cross-package decisions; package READMEs own install, usage, and package-level commands.
- Package changelogs own release history. A changelog records releases, not decisions or plans, and exists only when the project publishes.
- Do not repeat the architecture per package or copy the root README into a package.

## Reconcile before you start

Existing docs are not automatically true. If they are an archive or contradict the code, fix them first in one bounded pass:

- Collapse `## Done` to one line per item, or delete the list when the release shipped.
- Mark each replaced decision `superseded` with a one-line pointer, or delete it.
- Delete plan entries, decisions, architecture sections, and README text for work that no longer exists. Delete only when the code confirms the work is gone; keep a decision while its choice still governs the code, even if the entry is old.
- Fix every file, class, route, and command the docs name that the tree does not contain.
- Move unshipped capabilities out of the README and into the plan.
- Update `AGENTS.md` if it restates an old policy; keep it to stack, commands, and doc locations.

Do not rewrite prose that is still true or improve its style; stop when the docs match the code. This is an explicit exception to surgical edits: one docs-only commit, split by file if it exceeds about ±300 lines.

## Prune before adding

- Start every update by deleting what the change made false: removed features, renamed modules, dropped plan items, superseded designs.
- Edit the affected section in place. Do not append a new section below the old one; replace the old text instead of stacking a new version on top of it.
- Do not create a new section, table, or doc for a one-line change. Change the existing line; add it once if it is missing.
- Never duplicate a fact across docs. Each change has one owner: the plan holds the goal and acceptance, the decision holds the choice and reason, architecture holds the shape, the README holds usage. Link to the owner instead of restating it.
- Deleting work is not an event. Remove its plan item, decision traces, architecture section, and README text; never add a "removed" entry.
- No inventories: no test counts, line counts, or file lists. They rot and tell a reader nothing they cannot see in the tree.
- Git history is the archive. Do not keep full `cancelled`, `done`, or `superseded` entries; one line is the maximum.

## Decisions

- One entry per durable choice, newest last, at most six lines: date, status (proposed, agreed, superseded), the choice, one-sentence reason, and the rejected alternative.
- Record only choices that are expensive to reverse: architecture, contracts, dependencies, toolchain, data ownership. A rename or local implementation detail is not a decision.
- When a choice changes, replace the old entry with one line (`Superseded by <date>: <new choice>`) or delete it if it has no lasting value. Keep the current entry full; mark the old entry `superseded` in the same change that replaces it. Two entries that cannot both be true is a defect.
- Do not record implementation steps, file lists, or the diff.
- Keep architecture to a paragraph and link to `docs/architecture.md`.

## Architecture

- One page for most apps. Describe the current shape only: feature and module tree, the seam each layer crosses, dependency direction, and where the composition root sits.
- Cover shape, seams, dependency direction, libraries, and commands. One diagram at most; no release metadata, topics, changelogs, or file-by-file inventory.
- Name each library and why it exists; the build file owns versions. Record a version only when the doc is the only source of truth, such as a minimum OS or a pinned constraint with a reason. Delete a library's entry when the dependency is removed.
- For KMP and Flutter, show what is shared and what stays native.
- Link to the README for build, run, and test commands; record only commands specific to the architecture.
- Every path, type, route, and command named here must exist in the tree.
- No change history, `## Implementation`, step lists, or per-feature narrative. Rename in place; do not add a note that it was renamed.
- Update it in the same change that moves the code.

## Plan

The plan is a queue, not a history. Use `## Next`, `## In progress`, and optionally `## Done this release` with at most five one-line items, cleared at the release. Do not keep older releases or a full archive. The section is the status; do not repeat it in the entry, and never leave a checked box under `## Next` or `## In progress`.

Each open entry is at most ten lines:

```markdown
### Add offline workout sync

- **Release:** 1.2
- **Goal:** Sync completed workouts when online.
- **Requirements:** up to three bullets: must do, must not change, constraint.
- **Approach:** one or two sentences; link a decision for alternatives.
- **Sub-steps:** up to five checkboxes, only when the work spans commits.
- **Acceptance:** how to verify it is done.
```

Add a language level only when the work sets or raises it.

- No file lists, commit messages, per-step log, test counts, or full diff description.
- On completion, replace the entry with one line under `## Done this release`; clear that list at the release.
- On drop, delete the entry and its architecture, design, and README traces in the same change.
- A rename, dependency bump, or internal refactor gets no entry. A bug fix gets a one-line entry only when it needs reproduction and acceptance or spans commits.

## README

- Write for someone who has never seen the project: purpose, platforms, prerequisites, configuration, commands, layout, and license.
- Describe only shipped behavior, in the present tense. A capability that is planned belongs in `docs/plan.md`, not in the README as if it exists.
- Keep it current. Update only when behavior, setup, commands, or structure change.
- Link to `docs/architecture.md` for module detail instead of duplicating it.
- Every command, path, and feature named here must exist.
- Remove instructions and features that no longer exist; a stale README is worse than a short one.
- No changelog, decision log, or plan. Git history and `docs/` own those.

## Keeping documents in sync

Docs move with the code. The plan entry is only for work that needs tracking:

| Change | `docs/plan.md` | `docs/architecture.md` and design | `README.md` | `docs/decisions.md` |
| --- | --- | --- | --- | --- |
| Feature added | Short entry while open; one line when done | Edit the current tree and dependencies | Update behavior, setup, commands, layout | Add only if it changes a durable choice |
| Feature or work removed | Delete the entry; no done line or removal entry | Delete the section and its paths | Delete the text | Delete the entry if it exists |
| Bug fixed | Only if it needs reproduction and acceptance, or spans commits | Only if the fix moves a seam or dependency | Only if behavior, setup, or commands change | Only if it changes a durable choice |
| Rename or internal refactor | None | Rename in place and delete the old name | Update the name where it appears | None |
| Dependency added or removed | Only if it is feature work | Update the one-line reason; versions stay in the build file | Update prerequisites and setup | Only if the choice is not obvious |
| Command or tooling change | Only if it is feature work | Link to the README command or update an architecture-specific one | Update the command | Only if the choice is not obvious |

- **Delete first:** every doc update removes or rewrites the affected lines. An all-additive diff is a defect when the change removed, renamed, replaced, or completed work; a new feature adds lines but must not restate an existing fact.
- **Budget:** a rename or small fix touches at most about ten doc lines total; a feature at most about thirty; a scaffold one page per doc. Exceed it only for a real design change and say why.
- **Check the weight:** `git diff --stat -- README.md docs/` should be proportional to the code diff. If the docs grew faster, cut them.
- **Check the total:** if `README.md` plus `docs/` exceeds roughly a third of the source size, or has grown across several changes without a deletion, treat it as an archive. It is a signal to inspect, not a hard cap; a tiny project legitimately carries proportionally more docs.
- **Definition of done:** the plan contains only current work and at most five one-line done items; architecture, design, and README match the code; decisions record the durable choices; no document contradicts another; every path, type, route, and command named in a doc exists; no document describes removed work.
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
