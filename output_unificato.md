# Tailscale

noip richiede un ip **pubblico**, i provider di rete potrebbero fornire un indirizzo tipo 100.X.X.X (es. 100.64.x.x) che corrisponde ad un CGNAT e con questo non è possibile lavorarci, occorre utilizzare tailscale oppure richiedere al provider un ip **pubblico** (anche dinamico va bene, purché pubblico).

Occorre installare il plugin os-tailscale su opnsense.
Poi attivarlo in VPN>Tailscale>Settings checkando enabled (salvare).
Poi andare in status e cliccare il Login URL per autorizzare la macchina.

## aggiunta reti

Abbiamo aggiunto OPNsense su tailscale, ma bisogna che agisca da ponte per raggiungere il resto del lab (Split Tunneling).

VPN -> Tailscale -> Settings -> tab **Advertised Routes**
aggiungere le reti:

192.168.1.0/24,10.0.10.0/24

Dal sito di tailscale (Admin Console), cliccare sui tre pallini di fianco al dispositivo OPNsense, scegliere "Edit route settings" e approvare le reti che ora compariranno nella sezione Subnets.

## smartphone

installare l'app di tailscale
login con lo stesso account

accendere semplicemente la VPN (su Android accetta le rotte in automatico e farà passare nel tunnel solo il traffico per il Lab). Selezionare "Use exit node" (OPNsense) *solo* se si vuole far passare l'intero traffico Internet del telefono da casa per ragioni di sicurezza su Wi-Fi pubblici.

## installare ed usare sul pc

accendere e sbloccare le rotte pubblicate da OPNsense:
`sudo tailscale up --accept-routes`

spegnere temporaneamente:
`sudo tailscale down`

