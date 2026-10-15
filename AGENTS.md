# AGENTS.md: org-wide protocol for AI agents

This is the single agent protocol for repositories in the `elsa-workflows` organisation and the `valence-works` repositories that adopt it. Each repository's own `AGENTS.md` links here and adds only repo-specific notes (build, structure, scope). If a repo note conflicts with this file, this file wins unless the repo note names an explicit owner exception.

You are one of several AI coding agents. GitHub issues and PRs are the only shared channel. Don't contact other agents directly. Your worker name is one of `codex`, `claude`, `cursor`, or your app slug.

## Issue Bus Lite

How agents find, claim and finish issues. Link here with `AGENTS.md#issue-bus-lite`.

### Labels

Every repo uses the same labels:

| Label | Colour | Meaning |
|---|---|---|
| `ready-for-agent` | `0E8A16` | Open for pickup |
| `blocked` | `B60205` | Not pickable; see the `blocked:` comment |
| `needs:decision` | `D93F0B` | Waiting on a maintainer decision; not pickable |
| `worker:codex` | `5319E7` | Lane: Codex |
| `worker:claude` | `8A63D2` | Lane: Claude |
| `worker:cursor` | `1D76DB` | Lane: Cursor (launched by a dispatcher, never self-polled) |
| `type:bug` | `D73A4A` | Something is broken |
| `type:feature` | `A2EEEF` | New capability |
| `type:docs` | `0075CA` | Documentation |
| `type:chore` | `CFD3D7` | Maintenance, tooling, refactoring |
| `type:question` | `D876E3` | Question, spike or discussion |
| `size:S` | `C2E0C6` | Small |
| `size:M` | `FEF2C0` | Medium |
| `size:L` | `F9D0C4` | Large; consider splitting |

An issue without a `worker:*` label is open to any self-polling worker. Don't take an issue labelled for another lane.

### Find work (oldest first)

```text
repo:<owner>/<repo> is:issue is:open no:assignee label:ready-for-agent -label:blocked -label:"needs:decision" sort:created-asc
```

Skip anything you can't verify end to end.

### Claim

1. Comment exactly `claim: <worker> starting`. If you post through the owner's account, your role tag may come first: `[Claude] claim: claude starting` is valid.
2. Re-read the comments. The earliest claim that hasn't been abandoned wins. If it isn't yours, comment `claim-abandoned` and stop: don't assign yourself, don't change labels, and leave the issue alone.
3. If you won, assign yourself and remove `ready-for-agent`. Assignment plus the missing label is the claim; there is no lease or board to update.

### Work

- One task per worker session and one PR per issue. A coordinator session that dispatches issues to separate worker sessions is fine.
- Branch `agent/<worker>/<issue>-<slug>` off the issue's target branch.
- Do exactly what the issue's objective and acceptance criteria say, and stay inside its scope.
- No deploys, releases, publishing, tags, public posts, secrets or workflow-permission changes.
- Prune what your change makes obsolete (dead code, stale tests, docs for retired features), or say in the PR why nothing is.
- Found extra work? Open a new issue. Don't widen your PR.

### Open the PR

Put `Fixes #<n>` in the PR body, with each acceptance criterion mapped to evidence. Then comment `pr: <url>` on the issue and request review by commenting `@elsa-workflows-bot review` on the PR. Workers never merge.

### Blocked or giving up

Comment `blocked: <reason>`, add `blocked` (or `needs:decision` if a decision is missing), and unassign yourself. Leave your branch and any draft PR for the next worker. To requeue, a maintainer removes `blocked` and re-adds `ready-for-agent`.

### Scheduled pickup routine

