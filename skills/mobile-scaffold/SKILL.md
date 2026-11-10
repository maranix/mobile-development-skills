---
name: mobile-scaffold
description: Scaffold a new mobile app on Android, iOS, Flutter, or Kotlin Multiplatform. Invoke before creating the project; it reads the request, infers the stack, scope, dependencies, and defaults, asks only for the gaps, records the choices in docs/decisions.md, and sets up docs/architecture.md, docs/plan.md, README.md, and AGENTS.md in sync.
disable-model-invocation: true
---

# Scaffold a mobile app

One-time recipe. The output is a skeleton that builds; the steps add structure, tests, documentation, one navigable screen, and optional lint.

**Generate, don't hand-write.** Start from the stack's recommended CLI or canonical wizard, then modify its output. Generated files are the baseline, so the project keeps upstream structure and fixes.

Run step 1, then read [`../mobile-conventions/SKILL.md`](../mobile-conventions/SKILL.md) and the stack reference it routes to. Each step ends with a check; do not proceed until it passes. Use the stack reference's inner loop; save the full gate for the end.

## 1. Read the request and fill the gaps

The prompt is the brief. Extract what it states, infer the rest, ask only what remains; never re-ask what it answers.

**Extract:** purpose and audience; framework or SDK when named; app name, application/bundle ID, and organization when present; features; stated constraints such as offline use, permissions, or target devices.

**Infer and state:**
- **Scope:** a request for an app means scaffold; "template", "starter", or "boilerplate" means vanilla, stopping after step 2.
- **Dependencies:** from features. Offline means local persistence; camera and object detection mean a camera permission and an on-device inference library; sync means networking. Take the smallest set the first feature needs.
- **Defaults:** platform state, navigation, DI, and standard lint (table).
- **Identity:** no name means propose 3–5 with lowercase reverse-DNS IDs derived from the organization, and let the user pick. Availability is unverified unless asked.
- **Agent instructions:** if the prompt asks for agent support or names Claude, Gemini, Copilot, or similar, create `AGENTS.md` without asking.

**Ask for the gaps in one batch:** identity (name, IDs, organization, platforms, minimum OS; KMP also shared vs native UI and Gradle vs Kotlin Toolchain); framework only if unclear; license from a short menu (MIT, Apache-2.0, GPL-3.0, proprietary) with one recommendation and reason; extras only if a feature needs a library the defaults do not cover or the user overrides; lint only if non-standard; agent instructions for scaffold scope unless already asked (recommended, since the next agent finds the docs and skill).

| Concern | Android | iOS | Flutter | KMP |
| --- | --- | --- | --- | --- |
| State | ViewModel + StateFlow (default) | `@Observable` (default), TCA (The Composable Architecture) | Riverpod (default), Bloc, Provider | StateFlow (default) |
| Navigation | Navigation 3 (default), type-safe Navigation Compose | NavigationStack (default) | go_router (default) | platform navigation (default) |
| DI | constructor (default), Hilt, Koin | constructor (default), Factory, Swinject | constructor (default), Riverpod, get_it | constructor (default), Koin, kotlin-inject |
| Persistence | Room (default), DataStore (prefs) | SwiftData (default), Core Data | drift (default), sqflite; shared_preferences (prefs) | SQLDelight (default), Room KMP |
| Networking | Retrofit/OkHttp (default), Ktor | URLSession (default), Alamofire | http (default), dio | Ktor (default) |

