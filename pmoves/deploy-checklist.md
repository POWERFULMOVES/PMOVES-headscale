# PMOVES Headscale — Post-Deploy Checklist

## Phase 4: Health Verify

- [ ] `docker ps --filter name=pmoves-headscale --format '{{.Status}}'` shows "healthy"
- [ ] `curl -s http://localhost:8096/health` returns 200
- [ ] `curl -s http://localhost:9090/metrics` returns Prometheus text (optional — metrics port may be internal)
- [ ] Container filesystem is read-only: `docker inspect pmoves-headscale | jq '.[0].HostConfig.ReadonlyRootFs'` returns `true`

## Phase 5: Post-Deploy

- [ ] Create reusable API key: `docker exec pmoves-headscale headscale apikeys create`
- [ ] Apply ACL policy: `docker exec -i pmoves-headscale headscale acl apply < /docker/{project}/config/acl.yaml`
- [ ] Create admin user: `docker exec pmoves-headscale headscale users create admin`
- [ ] Store API key via ce_memory_store — NEVER in output
- [ ] Verify noise key generated: `docker exec pmoves-headscale ls -la /var/lib/headscale/noise_private.key`
- [ ] Verify SQLite WAL: `docker exec pmoves-headscale ls -la /var/lib/headscale/db.sqlite-wal`

## Smoke Test (Optional)

- [ ] Generate pre-auth key: `docker exec pmoves-headscale headscale preauthkeys create --user admin --reusable --expiry 24h`
- [ ] Register a test node using the pre-auth key
- [ ] Verify node appears in `docker exec pmoves-headscale headscale nodes list`

## Rollback Triggers

- Container fails health check 3x → rollback
- Config validation error in logs → fix config, restart
- SQLite corruption → restore from backup volume snapshot
