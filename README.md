# vpn-server-with-wireguard

A comprehensive guide for deploying secure VPN infrastructure using WireGuard and OpenVPN across physical and virtualized environments. This repository documents complete setup instructions for configuring VPN servers on real networks, building isolated testing labs in VirtualBox, and replicating production-grade environments in VMware Workstation/ESXi. It includes detailed coverage of encryption workflows, key exchange mechanisms, routing table configuration, and secure remote-device onboarding, enabling reliable end-to-end testing, learning, and operational deployment.

This will involve three VMs the Server VM, Internal LAN and the Client VM, all Ubuntu, and having to set up the respective networks for it.  
To set up the networks in VMWare, go to Edit up top, click on it and click on Virtual Network Editor it will look like this:  
<img width="598" height="559" alt="image" src="https://github.com/user-attachments/assets/f533ca67-8576-4208-ac65-b411bb22aab5" />  
Then click on admin change settings and click yes, then you can change the network configs such as add or remove network. Click on add network, make it host only, and make the subnet 192.168.50.0 for VMnet8 and keep VMnet1 the same as default.  
Next set up the Ubuntu VPN Server VM by going through the usual steps to get it set up, after getting it set up, right click it in VMWare, click on settings and you will see this:  
<img width="756" height="733" alt="image" src="https://github.com/user-attachments/assets/faaf4724-ab69-4e16-bdbc-9939a2e9db8f" />  
Make sure the memory is 8 GB, click on add, then click on network adapter to add one of the networks that was created, and click on ok  
After that, start up the Server VM and go through the intall process, after its done, go to the terminal and type ip a to see the network configurations for both network adapters.  
After that type sudo apt update && sudo apt install wireguard -y to get Wiresguard installed on the system.  
Next generate the keys by typing the command: 

