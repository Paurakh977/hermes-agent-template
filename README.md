# Agent template monorepo (default profile + named profiles)

One repo, two consumers:

| Path | Consumed by | How |
|---|---|---|
| `default/` | Docker image build | `Dockerfile` bakes `default/config.overlay.yaml` (threshold, MCP) into the config template and `default/skills/` into `/opt/default-template/skills/`; `config-seed` seeds both on fresh boot only. See `default/README.md`. |
| `profiles/<name>/` | `hermes profile install` | Bind-mounted at `/opt/agent-profiles` (read-only install source, never executed in place). Each profile is a distribution: `distribution.yaml` at its root + `SOUL.md` + its own `config.yaml` + `skills/` (real files, never symlinks). |

Every profile inherits the deployment defaults: docker-sandbox `terminal:` (with its
own DIND-resolvable config-file mount), `compression.threshold: 0.85`, the
Parallel Search MCP (merged from the default profile at load when the profile
omits `mcp_servers`), and the better-auth skill pack (vendored per profile).

## Splitting into its own private repo (one time)

Keep this content here until then — it already works as a local stand-in.
When ready (uses `git subtree`: content always present on every clone, no
`--recurse-submodules` step to forget on a VPS):

```sh
# 1. Create an EMPTY PRIVATE repo on GitHub, e.g. you/agent-template.
#    Private: model choices, MCP endpoints and skills fingerprint you.
#    This dir currently contains zero secrets (scanned) — keep it that way;
#    agent-template/.gitignore blocks .env/auth/state/memories/sessions.
# 2. Split history + push (run from this repo root):
git subtree split --prefix=agent-template -b template-split
git remote add agent-template git@github.com:you/agent-template.git
git push agent-template template-split:main
# 3. From then on, sync explicitly (never automatic):
git subtree pull  --prefix=agent-template agent-template main --squash -m "sync template from agent-template"
git subtree push  --prefix=agent-template agent-template main
```

Why subtree, not submodule: a forgotten `submodule update` leaves
`agent-template/` empty — the Docker build then fails late (or worse, the
profiles bind mounts empty and installs silently find nothing). Subtree keeps
one working tree; sync points are explicit commits in both repos.

VPS side: give the VPS a read-only deploy key (least privilege) or use your
normal git auth there; `pull` before every `profile install`/`update`.
Per-machine `.env`/memories/sessions never enter either repo.
