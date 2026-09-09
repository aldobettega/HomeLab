# Configurazione Switch

## Summary

1: proxmox
2: Nas d-link vecchio
3: Access Point MikroTik
4: Nas nuovo
5: -
6: -
7: -
8: Accesso alla rete

## Descrizione

- Porta 1: deve far passare tutto il traffico etichettato (Tagged) delle nuove VLAN, facendolo arrivare a OPNsense
- Porte 2 e 4: Untagged, connesse a devices di storage (NAS)
- Porta 3: ibrido 
  - mantiene la VLAN 10 Untagged per la gestione
  - VLAN 30 e 40 Tagged per estensioni future (SSID multipli)

## Configurazione delle porte

![alt text](image.png)

## Fase in uscita: tabella 802.1Q

Questa tabella indica come lo switch debba comportarsi quando un pacchetto ha un'etichetta VLAN e deve farlo uscire da una porta fisica verso un dispositivo.

### Nozioni varie

1. Impostare una porta come Not Member di una VLAN significa che:
    - In uscita: mi assicuro che lo switch non invii mai un pacchetto con VLAN X la cui porta è Not Member di questa VLAN. Questo significa isolare dal traffico esterno un dispositivo.
    - In entrata: se qualcuno si collega alla porta dei NAS usando un PC configurato per inviare pacchetti con la VLAN 10 (per comunicare sulla sezione critica di management), ma lo switch lo rigetta perchè da quella porta è Not Member della VLAN 10.
2. Una porta deve essere Untagged su una sola VLAN per volta per due motivi:
    - ne devo tenere solo una untagged così so che i pacchetti non taggati provengono da quella VLAN sicuramente. In caso contrario non saprei da quale VLAN vengono.
    - siccome una porta ha un solo PVID allora non potrei gestire due VLAN untagged perchè dovrei usarne solo una delle due, dunque tutti i pacchetti in uscita andrebbero solamente in una VLAN.
3. Impostare una porta come Untagged di una VLAN significa che:
    - se il dispositivo è un Endpoint (PC o NAS), questo non capisce di rete, gli interessa ricevere il pacchetto non taggato.
    - se il dispositivo è un Access Point: solitamente è Untagged dalla VLAN di management (come è in questo caso sulla VLAN 10) perchè sta ricevendo un pacchetto dal punto di gestione ed è l'Access Point stesso il ricevitore finale, non lo deve inoltrare. Inoltre se su questa VLAN fosse tagged il MikroTik andrebbe configurato per saper "spacchettare" pacchetti taggati

### Porta 1
- Tagged su 10,20,30,40: Proxmox e OPNsense sono dispositivi di rete intelligenti -> sul singolo cavo passano 4 reti diverse per macchine virtuali diverse
- Untagged su 1: Proxmox ha IP sulla rete WAN casalinga (192.168.1.10), quindi se invio pacchetti senza etichetta dalla rete casalinga, lui risponde senza pacchetti. Questo garantisce di non rimanere chiuso fuori dall'homelab in caso di problemi con OPNsense.

Nota: se la VLAN 1 fosse taggata, lo switch prende i pacchetti dalla rete WAN e ci incolla il VID=1.
Questo pacchetto verrà rifiutato da proxmox per le configurazioni presenti in `/etc/network/interfaces` (indicano come IP=192.168.1.10) poichè non ha nessuna interfaccia virtuale configurata per rispondervi.
Ora si è stati "tagliati fuori" e non si ha più accesso all'interfaccia web sulla porta 8006.

### Porta 2 e 4
- Untagged su 20: i nas sono dispositivi stupidi a livello di rete, non producono tag VLAN in entrata. In questo modo lo switch toglie il tag che il dispositivo non riuscirebbe a tradurre.
- Not member sulle altre: isoliamo i due nas sulla VLAN 20, in questo modo se un dispositivo di un altra VLAN prova a comunicare con questi NAS il pacchetto viene bloccato. L'unico modo per accedervi è passare per il gateway di OPNsense e verificare le regole del firewall.

### Porta 3
- Not member su 1 e 20: l'access point non c'entra nè con i server che con la rete WAN. In particolare è Not Member sulla VLAN 20 perchè un utente connesso al wifi dell'access point non deve poter vedere i NAS del server bypassando eventuali regole del firewall. Un motivo secondario è di performance: i messaggi broadcast viaggiano all'interno di una VLAN, se includo questa VLAN nella porta 3 i dispositivi collegati al wifi dovrebbero ascoltare i messaggi broadcast dei NAS senza alcun motivo, intasando la banda e consumando la batteria di questi.
- Untagged su 10: il destinatario ultimo è lui (arriva dalla rete Manageriale) e non sa spacchettare i tag, dovremo configurare il mikrotik.
- Tagged su 30 e 40: lo mettiamo tagged per possibili estensioni future, creando più reti wifi: quando l'access point riceve i pacchetti tagged dallo switch, usa il tag per sapere su quale rete wifi trasmetterlo.

### Porta 8
- Untagged su 1: il router FritzBox non sa spacchettare i tag, deve ricevere pacchetti untagged e invia pacchetti non taggati.
- Not member su 10,20,30,40: il router della WAN non gestisce la sottorete e non deve conoscere di conseguenza le VLAN. Inoltre i pacchetti interni non devono mai finire su internet per errore, ma devono passare per forza per il firewall di OPNsense.

## Fase in ingresso: PVID

Dobbiamo configurare che cosa deve succedere quando una porta riceve un pacchetto in ingresso da un dispositivo. Serve quando un pacchetto esc

|PORT|PVID|
|----|----|
|Port 1|1|
|Port 2|20|
|Port 3|10|
|Port 4|20|
|Port 5|1|
|Port 6|1|
|Port 7|1|
|Port 8|1|


# Aggiornamento infrastruttura

Ora che abbiamo configurato correttamente le VLAN occorre aggiornare Proxmox affinchè tutto funzioni.

1. Preparazione infrastruttura: è necessario assicurarsi che Hypervisor e Firewall comunichino correttamente
    - Disabilitare Hardware Offloading su OPNsense -> evita che i driver virtuali VirtIO rimuovano le etichette 802.1Q dai pacchetti in transito
    - Configurazione Trunk Proxmox: modificare il file `/etc/pve/qemu-server/101.conf` aggiungendo i tag consentiti alla scheda di rete: `net0: ...,bridge=vmbr0,trunks=10;20;30;40`. In questo modo il bridge VLAN-aware di Proxmox è istruito a far passare i pacchetti taggati verso OPNsense.
    - Eseguire shutdown + restart della VM di OPNsense
2. Migrazione servizi:
    - Modificare l'interfaccia di rete del servizio posizionandolo sulla VLAN desiderata e aggiornare il Gateway sulla stessa VLAN.
    - Eseguire shutdown + restart del servizio
3. Verifica:
    - ping a 10.0.20.1 -> verifica L2 verso gateway di OPNsense
    - ping a 8.8.8.8 -> verifica che esca su internet (funziona firewall, routing NAT, L3)

