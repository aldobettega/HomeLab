# Documentazione As-Built & To-Be: Architettura HomeLab

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