spegnere definitivamente (non si avvierà da solo all'accensione del PC):
`sudo systemctl stop tailscaled`
`sudo systemctl disable tailscaled`# Home

This repo contains all the docs about my home-lab journey.

</div># Overview
In this example I setup a simple network, trying to figure out what i can do before setupping my actual homelab and how to do it virtually with cisco packet tracer.

# Initial Setup
The item i chose were:

![devices](img/image1.png){ width="400" }

- Two Pcs
- 2960-24TT switch
- 1941 router
 
I chose those devices becouse they have GigabitEthernet (1 Gbps at least) port by default, instead slower FastEthernet ports (100 Mbps)

With those item we will create a simple network between two machines, which we will divde into two VLANs

- VLAN10: the computer desktop vlan
- VLAN20: the server vlan

We will implement Router on a stick network, also knwon as a one-armed router, a network where the router is connected to multiple VLAN through only one phisical interface thanks to a switch connected with trunk connection.

![Router on a stick](img/router_on_a_stick.png){ width="400" }

In a LAN network without segmentation all devices are connected and can view each others, so we have to create isolated network using VLANs (one network for the servers, one network for desktop devices and one for IoT devices for example).

The Switch is a layer 2 network device that can separate network into VLANs and the Router is a layer 3 network device that can make communicate VLAN10 with VLAN20.

# CLI Commands

## Base commands

- `enable`: change from user mode (>) to privileged mode (#)
  
- `configure terminal`: enter the global configuration (`(config)#`), now we can configure
  
- `end`: exit the configuration

- `write memory`: move the changement to the volatile RAM to the NVRAM (non volatile) (it saves the modification)

## IOS switch CLI

- `vlan x`: create a vlan identified with the number x
  
- `name y`: assign a textual tag y to the created vlan x

- `interface portName portNumber`: enter into the contest of the pyhisical port

- `switchport mode access`: configure the port to receive data from endpoint devices like PCs and being able to receive data only by one device

- `switchport access vlan x`: insert the port that we are configuring into the vlan x, so the data emitted from the device connected to this port will be labeled with number x

- `switchport mode trunk`: configure the port to receive data from multiple devices labeled with different tags that represent the vlan and the 802.1Q tag

## IOS router CLI

- `no shutdown`: the ports are shutdown di default, so we have to disable this

- `encapsulation dot1Q x`: configure the virtual interface to accept packet from dot1Q standard (802.1Q) labeleb with x number, representing the VLAN

- `ip address x`: assign ip address x to the interface that is being configured, transformed it into the default gateway to all the devices on that sub-network


# Step by step guide

## Connect devices

First, you have to connect devices with the right camble and the right port

We connect PCs with Copper Straight-Through cable to the FastEthernet0 ports of the switch:

- CLIENT_DESKTOP to FastEthernet0/1
- SERVER_LAB to FastEthernet0/2

We connect the switch to router with Copper Straight-Through cable:

- GigabitEthernet0/1 of the siwtch to the GigabitEthernet0/0 of the router.

![cable](img/physical_connection.png){ width="800" }

## Configure PCs

We have to assign the network configuration for two PCs, through the ip configuration app

Client desktop:

- IPv4: 192.168.10.10
- Subnet Mask: 255.255.255.0
- Gateway: 192.168.10.1

Server lab:

- IPv4: 192.168.20.10
- Subnet Mask: 255.255.255.0
- Gateway: 192.168.20.1

In this way we orginized the two devices in two different subnet (one on the 10 and one on the 20), and the gateway is conventionally the first device of that network (so .1).

## Configure switch

Now we have to:

- create VLANs
- insert devices into them
- connect the router in trunk mode

```bash

    enable
    configure terminal

    vlan 10
    name CLIENT_DESKTOP
    exit

    vlan 20
    name SERVER_LAB
    exit

    interface fastEthernet 0/1
    switchport mode access
    switchport access vlan 10
    exit

    interface fastEthernet 0/2
    switchport mode access
    switchport access vlan 20
    exit

    interface gigabitEthernet 0/1
    switchport mode trunk
    exit

```

## Configure router

```bash

    enable
    configure terminal

    interface gigabitEthernet 0/0
    no shutdown
    exit

    interface gigabitEthernet 0/0.10
    encapsulation dot1Q 10
    ip address 192.168.10.1 255.255.255.0
    exit

    interface gigabitEthernet 0/0.20
    encapsulation dot1Q 20
    ip address 192.168.20.1 255.255.255.0
    exit

    end
    write memory
```

## Conclusion

Now we can ping the two devices and it should work:

![ping](img/ping.png){ width="700" }

and the network are working

![workingNetwork](img/workingNetwork.png){ width="500" }

you can find the working solution on [router_on_a_stick.pkt](https://github.com/aldobettega/HomeLab/blob/main/pkt/router_on_a_stick.pkt)# Architettura router on a stick

## Topologia e Concetti di Base

L'obiettivo di questo progetto è la creazione di un HomeLab isolato (VLAN 10) gestito da un firewall virtualizzato (OPNsense).

Questa architettura prende il nome di **Router on a Stick**. A differenza di un setup tradizionale dove un router ha più cavi fisici (uno per ogni rete), qui un singolo cavo fisico trasporta il traffico di più reti diverse simultaneamente. Per evitare che i pacchetti si mescolino, viene utilizzato il protocollo **IEEE 802.1Q**, che "etichetta" (Tag) ogni pacchetto ethernet con un numero identificativo.


## Il Bug del "Double-Tagging"

Questo è stato il blocco più ostico. OPNsense era configurato per gestire internamente il Tag 10, ma i pacchetti non arrivavano.

**Il Passaggio Corretto (Via CLI su Proxmox):**

1. Allineare la configurazione dell'host in `/etc/network/interfaces` aggiungendo:
* `bridge-vlan-aware yes`
* `bridge-ports nic0`


2. Forzare l'inserimento del Trunk sulla VM scavalcando la GUI:
* `qm set 101 --net0 virtio=MAC_ADDRESS,bridge=vmbr0,trunks=10`


3. Riavviare la VM (`qm reset 101`).

## Disabilitare funzioni di Router su AccessPoint

Un MikroTik nasce come router potente. Per usarlo come semplice Access Point (ponte radio), abbiamo dovuto annientare le sue funzioni di Livello 3 e disabilitare i protocolli di sicurezza di Livello 2, affrontando due problemi letali: l'STP e il Rogue DHCP.

**Il Passaggio Corretto (Via App MikroTik):**

1. **Creazione del Bridge:** Assicurarsi che nel menu *Bridge -> Ports* coesistano sia l'interfaccia fisica `ether1` (cavo) sia le interfacce wireless (`wlan1`, `wlan2`).
2. **Uccidere lo Spanning Tree Protocol (STP):** Nel menu principale del *Bridge -> STP*, impostare la modalità su **`none`**.
3. **Sopprimere il Rogue DHCP:** Nel menu *IP -> DHCP Server*, disabilitare o cancellare qualsiasi server DHCP attivo (tipicamente sulla rete 192.168.88.x).

**Perché lo facciamo:**

* **Il blocco STP (Forwarding: no):** Lo Spanning Tree Protocol è un sistema inventato per prevenire i loop di rete (quando gli switch formano un anello chiuso, causando tempeste di pacchetti che paralizzano la rete). Il MikroTik stava tenendo la porta `ether1` in uno stato di "blocco preventivo", temendo un loop. Essendo la nostra una topologia a linea retta, disattivare l'STP ha forzato la porta in stato di inoltro immediato (Forwarding: yes).
* **Il Rogue DHCP (Race Condition):** Il tuo telefono non riusciva a navigare perché, appena si collegava al Wi-Fi, il server DHCP nativo del MikroTik (192.168.88.x) gli offriva un IP prima che potesse farlo OPNsense (10.0.10.x). È una classica *Race Condition*: vince chi risponde più in fretta (ed essendo il MikroTik l'antenna fisica, vinceva sempre). Uccidendo il server ribelle, il telefono è stato obbligato ad attendere pazientemente l'IP fornito da OPNsense attraverso il tunnel VLAN.# Configurazione Switch

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

## Inserimento dei servizi nella VLAN 20

0. Spegnere la VM
1. Modificare il VLAN tag della VM da Hardware > Network > in VLAN tag selezionare 20
2. Accendere la VM e accedere la file di configurazione di rete con `/etc/newtoek/interfaces` e modificare la primary network interface da dhcp in:
   
```bash
    iface ens18 inet static
        address 10.0.20.x/24
        gateway 10.0.20.1
        dns-nameservices 8.8.8.8 1.1.1.1
```
# Abilitare iommu su proxmox

Dalla console di proxmox
    nano /etc/default/grub

e mettere in GRUB_CMDLINE_LINUX_DEFAULT, "quiet intel_iommu=on iommu=pt"

salvare le modifiche lanciando
    update-grub

# Installare vm debian

Installare e configurare vm debian con spec:
- Cores: 4
- RAM: 4GB
- Storage: il NAS

Aggiungere ad hardware il PCI device affinchè la vm sfrutti la scheda grafica UHD graphics fornita dalla nostra cpu i-8500T.
![alt text](image.png)

# Predisporre ambiente per Jellyfin

Installare pacchetti necessari:
    apt update && apt install cifs-utils curl wget nano -y

Creare cartella per mount:
    mkdir -p /mnt/jellyfin_media

Aggiungere configurazione di connessione al vecchio NAS:
    //192.168.1.50/Volume_1 /mnt/jellyfin_media cifs guest,vers=1.0,iocharset=utf8,_netdev 0 0

Montare il NAS:
    mount -a

Verificare che il NAS sia stato montato correttamente:
    df -h

Installare docker
    curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh

Preparare cartelle per jellyfin
    mkdir -p /opt/jellyfin && cd /opt/jellyfin

Creare file di configurazione docker per sbloccare la transcodifica hardware Intel Quick Sync:
    nano docker-compose.yml

    services:
        jellyfin:
            image: jellyfin/jellyfin:latest
            container_name: jellyfin
            network_mode: 'host'
            volumes:
            - ./config:/config
            - ./cache:/cache
            - /mnt/jellyfin_media:/media
            devices:
            - /dev/dri:/dev/dri
            restart: 'unless-stopped'

Avviare il conteiner docker con:
    docker compose up -d

# Accedere a Jellyfin

Accedere alla gui web tramite:
    ip_vm:8096

# Installazione VM

Creare un container LXC con
- OS: ubuntu server / debian 13
- RAM: 2GB
- Core: 4
- Disk: 20GB

Se ssh non funziona, fare da proxmox:

```shell
sudo apt update && sudo apt install openssh-server -y
sudo apt update && sudo apt install openssh-server -y
```

# Installazione pacchetti

Installare pacchetti necessari:
```shell
sudo apt update && sudo apt upgrade -y
sudo apt install cifs-utils curl nano -y
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

# Creazione directory 

```shell
sudo mkdir -p /mnt/cartella1
sudo mkdir -p /mnt/cartella2
```

Verificare dalla gui del synology in
    pannello di controllo>servizi file>sbm>impostazioni avanzate
che sia abilitato a sbm 3.

Creare file di configurazione per credenziali:
    `nano ~/.smbcredentials`
Inserendo:
    `username=tuo_utente_synology password=tua_password_synology`
e rendendolo non leggibile agli altri utenti della VM:
    `chmod 600 ~/.smbcredentials`

Modificare fstab di configurazione:
    sudo nano /etc/fstab
Inserendo:
```shell
    //192.168.1.224/cartella1nome /mnt/cartella1 cifs credentials=/home/scanner1/.smbcredentials,uid=1000,gid=1000,iocharset=utf8,vers=3.0 0 0
    //192.168.1.224/cartella2nome /mnt/cartella2 cifs credentials=/home/scanner1/.smbcredentials,uid=1000,gid=1000,iocharset=utf8,vers=3.0 0 0
```
Ricaricare con:
    `sudo systemctl daemon-reload`
Montare:
    `sudo mount -a`
Verificare con:
```shell
    ls -l /mnt/cartella1
    ls -l /mnt/cartella2
```

# Installare Czkawka

Creare dir per czkawka con docker file
```bash
mkdir ~/czkawka
cd ~/czkawka
nano docker-compose.yml
```

```bash
services:
  czkawka:
    image: jlesage/czkawka
    container_name: czkawka
    ports:
      - "5800:5800"
    environment:
      - USER_ID=1000
      - GROUP_ID=1000
      - TZ=Europe/Rome
    volumes:
      - ./config:/config:rw
      - /mnt/cartella1:/storage/cartella1:rw
      - /mnt/cartella2:/storage/cartella2:rw
    restart: unless-stopped
```

Avviare con:
    `sudo docker compose up -d`


Accedere alla gui con:
    `192.168.1.198:5800`
# Non ricordo la password di un LXC

Entra nella shell del nodo di proxmox come root con
    `pct enter ID_CONTAINER`

Modificare la password con
    `passwd root`# Come cambiare ID alle macchine

Per cambiare in sicurezza id a VM e Container occorre clonare questi servizi con un ID nuovo.
Per farlo basa eseguire questi passaggi.

## VM

0. spegnere la VM che vogliamo clonare
1. verificare che la VM che stiamo clonando abbia su Hardware > CD/DVD Drive su "Do not use any media"
2. clonare dalla shell del nodo proxmox con `qm clone ID_VM_ATTUALE ID_VM_NUOVO --name NOME --full 1`
3. controllare che si avvi correttamente la macchina clonata
4. eliminare quella vecchia se va tutto bene

## LXC

0. spegnere il container che vogliamo clonare
1. clonare con `pct clone ID_LXC_ATTUALE ID_LXC_NUOVO --name NOME --full 1`
2. controllare che si avvi correttamente il container clonato
3. eliminare quello vecchio se va tutto bene# Creare la VM

Per un servizio come Immich occorre una VM.
Scarichiamo da https://www.debian.org/CD/netinst/ la [amd64](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-13.6.0-amd64-netinst.iso).

# Installazione debian

### 1. Lingua e Tastiera

* **Lingua (Language):** Ti consiglio vivamente di scegliere **English**. Avere un server in inglese ti salverà la vita se in futuro dovrai cercare su Google eventuali messaggi di errore.
* **Paese (Country):** Seleziona **Other -> Europe -> Italy** (serve per il fuso orario).
* **Tastiera (Keymap):** Qui seleziona **Italian** (o quella corrispondente alla tastiera fisica con cui stai digitando, per evitare di impazzire con i caratteri speciali).

### 2. Configurazione di Rete

* **Hostname:** Inserisci un nome chiaro per riconoscerlo nel tuo router OPNsense, ad esempio `immich-server`.
* **Domain name:** Se nel tuo Unbound DNS su OPNsense hai configurato un dominio locale (es. `home.arpa` o `lab.local`), inseriscilo. Se non lo hai fatto o non lo ricordi, **lascia il campo completamente vuoto** e vai avanti.

### 3. Utenti e Password

* **Root password:** Scegli una password forte per l'utente "Dio" del sistema (root).
* **Nome del nuovo utente (Full name & Username):** Crea il tuo utente principale (es. il tuo nome o `homelab`). *Evita di usare nomi banali come `admin` o `user` per questioni di sicurezza*.
* **Password dell'utente:** Inserisci la password per questo utente (la userai spesso per connetterti via SSH).

### 4. Partizionamento Dischi (Cruciale)

Dato che stiamo operando su un disco virtuale (quello da 32-50GB che hai creato sull'NVMe), la configurazione è semplicissima:

* Scegli **Guided - use entire disk** (Guidato - usa l'intero disco).
* Seleziona l'unico disco che ti viene mostrato (probabilmente si chiamerà `vda` o `sda`).
* Schema di partizionamento: Scegli **All files in one partition** (Tutti i file in una partizione - raccomandato per i nuovi utenti). Per Docker va benissimo così.
* Seleziona **Finish partitioning and write changes to disk** e rispondi **Yes** alla schermata successiva di conferma.

### 5. Configurazione del Package Manager (Apt)

* Ti chiederà se vuoi scansionare altri CD/DVD: rispondi **No**.
* **Debian archive mirror country:** Seleziona **Italy**.
* **Debian archive mirror:** Seleziona **deb.debian.org** (è il più veloce e stabile).
* **HTTP Proxy:** Lascia il campo **vuoto** e continua.
* *Partecipazione al sondaggio sull'uso dei pacchetti (popularity-contest):* Rispondi **No**.

### 6. Selezione del Software (Il passaggio più importante)

Arriverai a una schermata con una lista di componenti (Software selection). Muoviti con le frecce su/giù e usa la **Barra Spaziatrice** per mettere o togliere gli asterischi `[*]`:

* **TOGLI** l'asterisco da *Debian desktop environment*
* **TOGLI** l'asterisco da *GNOME* (o Xfce/KDE, non deve esserci nulla che riguarda l'interfaccia grafica).
* **METTI** l'asterisco su **SSH server** (Fondamentale! Ti permetterà di usare il terminale dal tuo PC principale).
* **METTI** l'asterisco su **Standard system utilities**.
* Vai su *Continue* (tasto Tab, poi Invio).

### 7. Installazione del Bootloader (GRUB)

* Ti chiederà se vuoi installare il boot loader GRUB sul disco primario: rispondi **Yes**.
* Nella schermata successiva, **non scegliere "Enter device manually"**, ma seleziona il disco fisico mostrato nella lista (es. `/dev/vda`).

A questo punto l'installazione finirà. Ti chiederà di riavviare (Continue). La VM si riavvierà e, invece di un'interfaccia grafica, ti troverai davanti a una schermata nera con la scritta `immich-server login:`.

# Primo accesso e montaggio NAS d-link

1. accedere in ssh con l'utente base
2. diventare root con su -
3. installare pacchetti necessari: apt update && apt install sudo cifs-utils nano -y
4. abilitare utente base per il futuro: /sbin/usermod -aG sudo immich-user
5. creare dir di montaggio: mkdir -p /mnt/immich_photos
6. configurazioni per vecchio nas:
   - nano /etc/fstab
   - aggiungere alla fine
        //192.168.1.50/Volume_1 /mnt/immich_photos cifs guest,vers=1.0,iocharset=utf8,_netdev 0 0
7. montare: mount -a
8. verificare se il nas sia montato: df -h

# Predisposizione immich

1. Installare docker
    curl -fsSL https://get.docker.com -o get-docker.sh sh get-docker.sh
2. creare cartella applicazione
    mkdir -p /opt/immich
    cd /opt/immich
3. Scaricare file di immich
    wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
    wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
4. Dire ad immich che lo storage è il nas
    nano .env
    mettere in UPLOAD_LOCATION /mnt/immich_photos
5. avviare il server
    docker compose up -d

# Accedere ad immich

1. accedere da browser all'ip della vm sulla porta 2283 e creare un account
2. scaricare app sul telefono e abilitare backup automatico delle cartelle

Per accedere ad immich occorre connettersi alla vm: per farlo all'esterno della rete dell'homelab o fuori casa occorre usare strumenti come tailscale.

# Nozioni

## VM o Container?

Per OPNsense verrà usata una VM completa per due motivi

- isolamento dal kernel -> un container condivide il kernel linux con l'host proxmox, ma OPNsense si basa su un OS diverso (FreeBSD), non può girare su linux
- sicurezza e rete -> una vm garantisce che una compromissione del firewall (che esegue operazioni complesse e di basso livello) non si propaghi all'host proxmox

## Paravirtualizzazione o emulazione

Con una vm di solito l'hypervisor emula hw di rete -> ma emulare è un'operazione costosa per la cpu (continua traduzione).
La paravirtualizzazione (VirtIO) consente di usare un canale diretto per parlare direttamente con la scheda di rete dell'host, performance più alte.

## distribuzione delle risorse

No cpu pinning -> non riservo core di silicio fisici ma assegno vCPU: il servizio esegue calcoli con massimo 2 core per volta.
A gestire il tutto c'è CFS (Completly Fair Scheduler), gestisce risorse assegnate al sistema operativo proxmox (tutte: 6 core fisici).
Le vm passano gran parte del tempo in idle.

Il limite non è la somma delle vCPU ma il carico effettivo simultaneo.

Diverso per la ram: non posso allocare più ram di quanta ne possiedo fisicamente, quando configuro una vm. Con i container la distribuzione della ram imposto un limite massimo e ne viene fatto un uso e distribuzione intelligente tra i container.

## UFS vs ZFS
Sono due file system

- zfs consuma molta ram (la usa come cache del disco), molto accurato per l'integrità dei dati
- ufs è tradizionale, leggero e solido -> fa a caso nostro

## Disco

il disco sul quale installeremo il sistema sarà vtbd0 (Virtual I/O Bloc Device 0) -> il kernel di freebsd riconosce di star comunicando con un hypervisor (->carica driver paravirtualizzato) saltando emulazione.

## noVNC

server headless, soluzione: noVNC, client che mostra a browser che succede.

## Creazione VLAN

Proxmox ha vlan tag vuoto -> vtnet0 riceve traffico untagged dal router e tagged per il lab.

Dobbiamo dividere vtnet0 in più interfacce virtuali:

- vtnet_vlan10 ecc...

# Configurazione: OPNsense (Router on a Stick)

## Preparazione dell'Hypervisor (Proxmox)

* **VLAN Awareness:** Abilitata l'opzione *VLAN aware* sul bridge virtuale `vmbr0` per trasformarlo in uno switch gestito capace di leggere i tag 802.1Q senza isolare l'host.
* **Creazione VM ottimizzata:**
* Disabilitato il firewall di Proxmox sulla singola interfaccia di rete virtuale (vNIC) per evitare il doppio filtraggio.
* Impostato il disco su *VirtIO Block* per massimizzare le performance dell'SSD.
* Impostata la CPU su *Host* per esporre le istruzioni crittografiche AES-NI della tua CPU Intel.
* Disabilitato il *Ballooning* della RAM per prevenire instabilità del kernel di FreeBSD.

## Installazione e Risoluzione Errori (Console)

* **Provisioning:** Aumentata temporaneamente la RAM della VM a 4 GB per superare il limite del Ramdisk durante la copia del file system (Live CD).
* **File System:** Scelto *UFS* (rispetto a ZFS) per risparmiare preziosa memoria RAM in ottica HomeLab.

**3. Network Design e Topologia Logica (Console)**

* **Sdoppiamento Scheda (VLAN 10):** Creata la sottomarca `vtnet0_vlan10` sulla singola interfaccia fisica.
* **Assegnazione Ruoli:**
* **WAN** assegnata a `vtnet0` (traffico *Untagged* proveniente dalla rete di casa).
* **LAN** assegnata a `vtnet0_vlan10` (traffico *Tagged* per la futura rete di Management isolata).


* **Configurazione IP e Servizi LAN:**
* Impostato IP statico della LAN a `10.0.10.1/24`.
* Attivato il server DHCP per il range `10.0.10.100 - 200`.


## Sicurezza e Accesso di Emergenza (Shell & WebGUI)

* **Bypass Temporaneo:** Utilizzato il comando `pfctl -d` da shell per abbattere temporaneamente il packet filtering e accedere alla WebGUI dall'esterno (lato WAN).
* **Regola di Management (Backdoor sicura):** Creata una regola esplicita in ingresso sulla WAN per consentire il traffico HTTPS verso "This Firewall" proveniente solo dalla rete locale (`WAN net`).
* **Risoluzione Conflitto NAT:** Disattivato il blocco delle reti private (RFC 1918) e Bogon sull'interfaccia WAN per permettere al firewall di operare dietro il router casalingo (Double NAT).

## Ottimizzazione e Telemetria (WebGUI)

* **Allineamento Firmware:** Aggiornato il sistema base di OPNsense all'ultima versione disponibile per risolvere i conflitti di dipendenze del gestore pacchetti.
* **Integrazione Hypervisor:** Installato, abilitato e avviato il plugin `os-qemu-guest-agent` per comunicare metriche reali (RAM e IP) a Proxmox tramite il canale seriale VirtIO.

## Abilitare ssh su OPNsense

system>settings>administration> enable ssh

accedere da terminale con
    ssh root@10.0.10.1# Strutturazione rete

## VLAN

### Creazione dei tag  VLAN

Dividiamo la rete dell'homelab in sottoreti virtuali, questo consente di isolare la reti in più parti per renderla più sicura e manutenibile.

Per gestirle accedere a OPNsense > Interfaces > Devices > VLAN

Creando le seguenti sottoreti:
- VLAN 1 (Transit/WAN): rete casalinga del fritzbox, ospita l'host di Proxmox 
- VLAN 10 (Management): contiene dispositivi di infrastruttura di rete (router, switch, access point)
- VLAN 20 (Servers & Storage): contiene i servizi offerti dall homelab (nginx, jellyfin, immich, nas)
- VLAN 30 (Trusted): rete privilegiata che ospiterà future utenze esterne fidate.
- VLAN 40 (Users / Wifi): rete per i consumatori dei servizi offerti (wifi, utenze jellyfin, immich ecc..) 

![alt text](image.png)

### Assegnazione delle interfacce

Abbiamo appena creato i tag delle VLAN, ora dobbiamo trasformale in interfacce di rete gestibili

![alt text](image-1.png)

### Configurazione degli ip

Ora su OPNsense > Interfaces compariranno le nostre VLAN nel menu. Dovremo selezionarle per configurarle, assegnando un IPv4 che rappresenterà OPNsense su quelle VLAN. In questo modo ogni VLAN saprà che quell'indirizzo rappresenta il gateway.
Per ogni vlan selezioniamo
- enable interface
- IPv4 Configuration Tipe: Static IPv4
- IPv4 Address: l'indirizzo del gateway della VLAN (10.0.x.1)

### Creazione dei pool

Creiamo dei pool dhcp: un range di indirizzi che viene assegnato in automatico a nuovi dispositivi che si connettono alla rete.
Occorre accedere ad OPNsense > Services > Kea DHCP > Kea DHCPv4 > Subnets

+ Pool per VLAN30 (Trusted):
  - subnet: 10.0.30.0/24
  - 10.0.30.50 - 10.0.30.200

+ Pool per VLAN40 (Guests):
  - subnet: 10.0.40.0/24
  - 10.0.40.50 - 10.0.40.200

Salvare e **riavviare**.

## Configurazione firewall

Dobbiamo comunicare al firewal, che di default blocca tutto il traffico in entrata verso nuove interfacce, di garantire a queste reti la connessione internet.
Queste regole sono dei **Allow All** per le nostre VLAN:
- Action = Pass: indica al firewall cosa fare con il pacchetto dati. In alternativa si può bloccare o rigettare
- Protocol = Any: permettiamo traffico in UDP, TCP, ICMP ecc...
- Source * network: questa regola si attiverà solo se i dati saranno provenienti dalla rete * (nel nostro caso le VLAN)
- Destination = Any: permettiamo a questa rete di raggiungere qualsiasi punto di internet. 

Questa impostazione permissiva è utile su un setup iniziale.

Per creare nuove regole navigare su OPNsense > Rules > +.


### Regola per TRUSTED (VLAN 30)

Impostare:

- Action: Pass
- Protocol: Any
- Source: Trusted network
- Destination: Any

### Regola per GUESTS (VLAN 40)

Impostare:

- Action: Pass
- Protocol: Any
- Source: Guests network
- Destination: Any




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
carico nel kernel linux il modulo 8021q che trasforma questo switch software in managed -> ora posso creare la vm# Rete domestica

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



# 192.168.1.0/24

- 192.168.1.1 -> FritzBox router (WAN gateway)

- 192.168.1.10 -> Proxmox Server (Hermes)
- 192.168.1.217 -> OPNsense router (LAN gateway)

# Rete Homelab

## VLAN 10 - Management

- 10.0.10.1
- 10.0.10.2

## VLAN 20 - Servers & Storage

- 10.0.20.1: Gateway
- 10.0.20.10: Immich
- 10.0.20.20: Jellyfin


## VLAN 30 - Trusted (Utenze esterne)


## VLAN 40 - Guest (Utenze interne e WiFi)
# Finding hardware

To start a homelab on a budget, you have to search for the right hardware on the right sites for the right prices. I didn't have much experience in the hardware market, and in 2026 the prices are high, so you have to be thoughtful about what you purchase.

Here are a few tips I can share after my hardware research:

- An initial setup consists of about one mini PC, one switch, and some patch cables to connect the mini PC and additional devices to the switch.
- The mini PC is the brain of the server. You have to search for at least 16GB of DDR4 RAM and a CPU with at least 6 cores and 6 threads since you have to virtualize several services. I chose an i5-8500T (the 'T' indicates that the processor has a reduced Thermal Design Power (TDP) of 35 watts, an essential spec for a machine that has to stay up 24/7).
- Buy used; give a shot to private sellers on eBay. I found an Acer Veriton N4660G for 160€, a very good price for this PC.
- Search by CPU (for ex. i5-8500T) instead of by the PC model (Acer Tiny), so you can skim a lot faster through the substantial specs of your PC.
- For the switch, I chose a TL-SG608E 8-port (30€ new on Amazon); the E means that it is managed. I found out later that some devices like access points need PoE (Power over Ethernet), so I had to buy a PoE injector because the model of this switch doesn't have this feature. Consider buying one with both managed and PoE features, or plan to buy a PoE injector (8-20€).

So I started with:

- Acer Veriton N4660G (160€ used):
    - 16GB RAM
    - 512GB SSD
    - Intel Core i5-8500T
- TP-Link Managed Switch TL-SG608E (30€ new)
- 5 patch cables (0.25m) CAT6 (10€ new)

# Testing hardware

It is recommended to test your used hardware to know what you have actually bought. The Acer had Windows 10 installed, so I used this OS to run the checkups.

- RAM: mdsched.exe, a Windows program to detect the status of the RAM.
- SSD: CrystalDiskInfo, a program you have to download and install from a browser.
- CPU: Prime95 with CPUID HWMonitor. You run Prime95 to stress all the cores and monitor the test with HWMonitor; if the temperatures go over 80°C, consider changing the thermal paste.

It is also recommended to open the case and clean the dust.

# Troubleshooting

## BIOS problems

After I opened the case, the PC could power on but couldn't boot. I'm not a HW expert, but I did some checks, and after those, the PC magically booted:

- Switch the RAM or try booting with one RAM stick at a time.
- Change the BIOS battery (maybe it was dead).
- Reset the BIOS with the power button.
- Check that if you changed the thermal paste, it didn't spread too much; in that case, clean it.
- Pray.

After those checks and prayers, the computer booted with a warning:
 `THE CHASSIS WAS OPENED`
So maybe this was one of the problems. This is a security message that blocks the boot of the BIOS. You have to disable it from the BIOS settings, so every time the system boots you don't have trouble.# Documentazione As-Built & To-Be: Architettura HomeLab

Questo documento descrive lo stato dell'infrastruttura di rete, l'inventario hardware e l'IP Plan strutturato secondo principi di segmentazione (Zero Trust). Il setup è orientato alla sicurezza e scalabilità per futuri utenti e collaboratori.

## 1. Inventario Hardware e Mappatura Fisica (Layer 1)

L'infrastruttura si basa su dispositivi efficienti e compatti, interconnessi tramite uno switch managed centrale.

* **Server Host (Hypervisor):** Mini PC Acer Veriton N4660G (i5-8500T, 16GB RAM, 512GB NVMe). Ha una sola scheda di rete (`nic0`).
* **Switch Core:** TP-Link TL-SG608E (Gestito, L2, 8 porte Gigabit).
* **Access Point:** MikroTik wsAP ac lite (alimentato via PoE tramite Iniettore Tenda POE15F).
* **Storage di Rete:** 
  * Synology NAS (Nuovo/Principale - `DOMUS-NAS`)
  * D-Link NAS (Vecchio - per backup)
* **Router ISP:** Fritzbox (Fornisce connettività WAN alla rete).

### 1.1 Mappatura e Configurazione Porte Switch TP-Link (802.1Q & PVID)
La configurazione dello switch L2 gestisce il traffico in uscita (matrice Tagged/Untagged) e in entrata (PVID) per garantire il corretto instradamento e l'isolamento hardware (VLAN Ingress Filtering).

* **Porta 1 (Proxmox Host & OPNsense):**
  * **Uscita:** Tagged su VLAN 10, 20, 30, 40 (Trunk verso le macchine virtuali). Untagged su VLAN 1 (Native VLAN, previene il lockout su Proxmox).
  * **Ingresso:** PVID 1.
* **Porta 2 (Vecchio NAS D-Link):**
  * **Uscita:** Untagged su VLAN 20 (Storage). Not Member sulle altre (Filtro hardware contro accessi esterni).
  * **Ingresso:** PVID 20.
* **Porta 3 (MikroTik AP):**
  * **Uscita:** Untagged su VLAN 10 (Accesso diretto di Management per evitare lockout). Tagged su VLAN 30, 40 (Supporto SSID multipli per gli ospiti). Not Member su 1 e 20 (Isolamento rete WAN e abbattimento rumore broadcast dei server).
  * **Ingresso:** PVID 10.
* **Porta 4 (Nuovo NAS Synology):**
  * **Uscita:** Untagged su VLAN 20 (Storage). Not Member sulle altre.
  * **Ingresso:** PVID 20.
* **Porte 5, 6, 7 (Non connesse):**
  * **Uscita:** Untagged su VLAN 1.
  * **Ingresso:** PVID 1 (Default).
* **Porta 8 (Uplink WAN / Fritzbox):**
  * **Uscita:** Untagged su VLAN 1. Not Member su 10, 20, 30, 40 (Previene il leak di traffico interno sulla WAN).
  * **Ingresso:** PVID 1.

---

## 2. Architettura di Rete e IP Plan (Layer 2 & 3)

La rete adotta un approccio "Router on a Stick" gestito da un firewall virtualizzato (OPNsense). Proxmox ha la direttiva `bridge-vlan-aware yes` attiva sul bridge `vmbr0`. 
L'IP di Proxmox è volutamente mantenuto sulla rete dell'ISP per garantire l'accesso d'emergenza in caso di crash della VM del firewall (evitando il "lock-out").

* **VLAN 1 (Transit / Fallback) - `192.168.1.0/24`**
  * `192.168.1.1`: Fritzbox (ISP)
  * `192.168.1.10`: Proxmox Host (Interfaccia di emergenza fuori dal routing interno)
* **VLAN 10 (Management) - `10.0.10.0/24`**
  * Rete dedicata *esclusivamente* all'hardware di rete.
  * `10.0.10.1`: Gateway OPNsense
  * `10.0.10.2`: Switch TP-Link
  * `10.0.10.3`: MikroTik AP
* **VLAN 20 (Servers & NAS) - `10.0.20.0/24`**
  * Core applicativo e storage. Nessun accesso diretto dall'esterno.
  * `10.0.20.1`: Gateway OPNsense
  * `10.0.20.5`: Nginx Proxy Manager (IP Statico)
  * `10.0.20.10 - 10.0.20.49`: NAS fisici e container LXC (Immich, Jellyfin)
* **VLAN 30 (Trusted / Lab Admins) - `10.0.30.0/24`**
  * Rete privilegiata per amministratori. Può accedere a tutte le altre VLAN tramite regole firewall.
  * `10.0.30.1`: Gateway OPNsense
  * `10.0.30.50 - 10.0.30.200`: Pool DHCP per PC e dispositivi personali.
* **VLAN 40 (Guests / Users) - `10.0.40.0/24`**
  * Rete per utenti Wi-Fi isolata (Zero Trust). Accesso solo a Internet e porte specifiche dei servizi web.
  * `10.0.40.1`: Gateway OPNsense
  * `10.0.40.50 - 10.0.40.200`: Pool DHCP

### 2.1 Stato Configurazione Logica (OPNsense)
* **Servizio DHCP (Kea):** Attivo e funzionante. Pool configurati per VLAN 10 (Gestione), VLAN 30 (Trusted) e VLAN 40 (Guests).
* **Firewall Rules (Zero Trust base):** Le interfacce TRUSTED, GUESTS e SERVERS sono attualmente configurate con regole provvisorie "Allow All" (Action: Pass, IPv4, Protocol: Any, Source: [Nome] net, Dest: any) per validare la connettività Layer 3 e permettere il routing verso Internet durante la migrazione. Da restringere in fase di hardening.

---

## 3. Gestione Servizi e Nodi Proxmox

I servizi sono isolati tramite container LXC e VM su Proxmox:

* **101 (opnsense-router):** VM isolata, core network e firewall.
* **103 (pi-hole):** Container LXC per filtraggio DNS.
* **105 (nginx-proxy):** Container LXC (VLAN 20, IP `10.0.20.5`) con Docker nidificato. Espone i servizi internamente ed esternamente.
* **Nodi Applicativi:** `immich-server`, `jellyfin-server` e `czkawka-service`.

---

## 4. Architettura DNS Ibrida e Service Discovery

La rete usa un dominio locale (`lab.lan`) per accedere ai servizi senza usare gli IP. La catena di risoluzione:
1. **DHCP:** OPNsense assegna Pi-hole come server DNS univoco.
2. **Filtraggio:** Pi-hole (Quad9 per le query pubbliche).
3. **Inoltro Condizionato:** Pi-hole invia le richieste per `lab.lan` a OPNsense.
4. **Risoluzione Locale:** OPNsense (Unbound DNS) mappa tutti i sottodomini (es. `immich.lab.lan`) verso l'IP di Nginx Proxy Manager (`10.0.20.5`).
5. **Proxying:** Nginx smista il traffico alle porte e agli IP corretti dei container sulla VLAN 20.
# Setup iniziale

## Creazione container

Per nginx è sufficiente un container, diamogli
- 1 core
- 1 GB RAM
- 8 GB storage
- ip statico nella sottorete dell'homelab (10.0.10.5)

## Modifiche alla rete per DNS

Dobbiamo istruire OPNsense a riconoscere i nomi dei dispositivi locali:

- OPNsense > Services > Unbound DNS > General > abilitiamo la registrazione DHCP
    ![alt text](image.png)


# Aggiunta di alias di rete

Affinchè io possa digitare sul browser immich.lab.lan e accedere al server immich senza bisogno di ricordare ip e porta o fare affidamento alla cronologia, devo configurare OPNsense e nginx proxy manager.
Abbiamo configurato il sistema per utilizzare il DNS di OPNsense anzichè quello integrato in pi-hole, quest'ultimo avrà solamente la funzione di ad-blocker.
Il compito di OPNsense sarà collegare il nome di dominio al corrispondente IP+porta.

I browser tuttavia di default quando accedono ad un IP accede alla porta 80. A questo punto entra in gioco Nginx che legge la regola impostata per quell'IP, indicando la porta.

## Aggiunta Unbound DNS Overrides

Andare su:

- OPNsense > Services > Unbound DNS > Overrider
- aggiungere host, dominio e ip del nostro nuovo indirizzo che il router di opnsense smisterà

## Aggiungere proxy

Andare su nginx proxy manager e aggiungere un proxy host, scrivendo

- dominio (x.lab.lan)
- indirizzo
- checkare le tre caselle (cache, protezione da exploit...)# Installare vm

- os: debian13
- core: 1
- ram: 512MB
- storage: 4GB 
- static ip: 10.0.10.10/24
- gateway: 10.0.10.1

# Installazione pi-hole

1. Aggiornare sistema
    apt update && apt upgrade -y

2. Installare pi-hole
    curl -sSL https://install.pi-hole.net | bash

Procedere con l'installazione

# Configurare

![alt text](image-1.png)

Selezionare Quad9(filtered, DNSSEC) perchè include un filtro anti-malware e anti-phishing.

Aggiornare gravity: mantiene una lista dei siti segnalati come pericolosi, le ultime definizioni di tracker e banner pubblicitari.

Aggiungere su OPNsense il nuovo DNS di pi-hole, in questo modo impongo ai dispositivi connessi alla rete di utilizzare Pi-hole come DNS primario.
![alt text](image-2.png)

## Conditional fowarding

![alt text](image.png)

aggiungerte il conditional fowarding per smistare le richieste dei dispositivi connessi alla rete.
Se un dispositivo fa una richiesta normale viene smistato verso Quad9 ed il suo filtro anti-malware.
Se fa una richiesta di tipo x.lab.lan chiedendo una richiesta interna -> dirotta su OPNsense che ha la mappa di rete.

## Aggiungere proxy

Andare su nginx proxy manager e aggiungere un proxy host, scrivendo

- dominio (x.lab.lan)
- indirizzo
- checkare le tre caselle (cache, protezione da exploit...)# creare maccchina muletto

Ho un nas pieno che devo svuotare, voglio sfruttare la memoria del minipc, quindi creo un container su proxmox.

creare un container con debian-13 con 250 gb di memoria e altre impostazioni di default.
Importante: su general togliere la spunta unpriviledge -> serve che sia priviledge per far andare cifs

installiamo libreria per parlare con vecchio linguaggio nas e tmux permette di lanciare programmi in bg
    apt update && apt install rsync -y
    apt install openssh-server -y
    apt install cifs-utils tmux -y

sblocco porta con: 
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && systemctl restart ssh

# Avviare trasferimento dati

avvio tmux sul mio pc e lo sgancio per lasciare lavorare solo container su proxmox e nas
installare tmux
lanciarlo con
    tmux
fare
    sudo rsync -avhP -e "ssh -o ServerAliveInterval=60" /mnt/vecchionas/ root@192.168.1.95:/root/backup_nas/

sganciarsi con ctrl+b e poi d

Ho avuto molti problemi di blocchi nonostante avessi usato ServerAliveInterval=60, è un'operazione che richiede molto tempo se si spostano molti gb con dispositivi lenti (15mb/s), per questo occorre verificare lo stato ogni tanto e risvegliare il processo rilanciando rsync o ancora meglio non detatchare e mantenere la finestra del terminale aperta per darci uno sguardo ogni tanto.

# Formattazione nas

Un disco era rotto, quindi occorre implementare impostazione single disk. Sulle impostazioni del mio d-link sono andato su tools>raid

# Inserimento nel lab

Su opn sense occorre creare un alias per associarlo ad un ip
1. firewall>aliases>add
    - compilare con (Name: NAS_Storage, Type(s): Host(s), Content: 10.0.10.15)
    - salvare e applicare
2. riservare un indirizzo tramite MAC address
    - services>kea DHCP>kea DHCPv4
    - aggiungere una reservation con (subnet: 10.0.10.0/24, ip address: 10.0.10.15, mac address del nas)
    - salvare ed applicare
  
Dallo switch tp-link impostare porte e vlan (io collegherò il nas sulla porta 3):
1. VLAN>802.1Q VLAN
    - inserire vlan id=10 (la vlan dei dispositivi management dell'homelab) e impostare la porta dove si collegherà il nas su untagged
    - salvare ed applicare
2. VLAN>802.1Q PVID Setting
    - inserire vlan id=10 e applicarlo alla porta dove si collegherà il nas

Problemi: se il nas era già stato inserito in rete gli era già stato dato un ip e magari le impostazioni dell'ip erano statico anziche DHCP, occorre rimetterlo a DHCP affinchè gli venga assegnato l'ip corretto. Per riaccedere all'interfaccia occorre inserie il nas in una porta tagged che da sulla rete e non sulla sottorete del lab.

Se i problemi persistono, assegnarli l'ip statico desiderato.

# Configurare impostazioni di rete

1. creare un utenza per collegare D-link a proxmox
    - users/group>inserire (username, pw)
    - salvare
2. Accedere dalla gui del D-link a advanced>network access
    - category=user, user=utenza creata, permission=R/W
    - salvare

# Workaround NAS

## Integrare un NAS Legacy (D-Link) su Proxmox moderno

**Il Problema iniziale:**
I NAS vecchi usano protocolli obsoleti (SMB 1.0) e sistemi di crittografia password (NTLMv1) che i kernel Linux moderni di Proxmox hanno rimosso per sicurezza. Inoltre, ignorano il traffico instradato da VLAN diverse.

**La Soluzione:**
Spostare il NAS sulla rete principale, eliminare l'autenticazione tramite password (accesso Guest) e montarlo su Proxmox come cartella locale invece che come storage di rete.

### 1. Configurazione della Rete (Switch e NAS)

1. **Assegna un IP della rete principale al NAS:** Dal pannello del NAS, imposta un IP statico sulla stessa classe di Proxmox (es. `192.168.1.50`), con subnet `255.255.255.0` e gateway corretto (`192.168.1.1`).
2. **Aggiorna lo Switch (se gestito):** Sposta la porta fisica in cui è collegato il NAS sulla VLAN principale (VLAN 1):
* Togli la porta dalla vecchia VLAN.
* Impostala come **Untagged** sulla VLAN 1.
* Imposta il **PVID** della porta a **1**.

### 2. Permessi del NAS (Bypass NTLMv1)

Per evitare l'errore `Permission denied` di Proxmox che non sa più calcolare le vecchie password:

1. Entra nel pannello web del NAS (`192.168.1.50`).
2. Nelle impostazioni della cartella (es. `Volume_1`), imposta i permessi su **Guest Access** (oppure User: All) con diritti di **Lettura/Scrittura**. Nessuna password richiesta.

### 3. Mount Permanente su Proxmox (Shell di Linux)

Apri la Shell di Proxmox e digita:

1. Crea la cartella di destinazione:
`mkdir -p /mnt/NAS_Backup`
2. Apri il file degli hard disk di sistema:
`nano /etc/fstab`
3. Aggiungi questa riga alla fine del file per forzare il vecchio protocollo `vers=1.0` all'avvio:
`//192.168.1.50/Volume_1 /mnt/NAS_Backup cifs guest,vers=1.0,iocharset=utf8 0 0`
4. Salva (Ctrl+O, Invio) ed esci (Ctrl+X).
5. Monta il disco immediatamente per testarlo:
`mount -a`

### 4. Aggiungere lo Storage alla WebGUI di Proxmox

Ora che Linux vede il NAS come un disco interno, diciamo a Proxmox di usarlo:

1. Vai su **Datacenter** -> **Storage**.
2. Clicca su **Add** -> **Directory** *(Non usare SMB/CIFS!)*.
3. Compila così:
* **ID:** `NAS_Backup`
* **Directory:** `/mnt/NAS_Backup`
* **Content:** Scegli **VZDump backup file** (per i backup delle VM) e/o **ISO image**.


4. Clicca su **Add**.

# Backup NAS

## Effettuare un backup manuale

Dalla gui di proxmox selezionare vm, andare in backup>backup now e selezionare NAS_backup.

## Automatizzare backup

1. Aggiungere job automatico
   - andare in: Datacenter>Backup>Add
   - Storage: NAS_Backup
   - Schedule: scrivere 02:00 per farlo andare tutti i giorni alle 2 di notte
   - Selection mode: All per backuppare tutte vm e container
   - Compression: ZSTD, abbiamo una cpu abbastanza potente per farlo
   - Mode: Snapshot, backup a caldo che non disabilita i servizi mentre salva

2. Aggiungere regola di retention globale
   - andare in: Datacenter>storage>edit NAS_Backup>Backup retention
   - è buona pratica selezionare:
     - Keep last: 3
     - Keep daily: 2
     - Keep weekly: 1
     in questo modo non riempiamo il disco di backup ma teniamo solo i più recenti.

     
### 1. Specifiche Hardware

**NAS Principale: Synology DS216play**

* **Processore e RAM:** STM Monaco STiH412 (Dual-Core 1.5 GHz, 32-bit) con 1 GB di RAM DDR3.
* **Dischi Interni:** 2x Western Digital Red da 2 TB (WD20EFRX, 5400 RPM).
* **Configurazione Volume:** SHR/RAID 1 (Mirroring). Genera 1.8 TB di spazio totale (1 TB attualmente occupato) e garantisce continuità operativa in caso di guasto hardware di un disco.
* **Sistema Operativo:** DSM 7.0.

**Disco USB Esterno: WD Elements Desktop (WDBWLG)**

* **Specifiche:** Collegamento USB 3.0, hard disk meccanico interno a 5400 RPM (velocità di trasferimento ~100-130 MB/s).
* **Capienza:** Da verificare tramite il menu *Dispositivi Esterni* sul NAS.

**NAS Secondario: D-Link DNS-323**

* **Specifiche:** Modello legacy con 200 GB di archiviazione totale e vecchi protocolli di rete (SMBv1).

# Azioni

## Creazione cartella condivisa

Creare una cartella condivisa sul synology:

- mettere permessi di lettura scrittura a user con cui si fa accesso e senza crittografia

Predisporre container LXC di proxmox

- dalla gui di proxmox dalle impostazioni del container andare in options>features e attivare SMB/CIFS
- riavviare container
- accedere in ssh al container
- creare cartella con  mkdir -p /mnt/synology
- montare con
    mount -t cifs //ip_nas/cartella_condivisa_nas /mnt/synology -o username=,vers=2.0,cache=none,echo_interval=60### Server Host

![Acer Tiny](img/acer.webp){ width="200" }

* **Model:** Acer Veriton N4660G (Mini PC Form Factor)
* **Hardware Specifications:**
* **CPU:** Intel Core i5-8500T (6 Cores, 6 Threads, 2.10 GHz base clock).
* **RAM:** 16 GB DDR4 @ 2667MHz (Configured in Symmetric Dual-Channel).
* **Storage:** 512 GB Toshiba M.2 NVMe PCIe SSD.
* **Bought for:** €160 (Refurbished).

### Managed Network Switch

![Switch](img/switch.jpg){ width="200" }

* **Model:** TP-Link TL-SG608E (8-Port Gigabit Easy Smart Managed Switch)
* **Hardware Specifications:** 8x Gigabit (10/100/1000 Mbps) RJ45 ports, Layer 2 (L2) network features, durable metal chassis.
* **Bought for:** €27.99

### Power over Ethernet (PoE) Injector

![PoE](img/poe.jpg){ width="200" }

* **Model:** Tenda POE15F
* **Hardware Specifications:** 48V PoE Adapter, Fast Ethernet (10/100 Mbps), IEEE 802.3af/u compliant, 15.4W maximum power output.
* **Bought for:** 10.74€

### Wi-Fi Access Point

![mikrotik](img/mikrotik.jpg){ width="200" }

* **Model:** MikroTik wsAP ac lite
* **Hardware Specifications:** Dual-Band (2.4 GHz and 5 GHz) wireless capabilities, Fast Ethernet (10/100 Mbps) ports, PoE-In support, powered by the RouterOS operating system.
* **Estimated Market Value:** ~€60 (New).

### Network Cabling (Patch Cables)

* **Model:** Cat6 Network Cables (0.25m, Black, 5-Piece Multipack)
* **Hardware Specifications:** Category 6, 250MHz bandwidth, Halogen-Free, compatible with Gigabit (1000 Mbit/s) and 10-Gigabit standards.
* **Bought for**: 9.95€### 💼 I Progetti "Curriculum-Killer" (Networking & Cybersecurity)

* **Segmentazione di Rete Avanzata (VLANs & Firewalling):**
    * Usando pfSense o OPNsense, crea regole (ACL) che isolano i dispositivi insicuri (telecamere, smart TV) in una VLAN separata (IoT). Dimostra che il traffico non può passare verso la tua rete principale, ma che tu puoi raggiungere loro.
* **Reverse Proxy & Identity Provider (SSO):**
    * Esponi un servizio su Internet in modo sicuro. Invece di aprire porte a caso sul router, usa **Nginx Proxy Manager** o **Traefik**. Aggiungici un sistema di autenticazione come **Authelia** o **Authentik**: prima di accedere a una tua app, l'utente viene bloccato da una pagina di login con Autenticazione a Due Fattori (2FA).
* **Wazuh (SIEM) o Intrusion Detection System (IDS):**
    * Installa Wazuh (un sistema di monitoraggio per la cybersecurity) o attiva Suricata sul tuo firewall. Fai un attacco simulato e fai uno screenshot dell'allarme generato.
* **Infrastructure as Code (IaC) con Ansible:**
    * Invece di installare i programmi a mano sui tuoi server Linux, scrivi uno "script" Ansible (un file YAML) che si collega via SSH e configura il server in totale autonomia.

---

### 🎮 I Progetti "Life-Improver" (Belli, Utili e Divertenti)

Questi sono i servizi che, una volta installati, cambieranno in meglio la tua vita digitale quotidiana e ti daranno tantissima soddisfazione.

* **Pi-hole o AdGuard Home:** 
    * instradare tutto il traffico DNS della tua rete qui dentro farà sparire magicamente la pubblicità dai siti web e dalle app
* **Nextcloud:**
    * Il tuo Google Drive / iCloud personale. Sincronizza automaticamente le foto dal tuo smartphone, ospita i tuoi documenti e non ha abbonamenti mensili. Hai il controllo totale dei tuoi dati (Privacy).
* **Jellyfin o Plex:**
    * Il tuo Netflix personale. Scarichi i tuoi film o serie TV sul server e li guardi in streaming comodamente dalla smart TV del salotto, con locandine scaricate in automatico, sottotitoli e suddivisione per stagioni.Ecco l'elenco dei progetti estratti, deduplicati e classificati per una rapida consultazione:

## 🛠️ Infrastruttura & Reti

* **Nginx Proxy Manager** - *[Reverse Proxy]* Trasforma IP e porte in indirizzi locali facili da ricordare (es. `jellyfin.casa`).
* **Pi-hole / AdGuard Home** - *[DNS Sinkhole]* Blocca pubblicità, tracker e minacce informatiche a livello di rete per tutti i dispositivi connessi.
* **Uptime Kuma** - *[Monitoraggio]* Fornisce una dashboard di stato e invia notifiche automatiche se un servizio o server si spegne.



## 🔒 Sicurezza & Privacy

* **Vaultwarden** - *[Password Manager]* Cassaforte digitale self-hosted per autocompilazione e condivisione familiare sicura delle credenziali.
* **Kasm Workspaces** - *[Sandboxing]* Genera computer o browser virtuali isolati e "usa e getta" per navigare e aprire file sospetti in sicurezza.
* **SearXNG** - *[Motore di Ricerca]* Aggrega in modo anonimo i risultati di motori come Google e Bing senza farti tracciare.



## 📁 Produttività & Archiviazione

* **Nextcloud** - *[Cloud/Sync]* Alternativa a Google Workspace per sincronizzare file, contatti e calendari tra PC e smartphone.
* **Syncthing** - *[Sync P2P]* Sincronizzazione decentralizzata, istantanea e diretta (senza server intermedi) di file pesanti tra dispositivi.
* **Paperless-ngx** - *[Gestione Documentale]* Scansiona, indicizza (tramite OCR) e smista automaticamente documenti e bollette.
* **Stirling-PDF** - *[Tool Documenti]* Applicazione web per manipolare, unire e firmare PDF localmente in totale privacy.



## 🏡 Vita Quotidiana & Finanze

* **Home Assistant** - *[Domotica]* Hub locale e universale per integrare e automatizzare i dispositivi smart home di brand differenti.
* **Actual Budget** - *[Finanza]* Software per tracciare transazioni bancarie e pianificare le spese familiari con il metodo a base zero.
* **Mealie** - *[Cucina]* Estrae ricette dai siti web, pianifica i pasti settimanali e genera automaticamente la lista della spesa.
* **ChangeDetection.io** - *[Web Tracking]* Invia notifiche quando rileva cambiamenti su specifiche pagine web (es. cali di prezzo o nuovi bandi).



## 📚 Intrattenimento & Lettura

* **Audiobookshelf** - *[Audio]* Server per audiolibri e podcast che ricorda la posizione di ascolto e supporta Android Auto/CarPlay.
* **Kavita / Komga** - *[E-Reading]* Libreria server-side per organizzare e leggere ebook, fumetti e manga da qualsiasi dispositivo.
* **Wallabag** - *[Read-it-later]* Salva articoli dal web, eliminando pop-up e pubblicità, per archiviarli e leggerli in un secondo momento.