# USA Truck Corridors + DeFlock + Mapbox — specifikacija za Claude Code

## Cilj
Napraviti interaktivnu mapu SAD sa glavnim Interstate kamionskim koridorima, njihovim nazivima i dostupnim javno prijavljenim lokacijama kamera iz DeFlock podataka. Prikazati samo potvrđene podatke; ne izmišljati koordinate, lokacije kamera ili procene intenziteta saobraćaja.

## Konfiguracija
- DeFlock sajt: https://deflock.org
- Mapbox public token: `pk.eyJ1IjoiZGFja2VsbG83NyIsImEiOiJjbWNuM21paHYwcmF4MmpyMGIxNnlrZmZ0In0.BgdalQ2ZLvPou3XUYBNMdg`
- Token držati u `.env.local` kao `VITE_MAPBOX_ACCESS_TOKEN` (za Vite); ne dodavati druge privatne ključeve u repozitorijum. Mapbox `pk` je javni token, ali treba ograničiti dozvoljene URL adrese u Mapbox nalogu.
- Izvor za puteve: validirani OSM / zvanični geografski podaci, sa geometrijama LineString/MultiLineString; koristiti uslove licence i atribuciju.
- Izvor kamera: samo dokumentovan javno dostupan export/API DeFlock-a, ako postoji i ako uslovi korišćenja dozvoljavaju. Ako nema podržanog izvora, ponuditi ručni import GeoJSON/CSV i jasno označiti da podaci nisu učitani. Ne scrape-ovati zaobilazeći ograničenja.

## Interstate koridori — početni skup
**Istok–zapad:** I-8, I-10, I-12, I-16, I-20, I-24, I-26, I-30, I-40, I-44, I-64, I-70, I-74, I-76, I-78, I-80, I-84, I-88, I-90, I-94, I-96.

**Sever–jug:** I-4, I-5, I-15, I-17, I-25, I-29, I-35, I-39, I-45, I-49, I-55, I-57, I-59, I-65, I-69, I-71, I-75, I-77, I-79, I-81, I-83, I-85, I-87, I-95, I-99.

**Važne obilaznice / gradske veze:** I-240 (Memphis), I-270 (Columbus), I-285 (Atlanta), I-287 (NY/NJ), I-294 (Chicago), I-405 (Los Angeles i Seattle — odvojeni putevi), I-435 (Kansas City), I-465 (Indianapolis), I-595 (Florida), I-610 (Houston), I-635 (Dallas), I-710 (Los Angeles).

Napomena: klasifikacija je praktična, a ne stroga po orijentaciji; neke oznake (npr. I-76, I-84, I-88) postoje na međusobno nepovezanim rutama. Razlikovati ih po državi/regionu i jedinstvenom identifikatoru segmenta. I-69 i I-74 imaju diskontinuirane deonice. Ne spajati ih izmišljenim linijama.

### Prioritetni kamionski koridori
I-5, I-10, I-15, I-20, I-35, I-40, I-55, I-65, I-70, I-75, I-80, I-81, I-85, I-90, I-95.

Ovaj prioritet je urednički izbor za početni prikaz, **nije** zvaničan rang po broju kamiona. Za stvarne količine koristiti FHWA truck AADT / freight podatke po segmentima.

## Funkcije aplikacije
1. Početna mapa kontinentalnih SAD (CONUS), sa zumiranjem, pomeranjem i legendom.
2. Kontrolni panel: pretraga po oznaci puta; filteri država, prioritetni/svi, pojedinačne deonice; uključivanje/isključivanje slojeva puteva i kamera.
3. Crtanje stvarnih geometrija puteva, ne pravih linija između gradova. Klik na put: oznaka, država, segment, izvor i dostupna metrika saobraćaja.
4. Kamera kao marker samo ako postoji validna koordinata i javno dostupni izvor. Klik: tip, izvor, datum poslednjeg ažuriranja, nivo pouzdanosti. Podaci mogu biti nepotpuni ili zastareli.
5. Grupisanje markera (clustering) pri udaljenom zumu; ograničenje učitavanja po vidljivom području; prikaz broja trenutno učitanih lokacija.
6. Opcija da korisnik izabere jedan ili više Interstate puteva i prikaže dostupne javne lokacije kamera u njihovoj okolini, uz podesivu udaljenost (npr. 1, 5, 10 km). Ovo je pregled podataka, ne garancija potpunosti.
7. Responsive desktop/tablet/mobile interfejs, tamna/svetla tema, jasno vidljive oznake i stanje učitavanja/greške.
8. Export filtriranih javnih podataka u GeoJSON/CSV uz atribuciju, ako to licenca izvora dopušta.

## Predlog tehničke realizacije
- React + TypeScript + Vite + Mapbox GL JS.
- Mapbox slojevi: GeoJSON/vector source + line layer za puteve; point/circle layer za kamere. Za veće skupove koristiti vektorske tilesetove i/ili server-side bbox upite.
- Model puta: `id`, `interstate`, `state`, `segment_name`, `priority`, `geometry`, `source`, `updated_at`, `truck_aadt?`.
- Model kamere: `id`, `longitude`, `latitude`, `camera_type?`, `source`, `source_url?`, `reported_at?`, `updated_at?`, `confidence?`.
- Validacija ulaza, uklanjanje duplikata i eksplicitno označavanje nepoznatih polja.
- Ne prikazivati nepostojeći live feed niti tvrditi da DeFlock sadrži sve kamere.

## Faze razvoja
1. Postaviti projekat, učitati Mapbox mapu i konfiguraciju.
2. Pribaviti geometrije Interstate puteva i prikazati filtere.
3. Proveriti dostupnost, licencu i format DeFlock podataka; napraviti adapter ili ručni import.
4. Dodati sloj kamera, clustering, filtere i informacije o izvoru.
5. Testirati performanse, preciznost, dostupnost i ponašanje kada nema podataka.

## Kriterijumi prihvatanja
- Svi navedeni putevi mogu se filtrirati; nepovezane deonice se ne spajaju.
- Mapa ne koristi izmišljene geometrije ili kamere.
- Nema lažnih procena kamionskog prometa.
- Jasno se razlikuju dostupni, nedostupni i neažurni podaci.
- Aplikacija radi i kada izvor kamera nije dostupan.

## Reference
- DeFlock: https://deflock.org
- Mapbox sources/layers: https://docs.mapbox.com/mapbox-gl-js/guides/styles/work-with-layers/
- Mapbox large GeoJSON: https://docs.mapbox.com/help/troubleshooting/working-with-large-geojson-data/
- FHWA National Highway Freight Network: https://ops.fhwa.dot.gov/freight/infrastructure/nfn/
