# Accesso Remoto: Zero Trust VPN con Tailscale su OPNsense

I vecchi servizi di Dynamic DNS (come No-IP) richiedono un IP **pubblico**. Oggi molti provider (soprattutto su linee FWA o fibra) forniscono indirizzi del tipo `100.X.X.X` (es. `100.64.x.x`). Questo indica che siete dietro a un CGNAT (Carrier-Grade NAT). In questo scenario, il port forwarding tradizionale sul router non funziona. 

La soluzione moderna e più sicura (Zero Trust) è utilizzare **Tailscale**, che buca il NAT automaticamente senza aprire alcuna porta sul router.

## 1. Installazione su OPNsense

1. Installare il plugin `os-tailscale` dalla sezione Firmware di OPNsense.
2. Navigare in **VPN > Tailscale > Settings**, spuntare **Enable** e salvare.
3. Andare nella tab **Status** e cliccare sul **Login URL** generato per autorizzare il firewall tramite il vostro account Tailscale.

## 2. Aggiunta Reti (Subnet Router)

Abbiamo aggiunto OPNsense alla rete Tailscale, ma ora dobbiamo dirgli di agire da "ponte" (Subnet Router) per permetterci di raggiungere le VLAN interne dell'HomeLab dall'esterno (Split Tunneling).

1. Su OPNsense, andare in **VPN > Tailscale > Settings > Advanced**.
2. Nel campo **Advertised Routes**, aggiungere le reti del nostro Lab separandole da virgola:
   `192.168.1.0/24,10.0.10.0/24,10.0.20.0/24`
   *(Nota: La 10.0.20.0/24 è fondamentale perché è la VLAN che ospita i nostri server e Nginx Proxy Manager).*
3. Dal sito web di Tailscale (Admin Console), andare nella lista *Machines*, cliccare sui tre puntini di fianco al dispositivo OPNsense, scegliere **Edit route settings** e approvare (spuntare) le reti appena annunciate.

## 3. Configurazione DNS (Magic DNS e Pi-hole)

Per poter digitare dal telefono `immich.lab.lan` ed essere reindirizzati al nostro server anche quando siamo fuori casa in 4G, dobbiamo dire a Tailscale di usare il nostro Pi-hole interno.

Dalla Admin Console web di Tailscale, andare in **DNS**:
1. Alla voce **Nameservers**, cliccare *Add nameserver > Custom...* e inserire l'IP interno di Pi-hole sulla rete di Management: `10.0.10.4`.
2. Spuntare l'opzione **Override local DNS** per forzare i client a usare Pi-hole, bloccando la pubblicità anche in mobilità e permettendo la risoluzione del dominio `lab.lan`.

## 4. Utilizzo sui Client

### Smartphone
1. Installare l'app di Tailscale (iOS/Android).
2. Effettuare il login con lo stesso account.
3. Avviare la VPN. L'app accetterà automaticamente le rotte di OPNsense, facendo passare nel tunnel *solo* il traffico destinato al Lab (garantendo massima velocità per la normale navigazione web). 
*Nota: Selezionare "Use exit node" (OPNsense) solo se si è connessi a un Wi-Fi pubblico insicuro e si desidera far transitare l'intero traffico Internet (tunnel completo) in modo criptato verso casa.*

### PC (Linux / macOS)
Per agganciare le rotte del Lab e le regole DNS di Pi-hole dal terminale:

* **Accendere e sbloccare rotte e DNS:**
  `sudo tailscale up --accept-routes --accept-dns`

* **Spegnere temporaneamente:**
  `sudo tailscale down`

* **Disabilitare definitivamente (evita l'avvio al boot):**
  `sudo systemctl stop tailscaled`
  `sudo systemctl disable tailscaled`