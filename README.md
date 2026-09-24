<img src="wave.png" width="88" alt="wave" align="left" />

# wave

wave runs a plan to completion in [Claude Code](https://www.claude.com/product/claude-code) (only runtime for now). You break a feature into slices with dependencies, and wave works through them one at a time in dependency order, committing each one, until the work is done or it hits something that needs you.

## Why I built it

[This workshop](https://www.youtube.com/watch?v=-QFHIoCo-Ko) got me wanting to run [Ralph loops](https://ghuntley.com/ralph/) again. A Ralph loop is a single agent in a shell `while` loop, fresh context each pass, with state surviving in the repo (a TODO/state file plus git history). wave keeps that shape but runs inside your Claude Code session instead of a shell, and swaps the flat TODO list for a dependency graph of slices.

Planning is already solved. Matt Pocock's [grill-me](https://github.com/mattpocock/skills/tree/main/skills/productivity/grill-me) / [grill-with-docs](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs) interrogate an idea until it's sharp, then [to-prd](https://github.com/mattpocock/skills/tree/main/skills/engineering/to-prd) / [to-issues](https://github.com/mattpocock/skills/tree/main/skills/engineering/to-issues) write it up. Those land in your GitHub issue tracker; I wanted to stay local.

```
grill-me / grill-with-docs   →   sharpen the idea
to-plan / to-slices          →   write slices to .plans/   (local files, not GitHub issues)
/wave run                    →   execute them
```

So `to-plan` / `to-slices` (bundled here) do the same job but write `.plans/` files instead of issues. wave is the execution half that runs them.

It compounds on both: Matt's grilling and dependency-graphed slices for the plan, Ralph's loop-until-done patterns for the run. Everything stays local: plans are files in `.plans/`, progress lives in a `state.json`, and each slice is a commit, so there's no issue tracker, no API token, and no separate harness to run.

## Install

```
npx skills add nicotrop/wave-skills --all
```

That installs all three skills. To pick a subset, drop `--all` for an interactive prompt, or target one with `--skill wave`. Add `-g` to install globally across all projects.

Or install manually:

```
git clone https://github.com/nicotrop/wave-skills
cp -r wave-skills/skills/{wave,to-plan,to-slices} ~/.claude/skills/
```

Either way the folders land in your Claude Code skills directory (wave is Claude Code only for now).

## Skills

- **`/to-plan`** — turn a sharpened idea into a plan
- **`/to-slices`** — break the plan into dependency-ordered slices in `.plans/<slug>/`
- **`/wave run`** — work through the slices until done or a slice needs you

## How it works

A plan is `.plans/<slug>/`: a `state.json` keyed by slice with `blocked_by` edges, plus a markdown spec per slice. wave picks the next pending slice whose blockers are done, implements it in your working tree, runs its checks, marks it done in `state.json`, and commits it. Anything it learns that later slices should know goes into `learnings.md`.

It stops when the plan is done, when the next slice is marked HITL and needs a decision from you, or when a slice's checks won't pass. wave never pushes, so review the commits afterward and squash or amend as you like.

```
/wave run <slug>          run the whole plan
/wave run <slug> -s 1-4   run slices 1–4, then stop so you can review the commits
/wave status <slug>       see where the plan stands
```

If a run gets interrupted, run it again. `state.json` remembers where it left off.
