# DIY Ohmleader
DIY Ohmleader

Előzmények:
Rendelkezem hálózatra visszatápláló napelemes rendszerrel, amelynek teljesítménye 4 kWp és egy Fronius Symo 4.5-3-M inverter képezi az alapját.
Én még Magyarországon beleestem abba a körbe, akik 10 évig éves szaldó elszámolással kötöttek csatlakozási szerződést a villamosenergia szolgáltatóval. A legutóbbi elszámoláskor (május 12-e) kb. 800 kWh visszatáplálással zárult az egyenleg és mint tudjuk kishazánkban 5 Ft/kWh áron vásárolta vissza a szolgáltató ezt a megtermelt energiát. (A szomszédnak, meg el tudja adni extrém esetben 70 Ft/kWh áron...) Úgy döntöttem, hogy a visszatáplálásom minimálisra fogom szorítani, ez elsősorban életmódváltással és másodsorban egy olyan fogyasztó hálózatba kapcsolását jelenti, amely az aktuális visszatáplált villamos energiát fel tudja használni. Erre egy hőszivattyú tökéletes választás lehet, de sajnos a tesztelések alkalmával a visszatáplált energia változása nem lekövethető egy hőszivattyúval. Nem mondom, hogy nem lehetetlen, de én jobbnak láttam egy fűtőszál szabályozás megvalósítást. A családunk energiafelhasználását figyelemmel kísérve számomra egyébént megdöbbentő módon a HMV készítés energiaigénye vetekedett a fűtésre fordított energia mennyiségével. 

Eszközök:
- HomeWisard P1 Meter (Villamos fogyasztásmérő olvasására; export / import energia)
- Fronius Symo inverter, Datameneger kártyával (napelemekkel)
- Inepro PRO380-Mod 100A MID (fűtőszál fogyasztásmérésére)
- 1 db 3P 20A főkapcssoló
- 1 db 3P 10A B kismegszakító (Pl.: Eaton PL7-B10/3)
- 4P 25A 30mA áramvédő kapcsoló (Pl.: Eaton PF7-25/4/003-A, hibaáram védelemre)
- 3 db HOYMK SSR-25 DA szilárdtest relé (fontos, hogy nullaátmenet triggerrel rendelkezzen!)
- 1 db 4 csatornás optocsatoló izolációs kártya (Pl.: HL-OI-VT-4-N bemenet: 3,3V; kimenet: 24V)
- 1 db 230VAC / 24VDC tápegység
- KinCony KC868-A2v3 ESP32-S3 vezérlőkártya (ezt különálló elemekből is össze lehet rakni)
- 3 db AC-1-es üzemmódban 20A kapcsolni képes mágneskapcsoló (Pl.: Eaton DILMP20)
- 3kW-os 3x230V-os csillagba kapcsol fűtőpatron
- Home Assistant

Nézzük egyenként, mi mire kell:
1. HomeWizard P1 Meter: ez az eszköz szolgáltatja a PID szabályozó visszacsatoló jelét, ez tulajdonképpen bármilyen fogyasztásmérő lehet, nem muszály a szolgáltató mérőjét használni, az általa szolgáltatott pillanatnyi teljesítmény adatot le lehet cserélni az aktuális mérő adatára. Itt ezt a Home Assistanton keresztül kapjuk meg.
2. Inepro PRO380-Mod 100A MID: ez is egy fogyasztásmérő, ez fogja mérni a fűtőpatron által fogyasztott villamos energiát. Ez nem vesz részt a szabályozásban, ez csak tájékoztató adatot küld számunkra.
3. 3P-ú 20A-es főkapcsoló: Ezzel az eszközzel tudjuk feszültség mentesíteni a berendezésünket.
4. 3P-ú B védelmi karakterisztikájú 10A-es kismegszakító: Ez a készülék felel az érintésvédelemért, túláram- és zárlatvédelemért.
5. 30mA-es áramvédő-kapcsoló: ez a jelenleg érvényben lévő magyar szabványokban lévő kiegészítő védelem. Ez nem alakalmas önmagában a villamos védelemre, ez csak a túláram- és zárlatvédelmi készülék melleti kiegészítő hibaáram védelem.
6. HOYMK SSR-25 DA szilárdtest relé: Ez az eszköz fogja végezni a fűtőszál teljesítmény vezérlését. Nagyon fontos, hogy az eszköz nullaátmenet triggerrel rendelkezzen. A triak és a fázishasítás módszer sajnos az inverter H-hídját károsíthatja, így egyáltalán nem ajánlott, sőt kerülendő!
7. 4 csatornás optocsatoló izolációs kártya: Ez az ESP kimeneteit illeszti az SSR-ek részére, elméletileg az ESP-t közvetlenül is rá lehet kötni az SSR-re, de jobbnak láttam egy optikai leválasztást és egy szintillesztést közbeiktatni.
8. Tápegység: az elektronikák és SSR meghajtására ez bármilyen a célnak megfeleő tápegység lehet.
9. KC868-A2v3 vezérlőkártya: ezt különálló elemekből is össze lehet építeni, én kifejezetten olyan eszközt kerestem, ami sz alábbi funkciókkal rendelkezik:
    - ESP32-es kontroller
    - 2db relékimenet
    - 2db leválasztott bemenet
    - vezetékes ethernet port (WiFi a lemezszekrény miatt nem opció)
    - RS-485 kommunikációs felület
    - szabadon használható PWM csapok min. 3 db
