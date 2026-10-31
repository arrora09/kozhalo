# Projektleírás: népszerűség és hálózati központiság a Marvel-univerzumban

## Áttekintés

A projekt a Marvel Universe Social Network adathalmazból épített bipartit hős–képregény gráfot vizsgálja. A cél annak ellenőrzése, hogy a gráf szerkezetéből számolt fontosság mennyire adja vissza azt, amit egy gráftól független, külső lista a legfontosabb Marvel-szereplőknek tart.

Az adatok és a tisztítás részletes leírása a `DATA.md`-ben, a kód a `beadando.ipynb`-ben található.

## Elemzési kérdés

> **Mennyire adja vissza a gráf szerkezete azt, hogy kik a legfontosabb Marvel-szereplők?**

Alkérdések:

1. Mekkora az átfedés a külső top 40 lista és a gráfból számolt top 40 között?
2. Mennyire egyezik a sorrend a külső lista és a gráfból számolt rangsorok között?
3. Melyik centralitási mérték áll legközelebb a külső listához?
4. Mely szereplők térnek el a legjobban, és mi magyarázhatja az eltérést?
5. Fennmaradnak-e a következtetések, ha a külső listát egy másik évből vett listára cseréljük?

## Adat: a gráf

A vizsgált gráf a **ki miben szerepel** bipartit gráf. Egy hőst akkor köt él egy képregényhez, ha a hős szerepel benne.

| Tulajdonság | Érték |
|---|---|
| Forrás | `data/nodes.csv`, `data/edges.csv` |
| Hősök | 6439 |
| Képregények | 12 592 |
| Élek | 96 092 |
| Komponensek | 20, az óriáskomponens 18 974 csúcs (99,7%) |
| Élsúly | nincs |

A „hős” itt minden szereplőt jelent, tehát a gazembereket és a civil szereplőket is.

A hős–hős gráffal (`hero-network.csv`) nem dolgozunk.

## Alapigazság

Az adathalmazban nincs alapigazság, ezért külső listákat használunk.

