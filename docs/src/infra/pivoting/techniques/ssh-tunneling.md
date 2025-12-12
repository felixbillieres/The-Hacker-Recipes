---
authors: felixbillieres, H4tsuM1ku
category: infra
---

# SSH tunneling

## Theory

SSH tunneling uses the Secure Shell (SSH) protocol to create encrypted tunnels for network traffic. SSH provides built-in port forwarding and SOCKS proxy capabilities without additional tools.

**Use when:**
* SSH access is available on the pivot host
* Encrypted communication is required
* No additional software installation is possible

## Practice

### Port forwarding

**Local port forwarding:**

```bash
ssh -N -L $LOCAL_PORT:$TARGET_HOST:$TARGET_PORT $USER@$PIVOT_HOST
```

**Remote port forwarding:**

```bash
ssh -N -R $REMOTE_PORT:$TARGET_HOST:$TARGET_PORT $USER@$PIVOT_HOST
```

> [!TIP]
> For remote port forwarding, ensure `/etc/ssh/sshd_config` has `GatewayPorts yes`.

**SSH config:**

```bash
# ~/.ssh/config
Host pivot1
    HostName $PIVOT_HOST
    User $USER
    LocalForward 8080 192.168.1.100:80
    RemoteForward 2222 localhost:22
    ServerAliveInterval 60
```

### Dynamic port forwarding (SOCKS proxy)

Creates a SOCKS proxy through the SSH connection.

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

**Configure proxychains:**

```bash
# /etc/proxychains4.conf
socks5 127.0.0.1 1080

# Use tools through proxy
proxychains nmap -sT -Pn -n $TARGET
```

### SSH key authentication

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096

# Copy public key to pivot host
ssh-copy-id $USER@$PIVOT_HOST
```

### Useful flags

```bash
# -N: Don't execute remote command (tunnel only)
# -f: Background after authentication
# -T: Disable pseudo-terminal allocation
# -o ExitOnForwardFailure=yes: Fail if port forwarding fails
# -o ServerAliveInterval=60: Keep connection alive

ssh -N -f -T -o ExitOnForwardFailure=yes -o ServerAliveInterval=60 \
    -L 8080:$TARGET:80 $USER@$PIVOT_HOST
```

### Multi-hop

For multi-hop SSH tunneling, see [Multi-hop pivoting](../../multi-hop.md#multi-hop-with-port-forwarding).

## Resources

* [Choosing a technique](../choosing-a-technique.md) - When to use SSH tunneling
* [SOCKS proxy](socks-proxy.md) - Alternative SOCKS proxy methods
* [Multi-hop pivoting](../multi-hop.md) - Multi-hop scenarios
* [Operations](../operations.md) - Using SSH tunnels
* [Persistence](../persistence.md) - Maintaining SSH tunnel access

