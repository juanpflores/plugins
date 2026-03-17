---
name: slack-notification-triage
description: Triage recent Slack activity into a short personal priority list or task list for the user.
---

# Slack Notification Triage

Use this skill to produce a short personal priority queue or action list for the user from recent Slack messages. It is for surfacing what the user likely needs to read, reply to, or do next, not for summarizing Slack activity for a broader audience.

## Workflow

1. Treat this as personal triage for the user. Focus on messages directed at the user, messages likely needing a reply, and messages that create a concrete follow-up or task for the user.
2. Resolve the current user with `slack_read_user_profile` so you have the user's Slack ID for mention-based searches.
3. If the user provided channel names, DMs, people, or topic keywords, use that scope. If private channels, DMs, or group DMs are needed, request and wait for explicit consent before using `slack_search_public_and_private`.
4. Named channels: Resolve IDs through `slack_search_channels`, then call `slack_read_channel` with `limit` at `100` per channel.
5. Named people or DMs: Resolve people through `slack_search_users`, then use `slack_search_public_and_private` with small recent queries such as `from:<@USER_ID>`, `to:<@USER_ID>`, or `in:<@USER_ID>` to surface relevant DM or person-specific activity.
6. Search for messages that mention the user directly by querying `slack_search_public_and_private` with `<@USER_ID>` over a recent window. Mentions often contain direct asks, approvals, or work the user needs to pick up.
7. Named topics: Use `slack_search_public_and_private`, and if channels were also provided, keep the search inside those channels.
8. No explicit scope: Use `slack_search_public_and_private` with a few small recent queries such as `to:me after:YYYY-MM-DD`, `<@USER_ID> after:YYYY-MM-DD`, and `is:thread after:YYYY-MM-DD`, then expand the highest-signal hits with `slack_read_thread` for thread outcomes or `slack_read_channel` for surrounding channel context.
9. Use `slack_read_thread` when a result looks like a task, decision, blocker, escalation, or unanswered question and the search hit alone does not show the outcome or next step.
10. Prioritize direct asks, blockers, incidents, escalations, approvals, deadline changes, ownership changes, and messages that likely need a reply.

## Formatting

Format the triage as:

```md
*Slack Notification Triage — YYYY-MM-DD*

*Scope:* <channels + DMs + topics + coverage note>
*Summary:* <1–2 line overview of what the user most likely needs to read, reply to, or do next>

*Tasks for you*
- ...

*Worth skimming*
- ...

*Can ignore for now*
- ...

*Notes*
- <gaps, caveats, or partial coverage>
```

- Keep the triage compact; aim for 3–15 bullets total across all sections.
- Treat *Tasks for you* as the primary section whenever the triage is meant to produce a personal todo list.
- Include *Can ignore for now* only when the user explicitly asked to filter tasks.
- Start each bullet with the key update, then add the action the user may need to take when it is clear.
- Preserve exact channel names and mention DMs explicitly when helpful.
- Use *Notes* for coverage limits, sparse results, unsupported notification-state requests, or partial coverage.
