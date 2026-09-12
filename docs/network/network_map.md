# 192.168.1.0/24 (WAN / Rete Base Untagged)

- 192.168.1.1: FritzBox router (WAN gateway e DHCP base)
- 192.168.1.10: Proxmox Server (Hermes - Interfaccia di emergenza/fisica)
- 192.168.1.217: OPNsense router (Interfaccia WAN che riceve Internet)

# Rete Homelab

## VLAN 10 - Management & Core Infrastructure (10.0.10.0/24)

- 10.0.10.1: OPNsense router (Gateway VLAN 10)
- 10.0.10.2: Switch TP-Link (IP di gestione)
- 10.0.10.3: AccessPoint MikroTik (IP di gestione)
- 10.0.10.4: Pi-hole (DNS Server primario dell'infrastruttura)

## VLAN 20 - Servers, Storage & Apps (10.0.20.0/24)

- 10.0.20.1: OPNsense router (Gateway VLAN 20)
- 10.0.20.3: Nginx Proxy Manager (Porta d'ingresso per le richieste web)
- 10.0.20.10: Immich Server (VM 110)
- 10.0.20.11: Jellyfin Server (VM 111)
- 10.0.20.12: Czkawka Service (VM 112)
- 10.0.20.50: NAS vecchio stabile (Untagged dalla porta 2 dello switch)
- 10.0.20.51: NAS nuovo temporaneo (Untagged dalla porta 4 dello switch)

## VLAN 30 - Trusted / Home Network (10.0.30.0/24)

- 10.0.30.1: OPNsense router (Gateway VLAN 30)
- 10.0.30.x (DHCP): Dispositivi connessi alla rete Wi-Fi del MikroTik.

## VLAN 40 - Guest & Untrusted (10.0.40.0/24)
Rete isolata dal resto del lab, non conoscono Proxmox o i NAS e possono connettersi solo su internet.

- 10.0.40.1: OPNsense router (Gateway VLAN 40)
- 10.0.40.x (DHCP): Utenti ospiti (amici/parenti) connessi a un SSID Wi-Fi "Guest" dedicato sul MikroTik.
- 10.0.40.x (DHCP/Statico): Dispositivi Smart Home / IoT (TV, telecamere, lampadine) che richiedono solo l'accesso al cloud.
- 10.0.40.x: Eventuali VM "Sandbox" date in concessione a utenti esterni per fare esperimenti, senza che possano toccare i tuoi dati.

*(Gli utenti esterni che usano il tuo Immich da remoto, invece, non entrano in nessuna VLAN: si fermano fuori, sulla WAN, e Nginx preleva per loro i dati dalla VLAN 20 in modo sicuro).*