Lint levels: **standard** (the stack's formatter and linter per its reference, warnings failing CI), **strict** (`very_good_analysis`, SwiftLint `--strict`, a larger detekt config), or **none** (skip step 5). Android and Xcode templates ship without a lint config; Flutter ships `analysis_options.yaml` with `flutter_lints`; KMP templates usually ship none.

**Record** `docs/decisions.md` before generating: the request in one paragraph, scope, and agreed choices—stack, UI strategy, build, name, IDs, organization, platforms, minimum OS, state, navigation, DI, persistence, lint, license, agent instructions, and the README outline. Add a one-paragraph architecture sketch linking to `docs/architecture.md`, which step 10 fills in.

Example: "Create a new app for workout tracking which works offline and uses camera and object detection to recognize my workouts automatically" fixes purpose and features; remaining gaps are stack, name and ID, organization, platforms, and license. Offline means local persistence; camera and inference mean the camera permission and an on-device model; state, navigation, and DI take defaults.

**Check:** every gap has an answer, and decisions record the request and agreed choices.

## 2. Create and commit the project template

Use the official generator, kept current (`flutter upgrade`, `kotlin update`, current Xcode and `android` CLI):

- **Android:** prefer `android create empty-activity --name="My App" --application-id=com.example.myapp --output=./my-app`. Check that `android create` exists first; if missing, recommend installing it (it covers creation, devices, screenshots, and SDK management) and ask before installing per https://developer.android.com/tools/android-cli. Only if declined, fall back to Android Studio > New Project. `gradle init` does not create an Android app.
- **iOS:** Xcode > File > New > Project > App. If the agent cannot drive the GUI, ask the user to run the wizard, then take over. No third-party generator unless asked.
- **Flutter:** `flutter create --empty --org com.example --platforms=android,ios my_app`.
- **KMP:** `kotlin init` (install with `curl -fsSL https://kotl.in/install.sh | sh`). If unavailable, recommend the Kotlin Toolchain and ask before installing; if declined, use the Gradle path via `kmp.jetbrains.com` or IDE > New Project > Kotlin Multiplatform. The Toolchain is pre-1.0; choose Gradle when the project must stay on the standard build.

Generate the smallest app the tool offers. Keep the versions the generator chose (see [Keep the generator's toolchain](#keep-the-generators-toolchain-and-dependencies)).

**Check:** the empty app builds on the generator's toolchain and dependencies (JDK, Gradle, Xcode, SDK, Flutter, Dart, packages) and launches if a device or simulator is already attached. Never provision one without asking.

**Commit the template:** commit the freshly generated template intact as its own commit (no split) to establish the clean upstream baseline.

**Pause, plan, and confirm direction:** do not modify the template.
1. **Present the plan:** architecture, dependencies, and steps in `docs/plan.md`.
2. **Ask for direction:** *full scaffold* (steps 3–11), *targeted setup* (specific dependencies, build logic, or design system first), or *direct feature work* (a specific feature or domain module).
3. **Wait for approval**, then invoke `mobile-conventions` and proceed with small atomic commits.

## 3. Apply the folder structure

Create the feature-first tree from the stack reference, one shared core module, no features yet. The composition root constructs services.

**Check:** the tree matches the reference and each module declares its dependency direction.

## 4. Wire dependency injection

Construct the graph at the entry point and pass dependencies down. Start with constructor injection; add the framework (Hilt, SwiftUI environment, Riverpod, Koin) only for what the skeleton needs.

**Check:** one repository interface has a real implementation and a fake, both constructed at the edge.

## 5. Add formatting, lint, and CI

Skip if None. Use the chosen level: the stack's formatter and linter with root config, then a CI job running format-check, lint, and tests on every push, failing on warnings.

**Check:** a deliberately misformatted line fails locally (revert it), and CI is green on the empty app.

## 6. Add the navigation skeleton

Two typed routes and one screen moving between them; a deep link resolves to the second route.

**Check:** the deep link opens the right screen and back returns to the first.

## 7. Add the test harness

One unit test for logic, one UI test for the screen, both in CI. Add the CI job here if step 5 was skipped. Use fakes at the seams; no live network.

**Check:** a failing assertion fails locally (revert it), and the suite runs in CI.

## 8. Add the accessibility baseline

Label interactive elements, set the minimum target size, and check the screen at the largest text size.

**Check:** the inspector reports no missing labels, and the layout does not clip at the largest scale.

## 9. Add the README and license

Write `README.md` from decisions and the original description: purpose, platforms, prerequisites, configuration, build/run/test commands, layout, and license. Add the chosen `LICENSE`, or an "All rights reserved" notice for proprietary. Treat it as living: step 10 checks it against the architecture; future work updates it with the plan entry. Write it after commands and versions settle.

**Check:** a newcomer can build and run from the README alone, and the license matches decisions.

## 10. Record the decisions and architecture

Update decisions with what the build changed (actual package name, added dependencies, lint config). Write architecture: feature and module tree, seams, dependency direction, libraries and versions, platform split. Create plan with an empty `## Next` and the scaffold under `## Done`. Re-read README and align names, commands, and dependencies. Do this after the build settles; future work reads all four first.

**Check:** all four files exist, agree, and let a contributor name every choice and dependency from them alone.

## 11. Add agent instructions

If agreed or asked, write `AGENTS.md` telling the next agent to:

- read decisions, architecture, plan, and README before changing code;
- invoke `mobile-conventions` and follow its stack reference;
- start every feature, bug, or refactor with a plan entry;
- update architecture and README in the same change when shape, dependencies, commands, or behavior change;
- treat work as incomplete while plan, architecture, README, and decisions disagree;
- run build, lint, and tests before calling work done.

If the prompt named a tool, add `CLAUDE.md` or `GEMINI.md` as a one-line pointer to `AGENTS.md`. Vanilla: skip unless asked; then point only at decisions and the skill.

**Check:** a fresh agent finds the docs and skill from `AGENTS.md` alone and knows a plan entry comes first and docs move with code.

## Keep the generator's toolchain and dependencies

- Treat the generator's toolchain, compiler, SDK, deployment targets, and dependencies across all variants (Android, iOS, Flutter, KMP) as the baseline; a generated project that builds is the reference. Do not replace or downgrade versions for a secondary tool, linter, formatter, or library.
- If a tool cannot support them, stop and report a decision: name the package or tool, the versions it demands, what was provisioned, and the three options (upgrade/replace, omit, accept downgrade). Ask before choosing.
- Never silently pin an older runtime, SDK, compiler, or dependency, and never add a suppression whose only job is to hide version skew. Record dropped packages or accepted downgrades in decisions.
- Repeat unresolved conflicts in the final result.

Done when the project runs on the provisioned versions, or the user chose an exception and it is recorded.

## Keep the loop cheap

- **Probe before config:** in one command, list installed SDK platforms and Build-Tools, the JDK, and generator, wrapper, linter, and plugin versions. Choose installed, compatible versions and record why.
- **Warm once:** let the generator and first build download the wrapper, dependencies, and sometimes a JDK once; keep the daemon alive; no `--no-daemon`.
- **Inner loop while writing:** compile the touched target, run one filtered test, or analyze one directory (commands in the stack reference).
- **Gate once per batch:** format check, lint, full tests, and build after a group of edits. Prose changes need no code gate; check them against the code and links.
- **Batch before the gate:** fix all known import, lint, and formatting problems in one pass, then verify once.
- **Check for a device, don't create one:** probe `adb devices`, `xcrun simctl list devices booted`, or `flutter devices`; never download an image, runtime, or platform pack, or create a device, without asking and naming the size.
- **Write commands last:** README and architecture commands and versions after the build stops changing.

Done when the scaffold needed one cold build, a few inner-loop checks, and one final gate.

## Keep the skeleton small

- One navigation example, not a feature.
- Recommend the platform's built-in option first; a package earns its place when the first feature needs it.
- Delete generator sample code, counter state, and placeholder TODOs in the first commit after the template commit.
