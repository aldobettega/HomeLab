# Formattazione NAS D-Link

Un disco era rotto, quindi occorre implementare l'impostazione single disk. Sulle impostazioni del mio D-Link sono andato su **Tools > RAID**.

# Inserimento nel Lab (VLAN 20 - Storage)

*Nota: A causa di limitazioni hardware e timeout del client DHCP interno al dispositivo D-Link quando interfacciato con porte switch in modalità VLAN Access, il NAS fallisce l'acquisizione dinamica dell'IP. È obbligatorio bypassare il DHCP di OPNsense e configurarlo con un IP statico.*

1. **Assegnazione IP Statico (dalla GUI del NAS):**
    - **IP Address:** `10.0.20.50`
    - **Subnet Mask:** `255.255.255.0`
    - **Gateway:** `10.0.20.1` (OPNsense)
    - **DNS:** `10.0.10.4` (Pi-hole)

2. **Configurazione Switch TP-Link (Porta 2):**
    - **VLAN > 802.1Q VLAN:** Inserire VLAN ID = 20 (Rete Servers) e impostare la Porta 2 (dove è collegato il NAS) su **Untagged**. Salvare.
    - **VLAN > 802.1Q PVID Setting:** Inserire PVID = 20 e applicarlo alla Porta 2. Salvare.

# Risoluzione Problemi NTLMv1 (Workaround SMB1)

I NAS vecchi usano protocolli obsoleti (SMB 1.0) e sistemi di crittografia password (NTLMv1) che i kernel Linux moderni di Proxmox hanno rimosso di default per ragioni di sicurezza. Per evitare l'errore `Permission denied` in fase di mount, la soluzione più pratica in un ambiente domestico sicuro (essendo il NAS isolato in VLAN 20) è eliminare l'autenticazione tramite password.

1. Entra nel pannello web del NAS (`10.0.20.50`).
2. Nelle impostazioni della cartella condivisa (es. `Volume_1`), imposta i permessi su **Guest Access** (oppure User: All) con diritti di **Lettura/Scrittura**. In questo modo non sarà richiesta alcuna password per l'accesso.

# Connessione e Mount Permanente su Proxmox

Poiché il NAS si trova nella VLAN 20 e l'host Proxmox risiede nella VLAN 1 (per ragioni di management d'emergenza), occorre creare un'interfaccia virtuale su Proxmox per permettere la comunicazione diretta in Layer 2 (senza appesantire il firewall virtuale di OPNsense con traffico di routing pesante).

### 1. Creazione Interfaccia Virtuale (VLAN 20) su Proxmox
1. Dalla GUI di Proxmox, andare su **System > Network**.
2. Cliccare su **Create > Linux VLAN**:
    - **Name:** `vmbr0.20`
    - **IPv4/CIDR:** `10.0.20.100/24` *(IP assegnato a Proxmox nella VLAN Storage)*
    - **Gateway:** *Lasciare rigorosamente vuoto!* *(Proxmox ha già il router ISP come gateway predefinito).*
3. Cliccare **Apply Configuration**.

### 2. Mount tramite fstab
Apri la Shell di Proxmox e digita:

1. Crea la cartella di destinazione:
   `mkdir -p /mnt/NAS_Backup`
2. Apri il file degli hard disk di sistema:
   `nano /etc/fstab`
3. Aggiungi questa riga alla fine del file per forzare il vecchio protocollo `vers=1.0` all'avvio e puntare al nuovo IP:
   `//10.0.20.50/Volume_1 /mnt/NAS_Backup cifs guest,vers=1.0,iocharset=utf8 0 0`
4. Salva (Ctrl+O, Invio) ed esci (Ctrl+X).
5. Monta il disco immediatamente per testarlo:
   `mount -a`

### 3. Aggiungere lo Storage alla WebGUI di Proxmox
Ora che Linux vede il NAS come un disco interno, diciamo all'hypervisor Proxmox di usarlo:

1. Vai su **Datacenter > Storage**.
2. Clicca su **Add > Directory** *(Importante: Non usare la voce SMB/CIFS nativa di Proxmox!)*.
3. Compila in questo modo:
    - **ID:** `NAS_Backup`
    - **Directory:** `/mnt/NAS_Backup`
    - **Content:** Selezionare **VZDump backup file** (per i backup delle VM) e/o **ISO image**.
4. Clicca su **Add**.