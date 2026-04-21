@AGENTS.md

<!-- PMOVES.AI-CONTEXT-TAGS -->
## PMOVES.AI Skill Hints

**Primary Skills:** `/deploy:up`
**Context Files:** pmoves/config.yaml, pmoves/acl.yaml, pmoves/docker-compose.hostinger.yml
**Domain Tags:** `networking`, `infra`, `vpn`
**Context Tier:** 3 (Conditional (Integration))
<!-- /PMOVES.AI-CONTEXT-TAGS -->

## PMOVES Headscale Fork

This is the PMOVES-customized fork of headscale v0.28.0.
All PMOVES deployment artifacts live under `pmoves/`.

### File Map

| File | Purpose |
|------|---------|
| `pmoves/config.yaml` | Production config (v0.28.0 schema) |
| `pmoves/acl.yaml` | ACL policy (tag/group model) |
| `pmoves/docker-compose.hostinger.yml` | Hardened Hostinger API compose |
| `pmoves/env.example` | Environment variable template |
| `pmoves/deploy-checklist.md` | Post-deploy verification steps |
| `config-example.yaml` | Upstream reference (DO NOT MODIFY) |
| `PMOVES.AI_INTEGRATION.md` | Integration dossier |

### Critical Rules

1. NEVER modify `config-example.yaml` — it is the upstream v0.28.0 reference
2. Health endpoint is `/health` (NOT `/healthz`) — confirmed in hscontrol/app.go:523
3. Actual port is 8096 (NOT 8181) — 8181 is a documentation bug in main repo
4. `command: serve` is REQUIRED in Docker — without it, container prints help and exits
5. `read_only: true` requires `tmpfs: /var/run/headscale` for the unix socket
6. server_url MUST be HTTPS for DERP relay and Noise protocol to work
7. Config file is authoritative — do NOT use HEADSCALE_SERVER_URL/LISTEN_ADDR env vars
8. Image tag MUST be version-pinned (`:v0.28.0`) — NEVER use `:latest`

### Known Bugs in Main Repo (to be fixed separately)

- services-catalog.md line 466: says `/healthz` — should be `/health`
- vps-deployer.md line 97: says `curl healthz` — should be `curl /health`
- deploy/compose/README.md line 27: says `curl healthz` — should be `curl /health`
- kvm-exit-node-hosting-strategy.md: says port 8181 — should be 8096
- KVM_HOSTINGER_NETWORK_REPORT.md: says port 8181 — should be 8096
- pbnj/pinokio/api/pmoves-remote/SKILL.md: says port 8181 — should be 8096
- examples/distributed/tailscale/README.md lines 71,80: server_url includes `:8096` port — should NOT include port for HTTPS URLs

### v0.28.0 New Fields (vs 0.27.x)

- `node.routes.ha.probe_interval` / `probe_timeout` — HA subnet router health probing
- `database.gorm.*` — GORM tuning (prepare_stmt, parameterized_queries, etc.)
- `database.sqlite.wal_autocheckpoint` — WAL auto-checkpoint threshold
- `taildrop.enabled` — explicit Taildrop section
- `prefixes.allocation` — explicit allocation strategy
- `randomize_client_port` — firewall workaround flag
