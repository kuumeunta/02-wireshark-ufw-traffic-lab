# 02-wireshark-ufw-traffic-lab
Basic work with Wireshark.

## Goal
Capturing packets with Wireshark when connecting to opened and closed ports.

## Environment
- Host: Windows 10, VirtualBox 7.2.22
- VM1: Ubuntu 26.04, name: UbuntuMain, IP 192.168.56.101
- VM2: Ubuntu 26.04, name: UbuntuSub, IP 192.168.56.102
- Network: Host-Only

## Wireshark installation and setup
Installing Wireshark and making sure it works successfully

![installedws](screenshots/wsinstalled.png)
```
sudo apt update && apt upgrade
sudo apt install wireshark
wireshark
```
---
Adding user to wireshark group, allowing WS capturing traffic without starting up root session.

![grantingperms](screenshots/grantperms.png)
```
sudo usermod -aG wireshark $USER
```
After that re-entering current user session to apply group policies

---
Looking up for an interface that connects to inner traffic

![ipa](screenshots/ipa.png)
```
ip a
```
Interface named `enp0s8` has an ip 192.168.56.101, so i'll capture it's traffic

![aplliedws](screenshots/wsinterface.png)


## Ping capturing and analysis
## Filtering 
## Locating packets 

## Problems and conclusions
Problems:

Conclusions:
