# Contributing packages

Ground rules `tools/validate` (and CI) enforce:

- **Names** are `[a-z][a-z0-9-]*` and unique across ALL kinds — the index keys by bare name.
  (Package names are registry identifiers; deployment *instance* names remain underscore-only.)
- **Versions** are exact `X.Y.Z`. The registry's HEAD carries the current version of each
  package; history lives in git, and consumers pin the registry commit in their lockfile.
- **Services**: `source.rev` is a FULL commit SHA (no branches, no short SHAs); `image.ref`
  is digest-pinned (`…@sha256:<64 hex>`) — tag refs are rejected. `source.path` points at the
  service's dir inside a collection repo (e.g. one repo carrying several services).
- **Profiles**: `requires.service` must resolve (in-registry, or fully qualified for another
  registry) and be satisfied by that service's version. The config payload is a **nameless**
  config — the deployment's instance row supplies `name`.
- **Overlays**: the payload is a **delta only** (never a full config copy; `null` deletes a
  key). It must not set `name`/`service` — identity belongs to the instance row.
- **Suites**: references only — fully-qualified exact pins (`ns/name@X.Y.Z`), no `config/`
  dir, no nested suites.
- **index.json is GENERATED** (`tools/gen-index`). A stale or hand-edited index fails CI.

Flow: branch → add/edit the package → `./tools/validate` → `./tools/gen-index` → commit
(package + index) → PR. The easiest correct path from a working deployment is
`rig pkg promote … --to <this registry>`, which scaffolds all of the above.
