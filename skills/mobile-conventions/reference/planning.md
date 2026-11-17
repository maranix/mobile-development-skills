# Planning and project documents

Four documents record a project. Each answers a different question; link between them instead of copying.

| File | Answers | Holds |
| --- | --- | --- |
| `docs/decisions.md` | Why is it like this? | The request, the agreed choices, the alternatives rejected, and the date. A short architecture summary, not the detail. |
| `docs/architecture.md` | How is it built? | Module and feature tree, layer seams, dependency direction, libraries and versions, platform split, and the build, run, lint, and test commands. |
| `docs/plan.md` | What are we building next? | Release or goal checklists, and one entry per feature, bug, or refactor with requirements, sub-steps, and status. |
| `README.md` | How do I use it? | What the app does, the platforms it targets, prerequisites, configuration, the commands to build, run, and test, the folder layout, and the license. |

## Decisions

- One entry per decision, newest last: date, status (proposed, agreed, superseded), the request in a sentence, the choice, and the reason.
- Record the alternatives that were rejected and why, so the next person does not reopen them.
- When a choice changes, add a new entry and mark the old one superseded; do not rewrite history.
- Record any toolchain or dependency deviation as a decision: name the tool/package, versions involved, and user's choice. Silent downgrades and suppressions hiding version skew are defects.
- Keep the architecture to a paragraph and link to `docs/architecture.md` for the detail.

## Architecture

- Describe the current shape, not the history: feature and module tree, the seam each layer crosses, dependency direction, and where the composition root sits.
- List libraries with versions and the reason each is present. A dependency without a reason is a candidate for removal.
- For KMP and Flutter, show what is shared and what stays native.
- Record the build, run, lint, and test commands.
- Update it in the same change that moves the code.

## Plan

Organize by release (`## 1.2`, `## Next`) or by goal when the project does not version. Keep completed entries for history; do not delete them.

Each entry:

```markdown
### Add offline workout sync

- **Status:** proposed | agreed | in progress | done
- **Release:** 1.2
- **Goal:** Sync completed workouts when the device is online.
- **Requirements:** what it must do, what must not change, and any constraint.
- **Approach:** the change, in a sentence or two.
- **Language level:** the build file's level and the forms it enables, or `unchanged`.
- **Alternatives:** what else was considered and why it was rejected.
- **Sub-steps:**
  - [ ] ...
- **Acceptance:** how to verify it is done.
- **Docs:** the `docs/architecture.md` and `README.md` sections this entry changes, or `none`.
- **Links:** the decisions entry and the architecture section it touches.
```

## README

- Write for someone who has never seen the project: purpose, platforms, prerequisites, configuration, commands, layout, and license.
- Keep it current. A feature that changes what the app does, a dependency that changes setup, or a command that changes how it is run updates the README in the same change.
- Write the commands and versions after the build stops changing; a prose fix should not force another verification cycle.
- Link to `docs/architecture.md` for module detail instead of duplicating it.
- Remove instructions and features that no longer exist; a stale README is worse than a short one.

## Keeping documents in sync

The plan entry comes first. No feature, bug, or refactor starts without one, and the documents move with the code:

| Change | `docs/plan.md` | `docs/architecture.md` | `README.md` | `docs/decisions.md` |
| --- | --- | --- | --- | --- |
| Feature added or removed | Add or update the entry; mark removed features done | Update the feature tree, modules, and dependencies | Update what the app does, setup, commands, and layout | Add an entry when the feature changes a choice |
| Bug fixed | Add the reproduction, expected behavior, fix, and acceptance | Update only if the fix moves a seam or dependency | Update only if behavior, setup, or commands change | Add an entry when the fix changes a choice |
| Refactor | Add the entry and sub-steps | Update when modules, seams, or dependencies move | Update only if structure or commands change | Add an entry when the refactor changes a choice |
| Dependency added or removed | Add the entry if it is feature work | Record the library, version, and reason | Update prerequisites and setup | Add an entry when the choice is not obvious |
| Command or tooling change | Add the entry if it is feature work | Record the commands | Record the commands | Add an entry when the choice is not obvious |

- **Definition of done:** the plan entry is `done` and its sub-steps are checked; `docs/architecture.md` matches the code; `README.md` builds and runs from a clean checkout; `docs/decisions.md` records every choice; no document contradicts another.
- **Removed features:** keep the plan entry for history, mark it done, remove the feature from the architecture and README, and record the decision if the removal was a choice.
- **Prose-only changes need no code gate.** A README, decisions, architecture, or plan edit is checked against the code and its links; it does not need a build, lint, or test run. Batch document edits and verify them once.
- **Stale docs are a bug.** If code and documents disagree, fix them in the same change or open a plan entry before doing anything else.

## Feature and bug intake

Before writing an entry or code:

1. **Use a requirements skill if one is available.** Invoke it for the feature or bug. Otherwise ask the questions below.
2. **Understand the request.** What problem does it solve, who hits it, what happens today, what must not change, and how will success be measured?
3. **Look for a smaller path.** If an existing feature, a configuration change, or a different approach meets the need, propose it and say what it saves. If the user agrees, take that path and record it; otherwise keep gathering requirements and continue with the requested work.
4. **Agree before building.** Put the entry in `docs/plan.md` with requirements, approach, alternatives, sub-steps, acceptance, and the docs it will update. A bug entry also records reproduction and expected behavior. Present the plan and get user approval before writing code. After project creation, ask whether to one-shot the scaffold, configure dependencies, or build a specific feature first.
5. **Break it down into atomic commits.** Each sub-step is an atomic commit (under ±200 lines ideally, ±500 max). Commit often during features and fixes; if a change exceeds this limit, split sequentially as `summary (1/x)` unless tightly coupled or an initial scaffold. Record decisions in `docs/decisions.md`.
6. **Work the list.** Update status and check off sub-steps as they land. Update `docs/architecture.md` and `README.md` in the same change whenever the shape, dependencies, commands, or user-facing behavior move. A change is not done while the documents disagree.

Do not start implementation while the goal or acceptance is unclear; ask instead of guessing.
