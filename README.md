# Feeder — Jurnal Nutritional

Aplicatie web personala pentru tracking-ul zilnic al nutritiei, masuratorilor corporale si evolutiei compozitiei corporale. Construita ca single-page app, ruleaza local pe iPhone via GitHub Pages, datele se salveaza in IndexedDB pe dispozitiv.

**URL live:** <https://bsbuzatu.github.io/Feeder/>

-----

## Stack tehnic

- **Frontend:** HTML/CSS/JS pur, un singur fisier `index.html` (~136 KB)
- **Storage:** IndexedDB (browser local, fara backend)
- **Visualizari:** Chart.js 4.4.1 (CDN)
- **Fonturi:** Fraunces (display) + Inter (body) de la Google Fonts
- **Hosting:** GitHub Pages (public repo, protejat cu parola JS hash)
- **PWA:** Add to Home Screen support pe iOS pentru persistenta IndexedDB

-----

## Acces

Aplicatia e protejata cu parola la nivel client-side:

- Parola actuala: `bzt`
- Stocata ca hash JS in cod (nu in plaintext)
- Sesiunea persista pana la inchiderea tab-ului via `sessionStorage`

-----

## Functionalitati

### Tab 1: Astazi

Dashboard principal pentru ziua curenta.

**Card calorii** (verde, mare):

- Calorii ramase pentru ziua curenta vs tinta (default 2000 kcal)
- Bara progres
- Status: deficit estimat

**6 carduri macro:**

1. **Proteine** — tinta 195g (2g/kg corp)
1. **Carbohidrati** — calculat automat (~125g)
1. **Grasimi** — tinta 80g
1. **Fibre** — tinta 35g
1. **Fier** — limita stricta 10mg/zi cu avertisment (feritina crescuta + HFE His63Asp heterozigot)
1. **Colesterol** — limita stricta 200mg/zi cu avertisment (LDL 165 + statina + factor V Leiden)

Ambele carduri (Fier, Colesterol) trec in stare “warning” cu fundal salmon cand se depaseste limita.

**5 sloturi pentru mese:**

- `shake1` — Shake post-workout (08:30)
- `pranz` — Pranz (13:00)
- `cina` — Cina (17:30)
- `shake2` — Shake seara (20:30)
- `suplimente` — Suplimente (21:00)

Fiecare slot:

- Buton `+` deschide modal de selectie aliment
- Lista alimentelor adaugate cu macro per item
- Buton “Copiaza la Cina” apare automat sub Pranz daca Pranz are alimente si Cina e goala
- Buton de stergere pe fiecare entry

### Tab 2: Masuratori

Trei sectiuni cu campuri pentru ziua selectata:

**Greutate si corp:**

- Greutate (kg)
- Talie max (cm)
- % Grasime
- Masa musculara (kg)
- Masa grasa (kg)
- Masa slaba (kg)

**Activitate zilnica:**

- Calorii arse (Apple Watch)
- Minute miscare (Apple Watch)
- Somn (ore)

**Ultimele valori** (auto-populate):

- Greutate curenta + diferenta fata de prima masuratoare
- Talie curenta + diferenta
- Masa musculara curenta + diferenta
- % Grasime curent + diferenta

### Tab 3: Alimente (lista de cumparaturi)

- 134 alimente marcate cu flag `shop: true`
- Reprezinta lista personala de cumparaturi (ce ai voie sa cumperi conform planului)
- NU include preparate ocazionale (mici, ciorba burta, tiramisu etc.) pe care le mananci dar nu le cumperi tu
- Cautare in timp real, grupare pe categorii
- Afiseaza macro per portie standard si avertismente de fier

### Tab 4: Istoric

Patru grafice Chart.js (linie, ultimele 30 zile):

- Greutate (kg)
- Talie (cm)
- Proteine zilnice (g) cu linia tinta 195g
- Calorii zilnice (kcal) cu linia tinta 2000

Medii saptamanale calculate automat sub fiecare grafic.

### Tab 5: Setari

**Tinte (editabile):**

