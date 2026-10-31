# Adatok és gráfok: Marvel Universe Social Network

A `data/` mappa három fájljából két gráf építhető:

1. **Bipartit hős–képregény gráf:** melyik hős melyik képregényben szerepel. **A projekt ezzel a gráffal dolgozik**, ezt építi fel a `beadando.ipynb`.
2. **Hős–hős szereplési gráf:** melyik két hős szerepel együtt, és hányszor. A `hero-network.csv`-ből épül, a projektben már nem használjuk, itt csak leírásként szerepel.

A számok a jelenlegi adatokra vonatkoznak, amelyekben a `MALAN` és a `SPIDER-MAN/PETER PARKER` nevet már javítottuk. Ahol a szöveg nem mond mást, a bipartit gráf számai a `beadando.ipynb` tisztítása utáni állapotot írják le.

## Forrásfájlok

| Fájl | Sorok | Oszlopok | Tartalom |
|---|---|---|---|
| `nodes.csv` | 19 090 | `node`, `type` | Az összes csúcs: 6439 hős (`hero`) és 12 651 képregény (`comic`) |
| `edges.csv` | 96 104 | `hero`, `comic` | Hős–képregény szereplések, duplikátum nélkül |
| `hero-network.csv` | 574 467 | `hero1`, `hero2` | Hős–hős együttszereplések. Egy pár annyiszor szerepel, ahány közös képregényük van. A notebook nem használja. |

A `nodes.csv` és az `edges.csv` konzisztens egymással: minden hős és képregény mindkettőben megvan, és mindegyiknek van legalább egy éle. A képregényoszlopban viszont vannak hibás bejegyzések, ezeket a notebook tisztítja (lásd lent).

## 1. gráf: bipartit hős–képregény gráf

**Építés** (`beadando.ipynb`, „Data loading and cleaning” fejezet): a csúcsok és az élek a megtisztított `edges.csv`-ből jönnek (`bipartite=0` a hős, `1` a képregény). A hős- és képregénynevek között nincs ütközés.

| Mérőszám | Tisztítás után |
|---|---|
| Csúcsok | 19 031 (6439 hős + 12 592 képregény) |
| Élek | 96 092 |
| Bipartit sűrűség | 0,0012 |
| Komponensek | 20, az óriáskomponens 18 974 csúcs (99,7%) |
| Képregény / hős | átlag 14,9, medián 3, max 1577 (Spider-Man) |
| Hős / képregény | átlag 7,6, medián 6, max 111 (`COC 1`) |
| Csak 1 képregényben szereplő hős | 1795 |
| Csak 1 hőst tartalmazó képregény | 1420 |

Legtöbbet szereplő hősök: Spider-Man (1577), Captain America (1334), Iron Man (1150), Thing (963), Thor (956).

### Tisztítás

A nyers adat 19 090 csúcsot és 96 104 élt tartalmaz. A hősoldal tiszta, minden hiba a képregényoszlopban van:

1. **Nem képregény bejegyzések törlése (10 sor):** 5 szövegtöredék (pl. `Note: Appearances in`, `rse powers.`) és 5 kód helyére írt szereplőnév (pl. `HAWKEYE/CLINT BARTON`). Egyetlen hős sem maradt emiatt képregény nélkül.
2. **Kódok egységesítése (52 kód, a képregények száma 12 641-ről 12 592-re csökken):**
   - dupla szóköz (`AWC  7` → `AWC 7`);
   - `&and` a `&` helyett (`PM&andIF 100` → `PM&IF 100`);
   - oldalszám és `-BTS` utótag (`X 83 Pg 1-8` → `X 83`, `FF 19-BTS` → `FF 19`);
   - 10 elírás a `COMIC_FIXES` táblában (pl. `MCP 18/4` → `M/CP 18/4`).
3. **Duplikált sorok eldobása:** az egységesítés után 2 sor lett azonos.
4. **Átnézendő kódok (17 darab):** a notebook csak kiírja őket javaslattal, nem változtat rajtuk, mert az adatból nem dönthető el, mi a helyes alak (pl. `ASM1`, `ROA 3 & ROA 4`, `DESTROYER IV`). Összesen kb. 20 élt érintenek.

A hasonló, de külön sorozatnak tűnő kódokat nem vontuk össze, pl. `HAWK2` és `HAWK 2`, `X&M` és `XM`, `M/K` és `MK`.

### Rendelkezésre álló adatok

- **Csúcstípus:** hős vagy képregény.
- **A hősnév szerkezete** (csak a névből származtatható):
  - `ÁLNÉV/VALÓDI NÉV`, pl. `IRON MAN/TONY STARK`: 1607 név tartalmaz `/`-t. Közülük 431 `/`-re végződik, ezeknél nincs valódi név, pl. `BLADE/`.
  - `VEZETÉKNÉV, KERESZTNÉV`, pl. `JAMESON, J. JONAH`: 2184 név, főleg civil szereplők.
  - Alternatív univerzumbeli változat ` | ` utótaggal, pl. `WOLVERINE | MUTANT X`: 131 név. Ezek többsége Mutant X változat, néhány `TIMESLIDE`, `EARTH …` vagy `AOA`.
  - A nevek 20 karakterre vannak levágva (1350 név pontosan 20 karakter hosszú).
