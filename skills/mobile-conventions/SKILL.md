---
name: mobile-conventions
description: Conventions for Android, iOS, Flutter, and Kotlin Multiplatform apps, covering consistency, naming, feature boundaries, structure, layering, state, navigation, dependency injection, persistence, errors, lint, testing, verification, accessibility, and project documents, including the README. Use when changing, reviewing, or planning work in a mobile codebase.
---

# Mobile app conventions

Four stacks, one spine: Android (Kotlin + Compose), iOS (Swift + SwiftUI), Flutter (Dart), and Kotlin Multiplatform (shared Kotlin, platform UI). Each stack reference supplies the idiom for these shared rules.

The **seam** is the organising idea: every UI/logic/data/platform boundary must be a cut you can substitute. Keep seams narrow and test at them. Most mobile design bugs are a missing seam (UI reaching into the database) or a leaky one (a DTO in a composable).

## Identify the stack

Read the matching reference before proposing structure or APIs:

| Stack | Signals | Read |
| --- | --- | --- |
| Android | `build.gradle.kts`, `AndroidManifest.xml`, `@Composable` | [`reference/android.md`](reference/android.md) |
| iOS | `Package.swift`, `*.xcodeproj`, `import SwiftUI` | [`reference/ios.md`](reference/ios.md) |
| Flutter | `pubspec.yaml`, `lib/main.dart`, `package:flutter` imports | [`reference/flutter.md`](reference/flutter.md) |
| Kotlin Multiplatform | `commonMain`, `expect`/`actual`, `kotlin { }`, or `project.yaml`/`module.yaml` | [`reference/kmp.md`](reference/kmp.md) |

KMP has a platform UI too: read the KMP reference, then the Android or iOS reference for the UI layer. When the stack is ambiguous, the build file decides.

Reviewing rather than building? Walk [`reference/review-checklist.md`](reference/review-checklist.md) after the stack reference.

## Language and framework skills

Before writing code, check the available skills for the detected language, SDK, or framework and load the relevant one; use it as the source of truth for current syntax and idioms. Do this without being asked. This skill owns architecture and process, not a versioned feature list. If no skill exists, consult the official docs for the project's language version.

## Universal rules

1. **Layer by knowledge, not file type.** UI renders and forwards intent; logic owns rules and state; data owns I/O. DTOs, rows, and JSON become domain models at the data seam. A type importing both a UI framework and a network client marks a crossed seam.
2. **One source of truth per state.** State lives in logic; UI renders a snapshot. Exclusive states are one type, not parallel flags; derive values at read time, never store a second copy that can drift.
3. **Flow one way.** Events down, state up. Siblings never call each other; route through the owner.
4. **Inject at the edge.** Build dependencies at the entry point; logic depends on interfaces, not globals. A static accessor hides the seam and leaks state between tests.
5. **Failure is a value.** Data returns a result or typed error; logic maps it to UI state; UI renders it. Exceptions are programmer error.
6. **Navigation is data.** Typed routes carry identifiers, not strings or object graphs built at call sites, so deep links and back stacks stay testable.
7. **Platform owns platform UX.** Back, permissions, keyboard, safe areas, and share sheets follow the host, even when shared code could imitate them. A foreign back button is noticed faster than a shared abstraction saves code.
8. **Accessibility is done.** Label interactive elements, give 44–48 dp/pt targets, and survive the largest text size; verify with the platform inspector.
9. **Test at the seam.** Unit-test logic with fakes, contract-test data against a fake server or database, and keep one or two end-to-end UI flows per feature. Snapshots belong to the design system, not every screen.
10. **Format and lint are automated.** The formatter owns layout and the linter owns rules, so review attention goes to design. Wire both into build and CI; fix a wrong rule, not the call site.
11. **Plan first, confirm direction, move docs with code.** Every feature, bug, refactor, or post-scaffold implementation gets a `docs/plan.md` entry; present it and get approval before code; confirm direction after project creation. `docs/architecture.md` and `README.md` move in the same atomic commit as shape, dependencies, commands, or behavior. Docs disagreeing means not done.
12. **Match structure; write added code in the current idiom.** Follow existing naming, structure, architecture, and API choices unless told otherwise. For every line you add, use the newest syntax the project's language version supports, including syntactic sugar; newer syntax is not a "new API" tradeoff because it changes expression, not behavior. Prefer the simplest established approach for logic, error handling, and dependencies, not for syntax. This covers all added code, not only copied snippets; do this without being asked. Edited code keeps its surrounding style unless the edit already touches the construct; deleted code is not modernized. Check the language/SDK version and use any official language or framework skill available; it is authoritative for the current form. The project's language version is the ceiling: use the newest syntax it supports, never an older form than necessary, and do not propose an upgrade just to use newer syntax. If the current form cannot be used, mark `TODO` or record why. Comment unhandled errors and edge cases (what fails, why); comments state intent, not a transcript. A reader should not have to simulate the code to find them.
13. **Verify in proportion.** Inner loop: compile or analyze only what changed, plus one filtered test, so it costs seconds. Gate: format check, lint, full tests, and build once per batch, not per file. Keep the daemon warm, batch edits, skip the gate for prose. Probe SDK, JDK, and tool versions before build config; never provision an emulator, runtime, or large SDK without asking. If the gate runs more than a handful of times for one change, the loop is too expensive; fix it. Commands: stack reference.
14. **Keep the generator's toolchain and dependencies.** Keep the provisioned SDK, platform target, compiler, build tool, and dependency versions across all variants; never downgrade for a tool, linter, or library. Surface a conflict as a decision (tool/package, required versions, provisions replaced); the user chooses upgrade/replace, omit, or accept downgrade. Never silently pin older versions or suppress to hide skew. A silent downgrade forks the project from its generator. Record it in `docs/decisions.md`; repeat unresolved conflicts in the result.
15. **Commit small, atomic, and often.** Small commits keep rollbacks safe and history readable. Target ±200 lines, ±500 max; split larger as `summary (1/x)`. Exceptions: fresh scaffolds (commit intact) and inseparable changes.
16. **Read intent, not just words.** Names, flows, and steps are sketches unless the user says they are fixed. Understand the problem and outcome first, then plan the shape that serves them; if the literal request yields a worse design, follow intent and say what and why.
17. **Name for what it is.** Use the domain term for every feature, module, type, and variable; prompt wording is a hint. Overstated, understated, or wrong names are defects that outlive the task; rename before they spread into routes, tests, and docs.
18. **Features and modules are independent.** Depend on core utilities, repositories, shared contracts, and the design system, never another feature. No shared screens or view models. A boundary you cannot cut is one you cannot test or replace.
19. **Siblings through contracts, not imports.** Repository for data, service for behavior; contract in shared code, the consumer depends on it, and the provider implements it at the composition root; feature A never names feature B.
20. **Side effects with the caller.** Navigation and sharing are callbacks (`onOpenB: () -> Unit`), not calls the screen makes. The screen stays unaware of feature B, so it can be tested, kept, or replaced alone.
21. **Interfaces for everything injected.** Feature entry points and constructors take interfaces or function types for repositories, services, data sources, platform and navigation adapters, and features; build the concrete instance once at the edge. A test can pass a fake, and another build can pass a different implementation. A test needing a real database, network, or navigator has a leaked seam.

