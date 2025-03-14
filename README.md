# Building-My-Own-Soc-Lab

## Tools:
- Virtual Machine (VM): VirtualBox (Used for creating the Ubuntu VMs)
- Ubuntu Server (Operating system for both Splunk server and forwarder)
- Splunk: Splunk Enterprise (Server)
- Splunk Universal Forwarder (Client)
- Suricata(IPS tool)
- OSSEC(Host IPS)
- Wireshark
- Kali
- Nessus
- Linus(Command Line Security Auditing Tool)
## Command-Line Tools:
- ssh (For remote access, if needed)
- netstat (For network port checking)
- lsof (For identifying processes using ports)
- tail (For viewing log files)
- nano or vim (For editing configuration files)
- dpkg (For installing .deb packages)
- ufw (For firewall management)
- ping (For checking network connectivity)
- telnet (For checking port connectivity)
- ps (for process status)
- df (for disk usage)
- grep (for searching within files)
- chown (for changing file ownership)
- ls (for listing directory contents)
- mkdir (for making directories)
- rm (for removing files and directories)

## Project Steps:

## 1. Virtual Machine Setup:
- Download and install VirtualBox.
- Download the Ubuntu Server ISO image.
- Install Ubuntu Server on VM.
- Now go to the ubuntu network settings and set adapter1 network type to Internal network because we need that machine to be isolated an set static ipv4 address ranging from 192.168.56.0/24 and add adapter2  and set network type to NAT Network because to download the required things in future  .

