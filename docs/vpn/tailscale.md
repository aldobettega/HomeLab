# Tailscale

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