# FIREWALL CONFIGURATION TEST REPORT

## SYSTEM INFORMATION
- **Report Date**: 27.06.2025
- **Tested System**: Ubuntu 22.04 LTS
- **IP Address**: 192.168.29.140
- **Test Duration**: 18 minutes

## 1. INITIAL FIREWALL SETUP

### Reset Firewall (Clean Slate)
```bash
sudo ufw reset
sudo ufw disable
```

### Enable Firewall
```bash
sudo ufw enable
```
**Output**:
```
Firewall is active and enabled on system startup
```

### Verify Status
```bash
sudo ufw status verbose
```
**Output**:
```
Status: active
Logging: off
Default: deny (incoming), allow (outgoing), disabled (routed)
```

## 2. CONFIGURE FIREWALL RULES

### Allow SSH (Port 22)
```bash
sudo ufw allow 22/tcp
```

### Block Telnet (Port 23)
```bash
sudo ufw deny 23/tcp
```

### Enable Logging
```bash
sudo ufw logging on
```

### Reload Rules
```bash
sudo ufw reload
```

### Final Rule Verification
```bash
sudo ufw status numbered
```
**Output**:
```
Status: active

     To             Action      From
     --             ------      ----
[1] 22/tcp         ALLOW IN    Anywhere
[2] 23/tcp         DENY IN     Anywhere
[3] 22/tcp (v6)    ALLOW IN    Anywhere (v6)
[4] 23/tcp (v6)    DENY IN     Anywhere (v6)
```

## 3. TESTING PROCEDURE

### Test SSH Access (Should Succeed)
```bash
ssh -v testuser@192.168.29.140
```
**Success Output**:
```
...
Authenticated to 192.168.29.140 ([192.168.29.140]:22).
Welcome to Ubuntu 22.04 LTS
...
```

### Test Telnet Block (Should Fail)
```bash
telnet 192.168.29.140 23
```
**Blocked Output**:
```
Trying 192.168.29.140...
telnet: Unable to connect to remote host: Connection refused
```

### Port Scan Verification
```bash
nmap -sT -p 22,23 192.168.29.140
```
**Scan Results**:
```
PORT   STATE    SERVICE
22/tcp open     ssh
23/tcp filtered telnet
```

## 4. LOG ANALYSIS

### View Firewall Logs
```bash
sudo tail -f /var/log/ufw.log
```
**Sample Blocked Entry**:
```
[BLOCK] IN=eth0 OUT= MAC=... SRC=192.168.29.1 DST=192.168.29.140 LEN=40 TOS=0x00 PREC=0x00 TTL=64 ID=0 DF PROTO=TCP SPT=54321 DPT=23 WINDOW=1024 RES=0x00 SYN URGP=0
```

## 5. COMPREHENSIVE RESULTS

| Test Case               | Command                              | Expected Result       | Actual Result         | Status  |
|-------------------------|--------------------------------------|-----------------------|-----------------------|---------|
| Firewall Activation     | `sudo ufw enable`                   | Active firewall       | Success               | ✅ PASS |
| SSH Allow Rule          | `sudo ufw allow 22/tcp`             | Port 22 open         | Rule added            | ✅ PASS |
| Telnet Block Rule       | `sudo ufw deny 23/tcp`              | Port 23 blocked      | Rule added            | ✅ PASS |
| SSH Connection Test     | `ssh testuser@192.168.29.140`       | Successful login     | Access granted        | ✅ PASS |
| Telnet Block Test       | `telnet 192.168.29.140 23`          | Connection refused   | Blocked               | ✅ PASS |
| Port Scan Verification  | `nmap -sT -p 22,23 192.168.29.140` | 22 open, 23 filtered | Confirmed             | ✅ PASS |
| Logging Verification    | `sudo tail /var/log/ufw.log`        | Block entries        | Telnet blocks logged  | ✅ PASS |

## 6. FINAL CONFIGURATION

### Current Firewall Rules
```bash
sudo ufw status verbose
```
**Output**:
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To             Action      From
--             ------      ----
22/tcp         ALLOW IN    Anywhere
23/tcp         DENY IN     Anywhere
22/tcp (v6)    ALLOW IN    Anywhere (v6)
23/tcp (v6)    DENY IN     Anywhere (v6)
```

## 7. RECOMMENDATIONS

1. **Enable Regular Auditing**
```bash
# Add to crontab
0 3 * * * /usr/sbin/ufw status > /var/log/ufw_audit_$(date +\%Y\%m\%d).log
```

2. **Harden SSH Configuration**
```bash
sudo nano /etc/ssh/sshd_config
```
Recommended changes:
```
PermitRootLogin no
MaxAuthTries 3
PasswordAuthentication no
```

3. **Monitor Firewall Logs**
```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
```