## 2. Splunk Server Installation:
- Download the Splunk Enterprise .deb package in ubuntu.
### Install Splunk Enterprise:
sudo dpkg -i splunk-enterprise-*.deb
- Most important thing is make sure to replace the splunk-enterprise-*.deb to your actual downloaded file name
### Start Splunk:
sudo /opt/splunk/bin/splunk start --accept-license
### Configure a Splunk TCP input on port 9997 in /opt/splunk/etc/system/local/inputs.conf:
- (Edit /opt/splunk/etc/system/local/inputs.conf and add: [tcp://9997], connection_host=ip)
- Ensure that the outputs.conf file does not contain any forwarding information that sends data back to the server.
### Verify that the Splunk server is listening on port 9997:
sudo netstat -tuln | grep 9997
### Access the Splunk web interface:
http://<server_ip>:8000
here change server_ip to your adapter 2 ip address

## 3. Splunk Forwarder Installation:
- Download the Splunk Universal Forwarder .deb package.
### Install the forwarder:
cd Downloads/
sudo dpkg -i splunkforwarder-*.deb
### Start the forwarder:
sudo /opt/splunkforwarder/bin/splunk start --accept-license
### configure the forwarder to send data to the Splunk server:
sudo /opt/splunkforwarder/bin/splunk add forward-server <server_ip>:9997
### Restart the forwarder:
sudo /opt/splunkforwarder/bin/splunk restart
### Correct any file permission issues that arise with the fishbucket directory.
### Verify the forwarder's status:
sudo /opt/splunkforwarder/bin/splunk status
### Check the forwarder's logs for errors:
sudo tail -n 20 /opt/splunkforwarder/var/log/splunk/splunkd.log

## 4. Firewall Configuration:
- Configure the firewall (ufw) on VM to allow communication on port 9997:
sudo ufw allow 9997
sudo ufw enable

## 5. Data Collection and Analysis:
- Configure input data streams on the forwarder (e.g., log files):
- Edit /opt/splunkforwarder/etc/system/local/inputs.conf and add:
  [monitor:///var/log/syslog]
  sourcetype = syslog
- you can add other files that you need to monitor the logs.
- Search and analyze the collected data in the Splunk web interface.

## Key Considerations:
- Network Connectivity: Ensure that the server and forwarder VMs can communicate with each other.
- File Permissions: Pay close attention to file permissions, especially in the fishbucket directory of the forwarder.
- Splunk Configuration: Double-check the configuration files (inputs.conf, outputs.conf, server.conf).
- Log Files: Regularly check the Splunk server and forwarder log files for errors.

## Suricata (IPS Tool)

### 1. Suricata Installation:

sudo apt update
sudo apt install suricata -y
### 2. Suricata Configuration (Editing suricata.yaml):

sudo nano /etc/suricata/suricata.yaml
-Edited af-packet to use enp0s8 (or the correct interface).
-Edited eve-log to enable eve.json logging.
-Saved and closed the file.
### 3. Suricata Rules Update:

sudo apt install suricata-update -y
sudo suricata-update
### 4. Suricata User and Permissions:

sudo groupadd suricata
sudo useradd -g suricata suricata
sudo chown -R suricata:suricata /var/lib/suricata/rules
sudo chown suricata:suricata /var/lib/suricata/rules/classification.config
sudo chown suricata:suricata /var/lib/suricata/rules/suricata.rules
### 5. Suricata Service Management:

sudo systemctl start suricata
- 6. Ensure the splunk server and forwarder are running if not turn on like we did previously
### 7. Splunk Forwarder Configuration (inputs.conf):

sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf

[monitor:///var/log/suricata/eve.json]
sourcetype = suricata

- add the above 2 lines save and close the file.

### 8. Splunk Forwarder User and Permissions:

sudo chown splunkfwd:splunkfwd /var/log/suricata/eve.json
### 9. Splunk Forwarder Service Management:

sudo /opt/splunkforwarder/bin/splunk start --accept-license

### 10. Verify Logs in Splunk:

Access Splunk Web UI.
Search: index=* sourcetype=suricata
## 11. Forwarder Log Check:

- If you face any error then check below command and solve it.
sudo tail -n 50 /opt/splunkforwarder/var/log/splunk/splunkd.log

## OSSEC(HostIPS)

### 1. OSSEC Installation:

- Installed necessary dependencies: build-essential, libssl-dev, zlib1g-dev, libpcre2-dev, libsystemd-dev.
- Downloaded OSSEC from GitHub.
- Extracted the OSSEC archive.
- Ran the OSSEC installer (install.sh).
- Followed the installation prompts, choosing "server" installation, enabling log analysis, rootcheck, and integrity checks.
- Whitelisted the Kali VM's IP address(attacking machine).
- Enabled remote syslog.
### 2. OSSEC Service Management:

- Started the OSSEC service: /var/ossec/bin/ossec-control start.
- Checked OSSEC status: /var/ossec/bin/ossec-control status.
### 3. Splunk Forwarder Configuration:

Edited /opt/splunkforwarder/etc/system/local/inputs.conf to add the OSSEC log file as a monitor:
[monitor:///var/ossec/logs/alerts/alerts.log]
sourcetype = ossec
Restarted the Splunk Forwarder: /opt/splunkforwarder/bin/splunk restart.
- 4. Make sure the splunk sever and forwarder are running
### 5. OSSEC Log Verification in Splunk:

- Accessed Splunk Web.
- Ran the search: index=* sourcetype=ossec.
- Verified OSSEC events were present in the search results.
### 6. OSSEC Testing:

Modified the /etc/passwd file to generate a test alert.
Checked the OSSEC alerts log: /var/ossec/logs/alerts/alerts.log.

## Wireshark
### 1. Wireshark Installation:

sudo apt update
sudo apt install wireshark -y
### 2. Wireshark Permissions Configuration:

Answered "Yes" to the prompt "Should non-superusers be able to capture packets?" during installation.
Added the user to the wireshark group: sudo usermod -aG wireshark $USER (replacing $USER with the actual username).
Applied group changes: Logged out and back in, or ran newgrp wireshark.
### 3. Wireshark Launch and Basic Capture:

Launched Wireshark: wireshark (or through the applications menu).
Selected the network interface (e.g., enp0s8).
Started capturing packets by double-clicking the interface.
Generated network traffic (e.g., browsing, pinging).
Stopped the capture by clicking the red square "Stop capturing packets" button.
### 4. Basic Traffic Analysis:

Explored packet details in the packet details pane.
Apply display filters (e.g., http, dns, icmp, tcp.port == 80).
Follow TCP streams.

## Nessus Installation In Kaki( Attacking VM)

### 1. Tenable Account Creation:

- Navigat to the Tenable Nessus Essentials download page.
- Enter name and email address to create a Tenable account.
- Receives an activation code via email.
### 2. Nessus Essentials Download:

- Used the link in the email to download the Nessus Essentials installer (.deb file) for the 
  appropriate operating system (Kali Linux in this case).
### 3. Nessus Essentials Installation (on Kali Linux):

- Opened a terminal and navigated to the download directory.
- Installed Nessus Essentials using
sudo dpkg -i <nessus_file.deb> (replacing <nessus_file.deb> with the actual filename).
### 4. Nessus Service Startup:

Started the Nessus service using sudo systemctl start nessusd.service.
### 5. Nessus Web Interface Access:

- Opened a web browser and navigated to https://localhost:8834.
Bypassed the self-signed certificate warning.
6. Nessus Configuration:

- Followed the on-screen instructions to configure Nessus.
- Entered the activation code received from Tenable.
- Waited for Nessus to download and install plugins.
### 7. Nessus Ready for Scans:

- Nessus is now ready to perform vulnerability scans.

## Linus 

### 1. Installation:

- use the apt package manager, which is the standard way to install software on Ubuntu.
- The command we used was:

sudo apt install lynis
sudo grants administrative privileges, allowing us to install software.
apt install tells the package manager to install the specified package (lynis).
### 2. Verification:

- After installation, we verified that lynis was installed correctly by checking its version:
lynis --version
- This command displays the installed version of lynis.
### 3. Running an Audit:

- We then ran a basic system audit using lynis:

sudo lynis audit system
- sudo is again used for administrative privileges, as lynis needs to access system files.
- lynis audit system tells lynis to perform a comprehensive security scan of the system.
### 4. Reviewing the Report:

- lynis generated a report with the scan results.
- We note that the report includes:
- Warnings and suggestions for improving system security.
- Detailed information about the scan.
- The report is also saved to /var/log/lynis.log for later review.


