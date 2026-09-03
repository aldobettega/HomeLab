
# Documentazione As-Built: Architettura HomeLab

## 1. Inventario Hardware (Layer 1)

La spina dorsale fisica del laboratorio è composta da dispositivi compatti ed efficienti dal punto di vista energetico:

* **Server Host (L'Hypervisor):** Un Mini PC Acer Veriton N4660G. Sotto il cofano ospita un processore Intel Core i5-8500T (6 Core, 6 Thread), 16 GB di RAM DDR4 in Dual-Channel e un disco SSD NVMe PCIe da 512 GB.


* **Switch di Rete (Il vigile urbano):** TP-Link TL-SG608E. È uno switch gestito (Managed) di Livello 2 dotato di 8 porte Gigabit in metallo, fondamentale per la gestione delle VLAN.


* **Alimentazione (PoE):** Iniettore Tenda POE15F (48V, 15.4W). Serve a iniettare corrente sul cavo Ethernet per accendere l'Access Point.


* **Access Point Wi-Fi (Il ponte radio):** MikroTik wsAP ac lite. Dispositivo Dual-Band basato su RouterOS, alimentato interamente via PoE.


* **Cablaggio:** Cavi Patch Cat6 (Halogen-Free, 250MHz) per garantire velocità Gigabit e isolamento.



---

## 2. Topologia di Rete e Router on a Stick

L'intera infrastruttura si basa sul concetto di **Router on a Stick**. Poiché il Mini PC ha una sola porta di rete fisica, un singolo cavo trasporta simultaneamente il traffico di più reti isolate grazie allo standard IEEE 802.1Q (VLAN Tagging).

Il design logico delle reti è così strutturato:

* **VLAN 1 (Rete Domestica / Native VLAN):** La sottorete è `192.168.1.0/24`. Questa è la rete "Untagged" gestita dal FritzBox dell'ISP (`192.168.1.1`). Il traffico su questa VLAN costituisce la WAN per il nostro laboratorio e permette l'accesso alla gestione di Proxmox (`192.168.1.10`).


* **VLAN 10 (Management HomeLab):** La sottorete è `10.0.10.0/24`. Questa rete è isolata ("Tagged") e gestita esclusivamente dal firewall virtuale OPNsense (`10.0.10.1`).


* *Sviluppi futuri pianificati:* VLAN 20 (`10.0.20.0/24`) dedicata ai Server e VLAN 30 (`10.0.30.0/24`) per i Client Wi-Fi.



---

## 3. Configurazione Hypervisor (Proxmox VE)

Per permettere alla rete virtuale di comprendere i tag 802.1Q, il bridge Linux predefinito di Proxmox (`vmbr0`) è stato configurato con l'opzione **VLAN aware** abilitata. Questo carica il modulo 8021q nel kernel, trasformando il software bridge in un vero e proprio switch gestito di Livello 2.

### La Macchina Virtuale: OPNsense

OPNsense (basato su FreeBSD) è il cuore logico della rete. È stato installato come Macchina Virtuale completa per garantire l'isolamento dal kernel Linux di Proxmox e massimizzare la sicurezza.

* **Storage e CPU:** Utilizza il file system UFS (più parsimonioso in termini di RAM rispetto a ZFS) e un disco virtuale *VirtIO Block* (vtbd0) per saltare l'emulazione hardware. La CPU non ha pinning rigoroso, ma sfrutta il Completely Fair Scheduler (CFS) per l'allocazione dinamica delle risorse.


* **Interfacce di Rete Paravirtualizzate:**
* **WAN (`vtnet0`):** Interfaccia principale che riceve il traffico untagged dal FritzBox.


* **LAN (`vtnet0_vlan10`):** Interfaccia virtuale secondaria che applica e rimuove il tag 10 per il traffico del laboratorio. Per far funzionare questa divisione, nella configurazione di Proxmox, il parametro *Trunks* della scheda di rete della VM è stato forzato a 10 (`trunks=10`), evitando che Proxmox rimuova il tag prima di consegnarlo a OPNsense.




* **Servizi Attivi su OPNsense:**
* **Kea DHCP:** Distribuisce indirizzi IP sulla sottorete `10.0.10.0/24`. Nelle sue configurazioni sono state forzate esplicitamente le opzioni del router (`10.0.10.1`) e dei server DNS (`8.8.8.8`) per consentire la navigazione ai client.


* **Unbound DNS:** Attivo per la risoluzione dei nomi di dominio locali.


* **Regole Firewall:** Sulla WAN è stato disattivato il blocco delle reti private (RFC 1918) per consentire il funzionamento dietro il FritzBox (Double NAT). È stata aggiunta una regola *Pass* per consentire l'accesso alla WebGUI (HTTPS) dalla rete di casa.


* **Telemetria:** Installato il plugin `os-qemu-guest-agent` per far comunicare i dati reali (come il consumo RAM) a Proxmox.





---

## 4. Livello 2: Configurazione Switch e Access Point

### Lo Switch TP-Link

Lo switch divide fisicamente i flussi di traffico per garantire che ogni dispositivo riceva solo i pacchetti che sa interpretare:

* **Porta verso Proxmox (Trunk Port):** Impostata come Tagged per la VLAN 10. Questa porta permette il transito di più reti simultaneamente senza rimuovere le etichette 802.1Q.


* **Porta verso MikroTik (Access Port):** Impostata come Untagged per la VLAN 10. Fondamentale l'impostazione del **PVID a 10**: questo parametro istruisce lo switch ad "etichettare" con il numero 10 tutto il traffico pulito in ingresso proveniente dai dispositivi Wi-Fi.



### Il cablaggio PoE

Per evitare divisioni logiche errate, il traffico e l'alimentazione sono stati unificati. Il cavo dati esce dallo switch, entra nella porta "Data IN" dell'iniettore Tenda e ne esce combinato con la corrente per entrare direttamente nella Porta 1 (`ether1` / PoE IN) del MikroTik.

### L'Access Point MikroTik

Il MikroTik è stato "lobotomizzato" per disattivare tutte le sue funzioni di routing di Livello 3 e farlo agire come un puro ponte L2 trasparente (creando un bridge tra la porta cablata e le interfacce wireless).

* **STP Disabilitato:** Lo Spanning Tree Protocol è stato impostato su `none` per evitare che bloccasse preventivamente la porta temendo loop inesistenti.


* **Rogue DHCP:** Il server DHCP nativo (192.168.88.x) è stato distrutto per evitare condizioni di concorrenza (*Race Condition*) con il Kea DHCP di OPNsense. Ora i dispositivi connessi in Wi-Fi ricevono correttamente un IP nella classe `10.0.10.x`.



---

## 5. Accesso Remoto e VPN

Per accedere al laboratorio dall'esterno in totale sicurezza, sono percorribili due strade, a seconda della fornitura dell'operatore internet.

### Soluzione A: WireGuard (Standard)

Utilizzabile se l'ISP fornisce un IP Pubblico.

1. **DDNS:** Configurato un hostname (Record A) tramite No-IP (`nome.ddns.net`).


2. **Port Forwarding:** Il FritzBox inoltra il traffico UDP sulla porta `51820` verso l'indirizzo WAN di OPNsense (`192.168.1.127`).


3. **Firewall:** OPNsense possiede una regola WAN esplicita per consentire il traffico UDP in ingresso sulla porta `51820`.


4. **Istanza VPN:** Creata una sottorete isolata `10.200.0.1/24` e generata una coppia di chiavi crittografiche.


5. **Peer (Client):** Configurato lo smartphone con IP `10.200.0.2/32`. Questa notazione stringente (`/32`) assicura che le chiavi pubbliche del dispositivo siano vincolate a quel singolo IP.



### Soluzione B: Tailscale (Workaround per CGNAT)

Necessaria se l'operatore fornisce un IP nattato (es. `100.64.x.x`).

1. **Plugin:** Installato `os-tailscale` su OPNsense e autorizzato l'URL generato tramite la web console.


2. **Split Tunneling:** OPNsense agisce come *Subnet Router* pubblicando (Advertised Routes) le reti `192.168.1.0/24` e `10.0.10.0/24`.


3. **Client:** Sui dispositivi, tramite app o comando CLI (`sudo tailscale up --accept-routes`), il traffico viene reindirizzato in modo trasparente verso il laboratorio.



---

## 6. Procedura di Migrazione Dati (Il "Muletto")

Per svuotare il vecchio NAS prima della sua reintroduzione formattata nel Lab, è stato architettato un sistema di migrazione resiliente tramite l'uso di spazio temporaneo sul disco di Proxmox.

1. **Creazione Container:** Creato un Container LXC Debian privilegiato (spunta tolta da *Unprivileged*) con 250 GB di disco. Il privilegio di root totale è richiesto per eseguire il montaggio di dischi di rete (CIFS).


2. **Preparazione Ambiente:** All'interno del muletto sono stati installati i pacchetti `rsync`, `openssh-server`, `cifs-utils` e `tmux`. La connessione SSH è stata sbloccata per l'utente root modificando il file `sshd_config` (`PermitRootLogin yes`).


3. **Il Trasferimento Infallibile:**
* Il NAS è stato "montato" temporaneamente.
* Avviando una sessione protetta con il comando `tmux`, è stato lanciato il comando di copia differenziale: `sudo rsync -avhP -e "ssh -o ServerAliveInterval=60" /mnt/vecchionas/ root@192.168.1.95:/root/backup_nas/`.


* Il flag `ServerAliveInterval=60` previene l'errore `Broken pipe` mantenendo in vita il tunnel SSH durante la copia di file enormi, mentre la sessione `tmux` permette all'amministratore di scollegarsi (Ctrl+B, poi D) lasciando il processo attivo in background.