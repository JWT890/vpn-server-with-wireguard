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
*Note this will be changed later*

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
Type sudo hostnamectl set-hostname wg-server, wg-client1 and client2 on the respective VMs and then sign out and back in to confirm.  
Example on VM server:  
<img width="809" height="523" alt="image" src="https://github.com/user-attachments/assets/59a2ae62-98ef-4408-ad7f-be87b21df565" />  
Example on Client 1:  
<img width="837" height="579" alt="image" src="https://github.com/user-attachments/assets/ac4458da-106e-4460-8f2a-d01cea761524" />  
Example on Client 2:  
<img width="820" height="569" alt="image" src="https://github.com/user-attachments/assets/6a29b6c4-d151-404a-86cf-c4a023e60828" />  

# Host File
Type sudo nano /etc/hosts and add this at the end of each one in each VM.  
<img width="445" height="82" alt="image" src="https://github.com/user-attachments/assets/421946ba-dea9-4e85-854c-abb7dbcca53a" />  

# SSH Connectivity
Now make sure to verify connectivity by ssh into each vm by typing ssh and then the name of the vm such as ssh jon@wg-client1.  
From the server to the other VMs:  
Server to WG-Client1:  
<img width="992" height="670" alt="image" src="https://github.com/user-attachments/assets/56bc72f7-65fe-4980-92d3-8bb305e552fd" />  
Server to WG-Client2:  
<img width="953" height="690" alt="image" src="https://github.com/user-attachments/assets/2d099270-9c04-4763-8712-64baada7429b" />  

Client1 to WG-Server:  
<img width="906" height="667" alt="image" src="https://github.com/user-attachments/assets/b4466d4c-7f27-4f28-80fc-d5b719b07a95" />  
Client1 to CLient2:  
<img width="881" height="533" alt="image" src="https://github.com/user-attachments/assets/d2ca6258-63f0-42d4-9ded-5be3dd5a14dc" />  

Client2 to Server:  
<img width="827" height="543" alt="image" src="https://github.com/user-attachments/assets/bc6a7230-1e1c-441d-8508-4f1096d4c1be" />  
Client2 to Client1:  
<img width="735" height="505" alt="image" src="https://github.com/user-attachments/assets/24f54147-fe52-404c-bb50-cb803f7af3f7" />  

# WireGuard Installation  
Next its time to install Wiresguard on all the VMs, to do so run the command sudo apt install wireguard wireguard-tools -y and then wg version to check the version.  
Like so on the Client2 VM:  
<img width="712" height="52" alt="image" src="https://github.com/user-attachments/assets/b88721bd-1ed5-40ec-97b6-9b0544daf790" />  
Then for the VM Server only, port forwarding will be enabled permanently by following the picture below:  
<img width="781" height="90" alt="image" src="https://github.com/user-attachments/assets/d0f3e41c-7e15-460a-828e-b28ba0688fa4" />  
1 is the expected output since it shows it is working properly.  

# WireGuard Configuration
On the Server VM:  
First start by creating the directory by typing sudo mkdir -p /etc/wireguard, then sudo su to get into root, then type cd /etc/wireguard.  
After getting in type wg genkey | sudo tee server_private.key and copy it down somewhere for later.  
Then type sudo cat server_private.key | wg pubkey | sudo tee server_public.key and copy it down somewhere for later.  
<img width="665" height="75" alt="image" src="https://github.com/user-attachments/assets/afb1a790-b82e-4173-a2e4-cea3db9b4efb" />  

On the Client1 VM:  
Generate the private key by typing wg genkey | sudo tee client1_private.key and copy it done somewhere for later.  
Then type sudo cat client1_private.key | wg pubkey | sudo tee client1.public.key and copy it down somewhere for later.  
<img width="628" height="75" alt="image" src="https://github.com/user-attachments/assets/e3d3a1b0-cf29-4559-9c23-a1b00b7d1624" />  


On the Client2 VM:  
Generate the private key by typing wg genkey | sudo tee client2_private.key and copy it down somewhere for later.  
Then type sudo cat client2_private.key | wg pubkey | sudo tee client2.public.key and copy it down somewhere for later.  
<img width="608" height="97" alt="image" src="https://github.com/user-attachments/assets/8c4ce9fb-7979-415d-bc57-96c1548d6d6d" />  

Then go back to the Server VM and configure WireGuard.  
Type sudo nano /etc/wireguard/wg0.conf and input this info into it:  
<img width="777" height="546" alt="image" src="https://github.com/user-attachments/assets/94bf5029-83c4-4e47-8924-da43774a03f2" />  
Save it then type sudo chmod 600 /etc/wireguard/wg0.conf.  

