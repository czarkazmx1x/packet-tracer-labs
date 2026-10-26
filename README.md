# Packet Tracer Labs

I'm a QA automation engineer (5 years, mostly Playwright) moving toward security testing. Security sits on top of networking, so I'm going back to the basics: studying for CompTIA Network+ and working through hands-on labs in Cisco Packet Tracer.

This repo is where those labs live. Each one has the actual Packet Tracer file, screenshots of it working, and my notes on what I learned. I'm adding labs as I go.

## Why a QA person cares about this

Most of the bugs I care about now, like broken access control, leaky APIs and bad session handling, travel over a network. If I can't picture how a request gets from a browser to a server (which device forwards it, what the router does, why ARP happens first), I can't really test it or explain what went wrong. These labs are me making sure I understand that layer, not just memorizing it for an exam.

## What's in here

| Lab | What I set up | What it proved |
|-----|---------------|----------------|
| [Lab 1: First LAN](lab01-first-lan/) | Two PCs on one switch, same subnet | Devices on the same network talk directly through a switch, no router needed |
| [Lab 2: Two LANs + a router](lab02-two-lans-router/) | Two separate networks joined by a Cisco router | Traffic between networks has to go through a router, and each PC needs a default gateway to get there |
| [Lab 3: Switch security baseline](lab03-switch-baseline/) | A new switch locked down before use, with a management IP | A switch can be secured and managed remotely, and config only survives a reboot if you save it |
| [Lab 4: Small office LAN](lab04-office-lan/) | Four department PCs and a file server on one switch, with labeled ports | Every host can reach every other host, and the switch learns who is on which port by itself |
| [Lab 5: Subnet investigation](lab05-subnet-investigation/) | Four PCs on one switch, split across two /26 subnets | Being plugged into the same switch doesn't mean two hosts can talk. The subnet decides |

## How to open the files

The `.pkt` files open in [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer), which is free with a Cisco Networking Academy account. I used version 9.0.1. Open a file and you can click any device, check its settings, and run the same pings I did.

---

## Lab 1: First LAN

Two PCs, one switch, one network. The simplest setup there is, but it shows how a switch works.

![Lab 1 topology](lab01-first-lan/screenshots/lab01_topology.jpg)

| Device | IP address | Subnet mask | Gateway |
|--------|-----------|-------------|---------|
| PC1 | 192.168.10.10 | 255.255.255.0 | none |
| PC2 | 192.168.10.20 | 255.255.255.0 | none |

I pinged PC2 from PC1 and got 4 out of 4 replies.

![Lab 1 ping](lab01-first-lan/screenshots/lab01_ping.jpg)

