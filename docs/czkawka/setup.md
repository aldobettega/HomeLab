# Installazione VM

Creare un container LXC con
- OS: ubuntu server / debian 13
- RAM: 2GB
- Core: 4
- Disk: 20GB

Se ssh non funziona, fare da proxmox:

sudo apt update && sudo apt install openssh-server -y
sudo apt update && sudo apt install openssh-server -y

# Installazione pacchetti

Installare pacchetti necessari:

sudo apt update && sudo apt upgrade -y
sudo apt install cifs-utils curl nano -y
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Creazione directory 

sudo mkdir -p /mnt/cartella1
sudo mkdir -p /mnt/cartella2

Verificare dalla gui del synology in
    pannello di controllo>servizi file>sbm>impostazioni avanzate
che sia abilitato a sbm 3.

Creare file di configurazione per credenziali:
    nano ~/.smbcredentials
Inserendo:
    username=tuo_utente_synology
    password=tua_password_synology
e rendendolo non leggibile agli altri utenti della VM:
    chmod 600 ~/.smbcredentials

Modificare fstab di configurazione:
    sudo nano /etc/fstab
Inserendo:
    //192.168.1.224/cartella1nome /mnt/cartella1 cifs credentials=/home/scanner1/.smbcredentials,uid=1000,gid=1000,iocharset=utf8,vers=3.0 0 0
    //192.168.1.224/cartella2nome /mnt/cartella2 cifs credentials=/home/scanner1/.smbcredentials,uid=1000,gid=1000,iocharset=utf8,vers=3.0 0 0
Ricaricare con:
    sudo systemctl daemon-reload
Montare:
    sudo mount -a
Verificare con:
    ls -l /mnt/cartella1
    ls -l /mnt/cartella2

# Installare Czkawka

Creare dir per czkawka con docker file

mkdir ~/czkawka
cd ~/czkawka
nano docker-compose.yml


    services:
  czkawka:
    image: jlesage/czkawka
    container_name: czkawka
    ports:
      - "5800:5800"
    environment:
      - USER_ID=1000
      - GROUP_ID=1000
      - TZ=Europe/Rome
    volumes:
      - ./config:/config:rw
      - /mnt/cartella1:/storage/cartella1:rw
      - /mnt/cartella2:/storage/cartella2:rw
    restart: unless-stopped


Avviare con:
    sudo docker compose up -d


Accedere alla gui con:
    192.168.1.198:5800
