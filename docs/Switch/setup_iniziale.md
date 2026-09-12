# Architettura router on a stick

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
* **Il Rogue DHCP (Race Condition):** Il tuo telefono non riusciva a navigare perché, appena si collegava al Wi-Fi, il server DHCP nativo del MikroTik (192.168.88.x) gli offriva un IP prima che potesse farlo OPNsense (10.0.10.x). È una classica *Race Condition*: vince chi risponde più in fretta (ed essendo il MikroTik l'antenna fisica, vinceva sempre). Uccidendo il server ribelle, il telefono è stato obbligato ad attendere pazientemente l'IP fornito da OPNsense attraverso il tunnel VLAN.