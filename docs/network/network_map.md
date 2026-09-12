# 192.168.1.0/24 (WAN / Rete Base Untagged)

- 192.168.1.1: FritzBox router (WAN gateway e DHCP base)
- 192.168.1.10: Proxmox Server (Hermes - Interfaccia di emergenza/fisica su `vmbr0`)
- 192.168.1.217: OPNsense router (Interfaccia virtuale WAN che riceve Internet dal Fritzbox)

# Rete Homelab

## VLAN 10 - Management & Core Infrastructure (10.0.10.0/24)

- 10.0.10.1: OPNsense router (Gateway VLAN 10)
- 10.0.10.2: Switch TP-Link (IP di gestione)
- 10.0.10.3: AccessPoint MikroTik (IP di gestione)
- 10.0.10.4: Pi-hole (DNS Server primario dell'infrastruttura - Container LXC 101)

## VLAN 20 - Servers, Storage & Apps (10.0.20.0/24)

- 10.0.20.1: OPNsense router (Gateway VLAN 20)
- 10.0.20.3: Nginx Proxy Manager (Reverse Proxy e porta d'ingresso per i servizi web)
- 10.0.20.10: Immich Server (VM 110)
- 10.0.20.11: Jellyfin Server (VM 111)
- 10.0.20.12: Czkawka Service (VM 112)
- 10.0.20.50: NAS D-Link (Storage Principale - Untagged dalla Porta 2 dello switch, IP Statico)
- 10.0.20.51: NAS Synology (Nuovo/Temporaneo - Untagged dalla Porta 4 dello switch)
- 10.0.20.100: Proxmox Server (Interfaccia virtuale `vmbr0.20` dedicata al traffico di mount/backup Layer 2 verso i NAS)

## VLAN 30 - Trusted / Home Network (10.0.30.0/24)

- 10.0.30.1: OPNsense router (Gateway VLAN 30)
- 10.0.30.x (DHCP): Dispositivi personali affidabili connessi alla rete Wi-Fi del MikroTik.

## VLAN 40 - Guest & Untrusted (10.0.40.0/24)
Rete isolata dal resto del lab in ottica Zero Trust. I client non possono raggiungere Proxmox, i NAS o le altre VLAN; il traffico verso le reti private (RFC1918) è bloccato dal firewall. Possono connettersi solo a Internet.

- 10.0.40.1: OPNsense router (Gateway VLAN 40)
- 10.0.40.x (DHCP): Utenti ospiti (amici/parenti) connessi a un SSID Wi-Fi "Guest" dedicato sul MikroTik.
- 10.0.40.x (DHCP/Statico): Dispositivi Smart Home / IoT (TV, telecamere, lampadine) che richiedono solo l'accesso al cloud.
- 10.0.40.x: Eventuali VM "Sandbox" date in concessione a utenti esterni per fare esperimenti, isolate dai dati sensibili.

---
**Nota sull'Accesso Esterno (Zero Trust):**
Gli utenti esterni che utilizzano servizi come Immich da remoto non si interfacciano mai con la porta WAN esposta su Internet. L'accesso avviene tramite il tunnel VPN Tailscale configurato su OPNsense (Subnet Router). Il traffico entra criptato, viene instradato verso Nginx Proxy Manager (`10.0.20.3`), il quale preleva e serve i dati in modo sicuro interrogando i vari nodi sulla VLAN 20.