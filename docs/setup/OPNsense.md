# VM o Container?

Per OPNsense verrà usata una VM completa per due motivi

- isolamento dal kernel -> un container condivide il kernel linux con l'host proxmox, ma OPNsense si basa su un OS diverso (FreeBSD), non può girare su linux
- sicurezza e rete -> una vm garantisce che una compromissione del firewall (che esegue operazioni complesse e di basso livello) non si propaghi all'host proxmox

# Paravirtualizzazione o emulazione

Con una vm di solito l'hypervisor emula hw di rete -> ma emulare è un'operazione costosa per la cpu (continua traduzione).
La paravirtualizzazione (VirtIO) consente di usare un canale diretto per parlare direttamente con la scheda di rete dell'host, performance più alte.

# caricare una iso img, creare una vm, scegliere specs

# distribuzione delle risorse

No cpu pinning -> non riservo core di silicio fisici ma assegno vCPU: il servizio esegue calcoli con massimo 2 core per volta.
A gestire il tutto c'è CFS (Completly Fair Scheduler), gestisce risorse assegnate al sistema operativo proxmox (tutte: 6 core fisici).
Le vm passano gran parte del tempo in idle.

Il limite non è la somma delle vCPU ma il carico effettivo simultaneo.

Diverso per la ram: non posso allocare più ram di quanta ne possiedo fisicamente, quando configuro una vm. Con i container la distribuzione della ram imposto un limite massimo e ne viene fatto un uso e distribuzione intelligente tra i container.

# UFS vs ZFS
Sono due file system

- zfs consuma molta ram (la usa come cache del disco), molto accurato per l'integrità dei dati
- ufs è tradizionale, leggero e solido -> fa a caso nostro

# Disco

il disco sul quale installeremo il sistema sarà vtbd0 (Virtual I/O Bloc Device 0) -> il kernel di freebsd riconosce di star comunicando con un hypervisor (->carica driver paravirtualizzato) saltando emulazione.

# noVNC

server headless, soluzione: noVNC, client che mostra a browser che succede.

# Creazione VLAN

Proxmox ha vlan tag vuoto -> vtnet0 riceve traffico untagged dal router e tagged per il lab.

Dobbiamo dividere vtnet0 in più interfacce virtuali:

- vtnet_vlan10 ecc...

# disabilitare pf per accedere senza ethernet da porta dalla wan e accedere a porta di opnsense


Analizzando la dashboard nell'immagine `image_148cd4.jpg`, ti faccio i miei complimenti: è il quadro di un sistema perfettamente in salute. Noterai infatti che qui la RAM è al 19% (il valore reale) e il servizio *QEMU Guest Agent* è verde e operativo. L'aggiornamento alla versione 26.7.1_1 è andato a buon fine e le interfacce hanno gli IP corretti.

Ecco un riepilogo sintetico e strutturato di tutte le operazioni architetturali che abbiamo compiuto finora per costruire il nucleo del tuo HomeLab. Ti sarà molto utile come documentazione di progetto (As-Built).

### 📋 Diario di Configurazione: OPNsense (Router on a Stick)

**1. Preparazione dell'Hypervisor (Proxmox)**

* **VLAN Awareness:** Abilitata l'opzione *VLAN aware* sul bridge virtuale `vmbr0` per trasformarlo in uno switch gestito capace di leggere i tag 802.1Q senza isolare l'host.
* **Creazione VM ottimizzata:**
* Disabilitato il firewall di Proxmox sulla singola interfaccia di rete virtuale (vNIC) per evitare il doppio filtraggio.
* Impostato il disco su *VirtIO Block* per massimizzare le performance dell'SSD.
* Impostata la CPU su *Host* per esporre le istruzioni crittografiche AES-NI della tua CPU Intel.
* Disabilitato il *Ballooning* della RAM per prevenire instabilità del kernel di FreeBSD.



**2. Installazione e Risoluzione Errori (Console)**

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



**4. Sicurezza e Accesso di Emergenza (Shell & WebGUI)**

* **Bypass Temporaneo:** Utilizzato il comando `pfctl -d` da shell per abbattere temporaneamente il packet filtering e accedere alla WebGUI dall'esterno (lato WAN).
* **Regola di Management (Backdoor sicura):** Creata una regola esplicita in ingresso sulla WAN per consentire il traffico HTTPS verso "This Firewall" proveniente solo dalla rete locale (`WAN net`).
* **Risoluzione Conflitto NAT:** Disattivato il blocco delle reti private (RFC 1918) e Bogon sull'interfaccia WAN per permettere al firewall di operare dietro il router casalingo (Double NAT).

**5. Ottimizzazione e Telemetria (WebGUI)**

* **Allineamento Firmware:** Aggiornato il sistema base di OPNsense all'ultima versione disponibile per risolvere i conflitti di dipendenze del gestore pacchetti.
* **Integrazione Hypervisor:** Installato, abilitato e avviato il plugin `os-qemu-guest-agent` per comunicare metriche reali (RAM e IP) a Proxmox tramite il canale seriale VirtIO.