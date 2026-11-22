### 1. dia
A projektem a Marvel-univerzumról szól. Azt vizsgáltam, összefügg-e egy szereplő népszerűsége azzal, hogy hány képregényben szerepel.
Ehhez a szereplőket és a képregényeket egy hálózatba, pontosabban egy páros gráfba rendeztem, és ezt két olvasói szavazással vetettem össze.

### 2. dia
Az adathalmaz a Kaggle-ről van, egy 2002-es kutatás állította össze a Marvel Chronology Project alapján, ahol 1961-től a Marvel-korszak kezdetétől kb. 2001-ig gyűjti a képregényeket.
Páros gráfként kezelem: az egyik oldalon a szereplők, a másikon a képregények, és él akkor van, ha a szereplő szerepel az adott füzetben. A „szereplő” itt mindenkit jelent, hősök, gonoszok, civilek, stb.
Tisztítás után 6439 szereplő, 12 592 képregény és kb. 96 ezer él maradt, és a gráf 99,7%-a egyetlen összefüggő komponens.
A jobb oldali ábra egy kis részletet mutat: a 12 legtöbbet szereplő hőst bal oldalon, és jobb oldalon azt a 12 képregényt, amelyben közülük a legtöbben együtt szerepelnek. A körök mérete a teljes gráfbeli szereplésszámot jelzi.
[Kattintás] 
Ez pedig a teljes gráf: egy nagy, sűrű mag, és körülötte a ritkán szereplő karakterek.

### 3. dia
A népszerűséget nem a gráfból mérem, hanem egy külső forrásból: a Comic Book Resources oldal olvasói szavazásaiból. Az olvasók rangsorolt listákat küldtek be, és ezekből pontszám készült.
2011-ben egy top 100-as, 2019-ben top 50-es lista van. 
[Kattintás] 
Azért jó mérce, mert független a gráftól: nem a szereplések számából készült. A listák néhány szereplője viszont 2001 után jelent meg, ők nincsenek a gráfban, ezért kimaradnak. A 2011-es listából 91, a 2019-esből 46 szereplőt találtam meg.

### 4. dia
A főkérdésem: van-e összefüggés a szereplők közkedveltsége és aközött, hogy hány képregényben szerepelnek?
[Kattintás] 
Ezt hat alkérdésre bontottam: egyeznek-e a legtöbbet szereplők a kedvencekkel, mennyire hasonlít a sorrend, segít-e, ha a kapcsolatokat is figyelembe vesszük, kik a kivételek, megmarad-e az eredmény más listán, és hogyan oszlanak meg a szereplések.
A következő diákon először a gráfot írom le, utána jön az összevetés.

### 5. dia
Az első elemzés a fokszám-eloszlás: megszámoltam, hány képregényben szerepel egy hős, és ezt egy ugyanakkora, de véletlenszerűen összekötött gráffal vetettem össze.
Az ábrán a piros vonal a Marvel-gráf, a szürke a véletlen gráf. A véletlenben senki sem szerepel 32-nél több képregényben, a valóságban a vonal egészen 1577-ig nyúlik, ez Spider-Man.
[Kattintás] 
A medián szereplő mindössze 3 képregényben jelenik meg, a szereplők felső 1%-a viszont az összes szereplés harmadát adja. Ez nem véletlen: néhány sztár uralja a gráfot. A véletlen gráf esetében ez a medián 15 lett.

### 6. dia
A második elemzés a centralitás. A „fontos szereplő” többféleképpen definiálható, ezért öt klasszikus metrikát használtam, a rövid magyarázatuk a fejlécekben látható
A fokszám a szereplések száma, a közelség azt méri, milyen közel van valaki mindenki máshoz, a köztiség, hogy mennyire köti össze a gráf részeit, a sajátvektor, hogy fontos szereplőkhöz kapcsolódik-e, a PageRank pedig egy véletlen bolyongáson alapuló arányszám.
[Kattintás] 
A lista elején a mértékek nagyjából egyetértenek: Spider-Man, Captain America és Iron Man szinte mindenhol elöl van. A sajátvektor-centralitás tér el: a csapatok sűrű magját emeli ki, ezért kerül előre a Fantasztikus Négyes.
Ezek a rangsorok lesznek a későbbi szavazással való összevetés alapjai.

