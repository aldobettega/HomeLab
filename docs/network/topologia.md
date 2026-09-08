# Rete domestica

La rete di casa sta su
    192.168.1.0/24
con router del provider fritzbox su:
    192.168.1.1

Il proxmox host si trova su
    192.168.1.10
questo permette di accedervi da tutti gli access point della casa

OPNsense, il router del server, gestisce due interfacce:
- WAN: interfaccia esterna del firewall virtuale (vtnet0) -> riceve un ip dal fritzbox e lo usa per uscire su internet
- LAN: rete interna

# Rete homelab

Ci sono tre vlan:

- VLAN 10: 10.0.10.0/24 -> management homelab (infrastruttura di base) 
- VLAN 20: 10.0.20.0/24 -> server e vm
- VLAN 30: 10.0.30.0/24 -> client wifi

## VLAN 10

La rete 10.0.10.0/24 è strutturata come segue:

- 10.0.10.1: Gateway OPNsense
- 10.0.10.2 - 10.0.10.9: apparati di rete fisici
    - switch TP-link
    - access point MikroTik
- 10.0.10.10 - 10.0.10.49: server fisici e NAS
- 10.0.10.50 - 10.0.10.250: pool dinamico per eventuali client temporanei

### OPNsense

con gateway router
    10.0.10.1
rappresentato da OPNsense sull'interfaccia LAN (vtnet0_vlan10). Compiti:
- gateway lab
- distribuisce ip tramite kea dhcp
- risolve i nomi di dominio tramite unbound DNS

è possibile sia interagire con la gui che andara da proxmox sul terminale. Comandi utili

- ifconfig: mostra la rete di opnsense
- netstat -rn: mostra dove opnsense manda il traffico

## Mikrotik

Gli sono state tolte le funzionalità di router (no dhcp, no assengazione ip) e messo in bridge L2,
agisce solo da ripetitore wifi.

È stato inglobato nella rete dell'homelab con l'indirizzo 10.0.10.3

Ora se un dispositivo si connette a questa rete wifi (mikrotik-camere), al dispositivo viene assegnato un ip
    10.0.10.50 - 10.0.10.250
preso dal pool dinamico assegnato da OPNsense.



