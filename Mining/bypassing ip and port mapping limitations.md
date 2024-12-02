ADVANCED NETWORK GUIDE: BYPASSING IP AND PORT MAPPING LIMITATIONS

This guide helps you bypass Carrier-Grade NAT (CGNAT) and other network restrictions when you can’t modify port mapping rules or manage multiple rigs with a single public IP. By using a VPS with a public IP and a VPN service like ZeroTier, Tailscale, or Wireguard, you can redirect traffic to your rigs .



##### Who is this for?

Users facing CGNAT or similar network limits
Those unable to change port mapping or forwarding rules due to ISP limitations
Managing multiple rigs with one public IP



##### What you will achieve:

Set up a secondary SSH service on your VPS
Configure Fail2Ban for SSH security
Establish a VPN connection between your VPS and rigs using ZeroTier
Redirect traffic through the VPN for external rig access
Install and configure Loopin on your rigs



##### Prerequisites:

A VPS with a direct public IP (static or dynamic with DDNS)
Good to advanced Linux and networking skills
ZeroTier or similar VPN installed on both VPS and rigs
Familiarity with command-line and system configurations
IPv4 only, --> DISABLE IPV6 on the rig(s)



##### Disclaimer:

This guide involves advanced network configurations that may affect connectivity and security. Proceed at your own risk. The author is not responsible for any damage, data loss, or security issues. Back up your configurations and understand each step before proceeding



################################################# COMPLETE GUIDE:

**Bypass CGNAT or other network limitations.**
For those who cannot change "port mapping" rules.
For those who have multiple rigs but only one public IP address.
#################################################

**PREREQUISITES**
A VPS with a full and direct public IP address (whether static or dynamic with a DDNS service).
GOOD knowledge of Linux.
A VPN (e.g., ZeroTier).
Advanced Linux knowledge is highly recommended to troubleshoot any issues.
Warning: I assume no responsibility for the use of this guide. Use it at your own risk.

Attention Regarding the VPS
Modifying the SSH configuration on a VPS is delicate. An error could cause you to lose access to the server. To avoid this, we will configure a second SSH service on port 2222 without touching the default port 22. This way, you can always access the VPS even after the modifications.

**Recommendation:** Before modifying the SSH configuration, open at least two SSH sessions on your VPS and run htop in each to keep them active (to prevent timeout). If SSH fails or does not restart, you can use these sessions to correct any errors.

**Configuration on the VPS**

