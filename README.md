# DNS Butler traceroute agent

Run traceroutes for [DNS Butler](https://dnsbutler.com) from your own network.

The agent is a small worker that connects **out** to DNS Butler over a secure WebSocket and runs the
traceroutes you request from the DNS Butler traceroute page. It opens no ports, so it works behind
NAT and firewalls without port forwarding.

- Images for `linux/amd64` and `linux/arm64` (including Raspberry Pi)
- Outbound-only: no listening ports, no inbound firewall rules
- Runs read-only with a single capability: `NET_RAW` for ICMP

## Get a token

1. Sign in to [dnsbutler.com](https://dnsbutler.com) and open **Traceroute Agents** from the account menu.
2. Register a new agent and choose **Service / unattended**.
3. Copy the token it shows (`dbta_...`). It is only displayed once.

Keep the token secret. Use one token per running container; register another agent for each extra
location.

## Docker Compose

Download [`compose.yaml`](https://github.com/mjblacker/dnsbutler-traceroute-agent/blob/main/compose.yaml)
(also on the Traceroute Agents page), then create a `.env` file next to it:

```bash
DNSBUTLER_TRACEROUTE_AGENT_TOKEN=dbta_...
```

```bash
docker compose up -d
docker compose logs -f
```

Update with `docker compose pull && docker compose up -d`.

## Docker run

```bash
docker run -d --name dnsbutler-traceroute-agent --restart unless-stopped \
  --read-only --cap-drop ALL --cap-add NET_RAW \
  -e DNSBUTLER_TRACEROUTE_AGENT_TOKEN="dbta_..." \
  ghcr.io/mjblacker/dnsbutler-traceroute-agent:latest
```

Platforms that deploy an image with environment variables (Kubernetes, Coolify, Railway, Fly.io,
Render) work the same way: set `DNSBUTLER_TRACEROUTE_AGENT_TOKEN` as a secret. No service or health
check port is needed. On Kubernetes, add `NET_RAW` under `securityContext.capabilities.add`.

## Tags

| Tag | Meaning |
| --- | --- |
| `latest` | Most recent release |
| `1.2`, `1.2.0` | Minor and exact agent version |
| `sha-<commit>` | A specific build |

## Environment variables

| Variable | Default | Description |
| --- | --- | --- |
| `DNSBUTLER_TRACEROUTE_AGENT_TOKEN` | (required) | Service agent token from DNS Butler |
| `DNSBUTLER_TRACEROUTE_AGENT_PARALLELISM` | `10` | Maximum ICMP probes across all active traces |
| `DNSBUTLER_TRACEROUTE_AGENT_MAX_CONCURRENT_TRACES` | `10` | Traceroutes run at once (`1`-`100`) |
| `DNSBUTLER_TRACEROUTE_AGENT_DEBUG` | `false` | Include exception details in logs |

## Tokens and updates

Use **Rotate credential** on the agent in DNS Butler to issue a new token; the old one stops working
immediately. **Revoke** disables the agent entirely.

If DNS Butler requires a newer agent, the container logs an "agent deprecated" error and exits.
Pull the latest image and restart it; the token stays valid.
