Minipc deve agire da router e da server -> il singolo cavo che lo collega allo switch deve fare:

- WAN: traffico rete della casa
- LAN: traffico isolato dell'homelab
Si utilizza VLAN tagging per distinguere (802.1Q) -> si configura una porta dello switch collegata a proxmox come **trunk**

Due interfacce di rete:

- virtual WAN: no tag
- virtual LAN: tag con la VLAN dell'homelab -> gateway per tutti i server e acesspoint

# Separazione management e server

- VLAN 1: rete di casa WAN -> 192.168.1.0/24

- VLAN 10: management lab -> 10.0.10.0/24

- VLAN 20: server -> 10.0.20.0/24

- VLAN 30: client wifi -> 10.0.30.0/24

La creazione di vlan permette di separare logicamente la rete ed istruire il router a permettere classi dispositivi di vlan ad accedere/non accedere ad altre vlan.

# Standard IEEE 802.1Q

Un normale pacchetto dati è formato da MAC sorgente, MAC destinatario e dati, quando uso le VLAN devo aggiungere 4 byte per il tag VLAN (un numero da 1 a 4094)

# Access port e Trunk port

Per far funzionare le VLAN, le porte di uno switch possono comportarsi:

- Access port: si collegano dispositivi "stupidi" -> lo switch prima di dare il dato toglie il tag vlan (rende pacchetto untagged) e quando il dispositivo risponde gli riapplica il tag
- Trunk port: porta per collegare due dispositivi di infrastruttura (switch con proxmox) -> non toglie vlan tag, in questo modo più vlan possono viaggiare sullo stesso cavo fisico.

# Native vlan

Dopo aver attivato vlan su proxmox che non era su una vlan, ha continuato a funzionare per la native vlan.

Una porta trunk che si aspetta pacchetti con il tag vlan, accetta anche pacchetti untagged ai quali assegna in automatico la native vlan che di default è 1.

La native vlan non viene mai usata per far transitare dati utili o di management per prevenire un attacco chiamato vlan hopping. Nel nostro lab per non complicare l'ingresso del traffico wan del router casalingo, accettiamo che la rete di casa funzioni sul native vlan.

# Linux bridge

`vmbr0` significa virtual machine bridge 0 -> un bridge linux è uno switch lvl 2 creato via software.
In proxmox se vado sul modulo principale (hermes) -> system -> network -> vmbr0 -> vlan aware
carico nel kernel linux il modulo 8021q che trasforma questo switch software in managed -> ora posso creare la vm