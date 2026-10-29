# Packet Tracer Labs

I’m John Acosta, a QA professional developing my networking, security, and test automation skills.

This repository documents my hands-on Cisco Packet Tracer practice while studying networking fundamentals for CompTIA Network+. This repo is where those labs live. Each one has the actual Packet Tracer file, screenshots of it working, and my notes on what I learned. I'm adding labs as I go.

## Why a QA person cares about this

Most of the bugs I care about now, like broken access control, leaky APIs and bad session handling, travel over a network. If I can't picture how a request gets from a browser to a server (which device forwards it, what the router does, why ARP happens first), I can't really test it or explain what went wrong. These labs are me making sure I understand that layer, not just memorizing it for an exam.

## What's in here

| Lab | What I set up | What it proved |
|-----|---------------|----------------|
| [Lab 1: First LAN](lab01-first-lan/) | Two PCs on one switch, same subnet | Devices on the same network talk directly through a switch, no router needed |
| [Lab 2: Two LANs + a router](lab02-two-lans-router/) | Two separate networks joined by a Cisco router | Traffic between networks has to go through a router, and each PC needs a default gateway to get there |
| [Lab 3: Switch security baseline](lab03-switch-baseline/) | Basic console and privileged-access protection, with a management IP | Configured basic access protection, assigned a management IP, and verified IP reachability; saved the configuration |
| [Lab 4: Small office LAN](lab04-office-lan/) | Four department PCs and a file server on one switch, with labeled ports | Verified PC-IT connectivity to the other four hosts and switch; inspected learned MAC addresses |
| [Lab 5: Subnet investigation](lab05-subnet-investigation/) | Four PCs on one switch, split across two /26 subnets | Being plugged into the same switch doesn't mean two hosts can talk. The subnet decides |

## How to open the files

The `.pkt` files open in [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer), which is free with a Cisco Networking Academy account. I used version 9.0.1. Open a file and you can click any device, check its settings, and run the same pings I did.


## Scope and evidence

The linked lab folders contain the detailed write-ups, project files, and screenshots. Results describe the documented practice sessions; they are not a claim of production deployment or a new independent rerun.

[LinkedIn](https://www.linkedin.com/in/superhumanservices)