1. Install a Second SSH Service on Port 2222
  Copy the existing SSH configuration:
  sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config_custom
  Edit the custom configuration file:
  sudo nano /etc/ssh/sshd_config_custom
  Comment out the line Port 22 if it is present.
  Comment out the line Include /etc/ssh/sshd_config.d/*.conf if it exists.
  Add Port 2222.
  Create a systemd service for the new SSH instance:
  sudo nano /etc/systemd/system/sshd-custom.service


  

File content:

[Unit]
Description=OpenBSD Secure Shell server (custom instance on port 2222)
After=network.target

[Service]
ExecStart=/usr/sbin/sshd -D -f /etc/ssh/sshd_config_custom
ExecReload=/usr/sbin/sshd -t
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target



Reload systemd and start the new SSH service:
sudo systemctl daemon-reload
sudo systemctl start sshd-custom.service

Test SSH connection on port 2222 to ensure everything works.

Enable the service to start on boot:
sudo systemctl enable sshd-custom.service

2. Configure Fail2Ban to Secure SSH on Port 2222
Edit the Fail2Ban configuration:
sudo nano /etc/fail2ban/jail.local
Add:
[sshd-2222]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 10m

Restart Fail2Ban:
sudo systemctl restart fail2ban
Verify that the new jail is active:


sudo fail2ban-client status
You should see:
Status
- Number of jail:      2
- Jail list:   sshd, sshd-2222

Important: It is recommended to install Fail2Ban for port 22 on the VPS and on your rigs as well.

3. Install ZeroTier and Connect the Machines
Install ZeroTier on both the VPS and the RIG by following the official documentation.

Connect both machines to the same ZeroTier network via the web interface.

The VPN will assign an additional IP address to each of your machines.

Part 2: Network Configuration adding RIG
Example:
Public IP of the VPS: 163.172.xxx.x1
ZeroTier IP of the VPS: 172.28.99.114
ZeroTier IP of the RIG: 172.28.167.244

1. **On the RIG**
Redirect traffic through the VPN (to be done after each reboot):
sudo ip route add default via 172.28.99.114 dev <zerotier_interface>
Replace <zerotier_interface> with the name of the interface created by ZeroTier (e.g., zt0). You can use the command ip addr to identify the correct interface name.

Enable IP forwarding:
sudo nano /etc/sysctl.conf
Add or modify the line:

net.ipv4.ip_forward = 1

Apply the changes:
sudo sysctl -p

2. **On the VPS**
Enable IP forwarding:
sudo nano /etc/sysctl.conf
Add or modify the line:

net.ipv4.ip_forward = 1

Apply the changes:
sudo sysctl -p

Allow certain ports via iptables (to be done after each reboot):
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 2222 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 9789 -j ACCEPT

Configure port forwarding rules (to be done after each reboot):
##### For port 22 (SSH to the RIG)
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x1 --dport 22 -j DNAT --to-destination 172.28.167.244:22
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.244 -j MASQUERADE

##### For port 10250
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x1 --dport 10250 -j DNAT --to-destination 172.28.167.244:10250
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.244 -j MASQUERADE

##### For port 10256
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x1 --dport 10256 -j DNAT --to-destination 172.28.167.244:10256
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.244 -j MASQUERADE

##### For the port range 30000:32767
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x1 --dport 30000:32767 -j DNAT --to-destination 172.28.167.244
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.244 -j MASQUERADE

##### For port 9789
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x1 --dport 9789 -j DNAT --to-destination 172.28.167.244:9789
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.244 -j MASQUERADE

**Installing Loopin on the RIG**
Run the following commands:

curl -sS -o "./loopin" "https://files.loopin.network/loopin"
sudo chmod +x loopin
sudo ./loopin --code xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your Loopin code.



Practical Case: Adding a Second RIG
Information:
New public IP for the VPS: 163.172.xxx.x2 (purchasing an additional public IP).
Second ZeroTier IP for the VPS: 172.28.99.115.
ZeroTier IP of RIG2: 172.28.167.245.

Steps:
On the VPS
Configure the new public IP (usually via netplan or another network manager).

Assign a second ZeroTier IP to the VPS via the ZeroTier interface.

On RIG2
Redirect traffic through the VPN (to be done after each reboot):
sudo ip route add default via 172.28.99.115 dev <zerotier_interface>


On the VPS
Configure port forwarding rules for RIG2 (to be done after each reboot):
##### For port 22 (SSH to RIG2)
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x2 --dport 22 -j DNAT --to-destination 172.28.167.245:22
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.245 -j MASQUERADE

##### For port 10250
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x2 --dport 10250 -j DNAT --to-destination 172.28.167.245:10250
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.245 -j MASQUERADE

##### For port 10256
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x2 --dport 10256 -j DNAT --to-destination 172.28.167.245:10256
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.245 -j MASQUERADE

##### For the port range 30000:32767
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x2 --dport 30000:32767 -j DNAT --to-destination 172.28.167.245
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.245 -j MASQUERADE

##### For port 9789
sudo iptables -t nat -A PREROUTING -p tcp -d 163.172.xxx.x2 --dport 9789 -j DNAT --to-destination 172.28.167.245:9789
sudo iptables -t nat -A POSTROUTING -p tcp -d 172.28.167.245 -j MASQUERADE

Installing Loopin on RIG2
Run the following commands:

curl -sS -o "./loopin" "https://files.loopin.network/loopin"
sudo chmod +x loopin
sudo ./loopin --code xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

##### Final Notes

*Non-persistent iptables rules:* 

You can make these rules persistent so they survive reboots (e.g., using iptables-persistent or adding them to a startup script). However, not making them persistent can be a security measure. In case of a problem or misconfiguration, rebooting the VPS/RIG will disable these rules.

Why a second SSH service? After applying the iptables rules, port 22 on the VPS will redirect to the RIG. The second SSH service on port 2222 allows you to continue accessing the VPS despite the iptables rules.

*Points to Note:*

##### SSH Security: 

Redirecting port 22 to the RIG exposes it to external attacks. Ensure that SSH access on the RIG is secured (use Fail2Ban, etc.).

Variable ZeroTier Interface: The ZeroTier interface (e.g., zt0) may have a different name on your machine. Use the command ip addr (ip a) to identify the correct interface name.

Modifying the Default Route on the RIG: By redirecting the default route through the VPN, all traffic from the RIG will pass through the VPS. This can affect network performance. You might consider redirecting only the necessary traffic if you want.

Installing Fail2Ban on the RIG: It is highly recommended to install Fail2Ban on the RIG to protect the SSH port.

##### Summary of How It Works

Loopin will be installed on the RIG and will use the native public IP of your VPS to access the RIG machine.

When the VPS receives requests intended for Loopin, it will redirect them to the ZeroTier IP of your RIG.

Note: Once the iptables rules are applied on the VPS, if you try to connect to it on port 22, you will be redirected to the RIG. That's why we configured a second SSH service on port 2222 to maintain access to the VPS despite the iptables rules.



##### Note

In some cases, IPv4 forwarding may not be properly applied on your rig and/or VPS. 

As a last resort, to ensure IPv4 forwarding is functional, you can define it directly in GRUB. 

However, proceed with caution: only do this if you know what you are doing. 

A misconfiguration in GRUB can result in your machine requiring manual intervention to reboot properly. 

Follow these steps carefully if you decide to proceed. 

sudo nano /etc/default/grub add net.ipv4.ip_forward=1

like this: GRUB_CMDLINE_LINUX_DEFAULT="quiet splash net.ipv4.ip_forward=1" 

then update grub: sudo update-grub 

and reboot: sudo reboot

<u>If you don't want to edit grub</u>: Do this, but do it at each reboot: sudo sysctl -w net.ipv4.ip_forward=1 sudo sysctl -p 
