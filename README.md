# themole.de → the78mole.de

Reine Weiterleitung auf GitHub Pages. Ersetzt die Strato-Weiterleitungsfunktion,
die mit dem Registrar-Transfer am 13.09.2026 erloschen ist (die Domain lieferte
danach HTTP 404).

## Warum so

Ein CNAME am Apex ist im DNS nicht erlaubt, und deSEC bietet kein ALIAS/ANAME
als Ersatz. GitHub Pages löst das über feste Anycast-Adressen, die sich als
A/AAAA eintragen lassen — und weil sie statisch sind, spielt die deSEC-Mindest-
TTL von 3600 s keine Rolle.

`404.html` ist eine Kopie von `index.html`: GitHub Pages liefert sie für jeden
unbekannten Pfad aus, damit leitet auch `themole.de/beliebig` weiter.

## Einrichtung

1. Repo auf GitHub anlegen und pushen
2. Settings → Pages → Source: Branch `main`, Ordner `/`
3. Custom domain: `themole.de` (die Datei `CNAME` setzt das mit)
4. „Enforce HTTPS" aktivieren, sobald das Zertifikat ausgestellt ist

## DNS bei deSEC

```
@     A      185.199.108.153, 185.199.109.153, 185.199.110.153, 185.199.111.153
@     AAAA   2606:50c0:8000::153, 2606:50c0:8001::153, 2606:50c0:8002::153, 2606:50c0:8003::153
www   CNAME  <benutzer>.github.io.
```

Der bestehende A-Record `217.160.0.247` (Stratos Weiterleitungsserver) wird
dabei ersetzt.

## Einschränkung

Die Weiterleitung ist clientseitig, kein HTTP 301 — GitHub Pages kann keine
serverseitigen Redirects. Für einen echten 301 bräuchte es einen eigenen
Webserver, etwa einen nginx-vhost mit
`return 301 https://the78mole.de$request_uri;`.
