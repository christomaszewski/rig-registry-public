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

## Schema 2 migration (2026-08-17)

Profile identity is now the `(service, short-name)` tuple, spelled `service:short` in every ref
(`public/camera-service:siyi-zr30@1.0.0`). On disk a profile lives at
`profiles/<service>/<short>/` — the directory is a projection of the manifest: `name:` holds only
the short half, and `requires.service` must name the parent directory (unqualified; foreign-target
profiles keep the fully-qualified pin in `requires.service`). Flat `profiles/<name>/` layouts and
`schema: 1` are refused by rig ≥ 0.2.0 with a migration pointer. This was a clean break: history
before the move is not served for the migrated profiles.
