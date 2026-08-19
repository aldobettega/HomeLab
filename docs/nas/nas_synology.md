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

---

### 2. Future Impostazioni (Roadmap 3-2-1)

Per mettere in completa sicurezza i tuoi dati ed evitare sprechi di memoria, lavoreremo su queste tre fasi:

**Fase A: Migrazione e rimozione duplicati**

1. Creare una cartella di transito sul Synology.
2. Usare un computer come "ponte" per copiare i 200 GB dal vecchio D-Link al Synology.
3. Utilizzare software come **dupeGuru** (su PC) o l'app **Analizzatore Archiviazione** (sul NAS) per identificare ed eliminare in blocco i file già presenti sul Synology, tenendo solo le novità.

**Fase B: Backup Storico Locale**

1. Dedicare il disco USB WD Elements *esclusivamente* ai backup.
2. Installare e configurare l'applicazione **Hyper Backup** per eseguire salvataggi automatici incrementali.
3. Attivare lo storico delle versioni (versioning) per poter recuperare dati in caso di cancellazione umana, corruzione o attacchi ransomware.

**Fase C: Backup Remoto (Antidisastro)**

1. Formattare il D-Link una volta svuotato.
2. Posizionarlo fisicamente in un altro edificio (casa di parenti/ufficio).
3. Configurarlo per ricevere dal Synology una copia di sicurezza automatica dei soli dati vitali, garantendoti protezione da disastri fisici domestici (furti, incendi, fulmini).

*(Nota opzionale di sicurezza: ricordati che, qualora il NAS Synology fosse in un luogo accessibile a estranei, potrai disabilitare il tasto RESET fisico dal Pannello di Controllo).*