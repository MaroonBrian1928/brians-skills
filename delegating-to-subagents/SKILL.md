---
name: delegating-to-subagents
description: Subagent delegation (the Agent tool). Use before dispatching a subagent to implement something, when splitting work into parallel chunks, and when a subagent hands back.
---

# Delegating to subagents

The lead coordinates and reviews, subagents implement. The lead's context is the scarcest resource on the team: spend it on decisions and merges, and let the diff and the repo's check command carry the detail.

## Steps

1. **Settle.** List every design choice the work depends on and resolve each one before dispatch: ask the user, or pick a default and say so. Done when every brief reads "do X", with no "decide X" left in it.
2. **Cut.** Split by the code each chunk owns, drawing boundaries for the widest parallel frontier: most chunks should touch disjoint files and run at once. When several chunks need the same file or interface, land that shared edit first as its own small chunk and commit it (see Worktrees), then fan out. Keep a change yourself when its brief would be longer than the change. Done when every file has exactly one owning chunk at a time and only true dependencies run in sequence.
3. **Brief.** Fill the template below for each chunk. Done when a stranger could finish the chunk without asking a question.
4. **Dispatch.** Run implementation agents on `model: "haiku"`; step up to a stronger model only for a chunk that needs design judgement, debugging, or reading a large unfamiliar area, and say why. Give every parallel implementation agent `isolation: "worktree"`; sequential chunks may share the main tree.
5. **Steer.** Send a running agent small additions with SendMessage. When a requirement changes direction, stop the agent (TaskStop) and dispatch a fresh one with the new brief. A stopped agent may have partly run a command: check its worktree before reporting what happened.
6. **Review.** Review each finished diff yourself: you hold the design context a reviewer lacks. When the diffs are too large to read whole, hand them to a reviewer on a stronger model than the implementer, then read its findings and the riskiest hunks yourself. Done when every problem in each diff is listed.
7. **Fix.** Make a fix yourself in the worktree when explaining it would take longer than making it; send larger fixes back to the implementing agent with SendMessage: it keeps its context and worktree, so the fix costs no re-read. Cap this at two fix rounds; a chunk still failing after that has a brief or model problem, not a context one. Then rewrite the brief and dispatch a fresh agent, a stronger one if needed, without `isolation` and pointed at the existing worktree path, so it builds on the work already there. Done when the check command passes in each chunk's worktree, the brief's **Tests** are met, and each reported deviation is accepted or fixed.
8. **Land** each chunk into the main tree one at a time (see Worktrees). Done when every chunk has landed, the check passes on the main tree, and no worktree remains. Deploy only after that.

## Brief template

- **Goal**: the behaviour wanted, described from the user's side, and why.
- **Owns**: the files and directories this agent edits; everything else is read-only to it.
- **Interfaces**: exact signatures it must use or produce, when other chunks depend on them.
- **Tests**: every behaviour change gets a test that goes red without the change, including the edge cases you already know about. For a fix that guards a subtle failure (races, double-posting, fallbacks), have the agent remove the fix, watch its test go red, then restore it.
- **Verify**: the repo's single check command (e.g. `mise run check`). Leave changes uncommitted in your worktree.
- **Report**: at most 10 lines: files changed, tests added, deviations, open risks.
- **Pace**: end every brief, including ones for research and review agents, with this sentence verbatim: "Time matters here: do not spend time that can be avoided, and the earlier a correct result is obtained, the better." Without it, subagents put thoroughness ahead of speed.

Point at the repo's rules (AGENTS.md, the check command) instead of restating them; a rule the check command enforces needs no sentence in the brief.

## Worktrees

- **Base.** A new worktree starts from `HEAD`, so it sees only committed work. Subagents never commit; they leave their changes uncommitted for the lead to check. The lead commits whenever later worktrees must see landed work.
- **Main tree.** Keep it still while worktree agents run: landing and committing finished chunks only, no checkouts, resets or deploys.
- **Landing.** `git -C <wt> add -N . && git -C <wt> diff --binary HEAD > <wt>.patch && git apply --3way <wt>.patch` (`add -N` brings new files into the diff; `HEAD` catches staged edits; the patch file survives as a backup). If the apply fails, stop and keep the worktree. Otherwise re-run the check; only if it passes, `git worktree remove --force <wt>` (forced because its changes stay uncommitted).
