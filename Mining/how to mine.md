Please follow these simple guidelines:

- Ensure you meet the project's prerequisites:
- Do you have 3090, 4090, or more powerful GPUs? Nothing below these is supported.
- Have enough "stacked" resources to get started? For eg, you need around 50 Loopin per 3090. To check the current needed amount of loopin you can have a look here :
  https://loopinrewards.ddns.net/
- Network-wise, you need a full-stack IP, or you must use a VPS/VPN.
- Map your ports:

For loopin you will need to map these tcp ports: 22 9789 10250 10256 range 30000-32767

- Make sure your network ports are mapped. 
- Test your ports:

Get your "public ip" with this command: curl ifconfig.me then test the ports: nc -zv your_public_ip port_number Test all ports except the range of 30000.

- Use Linux, specifically Ubuntu. WSL is not supported.
- Verify that your Nvidia GPUs are correctly recognized; the nvidia-smi command must work properly.
- Ensure you meet ALL prerequisites (driver versions, RAM, disk space).

- At the end of the first part of the installation, under Bash, when it says "ok connected on port 9789," immediately check the worker:

sudo systemctl status loopin-worker.service If it shows error EOF, restart it immediately: sudo systemctl restart loopin-worker.service

- Still at the end of the Bash part of the installation, ensure your /etc/hosts file contains (use cat /etc/hosts to view it):

159.69.58.107    easzlab.io.local 

If not, fix it immediately; no reboot is needed to apply this change. At each installation or reinstallation of loopin, the hosts file as to be checked.

- You can use these commands to detect potential issues:

sudo journalctl -u containerd -n 5 -f 
sudo journalctl -u kubelet -n 5 -f 
sudo journalctl -u loopin-worker -n 5 -f

- Ubuntu 22.04 LTS is recommended.
- If you use HiveOS, note that HiveOS resets the /etc/hosts file on every reboot.