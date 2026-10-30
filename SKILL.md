---
name: career-update
description: Log recent GitHub/Jira/Confluence work into the career evidence file, tagged against the Staff/Principal rubrics.
disable-model-invocation: true
---

# Career Update

User-invoked only, run weekly-ish. Appends one dated section to `~/.claude/career/updates.md` covering work since the last logged date.

## Steps

1. **Find the window.** Read `~/.claude/career/updates.md`; its most recent dated heading is the window start. Empty file → ask the user for a start date.

2. **Load the rubrics fresh.** Read `~/.claude/career/2026-h2-goals.md` (the primary lens, manager-set), `staff-rubric.md`, `principal-rubric.md` (numbered items, source for `[Rubric-Category#N]` tags), and `gaps.md` (standing gaps this window's work might close).

3. **Pull GitHub evidence.**
   ```
   gh search prs --author "@me" --merged --merged-at ">=<window-start>" --limit 200 \
     --json repository,number,title,closedAt \
     -q '.[] | "\(.closedAt|split("T")[0]) \(.repository.name)#\(.number) \(.title)"'
   ```
   This searches every repo you can see. Ask which repos/orgs to cover only if results span unexpected ones.
   For PRs about performance, CI, cost, or memory, read the PR body (`gh pr view <n> --json body`) and carry any before/after figures into the bullet with the PR as source; titles alone drop the measured impact.

4. **Pull Jira evidence** (`mcp__plugin_atlassian_atlassian__searchJiraIssuesUsingJql`):
   ```
   assignee = currentUser() AND resolved >= "<window-start>" ORDER BY resolved DESC
   assignee = currentUser() AND updated >= "<window-start>" AND statusCategory != Done ORDER BY updated DESC
   ```
   The first query is completed work (the stronger claim); the second is in-progress work. A resolved ticket with no matching merged PR in the window is usually a bulk move or project migration; confirm with the user before citing it.

5. **Pull Confluence evidence** (`mcp__plugin_atlassian_atlassian__searchConfluence`), both queries: they catch different things.
   ```
   creator = currentUser() AND type = page AND created >= "<window-start>"
   contributor = currentUser() AND type = page AND lastmodified >= "<window-start>"
   ```
   Label results by which query found them: `creator` hits are owned docs (stronger signal); `contributor` hits are edits to someone else's page (weaker), and the match can come from an older edit of yours on a page someone else changed this window; cite one only when the user confirms an edit in the window.

6. **Ask for off-system work, then synthesize.** Before drafting, ask the user for work this window that leaves no GitHub/Jira/Confluence trace: hiring and interviews, mentoring and coaching, talks and demos, Slack threads. These often carry the strongest `[Principal-Collaboration#N]` evidence. Ask for a link to any feedback or thanks and quote it verbatim. Then synthesize by theme, not by source system. One bullet can combine a PR, its Jira ticket, and a doc it produced. Tag every bullet `[Rubric-Category#N]`. A number in a bullet (PR count, dollar amount, event count) needs an inline source; without one, tag it `[unverified]`.

7. **Reconcile `gaps.md`.** Check whether anything pulled this window closes a standing gap: a second mentee, cross-team architecture work, a tool adopted by another team. If so, name the gap and propose the edit to `gaps.md`; leave it untouched otherwise.

8. **Confirm scope, then append.** If any pull was ambiguous (repo choice, JQL scope, a borderline tag), show the drafted section before writing. Otherwise append a new `## <window-start> to <today>` section to `updates.md`; prior sections are immutable history, not draft material.

9. **Sync the Confluence mirror.** Replace each private page's body with its local file (full content minus the `#` title line), so the pages match exactly. Repeat after any later edit to either file in the same session. Done when every page's latest version matches its file.
   - `updates.md` → "Career Updates", page `4165042656`
   - `gaps.md` → "Career Gaps / Watch-outs", page `4165795859`
   - cloudId `3a4014c4-180a-4b0f-8524-060165699381`

## Completion bar

Every source in steps 3 to 5 that returned zero results for the window gets an explicit "no evidence found" line, not silence and not a filler bullet. A theme with one real PR behind it is one bullet, not three generic-sounding ones.

## Boundaries

`updates.md` and `gaps.md` stay separate files. Gap tracking never gets copied back into the evidence log (see the note at the top of `gaps.md`). Nothing this skill reads gets promoted into `CLAUDE.md` or any other always-loaded file; that reversal is the one mistake that would put the context cost right back.
