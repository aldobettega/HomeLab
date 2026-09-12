# Configurazione: OPNsense (Router on a Stick)

## 1. Preparazione dell'Hypervisor (Proxmox)

* **VLAN Awareness:** Abilitata l'opzione *VLAN aware* sul bridge virtuale `vmbr0` per trasformarlo in uno switch gestito capace di leggere i tag 802.1Q senza isolare l'host.
* **Creazione VM ottimizzata:**
  * Disabilitato il firewall di Proxmox sulla singola interfaccia di rete virtuale (vNIC) per evitare il doppio filtraggio.
  * Impostato il disco su *VirtIO Block* per massimizzare le performance dell'SSD.
  * Impostata la CPU su *Host* per esporre le istruzioni crittografiche AES-NI della CPU Intel.
  * Disabilitato il *Ballooning* della RAM per prevenire instabilità del kernel di FreeBSD.

## 2. Installazione e Risoluzione Errori (Console)

* **Provisioning:** Aumentata temporaneamente la RAM della VM a 4 GB per superare il limite del Ramdisk durante la copia del file system (Live CD).
* **File System:** Scelto *UFS* (rispetto a ZFS) per risparmiare preziosa memoria RAM in ottica HomeLab.

## 3. Network Design e Topologia Logica (Console)

* **Sdoppiamento Scheda (VLAN 10):** Creata l'interfaccia virtuale `vtnet0_vlan10` basata sulla singola interfaccia fisica.
* **Assegnazione Ruoli Base:**
  * **WAN:** assegnata a `vtnet0` (traffico *Untagged* proveniente dal router ISP / rete base).
  * **Management:** assegnata a `vtnet0_vlan10` (traffico *Tagged* isolato per l'hardware di rete).
* **Configurazione IP e Servizi (VLAN 10):**
  * Impostato IP statico del gateway a `10.0.10.1/24`.
  * Attivato il server **Kea DHCP** per il range `10.0.10.100 - 200`.
* **Completamento Router-on-a-Stick:** Successivamente (dalla WebGUI), la stessa procedura di sdoppiamento dell'interfaccia è stata ripetuta per creare le altre sottoreti (VLAN 20 Servers, VLAN 30 Trusted, VLAN 40 Guests), agganciandole sempre all'interfaccia genitore `vtnet0`.

## 4. Sicurezza e Accesso di Emergenza (Shell & WebGUI)

* **Bypass Temporaneo:** Utilizzato il comando `pfctl -d` da shell per abbattere temporaneamente il packet filtering e accedere alla WebGUI dall'esterno (lato WAN).
* **Regola di Management (Backdoor sicura):** Creata una regola esplicita in ingresso sulla WAN per consentire il traffico HTTPS verso "This Firewall" proveniente solo dalla rete locale (`WAN net`).
* **Risoluzione Conflitto NAT:** Disattivato il blocco delle reti private (RFC 1918) e Bogon sull'interfaccia WAN per permettere al firewall di operare dietro il router casalingo (Double NAT).

## 5. Ottimizzazione e Telemetria (WebGUI)

* **Allineamento Firmware:** Aggiornato il sistema base di OPNsense all'ultima versione disponibile per risolvere i conflitti di dipendenze del gestore pacchetti.
* **Integrazione Hypervisor:** Installato, abilitato e avviato il plugin `os-qemu-guest-agent` per comunicare metriche reali (RAM e IP) a Proxmox tramite il canale seriale VirtIO.

## 6. Abilitare l'Accesso SSH

Per facilitare la gestione da terminale:
1. Andare su **System > Settings > Administration**.
2. Abilitare la voce **Enable Secure Shell (SSH)**.
3. (Opzionale ma consigliato) Permettere l'accesso root e l'autenticazione tramite chiave pubblica.

Accedere da terminale con:
```bash
ssh root@10.0.10.1