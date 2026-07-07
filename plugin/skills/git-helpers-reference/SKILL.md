---
name: git-helpers-reference
description: "Canonical schema reference for gitFeature.config.json, loaded by other skills rather than invoked directly"
user-invocable: false
skillmancy-version: "0.2.0"
---

# Git feature reference

## Resources

### Config file

**Location and format** — `gitFeature.config.json` lives at the repo root (not inside `plugin/`), one JSON object. Its absence is expected and valid — referencing skills must not treat it as an error.

**Properties**

| Property | Type | Required | Default | Description |
|---|---|---|---|---|
| `prTemplate` | string (system path — absolute or relative) | No | None — consumers define their own fallback when unset | Path to a user-provided PR template, overriding the one a consuming skill would otherwise use. |

**Schema evolution** — New properties may be required or optional, and may or may not carry a schema-level default — decide per property when it's added. Existing property names, meanings, required/default status are never changed or removed without updating every skill that reads them — additive only.

**Malformed or missing file handling** — If the file is absent, fails to parse as JSON, or a property's value doesn't match its declared type, treat that property as not defined. This schema only defines what's valid — fallback behavior when a property is undefined is up to whichever skill consumes it.
