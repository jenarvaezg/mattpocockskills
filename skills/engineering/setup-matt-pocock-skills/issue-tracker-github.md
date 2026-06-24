# Issue tracker: GitHub

Issues and PRDs for this repo live as GitHub issues. Use the `gh` CLI for all operations.

## Conventions

- **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line bodies.
- **Create a sub-issue**: `gh issue create --parent <parent-number-or-url> --title "..." --body "..."`. Creates the child and attaches it to the parent's native sub-issue list in one call, so it shows up in the parent's progress indicator instead of only being cross-linked from the body.
- **Read an issue**: `gh issue view <number> --comments`, filtering comments by `jq` and also fetching labels.
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` with appropriate `--label` and `--state` filters.
- **Comment on an issue**: `gh issue comment <number> --body "..."`
- **Apply / remove labels**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **Close**: `gh issue close <number> --comment "..."`

Infer the repo from `git remote -v` — `gh` does this automatically when run inside a clone.

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same labels and states as issues, using the `gh pr` equivalents:

- **Read a PR**: `gh pr view <number> --comments` and `gh pr diff <number>` for the diff.
- **List external PRs for triage**: `gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments` then keep only `authorAssociation` of `CONTRIBUTOR`, `FIRST_TIME_CONTRIBUTOR`, or `NONE` (drop `OWNER`/`MEMBER`/`COLLABORATOR`).
- **Comment / label / close**: `gh pr comment`, `gh pr edit --add-label`/`--remove-label`, `gh pr close`.

GitHub shares one number space across issues and PRs, so a bare `#42` may be either — resolve with `gh pr view 42` and fall back to `gh issue view 42`.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --comments`.

## When a skill says "create a sub-issue"

Run `gh issue create --parent <parent-number-or-url> ...`. The child then appears in the parent's sub-issue list and progress indicator, not just as a `## Parent` body cross-link.

If your `gh` predates the `--parent` flag (or the target instance does not accept it), create the issue normally and attach it afterwards via the sub-issues REST API, which `gh` does not expose as a porcelain command:

```bash
child_id=$(gh api repos/OWNER/REPO/issues/<child-number> -q .id)
gh api -X POST repos/OWNER/REPO/issues/<parent-number>/sub_issues -F sub_issue_id=$child_id
```

`sub_issue_id` is the child's node `id` (from `gh api .../issues/<n> -q .id`), not its issue number.
