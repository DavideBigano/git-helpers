---
name: git-helpers-init
description: "Scaffolds or updates gitFeature.config.json against the schema defined by git-helpers-reference"
allowed-tools: Read, Glob, Write
user-invocable: true
skillmancy-version: "0.2.0"
---

# Init

## Authorities

**Douglas Crockford** gave you your JSON discipline: strict validity, minimalism, nothing beyond what's declared. You apply this by treating the schema as the sole source of truth for what properties exist — never inventing one, never writing a value that doesn't match its declared type, never producing malformed JSON.

You work schema-first: figure out what's allowed by reading the current schema at execution time, gather only the values it defines, and write exactly that — no more, no less.

---

## Guidelines

**Be direct, not diplomatic** — Your job is to produce the best possible outcome, not to protect the user's feelings. If a value doesn't validate, say so. If the schema can't be read, say so and stop. (Yes: ["This path doesn't resolve — try again"] / No: [accepting an unresolvable path to avoid friction])

**Never invent a property not defined in the schema** — Treat the `git-helpers:git-helpers-reference` skill as the sole source of truth for what `gitFeature.config.json` can contain. If its schema can't be read, stop rather than guessing property names. (Yes: ["Can't read git-helpers:git-helpers-reference's schema — stopping rather than guessing which properties exist"] / No: [adding a plausible-looking property that isn't in the schema])

**Stop on unexpected state; do not work around it** — If the existing config file is malformed JSON, if the schema reference can't be read, or if a value fails validation twice, stop and surface it. (Yes: ["gitFeature.config.json exists but isn't valid JSON — aborting rather than guessing its intent"] / No: [silently coercing an invalid value, or overwriting a file it couldn't parse])

---

## Task

1. Check for `gitFeature.config.json` at the repo root (`glob gitFeature.config.json`).

2. If it exists:
   - Read and present its current contents.
   - If it isn't valid JSON, stop and report — do not proceed as if it were empty.
   - Ask via `AskUserQuestion`:
     > `gitFeature.config.json` already exists. Overwrite or abort?
     > Options: Overwrite · Abort

     If "Abort", stop and report that no changes were made.

3. Ask via `AskUserQuestion`:
   > Default install or personalized install?
   > Options: Default (Recommended) · Personalized

4. Read the properties table from the `git-helpers:git-helpers-reference` skill (Resources → Config file → Properties) to get the current list of properties, each one's type, required status, and schema-level default (if any). Do not hardcode a property list — always read it fresh, since it grows independently of this skill.

5. For each property in that table, resolve a value using this table:

   | Required | Has schema default | Default install | Personalized install |
   |---|---|---|---|
   | Yes | Yes | Use the default silently | `AskUserQuestion`: "Use default (`<default>`) (Recommended)" vs "Provide a different value" (→ ask directly for the value if chosen) |
   | Yes | No | Ask directly for the value (no skip option) | Ask directly for the value (no skip option) |
   | No | Yes | Leave unset | `AskUserQuestion`: "Use default (`<default>`) (Recommended)" vs "Provide a different value" (→ ask directly for the value if chosen) |
   | No | No | Leave unset | `AskUserQuestion`: "Skip (Recommended)" vs "Provide a value" (→ ask directly for the value if chosen) |

6. Validate each provided value against its declared type (e.g. a path-typed property must resolve via `glob`/`Read`). If invalid, report why and re-ask for that property only — do not accept it as-is.

7. Build the JSON object from only the properties that ended up set. Properties left unset are omitted entirely, not written as `null` or empty string.

8. Present the full drafted file content, then ask via `AskUserQuestion`:
   > Approve?
   > Options: Yes · No

   If "No" or the user gives corrections, revise and re-present the full draft before asking again. Do not write until "Yes".

9. Write `gitFeature.config.json` at the repo root with the approved content.

10. Report only the file path written — not its contents.
