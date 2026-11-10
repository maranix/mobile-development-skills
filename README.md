# Mobile App Development Skills

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/format-Agent%20Skills-5B5BD6)](https://agentskills.io)
[![Platforms](https://img.shields.io/badge/platforms-Android%20%7C%20iOS%20%7C%20Flutter%20%7C%20KMP-3DDC84)](#skills)
[![Agents](https://img.shields.io/badge/agents-7%20supported-111827)](#supported-agents)

Agent skills that give coding assistants a shared engineering baseline for Android, iOS, Flutter, and Kotlin Multiplatform. They cover how a mobile app is structured and layered, where state belongs, how work is planned and recorded, and how a new project is scaffolded from the platform's official generator.

Both skills work with any agent that supports the [Agent Skills](https://agentskills.io) format. Skill bodies load on demand. `mobile-conventions` is model-invocable, so an agent can consult it while changing or reviewing mobile code; `mobile-scaffold` is marked explicit-only because it creates files and runs generators.

## Contents

- [Why these skills](#why-these-skills)
- [Skills](#skills)
- [Install](#install)
- [Usage](#usage)
- [Supported agents](#supported-agents)
- [How it works](#how-it-works)
- [Repository layout](#repository-layout)
- [Contributing](#contributing)
- [License](#license)

## Why these skills

- **Consistency for structure, current idiom for added code.** New code follows the source's structure, architecture, and API choices; names are chosen for their role in context, not copied from the prompt. Edits stay surgical, touching only the lines the change owns. Every added line uses the newest syntax the project's language version supports. The skill loads any official language or framework skill as the authority on current syntax, and never proposes an upgrade just to use newer syntax. Unhandled failures are named in a brief comment instead of being left for the reader to discover.
- **One spine, four stacks.** The same rules apply everywhere; each stack reference supplies the architecture idiom, recommended libraries, and platform gotchas, while official language or framework skills supply current syntax.
- **Seams over folder conventions.** Every boundary between UI, logic, data, and platform code is a cut you can substitute. The skills keep those seams narrow and testable instead of enforcing a directory shape.
- **Name by role, split by responsibility.** Classes, interfaces, variables, constants, and modules take the domain term that fits their context and purpose; prompt or reference names are only a starting point. Split only when a distinct responsibility, state, lifecycle, or seam earns it, so maintenance and data flow stay easy to follow.
- **Official generators first.** New projects start from the stack's recommended CLI or canonical wizard, never a hand-built tree, so the result keeps upstream structure and fixes.
- **Cheap feedback, one honest gate.** The inner loop compiles the touched target and runs one filtered test; the formatter check, linter, full tests, and build run once per batch. Toolchain versions are probed before they reach build files, and no emulator or runtime is downloaded unasked.
- **The generator owns the toolchain and dependencies.** The official CLI or wizard's platform targets, toolchain, SDK, compiler, and dependency versions stand across every variant (Android, iOS, Flutter, KMP). A tool or package that cannot run on them is surfaced as a decision with the exact versions, never fixed by downgrading the project or adding a suppression that hides the skew.
- **Small, atomic, frequent commits.** State is saved regularly during features and fixes to keep git history clean and predictable. Diffs target under ±200 lines (stretching to ±500 max), splitting larger units as `(1/x)`. Fresh project scaffolds are committed intact before implementation starts.
- **Plan first, confirm direction.** Every feature, fix, or post-scaffold implementation presents a plan and gets user approval before writing code. After project creation, the agent confirms direction (one-shot scaffold, dependency setup, or feature work) to stay aligned with user intent.
- **Documentation is part of the deliverable.** Decisions, architecture, planned work, and the project README stay in sync: a feature, bug, or refactor starts with a `docs/plan.md` entry, and `docs/architecture.md` and `README.md` move with the code. An optional `AGENTS.md` gives the next agent that contract.
- **Quiet by default.** A skill's body loads only when needed. The conventions skill can be matched automatically; the scaffold skill is explicit-only, and Claude Code and Codex can enforce that.

## Skills

### [`mobile-conventions`](skills/mobile-conventions/SKILL.md)

The shared reference for changing or reviewing a mobile codebase: consistency, naming, granularity, structure, layering, state ownership, navigation, dependency injection, persistence, error handling, formatting, lint, testing, verification, accessibility, and project documents, including keeping the README in sync.

A per-stack reference adds the idiom and gotchas. A [review checklist](skills/mobile-conventions/reference/review-checklist.md) turns the rules into pass/fail items, and a [planning reference](skills/mobile-conventions/reference/planning.md) covers the project documents and the feature and bug intake.

| Stack | Reference |
| --- | --- |
| Android (Kotlin, Compose) | [`reference/android.md`](skills/mobile-conventions/reference/android.md) |
| iOS (Swift, SwiftUI) | [`reference/ios.md`](skills/mobile-conventions/reference/ios.md) |
| Flutter (Dart) | [`reference/flutter.md`](skills/mobile-conventions/reference/flutter.md) |
| Kotlin Multiplatform | [`reference/kmp.md`](skills/mobile-conventions/reference/kmp.md) |

**Example prompts**

| Task | Prompt |
| --- | --- |
| Review | "Review this checkout screen for seam violations and tell me what to move." |
| Plan a feature | "Add offline sync to the workout app. Write the plan entry and requirements first." |
| Make a decision | "We are adding offline support. Which layer owns the cache, and what belongs in shared code?" |
| Configure tooling | "Set up ktlint, detekt, and a CI gate for this Android module." |
| Check shared code | "Is this file safe to move into `commonMain`?" |

### [`mobile-scaffold`](skills/mobile-scaffold/SKILL.md)

A one-time setup workflow. It reads the request, infers the stack, scope, dependencies, and defaults, and asks only for what is missing. It records the request and agreed choices in `docs/decisions.md`, generates the project with the official CLI or wizard, and applies the scaffold steps: folder structure, dependency injection, lint and CI, a navigation skeleton, a test harness, the accessibility baseline, the README and license, the detailed `docs/architecture.md`, a seeded `docs/plan.md`, and optional `AGENTS.md` instructions, with the README kept in sync with the architecture and plan. It also keeps the feedback loop cheap: probe the toolchain first, use an inner loop while writing, and run one full gate at the end.

**Example prompts**

| Scenario | Prompt |
| --- | --- |
| Description first | "Create a new app for workout tracking which works offline and uses camera and object detection to recognize my workouts and track them automatically." |
| Framework named | "Scaffold a Flutter app for tracking expenses. Bundle ID `com.acme.expenses`." |
| Agent support | "Scaffold a Flutter app for tracking expenses and set up agent instructions for Claude." |
| Overrides | "Scaffold a KMP app with Compose Multiplatform, Koin, SQLDelight, and strict lint." |
| Vanilla template | "Scaffold an iOS starter in Xcode. Just the template, no extras." |
| Bare prompt | "Scaffold a field-notes app; suggest names, a license, and a README." |

## Install

Install globally to make the skills available in every project, or into a project's `.agents/skills/` or `.claude/skills/` directory to share them with a team.

### With the skills CLI (recommended)

The [`skills` CLI](https://www.skills.sh/) resolves each agent's skill directory and installs both skills for the agents you name:

```sh
npx skills add maranix/mobile-app-development-skills -g -y \
  -a claude-code -a codex -a gemini-cli -a antigravity -a opencode -a cursor -a github-copilot
```

Requires Node.js 22.20 or later. Omit `-g` for a project install, add `--copy` if symlinks are unsupported, and set `DISABLE_TELEMETRY=1` to opt out of the CLI's anonymous telemetry. Update later with `npx skills update`.

### Manual installation

Clone the repository and link the skill folders into the directory your agent discovers:

```sh
git clone https://github.com/maranix/mobile-app-development-skills.git
cd mobile-app-development-skills

# Claude Code
mkdir -p ~/.claude/skills
ln -sfn "$PWD/skills/mobile-conventions" ~/.claude/skills/mobile-conventions
ln -sfn "$PWD/skills/mobile-scaffold" ~/.claude/skills/mobile-scaffold

# Codex, Gemini CLI, OpenCode, and other .agents/skills readers
mkdir -p ~/.agents/skills
ln -sfn "$PWD/skills/mobile-conventions" ~/.agents/skills/mobile-conventions
ln -sfn "$PWD/skills/mobile-scaffold" ~/.agents/skills/mobile-scaffold
```

For a project install, use `.claude/skills/` and `.agents/skills/` inside the repository instead of the home directory. For Antigravity, Cursor, GitHub Copilot, and other agents, use the path in [Supported agents](#supported-agents). Copy the folders instead of symlinking if symlinks are unsupported.

Earlier versions used the `mobile-app-conventions` and `mobile-app-scaffold` names. Remove those folders from your agent's skill directory before reinstalling, or the old copies will remain alongside the new ones.

## Usage

Name the skill, then describe the task. The conventions skill can also load on its own when the agent is changing or reviewing mobile code; the scaffold skill is explicit-only on Claude Code and Codex.

```text
/mobile-conventions Review this checkout screen for seam violations.
```

```text
mobile-scaffold: Scaffold a Flutter app for tracking expenses.
```

Agents with slash commands take the name as a command; agents without one take the name at the start of the request.

The conventions skill selects the stack reference from the build files, reads it, loads any official language or framework skill, and applies the rules. The scaffold skill runs its discovery step first, then generates and layers the project. Neither skill is a runtime dependency: they are instructions the agent follows in the conversation.

## Supported agents

| Agent | Project path | Global path | Invoke |
| --- | --- | --- | --- |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` | `/mobile-conventions` |
| Codex | `.agents/skills/` | `~/.agents/skills/` | `$mobile-conventions` or `/skills` |
| Gemini CLI | `.agents/skills/` or `.gemini/skills/` | `~/.agents/skills/` or `~/.gemini/skills/` | Mention the name; `/skills list` confirms it is loaded |
| Antigravity | `.agents/skills/` | `~/.gemini/config/skills/` | `/mobile-conventions` |
| OpenCode | `.opencode/skills/` or `.agents/skills/` | `~/.config/opencode/skills/` or `~/.agents/skills/` | Mention the name; the agent calls its `skill` tool |
| Cursor | `.agents/skills/` | `~/.cursor/skills/` | Mention the name |
| GitHub Copilot | `.agents/skills/` | `~/.copilot/skills/` | Mention the name |

`mobile-scaffold` sets `disable-model-invocation: true` for Claude Code and ships `agents/openai.yaml` with `policy.allow_implicit_invocation: false` for Codex, so neither agent runs it on its own. `mobile-conventions` has neither restriction: it can load automatically, and naming it is still the reliable way to force it. Gemini CLI, Antigravity, and OpenCode may also activate a skill from its description; the explicit-only flags are only enforced by Claude Code and Codex, so name `mobile-scaffold` when you need to be certain.

## How it works

The skills follow the Agent Skills progressive-disclosure model:

1. **Discovery.** The agent sees each skill's name and description, except skills that opt out of model invocation.
2. **Activation.** Naming the skill, or the agent matching it, loads its `SKILL.md`.
3. **References.** The skill reads a stack reference, checklist, or planning guide only when the task needs it.

This keeps the default context small while leaving the full rules available on demand. There is no build step, package manager, or generated artifact: the repository is the distributable.

## Repository layout

```text
skills/<skill-name>/
  SKILL.md            # name and description frontmatter, plus the agent's instructions
  agents/openai.yaml  # explicit-only skills: Codex invocation policy
  reference/          # material reached only through a pointer in SKILL.md
```

## Contributing

Contributions are welcome. To add a skill:

1. Create `skills/<skill-name>/SKILL.md` with `name` and `description` frontmatter.
2. Decide the invocation mode. Reference skills can be model-invocable; side-effecting workflows should add `disable-model-invocation: true` and `agents/openai.yaml` with `policy.allow_implicit_invocation: false`, so they run only when named.
3. Keep the main file to the process and the rules every run needs. Move branch-specific detail into `reference/` and point at it from `SKILL.md`.
4. Add the skill to [Skills](#skills) and open a pull request.

Keep skills stack-specific where the idiom differs, and keep the shared rules consistent across stacks. If a change alters a convention, update the matching reference and the review checklist together.

## License

Released under the [MIT License](LICENSE).
