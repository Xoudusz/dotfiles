---
name: portainer-logs
description: >
  This skill should be used when the user wants to inspect Docker container logs on portainer-lxc
  (192.168.68.103) without SSH. Triggers on "check logs", "what's in the logs", "cortex logs",
  "container logs", "why is it crashing", or any request to inspect running service output.
---

# /portainer-logs

To inspect container logs: fetch them directly via Portainer REST API — no user action or SSH required.

## Prereqs

- `PORTAINER_URL` and `PORTAINER_TOKEN` must be set in `~/.claude/settings.json` env section.
- Portainer endpoint ID: **2** (name: "local")

## Get Container ID

```bash
curl -sk -H "X-API-Key: $PORTAINER_TOKEN" \
  "$PORTAINER_URL/api/endpoints/2/docker/containers/json?all=1" | \
  jq -r '.[] | select(.Names[] | contains("cortex-mcp")) | .Id'
```

Replace `cortex-mcp` with any container name substring to target a different container.

## Pull Logs

```bash
CID=$(curl -sk -H "X-API-Key: $PORTAINER_TOKEN" \
  "$PORTAINER_URL/api/endpoints/2/docker/containers/json?all=1" | \
  jq -r '.[] | select(.Names[] | contains("cortex-mcp")) | .Id')

curl -sk -H "X-API-Key: $PORTAINER_TOKEN" \
  "$PORTAINER_URL/api/endpoints/2/docker/containers/$CID/logs?stdout=1&stderr=1&tail=100&timestamps=1"
```

Adjust `tail=100` for more/fewer lines. Add `&since=<unix_timestamp>` to filter by time.

## Check Container State

```bash
curl -sk -H "X-API-Key: $PORTAINER_TOKEN" \
  "$PORTAINER_URL/api/endpoints/2/docker/containers/json?all=1" | \
  jq '.[] | select(.Names[] | contains("cortex-mcp")) | {name: .Names[0], state: .State, status: .Status, image: .Image}'
```

## List All Containers (with state)

```bash
curl -sk -H "X-API-Key: $PORTAINER_TOKEN" \
  "$PORTAINER_URL/api/endpoints/2/docker/containers/json?all=1" | \
  jq '.[] | {name: .Names[0], state: .State, status: .Status}'
```

## Notes

- `-sk` flags: `-s` silences curl progress, `-k` skips TLS cert verification (self-signed cert on .103)
- Log output contains Docker multiplexing headers (8 bytes per line) — pipe through `strings` or `cat` if they appear garbled
- `timestamps=1` adds ISO timestamps per log line
- `all=1` includes stopped containers; omit to list only running ones
