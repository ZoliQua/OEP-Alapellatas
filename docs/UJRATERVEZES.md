# Újratervezés — információs architektúra és a lapok egymásra épülése

Állapotfelmérés és terv · 2026-10-07 · a `docs/PLAN.md` kiegészítése

A projekt adatoldala kész: 26 pipeline-lépés, ~30 adatfájl, ország-, vármegye-
és településszintű számok mindenből. A megjelenítés viszont a növekedés
sorrendjében épült, nem egy szerkezet szerint — ez a dokumentum azt írja le,
mi ebből a baj, és milyen felépítés felelne meg az adatnak.

---

## 1. Hol tartunk ma

**Öt belépési pont:** `index.html` (landing), `elemzo.html`, `eeszt.html`,
`szakellato.html`, `embed.html`, plusz két generált statikus halmaz:
`/telepules/<slug>/` (3177 lap + index + sitemap) és `/megye/<slug>/`
(20 lap, ami valójában csak OG-metaadat és átirányítás).

**A főoldal tizenegy szekciót görget egymás alá:** Hero → ScrollyIntro →
Térkép → Miért fontos (benne a 923 soros PreventionStory) → Nálam → Rangsor →
Statisztika → Összevetés → Távolság → Adatok → Módszertan.

**Az Elemző kilenc elemzést** sorol egy tartalomjegyzék mögött, az EESZT-lap
egyetlen hosszú szekciót (benne öt alblokk), a Szakellátás-lap hét modálist.

---

## 2. Diagnózis — tíz pont

### 2.1 Négyféle navigáció ugyanarra az oldalra
A főoldal `NavMenu`-je almenüs és horgonyokra mutat. Az aloldalak `PageNav`-ja
lapos, és **lapja válogatja, mit lát belőle**: az Elemzőről az EESZT és a
Szakellátás érhető el, a Szakellátásról az Elemző és az EESZT, a
településlapok statikus menüjéből pedig hiányzik az EESZT, viszont van benne
egy „Települések" pont, amit az SPA egyik lapja sem ismer. Ugyanaz az oldal
négy különböző térképet ad magáról.

### 2.2 Az ág (fogorvos / háziorvos) nem utazik
A főoldal a Zustand-store-ban tartja és az URL-be írja (`?k=`). Az Elemző
**saját `useState('dental')`-lal indul**, az EESZT-lap a store-t használja, de
friss betöltéssel, a Szakellátás-lapnak nincs is ága, a településlap pedig
statikus. Aki a főoldalon háziorvosit választ és átmegy az Elemzőre,
fogorvosi adatot kap — kérdés nélkül.

### 2.3 A terület sem utazik
A térkép `?m=Baranya`-t ír az URL-be, de ezt rajta kívül **senki nem olvassa**.
Minden elemzés újra az országos képpel indul, hiába néztük az előbb egyetlen
vármegyét.

### 2.4 A 3177 településlap zsákutca visszafelé
Odafelé rendben van: minden lap visszalinkel a térképre, az Elemzőre, az
adatokra. Viszont **egyedül a kereső (`SearchSection`) visz oda**. Az elemzések
egyetlen táblája sem linkel településlapra, pedig a klaszterek, a menetidő, a
buszelérés, az összetett index és a lefedettség mind településsoros.

### 2.5 A főoldal négy oldal munkáját végzi
Kérdés (hol a fogorvos), érvelés (miért fontos), személyes válasz (nálam),
országos statisztika (rangsor, összevetés, távolság), dokumentáció (adatok,
módszertan) — egy görgetésben, miközben már van négy aloldal, ami épp ezeket
a szerepeket tudná vinni.

