# Overview

In this page I trak my to-do ideas, my future projects and the things that I'm working on.

## What a homelab can do

- Backup storage (foto, documents)
- Various servers (Minecraft, hosting a web-site)
- Hosting an AI model

## Starting point
I'm planning to buy with a budget of 300€ these three components to simulate a functional network:

- Mini PC
- Switch
- Firewall

### Phase 1 - Set up
The first step is to create the environment in wich I can break things without worrying about anything.

- Install Proxmox VE
- setup Firewall (pfSense or OPNsense), connect it with the modem, configure a DHCP server, NAT rules ...
- deploy a local DNS with Ad-Blocking:
     - create a container on proxmox
     - install Pi-hole or AdGuard Home

### Phase 2 - Networking
Create VLAN: configure the switch and firewall to subnetting

### Phase 3 - Work in progress ...