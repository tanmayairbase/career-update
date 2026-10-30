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
   gh pr list --author "@me" --state merged --search "merged:>=<window-start>" --limit 200 \
     --json number,title,mergedAt,url,repository \
     -q '.[] | "\(.mergedAt|split("T")[0]) \(.repository.name)#\(.number) \(.title)"'
   ```
   Ask which repos/orgs to cover if more than one is plausible; guessing scope here silently drops evidence.

4. **Pull Jira evidence** (`mcp__plugin_atlassian_atlassian__searchJiraIssuesUsingJql`):
   ```
   assignee = currentUser() AND updated >= "<window-start>" ORDER BY updated DESC
   ```
   Mark each issue resolved/closed or merely touched; resolved is the stronger claim.

5. **Pull Confluence evidence** (`mcp__plugin_atlassian_atlassian__searchConfluenceUsingCql`), both queries: they catch different things.
   ```
   creator = currentUser() AND type = page AND created >= "<window-start>"
   contributor = currentUser() AND type = page AND lastmodified >= "<window-start>"
   ```
   Label results by which query found them: `creator` hits are owned docs (stronger signal); `contributor` hits are edits to someone else's page (weaker).

6. **Synthesize by theme**, not by source system. One bullet can combine a PR, its Jira ticket, and a doc it produced. Tag every bullet `[Rubric-Category#N]`. A number in a bullet (PR count, dollar amount, event count) needs an inline source; without one, tag it `[unverified]`.

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
