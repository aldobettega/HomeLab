# Creazione hostname

Creare un hostname su noip mettendo

* type A (IPv4)
* nome host
* ddns.net

# Inserire hostname nel fritzbox

Inserire nel fritxbox
internet>Abilitazioni>DyDNS

* abilitarlo
inserire:
* url
* nome -> mail
* nome dominio -> nome host + ddns.net

# Attivare wireguard da opnsense

Per poter accedere dall'esterno di opnsense (dalla wan con 192.168.1.127) bisogna accedere alla lan con 10.0.10.1 e deselezionare interfaces>wan>block private networks

Poi dalla gui di opnsense:
0. enable wireguard

1. creare un istanza del server da wireguard (vpn>wireguard>+) impostando come "Listen Port" la 51820.

La vpn ha bisogno di una sottorete di proprietà:
10.200.0.1/24

è necessario creare delle chiavi crittografiche per il server, una pubblica e privata. OPNsense consente di generarle automaticamente premento l'ingranaggio. Salvare la "Public Key" del server perché servirà sullo smartphone.

# Aprire le porte (Fritzbox e OPNsense)

**Sul Fritzbox:**
Bisogna dire al router di mandare il traffico VPN al firewall.

* Creare un'abilitazione porte per il dispositivo OPNsense (192.168.1.127)
* Protocollo: UDP
* Porta: 51820

**Su OPNsense:**
Ora il fritzbox manda il traffico alla porta di opnsense ma dobbiamo abilitare la ricezione delle chiamate da wireguard.

Da firewall>rules>WAN creiamo una regola con

* action: pass
* protocol: udp -> dobbiamo creare una connessione veloce per quando ci connetteremo alla vpn
* source: any
* destination: WAN address
* destination port range: single port>51820
* una descrizione come "consenti ingresso wg"

salvare e applicare.

# Creare il peer

Scaricare app di wireguard ufficiale su smartphone,

1. creare un tunnel wireguard con il tasto +
2. dare un nome
3. generare chiavi pubbliche e private
4. copiare la chiave pubblica e inserirla nella creazione del peer dall'interfaccia di opnsense

accedere dalla gui di opnsense a vpn>wireguard>peers

inserire nome (del dispositivo smartphone), public key generata dall'app.

* allowed IPs è: 10.200.0.2/32 (-> netmask che indica un singolo dispositivo, dico a wireguard che le chiavi crittografiche di questo smartphone sono collegate unicamente dal'indirizzo .2, serve a garantire che ogni dispositivo sulla vpn resti isolato nella sua sottorete) che è la sottorete vpn che avevamo creato.

*Nota finale per WireGuard: ricordarsi di tornare nell'Istanza creata al punto precedente e selezionare questo nuovo Peer nel menu a tendina per collegarli. Sull'app smartphone, inserire l'Endpoint (es. nome.ddns.net:51820) e la Chiave Pubblica del server OPNsense.*

# workaround con tailscale

noip richiede un ip **pubblico**, i provider di rete potrebbero fornire un indirizzo tipo 100.X.X.X (es. 100.64.x.x) che corrisponde ad un CGNAT e con questo non è possibile lavorarci, occorre utilizzare tailscale oppure richiedere al provider un ip **pubblico** (anche dinamico va bene, purché pubblico).

Occorre installare il plugin os-tailscale su opnsense.
Poi attivarlo in VPN>Tailscale>Settings checkando enabled (salvare).
Poi andare in status e cliccare il Login URL per autorizzare la macchina.

## aggiunta reti

Abbiamo aggiunto OPNsense su tailscale, ma bisogna che agisca da ponte per raggiungere il resto del lab (Split Tunneling).

VPN -> Tailscale -> Settings -> tab **Advertised Routes**
aggiungere le reti:

192.168.1.0/24,10.0.10.0/24

Dal sito di tailscale (Admin Console), cliccare sui tre pallini di fianco al dispositivo OPNsense, scegliere "Edit route settings" e approvare le reti che ora compariranno nella sezione Subnets.

## smartphone

installare l'app di tailscale
login con lo stesso account

accendere semplicemente la VPN (su Android accetta le rotte in automatico e farà passare nel tunnel solo il traffico per il Lab). Selezionare "Use exit node" (OPNsense) *solo* se si vuole far passare l'intero traffico Internet del telefono da casa per ragioni di sicurezza su Wi-Fi pubblici.

## installare ed usare sul pc

accendere e sbloccare le rotte pubblicate da OPNsense:
`sudo tailscale up --accept-routes`

spegnere temporaneamente:
`sudo tailscale down`

spegnere definitivamente (non si avvierà da solo all'accensione del PC):
`sudo systemctl stop tailscaled`
`sudo systemctl disable tailscaled`