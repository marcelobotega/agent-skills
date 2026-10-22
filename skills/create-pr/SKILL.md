---
name: create-pr
description: >-
  Create pull requests or write their descriptions with clear motivation, relevant evidence,
  and review risks when opening PRs, updating PR descriptions, or preparing changes for review.
license: MIT
compatibility: Requires GitHub CLI (gh) authenticated and available for publishing
metadata:
  version: "1.3.0"
---

# Create Pull Request

Help reviewers understand the purpose, assess correctness, and identify unresolved concerns.

## Prerequisites

- Read applicable repository instructions and PR templates. Follow explicit user and repository
  requirements before this skill's defaults, including validation order and required sections.
- Look for templates in `.github/`, `docs/`, and the repository root, including template
  directories. Select the applicable template.
- Confirm whether the request is description-only, a new PR, or an update to an existing PR.
  Default new PRs to draft unless the user or repository specifies otherwise.
- Respect the user's authorization for commits, pushes, and remote PR changes. A request to draft
  a description alone does not authorize publication.
- Preserve unrelated user work and published history. Follow the repository's branch-update
  policy; do not automatically rebase, amend, or force-push a shared branch.
- Keep one PR per logical change.

## Process

### Step 1: Confirm the Target and Change Scope

Inspect the repository, current branch, and working tree:

```bash
git status --short --branch
git remote -v
```

For a local description draft or a new PR with uncommitted work, inspect the intended committed
changes and relevant staged, unstaged, and untracked files. Confirm scope instead of including
unrelated user work:

```bash
git diff --cached
git diff
git ls-files --others --exclude-standard
```

Read the relevant untracked files; listing their names does not reveal their contents.
Use available local base refs for committed changes. Do not require GitHub access for a local
description draft; state any comparison limitations if the base is unavailable.

For remote operations, confirm `REPO` as the target `owner/repository`, not necessarily the push
destination. For an existing PR, resolve its number or URL first, inspect its body and base/head
metadata with `gh pr view --repo "$REPO"`, and analyze its diff with `gh pr diff --repo "$REPO"`.
Pass the explicit PR identifier to both commands; use that PR's scope, not an unrelated checkout.

For a new PR, confirm the base branch and its remote, fetch it, and review the remote-based diff:

```bash
REPO='<target-owner>/<repository>'
BASE='<confirmed-base-branch>'
BASE_REMOTE='<remote-for-target-repository>'
gh repo view "$REPO" --json nameWithOwner,defaultBranchRef
git fetch "$BASE_REMOTE"
BASE_REF="$BASE_REMOTE/$BASE"
git log "$BASE_REF"..HEAD --oneline
git diff "$BASE_REF"...HEAD
```

Use the requested base or the target repository's default. If the remote-tracking base is
missing, fetch that branch explicitly before continuing. Check whether the intended head
already has a PR in the target repository; confirm the update scope if one exists.

### Step 2: Understand the Change and Choose a Title

Read the commits and diff. Identify the problem, affected behavior or consumers, motivation,
and any compatibility or operational consequences. Consult linked issues when available;
record alternatives only when they were actually considered.

Choose a conventional-commit title using the repository's scopes and release policy:

| Type | Use |
| --- | --- |
| `fix` | Correct behavior or ship a dependency fix requiring a patch release |
| `feat` | Introduce a feature requiring a minor release |
| `refactor` | Restructure implementation without changing behavior |
| `chore` | Maintain tooling or configuration without requiring a package release |

Follow repository rules for other types and breaking changes. Dependency changes do not
automatically imply `chore`; check their release impact, especially for squash-merge titles.

```text
fix(client): update dependencies to address security advisories
```

### Step 3: Write the Description

Lead with what changed and why. Add issue links, meaningful alternatives, and reviewer context
that the diff does not explain. Use the repository's domain terminology.

Scale the presentation to the change rather than filling a fixed template:

| Change | Presentation |
| --- | --- |
| Small, straightforward | A few short paragraphs |
| Complex behavior or structure | Add headings and a compact visual if it improves understanding |
| Risky or operational | Explain affected consumers, compatibility, rollout, and rollback limits |

#### Evidence

Include concrete evidence when it supports a review decision:

- A regression test that fails before the fix and passes afterward.
- Screenshots for visual changes, with sensitive information removed.
- Measured behavior, performance, or security improvements, with their scope stated.

