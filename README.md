# static.marktkraam.site

Statische host voor losse bestanden bij Marktkraam: afbeeldingen in e-mails,
demobestanden en vergelijkbaar materiaal dat via een vaste URL bereikbaar moet
zijn.

Er hoort **geen site** te staan. De root verwijst door naar
<https://marktkraam.site>, zodat wie het domein uit een afbeeldings-URL plukt en
er zelf naartoe surft, niet gaat rondkijken maar op de echte site belandt.

## Indeling

Alles wat publiek is, staat in **`www/`**. Alleen die map wordt gepubliceerd;
de rest van de repo blijft erbuiten, zodat `README.md` en `VERSION` niet via het
web opvraagbaar zijn.

```
www/            ← documentroot: alleen dit wordt gepubliceerd
  index.html    doorverwijzing naar https://marktkraam.site/
  404.html      identiek, voor onbekende paden
  robots.txt    Disallow: /
  CNAME         custom domain voor GitHub Pages
.github/workflows/pages.yml   publiceert www/ naar Pages
README.md       ← blijft buiten de documentroot
VERSION
```

| Bestand | Doel |
|---|---|
| `www/index.html` | Doorverwijzing naar `https://marktkraam.site/` — `location.replace()` plus een `meta refresh` als JavaScript uitstaat, met een zichtbare link als beide falen. |
| `www/404.html` | Identiek aan `index.html`, zodat een onbekend pad doorverwijst in plaats van een kale foutpagina te tonen. |
| `www/robots.txt` | `Disallow: /` — niets hier hoort geïndexeerd te worden. `index.html` draagt daarnaast `noindex` en een canonical naar `marktkraam.site`. |
| `www/CNAME` | Zet het custom domain voor GitHub Pages. Moet in de gepubliceerde map staan, dus in `www/` en niet in de repo-root. |
| `.github/workflows/pages.yml` | Publiceert `www/` naar Pages bij elke push naar `main`. |
| `VERSION` | Versiebron volgens de Flux76-baseline (`MAJOR.MINOR.PATCH`). |

De pagina heeft bewust geen huisstijl. Kraam heeft nog geen vastgestelde
productbrand (zie `projects/kraam/branding/README.md` in App-Guidance), en de
kleuren die rond Kraam circuleren zijn die van één tenant — die horen niet op een
platformpagina.

## Bestanden toevoegen

Zet ze in een map **onder `www/`**, bijvoorbeeld `www/mail/logo.png`, en verwijs
ernaar zonder `www` in de URL: `https://static.marktkraam.site/mail/logo.png`.
Alleen de inhoud van `www` wordt gepubliceerd, dus de mapnaam verdwijnt uit het
pad. Raak
`www/index.html` niet aan — dat is de doorverwijzing.

Twee dingen om te onthouden bij bestanden die in e-mail gebruikt worden: een
bestand dat eenmaal is verstuurd blijft jaren opgevraagd worden, dus verplaats of
hernoem het niet meer, en `Disallow: /` in `robots.txt` houdt zoekmachines weg
maar maakt niets privé — alles wat hier staat is publiek voor wie de URL kent.

## Deploy

De site draait op **GitHub Pages** met custom domain `static.marktkraam.site`
(dat is wat `www/CNAME` regelt). Elke push naar `main` publiceert `www/` via
`.github/workflows/pages.yml`; er is verder niets te bouwen of te kopiëren.

Dat de site in `www/` staat en niet in de repo-root, vraagt één instelling:

> **Settings → Pages → Source = "GitHub Actions"** (niet "Deploy from a branch").

Die stap is niet optioneel. Pages serveert bij "deploy from a branch" uitsluitend
vanuit `/` of `/docs` — `/www` is geen keuze die het menu aanbiedt. Staat de
source op een branch, dan wordt `www/` genegeerd en publiceert Pages de repo-root,
inclusief deze README, met een 404 op de doorverwijzing tot gevolg.

Twee dingen die Pages gratis goed doet en die dus nergens geconfigureerd hoeven te
worden: `404.html` wordt vanzelf als foutpagina gebruikt, en er is geen directory
listing, dus `/mail/` valt niet te doorbladeren om te zien wat er staat.

Eén ding dat Pages níet kan: een echte `301` op de root. Dat kan daar alleen als
statische pagina, en precies dat doet `www/index.html`.

### Als dit ooit naar een gewone webserver verhuist

Wijs de documentroot dan naar `www/` (dus `.../static.marktkraam.site/www`, niet
de root van de checkout), zet directory listing uit (`autoindex off` bij nginx,
`Options -Indexes` bij Apache) en wijs `404.html` als foutpagina aan. Een `301`
mag dan wel, maar bind die aan het pad `/` en niet aan de hele server — anders
wordt ook `/mail/logo.png` weggestuurd en is de host nutteloos. `www/CNAME` is
buiten Pages betekenisloos en kan dan weg.

## Lokaal bekijken

```sh
python3 -m http.server 8080 --directory www
# open http://localhost:8080 — je hoort direct op marktkraam.site te belanden
```

Test de fallback door JavaScript uit te zetten: de `meta refresh` neemt het dan
over.

## Grenzen buiten deze repository

Deze host serveert alleen statische bestanden en accepteert geen uploads of
POST-verkeer, dus er is hier geen request-limiet om te documenteren. Wat wél kan
knellen is de andere kant op: als er ooit een groot demobestand bij komt, bepaalt
de webserver (en een eventuele reverse proxy ervoor) of dat volledig uitgeleverd
wordt. Loopt dat mis, dan hoort de grens hier als getal genoemd te worden — zie
`shared/guidelines/project-baseline.md` §6 in App-Guidance.

## Baseline

Deze repo volgt de Flux76-baseline uit
[`Flux76HQ/App-Guidance`](https://github.com/Flux76HQ/App-Guidance). Nog niet
overgenomen, omdat er geen build- of deploystraat is: `scripts/bump_version.py`,
`scripts/check_version_bump.py`, `.githooks/pre-commit` en de CI-versieguard.
