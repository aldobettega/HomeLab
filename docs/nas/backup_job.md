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
   - Keep last 2