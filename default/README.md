# Default-profile template (git-versioned fresh-boot defaults)

This directory is the independently-editable source of truth for what a
**fresh** deployment (`./data` empty, `hermes-config` volume missing) boots
with. Edit here, rebuild the image, and every new machine converges to the
same defaults. Existing machines are never touched (seed is copy-if-absent).

## Layout

| Path | Seeded to | How |
|---|---|---|
| `config.overlay.yaml` | `hermes-config` volume `config.yaml` | Applied at **build time** onto the upstream template by `scripts/patch_template_defaults.py` (terminal/model/version untouched), then byte-copied on fresh seed by `scripts/seed_config.py` |
| `skills/better-auth/` | `./data/skills/better-auth/` | Copied on every `up` **iff absent** (never overwrites your edits) |
| `skills/security/` | `./data/skills/security/` | Same as above |

## Workflow

```sh
# 1. Edit the overlay (threshold, MCP servers) or refresh skills below.
# 2. Rebuild + restart:
docker compose build hermes
docker compose up -d
# 3. Fresh machines pick it up automatically. Existing volumes keep their
#    config; new skill dirs appear under ./data/skills/ on next up.
```

> Post-`setup` note: on a fresh machine you run `hermes setup` once to enter
> API keys — setup regenerates the sections it manages from code defaults and
> resets `compression.threshold` to `0.50` (verified: pre-setup backup keeps
> `0.85`, post-setup file shows `0.50`; `mcp_servers` and `terminal:` survive).
> Re-apply afterwards (control plane, preserves everything else):
>
> ```sh
> docker compose exec -T -u 10000:10000 hermes hermes config set compression.threshold 0.85
> ```

Back up before any `down -v`: `sh scripts/backup-config.sh`.

## Refreshing better-auth skills from upstream

Vendored from https://github.com/better-auth/skills (only dirs containing
`SKILL.md`; `.claude-plugin/` and `commands/` intentionally omitted — no
`SKILL.md`, Claude-only).

```sh
git clone --depth 1 https://github.com/better-auth/skills /tmp/ba-skills
rm -rf agent-template/default/skills/better-auth agent-template/default/skills/security
cp -r /tmp/ba-skills/better-auth agent-template/default/skills/
cp -r /tmp/ba-skills/security agent-template/default/skills/
rm -rf agent-template/default/skills/better-auth/.claude-plugin agent-template/default/skills/better-auth/commands
```

Then rebuild. Local edits under `./data/skills/` are never overwritten, so
re-vendoring only affects fresh machines until you copy dirs over manually.
