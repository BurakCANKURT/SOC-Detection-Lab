# Nmap + DoS - Description

**Date:** 2026-11-10
**Analyst:** Burak CANKURT
**Lab:** Home-Lab (Wazuh)
**Severity:** High

## Summary
On 2026-11-10 at 20:55:19 Wazuh detected port scan and SYN flood at the sensor (20 each); only 8 scan alerts reached the SIEM and 0 flood alerts (lost to queue saturation). This phase continued from 20:55:19 to 20:57:03. After that 7 times "Agent event queue is full. Events may be lost." alert occurred and at the end "Agent event queue is back to normal load." alert occurred. These alerts continued  from 20:57:05 to 20:58:25. DoS attack pattern was detected at sensor, lost before SIEM. DoS attack filled the Wazuh's queue stack and prevented Wazuh's detection. 

## Alert Details
### Phase 1 - TCP Port and SYN flood
| Phase | Source → Target | Suricata SID / Signature | Wazuh Rule/Level| @Sensor(eve.json) | @SIEM(alerts.json)| Reached SIEM ? |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Targeting – port scan | .106 → .104 | 1000002 / LOCAL TCP port scan | 86601 / L3 | 20 | 8 | Yes | 
| DoS – SYN flood | .106 → .104:80 | 1000001 / LOCAL SYN flood | 86601 / L3 | 20 | 0 | Lost (Queue Full) |  
| DoS - Pipeline impact| - → home-lab | -- | 203 / L9 (Agent buffer full) | -- | 7 | Yes (For signal No) | 
## Findings
- From 20:55:19 to 20:57:03 TCP port scan and SYN flood alerts occurred sequentially. 
- 20 for TCP port scan and 20 for SYN flood alert occurred at sensor; 8 for TCP port scan and 0 SYN flood alert occurred at SIEM within ~2 s. These attempts probably completed with nmap tool. (For flood hping3 tool was used.) 
- From 20:57:05 to 20:58:25 6 times "Agent event queue is full. Events may be lost." (Level 9) with full_log: "wazuh: Agent buffer: 'full'." and finally "Agent event queue is back to normal load." (Level 3) alert occurred within ~1 s. 
- This pattern shows that some tool automatically filled the queue and after that canceled. 
- Wazuh did not correlate with these patterns but, at Suricata there are 20 SYN flood detected. 
- Combining them whole pieces it shows that this is DoS attack.
- Loss is proportional to the volume. 60% loss at the scan and 100% at flood. 


## Assessment
- **Detection: True Positive** The terminal alerts correctly correlated with DoS attack pattern. 
- The source first used nmap tool to recon the network. After that executed DoS attack. 
- If this problem gets disregarded, the service's availability is lost
- Impact is twofold: Service availability and monitoring visibility: the flood  blinded its own detection at SIEM.

## Recommendation
- Verify the source IP address. If it is unknown escalate.
- Block the source IP (192.168.1.106) at the firewall.
- Enable SYN rate-limiting / SYN cookies on the target and the gateway.
- Keep firewall/IDS rules and signatures up to date.
- detect-better / harden pipeline
    - Set flood rules as type limit. 
    - Extend agent's  queue size.