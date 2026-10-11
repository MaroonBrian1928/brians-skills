# Brian's skills

Agent skills for [Claude Code](https://claude.com/claude-code).

| Skill | What it does |
|-------|--------------|
| [delegating-to-subagents](delegating-to-subagents/SKILL.md) | Splits work into chunks that run in parallel subagents, each in its own git worktree, then reviews, fixes and lands their uncommitted changes. |

## Install

Copy a skill's folder into `~/.claude/skills/` (all projects) or `.claude/skills/` (one project):

```sh
git clone git@github.com:MaroonBrian1928/brians-skills.git
cp -r brians-skills/delegating-to-subagents ~/.claude/skills/
```
