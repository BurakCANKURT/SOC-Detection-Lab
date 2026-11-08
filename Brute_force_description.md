# SSH Brute Force Attempt - Description

**Date:** 2026-11-08
**Analyst:** Burak CANKURT
**Lab:** Home-Lab (Wazuh)
**Severity:** Medium

## Summary
On 2026-11-08 at 14:24:15, Wazuh detected multiple login failures. The source tried to authenticate with SSH against the endpoint "home-lab" (192.168.1.104) from source  192.168.1.103. Attempts occurred from 14:24:15 to 14:24:25. The timing and repetition match a brute-force attack.

## Alert Details
- Source: 192.168.1.103
- Agent-name/target: home-lab
- MITRE ID: T1110.001, T1021.004
- MITRE Tactic: Credential Access, Lateral Movement
- MITRE technique: Password Guessing, SSH
- Failed Attempts: 3
- Rule ID: 2502
- Rule Level: 10 
- Full Log: Nov 08 14:24:27 home-lab sshd-session[2661]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.1.103  user=attacker

## Findings
- From 192.168.1.103 login attempts occurred over a 10 second window. 
- Wazuh's correlation rule 2502 ("Users missed the password more than one time", level 10) then fired, grouping the repeated failures and confirming the brute-force pattern.

## Assessment
- **Detection: True Positive** The alert correctly identified real failed authentication matching a brute-force pattern. 
- The source is trying to gain access by guessing credentials.
- If this problem gets disregarded, problems such as these arise:
    - data theft 
    - malicious program injection
    - lateral movement 
    - persistence 
- In this scenario this guessing attempt was made by me so this is benign.
- In production, the same alert from an unknown source would be escalated to Tier 2 as a possible attack. 

## Recommendation
- Verify the source IP address. If it is unknown escalate.
- Block the IP, lock the target account and isolate the host.  
- Disable SSH to password auth, enforce MFA + account lockout, add rate-limiting/fail2ban, restrict SSH to known IPs.
- Tune brute-force thresholds; alert on "success after multiple failures". 
