---
name: dependabot-prs
description: Review and merge open Dependabot pull requests
disable-model-invocation: true
allowed-tools: Bash(gh *), Agent
---

# Dependabot PR Review and Merge

Follow these steps:

## 1. List open Dependabot PRs

Run `gh pr list --author "dependabot[bot]" --state open` to get all open Dependabot PRs.

If there are no open PRs, tell the user and stop.

## 2. Review each PR in a sub-agent

For each PR, launch a sub-agent (run them in parallel) to review it. Each sub-agent should:

1. Get the PR details: `gh pr view <number> --json title,body,statusCheckRollup,mergeable`
2. Check if all CI checks passed (all statusCheckRollup entries have conclusion "SUCCESS")
3. Read the changelog/release notes in the PR body for anything surprising:
   - Breaking changes or deprecations
   - Dropped support for language/framework versions this codebase uses
   - New behaviors that could cause issues (e.g. new errors being raised)
   - Security fixes worth highlighting
4. Return a summary with: PR number, title, CI status (pass/fail), whether it's safe to merge, and any concerns

## 3. Report findings to the user

Present a table summarizing all PRs:
- PR number and title
- CI status (pass/fail)
- Whether it looks safe to merge
- Any notable changes or concerns

## 4. Ask the user which PRs to merge

Ask the user which PRs they'd like to merge. Wait for their response.

## 5. Merge approved PRs

For each PR the user wants merged, one at a time:
1. Approve: `gh pr review <number> --approve`
2. Merge: `gh pr merge <number> --merge`

Report the result of each merge.
