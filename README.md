# Brian's skills

Agent skills for [Claude Code](https://claude.com/claude-code).

| Skill | What it does |
|-------|--------------|
| [delegating-to-subagents](delegating-to-subagents/SKILL.md) | Splits work into chunks that run in parallel subagents, each in its own git worktree, then reviews, fixes and lands their uncommitted changes. |

## Install

In Claude Code:

```
/plugin install brians-skills --marketplace MaroonBrian1928/brians-skills
```

On older versions, add the marketplace first with `/plugin marketplace add MaroonBrian1928/brians-skills`, then `/plugin install brians-skills@brians-skills`.

Every push to `main` is a new release. To get updates without running `/plugin marketplace update`, turn on auto-update for the `brians-skills` marketplace under `/plugin` → Marketplaces.
