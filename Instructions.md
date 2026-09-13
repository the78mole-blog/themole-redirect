# Auftrag: Weiterleitungs-Repo für themole.de anlegen

Diese Datei richtet sich an eine Claude-Instanz, die in diesem Verzeichnis
arbeitet. Die Inhalte liegen bereits vor; zu tun ist die Veröffentlichung.

## Ziel

`themole.de` soll auf `https://the78mole.de/` weiterleiten, unter Mitnahme von
Pfad, Query und Anker. Umgesetzt als statische Seite auf GitHub Pages.

## Warum es das gibt

Die Domain wurde am 13.09.2026 von Strato zu INWX transferiert. Stratos
Weiterleitungsfunktion war **kein DNS-Record**, sondern ein Dienst des
Vertrags — sie erlosch mit dem Transfer, und die Domain liefert seither
HTTP 404.

Ein CNAME am Apex ist im DNS nicht erlaubt, und deSEC (der DNS-Anbieter)
bietet kein ALIAS/ANAME als Ersatz. GitHub Pages löst das über feste
Anycast-Adressen, die sich als A/AAAA eintragen lassen.

## Vorhandene Dateien — NICHT verändern

| Datei | Zweck |
|---|---|
| `CNAME` | enthält `themole.de`, setzt die Custom Domain beim Push |
| `index.html` | Meta-Refresh **und** `location.replace()` für die Pfaderhaltung |
| `404.html` | Kopie von index.html — Pages liefert sie für unbekannte Pfade |
| `README.md` | Begründung und DNS-Angaben |

`404.html` ist bewusst identisch mit `index.html`. Ohne sie würde
`themole.de/beliebig` eine Fehlerseite zeigen statt weiterzuleiten.

## Vor dem Start klären

**Unter welchem Konto soll das Repo liegen?** Es gibt zwei sinnvolle Orte,
und die Wahl hat eine Folge im DNS:

- Organisation **`the78mole-blog`** — dort liegt bereits
  `the78mole-blog/the78mole-blog.github.io`, das Ziel der Weiterleitung
- Persönliches Konto **`the78mole`**

GitHub verlangt für Custom Domains unter Umständen einen TXT-Record
`_github-pages-challenge-<owner>`, und `<owner>` ist genau dieser Name. Die
Entscheidung deshalb **vom Benutzer bestätigen lassen**, nicht raten.

## Schritte

```bash
cd ~/GIT/the78mole/themole-redirect
git init -b main
git add .
git commit -m "Weiterleitung themole.de -> the78mole.de"

# <OWNER> durch das bestätigte Konto ersetzen
gh repo create <OWNER>/themole-redirect --public --source=. --remote=origin --push
```

Danach GitHub Pages aktivieren:

```bash
gh api -X POST repos/<OWNER>/themole-redirect/pages \
  -f 'source[branch]=main' -f 'source[path]=/'
gh api repos/<OWNER>/themole-redirect/pages     # Zustand pruefen
```

Sobald das Zertifikat ausgestellt ist, HTTPS erzwingen:

```bash
gh api -X PUT repos/<OWNER>/themole-redirect/pages -F https_enforced=true
```

## Prüfen

```bash
gh api repos/<OWNER>/themole-redirect/pages \
  | python3 -c 'import json,sys; d=json.load(sys.stdin); print(d["status"], d["html_url"], d.get("cname"))'
```

Erwartet: `built`, und `cname` gleich `themole.de`.

Verlangt GitHub eine Domain-Verifikation, erscheint sie in der Ausgabe oder
unter Settings → Pages. **Den geforderten TXT-Wert an den Benutzer melden** —
er wird im Projekt `~/GIT/the78mole/Playground/domainumzug` bei deSEC
eingetragen, nicht hier.

## Ausdrücklich NICHT tun

- **Kein DNS ändern.** Die Zone liegt bei deSEC und wird über das Projekt
  `domainumzug` gepflegt. Dort sind zu setzen:

  ```
  @    A      185.199.108.153, 185.199.109.153, 185.199.110.153, 185.199.111.153
  @    AAAA   2606:50c0:8000::153, 2606:50c0:8001::153, 2606:50c0:8002::153, 2606:50c0:8003::153
  www  CNAME  <OWNER>.github.io.
  ```

  Der bestehende A-Record `217.160.0.247` (Stratos Weiterleitungsserver) wird
  dabei ersetzt. **Erst nach erfolgreichem Build umstellen** — vorher lieferte
  GitHub eine Fehlerseite aus.

- **Den Inhalt nicht „verbessern".** Die Doppelung aus Meta-Refresh und
  Javascript ist Absicht: das Refresh greift ohne Javascript, das Skript nimmt
  den Pfad mit. Keins von beidem allein reicht.

- **Kein `jekyll`-Gedöns ergänzen.** Es sind zwei statische Dateien; eine
  `.nojekyll` ist nicht nötig, weil keine Unterstriche im Spiel sind.

## Bekannte Einschränkung

Die Weiterleitung ist clientseitig, **kein HTTP 301** — GitHub Pages kann keine
serverseitigen Redirects. Für Besucher unsichtbar, für Suchmaschinen dank
`canonical` weitgehend gleichwertig. Wer einen echten 301 will, braucht einen
eigenen Webserver (`return 301 https://the78mole.de$request_uri;`).

## Gleiches Muster, später

`bizzmark.de` → `bizzmark.io` und eventuell `4-null.org` sollen genauso
umgeleitet werden. Dieses Repo taugt als Vorlage; zu ändern sind nur `CNAME`
und die Zieladresse in beiden HTML-Dateien.
