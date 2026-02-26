# PMOVES.AI Integration Guide for Headscale

## Integration Overview

Headscale is a self-hosted Tailscale control server that manages VPN mesh networks (tailnets) for PMOVES.AI infrastructure. It handles node registration, IP allocation, policy enforcement, DERP relay, and ACL rules across distributed deployment targets (ai-lab, kvm4, cloudstartup).

## Service Details

- **Name:** Headscale VPN Control Plane
- **Slug:** headscale
- **Tier:** agent
- **Port:** 8096 (API), 9091 (Prometheus metrics)
- **Health Check:** http://localhost:8096/healthz
- **NATS Enabled:** False
- **GPU Enabled:** False

## Integration Points

### VPN Mesh for Distributed Agents
- Provides secure overlay network for multi-host agent orchestration
- Mesh Agent (NATS announcer) uses Tailscale network for node discovery
- CI runner lanes (ai-lab, kvm4, cloudstartup) connected via tailnet

### Infrastructure Connectivity
```
ai-lab host ──┐
kvm4 host ────┤── Headscale Control ── Tailnet Mesh
cloudstartup ─┘                        ├── Agent Zero
                                       ├── Docker Services
                                       └── CI Runners
```

### Key APIs
- `POST /api/v1/apikey` - Create API keys
- `GET /api/v1/machines` - List registered nodes
- ACL policy endpoints for access control

### Database
- SQLite (development) or PostgreSQL (production)

## Next Steps

### 1. Customize Environment Variables

Edit the following files with your service-specific values:

- `env.shared` - Base environment configuration
- Headscale config: `config.yaml` with server URL, DERP settings, ACL policies

### 2. Test Integration

```bash
# Test health check
curl http://localhost:8096/healthz

# Verify metrics endpoint
curl http://localhost:9091/metrics

# List registered nodes
curl -H "Authorization: Bearer <API_KEY>" http://localhost:8096/api/v1/machines
```

## Files Created

- `PMOVES.AI_INTEGRATION.md` - This integration guide

## Support

For questions or issues, see the PMOVES.AI documentation.
