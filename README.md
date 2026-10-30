# career-update

An agent skill that logs your recent work into a career evidence file, tagged against your Staff/Principal engineering rubrics.

Run it weekly, or whenever. It pulls:

- GitHub PRs merged since your last logged entry (`gh pr list`)
- Jira issues you touched (via an Atlassian MCP server)
- Confluence pages you created or edited (via CQL `creator`/`contributor` queries)

Then it writes one dated section into `~/.claude/career/updates.md`, tagged `[Rubric-Category#N]` against whatever rubric files you point it at.

It's a plain `SKILL.md`, so any agent or tool that reads that format can run it. Nothing here is tied to one vendor's CLI.

## Why user-invoked, not automatic

It only loads when you invoke it by name. Career tracking is the kind of thing that's tempting to wire into always-loaded instructions so it "just happens," but that means paying its context cost on every run, including the ones that have nothing to do with it. Manual invocation means it costs nothing until you actually want it.

## Setup

Expects these files to exist (edit the paths in `SKILL.md` if yours differ):

- `~/.claude/career/2026-h2-goals.md`, your current goals, the primary lens
- `~/.claude/career/staff-rubric.md`, `principal-rubric.md`, numbered rubric items
- `~/.claude/career/gaps.md`, standing gaps, checked each run, never auto-merged
- `~/.claude/career/updates.md`, the evidence log this skill appends to

None of these ship in this repo. They're personal; write your own.

Step 9 mirrors `updates.md` and `gaps.md` to two private Confluence pages. Replace the page IDs and cloudId in `SKILL.md` with your own pages, or delete the step if you don't want a mirror.

## Install

Drop this directory wherever your agent looks for skills. For Claude Code:

```
git clone https://github.com/tanmayairbase/career-update ~/.agents/skills/career-update
ln -s ~/.agents/skills/career-update ~/.claude/skills/career-update
```

Then invoke it by name: `/career-update`.
