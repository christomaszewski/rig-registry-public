# rig-registry-public — a rig package registry (namespace `public`)

A registry is a git repo (or a plain shared folder) of **manifests**; rig resolves and installs
from it. Four package kinds:

| kind | dir | carries |
|---|---|---|
| `service` | `services/<name>/` | pointer to a code repo pinned by FULL commit SHA and/or an image pinned by digest |
| `profile` | `profiles/<service>/<short>/` | device/sensor profile: hardware `match` identifiers, a required service, a **nameless** config payload — identity is the `(service, short)` tuple, keyed `service:short` (registry schema 2) |
| `overlay` | `overlays/<name>/` | versioned, project-tagged config **delta** targeting a service or instance |
| `suite`   | `suites/<name>/`   | ordered *references* to the above (fully-qualified exact pins) — never payloads |

Kind decision rule: different hardware or new match identifiers → new **profile**; project
tuning of an existing config → **overlay**.

## Consume

```
rig registry add public <url-of-this-repo>     # git-hosted (managed clone + `rig registry sync`)
rig registry add public --path <this-dir>      # or use this folder in place (local-dir)
```

## Author

From a deployment, rig scaffolds everything (rig ≥ v0.2.9):

```
rig pkg save <instance|service>       # UPDATE the package it came from, in place (next version)
rig pkg promote <instance> --to public   # something NEW: first publish, fork, kind change, suite
rig pkg outdated                      # dependency drift here (FIX column names repin/rebase)
rig pkg repin <pkg> --to public       # advance declared pins; `pkg rebase` reconciles payloads
rig registry pending / push --pr      # publish the promote/* branches (your git + gh; PR-gated)
rig pkg yank <pkg> --from public      # retract a mistake (previous version restored from history)
```

Editing by hand instead? Validate + regenerate the index before committing:

```
./tools/validate      # every rule CI enforces, locally
./tools/gen-index     # regenerate index.json (GENERATED — never hand-edit)
```

Commit both the package and the regenerated `index.json`. CI (GitHub Actions and GitLab
wrappers are both included) re-runs the same `tools/validate`. See CONTRIBUTING.md for the
rules (schema 2: tuple profile identity, placement law, exact pins).

`schemas/` holds informational JSON Schemas mirroring rig's validation rules for external
tooling; the authoritative validator is rig itself (`rig registry validate`).
