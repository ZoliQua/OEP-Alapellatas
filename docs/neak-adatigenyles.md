# Közérdekű adat igénylése — fogászati alapellátási körzetek ellátott települései

*Levélminta a NEAK-hoz. A szögletes zárójeles részeket kell kitölteni; a
melléklet a `docs/neak-adatigenyles-minta.xlsx`.*

---

**Címzett:** Kőrösi László részére
Nemzeti Egészségbiztosítási Alapkezelő
1139 Budapest, Váci út 73/A.
E-mail: neak@neak.gov.hu

**Feladó:** [név], [cím], [e-mail cím]

**Tárgy:** Közérdekű adat igénylése — a közfinanszírozott fogászati
alapellátási körzetekhez tartozó települések listája

**Kelt:** [település], [dátum]

---

Tisztelt Címzett!

Az információs önrendelkezési jogról és az információszabadságról szóló
2011. évi CXII. törvény (Infotv.) 28. § (1) bekezdése alapján közérdekű adat
megismerése iránti igénnyel fordulok Önökhöz.

## Amit kérek

A **közfinanszírozott fogászati alapellátási körzetek és az általuk ellátott
települések összerendelését**, azaz minden fogászati körzet (FIN kód /
szervezeti egység kód) mellé azoknak a településeknek a felsorolását,
amelyeket az adott körzet ellát.

Kizárólag körzet–település összerendelést kérek. **Személyes adatot nem
igénylek**: sem orvosnevet, sem beteg- vagy kártyaadatot, sem más, természetes
személyhez köthető információt.

## Miért ezt

A NEAK által havonta közzétett **háziorvosi** szerződött szolgáltatói listában
ez az adat szerepel: az „Ellátott települések" oszlop KSH-kóddal és névvel
felsorolja, mely településeket szolgálja az adott körzet. A **fogászati**
listában viszont csak a rendelő székhelye található meg.

Ennek az a következménye, hogy a nyilvános adatokból **3177 településből 2105
esetében nem állapítható meg, melyik fogászati körzethez tartozik** — a
települések kétharmada nem szerepel egyetlen fogászati körzet listáján sem.
Ez nem azt jelenti, hogy ott nincs ellátás, hanem azt, hogy a hozzárendelés
nem nyilvános.

Az adatot a [praxisterkep / oep-ellatas.vercel.app] nevű, nyilvános,
nonprofit térképen szeretném felhasználni, amely kizárólag nyilvános NEAK-,
EESZT- és KSH-adatokból dolgozik, és minden számánál megjelöli a forrását.
Jelenleg a fogászati ellátás esetében csak azt tudjuk kiírni, hogy a
legközelebbi rendelő hol van — azt nem, hogy hivatalosan melyik körzet az
illetékes. A kért adat ezt a hiányt pótolná.

## Formátum

Kérem az adatot elektronikus úton, gépi feldolgozásra alkalmas formában
(XLSX vagy CSV), a [hivatkozott e-mail címre]. A könnyebbség kedvéért
mellékelek egy **kitöltésre előkészített munkafüzetet**
(`neak-adatigenyles-minta.xlsx`), amely a javasolt oszlopszerkezetet
tartalmazza. Ha az Önök nyilvántartásából más szerkezetű kivonat egyszerűbben
előállítható, az is tökéletesen megfelel — a lényeg a FIN kód és az ellátott
település összerendelése.

Tájékoztatom, hogy az Infotv. 29. § (1) bekezdése szerint az igénynek a
beérkezéstől számított **legfeljebb 15 napon belül** kell eleget tenni.
Amennyiben az igény teljesítése költségtérítéshez kötött, kérem, hogy annak
összegéről előzetesen tájékoztassanak.

Segítségüket és munkájukat előre is köszönöm.

Tisztelettel:

[név]

---

## Megjegyzések a levélhez (nem a levél része)

- **A címzett beosztása**: ha ismert a pontos titulus, érdemes kiírni
  („főigazgató úr részére"). Ellenőrizetlen titulust nem írtam a mintába.
- **Mellékelt fájl**: `docs/neak-adatigenyles-minta.xlsx`, két munkalappal —
  *Kitöltendő* (oszlopfejlécek, három sárga példasorral, amit törölni kell)
  és *Útmutató* (mit jelent az oszlop, miért szűk a kérés).
  Újragenerálható: `python scripts/neak_template.py`.
- **Ha jön válasz**: a feldolgozó oldal kész — a háziorvosi listánál pontosan
  ilyen szerkezetű adatot dolgozunk fel (`parse_gp.py`), a fogászati
  megfelelője egy nap munka, utána mind a 3177 településlapon megjelenik,
  melyik körzet az illetékes.
- **Második kérdés, ha egyszer úgyis írsz**: a rendelési idők. Egyik nyilvános
  forrás sem közli őket, a NEAK-nak viszont a szerződésekből ismertek. Ezt
  szándékosan nem tettem ebbe a levélbe — egy levél egy kérdéssel gyorsabban
  megy végig —, de egy másodikban ugyanígy kérhető.
