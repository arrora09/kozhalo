# Adatok és gráfok: Marvel Universe Social Network

A `data/` mappa három fájljából két gráf építhető:

1. **Bipartit hős–képregény gráf:** melyik hős melyik képregényben szerepel.
2. **Hős–hős szereplési gráf:** melyik két hős szerepel együtt, és hányszor.

A számok a jelenlegi adatokra vonatkoznak, amelyekben a `MALAN` és a `SPIDER-MAN/PETER PARKER` nevet már javítottuk. A hős–hős gráf számai a `beadando.ipynb` tisztítása utáni állapotot írják le.

## Forrásfájlok

| Fájl | Sorok | Oszlopok | Tartalom |
|---|---|---|---|
| `nodes.csv` | 19 090 | `node`, `type` | Az összes csúcs: 6439 hős (`hero`) és 12 651 képregény (`comic`) |
| `edges.csv` | 96 104 | `hero`, `comic` | Hős–képregény szereplések, duplikátum nélkül |
| `hero-network.csv` | 574 467 | `hero1`, `hero2` | Hős–hős együttszereplések. Egy pár annyiszor szerepel, ahány közös képregényük van. |

A `nodes.csv` és az `edges.csv` teljesen konzisztens: minden hős és képregény mindkettőben megvan, és mindegyiknek van legalább egy éle.

## 1. gráf: bipartit hős–képregény gráf

**Építés:** a csúcsok a `nodes.csv`-ből jönnek (`bipartite=0` a hős, `1` a képregény), az élek az `edges.csv`-ből. A hős- és képregénynevek között nincs ütközés.

| Mérőszám | Érték |
|---|---|
| Csúcsok | 19 090 (6439 hős + 12 651 képregény) |
| Élek | 96 104 |
| Bipartit sűrűség | 0,0012 |
| Komponensek | 22, az óriáskomponens 19 029 csúcs |
| Képregény / hős | átlag 14,9, medián 3, max 1577 (Spider-Man) |
| Hős / képregény | átlag 7,6, medián 6, max 111 (`COC 1`) |
| Csak 1 képregényben szereplő hős | 1795 |
| Csak 1 hőst tartalmazó képregény | 1480 |

Legtöbbet szereplő hősök: Spider-Man (1577), Captain America (1334), Iron Man (1150), Thing (963), Thor (956).

### Rendelkezésre álló adatok

- **Csúcstípus:** hős vagy képregény.
- **A hősnév szerkezete** (csak a névből származtatható):
  - `ÁLNÉV/VALÓDI NÉV`, pl. `IRON MAN/TONY STARK`: 1607 név tartalmaz `/`-t. Közülük 431 `/`-re végződik, ezeknél nincs valódi név, pl. `BLADE/`.
  - `VEZETÉKNÉV, KERESZTNÉV`, pl. `JAMESON, J. JONAH`: 2184 név, főleg civil szereplők.
  - Alternatív univerzumbeli változat ` | ` utótaggal, pl. `WOLVERINE | MUTANT X`: 131 név. Ezek többsége Mutant X változat, néhány `TIMESLIDE`, `EARTH …` vagy `AOA`.
  - A nevek 20 karakterre vannak levágva (1350 név pontosan 20 karakter hosszú).
- **A képregénykód szerkezete:** `SOROZAT SZÁM`, pl. `ASM 22`, `FF 48`. Ebből kb. 670 sorozat-előtag nyerhető ki, a legnagyobbak: `ASM` (473), `M/CP` (462), `T` (461), `FF` (433), `CA` (429). Kivételek:
  - 2322 kódban `/` van (pl. `ASM 22/3`), aminek a jelentése az adatból nem derül ki;
  - 85 kódban nincs szám (pl. `DOOM`, `TB:LS`);
  - 2 kód két kiadást jelöl egyszerre (`TTA 12 | WWT 21`).

### Elemzési lehetőségek

- Fokszám-eloszlás külön a két oldalon (hősök és képregények), összevetés véletlen modellekkel.
- Bipartit vetítés a hősökre: ez adja a 2. gráfot, de tiszta forrásból (lásd lent). Vetíthető a képregényekre is: két képregény akkor kapcsolódik, ha van közös hősük.
- Sorozatszintű összesítés a kód előtagja alapján, pl. hős–sorozat gráf, vagy mely hősök kötik össze a sorozatokat.
- Bipartit klaszterezés és redundancia (`nx.bipartite`).

## 2. gráf: hős–hős szereplési gráf

**Építés** (`beadando.ipynb`, „Data loading and cleaning” fejezet):
1. szóközök levágása a nevekről;
2. hurokélek eldobása;
3. a duplikált párok egy irányítatlan éllé vonása, `weight` = az együttszereplések száma;
4–5. a 3 csonkolt név átnevezése (`BLADE/`, `SABRE/`, `SPIDER-MAN/PETER PARKER`).

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