## Rule conflicts

When rules compete, apply them in this order: user intent, correctness, seams and architecture, current language idiom for added code, local consistency. Local consistency never licenses superseded syntax in new code. If a rule cannot be met, name it and say why.

## Shared or platform code

For KMP and Flutter, decide what crosses the boundary before writing:

- Share domain models, validation, use cases, repository interfaces, and data contracts: anything without UI.
- Keep UI, navigation, permissions, and platform services native unless shared UI (Compose Multiplatform, Flutter) is an explicit product choice.
- Share a feature only when two platforms ship it; expose state and intent, never widgets or view controllers.

## Project documents

Keep each file to its job; link instead of copying.

| File | Holds | Update when |
| --- | --- | --- |
| `docs/decisions.md` | request, agreed choices, rationale, short architecture summary | a choice changes |
| `docs/architecture.md` | modules, seams, dependency direction, libraries and versions | the shape changes |
| `docs/plan.md` | release/goal checklists; one entry per feature or bug with requirements, steps, and status | work is planned, started, or finished |
| `README.md` | purpose, build/run/test commands, layout | behavior, setup, commands, or structure changes |

A feature, bug, or refactor gets a `docs/plan.md` entry before code. Architecture and README move with the code; a change is not done while they disagree. See [`reference/planning.md`](reference/planning.md) for the entry template, sync matrix, and intake questions, including when to propose a smaller approach.

`AGENTS.md` is the entry point: it points an agent at these files and this skill, and stays short enough to read first.

## Applying

1. **Name the stack.** Read the matching reference above.
2. **Record the work.** A feature or bug gets a `docs/plan.md` entry; a choice goes to `docs/decisions.md`. State intent, names, any departure from prompt wording, and the doc updates required. See [`reference/planning.md`](reference/planning.md).
3. **Place the change.** Name its layer and seam; if you cannot, the design is not settled.
4. **Write in the current idiom.** Check the language/SDK version and use any official stack skill; use the newest syntax that version supports for every line you add. Do not copy older surrounding syntax into new code.
5. **Check the seam.** UI importing data, logic importing UI, shared importing platform, or feature importing feature means move the dependency behind an interface.
6. **Sync the documents.** Update architecture and README in the same change. Done when plan, architecture, README, and decisions agree.
7. **Verify.** Stack inner loop, then one gate per batch; rule 13 sets the budget.