### 2.6 A sorrend nem történet
Ma: térkép → miért fontos → nálam → rangsor → statisztika → összevetés →
távolság. A látogató személyes kérdése („nálam mi van?") a harmadik helyen
áll, az érvelés a második; a távolság — ami a „mi lesz velem" kérdés —
leghátul. A lapok között pedig nincs semmilyen sorrend: az Elemző, az EESZT és
a Szakellátás mellérendelt, holott más-más mélységben vannak.

### 2.7 Két lap neve rendszert nevez meg, nem kérdést
A projekt kérdésekben beszél („Hol a fogorvos?", „Busszal elérhető?"), két
menüpont viszont „EESZT" és „Szakellátás". Az előbbi egy nyilvántartás neve,
amiről a látogató nem tudja, mit keres benne.

### 2.8 Hiányzik a vármegyei szint
A hierarchia ma: **ország → (ugrás) → település**. Pedig vármegyei bontása van
a lefedettségnek, az összetett indexnek, a menetidőnek, a buszelérésnek, a
gyógyszertáraknak, a segédeszköz-kiadóknak, a korösszetételnek és a
betöltetlenségnek is. A `/megye/<slug>/` lapok ma csak átirányítanak.

### 2.9 A mély tartalom nem megosztható
A táblázatok (`DataTableModal`) szűrt, rendezett állapota, a megnyitott
modálisok, a kiválasztott elemzés-fül — semmi nincs az URL-ben. Egy konkrét
szűrést nem lehet linkelni, pedig pont ez az, amit egy újságíró vagy egy
önkormányzat továbbküldene.

### 2.10 Horgonnyal betöltve nem görget
A `/#nalam` típusú linkek betöltéskor nem ugranak a szekcióra (a tartalom a
hash feldolgozása után érkezik). Régi, ismert hiba, de pont az
oldalak közti átjárást rontja.

---

## 3. Vezérelv — „egy kérdés, négy nagyítás"

A látogató egyetlen kérdéssel érkezik: **van-e orvos ott, ahol él.** Minden
más ennek a kérdésnek a nagyítása vagy háttere. Ezért a szerkezet ne téma
szerint álljon (térkép / elemzés / nyilvántartás), hanem **nagyítás szerint**:

```
ország  →  vármegye  →  település  →  körzet
```

és minden szinten ugyanaz a három dolog legyen elérhető:
**mi a helyzet · miből tudjuk · mi következik belőle.**

---

## 4. A kontextus mint első osztályú állapot

Ez a terv műszaki magja. Egyetlen kontextus: **ág + terület + hónap**.

- egy store (`useContextStore`) tartja, minden lap ebből indul;
- az URL-ben szerepel és onnan olvasódik vissza: `?k=gp&m=Baranya&t=szigetvar`;
- a lapok közti **minden link átviszi** (egy `contextHref()` segéd);
- a statikus településlap is ismeri: a kimenő linkjei `?t=<slug>`-gal mennek,
  így az Elemző már az ő településén nyílik ki;
- egy **kontextussáv** (`ContextBar`) minden lap tetején ugyanott mutatja, mire
  vonatkozik, amit nézünk, és egy kattintással tágítható („Baranya ×" →
  országos).

Ettől lesz „egymásra épülő" a felépítés: nem új oldalakat kell gyártani, hanem
a meglévőket **ugyanarra a kiválasztásra** állítani.

---

## 5. Az új laptérkép

| Lap | Szerep | Mi kerül rá |
|---|---|---|
| `index.html` | **A kérdés és a válasz** | Hero, térkép, „Nálam" kereső. Semmi más. |
| `megye.html` (új) | **Vármegyei lap** | Rangsor, összevetés, távolság, korösszetétel, gyógyszertár/ügyelet-elérés — egy vármegyére szűrve, települései listájával. A `/megye/<slug>/` innentől ide mutat, nem a térképre. |
| `/telepules/<slug>/` | **A legszemélyesebb szint** | Marad, de kap: szomszédos települések, „ugyanebben a járásban", és minden blokkjánál visszalink a szülő elemzésre a saját sorával. |
| `elemzo.html` | **Mi következik belőle** | A kilenc elemzés, kontextusérzékenyen: a kiválasztott vármegye/település kiemelve minden ábrán. |
| `nyilvantartas.html` (ma `eeszt.html`) | **Miből tudjuk** | EESZT-komplexum + a hivatalos összerendelés + egységkor. Menücímke: „Nyilvántartás". |
| `ellatas.html` (ma `szakellato.html`) | **Ki lát el még** | Szakellátás, gyógyszertár, segédeszköz. Menücímke: „Ki lát el még?" |
| `miert.html` (új) | **Az érvelés** | A mai „Miért fontos?" + PreventionStory, kiemelve a főoldalról. |
| `modszertan.html` (új) | **A dokumentáció** | Módszertan + adatexport + beutalási törzslista, kiemelve a főoldalról. |

A fájlnevek megváltoztatása **törné a meglévő linkeket**, ezért az `eeszt.html`
és a `szakellato.html` marad a helyén; csak a menücímke és a lap címsora
változik. Új fájl csak ott keletkezik, ahol új szerep van.

**A menü hat pontja a nagyítás sorrendjében:**

```
Térkép  ·  Vármegye  ·  Település  ·  Elemző  ·  Nyilvántartás  ·  Ki lát el még?
                                                    ^ almenü: Miért fontos? · Módszertan · Adatok
```

---

## 6. Az összekötések — ettől lesz dinamikus

1. **Minden települést tartalmazó táblacella link** a saját lapjára. Egy helyen
   megoldható: a `DataTableModal` oszlopdefiníciója kap egy `link: 'settlement'`
   jelzőt, és minden tábla örökli (ma 12 tábla érintett).
2. **„Következő kérdés" sáv** minden elemzés-szekció alján, 2–3 linkkel, a
   kontextus megtartásával. Például a menetidő alatt: *„És busszal? → · Kinek a
   legrosszabb mindkettő? → · Ez a település részletesen →"*.
3. **A térképről az elemzőbe**: a kiválasztott vármegye mellett megjelenik egy
   „Nézd meg az elemzőben" gomb, ami átviszi a kontextust.
4. **A településlapról vissza az elemzésbe** a saját sorával (`?t=<slug>`), nem
   az elemzés tetejére.
5. **Megosztható mélylinkek**: a nyitott modális és a szűrése a hash-be kerül
   (`#tabla=menetido&megye=Baranya`), a `DataTableModal` ebből állítja vissza
   magát.
6. **Horgony betöltéskor**: az adat megérkezése után egyszer újra lefuttatjuk a
   hash-görgetést.

---

## 6/b. Ami ebből elkészült (2026-10-07)

Mind az öt fázis. Amiben a megvalósítás eltér a fenti tervtől:

- A **statisztikai szekciók** nem a vármegyei lapra kerültek, mert országos
  elemzések: a vármegyei rangsor a `megye.html`-re ment (az a vármegye-szint),
  az országos trendek, a Háziorvos vs Fogorvos összevetés és a légvonalbeli
  távolság pedig az Elemzőre, a többi elemzés mellé. A főoldalon így tényleg
  csak a kérdés maradt: hero, térkép, „Nálam".
- A **menü hét pontos** lett hat helyett: a Módszertan önálló pont, mert egy
  nyilvános adatokra épülő oldalon a forrás nem almenübe való.
- A vármegyei laphoz új ETL-lépés készült (`etl/county_profiles.py` →
  `data/counties.json`, 265 KB): ugyanaz az adat vármegyére szeletelve, mert
  a 3,9 MB-os településprofil-fájlt nem lehet egy böngészőre bízni.
- A **modális táblák** állapota nem a hash-be, hanem a query stringbe került
  (`?tabla=…&tq=…`), mert a hash a szekcióhorgonyoké, és a kettő összeakadt
  volna.

## 7. Végrehajtási sorrend

A sorrend szándékosan olyan, hogy **minden fázis után működő, jobb oldal**
legyen, és egyik se igényelje a következőt.

**1. fázis — a kontextus (a legnagyobb haszon, a legkisebb kockázat)**
- `useContextStore` + URL-szinkron minden lapon
- `ContextBar` komponens, mindenhol ugyanott
- `contextHref()`, és minden lapközi link átállítása rá
- az Elemző és a Szakellátás ágállapota a közös store-ból
- horgonygörgetés javítása

**2. fázis — egységes navigáció**
- egy `SiteNav`, a hat ponttal, a nagyítás sorrendjében
- a statikus generátor (`settlement-pages.mjs`) ugyanazt a nav-definíciót
  olvassa (közös JSON/TS modul), hogy ne lehessen szétcsúszni
- a két lap átcímkézése

**3. fázis — a főoldal tehermentesítése**
- `miert.html` és `modszertan.html` kiemelése
- a főoldalon marad: hero, térkép, Nálam
- a kiemelt szekciók helyén egy-egy rövid átvezető kártya

**4. fázis — a hiányzó vármegyei szint**
- `megye.html` a meglévő szekciókból, vármegyére szűrve
- a `/megye/<slug>/` lapok ide mutatnak
- a térképről és a településlapról is elérhető

**5. fázis — a dinamikus összekötések**
- településlinkek minden táblában
- „Következő kérdés" sávok
- megosztható modális-állapot

---

## 8. Amit ez a terv nem változtat meg

- Az adatcsővezeték, a fájlformátumok és a `data/` szerződés érintetlen.
- A `embed.html` beágyazó marad, ahogy van.
- A meglévő URL-ek (`eeszt.html`, `szakellato.html`, `/telepules/<slug>/`)
  tovább élnek.
- A vizuális nyelv (sötét téma, tipográfia, színek) marad; ez szerkezeti és
  nem látványterv.
