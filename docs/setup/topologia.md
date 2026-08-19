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

L'indirizzamento di questa sottorete è:
    10.0.10.0/24


## OPNsense

con gateway router
    10.0.10.1
rappresentato da OPNsense sull'interfaccia LAN (vtnet0_vlan10). Compiti:
- gateway lab
- distribuisce ip tramite kea dhcp
- risolve i nomi di dominio tramite unbound DNS

## Mikrotik

Gli sono state tolte le funzionalità di router (no dhcp, no assengazione ip) e messo in bridge L2,
agisce solo da ripetitore wifi.
Ora se un dispositivo si connette a questa rete wifi (mikrotik-camere), al dispositivo viene assegnato un ip
    10.0.10.x