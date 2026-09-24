---
name: wave
description: Work through a .plans/<slug> slice plan one slice at a time in the main tree, in dependency order, tracking progress in state.json. Use when the user wants to run a plan, work the next slice, check slice progress, or mentions wave, slices, or a .plans folder.
compatibility: Claude Code
disable-model-invocation: true
---

# wave

Executes slice plans. The pipeline is **`/to-plan` → `/to-slices` → `/wave run`**. The
`.plans/<slug>/` directory (`prd.md`, `slices/*.md`, `state.json`, `learnings.md`) is the contract
between the stages.

You do the work yourself, in the main tree, one slice at a time. No subagents, no worktrees.

## Commands

| Invocation | What it does |
|---|---|
| `/wave run <slug> [-s range]` | Work the plan until done, a HITL slice, or a failure. |
| `/wave status <slug>` | Show each slice's status and what is runnable next. |
| `/wave reset <slug> <slice>` | Set a slice back to `pending`. |
| `/wave done <slug> <slice>` | Force-mark a slice `done`. |

`-s` limits the run to slice numbers: `-s 3-7`, `-s 2`, `-s 3,5-6`. Default: all.

`state.json` maps each slice to `{ "status": "pending" | "done", "blocked_by": [...] }`. Edit it
directly. For `status`, `reset` and `done`, read or edit it and report.

## Running a plan

Read `prd.md` and `learnings.md` once at the start. Then loop:

1. **Pick** the lowest-numbered `pending` slice in range whose `blocked_by` slices are all `done`.
   None left → report what shipped and stop.
2. **HITL halts.** If the slice's `**Type:**` is `HITL`, stop and tell the user what decision or
   review it needs. Never implement a HITL slice unattended.
3. **Implement** the slice from `slices/<slice>.md`. Meet every acceptance criterion.
4. **Validate** with the steps the spec lists. Fix every error you introduced. If you cannot get
   it clean, stop and report — do not mark it done or move on.
5. **Record.** Set the slice to `done` in `state.json`. If you learned something later slices
   need (a gotcha, an interface contract, a decision), append it to `learnings.md` under a
   `## <slice>` heading.
6. **Commit** the slice's code (not `.plans/`) with the slice key as the message. Never push.

`.plans/` stays uncommitted unless the user commits it. To resume an interrupted run, run
`/wave run <slug>` again — `state.json` on disk says where you left off.
