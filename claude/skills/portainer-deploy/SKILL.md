---
name: portainer-deploy
description: >
  This skill should be used when the user wants to deploy or redeploy a Docker service on the portainer-lxc
  host (192.168.68.103). Triggers on "deploy cortex", "redeploy", "ship it" (after a push), "update
  container", "pull new image", or similar deploy-related requests.
---

# /portainer-deploy

To deploy a new version: verify CI, instruct the user to do the Portainer UI steps, then verify the container is running via API.

## Prereqs

- `PORTAINER_URL` and `PORTAINER_TOKEN` must be set in `~/.claude/settings.json` env section.
- Portainer endpoint ID: **2** (name: "local")

## Deploy Flow

### 1. Verify CI passed

```bash
gh run list --repo Xoudusz/cortex --limit 3
```

Confirm the latest run on `main` shows `completed` / `success`. If not, wait or investigate CI failure first (`/github-actions`).

### 2. Instruct user to redeploy in Portainer UI

Tell the user to do these steps in Portainer (https://192.168.68.103:9443):

1. **Containers** → find `cortex-mcp` → Stop → Remove
2. **Images** → find `ghcr.io/xoudusz/cortex-mcp` → Remove (forces pull of `:latest`)
3. **Stacks** → `cortex` → **Redeploy** (pulls new image, recreates container)

### 3. Verify container is running

After user confirms Portainer redeploy is done:

```bash
curl -sk -H "X-API-Key: $PORTAINER_TOKEN" \
  "$PORTAINER_URL/api/endpoints/2/docker/containers/json?all=1" | \
  jq '.[] | select(.Names[] | contains("cortex-mcp")) | {name: .Names[0], state: .State, status: .Status}'
```

Expected: `"state": "running"`. If `"state": "exited"`, pull logs via `/portainer-logs`.

### 4. (Optional) Verify image digest

```bash
curl -sk -H "X-API-Key: $PORTAINER_TOKEN" \
  "$PORTAINER_URL/api/endpoints/2/docker/containers/json?all=1" | \
  jq '.[] | select(.Names[] | contains("cortex-mcp")) | .ImageID'
```

## Notes

- Never manually create git tags or GitHub releases — CI does it automatically on push to `main` with new `VERSION`
- If CI already released the current VERSION (tag exists), bump VERSION again and push
- Stack name in Portainer: `cortex` (matches `docker-compose.yml` in repo root)