- Greutate (kg) — folosita pentru calculul proteinei
- Ratio proteine (g/kg)
- Calorii (kcal/zi)
- Grasimi (g/zi)
- Fier max (mg/zi)
- Colesterol max (mg/zi)

**Calculate automat:**

- Proteine zilnice
- Carbohidrati zilnici
- Fibre tinta (35g fix)

**Date — backup si restore:**

- Export tot istoricul (JSON) — descarca toate datele pe device
- Importa date (JSON) — adauga doar inregistrarile noi (deduplicare automata)
- Diagnostic baza de date — afiseaza statistici per zi
- Curata duplicatele din istoric — sterge entries duplicate
- Reaplica datele seed (istoric) — forteaza reaplicarea seed-ului v7
- Sterge toate datele — reset complet (confirmare dubla)

-----

## Baza de date alimente (FOOD_DB)

**176 alimente** organizate in 18 categorii:

|Categorie         |Numar|Note                                            |
|------------------|-----|------------------------------------------------|
|Proteine animale  |19   |Carne, peste, oua                               |
|Carne procesata   |3    |Mici Lidl, carnati, sunca — consum ocazional    |
|Supe si ciorbe    |6    |De casa + ciorba burta, fasole cu ciolan        |
|Lactate           |11   |+ branza de capra                               |
|Shake             |2    |Whey, creatina                                  |
|Suplimente        |2    |Omega-3 (per softgel), Magneziu (per cp)        |
|Proteine vegetale |10   |Leguminoase uscate + fierte, tofu, tempeh       |
|Carbohidrati      |17   |Cereale, paine (inclusiv maia Prospero), cartofi|
|Tarate si fibre   |5    |Grau, ovaz, secara, psyllium                    |
|Legume            |31   |+ salata cruditati                              |
|Preparate gratar  |12   |Pui, peste, vita, legume                        |
|Fructe recomandate|16   |Berries, mere, citrice                          |
|Fructe de vara    |8    |Cirese, pepene, caise etc.                      |
|Grasimi           |17   |Ulei, nuci, seminte, unt                        |
|Ciocolata neagra  |4    |70%, 80%, 85%, 90%                              |
|Deserturi         |1    |Tiramisu de casa                                |
|Bauturi           |5    |Cafea neagra, ceai, apa, vin rosu               |
|Condimente        |3    |Otet mere, mustar, lamaie                       |

Fiecare aliment are: `kcal, p (proteine), c (carbo), f (grasimi), fiber, iron, chol`. Unitatile sunt: `g` (default), `ml`, `buc`, `softgel`, `cp`, `portie`.

**Avertismente automate** pe alimente cu probleme pentru profilul utilizatorului:

- Fier mare/MARE pe spanac, cacao, tahini, seminte dovleac, vita, sardine etc.
- Grasimi saturate pe cascaval
- Ocazional pe carne procesata, prajeli, deserturi

-----

## Profilul medical care defineste limitele

**Bogdan-Sebastian Buzatu, 44 ani, M, 95.1 kg, 108cm talie**

Constrangeri principale:

- **Fier max 10 mg/zi** — feritina 368 + HFE His63Asp heterozigot (predispozitie hemocromatoza)
- **Colesterol max 200 mg/zi** — LDL 165 + statina (Coltowan) + ezetimibe + berberina
- **Factor V Leiden heterozigot** — hidratare 3L+/zi, evita imobilizare prelungita
- **Vitamina D 16.89** — deficit, suplimentare 4000 UI/zi
- **HbA1c 5.61** — pre-diabet, deficit caloric ~500 kcal/zi

Targets calculate:

- Greutate-pierdere: ~10kg in 6 luni (talie sub 94cm)
- Deficit: ~500 kcal/zi (2000 vs TDEE ~2500)
- Proteine: 2g/kg (195g/zi) pentru pastrarea masei musculare in deficit
- Antrenament: 4x ridicari + 2x cardio Zone 2

-----

## Seed data (istoric incarcat automat)

Aplicatia contine in cod 55 entries istorice (27-30 mai 2026) + 3 masuratori, care se aplica automat la prima deschidere:

