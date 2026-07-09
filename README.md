# Feeder — Jurnal de antrenament

Aplicație PWA single-file pentru înregistrarea antrenamentelor de forță și urmărirea compoziției corporale.
Toate datele rămân local, în IndexedDB. Zero servere, zero terți, zero telemetrie.

**Live:** `https://bsbuzatu.github.io/Feeder/`
**Build curent:** `2026-07-09-F` (afișat permanent în subtitlul din header)

---

## Arhitectură

| Componentă | Detaliu |
|---|---|
| Fișiere | Un singur `index.html` (~46 KB), fără dependențe externe |
| Stocare | IndexedDB — `bogdan_nutrition_tracker`, `DB_VERSION = 3` |
| Object stores | `workouts`, `metrics`, `settings`, `entries` (legacy, păstrat pentru export) |
| Hosting | GitHub Pages |
| Acces | PWA, adăugat pe Home Screen (necesar pentru persistența IndexedDB pe iOS) |
| Autentificare | Cod local, fără rețea |

Datele sunt legate de **origine**. Deschiderea `index.html` din Files sau de pe disc (`file://`)
rulează pe altă origine și nu vede baza de date. Este comportament normal, nu defect.

---

## Ecrane

### Gym
- Patru sesiuni Upper/Lower. Deschiderea unei zile intră în **mod consultare** — nimic nu se
  înregistrează, cronometrul nu pornește.
- **Start antrenament** activează logarea, cronometrul și Screen Wake Lock.
- Casetele sunt **precompletate** cu valorile ultimei sesiuni, set cu set, afișate în gri.
  `✓` confirmă valoarea ca atare. Scrisul peste o suprascrie.
- Se salvează **doar seturile bifate**. O valoare precompletată neconfirmată nu ajunge în baza de date.
- Timer de pauză, fără sunet și fără notificări.
- Pagina nu derulează automat după bifarea unui set.
- Banner roșu dacă store-ul `workouts` este gol.

### Stats
Input: greutate (kg), mușchi (kg), grăsime (kg), circumferință abdominală (cm).
Derivate automat: procent grăsime, masă slabă.
Dashboard: KPI cu delta față de măsurătoarea anterioară, grafic compoziție, grafic circumferință,
grafic volum (seturi/săptămână), tabel istoric.

`saveMetricsForDay()` face merge, nu suprascriere: câmpurile existente ale zilei (ex. `trainingDay`)
sunt păstrate la actualizări parțiale.

### Alimente
Listă statică, filtrată pentru creștere de masă slabă și pierdere de grăsime viscerală.
Opt categorii, inclusiv o secțiune **Interzis** (grapefruit — inhibitor CYP3A4, relevant sub atorvastatină).

### Setări
Export JSON, Import JSON, diagnostic (build, număr de antrenamente, măsurători, exerciții cu istoric mapat),
ștergere totală.

---

## Program

Upper/Lower, 4 zile. Zone 2 cardio 2–3×/săpt, **înregistrat pe ceas (Apple Health), nu în aplicație.**

### Monday — Upper A · Horizontal Push + Pull

| # | Exercise | Sets | Reps | RIR | Rest |
|---|---|---|---|---|---|
| 1 | Barbell Bench Press | 4 | 5–7 | 2 | 3 min |
| 2 | Weighted Pull-Up | 4 | 6–8 | 2 | 2.5 min |
| 3 | Standing Barbell OHP | 3 | 6–8 | 2 | 2.5 min |
| 4 | Single-Arm DB Row | 3 | 8–12 /arm | 1–2 | 90 s |
| 5 | Cable Lateral Raise (one arm) | 4 | 12–20 | 0–1 | 60 s |
| 6 | Cable Reverse Fly | 3 | 12–20 | 0–1 | 60 s |
| 7 | Incline DB Curl | 3 | 8–12 | 0–1 | 90 s |
| 8 | Overhead Cable Triceps Extension | 2 | 10–12 | 0–1 | 60 s |

### Tuesday — Lower A · Squat Emphasis

