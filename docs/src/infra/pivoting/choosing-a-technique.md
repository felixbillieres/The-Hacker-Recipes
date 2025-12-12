---
authors: felixbillieres, H4tsuM1ku
category: infra
---

# Choosing a pivoting technique

## Decision matrix

| Need | Technique | When to use |
|------|-----------|-------------|
| Single service access | [Port forwarding](techniques/port-forwarding.md) | One specific port (RDP, SSH, web service) |
| Multiple tools, TCP-based | [SOCKS proxy](techniques/socks-proxy.md) | Need to use various tools through proxy |
| Full network access, all protocols | [Ligolo-ng](techniques/ligolo-ng.md) | Nmap, AD tools, LDAP, best compatibility |
| SSH already available | [SSH tunneling](techniques/ssh-tunneling.md) | SSH access on pivot, encrypted, no extra tools |
| Multi-hop required | [Multi-hop pivoting](multi-hop.md) | Multiple network segments to traverse |
| OPSEC critical | [SSH tunneling](techniques/ssh-tunneling.md) | Encrypted, common protocol, less suspicious |

## Quick decision guide

### Scenario 1: Single service access

**Example:** Need to access RDP on `192.168.1.100` through pivot.

→ Use **[Port forwarding](techniques/port-forwarding.md)**

**Why:** Minimal setup, specific port only, low overhead.

### Scenario 2: Full network enumeration

**Example:** Need to scan entire subnet `192.168.1.0/24`, enumerate AD, use multiple tools.

→ Use **[Ligolo-ng](techniques/ligolo-ng.md)**

**Why:** Best tool compatibility, transparent access, no proxychains needed.

### Scenario 3: Multiple tools, limited protocols

**Example:** Need to use nmap, gobuster, curl through pivot, but no AD tools.

→ Use **[SOCKS proxy](techniques/socks-proxy.md)**

**Why:** Good for TCP-based tools, requires proxychains but works well.

### Scenario 4: SSH access available

**Example:** Have SSH credentials on pivot host.

→ Use **[SSH tunneling](techniques/ssh-tunneling.md)**

**Why:** No additional tools, encrypted, built-in port forwarding and SOCKS support.

### Scenario 5: Multi-hop required

**Example:** Need to traverse multiple network segments (Attacker → Pivot1 → Pivot2 → Target).

→ Use **[Multi-hop pivoting](multi-hop.md)** with:
* **Ligolo-ng** (preferred) - Transparent multi-agent setup
* **Chained SOCKS proxies** - If Ligolo-ng not available

**Why:** Ligolo-ng handles multi-hop automatically, chained SOCKS requires manual configuration.

## Anti-patterns

> ❌ **Anti-pattern:** Scanning an entire subnet via chained port forwarding.
> 
> → Use Ligolo-ng or SOCKS proxy instead.

> ❌ **Anti-pattern:** Using SOCKS proxy when only one port is needed.
> 
> → Use port forwarding for simpler setup.

> ❌ **Anti-pattern:** Using port forwarding for full network enumeration.
> 
> → Use Ligolo-ng or SOCKS proxy for better tool compatibility.

## Resources

* [Port forwarding](techniques/port-forwarding.md) - Single port access
* [SOCKS proxy](techniques/socks-proxy.md) - Multiple tools via proxy
* [Ligolo-ng](techniques/ligolo-ng.md) - Full network access
* [SSH tunneling](techniques/ssh-tunneling.md) - SSH-based pivoting
* [Multi-hop pivoting](multi-hop.md) - Multiple network segments

