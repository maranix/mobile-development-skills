# Review checklist

Walk every item. A "no" is a finding, not a style preference. Read the stack reference first, then this list.

## Seams

- [ ] UI imports no data-layer types: DTOs, database rows, HTTP clients.
- [ ] Logic imports no UI framework.
- [ ] Shared code imports no platform APIs.
- [ ] Dependencies are injected at the entry point; logic holds no global singletons.
- [ ] Transport types become domain models at the repository boundary.

## Modularity

- [ ] No feature or module imports another; dependencies point at core utilities, repositories, shared contracts, or the design system.
- [ ] Data and behavior shared across features travel through a repository or service interface, not a direct import.
- [ ] Side effects such as navigation leave a screen as callbacks supplied by the caller, so the screen does not name the destination feature.
- [ ] Feature entry points and constructors take interfaces or function types, not concrete implementations: repositories, services, data sources, platform and navigation adapters alike.
- [ ] A feature can be tested, kept, or replaced without compiling the features around it.
- [ ] The change does not split one responsibility across more classes, files, or modules than the context needs; each new entity has a distinct responsibility, state, lifecycle, or seam.

## State

- [ ] Each piece of state has one owner; no second copy that can drift.
- [ ] Exclusive states are one type, not parallel flags.
- [ ] UI renders a snapshot; business rules live in the logic layer.
- [ ] One-shot effects are not stored as booleans in state.

## Navigation

- [ ] Routes are typed and carry identifiers, not object graphs.
- [ ] Deep links parse into the same route types the UI pushes.
- [ ] Back behavior is expressed in the back stack, not by relaunching the app.

## Failure

- [ ] Expected failures are values at the boundary; exceptions are for programmer error.
- [ ] Every failure maps to a UI state the user can act on.

## Platform fit

- [ ] Back gesture, permissions, keyboard, and safe areas follow the host platform.
- [ ] Shared UI is a recorded decision, not drift from screen to screen.

## Accessibility

- [ ] Interactive elements have labels.
- [ ] Targets meet 44–48 dp/pt.
- [ ] Layout survives the largest text size.

## Quality gates

- [ ] Formatter and linter run in CI and pass.
- [ ] Verification matched the change: inner loop while editing, one gate for the batch.
- [ ] No build, lint, or test gate was spent on a documentation-only change.
- [ ] No emulator, simulator runtime, or large SDK download was provisioned without asking.
- [ ] Toolchain and dependency versions match the generator baseline; no tool forced a downgrade.
- [ ] Incompatible tools were escalated as decisions, not silently pinned, dropped, or suppressed.
- [ ] Logic has unit tests at the seams; one end-to-end flow per feature.
- [ ] No suppression without a stated reason.
- [ ] No lint rule, ignore comment, or compiler warning was disabled to hide version skew.
- [ ] Commits are small, atomic, and frequent (under ±200 lines ideally, ±500 max), or split as (1/x) unless tightly coupled or an initial project scaffold.

## Documentation

- [ ] Any plan shown to the user was approved before code; direction was confirmed after project creation.
- [ ] The plan followed the request's intent, not its literal wording.
- [ ] A tracked feature or behavior change has a short plan entry; a rename, dependency bump, internal refactor, or trivial fix added none.
- [ ] Plan entries are at most about twelve lines with no file list, commit message, or per-step log; completed entries are one line and dropped entries are deleted.
- [ ] Architecture and any design doc describe only the current state: no step list, change history, or superseded design.
- [ ] Every doc update deleted or rewrote the text it made false; the diff is not all additions.
- [ ] The doc diff is proportional: a rename or small fix is about ten lines, a feature about thirty, and a scaffold one page per doc.
- [ ] `README.md` matches what the app does and how to build, run, and test it.
- [ ] No document describes a feature, command, or plan item that no longer exists.

## Language level

- [ ] The build file's language level was read before the first edit and recorded in the plan or review notes.
- [ ] Every added line uses the newest form that level supports; no older form was copied from surrounding code.
- [ ] Dart at 3.13+ uses primary constructors for simple state-bearing classes and `new`/`factory` for in-body constructors; 3.10+ uses dot shorthands where the context type is known; 3.0+ uses patterns, records, sealed classes, and switch expressions instead of `is` plus casts.
- [ ] No form above the language level is used, and no experiment was enabled to reach one.

## Consistency

- [ ] The diff touches only the files and lines the change owns; no unrelated reformatting, reordering, renames, or refactors.
- [ ] New code follows the source's structure, architecture, and API choices.
- [ ] Edited code keeps its surrounding style unless the edit already touches the construct; deleted code was not modernized.
- [ ] Every class, interface, variable, constant, feature, and module is named for its role in this codebase; prompt or reference names were used only where they fit context and purpose.
- [ ] Names and flows that depart from the request's wording are explained in the plan or the result.
- [ ] Added code that could not use the current syntax has a `TODO` or a documented reason, plus a probable replacement.
- [ ] Unhandled errors and edge cases are named in a brief comment: what can fail, and why.
- [ ] Comments and documentation describe intent; they are not a scratchpad or a transcript.

## Per stack

Run the checks in the matching reference: [android](android.md) · [ios](ios.md) · [flutter](flutter.md) · [kmp](kmp.md).