| # | Exercise | Sets | Reps | RIR | Rest |
|---|---|---|---|---|---|
| 1 | Barbell Back Squat | 4 | 5–7 | 2 | 3 min |
| 2 | Romanian Deadlift | 3 | 8–10 | 2 | 2.5 min |
| 3 | Leg Press | 3 | 10–12 | 1 | 2 min |
| 4 | Seated Leg Curl | 3 | 10–15 | 0–1 | 90 s |
| 5 | Weighted One-Leg Standing Calf Raise | 4 | 10–15 /leg | 0–1 | 90 s |
| 6 | Machine Ab Crunch | 3 | 10–15 | 0–1 | 60 s |

### Thursday — Upper B · Incline / Vertical

| # | Exercise | Sets | Reps | RIR | Rest |
|---|---|---|---|---|---|
| 1 | Incline DB Press (30°) | 4 | 8–10 | 2 | 2.5 min |
| 2 | Chest-Supported DB Row | 4 | 8–12 | 1–2 | 2 min |
| 3 | Cable Chest Fly | 3 | 12–15 | 0–1 | 90 s |
| 4 | Close-Grip Lat Pulldown | 3 | 10–12 | 1 | 2 min |
| 5 | DB Lateral Raise | 4 | 12–20 | 0–1 | 60 s |
| 6 | Face Pull | 3 | 15–20 | 0–1 | 60 s |
| 7 | Weighted Triceps Dip | 3 | 8–12 | 1–2 | 2 min |
| 8 | Hammer Curl | 3 | 10–12 | 0–1 | 60 s |

### Friday — Lower B · Hinge / Unilateral

| # | Exercise | Sets | Reps | RIR | Rest |
|---|---|---|---|---|---|
| 1 | Barbell Back Squat | 4 | 8–10 | 2 | 3 min |
| 2 | Conventional Deadlift | 4 | 3 | 2–3 | 4 min |
| 3 | Bulgarian Split Squat | 3 | 8–10 /leg | 1–2 | 2 min |
| 4 | Lying Leg Curl | 3 | 10–15 | 0–1 | 90 s |
| 5 | Seated Calf Raise (2s pause) | 4 | 12–15 | 0–1 | 60 s |
| 6 | Pallof Press | 3 | 10–12 /side | — | 45 s |

### Volum săptămânal

| Grupă | Seturi | Fereastră |
|---|---|---|
| Piept | 11 | 10–20 |
| Spate | 14 | 10–20 |
| Deltoid lateral | 8 | 8–12 |
| Deltoid posterior | 6 | 6–10 |
| Biceps (direct) | 6 | + indirect |
| Triceps (direct) | 5 | + indirect |
| Cvadriceps | 14 | 10–20 |
| Ischiogambieri | 11 | 10–20 |
| Gambe | 8 | — |
| Core | 6 | — |
| **Încărcare axială grea** | **15** | prag de monitorizare |

Total: **94 seturi/săptămână**, frecvență 2×/grupă.

---

## Reguli de logare

1. **Nu se loghează seturile de încălzire.** Doar seturile de lucru. Altfel precompletarea,
   dubla progresie și volumul din Stats devin false.
2. **Progresie dublă**, o dată pe săptămână, pe exercițiu:

| Situație la ultimul set | Acțiune |
|---|---|
| Capătul de sus al intervalului atins la toate seturile | +2.5 kg (bară) / +1–2 kg (ganteră, cablu), revii la capătul de jos |
| În interval | Aceeași greutate, +1 repetare pe cât de multe seturi poți |
| Sub capătul de jos la ultimul set | Aceeași greutate, repeți săptămâna |

Întâi repetările până la plafon, apoi greutatea. Niciodată ambele simultan.

3. **Deadlift 4×3:** greutatea aleasă astfel încât 3 repetări să lase RIR 2–3. Dead stop între
   repetări, nu touch-and-go. Dacă a treia repetare arată diferit de prima, setul se oprește.

---

## Zone 2 cardio

Nu se înregistrează în aplicație.

| Parametru | Valoare |
|---|---|
| Frecvență | 2–3×/săptămână |
| Durată | 30–45 min |
| Modalitate | Bicicletă staționară sau mers înclinat. Nu alergare. |
| Interval FC (Karvonen, HRmax est. 177, RHR 60) | 130–142 bpm |
| Validare | Talk test: propoziții complete, respirație nazală susținută |
| Zile | Miercuri, sâmbătă. Niciodată înainte de Lower. |
| Plafon | ≤150 min/săptămână |

