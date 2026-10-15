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

Put `Fixes #<n>` in the PR body, with each acceptance criterion mapped to evidence. Then comment `pr: <url>` on the issue and request review by commenting `@elsa-bot review` on the PR. Workers never merge.

### Blocked or giving up

Comment `blocked: <reason>`, add `blocked` (or `needs:decision` if a decision is missing), and unassign yourself. Leave your branch and any draft PR for the next worker. To requeue, a maintainer removes `blocked` and re-adds `ready-for-agent`.

## Review and merge rules
- An author never reviews, approves or merges its own PR, or one it contributed to. Anyone who committed to a PR counts as an author.
- Exactly one blocking reviewer per PR: the Code Review app. Other reviews (spec, standards, CodeRabbit, Copilot, Greptile, Bugbot) are advisory: the Code Review app weighs them, but the decision is its own, and only findings it adopts block.
- Merges are done only by the repo's Code Review app (in elsa-workflows, `elsa-workflows-review-bot[bot]`, for every repo), after a separate, non-author approval on the **exact head commit** and green CI. It merges automatically once its review finds nothing actionable. New commits after an approval need a fresh approval.
- Fix review feedback in new commits (no force-push) and reply `Fixed in <sha>` or `Won't fix: <reason>` on each thread. Review is a loop: findings are fixed and pushed, then re-reviewed, for up to 6 rounds. If blocking issues remain after that, the PR is escalated to the owner (Sipke).
- Automatic merge covers every PR, including runtime, persistence and public-rename PRs (elsa-foundation ADR 0080 D7 no longer needs the owner's merge). A repo's `AGENTS.md` may still name other owner exceptions.
- An explicit instruction from the owner (Sipke) in an agent session counts as an exception for that one action only. Record it in the PR or release, for example `merged on Sipke's instruction`.
- Fallback: if the Code Review agent is unavailable (for example credits run out, or an outage), a maintainer (Sipke) may approve and merge manually. Agents pause and wait; they never self-merge.

## Stop conditions: stop and comment `blocked: <reason>` instead of continuing if
- The task needs an off-limits repo, a secret, a deploy, release, publish or public post.
- The acceptance criteria are contradictory or can't be tested.
- Required checks on the base branch are already red for reasons unrelated to your change. Non-required suites don't count.
- Your change would go beyond the issue's stated scope.
- You've failed the same check 3 times in a row.
- Someone else already has an open PR for the issue.

## Identities
- Each agent posts on GitHub as its own GitHub App, installed only on the repos it works in. Its comments show the login `<app-slug>[bot]` (for example `elsa-4-qa[bot]`). See the table under Mentions.
- Posting as your app: no `[Role]` prefix. The mention router ignores comments by bot accounts.
- Until an agent's app exists, it may post through the owner's account and must start each comment with its current role name (for example `[Training Engineer]`). The router ignores comments that start with a known role tag, except a review request (`@elsa-bot review`).
- When the router wakes you for a mention and your app can react there, it adds 👀 to the triggering comment as your app and passes `eyes_reaction.delete_url`. After replying, delete that reaction and add 👍 as your app.
- Never print, share or commit an app private key or other credential.

## Mentions

Mention an agent in an issue, PR or review comment on any router repo: elsa-workflows/elsa-core, elsa-workflows/elsa-foundation, valence-works/bpmn, valence-works/elsa-control and valence-works/elsa-plus-training. Only trusted humans trigger the router, never bots. Team handles go to the team lead, who decides who handles it. Direct handles go to that agent and cc its lead. Routing lives in the router configuration, so it can change without a redeploy.

In `elsa-workflows`, use the single handle `@elsa-bot` (it reaches the repo's owning lead) and request review with `@elsa-bot review` (it reaches the Code Review agent). The `@vw-*` handles below are retired there and apply only in valence-works repos.

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
| Elsa 4 Lead | `elsa-4-lead[bot]` | `@vw-elsa4`, `@vw-elsa4-lead` | Elsa 4 Crew lead |
| Elsa 4 QA | `elsa-4-qa[bot]` | `@vw-elsa4-qa` | Elsa 4 docs and guide QA |
| Elsa 4 UX | `elsa-4-ux[bot]` | `@vw-elsa4-ux` | Elsa 4 Studio UX |
| Elsa 3 Lead | `elsa-3-lead[bot]` | `@vw-elsa3`, `@vw-elsa3-lead` | Elsa 3 Crew lead |
| Elsa 3 Code Review | `elsa-3-code-review[bot]` | `@vw-elsa3-review` | Elsa 3 code review |
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
