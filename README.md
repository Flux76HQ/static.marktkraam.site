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
| `_redirects` | Serverzijdige `301` naar `https://marktkraam.site/:splat`. **Werkt alleen op Cloudflare Pages en Netlify**; op een eigen nginx/Traefik-origin is het een ongebruikt bestand en doet `index.html` het werk. |
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

De hostingkeuze voor dit subdomein staat nog niet vast. Twee routes, met wat elk
betekent voor de doorverwijzing:

- **Cloudflare Pages / Netlify** — `_redirects` levert een echte `301`; de browser
  komt nooit op `index.html`. Niets extra's in te richten.
- **Eigen origin (Traefik op de gedeelde host, zie `shared/infrastructure/adding-an-app.md`)** —
  `_redirects` doet niets. Wil je hier ook een `301` in plaats van een pagina die
  zichzelf doorstuurt, dan hoort die regel bij de reverse proxy of in een Cloudflare
  Redirect Rule, niet in deze repo.

Beide werken; de pagina-doorverwijzing is alleen iets trager en laat de bezoeker
kort een tussenpagina zien.

## Grenzen buiten deze repository

Deze site serveert alleen statische bestanden en accepteert geen uploads of
POST-verkeer, dus er is hier geen request-limiet om te documenteren. Komt er later
wél een formulier of upload bij, dan hoort de maximale requestgrootte hier als
getal genoemd te worden, met de aantekening dat alles wat ervóór staat (CDN,
reverse proxy) minstens diezelfde omvang moet doorlaten — zie
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