---

## Backup și restaurare

**Export înainte de fiecare deploy.** Fără excepții. Setări → Export JSON. ~120 KB.

Formatul exportului (v3):

```
{
  exportedAt, version: 3, app, user,
  settings, exerciseCatalog, program,
  workouts[], metrics[], entries[]
}
```

Importul este **deduplicat** (cheie `date|sessionId`) și **nedistructiv** — valorile existente au
prioritate, cele lipsă se completează din backup. Rularea de două ori nu creează dubluri.

---

## Cache — procedura corectă de deploy

GitHub Pages servește HTML prin CDN (~10 min TTL), iar Safari cachează agresiv paginile PWA.

| Metodă | Efect asupra IndexedDB |
|---|---|
| Deschide `.../Feeder/?v=N` cu N incrementat | **Sigur.** Query string nou = fișier nou. |
| Șterge icoana de pe Home Screen și readaug-o | **Sigur.** |
| Safari → Advanced → Website Data → șterge `github.io` | **Șterge datele.** Evită. |
| Settings → Safari → Clear History and Website Data | **Șterge tot.** Niciodată. |

Procedura:

1. Export JSON.
2. Urcă `index.html`. Așteaptă ~1 minut.
3. Deschide `bsbuzatu.github.io/Feeder/?v=N+1` în Safari.
4. Verifică build-ul în subtitlul de sub „Gym".
5. Setări → verifică `Antrenamente: N`.
6. Add to Home Screen.

---

## Mapare istoric

Exercițiile sunt identificate prin `exId` canonic. Numele vechi sunt rezolvate printr-un tabel de
aliasuri, astfel încât redenumirile nu pierd istoricul.

| exId | Nume canonic | Aliasuri recunoscute |
|---|---|---|
| `bbSquat` | Barbell Back Squat | Back Squat (high-bar, sub paralel / below parallel), Genuflexiuni |
| `pullup` | Weighted Pull-Up | Lat Pulldown / Tractiuni, Lat Pulldown / Pull-Up |
| `triDip` | Weighted Triceps Dip | Weighted Dip (4 variante) |
| `facePull` | Face Pull | Face Pull / Reverse Fly |
| `oneLegCalf` | Weighted One-Leg Standing Calf Raise | Standing Calf Raise, Ridicari pe varfuri |
| `ohTri` | Overhead Cable Triceps Extension | Overhead Triceps Extension |
| `sarow` | Single-Arm DB Row | Ramat cu haltera |
| `csRow` | Chest-Supported DB Row | Ramat la cablu |
| … | | vezi obiectul `EX` din `index.html` |

Exerciții prezente în istoric dar scoase din program (`Hanging Knee Raise`, `EZ-Bar Skull Crusher`,
`Barbell Hip Thrust`, `Cable Crunch`): datele rămân în IndexedDB și în export, dar nu se afișează.

`Machine Ab Crunch` nu este aliasat la `Cable Crunch` — încărcările nu sunt comparabile, iar o
„ultimă valoare" falsă e mai rea decât absența ei.

---

## Convenții de cod

- **Fără diacritice în `index.html`.** Bug recurent. Textele vizibile folosesc forme fără diacritice.
- Fără `scrollIntoView` nicăieri.
- Bifarea unui set mută DOM-ul direct, fără re-render — poziția paginii și focus-ul rămân intacte.
- `saveMetricsForDay()` face merge; nu suprascrie câmpuri existente.
- Constanta `BUILD` se incrementează la fiecare modificare și este afișată în UI.

---

## Istoric build-uri

| Build | Modificări |
|---|---|
| `2026-07-09-F` | Banner „baza de date goala"; build vizibil imediat la pornire |
| `2026-07-09-E` | Meta tags `no-cache`; build afișat în subtitlu; cod mort eliminat |
| `2026-07-09-D` | Precompletarea casetelor cu valorile ultimei sesiuni; caseta de istoric eliminată; se salvează doar seturile bifate |
| `2026-07-09-C` | Deadlift 4×3; card „Split" eliminat; import deduplicat și nedistructiv |
| `2026-07-09-B` | Buton Start; mod consultare separat de sesiunea activă |
| `2026-07-09-A` | Rescriere: modul nutriție eliminat, program nou, Stats, Alimente, Export |