**What I took away:**
- A switch moves traffic using MAC addresses, not IPs.
- Before the first ping goes out, PC1 asks the network "who has 192.168.10.20?" (that's ARP). It needs PC2's MAC address before it can send anything.
- No gateway needed, because both PCs are on the same network.

---

## Lab 2: Two LANs connected by a router

Two classrooms, each on its own network (192.168.10.x and 192.168.20.x), joined by a router in the middle.

![Lab 2 topology](lab02-two-lans-router/screenshots/lab02_topology.jpg)

| Device | Interface | IP address | Gateway |
|--------|-----------|-----------|---------|
| Router | G0/0 (Classroom A side) | 192.168.10.1 | n/a |
| Router | G0/1 (Classroom B side) | 192.168.20.1 | n/a |
| PC1 | Fa0 | 192.168.10.10 | 192.168.10.1 |
| PC2 | Fa0 | 192.168.20.10 | 192.168.20.1 |

The router was set up from its command line:

```
enable
configure terminal
hostname R1
interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface g0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
end
show ip interface brief
```

Both router ports came up:

![show ip interface brief](lab02-two-lans-router/screenshots/lab02_show_ip_int_brief.jpg)

Then I pinged across the router from PC1 to PC2:

![Lab 2 ping](lab02-two-lans-router/screenshots/lab02_ping.jpg)

**What I took away:**
- Router ports start turned off. The links stayed red until `no shutdown`. Switch ports don't work that way.
- The default gateway is how a PC reaches anything outside its own network. PC1 can't reach 192.168.20.10 directly, so it hands the packet to the router.
- The first ping dropped one packet while ARP did its thing, then the second run was 4 for 4.
- The replies said TTL=127 instead of 128 like in Lab 1. The router knocked one off, which proves the packet actually went through it.

---

## Lab 3: Switch security baseline

A brand-new switch out of the box has no passwords, no name and no way to manage it over the network. This lab puts a basic security baseline on it before it goes into use. That's the network version of "secure by default", which is the part I care about as a tester.

![Lab 3 topology](lab03-switch-baseline/screenshots/lab03_topology.jpg)

| Device | Interface | IP address | Subnet mask |
|--------|-----------|-----------|-------------|
| SW1 | VLAN 1 (management) | 192.168.10.2 | 255.255.255.0 |
| PC1 | Fa0 | 192.168.10.10 | 255.255.255.0 |

The switch config:

```
enable
configure terminal
hostname SW1
no ip domain-lookup
enable secret class123
line console 0
 password cisco
 login
 exit
banner motd #Authorised Access Only!#
interface vlan 1
 ip address 192.168.10.2 255.255.255.0
 no shutdown
 exit
service password-encryption
end
copy running-config startup-config
```

(These are the lab's practice passwords. Never use passwords like these on a real device.)

![Baseline config](lab03-switch-baseline/screenshots/lab03_baseline_config.jpg)

VLAN 1 came up with its management address:

![show ip interface brief](lab03-switch-baseline/screenshots/lab03_show_ip_int_brief.jpg)

Then I pinged the switch itself from PC1, 4 for 4 on the second run:

![Lab 3 ping](lab03-switch-baseline/screenshots/lab03_ping.jpg)

**What each piece does:**
- `enable secret` puts a password on privileged mode and stores it as a hash. The older `enable password` stores it in plain text, which is why `secret` is preferred.
- `line console 0` + `password` + `login` means anyone plugging a cable into the console port has to log in.
- The banner is the legal "authorised access only" warning shown at login.
- `service password-encryption` scrambles any other passwords in the config file, so they aren't readable over someone's shoulder. It's weak encryption, but better than plain text.
- `no ip domain-lookup` stops the switch from freezing for ages trying to look up a typo as a hostname.
- `interface vlan 1` with an IP is an **SVI** (switched virtual interface). A switch doesn't need an IP to switch traffic, but you need one to reach it remotely for management.

**Running-config vs startup-config:** the running config is what the switch is doing right now, in memory. The startup config is what it loads when it boots. Changes only go into the running config until you run `copy running-config startup-config`. Reboot without saving and all of this is gone.

**Ping detail:** TTL=255 here, not 128 like the PCs. Cisco gear starts its TTL at 255, so the TTL also hints at what kind of device answered.

---

## Lab 4: Small office LAN

A small office: four users (Sales, Finance, HR, IT) and a shared file server, all on one switch and one network. The point of this one is doing the boring stuff properly: real device names, port descriptions, and proving every host can reach every other one.

![Lab 4 topology](lab04-office-lan/screenshots/lab04_topology.jpg)

| Device | Switch port | IP address | Subnet mask | Gateway |
|--------|-------------|-----------|-------------|---------|
| SW-OFFICE | VLAN 1 (management) | 192.168.40.2 | 255.255.255.0 | 192.168.40.1 |
| PC-SALES | Fa0/1 | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 |
| PC-FINANCE | Fa0/2 | 192.168.40.11 | 255.255.255.0 | 192.168.40.1 |
| PC-HR | Fa0/3 | 192.168.40.12 | 255.255.255.0 | 192.168.40.1 |
| PC-IT | Fa0/4 | 192.168.40.13 | 255.255.255.0 | 192.168.40.1 |
| SRV-FILES | Fa0/5 | 192.168.40.100 | 255.255.255.0 | 192.168.40.1 |

(There's no router in this lab yet. The .1 gateway is reserved for when the office gets one.)

The switch config:

```
enable
configure terminal
hostname SW-OFFICE
no ip domain-lookup
interface vlan 1
 ip address 192.168.40.2 255.255.255.0
 no shutdown
 exit
ip default-gateway 192.168.40.1
interface fa0/1
 description PC-SALES
interface fa0/2
 description PC-FINANCE
interface fa0/3
 description PC-HR
interface fa0/4
 description PC-IT
interface fa0/5
 description SRV-FILES
end
copy running-config startup-config
```

From PC-IT I pinged everything else on the network:

| From PC-IT to | Result |
|---------------|--------|
| PC-SALES (.10) | 4/4 |
| PC-FINANCE (.11) | 4/4 |
| PC-HR (.12) | 4/4 |
| SRV-FILES (.100) | 4/4 |
| SW-OFFICE (.2) | 3/4 (first one lost to ARP) |

![Lab 4 pings, part 1](lab04-office-lan/screenshots/lab04_ping_1.jpg)
![Lab 4 pings, part 2](lab04-office-lan/screenshots/lab04_ping_2.jpg)

Then `show mac address-table` on the switch:

![MAC address table](lab04-office-lan/screenshots/lab04_mac_table.jpg)

**What I took away:**
- The switch learned all five MAC addresses on its own, one per port, just from the traffic. Every time a frame comes in, the switch writes down "this MAC lives on this port".
- When the switch doesn't know a destination MAC yet, it floods the frame out every port except the one it came in on. Once the reply comes back, it knows where that host is.
- Names and port descriptions sound like busywork until you're troubleshooting at 2am. `description PC-HR` on Fa0/3 tells the next person exactly what's plugged in there, without tracing a cable.

---

## Lab 5: Subnet investigation

This one is a trap on purpose. A junior tech gave four PCs addresses with a /26 mask (255.255.255.192), all plugged into the same switch. The job: work out on paper which PCs should be able to talk, test it, then fix it.

![Lab 5 topology](lab05-subnet-investigation/screenshots/lab05_topology.jpg)

**Working it out first.** A /26 splits the last number into blocks of 64: .0–.63, .64–.127, .128–.191, .192–.255.

| Host | IP address | Network | Broadcast | Usable hosts |
|------|-----------|---------|-----------|--------------|
| PC1 | 192.168.50.10 | 192.168.50.0 | 192.168.50.63 | .1 – .62 |
| PC2 | 192.168.50.20 | 192.168.50.0 | 192.168.50.63 | .1 – .62 |
| PC3 | 192.168.50.70 | 192.168.50.64 | 192.168.50.127 | .65 – .126 |
| PC4 | 192.168.50.126 | 192.168.50.64 | 192.168.50.127 | .65 – .126 |

So PC1 and PC2 are in one subnet, PC3 and PC4 are in another, and there's no router. Prediction: only PC1↔PC2 and PC3↔PC4 work.

**Testing every pair:**

| Pair | Predicted | Actual |
|------|-----------|--------|
| PC1 ↔ PC2 | works | 4/4 |
| PC1 ↔ PC3 | fails | 0/4 |
| PC1 ↔ PC4 | fails | 0/4 |
| PC2 ↔ PC3 | fails | 0/4 |
| PC2 ↔ PC4 | fails | 0/4 |
| PC3 ↔ PC4 | works | 4/4 |

Every result matched the prediction.

![Pings from PC1](lab05-subnet-investigation/screenshots/lab05_ping_from_pc1.jpg)
![Pings from PC2](lab05-subnet-investigation/screenshots/lab05_ping_from_pc2.jpg)
![Ping from PC3](lab05-subnet-investigation/screenshots/lab05_ping_from_pc3.jpg)

**The fix.** The lab says to move PC4 into PC1 and PC2's subnet without changing the ranges. I gave it **192.168.50.30**: it's inside .1–.62, it isn't the network (.0) or broadcast (.63) address, and nobody else is using it. After the change PC4 reached both PC1 and PC2, 4/4 each:

![PC4 after the fix](lab05-subnet-investigation/screenshots/lab05_fixed_pc4.jpg)

(Side effect: PC3 is now alone in the .64 subnet, which is fine for this lab. The saved file has the fixed address.)

**What I took away:**
- A /26 has a block size of 64 and 62 usable hosts (64 minus the network and broadcast addresses).
- Hosts on the same switch can still fail to talk. Before sending, a PC checks if the destination is in its own subnet. If it isn't, it tries to use its default gateway, and here there isn't one, so the packet never leaves.
- This is a good QA lesson too: "it's all plugged into the same box" is a physical assumption, and the bug was in the config. Predicting the result before running the test is what made the trap obvious instead of confusing.

---

Questions or feedback welcome. Find me on [LinkedIn](https://www.linkedin.com/in/superhumanservices).
