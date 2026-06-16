---
name: summarize-work-context
description: Create brief cross-source summaries from Gmail, Slack, Jira, and Bitbucket for a specific topic, task name, issue key, pull request, branch, commit, or user. Use when Codex needs to gather recent context across work communications and development systems, attribute each fact to its source and sender, and return a concise summary of at most 10 lines.
---

# Summarize Work Context

## Purpose

Summarize a specific topic, task, issue, pull request, branch, commit, or user by searching Gmail, Slack, Jira, and Bitbucket when those connectors are available. Keep the final answer brief: at most 10 lines total.

## Inputs

Accept any of these as the search target:

- Topic or phrase, such as a project name, bug symptom, customer name, feature name, or incident.
- Task identifier, such as a Jira key, branch name, commit hash, pull request title, or PR number.
- User, such as a name, email, Slack handle, Jira account, or Bitbucket username.

If the user gives no time range, default to the last 30 days. Widen only when results are too sparse or the user asks for historical context.

## Source Workflow

1. Search Gmail for matching email subjects, senders, recipients, and thread text. Read matching emails only as needed for summary evidence, and if reading them changes their read/unread state, mark them unread again after extracting the needed context. Treat the sender as the email `From` person.
2. Search Slack only in channels and small group conversations connected to a channel context, such as public channels, accessible private channels, group channels, and relevant threads. Do not search one-to-one DMs unless the user explicitly asks for DMs and grants any required consent. Treat the sender as the Slack message author.
3. Search Jira with Rovo Search or JQL. Fetch candidate issues before summarizing. Treat the sender as the reporter for issue-level facts and the comment author for comment-level facts.
4. Search Bitbucket for matching pull requests, branches, commits, and comments. Treat the sender as the PR author, commit author, or comment author.
5. Deduplicate repeated facts across sources and prefer the newest, most direct evidence.
6. Do not invent missing connector results. If a source is unavailable or inaccessible, say so in one short line only if it materially affects the summary.

## Evidence Rules

Every factual line that cites discovered work context must include:

```text
Source | Sender | Detail
```

Use the concrete source name: `Gmail`, `Slack`, `Jira`, or `Bitbucket`.

Include dates, issue keys, PR numbers, branch names, or channel names only when they help identify the evidence. Avoid quoting long message bodies, email text, ticket descriptions, or comments.

## Output Format

Return at most 10 lines total. Prefer this shape:

```text
Summary: <one-line synthesis>
Gmail | <sender> | <brief relevant fact>
Slack | <sender> | <brief relevant fact>
Jira | <sender> | <brief relevant fact>
Bitbucket | <sender> | <brief relevant fact>
Next: <one-line suggested follow-up, only when useful>
```

If there are many results, include only the highest-signal 4 to 8 evidence lines. If there are no useful results, return one line saying no relevant context was found and list the sources searched.

## Guardrails

- Keep the final answer to 10 lines or fewer, even when many results exist.
- Keep summaries factual and evidence-backed.
- Do not expose private message contents beyond the minimum needed to summarize the task.
- For Gmail, preserve the user's mailbox state: if a relevant unread email is opened/read during analysis, mark it unread again after summarizing.
- For Slack, stay scoped to channels, group channels, and their threads by default; avoid one-to-one DMs unless explicitly requested.
- Do not include credentials, tokens, raw connector payloads, or unrelated personal data.
- Do not write to Gmail, Slack, Jira, or Bitbucket unless the user explicitly asks for a follow-up action.
- Mention uncertainty when sources disagree or when access is incomplete.
