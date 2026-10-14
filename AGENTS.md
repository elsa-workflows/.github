# AGENTS.md: org-wide protocol for AI agents

This is the single source of the agent collaboration protocol for repositories in the `elsa-workflows` organisation, and for the `valence-works` repositories that adopt it. Each repository's own `AGENTS.md` links here and adds only repo-specific notes (build, structure, scope, labels). If a repo note conflicts with this file, this file wins unless the repo note names an explicit owner exception.

You are one of several AI coding harnesses. GitHub issues and PRs are the only shared channel. Don't contact other harnesses directly. Your harness ID is one of: `grok-bot`, `codex`, `claude-code`, `gh-app-bot`, `cursor`.

## 1. Find work
Open issues labelled `agent:ready` with no `agent:claimed-*` label, no `agent:blocked` or `agent:needs-human`, and no `harness-only:` label for another harness. Take the oldest S first, then M. Skip anything you can't verify end to end.

## 2. Claim
Comment `/claim <you> lease=24h`, add `agent:claimed-<you>`, remove `agent:ready`. If someone else's `/claim` is earlier, back off: comment `/release <you> reason=duplicate-claim` and remove your label. The lease lasts 24h from your last push, PR update, or `/heartbeat <you> <status>` comment.

## 3. Branch
`agent/<you>/<issue>-<slug>`, off the issue's target branch. Add the trailer `Agent-Harness: <you>` to each commit.

## 4. Work
- Do exactly what the objective and acceptance criteria say. Stay inside the constraints and out-of-scope list.
- Never touch elsa-workflows/elsa-studio, elsa-extensions, or elsa-apps. Work for these goes to elsa-workflows/elsa-core.
- No deploys, releases, publishing, tags, public posts, secrets, or workflow-permission changes.
- **Prune:** remove what your change makes obsolete (dead code, stale tests, docs pointing at retired functionality), as listed in the issue's Pruning section. If nothing is obsolete, say why in the PR. The coordinator blocks merge without it.
- Found extra work? Open a new issue with the agent-task form. Don't widen your PR.

## 5. Open the PR
Open a **draft** PR early using the repo PR template: `Closes #N` (or `Refs #N` for a prune PR from a monthly cleanup sweep, which must not close the sweep issue), your harness, each AC mapped to evidence, test evidence, risks. Title: `<type>(<area>): <summary> (#N)`. Mark it ready and add `agent:in-review` only when all ACs have evidence and CI is green on the head commit.

## 6. Review and merge rules
- An author never reviews, approves, or merges its own PR, or one it contributed to. Any harness that committed to a PR counts as an author.
- Exactly one advisory reviewer per PR. Bugbot is advisory: only findings the Code Review agent adopts block.
- Merge happens only through the coordinator, after a separate, non-author Code Review approval on the **exact head commit** and green CI. New commits after an approval need a fresh approval.

### Respond to review
Feedback arrives as review threads and a coordinator `/route <you>` comment. Fix it in new commits (no force-push), reply `Fixed in <sha>` or `Won't fix: <reason>` on each thread, then comment `/ready <you>` and re-add `agent:in-review`. After 3 rounds the coordinator brings in a human.

## 7. Release a claim
If you can't finish and have an open PR, either hand it off (7a) instead of releasing, or close the PR (unmerged) first. To release, comment `/release <you> reason=<why + what you learned>`, remove your label, re-add `agent:ready`, and leave your branch for reference. If a decision is missing, use `/blocked <you> reason=...` or `/needs-human <you> question=...` instead.

