# 02-wireshark-ufw-traffic-lab

Capturing and comparing TCP and ICMP traffic between two VMs, with a port closed and open in ufw.

## Goal

Capturing packets with Wireshark when connecting to opened and closed TCP ports in ufw.

## Environment

- Host: Windows 10, VirtualBox 7.2.22
- VM1: Ubuntu 26.04, Hostname: UbuntuMain, username: mainvm, IP 192.168.56.101
- VM2: Ubuntu 26.04, Hostname: UbuntuSub, username: subvm, IP 192.168.56.102
- Network: Host-Only

## Wireshark installation and setup

Installing Wireshark and checking that it starts

![installedws](screenshots/wsinstalled.png)

```
sudo apt update && sudo apt upgrade
sudo apt install wireshark
wireshark
```

---

Adding user to wireshark group, allowing Wireshark capturing traffic without runnin it as root.

![grantingperms](screenshots/grantperms.png)

```
sudo usermod -aG wireshark $USER
```

Than logging out and back in to apply the new group membership.

---

Looking for the interface connected to Host-Only network.

![ipa](screenshots/ipa.png)

```
ip a
```

Interface named `enp0s8` has the IP 192.168.56.101, so I'll capture its traffic

![appliedws](screenshots/wsinterface.png)

## Ping capturing and analysis

Verifying that ufw on subvm still works, checking up that IP address is unchanged, then pinging subvm from mainvm.

![ping](screenshots/pings.png)

```
ping -c 4 192.168.56.102
```

Then typing in `icmp` filter in Wireshark and looking up for ping being registered

![wsping](screenshots/wsping.png)

Seeing 8 lines filtered by ICMP protocol, 4 echo requests (mainvm>subvm), 4 echo replies (subvm>mainvm).

## Connecting to a closed TCP port (locked by ufw)

Now starting test server on port 8000 on subvm and trying to curl it from mainvm

![enablingserv](screenshots/servstartup.png)

```
python3 -m http.server 8000
```

Then starting a new capture in Wireshark

![newcapture](screenshots/newcapture.png)

And trying to curl (+ applying `tcp.port` filter in WS)

![wscurl](screenshots/curling.png)

```
curl --max-time 5 192.168.56.102:8000
```

As a result, captured 4 SYN packets (retransmissions of the same connection attempt) and no SYN-ACK in reply at all

## Connecting to an open TCP port (allowed in ufw)

Allowing 8000/tcp in ufw on subvm and starting the test server again.

![tcpallowed](screenshots/allowedtcp.png)

```
sudo ufw allow 8000/tcp
python3 -m http.server 8000
```

Then re-enabling Wireshark capturing and trying to curl

![re-curl](screenshots/curlagain.png)

```
curl --max-time 5 192.168.56.102:8000
```

curl returned the HTML directory listing from the test server. Checking up Wireshark capture

![wsrecapt](screenshots/synack.png)

Wireshark shows the full three-way handshake: [SYN], [SYN, ACK], [ACK]. The connection is established.

## Problems and conclusions

Problems:
- Wireshark showed no network interfaces, only "remote capture" entries.
  - Fix: added the user to the `wireshark` group and logged in again.
- A stray character in the capture filter field made it invalid (yellow field).
  - Fix: cleared the field; display filters are entered in the top bar after the capture starts.
- ping showed no replies at first because subvm was not running.
  - Fix: started subvm and checked its IP and ufw status.

Conclusions:
- With a closed port, the client only sends repeated SYN packets and never receives SYN-ACK. From the client side this looks like silence (curl timeout).
- With an open port, the full three-way handshake appears: SYN, SYN-ACK, ACK, followed by the HTTP exchange.

