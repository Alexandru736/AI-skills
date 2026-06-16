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

## Search Strategy

Build search variants before calling connectors:

- Use the original user input exactly.
- For Jira keys and task names, also try hyphen, underscore, and spaced variants, such as `MKT-532`, `MKT_532`, and `MKT 532`.
- When a ticket or PR title is discovered, search the exact title and normalized title variants with hyphens replaced by underscores and spaces.
- For long titles, search the most distinctive phrase as well as the full title.

## Source Workflow

1. Search Gmail through the Codex Gmail integration when it is available. Search matching email subjects, senders, recipients, and thread text. Read matching emails only as needed for summary evidence, and if reading them changes their read/unread state, mark them unread again after extracting the needed context. Treat the sender as the email `From` person.
2. Search Slack through the Codex Slack integration when it is available. Search only in channels and small group conversations connected to a channel context, such as public channels, accessible private channels, group channels, and relevant threads. Do not search one-to-one DMs unless the user explicitly asks for DMs and grants any required consent. Treat the sender as the Slack message author.
3. Search Jira through the Codex Atlassian/Jira integration when it is available, using Rovo Search or JQL as appropriate. Fetch candidate issues before summarizing. Treat the sender as the reporter for issue-level facts and the comment author for comment-level facts.
4. Search Bitbucket for matching pull requests, branches, commits, and comments. Prefer a Bitbucket connector if one is available; otherwise frame the work as a concrete Bitbucket PR/repository lookup so any applicable local Bitbucket capability can trigger from its own metadata before using any manual API fallback. Treat the sender as the PR author, commit author, or comment author.
5. Deduplicate repeated facts across sources and prefer the newest, most direct evidence.
6. Do not invent missing connector results. If a source is unavailable or inaccessible, say so in one short line only if it materially affects the summary.

## Connector Access

Use built-in Codex connectors before local fallbacks.

For Gmail:

- Try the Codex Gmail integration first.
- Search before reading full messages.
- Track which matching Gmail messages were unread before reading them, and mark those messages unread again after summarizing if the integration changes read state.
- If the Gmail integration is unavailable, report Gmail as unavailable in the platform summary only when that absence materially affects the answer.

For Slack:

- Try the Codex Slack integration first.
- Use Slack search for broad topic matching, then read relevant channels or threads only when more context is needed.
- Prefer public channels and channel-linked small groups. Search private channels only when accessible and appropriate; avoid one-to-one DMs unless the user explicitly requests DMs and grants any required consent.
- Preserve source context such as channel name, thread, and message author when it helps identify the evidence.
- If the Slack integration is unavailable, report Slack as unavailable in the platform summary only when that absence materially affects the answer.

For Jira:

- Try the Codex Atlassian/Jira integration first.
- Use Rovo Search for broad matching unless the user asks for JQL or the task needs precise issue filtering.
- Use JQL for exact issue keys, project/status filters, assignee/reporter searches, and date-bounded issue searches.
- Fetch issue details before summarizing so status, reporter, assignee, comments, and descriptions are attributed correctly.
- If the Jira integration is unavailable, report Jira as unavailable in the platform summary only when that absence materially affects the answer.

For Bitbucket:

- Try a Bitbucket connector first if one is available.
- If no connector is available, phrase the next step as an ordinary Bitbucket PR/repository task, such as "look up PRs for this source branch", "read Bitbucket PR comments", "summarize commits on this Bitbucket branch", or "inspect a Bitbucket diff". Let the environment auto-trigger the applicable Bitbucket capability from that task description.
- Use the Bitbucket Cloud API with the local macOS Keychain token only as a fallback when no Bitbucket capability is available or when it cannot answer the specific read-only lookup.
- Retrieve any fallback token with `security find-generic-password` only into an environment variable or command substitution used immediately by the API call. Try likely service/account labels such as `bitbucket`, `bitbucket.org`, `api.bitbucket.org`, or `BITBUCKET_TOKEN` when the exact Keychain label is not known.
- Never print, log, store, commit, or include the Bitbucket token in summaries.
- If multiple Keychain items might match, inspect item metadata only; do not dump secret values while discovering the correct item.
- Use read-only API calls for repository, PR, branch, commit, diff, and comment lookup unless the user explicitly asks for a write action.

## Evidence Rules

Every factual line that cites discovered work context must include:

```text
Source | Sender | Detail
```

Use the concrete source name: `Gmail`, `Slack`, `Jira`, or `Bitbucket`.

Include dates, issue keys, PR numbers, branch names, or channel names only when they help identify the evidence. Avoid quoting long message bodies, email text, ticket descriptions, or comments.

## Output Format

Return Markdown at most 10 lines total. Summarize per platform, not per comment or per raw search hit. Prefer one concise line per platform, combining the highest-signal facts and sender names for that platform.

```markdown
**Summary:** <one-line synthesis>
- **Gmail** | <sender(s)> | <brief relevant platform summary>
- **Slack** | <sender(s)> | <brief relevant platform summary>
- **Jira** | <sender(s)> | <brief relevant platform summary>
- **Bitbucket** | <sender(s)> | <brief relevant platform summary>
**Next:** <one-line suggested follow-up, only when useful>
```

If there are many results, collapse them into the most relevant platform-level summaries. Do not list every comment, email, Slack message, commit, or PR comment. If there are no useful results, return one Markdown line saying no relevant context was found and list the sources searched.

## Guardrails

- Keep the final answer to 10 lines or fewer, even when many results exist.
- Use Markdown for the final answer.
- Summarize per platform and prioritize relevance over completeness.
- Keep summaries factual and evidence-backed.
- Do not expose private message contents beyond the minimum needed to summarize the task.
- For Gmail, preserve the user's mailbox state: if a relevant unread email is opened/read during analysis, mark it unread again after summarizing.
- For Slack, stay scoped to channels, group channels, and their threads by default; avoid one-to-one DMs unless explicitly requested.
- Do not include credentials, tokens, raw connector payloads, or unrelated personal data.
- Do not write to Gmail, Slack, Jira, or Bitbucket unless the user explicitly asks for a follow-up action.
- Mention uncertainty when sources disagree or when access is incomplete.