## 7a. Hand off or pick up (credit-aware failover)
- Running out of credits/quota/time with useful work done? Push everything, open a **draft PR** if none exists yet, post `/handoff <you>` with *Done so far, Next steps, Branch, PR, Open review threads, Blockers*, then swap `agent:claimed-<you>` for `agent:handoff`. Leave the PR open.
- Picking up `agent:handoff`: comment `/claim-handoff <you> lease=24h`, swap the label for `agent:claimed-<you>`, continue on the **same branch and PR** from the latest note (don't restart), and add `Continued by <you> from <previous>` to the PR body.
- Silent expiry: the coordinator writes the handoff note for you.

## 8. Stop conditions: stop and comment instead of continuing if
- The task needs an off-limits repo, a secret, a deploy, release, publish, or public post.
- The ACs are contradictory or can't be tested.
- Base-branch CI is already red for reasons unrelated to your change.
- Your change would go beyond the issue's stated paths or scope.
- You've failed the same check 3 times in a row.
- Another harness has an open PR for the same issue (a PR you picked up via `/claim-handoff` is yours to continue).

## 9. Identities
- Each agent posts on GitHub as its own GitHub App, installed only on the repos it works in. Its comments show the app's display name and the login `<app-slug>[bot]` (for example `elsa-4-qa[bot]`). See the table in section 10.
- Posting as your app: no `[Role]` prefix. The mention router ignores comments by bot accounts, so the prefix isn't needed.
- Until an agent's app exists, it may post through the owner's account and must start each comment with its current role name (for example `[Training Engineer]`), never a retired one. The router ignores comments that start with a known role tag.
- When the router wakes you for a mention and your app can react there, it adds 👀 to the triggering comment, issue or PR as your app and passes `eyes_reaction.delete_url` in the payload. After posting your reply, delete that reaction and add 👍 as your app.
- An app with read-only code access can't merge; merges then go through the owner's account under the rules in section 6.
- Never print, share, or commit an app private key or routine key.

## 10. Mentions

Mention an agent in an issue, PR, or review comment (training repo only for now; only trusted humans, never bots). Team handles go to the team lead, who decides who handles it. Direct handles go to that agent and copy (cc) its lead. Routing lives in Supabase `agent_router_config` (key `handles`), so it can be changed without a redeploy.

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

Internal agent ids are the agent name in UPPER_SNAKE (e.g. `ELSA_CONTROL_LEAD`, `TRAINING_QA`, `INTEGRATIONS_ENGINEER`, `ELSA3_REVIEW` for Code Review). Each agent's routine key is the Edge Functions secret `<ID>_ROUTINE_KEY`.

Handles inside code spans or code blocks are ignored.

## 11. Conduct
- No public posts (Discord, social, blog) and no contact with people outside the org on the owner's behalf.
- Be plain and concise. Put evidence (commands, output, screenshots, SHAs) on the PR or issue being worked, not in new issues.
- Escalate decisions to the owner briefly, through the coordinator, with `/needs-human`.

## Issue Bus Lite

A lightweight way for agents to pick up, claim and finish issues. Link here from a repo's AGENTS.md with `#issue-bus-lite`. Based on the full [Issue Bus](https://github.com/valence-works/elsa-control/blob/main/docs/product/issue-bus.md), minus the Project board sync, lease expiry and drift checks.

### Labels
- `ready-for-agent`: open for pickup.
- `blocked`, `needs:decision`: not pickable.
- `worker:codex`, `worker:claude`, `worker:cursor`: lane. No lane label = any self-polling worker (Codex, Claude). Cursor sessions are launched only for `worker:cursor` issues.
- `type:bug|feature|docs|chore|question`, `size:S|M|L`.

### Pickup (oldest first)
```text
repo:<owner>/<repo> is:issue is:open no:assignee label:ready-for-agent -label:blocked -label:"needs:decision" sort:created-asc
```
Skip issues labelled for another worker lane.

### Claim
1. Comment exactly `claim: <worker> starting`.
2. Re-read the comments. The earliest active claim wins. If you lost, comment `claim-abandoned` and stop.
3. If you won, assign yourself and remove `ready-for-agent`. Assignment plus the missing label is the claim state.

### Work
- One task per session, one PR per issue; the PR body contains `Fixes #<n>`.
- When the PR opens, comment `pr: <url>` on the issue.
- If stuck, comment `blocked: <reason>`, add `blocked`, and unassign yourself.
- Workers never merge. Review and merge follow section 6 above.

To requeue, a maintainer removes `blocked` and re-adds `ready-for-agent`.
