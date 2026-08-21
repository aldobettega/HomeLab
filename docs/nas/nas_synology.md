### 1. Specifiche Hardware

**NAS Principale: Synology DS216play**

* **Processore e RAM:** STM Monaco STiH412 (Dual-Core 1.5 GHz, 32-bit) con 1 GB di RAM DDR3.
* **Dischi Interni:** 2x Western Digital Red da 2 TB (WD20EFRX, 5400 RPM).
* **Configurazione Volume:** SHR/RAID 1 (Mirroring). Genera 1.8 TB di spazio totale (1 TB attualmente occupato) e garantisce continuità operativa in caso di guasto hardware di un disco.
* **Sistema Operativo:** DSM 7.0.

**Disco USB Esterno: WD Elements Desktop (WDBWLG)**

* **Specifiche:** Collegamento USB 3.0, hard disk meccanico interno a 5400 RPM (velocità di trasferimento ~100-130 MB/s).
* **Capienza:** Da verificare tramite il menu *Dispositivi Esterni* sul NAS.

**NAS Secondario: D-Link DNS-323**

* **Specifiche:** Modello legacy con 200 GB di archiviazione totale e vecchi protocolli di rete (SMBv1).

# Azioni

## Creazione cartella condivisa

Creare una cartella condivisa sul synology:

- mettere permessi di lettura scrittura a user con cui si fa accesso e senza crittografia

Predisporre container LXC di proxmox

- dalla gui di proxmox dalle impostazioni del container andare in options>features e attivare SMB/CIFS
- riavviare container
- accedere in ssh al container
- creare cartella con  mkdir -p /mnt/synology
- montare con
    mount -t cifs //ip_nas/cartella_condivisa_nas /mnt/synology -o username=,vers=2.0,cache=none,echo_interval=60