| Szerep | Lista | Módszer |
|---|---|---|
| **Fő alapigazság** | [CBR 2011 Top Marvel Characters](https://www.cbr.com/2011-top-marvel-characters-100-97/), top 40 | Közel 1400 olvasó szavazata, pontozással |
| **Ellenőrzés** | [CBR 2019 Top Marvel Characters](https://www.cbr.com/top-100-dc-marvel-characters-master-list/), top 40 | Közel 1100 olvasó szavazata, pontozással |

A listák külön fájlban vannak: `data/baseline_2011.csv` és `data/baseline_2019.csv`.

A választás indoklása:

- **Független a gráftól:** a lista szavazásból készült, nem a szereplések számából. Egy szereplésszámon alapuló lista (pl. a FiveThirtyEight megjelenési adatai) körkörös lenne, mert lényegében a fokszámot adná vissza.
- **Időben közeli:** az adathalmaz kb. 2001-ig tart, a 2011-es lista a legközelebbi megfelelő forrás.
- **Pontszámokat is ad:** így nemcsak a sorrend, hanem a szavazatok aránya is rendelkezésre áll.

Az alapigazság korlátja, hogy a kedveltséget méri, nem a hálózati fontosságot. A két fogalom eltérése nem hiba, hanem a vizsgálat tárgya: a gráf a jelenlétet mutatja, a szavazás a népszerűséget.

## Módszer

### 1. Adatelőkészítés

A gráf beolvasása, tisztítása és kirajzolása a `beadando.ipynb` „Data loading and cleaning” fejezetében készült el.

### 2. A külső listák előkészítése

A CBR 2011 és 2019 top 40 egy-egy CSV fájlban van, a szereplők párosítva a gráf csúcsneveivel (pl. `Iron Man` → `IRON MAN/TONY STARK`).

| Oszlop | Tartalom |
|---|---|
| `poll_year`, `rank`, `character` | A szavazás éve, a helyezés és a szereplő neve a forrás szerint |
| `points`, `first_place_votes` | Pontszám és első helyes szavazatok száma |
| `graph_node` | A párosított csúcs neve. Üres, ha a szereplő nincs a gráfban. |
| `match_note` | Megjegyzés a nem egyértelmű párosításokhoz |
| `source_url` | A forrásoldal |

A 2011-es listából 39, a 2019-esből 37 szereplő van meg a gráfban. Hiányzik az X-23 mindkét listáról, valamint Miles Morales és Ms. Marvel (Kamala Khan) a 2019-esről, mert az adathalmaz vége után jelentek meg. A két lista gráfban is meglévő szereplői közül 33 közös.

Párosítási szabályok:

- A párosítás kézzel készül, mert a gráf nevei 20 karakterre vannak levágva.
- Egy listaelemhez a szereplő fő változata tartozik. A ` | ` utótagú alternatív univerzumbeli változatok nem számítanak bele.
- Ha egy szereplő nincs a gráfban, pl. mert az adathalmaz vége után jelent meg, azt rögzítjük, és kihagyjuk a rangkorrelációból.
- Ha egy név több szereplőt is jelölhet (pl. több Captain Marvel), a listaoldal leírása alapján döntünk, és a döntést rögzítjük.

### 3. A bipartit gráf elemzései

A `DATA.md` alapján ezeket az elemzéseket végezzük el a bipartit gráfon:

- **Fokszám-eloszlás:** külön a két oldalon (hősök és képregények), összevetés véletlen modellekkel.
- **Bipartit centralitás** (`nx.bipartite`): a hősök rangsorolása a képregényeken keresztüli kapcsolataik alapján. Ennek részleteit a következő lépés írja le.
- **Sorozatszintű összesítés:** a képregénykód előtagja alapján, pl. hős–sorozat gráf, vagy mely hősök kötik össze a sorozatokat.
- **Bipartit klaszterezés és redundancia** (`nx.bipartite`).

### 4. Rangsorok számítása

A hősök rangsorolása a bipartit gráfon, több mértékkel:

| Mérték | Mit mutat |
|---|---|
| Fokszám (`nx.bipartite.degree_centrality`) | Hány képregényben szerepel a hős |
| Közelségi centralitás (`nx.bipartite.closeness_centrality`) | Milyen közel van a hős a gráf többi részéhez |
| Köztiségi centralitás (`nx.bipartite.betweenness_centrality`) | Mennyire köti össze a gráf különböző részeit |
| Sajátvektor-centralitás (`nx.eigenvector_centrality`) | Mennyire kapcsolódik más fontos csúcsokhoz |
| PageRank (`nx.pagerank`) | Mekkora eséllyel jut el hozzá egy véletlen bolyongás |

A nem fokszám alapú mértékeket az óriáskomponensen számoljuk. A rangsorokba csak a hősök kerülnek, a képregények nem. A köztiségi centralitás ezen a méreten lassú lehet, ezért szükség esetén mintavételes közelítést használunk, és ezt jelezzük.

### 5. Összevetés a külső listával

| Mutató | Kérdés |
|---|---|
| Top-40 átfedés | Hány szereplő van benne mindkét top 40-ben? |
| Spearman-féle rangkorreláció | Mennyire egyezik a listás és a gráfbeli sorrend? |

- **Viszonyítási alap:** ha 6439 hősből véletlenszerűen választanánk 40-et, a várható átfedés a listával kb. 0,25 szereplő.
- **Rangkorreláció:** a párosított szereplőkre számoljuk, a teljes hősrangsorbeli helyük alapján.

### 6. Értelmezés és ellenőrzés

1. A mértékek összehasonlítása: melyik áll legközelebb a listához.
2. A legnagyobb eltérések elemzése mindkét irányban: akik a listán előrébb vannak, mint a gráfban, és akik a gráfban vannak előrébb.
3. A teljes elemzés megismétlése a CBR 2019 listával.
4. A gráf és a lista eltérésének szemléltetése ábrákkal.

## Elvárt eredmények

- A külső listák CSV formátumban, a forrás megjelölésével (`data/baseline_2011.csv`, `data/baseline_2019.csv`).
- A `beadando.ipynb` „Analysis” fejezete: a bipartit gráf elemzései, rangsorok, összevetések, ábrák.
- Írásos válasz az elemzési kérdésre és az alkérdésekre, az eltérések magyarázatával.
- Annak bemutatása, hogy melyik eszköz mire alkalmas, és mit nem tud megmutatni.

## Korlátok

- **Időbeli eltérés:** az adathalmaz kb. 2001-ig tart, a listák 2011-esek és 2019-esek. Az újabb filmek és sorozatok hatása az adatban nem látszik, a szavazásban viszont igen.
- **Szereplésszám és fontosság:** egy képregényben minden szereplés egyenértékű, a főszerep és egy rövid megjelenés között nincs különbség.
- **Névváltozatok:** egy szereplő több csúcsként is megjelenhet, ami csökkentheti a mért centralitását.
- **Sorozatok súlya:** a sokáig futó sorozatok szereplői a szereplések számában eleve előnyben vannak.
- **Átnézendő kódok:** 17 képregénykód javítása nyitott maradt, kb. 20 élt érint, ezért az eredményt érdemben nem befolyásolja.

## Állapot

| Feladat | Állapot |
|---|---|
| Bipartit gráf beolvasása, tisztítása, kirajzolása | Kész |
| Elemzési kérdés és alapigazság kiválasztása | Kész |
| CBR 2011 és 2019 top 40 összegyűjtése | Kész |
| Nevek párosítása a gráffal | Kész |
| A bipartit gráf elemzései | Kész |
| Centralitások számítása | Kész |
| Összevetés a listákkal | Kész |
| Értelmezés és ellenőrzés | Kész |
