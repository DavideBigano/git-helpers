# Todo — git-feature skill

## Bugs

- [ ] **`new` mode: no dirty working tree check** — `git checkout main` will fail or silently carry uncommitted changes to main if the user has unstaged work. Add a `git status` check before step 2; stop and surface the issue if the tree is dirty, asking the user to commit or stash first.
  `.claude/skills/git-feature/SKILL.md` — Mode: new, step 2

- [ ] **`close` mode: chained cleanup swallows partial failure** — `git checkout main && git branch -d <branch-name> && git pull --prune` uses `&&`; if `branch -d` fails the `pull --prune` is skipped silently. Split into three separate commands and surface the result of each, or add explicit failure handling for the `-d` step.
  `.claude/skills/git-feature/SKILL.md` — Mode: close, step 4

- [ ] **`argument-hint` formatting inconsistency** — Current value mixes raw text and backtick-delimited tokens inside a double-quoted YAML string. Simplify to `"<mode> [type/branch-name | pr-number]"`.
  `.claude/skills/git-feature/SKILL.md` — frontmatter

---

## Improvements

- [ ] **Commit message guidance: clarify what vs. why** — "One line and declarative" permits motivation-style messages alongside change-style ones. Torvalds' lens is unambiguous: the message describes what changed; why belongs in the PR body. Add a one-line clarification in the commit mode step.
  `.claude/skills/git-feature/SKILL.md` — Mode: commit, step 3

- [ ] **No mode covers pulling main into a feature branch** — The workflow doc has an explicit section for `git pull origin main`. Consider adding a `pull` mode.
  `.claude/skills/git-feature/SKILL.md` — Task

- [ ] **`pr` mode step 4 under-specifies template field population** — "Following the template structure" is vague in cold context. The template has four named sections (Type, Reasons for change, Change, Checklist); enumerate them explicitly in step 4.
  `.claude/skills/git-feature/SKILL.md` — Mode: pr, step 4

---

## Pending scope decision

- [ ] **Add dangerous git command catalog** — Define an explicit catalog of dangerous git commands (e.g. `git push --force`, `git reset --hard`, `git clean -fd`, `git rebase`, `git filter-branch`, `git reflog delete`) with per-command rationale and refusal behavior. Decide whether this belongs as an extension of `git-feature`, a standalone `git-helper` skill, or a dedicated `safety-helper` skill.
