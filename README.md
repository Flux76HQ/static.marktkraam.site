# static.marktkraam.site

Statische site op `static.marktkraam.site`. Er staat **nog geen inhoud**: de root
verwijst voorlopig door naar <https://marktkraam.site>, zodat bezoekers hier niet
zomaar terechtkomen zolang de pagina in aanbouw is.

## Wat er nu staat

| Bestand | Doel |
|---|---|
| `index.html` | Doorverwijzing naar `https://marktkraam.site/` — `location.replace()` plus een `meta refresh` als JavaScript uitstaat, met een zichtbare link als beide falen. |
| `404.html` | Identiek aan `index.html`, zodat ook `/wat-dan-ook` doorverwijst in plaats van een foutpagina te tonen. |
| `robots.txt` | `Disallow: /` — de doorverwijspagina hoort niet geïndexeerd te worden. `index.html` draagt daarnaast `noindex` en een canonical naar `marktkraam.site`. |
| `VERSION` | Versiebron volgens de Flux76-baseline (`MAJOR.MINOR.PATCH`). |

De pagina heeft bewust geen huisstijl. Kraam heeft nog geen vastgestelde
productbrand (zie `projects/kraam/branding/README.md` in App-Guidance), en de
kleuren die rond Kraam circuleren zijn die van één tenant — die horen niet op een
platformpagina.

## Lokaal bekijken

```sh
python3 -m http.server 8080
# open http://localhost:8080 — je hoort direct op marktkraam.site te belanden
```

Test de fallback door JavaScript uit te zetten: de `meta refresh` neemt het dan over.

## Deploy

Dit domein draait **niet** achter Cloudflare, dus er is geen edge die de
doorverwijzing voor ons kan afhandelen: `index.html` doet het volledige werk.
Kopieer de bestanden naar de documentroot van de webserver die
`static.marktkraam.site` serveert en je bent klaar. Er is niets te bouwen.

Twee dingen zijn wel afhankelijk van de serverconfiguratie:

- **`404.html` wordt alleen gebruikt als je de server dat vertelt.** Zonder die
  regel krijgt een bezoeker op `/wat-dan-ook` de standaard foutpagina in plaats
  van de doorverwijzing.
- **Een echte `301`** is sneller en netter dan een pagina die zichzelf doorstuurt,
  maar hoort in de serverconfiguratie thuis, niet in deze repo. Doe je dat, dan
  worden `index.html` en `404.html` overbodig — laat ze staan als vangnet voor het
  geval de regel ooit wegvalt.

Voor nginx:

```nginx
# vangnet-variant: serveer de statische bestanden
error_page 404 /404.html;

# of, netter: laat de server zelf doorverwijzen
return 301 https://marktkraam.site$request_uri;
```

Voor Caddy:

```caddy
static.marktkraam.site {
    redir https://marktkraam.site{uri} permanent
}
```

Voor Apache, in `.htaccess` of de vhost:

```apache
ErrorDocument 404 /404.html

# of, netter:
Redirect permanent / https://marktkraam.site/
```

Kies één van de twee per server — een `return 301` naast de statische bestanden
betekent dat de bestanden nooit geserveerd worden.

## Grenzen buiten deze repository

Deze site serveert alleen statische bestanden en accepteert geen uploads of
POST-verkeer, dus er is hier geen request-limiet om te documenteren. Komt er later
wél een formulier of upload bij, dan hoort de maximale requestgrootte hier als
getal genoemd te worden, met de aantekening dat alles wat ervóór staat
(webserver, reverse proxy) minstens diezelfde omvang moet doorlaten — zie
`shared/guidelines/project-baseline.md` §6 in App-Guidance.

## Als de echte pagina er komt

Vervang `index.html` door de inhoud en verwijder de doorverwijzing bewust in één
commit: `robots.txt`, de `noindex`-meta en de canonical horen dan mee te veranderen,
anders staat de nieuwe pagina live maar blijft hij onvindbaar. `404.html` mag dan
een echte foutpagina worden.

## Baseline

Deze repo volgt de Flux76-baseline uit
[`Flux76HQ/App-Guidance`](https://github.com/Flux76HQ/App-Guidance). Nog niet
overgenomen, omdat er nog geen build- of deploystraat is:
`scripts/bump_version.py`, `scripts/check_version_bump.py`, `.githooks/pre-commit`
en de CI-versieguard. Die horen erbij zodra hier meer staat dan een doorverwijzing.
