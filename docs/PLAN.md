# PRAXISTÉRKÉP — Hol a fogorvos? Hol a háziorvos?

**Interaktív magyar alapellátási körzet- és betöltetlenség-térkép**
Tervezési dokumentum · v0.1 · 2026-08-22

Vizualizációs minta: [holadelej.hu](https://holadelej.hu) — egyoldalas, görgethető, sötét témájú, narratív adat-dashboard, laikusnak is érthető magyarázatokkal, "mennyi ez valójában?" kontextualizálással, precíz forrásmegjelöléssel, HU/EN nyelvi váltóval.

---

## 1. Vízió és cél

Egy nyilvános webapp, ami **havonta frissülő NEAK-adatokból** megmutatja:

- hol vannak **betöltetlen háziorvosi és fogorvosi körzetek** Magyarországon,
- **mióta** betöltetlenek, és **hány embert** érint,
- hogyan **változik a trend** hónapról hónapra és évről évre,
- milyen messze van a legközelebbi **betöltött** praxis egy adott településtől,
- mindezt település / járás / vármegye / országos szinten, interaktív térképen.

**Célközönség (3 réteg):**
1. **Lakosság** — "Mi a helyzet az én településemen?" (kereső + körzetkártya)
2. **Sajtó / döntéshozók** — megosztható, hivatkozható számok és térképek (a 40 oldalas problématérkép "élő" változata)
3. **Szakma** — praxisjogot kereső orvosok/fogorvosok: hol van üres körzet, mekkora lakossággal, mióta.

**Pozicionálás:** hivatalos, nyilvános adatokból építkező, pártatlan, forrásait pontosan megjelölő eszköz — a holadelej.hu etikája: "az adat tájékoztató jellegű, ez a forrása, így értelmezd".

---

## 2. Mit tanulunk a holadelej.hu-tól (design-elvek)

| Elv | Alkalmazás nálunk |
|---|---|
| Egyoldalas, szekciókra tagolt scrollytelling, horgony-navigációval | `#terkep` `#fogorvos` `#haziorvos` `#trend` `#nalam` `#rangsor` `#forrasok` |
| "Most" számok nagy tipóval, változásjelzéssel | "Betöltetlen fogorvosi körzet **most: N** · ▲+3 az előző hónaphoz képest" |
| Laikus magyarázat minden metrika mellett | pl. mi az a "tartósan betöltetlen" (6 hónap, 313/2011. Korm. r.), miért tér el a KSH és a NEAK száma |
| "Mennyi ez valójában?" átszámítások | "Ennyi ember él ellátatlan fogorvosi körzetben: **X fő** — ez kb. Y darab Szombathely" |
| Mértéktartó, sötét, adatközpontú vizuál | sötét alaptéma, 2 fő szín (fogorvos / háziorvos), színvak-biztos skálák |
| Forrás-lábléc CC-licencekkel, frissítési idővel | NEAK, OKFŐ, KSH, OSM — minden adat mellett "utolsó frissítés: YYYY-MM" |
| Előrejelzés/becslés vizuálisan elkülönítve a ténytől | interpolált/becsült adat szaggatott vagy halvány |

---

## 3. Adatforrások

### 3.1 Elsődleges (havi frissítésű, NEAK)

| # | Forrás | Formátum | Tartalom | Kulcs |
|---|---|---|---|---|
| A | NEAK **Betöltetlen fogorvosi szolgálatok** (letölthető, havi) | PDF (táblázatos) | FIN kód, vármegye, telephely(ek) címe, betöltetlenség kezdete, ellátandó lakosságszám, szolgálattípus (felnőtt/gyermek/vegyes/iskolai) | FIN kód |
| B | NEAK **Betöltetlen háziorvosi szolgálatok** (letölthető, havi) | PDF | ugyanez háziorvosi (felnőtt/gyermek/vegyes) bontásban | HSZ kód |
| C | NEAK **szerződött szolgáltatók / háziorvosi és fogorvosi szolgálatok törzsadatai** | XLS/PDF | az összes (betöltött + betöltetlen) szolgálat listája — a nevező az arányszámokhoz | FIN/HSZ kód |
| D | NEAK **általános finanszírozási havi adatok** (kasszánként) | XLS | háziorvosi kártyaszámok (bejelentkezett TAJ), fogászati teljesítmény (pontszám, esetszám), díjazás | szolgálatkód |

### 3.2 Másodlagos

| # | Forrás | Frissülés | Tartalom |
|---|---|---|---|
| E | OKFŐ Alapellátási Igazgatóság — **tartósan betöltetlen körzetek** listái (háziorvosi + fogorvosi) | folyamatos | jogi értelemben tartósan betöltetlen (≥6 hónapja megszűnt szerződés) körzetek; keresztellenőrzés az A/B listákkal |
| F | KSH — települési lakónépesség, korösszetétel | évi | fajlagos mutatók nevezője (10 000 főre jutó praxis, ellátatlan lakosság aránya) |
| G | Közigazgatási határok: OSM / KSH településhatár-geometriák (GeoJSON) | ritkán | térképi alapréteg (település → járás → vármegye aggregáció) |
| H | Geokódolás: telephelycímek → koordináta (Nominatim/OSM, cache-elve) | egyszeri + delta | pontszerű megjelenítés, elérhetőség-számítás |
| I | Saját ügyeleti adatbázis (a Kapolka-féle fogászati ügyeleti dokumentum adattáblája) | kézi, negyedéves | fogászati ügyeleti pontok rétege |

### 3.3 Ismert adatkorlátok (kötelezően kommunikálandó a UI-ban)

- A NEAK-listák **"tájékoztató jellegűek"**, a havi finanszírozási szerződésadatokon alapulnak — nem valós idejű működési állapot.
- **Körzethatár-poligonok nincsenek egységesen publikálva** → a térkép települési szinten aggregál, a körzet pontszerűen (telephely) jelenik meg. Ez tudatos, vállalt egyszerűsítés, magyarázattal.
- **KSH ≠ NEAK**: az orvosszám és a praxisszám két külön nyilvántartás; a különbség kb. a betöltetlen praxisok száma — ezt egy külön magyarázó blokk mutatja be (Kincses-tanulmány hivatkozással).
- Helyettesítéssel ellátott betöltetlen körzet ≠ ellátatlan lakosság — a "betöltetlen" és az "ellátatlan" fogalmát a UI szigorúan szétválasztja.
- Praxisjogosok életkora praxisszinten nem nyilvános → korfa csak aggregált (tanulmány-alapú) szinten, becslésként jelölve.

---

## 4. Mutatók (a "összes létező statisztika")

### 4.1 Állapotmutatók (adott hónapra)

| Mutató | Szint | Forrás |
|---|---|---|
| Betöltetlen körzetek száma és aránya (%) — fogorvosi / háziorvosi, típusbontásban (felnőtt · gyermek · vegyes · iskolai) | ország / vármegye / járás / település | A+B+C |
| Betöltetlenség hossza (hónap) — eloszlás, medián, "top 20 legrégebb óta üres" | körzet | A+B |
| Tartósan betöltetlen (jogi kategória) vs. frissen megüresedett | körzet | A+B+E |
| Ellátandó lakosság betöltetlen körzetben (fő, és a lakosság %-a) | minden szint | A+B+F |
| 10 000 főre jutó működő praxis | járás / vármegye | C+F |
| Kártyaszám / praxis (háziorvosi) — túlterheltség-proxy | körzet / járás | D |
| Fogászati teljesítmény (pont, eset) / praxis | körzet / járás | D |
| Elérhetőség: légvonalbeli / közúti távolság a legközelebbi **betöltött** azonos típusú praxistól | település | C+G+H |
| "Fehér foltok": települések, ahonnan X km-en belül nincs betöltött fogorvosi praxis | település | származtatott |
| Fogászati ügyeleti lefedettség (legközelebbi ügyeleti pont távolsága) | település | I |

### 4.2 Trendmutatók (idősor, minden hónap archiválva)

- Betöltetlen körzetek számának alakulása havonta (2010-ig visszavezethető az OKFŐ/NEAK archívumokból, ahol elérhető; különben a saját gyűjtés kezdetétől).
- Nettó változás: megüresedett vs. betöltött körzetek havonta ("ki-be áramlás").
- Vármegyei rangsor változása (bump chart).
- Előrejelzés **nincs** (v1-ben) — csak tény és trend; ha később lesz, vizuálisan elkülönítve.

### 4.3 Kontextualizáló blokkok ("Mennyi ez valójában?")

- Ellátatlan körzetben élők száma = hány Szombathely / hány Vas vármegye.
- A leghosszabb betöltetlenség = "ez a körzet azóta üres, hogy…" (idővonal-horgony).
- Fogorvosi vs. háziorvosi betöltetlenségi arány egymás mellett — a fogászat relatív helyzetének megmutatása.

---

## 5. Funkciók és képernyő-szekciók (egyoldalas app)

1. **Hero + "most" számok** — 4-6 nagy szám (betöltetlen fogorvosi, háziorvosi, érintett lakosság, havi változás), utolsó frissítés dátuma.
2. **Országtérkép** (`#terkep`) — MapLibre GL; váltók: *fogorvosi | háziorvosi*, *összes | felnőtt | gyermek | vegyes | iskolai*, metrika-választó (betöltetlenségi arány / lakosság / elérhetőség). Choropleth járás/vármegye szinten + pont-réteg a betöltetlen telephelyekre. Hover-kártya, kattintásra részletpanel.
3. **"Mi a helyzet nálam?"** (`#nalam`) — településkereső; eredmény: a település körzetei, státuszuk, mióta, legközelebbi betöltött praxis és ügyelet távolsága. Megosztható URL (`/telepules/szombathely`).
4. **Trend** (`#trend`) — idősor-grafikonok (D3), ki-be áramlás, típusbontás; csúszkával "időutazás" a térképen (hónap-slider).
5. **Rangsorok** (`#rangsor`) — top/bottom járások, leghosszabb betöltetlenségek, legnagyobb érintett lakosság.
6. **Magyarázatok** — mi a praxisjog, mi a tartósan betöltetlen, KSH vs. NEAK, mit jelent a helyettesítés. (A holadelej "Paks · közelről" szekció mintájára: egy jól megírt, laikus mélymagyarázó blokk.)
7. **Ügyeleti réteg** (`#ugyelet`, v2) — fogászati ügyeleti pontok + lefedettség.
8. **Forrás-lábléc** — minden forrás, licenc, frissítési gyakoriság, módszertani jegyzet linkje; "az adatok tájékoztató jellegűek" disclaimer.
9. **HU/EN** nyelvi váltó (EN v2).

---

## 6. Architektúra

**Elv: statikus frontend + havi batch ETL. Nincs futó backend, nincs adatbázis-szerver üzemeltetés.**

```
┌─ GitHub Actions (cron: havi 2×, pl. 5-én és 15-én) ─────────────┐
│  Python ETL                                                     │
│  1. fetch: NEAK PDF/XLS letöltés (A,B,C,D) + OKFŐ (E)           │
│  2. parse: pdfplumber / openpyxl → nyers táblák                 │
│  3. normalize: FIN kód kulcs, címtisztítás, típuskódolás        │
│  4. geocode: új címek → Nominatim (cache: geocode_cache.json)   │
│  5. validate: sorszám-, összeg- és sémaellenőrzés,              │
│     diff az előző hónaphoz (anomália → PR-be, nem auto-merge)   │
│  6. build: aggregátumok minden szintre → data/YYYY-MM/*.json    │
│     + latest.json + timeseries.json (Parquet archívum mellé)    │
│  7. commit → repo → Cloudflare Pages / GitHub Pages deploy      │
└─────────────────────────────────────────────────────────────────┘
                          ↓
   React 18 + Vite + TypeScript SPA (statikus JSON-t fogyaszt)
   MapLibre GL JS (térkép) + D3 (grafikonok) + Zustand (állapot)
```

### Technológiai döntések

| Réteg | Választás | Indok |
|---|---|---|
| ETL | Python 3.12, pdfplumber, pandas, openpyxl | a NEAK PDF-táblák parsolásához ez a bevált stack (problématérkép-munkában már használt) |
| Adattár | verziózott JSON a repóban + Parquet archívum | ingyenes, auditálható (git diff = adattörténet!), nincs DB-üzemeltetés |
| Térkép | MapLibre GL JS + saját GeoJSON (település/járás/vármegye, simplify-olva ~1-2 MB-ra) | nyílt, nincs API-kulcs/díj |
| Grafikon | D3 | teljes kontroll a holadelej-szintű egyedi vizuálhoz |
| Frontend | React 18 + Vite + TS, Zustand | a meglévő ZoliQua-stack, azonnal produktív |
| Hosting | Cloudflare Pages (vagy GitHub Pages) | ingyenes, gyors, custom domain (pl. `praxisterkep.hu`) |
| Geokód | Nominatim, 1 req/s, teljes cache | csak a delta címeket kérdezi; a cache maga is repó-adat |

### Adatmodell (mag)

```typescript
interface Praxis {
  id: string;              // NEAK FIN / HSZ kód
  kind: 'dental' | 'gp';
  type: 'adult' | 'child' | 'mixed' | 'school';
  status: 'filled' | 'vacant' | 'vacant_longterm';
  vacantSince?: string;    // YYYY-MM
  population?: number;     // ellátandó lakosságszám
  settlementId: string;    // KSH településkód
  county: string;
  sites: { address: string; lat?: number; lon?: number }[];
  metrics?: { cards?: number; points?: number; cases?: number }; // D forrás
}
// + Settlement, District (járás), County aggregátumok
// + Timeseries: { month: string; snapshot: AggregateCounts }[]
```

---

## 7. ETL-részletek és buktatók

1. **PDF-parsing törékenység**: a NEAK bármikor átrendezheti a táblázatot → séma-ellenőrzés kötelező; hibánál a pipeline **nem** publikál, hanem issue-t nyit. Minden nyers letöltött fájl archiválódik (`raw/YYYY-MM/`), így a parser utólag javítható és a hónap újragenerálható.
2. **Kulcsstabilitás**: a FIN kód a kanonikus azonosító; körzet-összevonás/megszüntetés (praxiskezelői körzetmódosítás) esetén a kód eltűnhet → "megszűnt" státusz külön kategória, nem "betöltött".
3. **Címminőség**: a NEAK-címek vegyesek ("Dózsa Gy. u. 3.") → normalizáló lépés (rövidítés-feloldás, irányítószám-alapú validálás), sikertelen geokód esetén település-középpont fallback, megjelölve.
4. **Történeti visszatöltés**: induláskor a Wayback Machine + saját archívumok bejárása a korábbi havi listákért — amennyi elérhető, annyi; a idősor kezdőpontja adatfüggő és dokumentált.
5. **Duplikált telephelyek**: egy szolgálat több telephellyel (SZ-jelölés = székhely) → telephely-tömb, a térképen az első rendelő-telephely a pont.

---

## 8. Jogi, etikai, kommunikációs keretek

- **Csak nyilvános, hivatalos adat** (NEAK, OKFŐ, KSH, OSM). Személynév (helyettesítő orvos stb.) **nem** kerül megjelenítésre, még ha a forrásban szerepel is — csak körzet, cím, státusz.
- Minden szám mellett forrás + hónap; módszertani oldal (`/modszertan`) a teljes pipeline leírásával; a repo nyilvános → **teljes reprodukálhatóság** (a no-hallucination elv adatban).
- Semleges hangnem: a tény beszél. A szerepedből adódóan (kollegiális vezető) különösen fontos, hogy az oldal **ne** tűnjön politikai fegyvernek — az "impresszum/miért készült" oldal ezt explicit kezeli.
- Licenc: kód MIT, adat-feldolgozások CC BY 4.0, forrásadatok az eredeti kibocsátó feltételei szerint.

---

## 9. Ütemterv

### MVP (3-4 hét munkaidő-ekvivalens)
- [ ] ETL az A + B + C forrásra (aktuális hónap), geokódolás, validáció
- [ ] GeoJSON alaprétegek (település/járás/vármegye) előállítása
- [ ] Térkép-szekció choroplethtel + pontokkal, fogorvos/háziorvos váltóval
- [ ] "Most" számok + település-kereső + körzetkártya
- [ ] Forrás-lábléc, módszertan-oldal, deploy custom domainre

### v1 (+3-4 hét)
- [ ] D forrás (kártyaszám / fogászati teljesítmény) integrálása
- [ ] Idősor: havi archiválás élesítése + történeti visszatöltés (Wayback)
- [ ] Trend-szekció, hónap-slider, ki-be áramlás
- [ ] Rangsorok, "Mennyi ez valójában?" blokkok
- [ ] Elérhetőség-számítás (legközelebbi betöltött praxis, légvonal; közúti v2)
- [ ] OG-képek megosztáshoz (megye/település szintű kártyák)

### v2
- [ ] Fogászati ügyeleti réteg (I forrás)
- [ ] Közúti izokron elérhetőség (OSRM, offline előszámítva)
- [ ] EN verzió
- [ ] Letölthető adatcsomagok (CSV/Parquet) kutatóknak, sajtónak
- [ ] Automatikus havi "változás-riport" (mi ürült ki, mi töltődött be) — akár sajtó-hírlevélként

---

## 10. Repó-struktúra

```
praxisterkep/
├── CLAUDE.md                # fejlesztési szabályok (külön dokumentum)
├── etl/
│   ├── fetch_neak.py        # letöltés + raw archiválás
│   ├── parse_dental.py      # A forrás parser
│   ├── parse_gp.py          # B forrás parser
│   ├── parse_registry.py    # C (törzs) parser
│   ├── geocode.py           # Nominatim + cache
│   ├── validate.py          # séma- és diff-ellenőrzés
│   └── build.py             # aggregátumok → data/
├── data/
│   ├── raw/YYYY-MM/         # eredeti PDF/XLS (audit)
│   ├── YYYY-MM/*.json       # havi snapshot
│   ├── latest.json
│   ├── timeseries.json
│   └── geo/*.geojson        # határok (simplify-olt)
├── web/                     # React + Vite + TS
│   ├── src/components/      # Map, Trend, SettlementCard, ...
│   ├── src/store/           # Zustand
│   └── src/i18n/            # hu.json (en.json v2)
└── .github/workflows/
    └── monthly-etl.yml      # cron + deploy
```

---

## 11. Nyitott kérdések (döntést igényel)

1. **Domain**: `praxisterkep.hu`? `holafogorvos.hu`? (utóbbi a holadelej-re rímel, lakossági; előbbi szakmaibb)
2. **Történeti mélység**: mennyi energiát érdemes a Wayback-visszatöltésbe tenni vs. "mostantól gyűjtünk"?
3. **Kettős vagy fogászat-fókuszú indulás**: az MVP mehet-e csak fogorvosi adattal (gyorsabb, a te terepen), és a háziorvosi jön v1-ben? A pipeline-t mindenképp kétkasszásra tervezzük.
4. **Szerzőség/impresszum**: magánprojekt (ZoliQua) vs. szakmai szervezethez kötött megjelenés — a kollegiális vezetői szerep miatt ezt érdemes előre eldönteni.