10. Mágneskapcsolók: olyan mágneskapcsolót válasszunk, amely AC-1 üzemmódban tudja kapcsolni a fűtőpatronokat. Tehát, ha azt látod, hogy AC-3 25A, az nem biztos, hogy megfelelő lesz!
Az első mágneskapcsoló a fő mágneskapcsoló, ezt vezéreljük, ill. kapcsoljuk le ha rendellenes üzemállapot van. Ez biztonsági kérdés. Szükség van olyan pl. kapilláriscsöves termosztátra, amyelyet a tartály hőmárőhövelyébe helyezünk és a beállított hőmérséklet elérésekor a mágneskapcsoló által a fűtőartonokat lekapcsolja a hálózatról.
A másik két mágneskapcsoló a HMV és puffertartályban lévő patronokat kapcsolja az SSR-k után. Ezek felelnek a patronok kiválasztásáért.
11. Home Assistant: Ez lesz a megjelenítő felületünk, itt mindenki saját maga létrehozhatja az ESP által szolgáltatott adatokat.
    Második funkciója, hogy egy pár érzékelő értékét is szolgáltatja az ESP számára:
    - P1 mérő adatai
    - HMV tartály hőmérséklete
    - Puffer tartály hőmérséklete
    Ha a tartály hőmérséklet adatai nem álnak rendelkezésre, akkor azokat pl. DS18B20 hőmérővel lehet helyetesíteni, természetesen ebben az esetben az ESP programját módosítani kell.

Figyelmeztetés!
Jelen projekt 3x230/400V 50Hz TN-S hálózatra készült!
Minden esetben tartsa be az oszágában évrényes szabványokat, jogszabályokat a villamos berendezések tervezésére, kivitelezésére, felülvizsgálatára vonatkozóan!
A szerző semmilyen jogi következményt nem vállal a hibás és nem megfelelő méretezésből és kivitelezésből származó balesetek, tűzesetek miatt!
Minden nemű a villamos hálózatra kapcsolt saját gyártmányú nem minősített berendezés hálózatra kapcsolása az Ön felelősége!

⚙️ 1. Működési módok

    A rendszer két dimenzióban kezeli a működést:
    
      A. Üzemmód (Auto / Kézi)
      Auto: PID szabályozó működik.
      Kézi: A kcel (%) alapján fix PWM-et ad ki.
    
    B. Fázisválasztás (Egy / Három)
      Egy: Mindhárom fázis külön PID alapján megy (power1 / power2 / power3).
      Három: A 3 fázis közös PID alapján kap egyforma PWM-et (power script).

🔥 2. Fűtés engedélyezése (évszak+napfény+fennmaradó energia)

    Két engedélyező binary_sensor:
    
    őszi–tavaszi (futas_engedelyezett_ev):
    Sept 15 – Dec 31
    Jan 1 – May 15
      - napközben
       fve > 0
    
    nyári (futas_engedelyezett_nyar):
      May 15 – Sept 15
      fve > 0
    
    Ha mindkettő false, az összes PWM letilt → ez kiváló biztonság.

📡 4. Mérések

    Modbusról jön:
     - pillanatnyi teljesítmény (össz + fázisonként)
     - energia (kWh)
    
    HomeAssistantból jön:
     - fogyasztásmérő 1–2–3 fázis + összes
     - tartályhőmérsékletek (HMV, puffer alsó/felső)
     - fennmaradó energia (PV → fűtés)

🎛 5. PID-ek működése
    PID cél: –100 W
    
    Tehát cél a 100 W export, hogy ne legyen visszatáplálás.
    Alap PID formula:
    - error = setpoint - measurement
    - kp = 0.5
    - ki = 0.01
    - kd = 0.1
    
    Integrál korlátozás:
    - 1f esetén ±1000
    - 3f esetén ±3000

🔌 6. Relélogika (mk1, mk2 interval)

   mk1 (Relé1):
   5 másodpercenként:  
    
    - ha felső tartályhőmérséklet < max → relé1 ON
    - vagy ha HMV < max → relé1 ON
    - különben OFF

  mk2 (Relé2):
  “HMV vagy puffer” váltás.

    Auto:
    - ha HMV >= cél → pufferre kapcsol
    - ha HMV < cél → HMV-re kapcsol
    
    Kézi:
    - Futés opció szerint választ (HMV vagy Puffer)

🧠 Összefoglaló – így működik a rendszer

    1️⃣ Először engedélyezi-e az évszak/napfény/fennmaradó energia?
        → ha nem, mindent letilt
    2️⃣ Auto vagy kézi üzem?
    3️⃣ Egy vagy három fázis?
    4️⃣ Melyik tartályba fűt? (HMV/puffer)
    5️⃣ PID fut → PWM beállítás → SSR vezérlés
    6️⃣ Relék 5s ciklusban váltanak HMV és puffer között
