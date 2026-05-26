# SOC Analyst lab with LimaCharlie EDR and Sliver C2
Link to the Project (https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-intro)

## Objective
1. Build and configure a virtual SOC lab environment

2. Detect, analyze, and respond to simulated adversary activity

3. Create and tune detection & response rules in LimaCharlie.

4. Investigate endpoint telemetry and security events.

### Skills Learned

1.SOC Environment Configuration

2.Threat Dectection and Analysis

3.Detection & Response Rule Creation

4.Endpoint Telemetry investigation

5.Incident Response and Threat Mitigation

6.Security Event Monitoring and Reporting
### Tools Used

1.VMware Workstation Pro

2.Ubuntu Server 22.04 LTS

3.Windows 11 VM

4.LimaCharlie EDR Platform

5.Sysmon

6.Sliver C2

### Relevant MITRE ATT&CK Techniques

 ID      |	   Technique Name	                  | How It Was Demonstrated

T1071	   |    Application Layer Protocol	      | Sliver HTTP C2 communication

T1003	   |    OS Credential Dumping	            | LSASS access detection activity

T1490	   |    Inhibit System Recovery	          | vssadmin delete shadows /all detection and blocking

T1059	   |    Command and Scripting Interpreter |	PowerShell and command execution telemetry

T1105	   |    Ingress Tool Transfer	            | Payload download from attacker VM


## Steps
# PART 1 - Environment Setup
The first phase focused on building the virtual lab environment
VMware Workstation Pro was used to create both the Ubuntu attacker machine and the Windows 11 victim machine.
The Ubuntu virtual machine(Server 22.04)was configured with:

14 GB Disk Space

2 CPU Cores

2 GB RAM

After the initial deployment, network connectivity testing showed that DNS resolution was not functioning correctly on the Ubuntu attacker VM

<img width="665" height="185" alt="Untitled" src="https://github.com/user-attachments/assets/730596a8-71fd-428f-9289-0b5156190e7c" />


To resolve the issue, the DNS configuration was updated (sudo nano /etc/resolv.conf) by adding Google’s public DNS server (8.8.8.8). Once the configuration was updated, connectivity was confirmed.

<img width="729" height="389" alt="Untitled" src="https://github.com/user-attachments/assets/227990eb-e744-4ca9-ab8b-a69caa0b87a5" />


This Unbuntu VM will act as an attacker. Microsoft Defender was disabled on windows VM to allow controlled malware simulation and C2 execution without interference.

Sysmon was installed to improve endpoint visibility and generate detailed telemetry.

I installed LimaCharlie to the Windows VM to collect telemetry and provide centralized monitoring.

![Screenshot (10)](https://github.com/user-attachments/assets/9b6e2b30-f9ef-49d2-ab4a-b85c0c0ae42d)

Once the installation completed, the Windows endpoint began reporting telemetry to LimaCharlie.


# Part 2 - Command and Control (C2)
The second phase focused on simulating attacker behavior using Sliver C2.

A Windows payload was generated from the Ubuntu attacker machine using Sliver.

<img width="823" height="308" alt="Screenshot (133)" src="https://github.com/user-attachments/assets/ce6d63f2-49ab-4e1c-8917-1a3127fe92f5" />

The payload established communications between the victim(Windows VM) endpoint and the attacker system(Ubuntu VM).

The payload was then downloaded and executed on the Windows victim machine.

Once executed, the Sliver server established an active C2 session with the victim machine.

<img width="837" height="301" alt="Untitled" src="https://github.com/user-attachments/assets/ea34477f-cc76-4c0f-b9bb-bd084686974e" />

The successful callback generated endpoint telemetry within LimaCharlie, including:

1.Process execution events

2.Network connection telemetry

3.Windows Event Log activity

4.Command-line execution data

The telemetry was reviewed within LimaCharlie’s Timeline and Network views to validate visibility into the simulated attack activity.
![Screenshot (19)](https://github.com/user-attachments/assets/98a6d627-e8a9-426b-8eb5-4bf4c50ed02f)
![Screenshot (20)](https://github.com/user-attachments/assets/57a508ca-8069-493c-a930-6a9290061431)

# PART 3 - Detection Engineering
The third phase focused on creating detections for suspicious activity associated with credential access behavior.

Within LimaCharlie’s Timeline view, the SENSITIVE_PROCESS_ACCESS event type was used to locate activity involving lsass.exe, which is commonly targeted during credential dumping attacks.
![image](https://github.com/user-attachments/assets/f72581a8-aa8d-424e-9cb9-1838318235a3)

A custom Detection & Response (D&R) rule was then created within LimaCharlie to identify suspicious access attempts involving lsass.exe.

After creating the rule, test events were used to validate that the detection logic correctly identified the targeted activity.

![image](https://github.com/user-attachments/assets/1241def0-18b1-4627-beec-89db52d063ec)

# PART 4 - Response & Mitigation
The final phase focused on implementing automated defensive actions using LimaCharlie D&R rules.

A detection rule was created to identify execution of the vssadmin delete shadows /all command, which is commonly associated with ransomware activity and destructive behavior.

<img width="818" height="221" alt="Screenshot (119)" src="https://github.com/user-attachments/assets/f741bccc-694c-4940-81e1-f3e0865980af" />

Telemetry related to the command execution was identified within LimaCharlie’s Timeline view.

![Screenshot (22)](https://github.com/user-attachments/assets/c2e071f6-d62f-4d41-af42-a06be25fbb49)

A custom D&R rule was then configured to:

Detect execution of vssadmin.exe

Identify shadow copy deletion commands

Trigger an automated response action
The response action used LimaCharlie’s deny_tree capability to terminate the offending process tree.

![Screenshot (23)](https://github.com/user-attachments/assets/b1c492de-c1d5-41f5-8eb1-1d4cde8397be)

After deploying the rule, the command was executed again to validate the response behavior.

The rule successfully detected the activity and terminated the shell session, demonstrating successful automated mitigation of the simulated attack.

![Screenshot (24)](https://github.com/user-attachments/assets/3cd1adb0-4b54-4fce-bb40-bb93b8689fb0)

# Conclusion 

This project shows the importance of endpoint visibility, telemetry analysis, and detection tuning when identifying and responding to malicious activity within enterprise environments.

By combining attack simulation with defensive monitoring and automated response capabilities, the lab demonstrated how modern SOC workflows can be used to detect and mitigate threats in real time.
