---
name: dependabot-prs
description: Review and merge open Dependabot pull requests
disable-model-invocation: true
allowed-tools: Bash, Read, Agent
---

# Dependabot PR Review and Merge

Follow these steps:

## 1. Load per-repo preferences

Read `~/.claude/dependabot-prs.json` (if it exists) and look up the current repo's entry:

```bash
config="$HOME/.claude/dependabot-prs.json"
repo=$(gh repo view --json nameWithOwner --jq .nameWithOwner)
preferences=""
if [ -f "$config" ]; then
  preferences=$(jq -r --arg repo "$repo" '.[$repo] // empty' "$config")
fi
```

The file maps `owner/repo` to freeform instructions, e.g.:

```json
{
  "ryanb/core-gem": "Only handle PRs with the javascript label."
}
```

If `$preferences` is non-empty, apply those instructions throughout the rest of the steps — for example, filtering the PR list, narrowing what to merge, or changing the merge strategy.

## 2. List open Dependabot PRs

Run `gh pr list --author "dependabot[bot]" --state open` to get all open Dependabot PRs.

If there are no open PRs, tell the user and stop.

## 3. Review each PR in a sub-agent

For each PR, launch a sub-agent (run them in parallel) to review it. Each sub-agent should:

1. Get the PR details: `gh pr view <number> --json title,body,statusCheckRollup,mergeable`
2. Check if all CI checks passed (all statusCheckRollup entries have conclusion "SUCCESS")
3. Read the changelog/release notes in the PR body for anything surprising:
   - Breaking changes or deprecations
   - Dropped support for language/framework versions this codebase uses
   - New behaviors that could cause issues (e.g. new errors being raised)
   - Security fixes worth highlighting
4. Return a summary with: PR number, title, CI status (pass/fail), whether it's safe to merge, and any concerns

## 4. Report findings to the user

Present a table summarizing all PRs:
- PR number and title
- CI status (pass/fail)
- Whether it looks safe to merge
- Any notable changes or concerns

## 5. Ask the user which PRs to merge

Ask the user which PRs they'd like to merge. Wait for their response.

## 6. Merge approved PRs

For each PR the user wants merged, one at a time:
1. Approve: `gh pr review <number> --approve`
2. Merge: `gh pr merge <number> --merge`

Report the result of each merge.
