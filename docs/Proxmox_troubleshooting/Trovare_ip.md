# Problema nel trovare IP dispositivi

Nello spostare il mikrotik da una rete all'altra ho avuto diversi problemi nel trovare il suo ip: era nella rete interna del lab nella VLAN 30 e spostandolo nella  WAN di casa 192.168.1.0/24 non vedevo più il dispositivo.
È stato necessario:
- eliminare dalle interfacce di rete il suo vecchio IP sulla VLAN 30
- spegnere e riaccendere la porta dello switch dalle sue impostazioni.
Probabilmente solo spegnendo e riaccendendo si è costretto l'access point in modalità DHCP a richiedere un nuovo IP, probabilmente era rimasto "bloccato" su una vecchia configurazione di rete e non chiedeva l'IP sulla WAN con la conseguenza di essere invisibile.
Il miglior troubleshooting, ancora una volta, si è rivelato spegnere e riaccendere.