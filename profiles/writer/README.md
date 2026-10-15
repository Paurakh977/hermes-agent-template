# Writer profile

Drafting, voice, narrative flow and docs. Inherits deployment defaults:
docker sandbox terminal, `compression.threshold: 0.85`, the Parallel Search
MCP (merged from the default profile at load), and the better-auth skill pack.
Ships its own DeepWiki MCP (free, keyless) for reading public GitHub repos.

## Install (inside the deployment)

```sh
# from the repo root on the host (monorepo already bind-mounted):
docker compose exec -u 10000:10000 hermes hermes profile install /opt/agent-profiles/writer
cp agent-template/profiles/writer/.env.EXAMPLE data/profiles/writer/.env  # then fill in
docker compose exec -u 10000:10000 hermes hermes -p writer gateway start   # only for gateway use
```

Update later: `git pull` the monorepo on the host, then
`hermes profile update writer` (ships SOUL/skills fresh, preserves
`config.yaml` unless `--force-config`, never touches memories/sessions/`.env`).
