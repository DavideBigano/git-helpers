# Releases

Releases are cut from `main` after all planned work for a version has merged.

## Process

1. **Merge all in-scope PRs to `main`** — including any maintenance PRs (e.g. version bump).

3. **Open a PR** with head `main` and base `latest`, using `.github/RELEASE_PULL_REQUEST_TEMPLATE.md`:
   ```
   gh pr create --title "[Release] vX.Y.Z" --base latest --head main
   ```

4. **Fill in the release PR:**
   - `## What's new` — 2–4 sentence narrative: what ships, why it matters, what direction it moves the project.
   - `## Changelog` — end-state snapshot grouped by component. No PR references, no commit history.

5. **Complete the checklist** before merging:
   - Version bumped in `plugin/.claude-plugin/plugin.json`
   - Changelog complete
   - Documentation up to date