Use before/after comparisons only when both sides are supported. Distinguish local results
from CI, and completed validation from pending, failed, or blocked checks. State limitations
instead of inventing measurements, passing checks, or screenshots.

Summarize relevant validation outcomes rather than duplicating the CI check list. Keep testing
plans and step-by-step checklists out of the description unless the user or repository requires
them; useful validation results still belong when they inform the review.

#### Visuals and Risks

Use the smallest visual that explains relationships or behavior more clearly than prose:
a call tree, diff sketch, file tree, sequence diagram, or screenshot. Show only the relevant
parts. A command history or changed-file list alone rarely explains the purpose of a PR.

Explain material risks in concrete terms: who is affected, what could fail, what remains
unresolved, and how rollout or rollback works. Distinguish reverting code from reversing
data changes or effects already observed by consumers. Omit generic risk labels and empty
sections; headings such as `Evidence` or `Risks` are optional.

For contrasting examples of small fixes, dependency updates, complex changes, and migrations,
read [Description Examples](references/description-examples.md) when choosing the presentation.
The examples illustrate style, not evidence to copy into another PR.

For description-only requests, present the title and body in the conversation and stop here.

### Step 4: Publish or Update the PR

For code publication, run the checks required by the repository or requested by the user before
publishing by default. For a requested draft before validation, publish with checks marked
pending, then run those checks and update the description with observed results and blockers.
Follow repository requirements when they mandate validation first. Keep the PR a draft until
a state change is authorized. Metadata-only edits do not require new commits, a push, or
unrelated validation.

Before publishing code, ensure only the intended changes are committed. Use `conventional-commit`
with the required authorization when available, or follow repository commit conventions directly.
Recheck that the committed diff matches the reviewed scope. For an existing PR, verify the local
branch and push destination match its head. Confirm the remote and push with required approval:

```bash
PUSH_REMOTE='<remote-for-head-repository>'
HEAD_BRANCH='<confirmed-head-branch>'
EXPECTED_HEAD=$(git rev-parse HEAD)
git push --set-upstream "$PUSH_REMOTE" "HEAD:refs/heads/$HEAD_BRANCH"
git ls-remote --exit-code "$PUSH_REMOTE" "refs/heads/$HEAD_BRANCH"
```

Confirm the reported remote commit matches `EXPECTED_HEAD`; stop on a mismatch or push failure.
For a fork PR, push to the fork and set `PR_HEAD` to `fork-owner:branch`; otherwise use the head
branch name.

Write the one-line title and the description to separate UTF-8 files in an approved temporary
location outside tracked project files. Use a file-writing tool or a quoted heredoc (`<<'EOF'`)
so backticks and `$()` stay literal while creating the files. Read the title as a quoted argument;
pass the description with `--body-file`. Include required template content yourself rather than
relying on gh to apply a template when a body file is supplied.

```bash
TITLE_FILE='<absolute-path-to-title.txt>'
BODY_FILE='<absolute-path-to-description.md>'
gh pr create --repo "$REPO" --base "$BASE" --head "$PR_HEAD" --draft \
  --title "$(cat "$TITLE_FILE")" --body-file "$BODY_FILE"
```

Omit `--draft` only when a ready PR is requested or required by repository policy.

For an existing PR, start from its fetched body and edit only the agreed content. Re-fetch
immediately before submitting and merge any intervening human edits instead of overwriting them.
Use the confirmed numeric `PR_NUMBER` and target repository:

```bash
gh pr edit "$PR_NUMBER" --repo "$REPO" --body-file "$BODY_FILE"
```

Change the title only when requested or required by the agreed scope. If an older GitHub CLI
fails with the Projects classic deprecation error, prefer upgrading it. Otherwise use the
REST API with file-based input rather than interpolating the Markdown into a shell string:

```bash
gh api --method PATCH "repos/$REPO/pulls/$PR_NUMBER" \
  --field body=@"$BODY_FILE"
```

For an authorized title edit, also pass `--title "$(cat "$TITLE_FILE")"` to `gh pr edit`, or
`--raw-field title="$(cat "$TITLE_FILE")"` to the REST command.
Otherwise leave the title unchanged.

After creation, use the returned PR number or URL to read it back. After editing, read back
the same PR. Confirm the title, body, base, head, `headRefOid`, and draft state in the target
repository. For code publication, compare `headRefOid` to `EXPECTED_HEAD`; for metadata-only
edits, confirm the head and draft state stayed unchanged. Report the URL and any pending or
blocked validation; mark checks successful only when their results are available.