Then go to the Client1 VM and configure WireGuard.  
Type sudo nano /etc/wireguard/wg0.conf and input this info into it:  
<img width="819" height="531" alt="image" src="https://github.com/user-attachments/assets/4d3ce026-c18d-49c1-8e42-36b96925224c" />  
Save it and then type sudo chmod 600 /etc/wireguard/wg0.conf.  

Then go to the Client2 VM and configure WireGuard.  
<img width="831" height="526" alt="image" src="https://github.com/user-attachments/assets/15afc9e5-9eb3-4cbd-a3ed-ca05ef978be5" />  
Save it and then type sudo chmod 600 /etc/wireguard/wg0.conf.  

Then go back to the Server VM and configure the Firewall.  
Do the following:  
<img width="638" height="514" alt="image" src="https://github.com/user-attachments/assets/a7fc8e4f-21cf-44e5-9f32-439c7195e263" />  

# VPN Testing & Verification  
Follow this link since there will be issues: https://claude.ai/share/5874641e-f9d8-4266-b495-874298e11a0a  
To continue: https://gemini.google.com/share/0b00f0bda2d2  
On the Server VM, go and enable Wireguard by doing the following:  
<img width="1090" height="552" alt="image" src="https://github.com/user-attachments/assets/37abb017-eaba-4fbb-9cef-635933496d69" />  
Then type ip addr show wg0 and see this result:  
<img width="1074" height="114" alt="image" src="https://github.com/user-attachments/assets/ce1e21e4-1230-4158-9e07-f11adf8297b5" />  
Then type sudo wg show and get this result:  
<img width="630" height="292" alt="image" src="https://github.com/user-attachments/assets/650c2939-6734-4d67-a205-ff888d85fae9" />  

Then go to the Client1 VM and enable Wireguard by doing the following:  
<img width="1199" height="585" alt="image" src="https://github.com/user-attachments/assets/6af29f7d-baec-49f5-99e5-dd07f11b0102" />  
Then type ip addr show wg0, then sudo wg show and see this result:  
<img width="1037" height="353" alt="image" src="https://github.com/user-attachments/assets/74a3e04e-8fb6-4a1a-8ce1-24ef7575b816" />

Then go to the Client2 VM and enable Wireguard by doing the following:  
<img width="1200" height="546" alt="image" src="https://github.com/user-attachments/assets/0ea5e14b-08c0-4b22-b8c2-879925ffd399" />  
Then type ip addr show wg0, then sudo wg show and see this result:  
<img width="1138" height="350" alt="image" src="https://github.com/user-attachments/assets/ccd26735-27a2-479c-8b1c-4fc859984da5" />  

Then go back to the Server VM to verify the connection with the clients.  Make sure to the follow the links in the section.  
<img width="723" height="399" alt="image" src="https://github.com/user-attachments/assets/13779ac5-555e-4e84-b687-f032c79f5441" />  

Then go test VPN connectivity between the VMs.  
Server to Client 1 and 2:  
<img width="708" height="446" alt="image" src="https://github.com/user-attachments/assets/6f86cdb3-20f2-4acd-99f7-af87ce868296" />  

Client 1 to Server and 2:  
<img width="829" height="670" alt="image" src="https://github.com/user-attachments/assets/99e73c5c-ef32-4c4a-b8db-070f6ccb3ff6" />  

Client 2 to Client 1 and Server:  
<img width="767" height="671" alt="image" src="https://github.com/user-attachments/assets/2b5dffaf-6111-4583-9920-b6011f911fb2" />  

Then from one of the VMs run ip route show and get this:  
<img width="794" height="94" alt="image" src="https://github.com/user-attachments/assets/d8850178-5829-4f4c-9ac2-13241af251dc" />  

Then run a traceroute from Client1 to Server like so:  
<img width="694" height="73" alt="image" src="https://github.com/user-attachments/assets/cb89db99-9e71-4a94-a1dd-811383279655" />  

# Performance Test
On both Server and Client1 run the sudo apt install iperf3 -y command onto both.  
On Client run the command iperf3 -s and on client1 run iperf3 -c 10.8.0.1 -t 10.  
Client1 output:  
<img width="868" height="444" alt="image" src="https://github.com/user-attachments/assets/ac5d93f1-2081-4ad6-9b5c-cf0e4fff8ebe" />  
Server Output:  
<img width="798" height="477" alt="image" src="https://github.com/user-attachments/assets/8530d684-5ebb-4925-87e1-25204538ac36" />  
And with Client2 output:  













































