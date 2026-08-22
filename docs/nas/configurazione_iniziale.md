# creare maccchina muletto

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

     
