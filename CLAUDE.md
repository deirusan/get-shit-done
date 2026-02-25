# GSD Token Efficiency Project

This is a fork of [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) focused on reducing token consumption without changing behaviour.

**Fork:** `deirusan/get-shit-done`
**Upstream:** `gsd-build/get-shit-done`
**Install target:** `~/.claude/` (Claude Code global)

---

## How to Reinstall After Changes

```bash
node bin/install.js --claude --global
```

No build step needed unless you edit files under `hooks/src/` — in that case run `npm run build:hooks` first.

**Verify changes landed:**
```bash
grep "research:" ~/.claude/get-shit-done/bin/lib/core.cjs
grep "@.*execute-plan\|@.*checkpoints\|@.*tdd" ~/.claude/get-shit-done/workflows/execute-phase.md
```

---

## Architecture You Must Understand

GSD has three layers:

1. **Commands** (`commands/gsd/*.md`) — slash command entry points, loaded into the main Claude Code context. They `@`-reference workflow files.
2. **Workflows** (`get-shit-done/workflows/*.md`) — orchestration logic. The main Claude Code session runs these. They spawn subagents via `Task()`.
3. **Agents** (`agents/gsd-*.md`) — system prompts for subagents. Each spawned `Task(subagent_type="gsd-executor")` loads the corresponding `agents/gsd-executor.md` as its system prompt.

**Critical:** When a workflow spawns a Task with `subagent_type="gsd-executor"`, the agent's system prompt is `agents/gsd-executor.md`. Any `@`-references in the Task `prompt` parameter are loaded as additional user-turn context. If the agent system prompt already covers the same content, those `@`-references are pure duplication.

**Config defaults** live in `get-shit-done/bin/lib/core.cjs` in the `defaults` object inside `loadConfig()`. These are overridden by `.planning/config.json` in the user's project.

---

## Changes Already Made (commit 35f55ec)

### 1. Removed redundant `@`-references from executor Task prompts
**File:** `get-shit-done/workflows/execute-phase.md`

When spawning `gsd-executor` agents, the prompt previously included:
```
@~/.claude/get-shit-done/workflows/execute-plan.md   (445 lines)
@~/.claude/get-shit-done/references/checkpoints.md   (776 lines)
@~/.claude/get-shit-done/references/tdd.md
```

These are fully covered by `agents/gsd-executor.md` (the agent's own system prompt). Removed all three. Kept only `@summary.md` template since it's the output target. Saves ~1,200 lines per executor spawn; ~3,600 lines per 3-agent wave.

### 2. Research defaults to off
**File:** `get-shit-done/bin/lib/core.cjs`

Changed `research: true` → `research: false` in the config defaults. Research now requires an explicit `--research` flag or `workflow.research: true` in `.planning/config.json`. Saves 30–50K tokens per phase for users who don't need domain investigation.

### 3. Researcher output cap
**File:** `agents/gsd-phase-researcher.md`

Added instruction to keep RESEARCH.md under 1500 tokens (~1200 words). Previously unbounded — a verbose researcher on a complex domain could produce 10–15K tokens that then get loaded into the planner's context.

### 4. Plan-checker iteration limit reduced
**File:** `get-shit-done/workflows/plan-phase.md`

Reduced max revision loop from 3 → 2 iterations. Each iteration is a full agent spawn (~12–18K tokens). Plans needing 3 passes are usually underspecified.

---

## Remaining Improvements (Prioritised)

### ~~Priority 1 — Phase-scoped document excerpts~~ DONE

**`get-shit-done/bin/gsd-tools.cjs`**: Added `roadmap phase-brief <phase>` command returning compact JSON + `brief_md` with only the current phase entry from ROADMAP.md and the requirement IDs it references.

**`get-shit-done/workflows/plan-phase.md`**: Researcher, planner, and plan-checker agents now receive `<phase_brief>` context instead of full ROADMAP.md + REQUIREMENTS.md file paths.

**Saving:** ~70–80% reduction on ROADMAP.md + REQUIREMENTS.md reads across the 3 agents spawned per plan-phase invocation.

---

### ~~Priority 2 — Consolidate the `<project_context>` boilerplate~~ DONE

Extracted the `<project_context>` block (repeated verbatim in every agent) to `get-shit-done/references/project-context.md`. All agent system prompts now reference it via `@~/.claude/get-shit-done/references/project-context.md`. Future updates to the pattern are a single-file change.

---

### ~~Priority 3 — Compress verbose agent prompts~~ DONE

Both large agents compressed by ~30% with no behaviour change:
- `agents/gsd-debugger.md` — 1,201 → ~840 lines
- `agents/gsd-planner.md` — 1,194 → ~836 lines

Multi-paragraph prose converted to tables and bullet lists throughout.

---

### ~~Priority 4 — Consolidate or remove `execute-plan.md`~~ DONE

`get-shit-done/workflows/execute-plan.md` deleted. `/gsd:execute-plan` command was already removed (CHANGELOG). References cleaned up in `agents/gsd-planner.md`, `templates/phase-prompt.md`, `commands/gsd/execute-phase.md`, `get-shit-done/workflows/execute-phase.md`, `templates/codebase/structure.md`, `templates/summary.md`.

---

### ~~Priority 5 — Cap orchestrator reads of SUMMARY.md files~~ DONE

**`agents/gsd-executor.md`**: Added `Self-Check`, `One-liner`, and `Key-files` fields to `<completion_format>`.

**`get-shit-done/workflows/execute-phase.md`**: Spot-check in `execute_waves` step 4 now uses:
- File existence: `[ -f {path} ]` on `Key-files` from executor return
- Commit check: `git log --oneline --all --grep="{phase}-{plan}" | wc -l`
- Self-check: `Self-Check:` field from executor return (grep fallback if agent failed)

Wave report and aggregate results now use `One-liner from executor return` instead of reading full SUMMARY.md.

---

### ~~Priority 6 — Review auto-advance execute-phase prompt~~ DONE

**`get-shit-done/workflows/plan-phase.md`**: Removed `checkpoints.md` and `tdd.md` from the auto-advance Task prompt. Orchestrator's checkpoint handling is fully inline in `execute-phase.md`; TDD is an executor-only concern. Kept `model-profile-resolution.md` (orchestrator uses executor/verifier model selection). Saves 1,039 lines per auto-advance spawn.

---

## What NOT to Change

- Do not change agent tool lists or frontmatter `tools:` fields
- Do not change any gsd-tools CLI behaviour — only the config defaults and workflow markdown files
- Do not alter the output formats (`## PLANNING COMPLETE`, `## PLAN COMPLETE`, etc.) — these are parsed by orchestrators
- Do not remove the `summary.md` template reference from executor prompts — it is the active output template
- `execute-plan.md` has been deleted (Priority 4 complete) — do not recreate it
