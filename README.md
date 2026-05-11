# iOS Agent Workflow

Claude Code slash commands for agent-driven iOS development. Drop these into any iOS project to get a full 7-agent pipeline: spec → plan → build → test → review → bugfix → release.

Every feature follows the same disciplined path — you approve the spec and the plan, everything else runs autonomously until you hit merge.

**See it in action:** [FinanceTracker](https://github.com/akshaypimprikar/financetracker-ios) — a real iOS app built entirely with this workflow, including specs, plans, and PRs going back to day one.

---

## Install

```bash
cp -r .claude/commands/ /path/to/your-ios-app/.claude/commands/
```

Then update the app target name, simulator, and build commands in each file to match your project. Each command is plain markdown — no code to run.

---

## Commands

| Command | Agent | Trigger |
|---|---|---|
| `/spec "feature idea"` | Spec Agent | Turn a feature idea into an approved design spec |
| `/plan docs/specs/my-spec.md` | Planner Agent | Turn an approved spec into a task-by-task implementation plan |
| `/feature docs/plans/my-plan.md` | Feature Agent | Execute an approved plan — TDD, one commit per task |
| `/test` | Test Agent | Write tests for a feature branch (runs parallel with `/review`) |
| `/review` | Review Agent | Review a PR for architecture compliance |
| `/bugfix "description"` | Bug Fix Agent | Fix a bug with a regression test — test-first |
| `/release 1.0.0` | Release Agent | Version bump, changelog, PR to main, tag |

---

## Pipeline

```
Idea
  └─ /spec    → proposes 2–3 approaches, you choose, spec doc saved
  └─ /plan    → task-by-task plan with exact code + xcodebuild commands
  └─ /feature → TDD per task: write failing test → implement → pass → commit
  └─ /review ─┐
  └─ /test   ─┘ (run in parallel on the PR)
  └─ merge to develop
  └─ (repeat for more features)
  └─ /release → develop → PR to main → tag

Bug report → /bugfix → PR → /review → merge
```

You approve twice: after `/spec` and after `/plan`. Everything else is autonomous.

---

## How the Feature Agent works

Each task in the plan follows strict TDD:

1. Write the failing test — confirm it fails for the right reason
2. Implement the minimal code to make it pass
3. Run the full test suite — no regressions allowed
4. Commit — one commit per task, no batching

The agent never proceeds to the next task if tests are red.

---

## Branch Strategy (Gitflow)

```
main        — production, tagged on release only, never receives direct feature PRs
develop     — integration branch, all features merge here
feature/*   — off develop
fix/*       — off develop  (hotfix/* off main)
release/*   — off develop, PR to main, back-merged to develop
spec/*      — off develop, for spec + plan docs
```

---

## Architecture Rules (enforced by `/review` on every PR)

- Views contain no business logic
- Domain Services have zero SwiftData imports — 100% unit testable without a simulator
- Repository Protocols import Foundation only
- ViewModels depend on protocols, never concrete implementations
- All money values use `Decimal`, never `Double`
- No force-unwraps (`!`) in production code

---

## Assumptions

- **MVVM + Repository** architecture
- **SwiftData** for persistence
- **Swift Testing** framework — `import Testing`, `@Suite`, `@Test`, `#expect()` for unit/integration tests; XCUITest for UI tests
- **iOS 26.4+** — default simulator target is `iPhone 17`
- **`PBXFileSystemSynchronizedRootGroup`** (Xcode 16+) — files auto-compile when placed in the correct directory; never edit `project.pbxproj`
- A `CLAUDE.md` at the repo root with project-specific build commands and architecture rules

---

## Customising for your project

1. Copy `.claude/commands/` into your project
2. In each file, replace `<AppName>` with your module name
3. Update `CLAUDE.md` with your project's build commands, simulator name, and any additional architecture rules
4. The agents read `CLAUDE.md` before every task — that's where project-specific context lives
