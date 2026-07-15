# DSGVO-Checker auf Coolify deployen

Enthält: `index.html` (gebündelte App), `Dockerfile`, `nginx.conf`.

## Schritt 1: Dateien auf ein Git-Repo bringen
Coolify deployt am einfachsten aus einem Git-Repository (GitHub/GitLab/Gitea).
1. Neues Repo anlegen (privat reicht), z.B. `dsgvo-checker`.
2. Diese drei Dateien (`index.html`, `Dockerfile`, `nginx.conf`) hochladen/pushen.

Falls du lieber ganz ohne GitHub arbeiten willst: Alternative B unten (direkt per SCP auf den Server).

## Schritt 2: In Coolify anlegen
1. Coolify-Dashboard öffnen (http://178.105.170.226:8000)
2. **New Project** → Projekt benennen, z.B. "DSGVO-Checker"
3. **+ New Resource** → **Public Repository** (oder GitHub App verbinden, falls privates Repo)
4. Repo-URL eintragen, Branch `main`
5. **Build Pack**: Dockerfile (wird automatisch erkannt, da ein `Dockerfile` im Repo liegt)
6. **Port**: 80
7. Domain vergeben: entweder Subdomain auf deinem Server (z.B. `dsgvo-checker.victualien.de`) oder erstmal nur über die Server-IP testen
8. **Deploy** klicken

Coolify baut das Docker-Image, startet den Container, und richtet bei eingetragener Domain automatisch SSL (Let's Encrypt) über den eingebauten Reverse Proxy ein.

## Schritt 3: DNS umstellen
Für die gewünschte Domain (z.B. `dsgvo-checker.victualien.de` oder direkt `victualien.de/dsgvo-checker`):
- A-Record bei Strato/IONOS auf `178.105.170.226` setzen
- Warten bis DNS propagiert (bis zu 24h)
- Danach in Coolify die Domain als "verified" prüfen — SSL-Zertifikat wird automatisch ausgestellt

## Schritt 4: Alte Strato-Version abschalten
Erst wenn die neue Version unter der Zieldomain läuft und getestet ist, das alte `/dsgvo-checker/`-Verzeichnis auf Strato Hosting Plus entfernen bzw. die Domain-Weiterleitung entfernen.

---

## Alternative B: Ohne Git, direkt per SCP

Falls kein Git-Repo gewünscht ist:

```bash
scp -r dsgvo-checker-deploy root@178.105.170.226:/opt/dsgvo-checker
ssh root@178.105.170.226
cd /opt/dsgvo-checker
docker build -t dsgvo-checker .
docker run -d --name dsgvo-checker --restart unless-stopped -p 8081:80 dsgvo-checker
```

Danach in Coolify **nicht** als "Resource" verwaltet, sondern manuell — Coolify sieht den Container dann nicht in seiner UI. Für den Anfang/Test okay, für den Dauerbetrieb ist Alternative A (über Coolify + Git) sauberer, weil Updates, Logs und SSL dann zentral über das Dashboard laufen.

## Wichtiger Hinweis
Dies ist der **Fragebogen-Teil** (Ampel-Check), nicht der Live-Crawler. Der Crawler ist ein separates, noch zu bauendes Modul und bekommt ein eigenes Deployment (braucht Node.js/Playwright-Backend statt reinem nginx-Static-Server).
