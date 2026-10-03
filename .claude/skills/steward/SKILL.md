---
name: steward
description: How to follow a PR in a Claude Code web/remote session — react to PR events, schedule no check-ins. Read before acting on CI or review events.
---

# Following a PR: events only, no check-ins

The cloud harness reads this file before acting on PR events, and it
overrides the default of keeping a check-in scheduled until the PR is done.

## Why

Each `send_later` / trigger wake is a full turn that re-sends the whole
conversation. The prompt cache lasts about an hour, so hourly wakes often miss
it and pay full price. A PR waiting on the owner overnight cost about 8 turns
that did nothing (physics_sim, 2026-09-24). The owner merges when awake. Nothing
the agent does in between speeds that up.

## Rules

1. **Subscribe, don't schedule.** After opening a PR, call
   `subscribe_pr_activity`. CI failures, reviews and comments arrive as events
   and wake the session on their own. **Never arm `send_later` or a trigger to
   re-check a PR.** A local CLI session (e.g. a long-running tmux one) has no
   PR events; it costs nothing while idle, so don't give it a timer either
   (`/loop`, `CronCreate`, `ScheduleWakeup`). Check the PR at the next real turn.
2. **Before pushing, check the result instead of waiting for it.** Run the
   repo's checks locally (`npm run build && npm test`, `npm run guards`) so the push is expected to
   go green. An event wakes you if it doesn't.
3. **A green PR waiting on the owner needs nothing.** Say once that it's ready
   to merge, then end the turn. Don't comment on the PR to say so.
4. **On an event, act on the whole PR once:** CI on the latest head, merge
   conflicts, open review threads. Fix and push, or say what's blocking. Then
   end the turn.
5. **Clean up what's armed.** If a check-in exists anyway (an older session or
   the harness armed one), run `list_triggers` and delete it once its PR is
   merged or closed. After a merge, unsubscribe and confirm the commit landed:
   `git merge-base --is-ancestor <sha> origin/main`.

**Exception:** the owner asks for a check-in in so many words ("check back in
an hour"). Then schedule exactly that one. It isn't standing permission.
