# vpn-server-with-wireguard

A comprehensive guide for deploying secure VPN infrastructure using WireGuard and OpenVPN across physical and virtualized environments. This repository documents complete setup instructions for configuring VPN servers on real networks, building isolated testing labs in VirtualBox, and replicating production-grade environments in VMware Workstation/ESXi. It includes detailed coverage of encryption workflows, key exchange mechanisms, routing table configuration, and secure remote-device onboarding, enabling reliable end-to-end testing, learning, and operational deployment.

Make sure the vmnets are set to VMnet1 as host only with local dhcp off and set to 192.168.56.0 and VMnet8 set to NAT with dhcp enabled and set to 192.168.8.0  

This will involve three VMs the Server VM, Internal LAN and the Client VM, all Ubuntu, and having to set up the respective networks for it.  
To set up the networks in VMWare, go to Edit up top, click on it and click on Virtual Network Editor it will look like this:  
<img width="598" height="559" alt="image" src="https://github.com/user-attachments/assets/f533ca67-8576-4208-ac65-b411bb22aab5" />  
Then click on admin change settings and click yes, then you can change the network configs such as add or remove network. Click on add network, make it host only, and make the subnet 192.168.50.0 for VMnet8 and keep VMnet1 the same as default.  

# VPN Server VM

Next set up the Ubuntu VPN Server VM by going through the usual steps to get it set up, after getting it set up, right click it in VMWare, click on settings and you will see this:  
<img width="756" height="733" alt="image" src="https://github.com/user-attachments/assets/faaf4724-ab69-4e16-bdbc-9939a2e9db8f" />  
Make sure the memory is 8 GB, click on add, then click on network adapter to add one of the networks that was created, and click on ok  
After that, start up the Server VM and go through the intall process, after its done, go to the terminal and type ip a to see the network configurations for both network adapters.  
After that type sudo apt update && sudo apt install wireguard -y to get Wiresguard installed on the system.  
Then type up: sudo nano /etc/netplan/00-installer-config.yaml and add the following to it:  
<img width="1198" height="669" alt="image" src="https://github.com/user-attachments/assets/9588e624-b40e-49ee-84ae-ef80c36319aa" />  
And save it and type sudo netplan apply  
Next generate the keys by typing the command: wg genkey | tee server_private.key | wg pubkey > server_public.key  
Then type cat server_private.key and cat server_public.key and write them down or save them somewhere.  
Then type sudo nano /etc/wiresguard/wg0.conf and enter this information:  
[Interface]  
Address = 10.8.0.1/24  
ListenPort = 51820  
PrivateKey = <server_private_key>  

PostUp   = iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o ens33 -j MASQUERADE  
PostDown = iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -o ens33 -j MASQUERADE  

Then enable forwading by typing "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf and then sudo sysctl -p  
Then type sudo systemctl enable wg-quick@wg0 and then start. Then check the status of it.  

# First Client VM

Now its time to set up the VPN Client VM.  
Use an Ubuntu Linux VM, 8096 MB for Memory, 80 GB for storage and for the network adapter put it on VMnet 8.  
After getting it setup, go the terminal and type sudo sudo nano /etc/netplan/00-installer-config.yaml. 
Then add this:  
<img width="810" height="569" alt="image" src="https://github.com/user-attachments/assets/04799a0a-101f-4eca-8cad-5f6d636360e7" />  
Save it and type sudo chmod 600 /etc/netplan/00-installer-config.yaml and sudo netplan apply. 
Then type the command wg genkey | tee client_private.key | wg pubkey > client_public.key to generate the public and private keys, then cat both of them and copy and paste them both in a txt file for later.  

# Second Client VM
Now its time to setup the VPN Client VM 2.  
Use an Ubuntu Linux VM, 8096 MB for memory, 80 GB for storage and for the network adapter put it on VMnet 8.  
After getting it setup, go to the terminal and type sudo nano /etc/netplan/00/installer-config.yaml.  
Then add this:  
<img width="826" height="535" alt="image" src="https://github.com/user-attachments/assets/4582509b-c0c3-478c-a102-5292df5c06b0" />  
Save it and type sudo chmod 600 /etc/netplan/00-installer-config.yaml and sudo netplan. To confirm type ip addr show ens33 and then ping -c 3 192.168.8.2.  

# Internet Connectivity
On each VM, then try to test internet connectivity by typing ip addr show ens33 which should show each one saying 192.168.8.10-30 like below:  
Server:  
<img width="575" height="48" alt="image" src="https://github.com/user-attachments/assets/8171f158-9ad7-4082-96d2-cf16314f300d" />  
*Expected result to not cause routing issues*  
Client 1:  
<img width="802" height="266" alt="image" src="https://github.com/user-attachments/assets/cfbde3e8-9537-42cd-ba4c-b5064de779f9" />  
Client 2:  
<img width="802" height="196" alt="image" src="https://github.com/user-attachments/assets/cc3c49d2-7be1-4dc7-a743-35d2f99b8fae" />  
Then do ping -c 3 192.168.8.2 on all of them with an example:  
<img width="656" height="206" alt="image" src="https://github.com/user-attachments/assets/6115d11d-e35c-4106-bac4-b77444070cfc" />  
Then do ping -c 3 8.8.8.8 and ping -c 3 google.com on them with examples:  
<img width="658" height="197" alt="image" src="https://github.com/user-attachments/assets/0b24f78b-afe2-4e78-a965-8c5d52496a05" />  
<img width="809" height="261" alt="image" src="https://github.com/user-attachments/assets/9ccb5b1b-9c6a-4def-b65f-2dc4378571c3" />  

# VM to VM connection
From the server ping the addresses for both clients.  
<img width="889" height="609" alt="image" src="https://github.com/user-attachments/assets/4f9a0b6c-9b22-49d3-891f-dbbc4c0ebc27" />  

From the first vm client:  
<img width="1093" height="659" alt="image" src="https://github.com/user-attachments/assets/205dafa7-c40c-4779-986d-726e09efed4e" />  
<img width="660" height="397" alt="image" src="https://github.com/user-attachments/assets/289f0861-a329-4049-94d7-12406555797a" />  

From the second vm client:  
<img width="856" height="638" alt="image" src="https://github.com/user-attachments/assets/f25dc4ca-cba7-47c6-b8bc-7df59d19a4b8" />  
<img width="712" height="410" alt="image" src="https://github.com/user-attachments/assets/5d96d4e6-9474-43ea-9476-09f036ddb8ba" />  

# SSH Installation
On each VM go and install ssh on them by running these commands:  
sudo apt install net-tools.  
sudo apt-get update.  
sudo apt install openssh-server -y.  
sudo systemctl status ssh, then sudo systemctl start ssh, sudo systemctl enable ssh, then sudo systemctl status ssh to check the status:  
<img width="981" height="403" alt="image" src="https://github.com/user-attachments/assets/acd950ba-1e2c-4f7f-b95a-99404a8e8e9f" />  

# Hostname














