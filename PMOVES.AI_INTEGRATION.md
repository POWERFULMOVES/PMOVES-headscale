# PMOVES.AI Integration Dossier

_Last updated: 2026-04-21_

## Module
- Name: Headscale
- Path: vendor/PMOVES-headscale (git submodule)

## Purpose in PMOVES.AI
- Runtime: VPN mesh control plane (Tailscale-compatible self-hosted control server)
- Provides: node management, auth key creation, route advertisement, ACL enforcement, DERP relay coordination
- Fleet role: Exit Node tier (network edge)

## PMOVES Overlay Surface
- pmoves-integrations/ overlay path: N/A (config-driven, no code overlay)
- Compose/profile wiring: pmoves/docker-compose.remote.yml (headscale service, profile: remote)
- Standalone compose: pmoves/docker-compose.hostinger.yml (fork: pmoves/docker-compose.hostinger.yml)
- Env/secret inputs: DOCKED_MODE, PARENT_SYSTEM, PARENT_VERSION (informational only)
- Auth/JWT requirements: None (Headscale uses its own Noise protocol + pre-auth keys)

## Contracts and Topics
- NATS subjects: None (Headscale does not use NATS)
- Supabase schema/tables touched: None directly (BoTZ VPN MCP writes session logs to Supabase)
- MCP endpoints/skills: BoTZ VPN MCP (port 8110, SSE transport) — consumes Headscale REST API

## Boot Order and Health
- Bring-up dependency order: No dependencies (Headscale starts first on exit node VPS)
- Health endpoints: GET /health (NOT /healthz — confirmed in hscontrol/app.go:523)
- Smoke targets: Container healthy + API key created + ACL applied + admin user created

## Hardening Notes
- Image pinning / provenance: ghcr.io/juanfont/headscale:v0.28.0 (GHCR, upstream)
- Secrets source: API key generated post-deploy via `headscale apikeys create`, stored in ce_memory_store
- Network/security policy constraints:
  - read_only: true (immutable filesystem)
  - tmpfs: /var/run/headscale (unix socket — required with read_only)
  - Config volume mounted :ro
  - command: serve (REQUIRED)
  - Distroless base image (no shell in non-debug variant)

## Source Documentation
- Upstream docs entrypoint: README.md, docs/ directory
- Upstream config reference: config-example.yaml (v0.28.0, 483 lines)
- PMOVES config: pmoves/config.yaml (this fork)
- PMOVES ACL: pmoves/acl.yaml (this fork)
- PMOVES compose: pmoves/docker-compose.hostinger.yml (this fork)
- PMOVES docs index reference: pmoves/docs/architecture/vpn-mesh-architecture.md

## Owner / Audit
- Owning lane: Networking / Infrastructure
- Last integration audit run: 2026-04-21
