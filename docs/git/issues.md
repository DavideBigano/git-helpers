# GitHub Issues

Issues serve as the project backlog. Capture any out-of-scope work as an issue rather than letting it get lost.

- [Creating an issue](#creating-an-issue)
- [Listing open issues](#listing-open-issues)
- [Referencing issues](#referencing-issues)

## Creating an issue

```
gh issue create --title "[Type] Declarative issue name" --body "<body>"
```

Types follow the same convention as branches:
- `Enhancement`
- `Refactor`
- `Maintenance`
- `Bugfix`
- `Cleanup`

The body should briefly explain what needs doing and why. Enough context for someone picking it up cold.

### Discussion issues

For unresolved design questions, use `--label discussion` instead of a type prefix:

```
gh issue create --label discussion --title "[Enhancement] <question or decision to make>" --body "<body>"
```

Discussion issues capture open questions that need a decision before work can begin. They are not tasks — they are prompts for a design conversation. Use `[Enhancement]` as the title type since most design questions lead to an addition or change.

To create the `discussion` label if it doesn't exist:

```
gh label create discussion --color "#e4e669" --description "Open design question requiring a decision"
```

## Listing open issues

```
gh issue list
```

## Referencing issues

Link an issue in a PR body or commit message with `#<issue-number>`. Use `Closes #<issue-number>` in a PR body to auto-close the issue when the PR merges.
