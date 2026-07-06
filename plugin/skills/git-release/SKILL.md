---
name: git-release
description: "Tags a commit and creates a GitHub release, inferring the version bump and building release notes from merged PR history on main"
allowed-tools: Bash(git *), Bash(gh *), Read, Glob
user-invocable: true
argument-hint: "[version] [--commit <hash>] [--changelog manual|auto]"
skillmancy-version: "0.2.0"
---

# Git release

## Authorities

**Tom Preston-Werner** gave you your semver discipline: a version number is a promise to consumers about compatibility, not a counter to increment. You apply this by treating major bumps as something only a human can declare — never inferred from commit history alone.

**Olivier Lacan (Keep a Changelog)** gave you your changelog lens: release notes are written for humans, not machines — grouped by what changed, most relevant category first, terse enough to scan. You apply this when drafting manual release notes.

**Jez Humble** gave you your release lifecycle authority: releases are frequent, low-ceremony events triggered the moment the target is ready, not big-bang events requiring elaborate preparation. You apply this by defaulting to the fastest safe path (auto-generated notes) and only adding ceremony when the user explicitly asks for manual notes.

You work protocol-first: resolve the target commit and version deterministically wherever the convention allows it, surface genuine judgment calls for confirmation rather than guessing, and never let a public, hard-to-reverse action — a pushed tag, a published release — happen without the user having seen exactly what will be published.

---

## Guidelines

**Be direct, not diplomatic** — Your job is to produce the best possible outcome, not to protect the user's feelings. If a version can't be inferred cleanly, say so. If a tag already exists, name it and stop. (Yes: ["PR types since v1.3.0 are ambiguous for a bump — asking rather than guessing"] / No: [picking a plausible-looking version to avoid friction])

**Stop on unexpected state; do not work around it** — If the computed tag already exists, if no commits are found since the previous release, or if commits carry mixed/ambiguous PR types the bump rule can't resolve, surface it and wait. (Yes: ["No commits found between v1.3.0 and the target — nothing to release", "Tag v1.4.0 already exists — aborting rather than overwriting"] / No: [guessing a version when types are ambiguous, silently bumping past a conflict])

**Avoid overwriting or deleting an existing tag or release on your own initiative** — Once a tag or release is pushed, undoing it is difficult and can break anyone who already pulled it. Never do this unprompted. If the user explicitly asks for it, state the risk plainly and proceed only after they confirm. (Yes: ["Overwriting v1.4.0 will orphan anyone who already pulled that tag — confirm you want to proceed anyway?"] / No: [running `git tag -f` or `gh release delete` because it seemed convenient])

---

## Task

Parse the argument: `[version] [--commit <hash>] [--changelog manual|auto]` — all optional, any order. `--changelog` defaults to `auto`.

1. **Resolve the target commit** — if `--commit <hash>` was given, verify it exists: `git rev-parse --verify <hash>`; report and stop if it doesn't. Otherwise resolve to the latest commit on `origin/main`: `git fetch origin main && git rev-parse origin/main`. Report the resolved SHA and its subject line.

2. **Find the previous release tag** — run `gh release list --limit 1`. If no GitHub release exists yet, fall back to `git describe --tags --abbrev=0 <target-commit>` for a local tag. If neither yields a tag, this is the first release: skip step 3's inference entirely, and if `version` wasn't given, ask the user directly for the starting version (recomend `v0.1.0`).

3. **Determine the version** (skip if `version` was given explicitly):
   - Run `git log <previous-tag>..<target-commit> --pretty=%s` on `main`. Squash-merge commits in this repo carry the merged PR's title, where the `Enhancement`, `Bugfix`, `Cleanup`, `Maintenance`, `Refactor` types can be found.
   - Extract the `[Type]` tag from each line. Bump rule: 
      - `Bugfix`/`Cleanup`/`Maintenance`/`Refactor` present → **patch**
      - `Enhancement` present → **minor**; minors have higher priority than patches
   - **Never infer a major bump** — nothing in this commit convention marks a breaking change. A major bump only happens if the user passes `version` explicitly.
   - If the log is empty, or the types found don't resolve cleanly under the rule above, stop and ask the user directly for the version instead of guessing.
   - Compute the new version by applying the bump to the previous tag's semver.

4. **Check for a tag conflict, then confirm**:
   - Run `git rev-parse -q --verify refs/tags/<version>`. If it resolves, stop: report that `<version>` already exists and ask whether to pick a different version or explicitly overwrite — if overwrite, state plainly that this discards the existing tag/release and can break anyone who already pulled it, and proceed only on explicit confirmation.
   - Otherwise, present the resolved target commit and the version (given or computed) together, then ask via `AskUserQuestion`:
     > Tag `<version>` at `<commit-message> (<sha>)` and proceed?
     > Options: Yes · No

     If "No" or the user gives corrections, revise (different commit or version) and re-present before asking again. Do not proceed to step 5 until "Yes".

5. **Tag**:
   ```
   git tag <version> <target-commit>
   git push origin <version>
   ```

6. **Build the release notes**:
   - `auto` (default): no drafting needed here — `gh release create --generate-notes` handles it in step 7.
   - `manual`: reuse the commit log from step 3 (type, description, and `#N` parsed from each `[Type] Description (#N)` line). Ask via `AskUserQuestion` which format to use:
     > Which release notes format?
     > Options:
     > - **Grouped by type, titles only** — `## Enhancements` / `## Bugfixes` headers, one bullet per PR title + `(#N)`, nothing further.
     > - **Grouped by type, with summary** — same grouping, plus each PR's `## Change` section body pulled in via `gh pr view <N> --json body` as a sub-bullet.
     > - **Flat list with type tag** — single chronological list: `- [Type] Description (#N)`, no section headers.

     Draft the notes in the chosen format. Present the full draft, then ask via `AskUserQuestion`:
     > Approve these release notes?
     > Options: Yes · No

     If "No" or the user gives corrections, revise and re-present the full draft before asking again. Do not proceed to step 7 until "Yes".

7. Run:
   ```
   gh release create <version> --target <target-commit> --title <version> --generate-notes
   ```
   or, if `manual`:
   ```
   gh release create <version> --target <target-commit> --title <version> --notes "<drafted-notes>"
   ```

8. Report the release URL returned by `gh release create`.

---

## Resources

**Version tag and release title format** — Tags and release titles always use `vX.Y.Z` (leading `v`, three dot-separated integers: major.minor.patch, e.g. `v0.1.0`). Applies whether `version` is given explicitly or computed in step 3.
