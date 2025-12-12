---
authors: felixbillieres, H4tsuM1ku
category: infra
---

# Enumeration

## Theory

Before establishing a pivot, it's essential to identify systems that can serve as pivot points. These systems typically have access to multiple networks or can reach internal resources that are not directly accessible from the attacker's position.

> [!NOTE]
> This enumeration phase should be performed after initial access is gained on a compromised system. If initial access hasn't been established yet, refer to [Initial access (protocols)](/infra/protocols/index.md) for techniques to gain foothold.

Key indicators of pivot opportunities:
* Systems with multiple network interfaces (dual-homed systems)
* Systems with routes to internal networks
* Systems that can reach firewalled services
* Systems with network access to target segments

Once pivot opportunities are identified, proceed to [choose a pivoting technique](choosing-a-technique.md) and establish the pivot.

## Practice

### Network topology discovery

The network topology and accessible networks can be identified from the compromised system.

::: tabs

=== UNIX-like

Network information can be gathered using various command-line tools.

```bash
# Display all network connections
netstat -an

# Display routing table
netstat -rn

# Display network interface configuration (preferred)
ip addr show
# Or with ifconfig (legacy)
ifconfig -a

# Display routing tables (preferred)
ip route show
# Or with route (legacy)
route -n

# Display network interfaces (preferred)
ip link show
# Or with ip brief format
ip -br a
```

=== Windows

Network information can be gathered using Windows-specific commands.

```bash
# Display all network connections with process information
netstat -ano

# Display network interface configuration
ipconfig /all

# Display routing table
route print
```

:::

### Dual-homed systems

Systems with multiple network interfaces can serve as pivot points between different network segments.

::: tabs

=== UNIX-like

Multiple network interfaces can be identified using network configuration commands.

```bash
# Check for multiple network interfaces
ifconfig | grep "inet "
# Or
ip addr show | grep "inet "
```

=== Windows

Multiple network interfaces can be identified using Windows commands.

```bash
# Check for multiple network interfaces
ipconfig /all | findstr "IP Address"
```

:::

### Accessible networks

Networks that can be reached from the compromised system can be identified through routing tables and connectivity tests.

::: tabs

=== UNIX-like

Routing tables and connectivity tests can reveal accessible networks.

```bash
# Check routing table for network routes
ip route show
route -n

# Test connectivity to internal networks (safer method)
# Using fping (if available)
fping -a -g 192.168.1.0/24 2>/dev/null

# Using nmap (more reliable than bash loop)
nmap -sn --min-rate 100 192.168.1.0/24

# Alternative: sequential ping (slower but safer than parallel)
for i in {1..254}; do timeout 1 ping -c 1 192.168.1.$i 2>/dev/null && echo "192.168.1.$i is up"; done
```

=== Windows

Windows routing tables and ping sweeps can identify accessible networks.

```bash
# Check routing table for network routes
route print

# Test connectivity to internal networks
for /L %i in (1,1,254) do @ping -n 1 -w 100 192.168.1.%i | find "TTL"
```

:::

### DNS and network resolution

DNS configuration and network neighbor information can reveal network topology and accessible systems.

::: tabs

=== UNIX-like

DNS and ARP information can be gathered using various commands.

```bash
# Display ARP table (neighbors)
ip neigh show
# Or legacy
arp -a

# Display DNS configuration
cat /etc/resolv.conf
resolvectl status

# Display network statistics
ss -tuln
```

=== Windows

Windows network resolution information.

```powershell
# Display ARP table
arp -a

# Display DNS configuration
Get-DnsClientServerAddress

# Display network interfaces with PowerShell
Get-NetIPAddress
Get-NetRoute
```

:::

### Egress connectivity testing

Before establishing a pivot, verify that the compromised host can reach the attacker's machine.

```bash
# Test connectivity to attacker machine
# Test common ports used for pivoting
nc -zv $ATTACKER_IP 11601  # Ligolo-ng
nc -zv $ATTACKER_IP 1080   # SOCKS proxy
nc -zv $ATTACKER_IP 443    # HTTPS (for Chisel)
nc -zv $ATTACKER_IP 80     # HTTP (for Chisel)

# Or with PowerShell on Windows
Test-NetConnection -ComputerName $ATTACKER_IP -Port 11601
```

### Network scanning from compromised host

Once a potential pivot point is identified, accessible networks can be scanned to discover targets.

> [!TIP]
> If SSH access is available on the compromised host, consider using [SSH tunneling](techniques/ssh-tunneling.md) for scanning. For comprehensive network access, [Ligolo-ng](techniques/ligolo-ng.md) or [SOCKS proxy](techniques/socks-proxy.md) provides better tool compatibility.

> [!NOTE]
> When scanning through SOCKS proxies, note that ICMP-based host discovery (`nmap -sn`) and OS detection (`nmap -O`) are not reliable. Use TCP connect scans (`-sT`) instead. See [operations](operations.md) for details.

::: tabs

=== UNIX-like

Network scanning can be performed directly from the compromised host.

```bash
# Using nmap from compromised host
nmap -sn 192.168.1.0/24

# Check for specific services
nmap -p 80,443,3389,5985 192.168.1.0/24

# Service version detection
nmap -sV -p- $TARGET
```

=== Windows

Windows systems may require different tools or PowerShell-based scanning.

```bash
# Using PowerShell for host discovery
1..254 | ForEach-Object { Test-Connection -ComputerName "192.168.1.$_" -Count 1 -ErrorAction SilentlyContinue }

# Port scanning with PowerShell (must loop for multiple ports)
80,443,3389,5985 | ForEach-Object { Test-NetConnection -ComputerName $TARGET -Port $_ -InformationLevel Quiet }
```

:::

### Identifying pivot opportunities

Pivot opportunities can be identified by looking for:
* Systems with routes to internal subnets
* Systems that can reach domain controllers or critical infrastructure
* Systems with access to multiple VLANs
* Systems that can reach DMZ or internal networks

> [!NOTE]
> After identifying a viable pivot point, see [Choosing a technique](choosing-a-technique.md) to select the appropriate pivoting method.

## Resources

* [Choosing a technique](choosing-a-technique.md) - Select pivoting method
* [Techniques](techniques/port-forwarding.md) - Implementation details
* [Operations](operations.md) - Perform operations once pivot is established