### 7. dia
A harmadik elemzésben a képregénykódokból sorozatokat képeztem: például az ASM 22 az ASM, vagyis az Amazing Spider-Man sorozathoz tartozik. Így meg tudtam nézni, hány különböző sorozatban szerepel egy szereplő.
Az adathalmazomban 650 sorozatot találtam, és a szereplők 57%-a csak egyetlen sorozatban jelenik meg. A jobb oldalon a legszélesebb elérésűek láthatók, Spider-Man 147 sorozattal.
[Kattintás] 
Ők nagyjából ugyanazok, akik a legtöbbet szerepelnek. A sorozatok számát hatodik rangsorként szintén összevetettem a szavazással.

### 8. dia
A páros gráfban nincsenek háromszögek, ezért a szokásos klaszterezettség itt nem működik. Helyette a bipartit változatot és a redundanciát számoltam, amik azt mérik, hogy egy szereplő egy csapatban szerepel-e.
Viszonyításként egy konfigurációs modellt használtam: ez egy véletlen gráf, de ugyanannyi kapcsolattal mindenkinek. A Marvel-gráf értékei ennek többszörösei, tehát a csapatszerkezet valódi, nem a fokszámokból következik.
[Kattintás] 
A legtöbbet szereplők között a Fantasztikus Négyes redundanciája kiugró, ők szinte mindig együtt jelennek meg. Spider-Man és Hulk inkább szólószereplő a sorrend szerint, de ez nem teljesen azt jelenti, hogy nem csapatok részei. Ha nagyon sok csapatban szerepel az adott hős, akkor ez is magyarázhatja az alacsony redundanciát.

### 9. dia
Az összevetéshez két mutatót használtam. Az átfedés azt mutatja, hogy a kedvencek közül hányan vannak a gráf toplistáján. A Spearman-rangkorreláció pedig azt méri, mennyire egyezik a két sorrend.
Ahogy a táblázaton is látszik, kb 60%-os az átfedés, pl 2011-es top 40 listán szereplő hősökből 39 benne van a gráfban, akikből 23 a gráf 40 legtöbbet szerepelt hőse között is ott van. A véletlen választás esetében ez egy sem lenne, így a különbség kb százszoros. A korreláció közepes, 0,42 és 0,57 között van.
[Kattintás] 
Az ábra a 2011-es top 100-at mutatja. Pirossal a kivételek egyik csoportja: Squirrel Girl, Rocket Raccoon, Deadpool, akiket az olvasók szeretnek, de keveset szerepelnek. Sárgával a másik: Human Torch, Mister Fantastic vagy Mary Jane, akik sokat szerepelnek, de kevésbé kedveltek.

### 10. dia
Összefoglalva: van összefüggés, pozitív és közepes erősségű. A kedvencek többsége valóban sokat szerepel, de a szereplések száma nem határozza meg a népszerűséget, valamint még a sorrend is csak részben egyezik. 
[Kattintás] 
A kapcsolatokat figyelembe vevő mértékek sem javítanak érdemben. Kivételek persze vannak: MJ-hez hasonló mellékszereplők sokat szerepelnek, mégsem ők a kedvencek, Deadpool vagy Mordály pedig valószínűleg 2001 után lett népszerű.
[Kattintás] 
Az összefüggés a hosszabb listán és 2019-ben is megmarad, csak az erőssége változik. 
Végül a szereplések nagyon egyenlőtlenül és csapatokban oszlanak meg, ez az eltérések egy részét megmagyarázza.

### 11. dia
Röviden: a gráf megtalálja a kedvenceket, de a sorrendjüket csak részben adja vissza. Köszönöm a figyelmet.