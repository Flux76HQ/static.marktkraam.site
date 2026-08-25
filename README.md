# static.marktkraam.site

Statische host voor losse bestanden bij Marktkraam: afbeeldingen in e-mails,
demobestanden en vergelijkbaar materiaal dat via een vaste URL bereikbaar moet
zijn.

Er hoort **geen site** te staan. De root verwijst door naar
<https://marktkraam.site>, zodat wie het domein uit een afbeeldings-URL plukt en
er zelf naartoe surft, niet gaat rondkijken maar op de echte site belandt.

## Wat er nu staat

| Bestand | Doel |
|---|---|
| `index.html` | Doorverwijzing naar `https://marktkraam.site/` — `location.replace()` plus een `meta refresh` als JavaScript uitstaat, met een zichtbare link als beide falen. |
| `404.html` | Identiek aan `index.html`, zodat een onbekend pad doorverwijst in plaats van een kale foutpagina te tonen. |
| `robots.txt` | `Disallow: /` — niets hier hoort geïndexeerd te worden. `index.html` draagt daarnaast `noindex` en een canonical naar `marktkraam.site`. |
| `VERSION` | Versiebron volgens de Flux76-baseline (`MAJOR.MINOR.PATCH`). |

De pagina heeft bewust geen huisstijl. Kraam heeft nog geen vastgestelde
productbrand (zie `projects/kraam/branding/README.md` in App-Guidance), en de
kleuren die rond Kraam circuleren zijn die van één tenant — die horen niet op een
platformpagina.

## Bestanden toevoegen

Zet ze in een map onder de root, bijvoorbeeld `mail/logo.png` of `demo/`, en
verwijs ernaar met de volle URL: `https://static.marktkraam.site/mail/logo.png`.
Raak `index.html` niet aan — dat is de doorverwijzing.

Twee dingen om te onthouden bij bestanden die in e-mail gebruikt worden: een
bestand dat eenmaal is verstuurd blijft jaren opgevraagd worden, dus verplaats of
hernoem het niet meer, en `Disallow: /` in `robots.txt` houdt zoekmachines weg
maar maakt niets privé — alles wat hier staat is publiek voor wie de URL kent.

## Deploy

Dit domein draait **niet** achter Cloudflare, dus er is geen edge die iets voor
ons afhandelt. Kopieer de bestanden naar de documentroot van de webserver die
`static.marktkraam.site` serveert; er is niets te bouwen.

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
root /var/www/static.marktkraam.site;
autoindex off;                                    # geen directory listing
error_page 404 /404.html;

location = / {
    return 301 https://marktkraam.site/;          # alleen de root
}
```

Voor Caddy:

```caddy
static.marktkraam.site {
    root * /var/www/static.marktkraam.site
    redir / https://marktkraam.site/ permanent    # alleen het exacte pad /
    file_server                                   # zonder browse: geen listing
    handle_errors {
        rewrite * /404.html
        file_server
    }
}
```

Voor Apache, in de vhost of `.htaccess`:

```apache
Options -Indexes
ErrorDocument 404 /404.html
RedirectMatch permanent ^/$ https://marktkraam.site/
```

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
