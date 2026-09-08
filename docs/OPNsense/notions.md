# Nozioni

## VM o Container?

Per OPNsense verrà usata una VM completa per due motivi

- isolamento dal kernel -> un container condivide il kernel linux con l'host proxmox, ma OPNsense si basa su un OS diverso (FreeBSD), non può girare su linux
- sicurezza e rete -> una vm garantisce che una compromissione del firewall (che esegue operazioni complesse e di basso livello) non si propaghi all'host proxmox

## Paravirtualizzazione o emulazione

Con una vm di solito l'hypervisor emula hw di rete -> ma emulare è un'operazione costosa per la cpu (continua traduzione).
La paravirtualizzazione (VirtIO) consente di usare un canale diretto per parlare direttamente con la scheda di rete dell'host, performance più alte.

## distribuzione delle risorse

No cpu pinning -> non riservo core di silicio fisici ma assegno vCPU: il servizio esegue calcoli con massimo 2 core per volta.
A gestire il tutto c'è CFS (Completly Fair Scheduler), gestisce risorse assegnate al sistema operativo proxmox (tutte: 6 core fisici).
Le vm passano gran parte del tempo in idle.

Il limite non è la somma delle vCPU ma il carico effettivo simultaneo.

Diverso per la ram: non posso allocare più ram di quanta ne possiedo fisicamente, quando configuro una vm. Con i container la distribuzione della ram imposto un limite massimo e ne viene fatto un uso e distribuzione intelligente tra i container.

## UFS vs ZFS
Sono due file system

- zfs consuma molta ram (la usa come cache del disco), molto accurato per l'integrità dei dati
- ufs è tradizionale, leggero e solido -> fa a caso nostro

## Disco

il disco sul quale installeremo il sistema sarà vtbd0 (Virtual I/O Bloc Device 0) -> il kernel di freebsd riconosce di star comunicando con un hypervisor (->carica driver paravirtualizzato) saltando emulazione.

## noVNC

server headless, soluzione: noVNC, client che mostra a browser che succede.

## Creazione VLAN

Proxmox ha vlan tag vuoto -> vtnet0 riceve traffico untagged dal router e tagged per il lab.

Dobbiamo dividere vtnet0 in più interfacce virtuali:

- vtnet_vlan10 ecc...

