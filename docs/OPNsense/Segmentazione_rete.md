# Segmentazione Rete e Firewall (Zero Trust)

## 1. Segmentazione Logica (VLAN)

### 1.1 Creazione dei Tag VLAN

Dividiamo la rete dell'homelab in sottoreti virtuali (VLAN). Questo consente di isolare il traffico per rendere l'infrastruttura sicura e manutenibile.
Per gestirle accedere a **OPNsense > Interfaces > Other Types > VLAN**.

Creare le seguenti reti:
- **VLAN 1 (Transit/WAN):** Rete casalinga del Fritzbox, ospita l'interfaccia di emergenza di Proxmox.
- **VLAN 10 (Management):** Contiene dispositivi di infrastruttura di rete (router, switch, access point, DNS).
- **VLAN 20 (Servers & Storage):** Contiene i servizi offerti dall'homelab (Nginx, Jellyfin, Immich, NAS).
- **VLAN 30 (Trusted):** Rete privilegiata per amministratori.
- **VLAN 40 (Guests / Users):** Rete per utenti Wi-Fi isolata dal resto dell'infrastruttura.

*(Inserire qui lo screenshot `image.png`)*

### 1.2 Assegnazione delle Interfacce

Dopo aver creato i tag 802.1Q, occorre trasformarli in interfacce di rete gestibili navigando su **OPNsense > Interfaces > Assignments**.

*(Inserire qui lo screenshot `image-1.png`)*

### 1.3 Configurazione degli IP (Gateway delle VLAN)

Ora su **OPNsense > Interfaces** compariranno le nuove VLAN nel menu laterale. Selezionarle una ad una per assegnare un IPv4, che rappresenterà il Gateway di OPNsense su quella specifica sottorete.
Per ogni VLAN impostare:
- **Enable Interface:** (Spuntato)
- **IPv4 Configuration Type:** Static IPv4
- **IPv4 Address:** L'indirizzo del gateway (es. `10.0.10.1/24`, `10.0.20.1/24`, ecc.)

---

## 2. Configurazione Servizi DHCP (Kea)

Creiamo i pool DHCP: range di indirizzi assegnati in automatico ai dispositivi che si connettono.
Navigare su **OPNsense > Services > Kea DHCP > Kea DHCPv4 > Subnets** (oppure ISC DHCP se non siete ancora passati a Kea).

- **Pool per VLAN 10 (Management):**
  - Subnet: `10.0.10.0/24`
  - Range: `10.0.10.100 - 10.0.10.200`
- **Pool per VLAN 30 (Trusted):**
  - Subnet: `10.0.30.0/24`
  - Range: `10.0.30.50 - 10.0.30.200`
- **Pool per VLAN 40 (Guests):**
  - Subnet: `10.0.40.0/24`
  - Range: `10.0.40.50 - 10.0.40.200`

*(Nota: I NAS e i server sulla VLAN 20 non necessitano di un pool DHCP dinamico, poiché richiedono IP statici per garantire affidabilità ai servizi e ai mount di rete).*

Salvare e **riavviare il servizio DHCP**.

---

## 3. Configurazione Firewall (Zero Trust Base)

Di default, OPNsense blocca tutto il traffico in entrata sulle nuove interfacce. Dobbiamo creare delle regole navigando su **OPNsense > Firewall > Rules > [Nome Interfaccia]**.

### Regola per TRUSTED (VLAN 30) - Allow All
Questa impostazione permissiva permette alla rete fidata degli amministratori di uscire su internet e di raggiungere le altre VLAN per gestire i server.
- **Action:** Pass
- **Protocol:** Any
- **Source:** `Trusted net`
- **Destination:** Any

### Regole per GUESTS (VLAN 40) - Isolamento Zero Trust
*Attenzione: Non usare MAI un banale "Allow All" sulla rete ospiti! I dispositivi IoT o i telefoni degli ospiti compromessi potrebbero attaccare i vostri server.*
Per farli navigare su Internet ma bloccarli dalle reti private, occorre prima creare un Alias (es. `RFC1918` contenente le reti locali `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) sotto *Firewall > Aliases*, e poi creare due regole in ordine:

1. **Regola 1 (Blocco verso il Lab):**
   - **Action:** Block
   - **Protocol:** Any
   - **Source:** `Guests net`
   - **Destination:** Alias `RFC1918` (Le tue reti private)
2. **Regola 2 (Accesso a Internet):**
   - **Action:** Pass
   - **Protocol:** Any
   - **Source:** `Guests net`
   - **Destination:** Any