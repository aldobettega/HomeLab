# Problema: nmap mostra tutti e 256 gli indirizzi

Se si ha attiva una vpn come tailscale, pouò essere che eseguendo:
    `nmap -sn 192.168.1.0/24`
Si ottenga risposta da tutti i 256 indirizzi, rendendo "sporco" l'output del comando poichè il traffico sta finendo dentro il tunnel della VPN.
Invece di usare la scheda di rete wifi con il protocollo ARP, vengono iniettati i pacchetti all'interno di `tailscale0` che crede che l'host di qualsiasi indirizzo sia attivo.
Per questo serve forzare il comando ad usare la scheda di rete corretta:
    `sudo nmap -sn -PR -e wlp1s0 192.168.1.0/24`

- `-sn` ping scan: esegue solo host discovery e non analisi accurata delle porte di ognuno
- `-PR` ARP ping: forza l'uso di richieste ARP invece di ping ICMP o probe TCP
- `-e wlp1s0` forza l'uscita dei pacchetti esclusivamente sulla scheda wifi fisica, impedendo l'OS di consultare la tabella di routing interna, dirottando le richieste verso l'interfaccia virtuale di tailscale

Oppure semplicemente spegnere tailscale.