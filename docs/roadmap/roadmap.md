
### 🛒 1. Roadmap Hardware: L'approccio a strati (Scale-Out)

Ogni strato si aggiunge al precedente, senza mai sostituirlo.

**Strato 1 (La Fondamenta): Il Nodo Compute (Virtualizzazione)**

* **Cosa comprare:**
    * **Mini PC**
* **Requisiti:**
    * Processore Intel N100 (nuovo) oppure Intel Core i5 dall'8a generazione in su (usato, come i Lenovo ThinkCentre M720q o i Dell OptiPlex Micro)
    * Minimo **16GB di RAM** (meglio 32GB)
    * un SSD NVMe da 500GB.
    * Consumo: 10-15 Watt.
* **Costo stimato:** 150€ - 250€.
* **Cosa ci fai:** Qui installerai **Proxmox**. Sarà il cervello del tuo homelab, dove farai girare le tue macchine virtuali e i container Docker.

**Strato 2 (Il Networking): Switch Managed ed eventuale Firewall Fisico**

* **Cosa comprare:**
    * Uno **Switch Layer 2/3 Managed** a 5 o 8 porte (es. TP-Link TL-SG105E o simili).
    * Un mini PC fanless con 4 porte di rete fisiche
* **Costo stimato:**
    * 30€ per lo switch
    * 150€ per il router/firewall fisico.
* **Cosa ci fai:** l
    * lo switch ti servirà per dividere le VLAN fisicamente
    * Il PC con le 4 porte diventerà il tuo **Router pfSense/OPNsense** bare-metal, gestendo la sicurezza di tutta la tua casa.

**Strato 3 (I Dati): Il NAS (Network Attached Storage)**

* **Cosa comprare:**
    * Un NAS pre-assemblato (come un Synology a 2 o 4 bay) oppure un PC custom in cui inserire dischi meccanici molto capienti (IronWolf o WD Red).
* **Costo stimato:**
    * Dai 200€ in su, dipende dalla capienza dei dischi.
* **Cosa ci fai:**
    * Lo usi puramente per lo storage (TrueNAS o Unraid). Farà i backup automatici del tuo Mini PC (Strato 1) e archivierà i tuoi film e documenti.

---

### 💼 2. I Progetti "Curriculum-Killer" (Networking & Cybersecurity)

* **Segmentazione di Rete Avanzata (VLANs & Firewalling):**
    * Usando pfSense o OPNsense, crea regole (ACL) che isolano i dispositivi insicuri (telecamere, smart TV) in una VLAN separata (IoT). Dimostra che il traffico non può passare verso la tua rete principale, ma che tu puoi raggiungere loro.
* **Reverse Proxy & Identity Provider (SSO):**
    * Esponi un servizio su Internet in modo sicuro. Invece di aprire porte a caso sul router, usa **Nginx Proxy Manager** o **Traefik**. Aggiungici un sistema di autenticazione come **Authelia** o **Authentik**: prima di accedere a una tua app, l'utente viene bloccato da una pagina di login con Autenticazione a Due Fattori (2FA).
* **Wazuh (SIEM) o Intrusion Detection System (IDS):**
    * Installa Wazuh (un sistema di monitoraggio per la cybersecurity) o attiva Suricata sul tuo firewall. Fai un attacco simulato e fai uno screenshot dell'allarme generato.
* **Infrastructure as Code (IaC) con Ansible:**
    * Invece di installare i programmi a mano sui tuoi server Linux, scrivi uno "script" Ansible (un file YAML) che si collega via SSH e configura il server in totale autonomia.

---

### 🎮 3. I Progetti "Life-Improver" (Belli, Utili e Divertenti)

Questi sono i servizi che, una volta installati, cambieranno in meglio la tua vita digitale quotidiana e ti daranno tantissima soddisfazione.

* **Pi-hole o AdGuard Home:** 
    * instradare tutto il traffico DNS della tua rete qui dentro farà sparire magicamente la pubblicità dai siti web e dalle app
* **WireGuard (VPN Personale):**
    * Accendi la VPN sul telefono e, magicamente, la tua connessione viene criptata fino a casa tua, navigando in sicurezza
* **Nextcloud:**
    * Il tuo Google Drive / iCloud personale. Sincronizza automaticamente le foto dal tuo smartphone, ospita i tuoi documenti e non ha abbonamenti mensili. Hai il controllo totale dei tuoi dati (Privacy).
* **Jellyfin o Plex:**
    * Il tuo Netflix personale. Scarichi i tuoi film o serie TV sul server e li guardi in streaming comodamente dalla smart TV del salotto, con locandine scaricate in automatico, sottotitoli e suddivisione per stagioni.