|Data      |Entries|Note                                                  |
|----------|-------|------------------------------------------------------|
|2026-05-27|15     |Pui 660g total (zi cu fier mare)                      |
|2026-05-28|14     |3 oua seara (zi cu colesterol mare)                   |
|2026-05-29|16     |Supa pui cu legume radacinoase, banana + nuci la pranz|
|2026-05-30|10     |Omleta 3 oua, pranz mixt cu fructe, suplimente        |

Logica de aplicare:

- Flag `seedApplied` cu versiunea curenta (v7) in `settings`
- Dublu sistem de deduplicare:
1. **Cheie completa**: `date|meal|foodId|qty|timestamp` — prinde duplicate exacte
1. **Cheie structurala**: `date|meal|foodId|qty` — limiteaza numarul de instante identice in DB la cat are seed-ul (preveine dublarea la bumpari de versiune)

-----

## Probleme cunoscute si solutii

### IndexedDB efemeritate pe iOS Safari

Safari pe iOS sterge IndexedDB-ul site-urilor dupa ~7 zile inactivitate sau cand storage-ul e plin. Singura solutie permanenta:

- **Adauga la Home Screen** (Share -> Add to Home Screen)
- Acceseaza aplicatia DOAR din icoana de Home Screen, niciodata din Safari direct
- Datele Safari + datele PWA sunt sandboxuri separate

### Cache HTTP pe GitHub Pages

Fisierul are meta tags `Cache-Control: no-store, no-cache, must-revalidate` ca sa forteze browser-ul sa ia mereu versiunea proaspata.

Daca dupa upload pe GitHub aplicatia tot afiseaza versiunea veche:

- Hard refresh (kill PWA app + redeschide)
- Sau Settings -> Safari -> Clear History (sterge si datele!)

### Backup discipline

**Recomandare:** Setari -> Exporta tot istoricul (JSON) la fiecare 2-3 zile. Salveaza in Files pe iPhone. In caz de pierdere date, reimporti.

-----

## Versioning seed

|Versiune|Schimbari                                        |
|--------|-------------------------------------------------|
|v1      |Seed initial cu 15 entries (27 mai)              |
|v2      |Adaugat 28 mai (14 entries)                      |
|v3      |Adaugat 29 mai, supa pui + banana/nuci           |
|v4      |Bug fix deduplicare seed                         |
|v5      |Adaugat 30 mai (10 entries)                      |
|v6      |Bumpat pentru forta reaplicare                   |
|v7      |Adaugat camp `chol` (colesterol) la toate entries|

-----

## Conventii de cod

- **Toate textele UI in romana fara diacritice** (a in loc de a, s in loc de s, etc.)
- **Em-dashes inlocuite cu hyphens** (—  ->  -)
- Macronutrienti per 100g/100ml pentru g/ml; per piesa pentru `buc`, `softgel`, `cp`, `portie`
- Logica `isPerPiece = unit !== 'g' && unit !== 'ml'` pentru calcul multiplicator

-----

## Dezvoltare

Modificarile se fac direct prin GitHub web editor (nu exista build pipeline). Fluxul de lucru:

1. Edit `index.html` in browser pe github.com/bsbuzatu/Feeder
1. Commit (deploy GitHub Pages automat in 1-2 min)
1. Pe iPhone: kill app + redeschide din Home Screen
1. Daca s-a schimbat versiunea seed, seed-ul se reaplica automat la deschidere

-----

## Date sensibile

Repository-ul este public pe GitHub. Codul (HTML + JS) e vizibil. **Datele tale personale (mese, masuratori) sunt salvate doar pe device-ul tau in IndexedDB**, nu pe server. Singurul mod prin care altcineva poate accesa aplicatia este sa ghiceasca parola `bzt` din hash JS (banal de spart, dar suficient ca filtru pentru randomi).

Daca vreodata vrei sa migrezi la backend real (sincronizare intre device-uri, cloud backup):

- Supabase free tier — 500MB DB, autentificare inclusa, ~30 min setup
- Sau Firestore — gratuit pentru uz personal