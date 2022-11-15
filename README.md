# Open5GS-Deployment
1. <a href="https://github.com/open5gs/open5gs">Github Repository</a>
2. <a href="https://github.com/nitinrajput1997/open5gs">Reference1</a>
3. <a href="https://github.com/RohitArora7/IP-Static-and-Dynamic">Reference2</a>

<img src="https://user-images.githubusercontent.com/97805339/162958665-e6d299a8-6811-4540-baab-96a283e3b7d7.png" 
      width="500" height="300">

* Create 2 VMs for Open5gs and UERANSIM like this.

<img src="https://user-images.githubusercontent.com/97805339/162959165-9fed45ca-15c4-4105-8695-8930ff521f01.png" 
      width="500" height="300">

* Check the IP of each VM and note it down.
  ```bash
  ## to check the IP of a system
  ip a
  ```
  
  <img src="https://user-images.githubusercontent.com/97805339/162960015-85ea7511-31fb-4ceb-9254-7519ffa09d64.png" 
      width="500" height="300">

*  Open the local terminal and connect each VM to the different terminals.

```bash
## connect to the VM
ssh <VM_login_name>@<VM_ip>
```

<img src="https://user-images.githubusercontent.com/97805339/162961173-5edde2f2-7484-4673-8be9-7a9521b8df27.png" 
      width="500" height="300">

* Make the IP address of each VM, static, to make it easier for customers to find you via DNS.

```bash
## to change ip
cd /etc/netplan/
ls
sudo vim 00-installer-config.yaml
```

<img src="https://user-images.githubusercontent.com/97805339/162963861-f30481b9-8f53-4653-91e0-c772aa3a6900.png"  
     width="500" height="300">


*Vim keys to use:* <br>
*Press 'd' button to delete text.*<br>
*Press ‘i’ button for insertion.*<br>
*Press ‘ESC’ button to exit insertion.*<br>
*Press ‘:x’ to save and exit.*<br>

* A vim editor will open. Remove all context and paste the below context there. Then save and exit.

``` bash
## change ip address of 'addresses field' as preferred by you
network:
 ethernets:
  enp1s0:
    addresses:
     - 192.168.122.245/24
    gateway4: 192.168.122.1
    nameservers:
      addresses:
      - 8.8.8.8
      - 8.8.4.4
      search: []
 version: 2
 ```
 
<img src="https://user-images.githubusercontent.com/97805339/162965243-22a985a9-9e6e-4f96-848f-da7a4c11ca91.png"  
    width="500" height="300">
     
```bash
sudo netplan apply
```

* The command is running in this terminal. So, open another terminal and login same VM again. And close the running terminal.

* Do similar with the other VM.

* First we deploy Open5gs then UERANSIM because we need Open5gs configuration in UERANSIM.

* **Run the following commands in a single terminal, logged in to a VM for Open5gs deployment.**

### Install Open5GS

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs
```

* Update the amf config

```bash
## to edit amf config 
sudo vim /etc/open5gs/amf.yaml 
```

* Change the ngap address with your Open5gs IP and then save and exit.

<img src="https://user-images.githubusercontent.com/97805339/162971589-64ad84d6-4595-4e90-a7de-8caf99aef1c0.png"
width="500" height="300">

```bash
sudo systemctl restart open5gs-amfd
```

* Update the upf config

```bash
## to edit upf config 
sudo vim /etc/open5gs/upf.yaml 
```

* Change the gtpu address with your Open5gs IP address and save it.

<img src="https://user-images.githubusercontent.com/97805339/162972771-29743dfc-6033-4dfc-93d1-df342d9380dd.png"
width="500" height="300">

```bash
sudo systemctl restart open5gs-upfd
```

### NAT(Network Address Translation) Port Forwarding

* To create a connection between 5G Core and Internet, we need to enable IP forwarding and add a NAT rule to the IP Tables.

```bash
## change eth0 with your ethernet interface
## In my case it is enp1s0,

## en   --> ethernet 
## p1   --> bus number (1) 
## s0   --> slot number (0)

sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo systemctl stop ufw
sudo iptables -I FORWARD 1 -j ACCEPT
```

```bash
## to get your ethernet interface
ip a
```

<img src="https://user-images.githubusercontent.com/97805339/162974310-18ade2e5-8984-40ea-824e-390f444dc1a7.png"
width="500" height="300">

### Access Open5gs Dashboard

```bash
sudo apt update
sudo apt install curl
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt install nodejs
git clone https://github.com/open5gs/open5gs.git
```

```bash
cd open5gs/webui/
npm ci --no-optional && npm run build
npm run dev --host 0.0.0.0
```

* Open a new terminal and run this command.

```bash
# run this command if you are on remote server and want to access dashboard locally
ssh -L localhost:3000:localhost:3000 open5gs_VMlogin_name@ip
```

* Open any browser and search for this website:
http://localhost:3000

* Login credentials : <br>
username - admin <br>
password - 1423

<img src="https://user-images.githubusercontent.com/97805339/162983240-0fa15fa5-a94e-464e-81cc-4eddfb0c780a.png"
width="500" height="300">

* Add new subscriber from dashboard : <br>
IMSI: 901700000000001, <br>
Subscriber Key: 465B5CE8B199B49FAA5F0A2EE238A6BC, <br>
USIM Type: OPc,<br>
Operator Key: E8ED289DEBA952E4283B54E88E6183CA

* Or only type IMSI Number, rest will filled automatically.

<img src="https://user-images.githubusercontent.com/97805339/162982418-c60a32b5-06b0-4b7d-bf09-0cd9059e2206.png"
width="500" height="300">

* **On a new terminal, logged in to another VM for UERANSIM deployment.**

### Install UERANSIM

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install iproute2
sudo snap install cmake --classic
sudo apt install gcc
sudo apt install g++
sudo apt install libsctp-dev
```

```bash
## clone ueransim
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM
sudo apt install make
make
```

### Setup gNB

* Update the `linkIp`, `ngapIp`, `gtpIp` field with UERANSIM IP and change the `amfConfigs address` field with Open5gs IP and save it.

```bash
sudo vim config/open5gs-gnb.yaml 
```

<img src="https://user-images.githubusercontent.com/97805339/162977612-6223f7ef-fbc7-4e02-a50e-e91b2eef8084.png"
    width="500" height="300">
     
```bash
cd UERANSIM/
sudo ./build/nr-gnb -c config/open5gs-gnb.yaml
```

### Setup UE

* **Open a new terminal, logged in to same VM as used in gNB setup.**

* Update the `gnbSearchList` with the IP address of the UERANSIM.

```bash
sudo vim UERANSIM/config/open5gs-ue.yaml
```

<img src="https://user-images.githubusercontent.com/97805339/162978851-a7314c6f-b37d-4ce4-b7e6-a2730b2af08c.png"
     width="500" height="300">

```bash
cd UERANSIM/
sudo ./build/nr-ue -c config/open5gs-ue.yaml
```

## Wireshark 

* Keep the last 2 terminals running gNB and UE setup opened.

* Open another terminal and login to UERANSIM VM and run these commands to store the data packets.

```bash
## change ip field with your UERANSIM IP.
## file.pcap is the name of the file, you want to store packets. You can change the name of the file.
sudo tcpdump host <ip> -i any -w file.pcap
```

* Open a new local terminal and run this command:

```bash
scp <UERANSIM-VMlogin-name>@<ip>:<location_of_the_file> <location_to_store_the_packets_in_the_local_device>
```

* For example, in my case, it is:

```bash
scp s2@192.168.5.182:/home/UERANSIM/file.pcap /home/shubham/file.pcap
```

* After sometime, stop the terminal running “sudo tcpdump host 192.168.122.11 -i any -w gNB1.pcap” command by pressing “shift+c” button.

* The packets are stored in the pcap file.
* Open the file in wireshark.
* You can see the flow of data packets and protocols used.

<img src="https://user-images.githubusercontent.com/97805339/162981494-ea6fb369-a587-4eda-ae1e-aaeb7f2ca562.png"
    width="500" height="300">
