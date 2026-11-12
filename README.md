# ai.spielmann.it

Eine einzelne HTML Seite, auf der man kleine Sprachmodelle direkt im Browser laden und damit chatten kann. Die Modelle laufen lokal auf der Grafikkarte des Besuchers (WebLLM, WebGPU). Der Server liefert nur die Seite aus und sieht keine Gespräche.

## Aufbau

| Pfad | Zweck |
|---|---|
| `site/index.html` | Die ganze Seite. Wird 1:1 ausgeliefert |
| `nginx/default.conf` | Konfiguration des Webservers (kein Cache für Seiten, `noindex`) |
| `docker-compose.yml` | nginx Container `ai-site`, hängt am Netzwerk `proxy` (Caddy) |
| `setup-winkel.sh` | Richtet alles auf HP-Winkel ein, kann beliebig oft laufen |

## Was die Seite von aussen lädt

Beim Besuch lädt der Browser die Bibliothek von `esm.run` (jsDelivr). Beim Laden eines Modells kommen die Gewichte von Hugging Face und die Modellbibliotheken von GitHub. Der eigene Server ist daran nicht beteiligt, besuchende Personen sprechen also mit diesen Diensten.

WebGPU funktioniert nur über HTTPS. Das übernimmt Caddy.

## Update einspielen

1. In `site/index.html` ändern, auf `main` committen, `git push`.
2. Live schalten: `git publish` (Alias auf dem Laptop) oder `git push origin HEAD:release`.
3. Nach höchstens einer Minute holt HP-Winkel den Stand (Pull Job `winkel-deploy.timer`). Kontrolle: `~/docker/winkel-deploy/deploy.log` auf HP-Winkel.

Vorher mit `git log origin/release..main` ansehen, was alles live ginge.

Ändert sich nur `site/`, startet kein Container neu. Ändern sich `nginx/` oder `docker-compose.yml`, baut der Pull Job den Container neu.

## Einrichtung auf HP-Winkel (einmalig)

1. **DNS bei Metanet:** A Record `ai` auf die öffentliche IP (46.126.153.195). Zuerst, sonst scheitert das Zertifikat.
2. Auf HP-Winkel:

        git clone -b release https://github.com/JanickSpielmann/ai-site.git ~/docker/ai-site
        cd ~/docker/ai-site && ./setup-winkel.sh

   Das Skript startet den Container, ergänzt den Block in `~/docker/caddy/Caddyfile` (mit Sicherung, bei einem Fehler wird sie zurückgespielt), lädt Caddy neu und testet die Seite.
3. **Pull Job:** Das Skript zeigt am Ende, wenn in `~/docker/winkel-deploy/winkel-deploy.sh` die Zeile fehlt. Sie lautet:

        deploy ai-site '^(nginx/|docker-compose\.yml)' '--force-recreate'

   Neu gebaut wird nur, wenn sich `nginx/` oder `docker-compose.yml` ändern. Änderungen an `site/` brauchen keinen Neustart.
4. Test von aussen mit mobilen Daten.

Das Repository ist öffentlich, der Klon braucht deshalb keinen Deploy Key.
