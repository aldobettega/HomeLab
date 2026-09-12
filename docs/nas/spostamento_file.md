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
