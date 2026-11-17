# Android conventions

Kotlin, Compose, and AndroidX. Pair with the universal rules in [`../SKILL.md`](../SKILL.md).

## Language level

Read the Kotlin `languageVersion` and toolchain from Gradle before the first edit, and keep the generated level. Use the current Kotlin idiom it supports (sealed interfaces, data objects, value classes, exhaustive `when`) rather than the older Java-shaped form. An official language skill, when present, is authoritative for syntax.

## Structure

Feature-first, with layers inside each feature:

```
app/                     composition root: Application, MainActivity, DI graph
feature/<name>/
  ui/                    screens, components, UiState
  domain/                models, use cases, repository interfaces
  data/                  DTOs, Room entities, repository implementations
core/                    design system, networking, analytics, common utils
```

`app` is the only module that knows every feature. A feature may depend on `core` and on another feature's domain interface, never on its UI.

## UI and state

- One stateful entry point per screen owns the `ViewModel`; everything below it is stateless and takes data plus lambdas. This is what makes a screen previewable and testable.
- Collect with `collectAsStateWithLifecycle()`, not `collectAsState()`, so collection stops when the screen is not visible.
- Model screen state as one immutable `UiState`; deliver one-shot effects through a `Channel`, not through a flag in state.
- Hoist state to the lowest common owner. A dialog's open flag belongs to the composable that shows it, not to the screen's `ViewModel`.
- Keep `Context` out of the `ViewModel`. Resolve resources in the UI layer, or inject a provider scoped to the application context.

## Navigation

- Use Navigation 3, or type-safe Navigation Compose on an older setup. Routes are `@Serializable` objects.
- The back stack is state. Express intent with `popUpTo` rather than launching a new task to clear it.

## Dependencies and data

- Constructor injection everywhere; Hilt only at the composition root. A `@Singleton` repository is a decision, not a default.
- Room for relational data, DataStore for preferences, and a typed result or sealed error from the repository.
- Coroutines: `viewModelScope` for UI work, `Dispatchers.IO` for blocking I/O, `supervisorScope` when one child failing must not cancel the rest. No `GlobalScope`.

## Testing

- JUnit and Turbine for `Flow` and `StateFlow` logic.
- Compose UI tests for the stateful entry point of each screen; assert on semantics, not pixels.
- Robolectric only where an instrumented test is impractical; it does not prove device behavior.

## Format and lint

- `ktlint` formats and `detekt` analyzes Kotlin; Android Lint covers platform and Compose checks. Format on save and run `ktlintCheck`, `detekt`, and `lintDebug` in CI.
- One `.editorconfig` at the repository root is the single source for ktlint; do not override rules per module.
- `@Suppress` needs a reason on the same line, or the suppression is deleted in review.
- Keep lint and detekt versions in the version catalog with the other plugins.
- The generator owns toolchain and dependencies: keep provisioned JDK, Gradle, SDK, AGP, Kotlin, and library versions. Never downgrade them for a linter, formatter, or third-party tool.
- If a tool rejects provisioned versions (e.g. detekt failing on the JDK), ask the user (upgrade tool, omit tool, or accept downgrade); never downgrade silently.
- Never disable lint rules (e.g. `AndroidGradlePluginVersion`, `GradleDependency`, `OldTargetApi`) to mask version skew. Every suppression requires a genuine defect reason.

## Build and verify loop

Split the work: a compile or one test after an edit, one gate for a batch.

```bash
# inner loop
./gradlew :app:compileDebugKotlin
./gradlew :app:testDebugUnitTest --tests '*HomeViewModelTest'

# gate, once per batch
./gradlew ktlintCheck detekt lintDebug testDebugUnitTest assembleDebug
```

- Keep the Gradle daemon; `--no-daemon` pays configuration and JIT cost on every call. Use `--offline` when the dependencies are cached.
- Run `ktlintFormat` on its own when formatting changes files, then the gate. With a warm daemon the follow-up costs seconds.
- `lintDebug` is the slow task: it analyzes the main, unit-test, and androidTest sources. `assembleDebug` is the other tail. Neither is a useful per-edit check.
- Verify a manifest change with `aapt2 dump badging app/build/outputs/apk/debug/app-debug.apk` instead of installing on a device.
- `connectedDebugAndroidTest` needs an attached device or a running emulator. Do not download a system image or create an AVD without asking, and name the size first.
- Before choosing `compileSdk`, a plugin, or a linter version, check the installed platforms, `java -version`, and the tool's compatibility notes. An unresolvable SDK or a linter that cannot parse the JDK fails minutes into the build.

## Gotchas

- Edge-to-edge is enforced for recent target SDKs. Handle insets in the scaffold, not per screen.
- `remember` dies on configuration change; `rememberSaveable` survives process death only for small, `Parcelable`-friendly values.
- R8 strips reflection targets. Keep rules for serialization models and anything constructed by name.
- Startup is measured with a baseline profile; a `ContentProvider` initializer that blocks the main thread shows up as cold-start time.