Self-polling workers (for example Claude and Codex on the maintainer's machine) run this as a scheduled routine, every one to two hours, locally so they post as their own app. Replace `<worker>` with the worker name (`claude`, `codex`, ...).

```text
You are the <worker> worker for Sipke's Elsa projects. Run one pickup cycle, then stop.

1. Read the "Issue Bus Lite" section of https://github.com/elsa-workflows/.github/blob/main/AGENTS.md and follow it exactly. It wins over anything below. Also follow the target repo's own AGENTS.md.
2. First, handle review findings on your own open PRs (author:<your app>): fix them in new commits, reply `Fixed in <sha>` or `Won't fix: <reason>` on each thread, and re-request review with `@elsa-workflows-bot review`. If any PR needed work, do that instead of claiming a new issue, then stop.
3. Find work, oldest first, in elsa-workflows/elsa-core, elsa-foundation, elsa-foundation-studio, elsa-foundation-designer, elsa-hub and elsa-blog:
   is:issue is:open no:assignee label:ready-for-agent -label:blocked -label:"needs:decision" sort:created-asc
   Skip issues labelled for another worker lane, issues that already have an open PR, anything that needs a secret, deploy, release, publish or public post, and anything in elsa-studio, elsa-extensions, elsa-apps or valence-works/groundwork.
4. If nothing qualifies, stop without posting anything.
5. Otherwise claim exactly one issue: comment `claim: <worker> starting`, re-read, and back off with `claim-abandoned` if someone claimed first. Then assign yourself and remove ready-for-agent.
6. Work on branch agent/<worker>/<issue>-<slug>, open one PR with `Fixes #<n>` and each acceptance criterion mapped to evidence, comment `pr: <url>` on the issue, and request review with `@elsa-workflows-bot review` on the PR. Never merge.
7. If you get stuck, follow the blocked steps (comment, label, unassign) and stop.

Post only as your own GitHub App, never as Sipke. Finish with a three-line summary: the issue, the PR, and its state.
```

## Review and merge rules
- An author never reviews, approves or merges its own PR, or one it contributed to. Anyone who committed to a PR counts as an author.
- "Never self-merge" means an author never merges its own PR. The Code Review app merging someone else's PR after its own approval is the intended flow, not a self-merge.
- Exactly one blocking reviewer per PR: the Code Review app. Other review tools (CodeRabbit, Copilot, Greptile, Bugbot) and other reviews (spec, standards) are advisory. The Code Review app always takes them into account, but the decision is its own: only findings it adopts block.
- Merges are done only by the repo's Code Review app (in elsa-workflows, `elsa-workflows-review-bot[bot]`, for every repo), after a separate, non-author approval on the **exact head commit** and green CI. New commits after an approval need a fresh approval.
- Review is a loop. If a round has actionable findings, they are fixed in new commits (no force-push), with `Fixed in <sha>` or `Won't fix: <reason>` on each thread, and once they are pushed the Code Review app reviews again.
- Merge trigger: the Code Review app merges automatically when its review leaves no actionable findings, with green CI on the exact head commit. No human merge is needed.
- At most 6 review rounds. If blocking issues remain after the sixth, the PR is escalated to the repo maintainer (Sipke): comment `blocked: review loop exhausted` and add `needs:decision`.
- A repo's `AGENTS.md` may name owner exceptions. (elsa-foundation ADR 0080 D7's human-merge cases were dropped by owner decision on 15 Oct 2026; those PRs follow the loop above.)
- An explicit instruction from the owner (Sipke) in an agent session counts as an exception for that one action only. Record it in the PR or release, for example `merged on Sipke's instruction`.
- Fallback: if the Code Review agent is unavailable (for example credits run out, or an outage), a maintainer (Sipke) may approve and merge manually. Agents pause and wait; no author merges its own PR.

## Stop conditions: stop and comment `blocked: <reason>` instead of continuing if
- The task needs an off-limits repo, a secret, a deploy, release, publish or public post.
- The acceptance criteria are contradictory or can't be tested.
- Required checks on the base branch are already red for reasons unrelated to your change. Non-required suites don't count.
- Your change would go beyond the issue's stated scope.
- You've failed the same check 3 times in a row.
- Someone else already has an open PR for the issue.

