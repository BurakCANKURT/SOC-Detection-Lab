# Nmap + Hydra Brute Force - Description

**Date:** 2026-11-09
**Analyst:** Burak CANKURT
**Lab:** Home-Lab (Wazuh)
**Severity:** High

## Summary
On 2026-11-09 at 20:58:17 Wazuh detected multiple login failures. The source (192.168.1.106) tried to gain "attacker" user's credential in "home-lab" endpoint. Attempt occurred from 20:58:15 to 20:58:17. 42 failed  password  attempts  were observed; no successful authentication followed. The brute-force attempt failed and the "attacker" account was not compromised. The evidence is at alert logs "sshd: authentication failed."  showed up 42 failed authentication attempts.  

## Alert Details
- Source 192.168.1.106
- Agent-name/target: home-lab
- MITRE ID: T1110	
- MITRE Tactic: Credential Access
- MITRE Technique: Brute Force
- Failed attempts: 42
- Rule ID: 5763 (sshd: brute force trying to get access to the system. Authentication failed.)
- Rule Level: 10
- Full Log: Nov 09 20:58:17.120 home-lab sshd-session[3334]: Failed password for attacker from 192.168.1.106 port 52356 ssh2

## Findings
- From 192.168.1.106 login attempts occurred over a 2 seconds. The 42 attempts in ~2 s indicate an automated, tool-driven brute-force.
- Hydra, Medusa or Ncrack tools could be used in this scenario. 
- 42 failed-password event occurred.
- After 18 alert with rule 5760(Level 5) occurred, Wazuh correlated these attempts with brute-force with rule 5763 (Level 10).

## Assessment
- **Detection: True Positive** The alert correctly identified as  brute-force attack. 
- The source was trying to gain access by using hydra tool with default "rockyou.txt" file.
- If this problem gets disregarded, problems such as these arise:
    - data theft 
    - malicious program injection
    - lateral movement 
    - persistence 
- In this scenario I used nmap to search which ports are open and hydra to find the password of "attacker" user. 
- The nmap scan produced no alert. Only Hydra phase was detected, via host  auth logs. This is a detection gap.
- To close this gap, add network-layer visibility: an IDS sensor (Suricata/Zeek) or firewall/ufw connection logging ingested into Wazuh, so recon (port scan) becomes visible.
- In production, the same alert from an unknown source would be escalated to Tier 2 as a possible attack. 

## Recommendation
- Verify the source IP address. If it is unknown escalate.
- Block the IP, lock the target account and isolate the host.  
- Disable SSH to password auth, enforce MFA + account lockout, add rate-limiting/fail2ban, restrict SSH to known IPs.
- Tune brute-force thresholds; alert on "success after multiple failures". 