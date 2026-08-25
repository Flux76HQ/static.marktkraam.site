# static.marktkraam.site

Statische host voor losse bestanden bij Marktkraam: afbeeldingen in e-mails,
demobestanden en vergelijkbaar materiaal dat via een vaste URL bereikbaar moet
zijn.

Er hoort **geen site** te staan. De root verwijst door naar
<https://marktkraam.site>, zodat wie het domein uit een afbeeldings-URL plukt en
er zelf naartoe surft, niet gaat rondkijken maar op de echte site belandt.

## Indeling

Alles wat publiek is, staat in **`www/`**. Die map is de documentroot van de
webserver; de rest van de repo hoort daarbuiten te blijven, zodat `README.md`,
`VERSION` en `.git/` nooit via het web opvraagbaar zijn.

```
www/            ← documentroot: alleen dit wordt geserveerd
  index.html    doorverwijzing naar https://marktkraam.site/
  404.html      identiek, voor onbekende paden
  robots.txt    Disallow: /
README.md       ← blijft buiten de documentroot
VERSION
```

| Bestand | Doel |
|---|---|
| `www/index.html` | Doorverwijzing naar `https://marktkraam.site/` — `location.replace()` plus een `meta refresh` als JavaScript uitstaat, met een zichtbare link als beide falen. |
| `www/404.html` | Identiek aan `index.html`, zodat een onbekend pad doorverwijst in plaats van een kale foutpagina te tonen. |
| `www/robots.txt` | `Disallow: /` — niets hier hoort geïndexeerd te worden. `index.html` draagt daarnaast `noindex` en een canonical naar `marktkraam.site`. |
| `VERSION` | Versiebron volgens de Flux76-baseline (`MAJOR.MINOR.PATCH`). |

De pagina heeft bewust geen huisstijl. Kraam heeft nog geen vastgestelde
productbrand (zie `projects/kraam/branding/README.md` in App-Guidance), en de
kleuren die rond Kraam circuleren zijn die van één tenant — die horen niet op een
platformpagina.

## Bestanden toevoegen

Zet ze in een map **onder `www/`**, bijvoorbeeld `www/mail/logo.png`, en verwijs
ernaar zonder `www` in de URL: `https://static.marktkraam.site/mail/logo.png`.
De map `www` is de documentroot en verdwijnt dus uit het pad. Raak
`www/index.html` niet aan — dat is de doorverwijzing.

Twee dingen om te onthouden bij bestanden die in e-mail gebruikt worden: een
bestand dat eenmaal is verstuurd blijft jaren opgevraagd worden, dus verplaats of
hernoem het niet meer, en `Disallow: /` in `robots.txt` houdt zoekmachines weg
maar maakt niets privé — alles wat hier staat is publiek voor wie de URL kent.

## Deploy

Dit domein draait **niet** achter Cloudflare, dus er is geen edge die iets voor
ons afhandelt. Er is niets te bouwen: wijs de documentroot naar `www/` en klaar.
Bij een deploy die de repo op de server uitcheckt, is dat de enige aanpassing —
de root van de checkout is níet de documentroot.

De serverconfiguratie moet drie dingen doen:

1. **Alleen de root doorverwijzen** — nooit de hele server. Een `301` over alles
   stuurt ook `/mail/logo.png` weg, en dan is de host nutteloos. `index.html`
   doet dit vanzelf goed omdat het één bestand op één pad is; wie liever een
   echte `301` heeft, moet die aan het pad `/` binden.
2. **Directory listing uitzetten.** Dit is de belangrijkste maatregel tegen
   rondkijken: zonder listing kun je `/mail/` niet openen om te zien wat er staat.
   De doorverwijzing op de root doet daar niets aan.
3. **`404.html` als foutpagina aanwijzen**, anders krijgt een onbekend pad de
   standaardfoutpagina van de server.

Voor nginx:

```nginx
root /var/www/static.marktkraam.site/www;         # let op: /www
autoindex off;                                    # geen directory listing
error_page 404 /404.html;

location = / {
    return 301 https://marktkraam.site/;          # alleen de root
}
```

Voor Caddy:

```caddy
static.marktkraam.site {
    root * /var/www/static.marktkraam.site/www
    redir / https://marktkraam.site/ permanent    # alleen het exacte pad /
    file_server                                   # zonder browse: geen listing
    handle_errors {
        rewrite * /404.html
        file_server
    }
}
```

Voor Apache, in de vhost:

```apache
DocumentRoot /var/www/static.marktkraam.site/www
Options -Indexes
ErrorDocument 404 /404.html
RedirectMatch permanent ^/$ https://marktkraam.site/
```

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
