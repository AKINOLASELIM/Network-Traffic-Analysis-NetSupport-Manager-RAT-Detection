# Network-Traffic-Analysis-NetSupport-Manager-RAT-Detection
# Network-Traffic-Analysis-NetSupport-Manager-RAT-Detection
Project Overview
As a SOC Analyst, I received a SIEM alert for NetSupport Manager RAT activity on 2026-02-28 at 19:55 UTC. The alert flagged an internal host communicating with the external C2 IP 45.131.214[.]85 over TCP port 443.
Using Wireshark, I retrieved and analyzed the packet capture (PCAP) to:

Identify the compromised host and user account
Confirm active C2 beaconing behaviour
Hunt for network-level IOCs
Document findings and recommend mitigation steps

Dataset Information
Source: Malware-Traffic-Analysis.net
Scenario Date: 2026-02-28
Original PCAP-https://www.malware-traffic-analysis.net/2026/02/28/index.html

Tools Used
Wireshark-PCAP analysis and packet inspection
SIEM- Initial alert detection
Virus- TotalC2 IP reputation and IOC enrichment
Unit 42 Threat Intel -NetSupport RAT IOC reference

Investigation Analysis
1. Understand the Environment
Used Statistics → Conversations → Endpoints to map all active hosts and identify the top talkers in the capture before applying any filters.
<img width="800" height="450" alt="statistics" src="https://github.com/user-attachments/assets/3f442783-10bb-4742-ba13-b129033f655a" />
2. Confirm the C2 Communication
ip.addr == 45.131.214.85
All traffic to the known C2 IP was exclusively between 45.131.214[.]85 and internal host 10.2.28.88 — confirming a single compromised machine.

3. Identify the Victim
   ip.addr == 10.2.28.88 && kerberos.CNameString
   Kerberos authentication traffic revealed the victim username as brolf. MAC address captured from packet-level Ethernet details.
<img width="2560" height="1440" alt="kerberos" src="https://github.com/user-attachments/assets/b3966515-cbe0-4766-b2c2-f0517efe1f72" />

4. Credential Hunting (Blocked)
Attempted to extract login credentials — blocked by TLS encryption on port 443. Without the SSLKEYLOGFILE, HTTPS traffic could not be decrypted.

5. IOC Discovery
http.request
Identified a suspicious HTTP POST to http://45.131.214[.]85/fakeurl.htm. Following the HTTP stream confirmed the User-Agent: NetSupportManager/1.3 — directly identifying the RAT family.

6. Beaconing Confirmed
   http.request.method == "POST" and ip.addr == 45.131.214.85
   Multiple POST requests to /fakeurl.htm at regular second-by-second intervals — confirming the host was actively under remote attacker control.


Key Findings
1.1C2 Beaconing Confirmed — 10.2.28.88 beaconing to 45.131.214[.]85/fakeurl.htm every few seconds
2.RAT Identified — User-Agent NetSupportManager/1.3 confirmed in HTTP stream
3.Protocol Anomaly — HTTP tunnelled over TCP/443 to evade port-base
4.Credentials Inaccessible — TLS encryption blocked credential recovery without SSLKEYLOGFILEd firewall rules
5.Single Host Compromised — No lateral movement detected at time of analysis
6.User Account Exposed — Username brolf recovered via Kerberos traffic

Victim Profile
Internal IP- 10.2.28.88
MAC Address- 00:19:d1:b2:4d:ad
Hostname- DESKTOP-TEYQ2NR
Username- brolf
DomainEASYAS123 / easyas123[.]tech

IOC Summary
C2 IP- 45.131.214[.]85
C2 Port- TCP 443
C2 URI- /fakeurl.htm
User-Agent- NetSupportManager/1.3
RAT Family- NetSupport Manager RAT
Beacon Interval- Every few seconds
Geolocation Beacon- geo.netsupportsoftware[.]com

Mitigation & Next Steps
Immediate (0–24 hrs)
*Isolate DESKTOP-TEYQ2NR from the network
*Disable user account brolf in Active Directory
*Block 45.131.214[.]85 at perimeter firewall and DNS

Short-Term (1–7 days)
Reimage the infected machine
Reset passwords for brolf and all accounts on that host
Hunt for same C2 IP across all endpoints in SIEM/firewall logs
Deploy EDR on all endpoints

Long-Term (1–4 weeks)

Implement DNS filtering (Cisco Umbrella / Cloudflare Gateway)
Deploy network behavioural analytics to auto-detect beaconing
Restrict remote access tool installations via AppLocker/WDAC
Conduct user awareness training on Google ad-based malware delivery
Enable SSL inspection or SSLKEYLOGFILE capture in test environments


Author
Akinola selim ishola |security Operations Center
📅 Date: 2026-02-28


