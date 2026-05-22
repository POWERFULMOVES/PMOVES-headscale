@AGENTS.md

# PMOVES Integration Overlay

This fork is **PMOVES.AI's self-hosted Tailscale control server**. Upstream `AGENTS.md` covers all the Go / SQLite / Postgres / NodeStore / Policy v2 dev workflow. This overlay holds only PMOVES-specific operational facts that don't belong upstream.

## Role in the PMOVES mesh

| Field | Value |
|---|---|
| Public hostname | `headscale.pmoves.ai` |
| Purpose | Tailnet control plane for the PMOVES fleet (Z890, 5090, 4090 laptop, Jetsons, KVM VPS nodes, edge devices) |
| Auth scope | **Third-party enrollment** (UNFCU contractors, guests, future creators) — the owner's nodes already join through standard Tailscale account auth, not this Headscale |
| Companion runbook | `pmoves/docs/operations/FLEET_REMOTE_ACCESS_RUNBOOK.md` (parent repo) |
| Companion docs | `pmoves/docs/operations/FLEET_INVENTORY_LIVE.md`, `pmoves/docs/operations/FLEET_CAPACITY_ANALYSIS.md` |

## Operational entry points (in the parent repo)

| Need | Skill / target |
|---|---|
| Enroll a new third-party node | `/fleet:enroll` (generates CHIT-signed enrollment token) |
| List fleet status | `/fleet:status` |
| Stale-node sweep (offline >60d) | `/fleet:stale-nodes` |
| ACL audit (live vs repo) | `/fleet:acl-audit` |
| RustDesk relay companion | `/fleet:rustdesk-check`, `/fleet:fix-relay` |
| Tailscale ops via MCP | `tailscale` MCP server in `.claude/mcp.json` |
| Cached fleet topology | Cipher Memory (`pmoves-cipher` SSE at `:8105`) |

**Never use raw Tailscale IPs in committed docs or scripts.** Per project convention, use hostnames, `/fleet:*` skills, MCP, or Cipher — the topology-leakage hook will block literal `192.168.*` / `100.64.*` strings.

## PMOVES-specific gotchas

- **Enrollment is for others, not the owner.** Per `feedback_enrollment_is_for_others.md`: Tailscale mesh is the owner's access layer (works through standard Tailscale account); Headscale exists to enroll third parties (UNFCU, guests). Don't burn cycles wiring owner nodes through this server.
- **Hostinger SSH for VPS nodes** is the bootstrap-only path; long-term access goes through Tailscale. See `pmoves/docs/operations/FLEET_REMOTE_ACCESS_RUNBOOK.md`.
- **ACL changes go through the repo, not the running server.** Headscale ACLs are versioned in this repo; `/fleet:acl-audit` compares live policy against the repo definition. Direct `headscale policy set` calls drift the live state.
- **Stale-node policy.** Nodes offline >60 days are flagged for removal via `/fleet:stale-nodes`. PMOVES has a planned cloud routine for weekly auto-sweep (Tailscale MCP-backed).

## When to load this file

- Adding a new node class to the enrollment flow (`/fleet:enroll` extension)
- Auditing or modifying ACL policy for the PMOVES tailnet
- Wiring Headscale ↔ RustDesk relay coordination (see `/fleet:fix-relay`)
- Touching DERP routing in the PMOVES deployment
- Otherwise, prefer upstream `AGENTS.md` directly — the Go dev workflow there is canonical.

## Cross-repo references

- `pmoves/docs/operations/FLEET_REMOTE_ACCESS_RUNBOOK.md` — operator-side fleet runbook
- `.claude/context/git-worktrees.md`, `runner-topology.md` (parent) — node topology context
- Memory: `feedback_tailscale_ops_pattern.md`, `feedback_enrollment_is_for_others.md`

<!-- PMOVES.AI-CONTEXT-TAGS -->
## PMOVES.AI Skill Hints

**Primary Skills:** `/fleet:enroll`, `/fleet:status`, `/fleet:stale-nodes`, `/fleet:acl-audit`, `/deploy:up`, `/botz:mcp`
**Context Files:** `runner-topology.md`, `FLEET_REMOTE_ACCESS_RUNBOOK.md`
**Domain Tags:** `networking`, `infra`, `mesh`
**Context Tier:** 3 (Conditional (Integration))
<!-- /PMOVES.AI-CONTEXT-TAGS -->
