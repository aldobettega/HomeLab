# Come cambiare ID alle macchine

Per cambiare in sicurezza id a VM e Container occorre clonare questi servizi con un ID nuovo.
Per farlo basa eseguire questi passaggi.

## VM

0. spegnere la VM che vogliamo clonare
1. verificare che la VM che stiamo clonando abbia su Hardware > CD/DVD Drive su "Do not use any media"
2. clonare dalla shell del nodo proxmox con `qm clone ID_VM_ATTUALE ID_VM_NUOVO --name NOME --full 1`
3. controllare che si avvi correttamente la macchina clonata
4. eliminare quella vecchia se va tutto bene

## LXC

0. spegnere il container che vogliamo clonare
1. clonare con `pct clone ID_LXC_ATTUALE ID_LXC_NUOVO --name NOME --full 1`
2. controllare che si avvi correttamente il container clonato
3. eliminare quello vecchio se va tutto bene