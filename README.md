# Pi-hole setup (Two VM cluster)
We will create two VM's for highly available pi-hole cluster. One ***pihole88*** and another ***pihole99***.

<br/><br/><br/><br/>

## Create VM  
---

Create two VirtualBox machine with below details on two different physical server. One for ***pihole88*** and another for ***pihole99***.

- Name: ***pihole***  
- Type: ***Linux (64-bit)***  
- Processor: ***1***  
- Memory: ***1 GB***  
- HDD: ***8 GB***  
- Network: ***Bridged Adapter***  

<br/><br/><br/><br/>

## Setup Ubuntu Server
---

Install Ubuntu server on both VM's.

### Set hostname  

Below for ***pihole99*** host
```bash
### Update hostname
hostnamectl hostname pihole99
```

Below for ***pihole88*** host
```bash
### Update hostname
hostnamectl hostname pihole88
```


Restart both the system 
```bash
### Restart the system 
sudo shutdown -r now
```

Verify the mentioned files on both host for correct hostname
```bash
### verify below two files for correct hostname
cat /etc/hostname
cat /etc/hosts
```


### Set Static IP

Verify network adapter name using below command.
```bash
ip addr
```

After above verification update file **00-installer-config.yaml** present in **/etc/netplan**
> - Replace "enp0s3" with adapter name you find in above verification task.  
> - Put IP: **192.168.1.88** for ***pihole88*** and IP: **192.168.1.99** for ***pihole99***.  

```bash
vi /etc/netplan/00-installer-config.yaml
```
```bash
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      addresses:
        - 192.168.1.99/24
      nameservers:
        search: [piholeserver]
        addresses: [192.168.1.1]
      routes:
        - to: default
          via: 192.168.1.1
  version: 2
```

```bash
# Check if configuration is ok
netplan try
# apply the configuration
netplan apply
```

<br/><br/><br/><br/>

## Install unbound
---
```bash
sudo apt install unbound
```
### Set custom setting
Update below file and place the content from [Unbound Installation](#references) page and follow instruction from document.
```bash
vi /etc/unbound/unbound.conf.d/pi-hole.conf
```
> Make sure Unbound is using port 5335 as mentioned in documentation else Unbound will collide with pi-hole setup.


<br/><br/><br/><br/>

## Install Pihole
---
Follow [pi-hole](#references) documentation page for complete details.

```bash
curl -sSL https://install.pi-hole.net | bash
```

<br/><br/><br/><br/>

## Install Gravity Sync
---
Follow - [gravity-sync](#references) wiki page for complete details.

<br/><br/><br/><br/>

## Update DNS Setting  
---
Login to router using "192.168.1.1" and credentials
---
- LAN setting
> Network -> LAN Settings -> Advanced -> DHCP Server  
>> Primary DNS: 192.168.1.99  
>> Secondary DNS: 192.168.1.88  

- WAN setting
> Network -> EWAN -> Advanced -> Advanced -> (Use the Following DNS Addresses:) Enable  
>> Primary DNS: 192.168.1.99  
>> Secondary DNS: 192.168.1.88  

<br/><br/><br/><br/>

## Update Adlists 
Copy green ticked list from [List](https://v.firebog.net/hosts/lists.php?type=tick) like and add into Adlist. post that run below command.
```bash
# update gravity list
pihole -g
```  
Refer the site <https://firebog.net/>. This site is mentioned in "Community Projects" on pi-hole documentation page.


<br/><br/><br/><br/>

## Scripts
---
- Block script, create this script to block sites
```bash
cd /usr/local/bin
sudo vi blockpihole.sh
```
```bash
#!/bin/bash

pihole --wild discordapp.com discord.com discord.gg discordapp.net roblox.com rbxcdn.com gamejolt.com steam.com steampowered.com minecraft.de minecraft.com hypixel.net newgrounds.com --comment "Games" 

pihole --wild googlevideo.com youtu.be youtube-nocookie.com youtube.com youtube.googleapis.com youtubei.googleapis.com ytimg.com ytimg.l.google.com --comment "Youtube" 

/usr/local/bin/pihole restartdns
```
- Unblock script, create this script to unblock sites
```bash
cd /usr/local/bin
sudo vi unblockpihole.sh
```
```bash
#!/bin/bash

/usr/local/bin/pihole --wild --nuke
/usr/local/bin/pihole restartdns
```

Set executable permissions
```bash
cd /usr/local/bin
sudo chmod 755 blockpihole.sh
sudo chmod 755 unblockpihole.sh
```

To block use command:
```bash
blockpihole.sh
```
To unblock use command:
```bash
unblockpihole.sh
```

<br/><br/><br/><br/>

## References:
- [Ubuntu Server](https://ubuntu.com/download/server)  
- [pi-hole](https://docs.pi-hole.net/)  
- [gravity-sync](https://github.com/vmstan/gravity-sync)  
- - https://www.youtube.com/watch?v=IFVYe3riDRA
- [Netplan](https://netplan.io/)  
- [Unbound Installation](https://docs.pi-hole.net/guides/dns/unbound/)

