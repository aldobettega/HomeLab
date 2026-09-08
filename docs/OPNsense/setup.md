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
    ssh root@10.0.10.1