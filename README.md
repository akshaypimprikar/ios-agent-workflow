# iOS Agent Workflow

Claude Code slash commands for agent-driven iOS app development. Drop these into any iOS project to get a full 7-agent workflow: spec → plan → build → test → review → bugfix → release.

## Install

```bash
cp -r .claude/commands/ /path/to/your-ios-app/.claude/commands/
```

Or copy only the commands you need.

## Commands

| Command | Agent | When to use |
|---|---|---|
| `/spec` | Spec Agent | Turn a feature idea into an approved design spec |
| `/plan` | Planner Agent | Turn an approved spec into a task-by-task implementation plan |
| `/feature` | Feature Agent | Execute an approved plan (TDD, one commit per task) |
| `/test` | Test Agent | Write tests for a feature branch (runs parallel with `/review`) |
| `/review` | Review Agent | Review a PR for architecture compliance |
| `/bugfix` | Bug Fix Agent | Fix a bug with a regression test (test-first) |
| `/release` | Release Agent | Version bump, changelog, tag, and merge to main |

## Workflow

```
Idea
 → /spec   → you approve
 → /plan   → you approve
 → /feature ∥ /test → PR opened
 → /review → merge to develop
 → (repeat)
 → /release → main tagged
```

Bug report → `/bugfix` → PR → `/review` → merge

## Assumptions

This workflow assumes:
- **MVVM + Repository** architecture
- **SwiftData** for persistence (repository pattern abstracts it)
- **Apple Testing framework** (`import Testing`, `@Suite`, `@Test`, `#expect`)
- **iOS 26.4+** — simulator target is `iPhone 17`
- **`PBXFileSystemSynchronizedRootGroup`** — files auto-compile, no `project.pbxproj` edits
- Money values use `Decimal`, never `Double`
- A `CLAUDE.md` at the repo root with project-specific build commands

## Architecture rules enforced by `/review`

- Views contain no business logic
- Domain Services have zero SwiftData imports
- Repository Protocols import Foundation only
- ViewModels depend on protocols, not concrete implementations
- No `Double` for money — `Decimal` only
- No force-unwraps in production code

## Customising for your project

Each command file is plain markdown. Edit them to match your project's:
- App target name (replace `FinanceTracker` with your module name)
- Simulator name (if not `iPhone 17`)
- Additional architecture rules in `CLAUDE.md`
