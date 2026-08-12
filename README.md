# rig-registry-public — a rig package registry (namespace `public`)

A registry is a git repo (or a plain shared folder) of **manifests**; rig resolves and installs
from it. Four package kinds:

| kind | dir | carries |
|---|---|---|
| `service` | `services/<name>/` | pointer to a code repo pinned by FULL commit SHA and/or an image pinned by digest |
| `profile` | `profiles/<name>/` | device/sensor profile: hardware `match` identifiers, a required service, a **nameless** config payload |
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

Add or edit a package (usually via `rig pkg promote` from a deployment, or by hand), then:

```
./tools/validate      # every rule CI enforces, locally
./tools/gen-index     # regenerate index.json (GENERATED — never hand-edit)
```

Commit both the package and the regenerated `index.json`. CI (GitHub Actions and GitLab
wrappers are both included) re-runs the same `tools/validate`.

`schemas/` holds informational JSON Schemas mirroring rig's validation rules for external
tooling; the authoritative validator is rig itself (`rig registry validate`).
