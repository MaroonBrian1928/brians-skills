---
name: delegating-to-subagents
description: Subagent delegation (the Agent tool). Use before dispatching a subagent to implement something, when splitting work into parallel chunks, and when a subagent hands back.
---

# Delegating to subagents

The lead coordinates and reviews, subagents implement. The lead's context is the scarcest resource on the team: spend it on decisions and merges, and let the diff and the repo's check command carry the detail.

## Steps

1. **Settle.** List every design choice the work depends on and resolve each one before dispatch: ask the user, or pick a default and say so. Done when every brief reads "do X", with no "decide X" left in it.
2. **Cut.** Split by the code each chunk owns, drawing boundaries for the widest parallel frontier: most chunks should touch disjoint files and run at once. When several chunks need the same file or interface, land that shared edit first as its own small chunk (see Worktrees), then fan out. Keep a change yourself when its brief would be longer than the change. Done when every file has exactly one owning chunk at a time and only true dependencies run in sequence.
3. **Brief.** Fill the template below for each chunk. Done when a stranger could finish the chunk without asking a question.
4. **Dispatch.** Run implementation agents on `model: "haiku"`; step up to a stronger model only for a chunk that needs design judgement, debugging, or reading a large unfamiliar area, and say why. Give every parallel implementation agent `isolation: "worktree"`; sequential chunks may share the main tree. Dispatch fresh agents, never `fork`: a fork ignores `model` and inherits the lead's whole context.
5. **Steer.** Send a running agent small additions with SendMessage. When a requirement changes direction, stop the agent (TaskStop) and dispatch a fresh one with the new brief. A stopped agent may have partly run a command: check its worktree before reporting what happened.
6. **Review.** Review each finished diff yourself: you hold the design context a reviewer lacks. When the diffs are too large to read whole, hand them to a reviewer on a stronger model than the implementer, then read its findings and the riskiest hunks yourself. Done when every problem in each diff is listed.
7. **Fix.** Make a fix yourself in the worktree when explaining it would take longer than making it; send larger fixes back to the implementing agent with SendMessage: it keeps its context and worktree, so the fix costs no re-read. Done when the check command passes in each chunk's worktree, the brief's **Tests** are met, and each reported deviation is accepted or fixed. A chunk not done after two fix rounds goes to Escalate.
8. **Escalate.** A chunk still failing after two fix rounds has a brief or model problem, not a context one. Rewrite the brief and dispatch a fresh agent, a stronger one if needed, without `isolation` and without **Base**, so it builds on the existing worktree. It starts in the lead's working directory and `cd` does not persist between its commands, so give absolute worktree paths in **Owns** and have it prefix every command with `cd <worktree> &&`. Then Review again; if this agent also fails two rounds, stop and ask the user.
9. **Land** each chunk into the main tree one at a time (see Worktrees). Done when every chunk has landed, the check passes on the main tree, and no worktree remains. Deploy only after that.

## Brief template

- **Base**: for a worktree agent, its first command, before any edit: `git reset --hard <sha>`. At dispatch the lead runs `git add` on any new files, then `sha=$(git stash create) && git rev-parse "${sha:-HEAD}"` in the main tree, and writes the printed SHA into the brief. It fails on unresolved conflicts or intent-to-add (`git add -N`) entries; resolve or stage those first.
- **Goal**: the behaviour wanted, described from the user's side, and why.
- **Owns**: the files and directories this agent edits; everything else is read-only to it.
- **Interfaces**: exact signatures it must use or produce, when other chunks depend on them.
- **Tests**: every behaviour change gets a test that goes red without the change, including the edge cases you already know about. For a fix that guards a subtle failure (races, double-posting, fallbacks), have the agent remove the fix, watch its test go red, then restore it.
- **Verify**: the repo's single check command (e.g. `mise run check`). Leave changes uncommitted in your worktree.
- **Report**: at most 10 lines: files changed, tests added, deviations, open risks.
- **Pace**: end every brief, including ones for research and review agents, with this sentence verbatim: "Time matters here: do not spend time that can be avoided, and the earlier a correct result is obtained, the better." Without it, subagents put thoroughness ahead of speed.

Point at the repo's rules (AGENTS.md, the check command) instead of restating them; a rule the check command enforces needs no sentence in the brief.

## Worktrees

- **Base.** Claude Code may branch a new worktree from the remote's default branch rather than the lead's `HEAD`; the brief's **Base** resets it to the lead's current tracked state: `git stash create` snapshots staged and unstaged edits to tracked files, including landed chunks, without touching the main tree. Untracked files are not included, so stage new files before dispatch. Subagents never commit (see **Verify**).
- **Main tree.** Keep it still while worktree agents run: only staging, committing and landing; no checkouts, resets or deploys.
- **Landing.** Stage or commit every main-tree edit first (yours and any sequential agent's): `--3way` implies `--index`, which rejects a patch touching a file with unstaged changes. Then `git -C <wt> add -N . && git -C <wt> diff --binary HEAD > <wt>.patch && git apply --3way <wt>.patch` (`add -N` brings new files into the diff; `HEAD` catches staged edits; the patch file survives as a backup). A failed apply may leave conflict markers in the main tree: stop and keep the worktree. Otherwise re-run the check; only if it passes, `b=$(git -C <wt> branch --show-current) && git worktree remove --force <wt> && git branch -D "$b"` (forced because its changes stay uncommitted; the branch would otherwise linger).