- Legnagyobb fokszám (különböző társszereplők száma): Captain America (1903), Spider-Man (1737), Iron Man (1521), Thing (1416), Mr. Fantastic (1377), Wolverine (1368).
- Legnagyobb súlyozott fokszám: Captain America (16 259), Spider-Man (13 717), Iron Man (11 817), Thor (11 427).
- Legerősebb élek: a Fantastic Four tagjai között 700 fölötti súlyok. A `MISS AMERICA`–`PATRIOT` pár 1894-es súlya viszont adathiba (lásd lent).

Hiányzik 18 hős, aki a `nodes.csv`-ben szerepel: ők csak olyan képregényekben vannak, ahol rajtuk kívül nincs más hős, ezért nincs élük. Ha kellenek, izolált csúcsként hozzáadhatók.

### Rendelkezésre álló adatok

- **Élsúly:** az együttszereplések száma.
- **Csúcsattribútumok:** nincsenek külön adatként, csak a névből származtathatók (álnév és valódi név, alternatív univerzum), illetve az 1. gráfból (képregények száma, sorozatok).

### Elemzési lehetőségek a laborok alapján

| Labor | Alkalmazás a gráfra |
|---|---|
| 02 Metrics | Fokszám-, közelség-, köztiség-, sajátvektor-centralitás, PageRank. A súlyozott és a súlyozatlan rangsor összevetése. |
| 03 Connectivity | Átmérő, átlagos úthossz, hidak, vágások, óriáskomponens, a központi hősök eltávolításának hatása |
| 04 Visualization | Részgráfok rajzolása (pl. a top-K hős, egy közösség, egy hős ego-hálózata) |
| 05 Random Networks | Fokszám-eloszlás összevetése ER és BA modellel, kisvilág-tulajdonság (magas klaszterezettség, rövid utak) |
| 06 Community Detection | Louvain, greedy modularity, label propagation. A közösségek összevetése az ismert csapatokkal (X-Men, Bosszúállók, Fantasztikus Négyes). |
| 07 Diffusion | LT és IC modell, a súlyokból képzett valószínűségekkel, és a legbefolyásosabb kezdőhősök keresése |

## A két gráf kapcsolata és egy fontos adathiba

A bipartit gráf hősökre vetítve (`nx.bipartite.weighted_projected_graph`) ugyanazt a 6421 hőst adja, de **a két forrás nem egyezik:**

- A vetítés 171 644 élt ad, a `hero-network.csv` 167 100-at. 4544 pár csak a vetítésben szerepel, és nincs olyan pár, amely csak a `hero-network.csv`-ben lenne.
- **53 pár súlya lehetetlenül nagy a `hero-network.csv`-ben:** több együttszereplést mutat, mint ahány képregényben a két hős közül a kevesebbet szereplő egyáltalán megjelenik. A legszembetűnőbb példák:
  - `MISS AMERICA/MADELIN`–`PATRIOT/JEFF MACE`: 1894 sor, de a Patriot csak 9 képregényben szerepel, és csak 8 közös képregényük van;
  - `BUCKY/BUCKY BARNES`–`CAPTAIN AMERICA`: 301, de a Bucky csak 47 képregényben szerepel.

**Következmény:** a súlyozott elemzésekhez (súlyozott centralitás, diffúziós valószínűségek, súlyozott layout) megbízhatóbb a vetített gráf súlyait használni, vagy legalább ellenőrizni a `hero-network.csv` súlyait. A súlyozatlan szerkezetet ez kevésbé érinti.

## Ami nincs az adatokban

- **Idő:** nincs kiadási dátum, így időbeli fejlődés nem vizsgálható. A képregénysorszámok csak sorozaton belül adnak sorrendet.
- **Szereplőattribútumok:** nincs nem, csapat, oldal (hős vagy gonosz), sem első megjelenés.
- **Képregénycím:** csak rövid kódok vannak (pl. `ASM`), a feloldásukhoz külső forrás kell.
- **Irány:** az élek irányítatlanok.
- **Valódi közösségbeosztás:** a Karate Club-bal ellentétben nincs ismert „helyes” felosztás. A közösségek csak kézzel vethetők össze ismert csapatokkal.

## Adatminőség: már kezelt és nyitott pontok

- **Kezelve a notebookban:** szóközök a nevek végén (141 név), hurokélek (2232 sor), duplikált párok, 3 csonkolt név.
- **Javítva az adatfájlokban:**
  - `MALAN Sometimes spel` → `MALAN`, mindhárom fájlban;
  - `SPIDER-MAN/PETER PARKERKER` → `SPIDER-MAN/PETER PARKER`, a `nodes.csv`-ben.
- **Szándékosan külön hagyva**, mert az adatból nem dönthető el, hogy ugyanaz-e a szereplő:
  - `ATOM SMASHER/` és `ATOM-SMASHER/`;
  - `DEAD HEAD/` és `DEADHEAD`;
  - `RAZOR WIRE/` és `RAZORWIRE`;
  - a ` | ` utótagú alternatív változatok, pl. `HUSK/PAIGE GUTHRIE |`.
- **Nyitott:** a `hero-network.csv` súlyai és a vetítés közti eltérés (lásd fent).
