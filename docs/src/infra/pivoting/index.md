---
authors: ShutdownRepo, felixbillieres, H4tsuM1ku
category: infra
---

# Pivoting

## TL;DR

* Un pivot = un transport réseau à travers un hôte compromis
* On choisit la technique **AVANT** les outils
* Ligolo-ng > SOCKS > Port forwarding (dans cet ordre de préférence)

## Theory

Pivoting extends access from a compromised system to other systems or networks that are not directly accessible from the initial attack point. It involves using a compromised host as a relay or proxy to access internal network resources.

> [!WARNING]
> Pivoting is **not** inherently stealthy. Pivot connections create new network flows, open new ports, and generate traffic that may be detected by monitoring systems (EDR, IDS, network monitoring). Consider OPSEC implications when establishing pivots.

## Workflow standard

1. **[Enumeration](enumeration.md)**: Identify pivot opportunities (dual-homed systems, accessible networks)
2. **[Choosing a technique](choosing-a-technique.md)**: Decide which pivoting method to use
3. **Deploy the pivot**: Implement using one of the [techniques](techniques/port-forwarding.md)
4. **[Operations](operations.md)**: Enumerate and exploit targets through the pivot
5. **[Persistence](persistence.md)**: Maintain pivot access for long-term operations

## Decision tree

* **SSH available?** → [SSH tunneling](techniques/ssh-tunneling.md)
* **Need full subnet access?** → [Ligolo-ng](techniques/ligolo-ng.md)
* **Single specific port?** → [Port forwarding](techniques/port-forwarding.md)
* **Multiple tools, TCP-based?** → [SOCKS proxy](techniques/socks-proxy.md)
* **Multi-hop required?** → [Multi-hop pivoting](multi-hop.md) (prefer Ligolo-ng or chained SOCKS)

## Resources

Follow the methodology in order:

1. [Enumeration](enumeration.md) - Identify pivot opportunities
2. [Choosing a technique](choosing-a-technique.md) - Decision guide
3. [Techniques](techniques/port-forwarding.md) - Implementation details
4. [Multi-hop pivoting](multi-hop.md) - Extend through multiple segments
5. [Operations](operations.md) - Use established pivots
6. [Persistence](persistence.md) - Maintain pivot access
