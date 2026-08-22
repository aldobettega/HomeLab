# Creare la VM

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
   - aggiungere alla fine: //192.168.1.50/Volume_1 /mnt/immich_photos cifs guest,vers=1.0,iocharset=utf8 0 0
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

