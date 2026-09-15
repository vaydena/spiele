# spiele.vaydena.de — „Blatti & Blumauto"

Ein kleines, eigenständiges Wiesen-Duell im Pokémon-Stil zum Mitspielen —
ein Geschenk für **Leon** zur Einschulung. Läuft komplett im Browser
(eine einzige `index.html`, keine Server-Logik, keine Datenbank), auf
Tablet und Smartphone genauso wie am Rechner.

**Live:** https://spiele.vaydena.de

## Inhalt

| Datei | Zweck |
|-------|-------|
| `index.html` | Das komplette Spiel (Markup + CSS + JS + eingebettetes Grußwort als Audio-Data-URI). Eigenständiges Dokument mit `<!doctype>`, Viewport-Meta und Grund-Reset. |
| `favicon.svg` | Seiten-Icon (grüne Kachel mit 🌱). |
| `deploy-version.txt` | Erste Zeile = eindeutiger Marker, den der Verify-Schritt live gegenprüft. |

Das Spiel ist die eigenständige Fassung des Artifacts „Blatti & Blumauto":
zwei spielbare Wesen (Blatti = Pflanze, Blumauto = Pflanze+Fee), Gegner
**Glühkäfer** (Feuer+Käfer), mehrere Attacken, Sieges-Animation und ein
gesprochenes Grußwort beim Start („Alles Gute Leon zu deinem ersten
Einschulungstag!"). Responsiv und touch-tauglich.

## Deploy-Pipeline

Standardweg wie bei allen statischen Vaydena-Seiten:

```
git push origin main  →  GitHub Actions (.github/workflows/deploy.yml)  →  curl-FTPS  →  Hostinger (public_html/spiele)
```

- **`.github/workflows/deploy.yml`** lädt bei jedem Push auf `main` (oder
  manuell per *Run workflow*) den Seitenbaum per FTPS nach `/spiele/` hoch
  und verifiziert anschließend live (Titel, Dateigrößen, Marker) — mit
  Browser-User-Agent gegen die hCDN-Bot-Challenge.
- **`deploy-local.ps1`** ist der Notweg für den Fall, dass das Secret noch
  fehlt oder es sofort live muss: lädt lokal per `curl.exe` (Windows,
  TLS 1.2) mit denselben Selbstheilungs-Ausweichwegen hoch. Passwort
  interaktiv, nie im Klartext.

### Einmalige Voraussetzung (der eine echte User-Schritt)

Damit der Workflow tatsächlich deployt, muss im Repo das Secret
**`FTP_PASSWORD`** gesetzt sein (dasselbe Deploy-FTP-Passwort wie bei den
anderen Vaydena-Seiten):

> **Settings → Secrets and variables → Actions → Tab „Secrets" →
> „New repository secret"** · Name `FTP_PASSWORD`, Wert = reines Passwort
> des Deploy-Kontos `u424339903.deploy`.

Solange das Secret fehlt, bleibt der Lauf **grün** und überspringt den
Upload (kein Fehlschlag). Wird es versehentlich als *Variable* statt als
*Secret* angelegt, bricht der Lauf mit klarer Anleitung ab.

**FTP-Konstanten** (im Workflow gesetzt): Host `ftp.vaydena.de`,
User `u424339903.deploy`, Zielordner `spiele`.

## ⛔ Wird nie mit hochgeladen

`deploy-local.*`, `README.md`, `.gitignore`, `.git/`, `.github/`,
`*.local.txt` — die Ausschlüsse stehen in beiden Deploy-Skripten.

## Konventionen (Deploy-Fallen, hart erkauft)

- **Nie Login-Wiederholungen** gegen Hostingers FTP: nach mehreren
  Fehl-Logins (530) sperrt der Brute-Force-Schutz die IP (danach curl 28).
  Beide Skripte machen genau **einen** Login-Versuch.
- **Uploads gebündelt** (6 Dateien je curl-Aufruf = ein Login, mehrere STOR).
- **Windows-curl** immer `--tlsv1.2 --tls-max 1.2` (Schannel brach sonst
  beim TLS-Abbau der Datenverbindung ab: „450 Link lost").
- **Verify mit Browser-UA** (sonst liefert hCDN evtl. eine Bot-Challenge
  statt der Datei → falsche Größe/Titel).
