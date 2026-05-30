# Wazuh-SOC-Homelab
Created an SOC homelab deploying a containerized Wazuh SIEM stack on Ubuntu Server to ingest, parse, and analyze Windows endpoint security telemetry.

#Tools Used
Ubuntu Server: Runs the Wazuh SIEM dashboard.
Windows Endpoint: Machine used to run programs, execute processes, and being monitored

* Both machines will communicate with one another on an isolated network.


# Mini Network Topology

                              Host System (Physical Laptop)
                                             |
              Ubuntu SIEM VM             < - -  -          Windows Client VM
           (Wazuh Server)   (Logs sent to Wazuh Server)  (Wazuh Agent)

# Use Case: Failed Authentication (Brute Force Monitorting)

# Scenario Objective
To verify that the host-based Wazuh Agent successfully captures local security events on the Windows Client and securely forwards to the Wazuh Server in Ubuntu.

# Execution steps
1. Navigate to Windows Client, sign out of the active user session.
2. Attempt to log in using an invalid username and random password
3. Repeat the invalid login attempt multiple times (at least 5 times) within a short interval time.

# Local Endpoint Ingestion Verification
To validate the Windows Kernel is properly auditing the behavior locally before shipping the log:
* Search for open **Event Viewer** ('event.msc') inside the Windows VM.
* Expand windows logs on the left margin and select the security log
* We should be able to observe flag entries with EventID 4625 which suggest failed user login

# Expected Wazuh Dashboard output
* Alert Rule triggered: 18152 (Wazuh core signature database file)
* Alert Level Description: Level 10 - Multiple Authentication Failures.
* In the Dashboard, it will display a sharp spike in alerts indicating that the target attempted an invalid login using invalid username and password which led to the login failure event.


