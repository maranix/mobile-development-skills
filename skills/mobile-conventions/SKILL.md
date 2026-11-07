---
name: mobile-conventions
description: Conventions for Android, iOS, Flutter, and Kotlin Multiplatform apps, covering consistency, naming, feature boundaries, structure, layering, state, navigation, dependency injection, persistence, errors, lint, testing, verification, accessibility, and project documents, including the README. Use when changing, reviewing, or planning work in a mobile codebase.
---

# Mobile app conventions

Four stacks share one spine: Android (Kotlin + Compose), iOS (Swift + SwiftUI), Flutter (Dart), and Kotlin Multiplatform (shared Kotlin, platform UI). The rules below hold on all four; each stack reference supplies the idiom that implements them.

The organising idea is the **seam**: every boundary between UI, logic, data, and platform code is a cut you can make and substitute. Keep each seam narrow, and test at the seam instead of through the whole app. Most mobile design mistakes are a missing seam (UI reaching into the database) or a leaky one (a DTO surfacing in a composable).

## Identify the stack

Read the reference for the stack in play before proposing structure or APIs:

| Stack | Signals | Read |
| --- | --- | --- |
| Android | `build.gradle.kts`, `AndroidManifest.xml`, `@Composable` | [`reference/android.md`](reference/android.md) |
| iOS | `Package.swift`, `*.xcodeproj`, `import SwiftUI` | [`reference/ios.md`](reference/ios.md) |
| Flutter | `pubspec.yaml`, `lib/main.dart`, `package:flutter` imports | [`reference/flutter.md`](reference/flutter.md) |
| Kotlin Multiplatform | `commonMain`, `expect`/`actual`, `kotlin { }`, or `project.yaml`/`module.yaml` | [`reference/kmp.md`](reference/kmp.md) |

A KMP project has a platform UI too: read the KMP reference first, then the Android or iOS reference for the UI layer. When the stack is ambiguous, the build file decides.

Reviewing rather than building? Walk [`reference/review-checklist.md`](reference/review-checklist.md) after reading the stack reference.

## Universal rules

1. **Layer by knowledge, not file type.** UI renders and forwards intent. Logic owns rules and state transitions. Data owns I/O. A type that imports both a UI framework and a network client marks a crossed seam. Transport shapes (DTOs, rows, JSON) stop at the data seam and become domain models.
2. **One source of truth per piece of state.** State lives in the logic layer; UI renders a snapshot. Model exclusive states as one type, not parallel flags, and derive values at read time instead of storing a second copy that can drift.
3. **Flow one way.** Events go down (UI to logic), state comes up (logic to UI). Siblings never call each other; route through the owner.
4. **Inject at the edge.** Build dependencies at the app entry point and pass them in. Logic depends on interfaces, not globals; a static accessor hides the seam and leaks state between tests.
5. **Model failure as a value at the boundary.** Data returns a result or a typed error; logic maps it to UI state; UI renders it. Reserve exceptions for programmer error.
6. **Navigation is data.** Routes are typed values carrying identifiers, not strings or object graphs assembled at call sites, so deep links and back stacks stay testable.
7. **The platform owns platform UX.** Back gesture, permissions, keyboard, safe areas, and share sheets follow the host platform even when a shared layer could imitate them. A foreign back button is noticed faster than a shared abstraction saves code.
8. **Accessibility is part of done.** Every interactive element has a label, a target of at least 44–48 dp/pt, and survives the largest text size. Verify with the platform's inspector, not by eye.
9. **Test at the seam.** Unit-test logic with fakes; contract-test data against a fake server or database; keep end-to-end UI tests to one or two flows per feature. Snapshot tests belong to the design system, not to every screen.
10. **Format and lint are automated.** A formatter owns layout and a linter owns the mechanical rules, so review attention goes to design. Wire both into the build and CI, and when a warning is wrong, change the rule rather than suppressing it at the call site.
11. **Plan first, confirm direction, and move docs with code.**
    - Every feature, bug, refactor, or post-scaffold implementation starts with an explicit plan entry in `docs/plan.md`.
    - Present the plan and get user approval before writing code. After project creation, confirm direction (full one-shot scaffold, dependency setup, or specific feature) rather than assuming.
    - `docs/architecture.md` and `README.md` update in the same atomic commit whenever shape, dependencies, commands, or behavior move. A change is not done while documents disagree.
12. **Match the codebase, then modernize.**
    - Follow the style, naming, structure, and APIs already in the source unless the request says otherwise.
    - Prefer the simplest idiomatic solution and established best practice, including error handling, over a newer API merely because it is new, or over copied code.
    - Use the current idiom and features of the language and SDK version the project targets; never introduce a deprecated API or older style.
    - Modernize a gist, answer, or open-source snippet before adding it. If it cannot be modernized, mark it `TODO` or document the original source and the older API or style it uses, and add a probable replacement when one exists.
    - Do not ask whether to fix or modernize it; either do it or record why it remains.
    - State the errors and edge cases you are not handling in a brief comment: what can fail, and why. A reader should not have to simulate the code to find them.
    - Keep comments and documentation to brief descriptions of intent, not a scratchpad or a transcript of the work.
