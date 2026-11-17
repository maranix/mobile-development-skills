# Flutter conventions

Dart, Flutter, and one state-management choice per project. Pair with the universal rules in [`../SKILL.md`](../SKILL.md).

## Language level

The package language version is the lower bound of `environment.sdk` in `pubspec.yaml`; read it before the first Dart edit. `^3.14.0` enables every form below. Use the newest form the bound allows; at 3.13+, a simple class that repeats its fields in a constructor is a defect.

| Form | Min | Use for |
| --- | --- | --- |
| Records, patterns, switch expressions, class modifiers | 3.0 | Outcomes, destructuring, exhaustive UI states |
| Extension types | 3.3 | Zero-cost domain wrappers |
| Digit separators | 3.6 | Large literals |
| Wildcard variables | 3.7 | Unused parameters and destructuring |
| Null-aware elements | 3.8 | Optional list, map, and set entries |
| Dot shorthands | 3.10 | Enum values, static members, constructors where the context type is known |
| Primary constructors; concise `new`/`factory` constructors | 3.13 | Fields and the main constructor in one declaration |

```dart
// Do not add this form for a simple value class at 3.13+.
class User {
  final String name;
  final int age;
  User(this.name, this.age);
}
```

```dart
// Add this form.
class User(final String name, final int age);

class const TripCard({
  super.key,
  required final Trip trip,
  final VoidCallback? onTap,
}) extends StatelessWidget {
  @override
  Widget build(BuildContext context) =>
      ListTile(title: Text(trip.name), onTap: onTap);
}
```

- Use a primary constructor for values, models, widgets, and states whose fields come straight from the constructor. Keep a traditional constructor only for a second non-redirecting generative constructor, a body the primary form cannot express, or a super call that cannot be written as a super parameter.
- Use `new name()` and `factory name()` for in-body constructors at 3.13+.
- Use dot shorthands where the context type is known: `.center`, `.min`, `.idle`, `.fromJson()`. Do not repeat the type at the call site.
- Model exclusive states with a sealed class and a `switch` expression; destructure with object or record patterns instead of `is` plus a cast.

```dart
Widget toolbar(List<Widget> children) => Column(
  mainAxisAlignment: .center,
  mainAxisSize: .min,
  children: children,
);

Widget body(HomeState state) => switch (state) {
  Loading() => const CircularProgressIndicator(),
  Loaded(:final items) => ItemList(items),
  Failed(:final error) => ErrorView(error),
};
```

A form above the constraint is unavailable. Do not enable an experiment to reach one; record the version gap instead. `dart migrate` (3.14+) upgrades a project; run it only when the plan includes the upgrade.

Before returning a Dart diff, scan it: collapse any new simple field-plus-constructor class to a primary constructor, replace repeated enum, static, or constructor type names with dot shorthands, and replace `is` plus a cast with a pattern. Apply this to new code only; leave untouched code alone.

## Structure

Feature-first, with layers inside the feature:

```
lib/
  app/                   composition root: MaterialApp, router, theme
  features/<name>/
    presentation/        screens, widgets, state notifiers
    domain/              models, use cases, repository interfaces
    data/                DTOs, API clients, local storage, repositories
  core/                  design system, networking, analytics
```

## State

- Pick Riverpod or Bloc for the project and enforce it. Mixing the two doubles the ways a screen can be wrong.
- Expose state as an immutable sealed class or `AsyncValue`; the screen renders the state rather than inferring it.
- Keep notifiers and blocs free of `BuildContext`. Widgets translate state to UI and send intents back.

## UI

- Import Material from the package the generator uses. Recent Flutter generates `package:material_ui/material_ui.dart`; `package:flutter/material.dart` is deprecated since 3.47. Keep the `material_ui` dependency the generator adds.
- The screen widget is the only stateful layer; leaf widgets take plain data and callbacks and can be `const`.
- Use `LayoutBuilder` for layout decisions and `MediaQuery` for device facts (text scale, insets). Choosing a layout from `MediaQuery.size` ignores the constraints of the widget you are building, which matters as soon as a pane is narrower than the window.
- Give list items keys when the list can reorder, or state follows the wrong row.

## Navigation and dependencies

- `go_router` with typed route objects; configure deep links per platform and parse them into the same routes the UI pushes.
- Riverpod providers, or a single `get_it` registration at the edge. No service-locator calls from widgets.

## Data and errors

- `dio` or `http` plus `json_serializable`; `freezed` for unions if the project already uses it.
- Wire `FlutterError.onError` and `PlatformDispatcher.instance.onError` to the crash reporter; an unhandled async error is otherwise invisible.

## Testing

- `flutter test` for domain and notifier logic; `flutter_test` widget tests for screen states; `integration_test` for one or two flows per feature.
- Golden tests for the design system, with a fixed font and surface size, or they fail on every machine.

## Format and lint

- `dart format` owns layout and `flutter analyze` owns the rules. Include `flutter_lints` or `very_good_analysis` in `analysis_options.yaml`, and start new projects on the stricter set.
- CI runs `dart format --output=none --set-exit-if-changed .` and `flutter analyze --fatal-infos --fatal-warnings`.
- Exclude generated files (`*.g.dart`, `*.freezed.dart`, `*.gr.dart`) from formatting and analysis; regenerate them instead of editing them.
- `// ignore:` needs a reason. Fix the code, or narrow the rule in `analysis_options.yaml`, before reaching for it.
- The generator owns toolchain and dependencies: keep provisioned Flutter, Dart, and package versions in `pubspec.yaml`. Do not downgrade them for a package or tool.
- If a package requires older SDK bounds, ask the user (modern alternative, omit, or accept downgrade); never downgrade silently.
- Never add `// ignore:` comments or disable analysis rules solely to mask SDK or dependency version skew.

## Build and verify loop

```bash
# inner loop
flutter analyze lib/features/home
flutter test test/features/home/home_view_model_test.dart

# gate, once per batch
dart format --output=none --set-exit-if-changed .
flutter analyze --fatal-infos --fatal-warnings
flutter test
```

- `flutter analyze` accepts a path, so analyze only the directory that changed while writing.
- `flutter run` and `flutter build apk|ipa` are the slow tail. Probe with `flutter devices` and run only on an attached device; do not download a platform, SDK, or emulator image without asking, and name the size first.
- `flutter pub get` once after a dependency change; use `flutter pub get --offline` when the cache is warm.
- Regenerate goldens once for the change (`flutter test --update-goldens`), not after every edit.

## Gotchas

- Check `mounted` (or `context.mounted`) after every `await` before touching `BuildContext`.
- Large JSON parsing on the main isolate drops frames; move it to `compute` or an isolate.
- Text scaling overflows fixed-height rows; test at the largest scale.
- Ship with `--split-debug-info` and `--obfuscate`; the default artifact carries symbol data the store does not need.