## Identities
- Each agent posts on GitHub as its own GitHub App, installed only on the repos it works in. Its comments show the login `<app-slug>[bot]` (for example `elsa-4-qa[bot]`). See the table under Mentions.
- In elsa-workflows, Claude sessions post as `elsa-workflows-claude[bot]`, Codex sessions as `elsa-workflows-codex[bot]`, and the other agents as `elsa-workflows-bot[bot]`. Only Code Review posts as `elsa-workflows-review-bot[bot]`.
- Posting as your app: no `[Role]` prefix. The mention router ignores comments by bot accounts, except a review request (`@elsa-workflows-bot review`) from one of these three author apps.
- Until an agent's app exists, it may post through the owner's account and must start each comment with its current role name (for example `[Training Engineer]`). The router ignores comments that start with a known role tag, except a review request (`@elsa-workflows-bot review`).
- When the router wakes you for a mention and your app can react there, it adds 👀 to the triggering comment as your app and passes `eyes_reaction.delete_url`. After replying, delete that reaction and add 👍 as your app.
- Never print, share or commit an app private key or other credential.

## Mentions

Mention an agent in an issue, PR or review comment on any router repo: elsa-workflows/elsa-core, elsa-workflows/elsa-foundation, elsa-workflows/elsa-foundation-studio, elsa-workflows/elsa-foundation-designer, elsa-workflows/elsa-hub, elsa-workflows/elsa-blog, elsa-workflows/.github, valence-works/bpmn, valence-works/elsa-control and valence-works/elsa-plus-training. A repo only routes once it has a webhook to the router; if a mention gets no 👀, tell the owner. Only trusted humans trigger the router, never bots (the one exception is an author app's review request). Team handles go to the team lead, who decides who handles it. Direct handles go to that agent and cc its lead. Routing lives in the router configuration, so it can change without a redeploy.

In `elsa-workflows`, use the single handle `@elsa-workflows-bot` (it reaches the repo's owning lead) and request review with `@elsa-workflows-bot review` (it reaches the Code Review agent). The old `@elsa-bot` handle still routes for now but is deprecated, because it's someone else's GitHub account. The `@vw-*` handles below are retired there and apply only in valence-works repos.

| Team handle | Lead | Direct handles (lead is cc'd) |
|---|---|---|
| `@vw-elsa4` | Elsa 4 Lead | `@vw-elsa4-lead`, `@vw-elsa4-qa`, `@vw-elsa4-ux` |
| `@vw-elsa3` | Elsa 3 Lead | `@vw-elsa3-lead`, `@vw-elsa3-engineer`, `@vw-elsa3-qa`, `@vw-elsa3-architect`, `@vw-elsa3-review`, `@vw-elsa3-release`, `@vw-elsa3-triage` |
| `@vw-elsa-control` | Elsa Control Lead | `@vw-elsa-control-lead`, `@vw-elsa-control-architect`, `@vw-elsa-control-qa`, `@vw-elsa-control-ux`, `@vw-elsa-control-review`, `@vw-elsa-control-launch` |
| `@vw-training` | Training Lead | `@vw-training-lead`, `@vw-training-engineer`, `@vw-training-curriculum`, `@vw-training-commercial`, `@vw-training-qa` |
| `@vw-integrations` | Integrations Lead | `@vw-integrations-lead`, `@vw-integrations-engineer` |
| `@vw-grok` | Grok (coordinator) | `@vw-architect` |

### Who's who

| Display name on GitHub | Bot login | Handle | Role |
|---|---|---|---|
| Elsa Workflows Bot | `elsa-workflows-bot[bot]` | `@elsa-workflows-bot` | Grok agents in elsa-workflows (leads, engineers, QA) |
| Elsa Workflows Review Bot | `elsa-workflows-review-bot[bot]` | `@elsa-workflows-bot review` | Code Review approvals and merges in elsa-workflows |
| Elsa Workflows Claude | `elsa-workflows-claude[bot]` | none | Claude sessions on Sipke's laptop |
| Elsa Workflows Codex | `elsa-workflows-codex[bot]` | none | Codex sessions |
| Elsa 4 Lead | `elsa-4-lead[bot]` | `@vw-elsa4`, `@vw-elsa4-lead` | Elsa 4 Crew lead |
| Elsa 4 QA | `elsa-4-qa[bot]` | `@vw-elsa4-qa` | Elsa 4 docs and guide QA |
| Elsa 4 UX | `elsa-4-ux[bot]` | `@vw-elsa4-ux` | Elsa 4 Studio UX |
| Elsa 3 Lead | `elsa-3-lead[bot]` | `@vw-elsa3`, `@vw-elsa3-lead` | Elsa 3 Crew lead |
| Elsa 3 Code Review | `elsa-workflows-review-bot[bot]` (acts via Elsa Workflows Review Bot) | `@vw-elsa3-review`; in elsa-workflows, `@elsa-workflows-bot review` | Elsa 3 code review |
| Elsa 3 Engineer | no app yet | `@vw-elsa3-engineer` | Elsa 3 engineering |
| Elsa 3 QA | no app yet | `@vw-elsa3-qa` | Elsa 3 QA |
| Elsa 3 Architect | no app yet | `@vw-elsa3-architect` | Elsa 3 architecture |
| Elsa 3 Release | no app yet | `@vw-elsa3-release` | Elsa 3 releases |
| Elsa 3 Triage | no app yet | `@vw-elsa3-triage` | Elsa 3 issue triage |
| Elsa Control Lead | `elsa-control-lead[bot]` | `@vw-elsa-control`, `@vw-elsa-control-lead` | Elsa Control Crew lead |
| Elsa Control Code Review | `elsa-control-code-review[bot]` | `@vw-elsa-control-review` | Elsa Control code review |
| Elsa Control Architect | no app yet | `@vw-elsa-control-architect` | Elsa Control architecture |
| Elsa Control QA | no app yet | `@vw-elsa-control-qa` | Elsa Control QA |
| Elsa Control UX | no app yet | `@vw-elsa-control-ux` | Elsa Control UX |
| Elsa Control Launch | no app yet | `@vw-elsa-control-launch` | Elsa Control launch |
| Training Lead (Valence) | `training-lead-valence[bot]` | `@vw-training`, `@vw-training-lead` | Training Crew lead |
| Training Engineer | no app yet | `@vw-training-engineer` | Training engineering |
| Training QA | no app yet | `@vw-training-qa` | Training kit QA |
| Training Curriculum | no app yet | `@vw-training-curriculum` | Training curriculum |
| Training Commercial | no app yet | `@vw-training-commercial` | Training commercial |
| Integrations Lead | `integrations-lead[bot]` | `@vw-integrations`, `@vw-integrations-lead` | Integrations Crew lead |
| Integrations Engineer | no app yet | `@vw-integrations-engineer` | Integrations engineering |
| Grok (Valence) | `grok-valence[bot]` | `@vw-grok` | Coordinator |
| Architect (Valence) | `architect-valence[bot]` | `@vw-architect` | Cross-team architect |

Shared team apps (`elsa-3-valence[bot]`, `elsa-control[bot]`, `training-valence[bot]`, `elsa-4-code-review[bot]`) have no handle of their own; mention the team handle instead.

Old unprefixed handles were dropped on 13 Oct 2026 (several were real GitHub accounts). Always use the `vw-` prefix.

Handles inside code spans or code blocks are ignored.

## Conduct
- No public posts (Discord, social, blog) and no contact with people outside the org on the owner's behalf.
- Be plain and concise. Put evidence (commands, output, screenshots, SHAs) on the PR or issue being worked, not in new issues.
- Escalate decisions to the owner briefly: add `needs:decision` and comment the question.
