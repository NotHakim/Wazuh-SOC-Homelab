# Wazuh-SOC-Homelab
Created a SOC Homelab deploying a containerized Wazuh SIEM stack (via Docker Desktop) to ingest, parse, and analyze Windows endpoint security telemetry, 
with a Kali Linux attacker machine used to simulate attacks.

# Tools
Used Docker Desktop (Host PC): Runs the Wazuh SIEM stack (manager, indexer, dashboard).
Windows 11 Client VM: Endpoint being monitored, running the Wazuh Agent, targeted via RDP.
Kali Linux VM: Attacker machine performs Brute-Force login attempt against the Windows 11 Client.

* All 3 components communicate over an isolated NAT network (VMware).


# Mini Network Topology

                              Host System (PC)
                              Docker: Wazuh Manager
                                             |
              Kali Attacker VM        ------>          Windows Client VM
                      (Logs forwarded to Wazuh
                      Manager via Wazuh Agent on Windows VM)       (Wazuh Agent)

# Use Case 1: RDP Brute-Force Authentication Monitoring

# Scenario Objective
To verify that the host-based Wazuh Agent on the Windows 11 VM successfully captures RDP Logon events (both failed and successful) and forwards tgem to the Wazuh Manager for analysis, triaging, correlation and MITRE ATT&CK mapping.

# Execution steps
1. Enabled Remote Desktop on the Windows 11 Client VM and disabled NLA requirement to allow connection attempts (isolated lab network only).
2. Added the target user account to the "Remote Desktop Users" group on the Windows VM.
3. From Kali VM, confirmed RDP was reachable via nmap -p 3389 <target ip address>.
4. Attenoted several RDP logins using xfreerdp with incorrect passwords, followed by one attempt with the correct credentials, simulating a brute-force pattern.

# Local Endpoint Ingestion Verification
* Open Event Viewer (eventvwr.msc) on the Windows Client and checked the Security log.
* We should be able to observe flag entries with Event ID 4625 which suggest failed user login, as well as Event ID 4624 for the successful login.

# Expected Wazuh Dashboard output
* 13 Authentication Failure alerts and 8 Authentication Success alerts generated and correlated to the 'DESKTOP-SUC8DVV' agent.
* Alerts automatically mapped to relevant MITRE ATT&CK techniques, including Remote Desktop Protocol (T1021.001) and Valid Accounts (T1078).
* In the Dashboard, it will display a sharp spike in alerts indicating that the target attempted an invalid login using invalid username and password which led to the login failure event.
* Raw Event data confirmed the attacker's source ip via the IPAddress field in the Windows Security Event Log, allowing direct correlation between Kali VM and the logged intrusion attempts.

# Lessons Learned/Troubleshooting
* Hydra's RDP module is unreliable. Initial attempts to use the module proved to be unsuccesful as it would return the following output '[ERROR] freerdp: The connection failed to establish'. Other instances include detecting correct password but reporting the Windows Client account as 'not active'. This is a known limitaion on hydra's part as the module itself identified it as experimental. Due to the unreliable and unstable tool, I pivoted to xfreerdp to manually simulate the attack which produced a successful and clean result.
* Network Level Authentication (NLA) blocked initial connections. This is due to the fact that Windows 11 enables NLA by default as a security feature to prevent any unauthorise remote access which conflicted with hydra's RDP module. I work around this setback by temporarily disabling 'Require devices to use Network Level Authenticationb' (System Properties -> Remote Desktop settings) resolved the connection failures. To add on, NLA should always be enabled in a production environment. This was only adjusted to accomodate the lab tooling.
* Local account lacked RDP permissions by default. Despite entering valid credentials, the login was unsuccessful and rejected with an 'account not active for remote desktop' response. This was resolved by explicitly adding the target user to the Remote Desktop Users local group (lusrmgr.msc -> Groups -> Remote Desktop Users -> Add), a step easy to overlook since Administrators are typically assumed to have RDP access by default in some configurations.