- **A képregénykód szerkezete:** `SOROZAT SZÁM`, pl. `ASM 22`, `FF 48`. Ebből kb. 650 sorozat-előtag nyerhető ki, a legnagyobbak: `ASM` (473), `M/CP` (462), `T` (461), `FF` (432), `CA` (429). Kivételek:
  - 1610 kód számrészében van `/` (pl. `ASM 22/3`), aminek a jelentése az adatból nem derül ki;
  - 84 kódban nincs szám (pl. `DOOM`, `TB:LS`);
  - 2 kód két kiadást jelöl egyszerre (`TTA 12 | WWT 21`).

### Elemzési lehetőségek

- Fokszám-eloszlás külön a két oldalon (hősök és képregények), összevetés véletlen modellekkel.
- Bipartit centralitás (`nx.bipartite`), hogy a hősöket a képregényeken keresztüli kapcsolataik alapján rangsoroljuk.
- Sorozatszintű összesítés a kód előtagja alapján, pl. hős–sorozat gráf, vagy mely hősök kötik össze a sorozatokat.
- Bipartit klaszterezés és redundancia (`nx.bipartite`).

### Elemzési lehetőségek a laborok alapján

| Labor | Alkalmazás a bipartit gráfra |
|---|---|
| 02 Metrics | Fokszám-, közelség-, köztiség- és sajátvektor-centralitás bipartit változatban (`nx.bipartite`), a rangsorok összevetése |
| 03 Connectivity | Átlagos úthossz, átmérő, komponensek, a központi hősök vagy nagy képregények eltávolításának hatása |
| 04 Visualization | Részgráfok rajzolása (pl. a top hősök és képregényeik, egy hős vagy sorozat környezete) |
| 05 Random Networks | A két oldal fokszám-eloszlásának összevetése véletlen bipartit modellel |
| 06 Community Detection | Közösségkeresés, a közösségek összevetése az ismert csapatokkal (X-Men, Bosszúállók, Fantasztikus Négyes) |
| 07 Diffusion | Terjedés hősökön és képregényeken keresztül, a legbefolyásosabb kezdőhősök keresése |

## 2. gráf: hős–hős szereplési gráf

Ezzel a gráffal nem megyünk tovább, a leírása csak tájékoztató jellegű.

**Építés** (a notebook korábbi változatában): szóközök levágása a nevekről, hurokélek eldobása (2232 sor), a duplikált párok egy irányítatlan éllé vonása, `weight` = az együttszereplések száma, 3 csonkolt név javítása.

| Mérőszám | Érték |
|---|---|
| Csúcsok | 6421 |
| Élek | 167 100 |
| Sűrűség | 0,0081 |
| Komponensek | 4 (méretük: 6403, 9, 7, 2) |
| Fokszám | átlag 52, medián 20, max 1903 (Captain America) |
| Élsúly | átlag 3,4, medián 1. Az élek 54%-a egyetlen együttszereplés. |
| Átlagos klaszterezettség | 0,775 |
| Tranzitivitás | 0,195 |

## Miért nem a `hero-network.csv`?

A fájl súlyai nem megbízhatók: **53 pár** több együttszereplést mutat, mint ahány képregényben a két hős közül a kevesebbet szereplő egyáltalán megjelenik. A legszembetűnőbb példák:

- `MISS AMERICA/MADELIN`–`PATRIOT/JEFF MACE`: 1894 sor, de a Patriot csak 9 képregényben szerepel, és csak 8 közös képregényük van;
- `BUCKY/BUCKY BARNES`–`CAPTAIN AMERICA`: 301, de a Bucky csak 47 képregényben szerepel.

A fájl az `edges.csv`-vel sem egyezik: 4544 hőspár közös képregénye hiányzik belőle. A bipartit gráfot ez nem érinti, mert az `edges.csv` közvetlen hős–képregény adatokból épül.

## Ami nincs az adatokban

- **Idő:** nincs kiadási dátum, így időbeli fejlődés nem vizsgálható. A képregénysorszámok csak sorozaton belül adnak sorrendet.
- **Szereplőattribútumok:** nincs nem, csapat, oldal (hős vagy gonosz), sem első megjelenés.
- **Képregénycím:** csak rövid kódok vannak (pl. `ASM`), a feloldásukhoz külső forrás kell.
- **Irány:** az élek irányítatlanok.
- **Alapigazság:** a Karate Club-bal ellentétben nincs ismert „helyes” felosztás vagy rangsor. Az eredmények külső forrással vethetők össze, pl. egy top 40 listával vagy az ismert csapatokkal.

## Adatminőség: már kezelt és nyitott pontok

- **Kezelve a notebookban:** a képregényoszlop hibái (lásd a Tisztítás részt).
- **Javítva az adatfájlokban:**
  - `MALAN Sometimes spel` → `MALAN`, mindhárom fájlban;
  - `SPIDER-MAN/PETER PARKERKER` → `SPIDER-MAN/PETER PARKER`, a `nodes.csv`-ben.
- **Szándékosan külön hagyva**, mert az adatból nem dönthető el, hogy ugyanaz-e a szereplő:
  - `ATOM SMASHER/` és `ATOM-SMASHER/`;
  - `DEAD HEAD/` és `DEADHEAD`;
  - `RAZOR WIRE/` és `RAZORWIRE`;
  - a ` | ` utótagú alternatív változatok, pl. `HUSK/PAIGE GUTHRIE |`.
- **Nyitott:** a 17 átnézendő képregénykód.
