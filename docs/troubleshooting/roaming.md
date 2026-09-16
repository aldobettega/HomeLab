# Roaming

Per creare una rete unificata con diversi Access Point in modo da staccarsi - collegarsi a quello col segnale più forte, senza che l'utente del dispositivo se ne accorga, serve impostare gli access point nel seguente modo:

- le reti devono avere stesso SSID (sia per wlan0 che 1, sarebbe 2.4ghz e 5ghz) e password
- sulla stessa rete, **stessi dispositivi**
- impostare gli AP su due canali radio differenti fissi (sia per wlan0 che wlan1)
- entrambi in modalità ap bridge
- configurare il Signla Strenght Rate in modo che l'access point scolleghi il dispositivo se il segnale è troppo basso:
  - Andare in Wireless > Access List > add new
  - impostare Strenght Rate: -120..-75
  - Controllare che Autentication non sia checcato