13. **Verify in proportion to the change.** Split verification into an inner loop and a gate. The inner loop compiles or analyzes only what changed and runs one filtered test, so it costs seconds. The gate runs the formatter check, linter, full tests, and build once per batch of edits, not after each file. Keep the build daemon warm, batch independent edits, and skip the gate for prose-only changes. Probe the toolchain (SDK platforms, JDK, tool versions) before writing build config, so an incompatible version fails in seconds rather than after a download. Never provision an emulator, simulator runtime, or large SDK image without asking. If the gate has run more than a handful of times for one change, the loop is too expensive; fix the loop before continuing. The stack reference lists the exact commands.
14. **Keep the toolchain and dependencies the generator provisioned.** Use the SDK, platform target, language/compiler version, build tool, and dependency versions that the official CLI or canonical generator selected across all variants (Android, iOS, Flutter, KMP); do not downgrade them to fit a linter, formatter, tool, or library. When a tool or dependency cannot run on or coexist with those versions, surface it as its own decision: name the tool or package, the versions it requires, and the provisions it would replace, then let the user choose between upgrading/finding a modern replacement, leaving it out, or accepting the downgrade. Never pin an older runtime, toolchain, or dependency silently, and never add a lint or compiler suppression whose only job is to hide version skew. Record the outcome in `docs/decisions.md`, and repeat an unresolved conflict in the final result.
15. **Commit small, atomic, and often.**
    - Commit regularly during feature work and bug fixes to keep progress traceable and rollbacks safe. Avoid monolithic commits with giant changelogs.
    - Target diffs under ±200 lines; stretch to ±500 lines only when necessary. When a logical change exceeds this, split it into sequential parts using the standard convention: `summary (1/x)`.
    - Exceptions: freshly initialized project scaffolds (commit the template intact without splitting) and changes too tightly coupled to separate without breaking context or buildability.
16. **Read the request for intent, not just its words.** A prompt names things loosely: feature names, flows, steps, and usage are sketches unless the user says they are fixed. Work out the problem and the outcome first, then plan the shape that serves them. Where the literal instruction would produce a worse design, follow the intent and say what you changed and why.
17. **Name a thing for what it is.** Prompt wording is a hint, not a naming standard. A feature, module, type, or variable takes the domain term that describes its job; a name that overstates, understates, or misstates it is a defect that outlives the task. Rename before the name spreads into routes, tests, and documents.
18. **Keep features and modules independent.** Feature and module are the same unit here. One may depend on core utilities, repositories, shared contracts, and the design system; it may never import another feature. No `feature-b` import in `feature-a`, and no shared screen or view model between them. A boundary you cannot cut is a boundary you cannot test or replace.
19. **Reach a sibling through a contract, not an import.** When one feature needs data or behavior another owns, route it through a service or repository, whichever fits: a repository when it is data, a service when it is behavior. The contract lives in shared code, the consumer depends on it, and the provider implements it at the composition root, so feature A never names feature B.
20. **Leave side effects with the caller.** Navigation, sharing, and other outward actions leave a screen as callbacks the screen receives, not calls the screen makes. A call-to-action that opens feature B takes an `onOpenB: () -> Unit`; the route orchestrator supplies the real behavior and a test supplies a no-op. The screen stays unaware of feature B, so it can be tested, kept, or replaced alone.
21. **Depend on interfaces, not implementations — for everything injected.** Feature entry points and constructors take interfaces or function types rather than concrete classes: repositories, services, data sources, platform and navigation adapters, and features themselves. A test can then pass a fake and another build can pass a different implementation. Build the concrete instance once at the edge. A parameter that drags a real database, network, or navigator into a test is a seam that leaked.

## Shared or platform code

For KMP and Flutter, decide what crosses the platform boundary before writing:

- Share domain models, validation, use cases, repository interfaces, and data contracts: the parts with no UI.
- Keep UI, navigation, permissions, and platform services native unless the product explicitly chooses shared UI (Compose Multiplatform, Flutter).
- Share a feature only when two platforms ship it. A single-platform app gains nothing from an abstraction built for a second platform that does not exist.
- Expose state and intent across the boundary, never widgets or view controllers.

## Project documents

Four files record the work. Keep each to its job, and link instead of copying:

| File | Holds | Update when |
| --- | --- | --- |
| `docs/decisions.md` | what was asked, what was agreed, and why, plus a short architecture summary | a choice is made or changed |
| `docs/architecture.md` | the current shape: modules, seams, dependency direction, libraries and versions | the shape changes |
| `docs/plan.md` | release or goal checklists, and one entry per feature or bug with requirements, sub-steps, and status | work is planned, started, or finished |
| `README.md` | what the app does, how to build, run, and test it, and the layout | behavior, setup, commands, or structure changes |

A feature, bug, or refactor gets an entry in `docs/plan.md` before code. Architecture and README move in the same change as the code; a change is not done while they disagree. Read [`reference/planning.md`](reference/planning.md) for the entry template, the sync matrix, and the intake questions, including when to propose a smaller approach instead.

`AGENTS.md` is the entry point: it points an agent at these files and at this skill, and stays short enough to read first.

## Applying

1. **Name the stack.** Done when you have read the matching reference above.
2. **Record the work.** New feature or bug work gets a `docs/plan.md` entry first; a choice goes to `docs/decisions.md`. State the intent behind the request and the names you will use; where a name or flow departs from the prompt's wording, record why. Note the architecture and README updates the entry will require. See [`reference/planning.md`](reference/planning.md).
3. **Place the change.** Name the layer it belongs to and the seam it crosses. If you cannot name the layer, the design is not settled.
4. **Check the seam.** Ask whether UI imports data, logic imports UI, shared code imports platform APIs, or one feature imports another. Any yes means the dependency moves behind an interface.
5. **Sync the documents.** Update `docs/architecture.md` and `README.md` in the same change. Done when the plan entry, architecture, README, and decisions agree.
6. **Verify with the stack's tools.** Work in the stack reference's inner loop, then run its gate once for the batch. A green build is the floor, not the proof; rule 13 sets the budget.
