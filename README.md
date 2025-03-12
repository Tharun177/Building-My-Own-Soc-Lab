# Building-My-Own-Soc-Lab

# Tools:
#   Virtual Machine (VM): VirtualBox (Used for creating the Ubuntu VMs)
#   Ubuntu Server (Operating system for both Splunk server and forwarder)
#   Splunk: Splunk Enterprise (Server)
#   Splunk Universal Forwarder (Client)
#   Command-Line Tools:
#     ssh (For remote access, if needed)
#     netstat (For network port checking)
#     lsof (For identifying processes using ports)
#     tail (For viewing log files)
#     nano or vim (For editing configuration files)
#     dpkg (For installing .deb packages)
#     ufw (For firewall management)
#     ping (For checking network connectivity)
#     telnet (For checking port connectivity)
#     ps (for process status)
#     df (for disk usage)
#     grep (for searching within files)
#     chown (for changing file ownership)
#     ls (for listing directory contents)
#     mkdir (for making directories)
#     rm (for removing files and directories)

# Project Steps:

# 1. Virtual Machine Setup:
#   Download and install VirtualBox.
#   Download the Ubuntu Server ISO image.
#   Create two virtual machines in VirtualBox:
#     Splunk Server VM
#     Splunk Forwarder VM
#   Install Ubuntu Server on both VMs.
#   Ensure that both VMs are on the same virtual network so that they can communicate with each other.

# 2. Splunk Server Installation:
#   Download the Splunk Enterprise .deb package.
#   Transfer the package to the Splunk Server VM.
#   Install Splunk Enterprise:
sudo dpkg -i splunk-enterprise-*.deb
#   Start Splunk:
sudo /opt/splunk/bin/splunk start --accept-license
#   Configure a Splunk TCP input on port 9997 in /opt/splunk/etc/system/local/inputs.conf:
#     (Edit /opt/splunk/etc/system/local/inputs.conf and add: [tcp://9997], connection_host=ip)
#   Ensure that the outputs.conf file does not contain any forwarding information that sends data back to the server.
#   Verify that the Splunk server is listening on port 9997:
sudo netstat -tuln | grep 9997
#   Access the Splunk web interface:
#     http://<server_ip>:8000

# 3. Splunk Forwarder Installation:
#   Download the Splunk Universal Forwarder .deb package.
#   Transfer the package to the Splunk Forwarder VM.
#   Install the forwarder:
sudo dpkg -i splunkforwarder-*.deb
#   Start the forwarder:
sudo /opt/splunkforwarder/bin/splunk start --accept-license
#   Configure the forwarder to send data to the Splunk server:
sudo /opt/splunkforwarder/bin/splunk add forward-server <server_ip>:9997
#   Restart the forwarder:
sudo /opt/splunkforwarder/bin/splunk restart
#   Correct any file permission issues that arise with the fishbucket directory.
#   Verify the forwarder's status:
sudo /opt/splunkforwarder/bin/splunk status
#   Check the forwarder's logs for errors:
sudo tail -n 20 /opt/splunkforwarder/var/log/splunk/splunkd.log

# 4. Firewall Configuration:
#   Configure the firewall (ufw) on both VMs to allow communication on port 9997:
#     On Splunk Server VM:
sudo ufw allow 9997
#     On Splunk Forwarder VM:
sudo ufw allow out 9997
sudo ufw enable

# 5. Data Collection and Analysis:
#   Configure input data streams on the forwarder (e.g., log files):
#     Edit /opt/splunkforwarder/etc/system/local/inputs.conf and add:
#     [monitor:///var/log/syslog]
#     sourcetype = syslog
#   Search and analyze the collected data in the Splunk web interface.

# Key Considerations:
#   Network Connectivity: Ensure that the server and forwarder VMs can communicate with each other.
#   File Permissions: Pay close attention to file permissions, especially in the fishbucket directory of the forwarder.
#   Splunk Configuration: Double-check the configuration files (inputs.conf, outputs.conf, server.conf).
#   Log Files: Regularly check the Splunk server and forwarder log files for errors.
