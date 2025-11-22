# DIY Ohmleader
DIY Ohmleader

Előzmények:
Rendelkezem hálózatra visszatápláló napelemes rendszerrel, amelynek teljesítménye 4 kWp és egy Fronius Symo 4.5-3-M inverter képezi az alapját.
Én még Magyarországon beleestem abba a körbe, akik 10 évig éves szaldó elszámolással kötöttek csatlakozási szerződést a villamosenergia szolgáltatóval. A legutóbbi elszámoláskor (május 12-e) kb. 800 kWh visszatáplálással zárult az egyenleg és mint tudjuk kishazánkban 5 Ft/kWh áron vásárolta vissza a szolgáltató ezt a megtermelt energiát. (A szomszédnak, meg el tudja adni extrém esetben 70 Ft/kWh áron...) Úgy döntöttem, hogy a visszatáplálásom minimálisra fogom szorítani, ez elsősorban életmódváltással és másodsorban egy olyan fogyasztó hálózatba kapcsolását jelenti, amely az aktuális visszatáplált villamos energiát fel tudja használni. Erre egy hőszivattyú tökéletes választás lehet, de sajnos a tesztelések alkalmával a visszatáplált energia változása nem lekövethető egy hőszivattyúval. Nem mondom, hogy nem lehetetlen, de én jobbnak láttam egy fűtőszál szabályozás megvalósítást. A családunk energiafelhasználását figyelemmel kísérve számomra egyébént megdöbbentő módon a HMV készítés energiaigénye vetekedett a fűtésre fordított energia mennyiségével. 

Eszközök:
- HomeWisard P1 Meter (Villamos fogyasztásmérő olvasására; export / import energia)
- Fronius Symo inverter, Datameneger kártyával (napelemekkel)
- Inepro PRO380-Mod 100A MID (fűtőszál fogyasztásmérésére) 
- 4P 25A 30mA áramvédő kapcsoló (Pl.: Eaton PF7-25/4/003-A, hibaáram védelemre)
- 3 db HOYMK SSR-25 DA szilárdtest relé (fontos, hogy nullaátmenet triggerrel rendelkezzen!)
- 1 db 4 csatornás optocsatoló izolációs kártya (Pl.: HL-OI-VT-4-P bemenet: 3,3V; kimenet: 24V)
- 1 db 230VAC / 24VDC tápegység
- KinCony KC868-A2v3 ESP32-S3 vezérlőkártya (ezt különálló elemekből is össze lehet rakni)
- 3 db AC-1-es őzemmódban 25A kapcsolni képes mágneskapcsoló (Pl.: Eaton DILMP20)

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
