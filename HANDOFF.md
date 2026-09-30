# AF Farm Planner – overdracht voor een nieuwe Claude-chat

> **Aan Claude:** lees dit hele bestand eerst. Het beschrijft een project dat al ~90 versies ver is.
> Antwoord de gebruiker in het **Nederlands**, informeel en kort. De UI van de planner is in het **Engels**.
> Ga verder waar we gebleven zijn; vraag niet opnieuw naar dingen die hieronder al vastliggen.

## 1. Wie en wat

- **Gebruiker:** Mike (mwillemsen112@gmail.com), Nederlands, speelt Call of War (CoW) in clan **AF**.
- **Game:** map 23114_3, *Europe Clash of Nations*.
- **Project:** de **AF Farm Planner**, één HTML-pagina die als Claude-artifact gepubliceerd is.
  - Toont de AI-startlegers van dag 1 (uit DTG).
  - Laat Mike routes, gevechten, patrouilles, productie, research en War Bonds plannen, met exacte speltijden op een tijdlijn.
- **Live link (artifact):** https://claude.ai/artifact/TqEeatkL24hPvuF33A39Ah
  - Laatste versie bij overdracht: **v98** (geproduceerde units komen altijd automatisch op de kaart; knop heet nu "🗺 Show on map").
  - Capabilities: `db` en `user` (farms opslaan in het account) en `downloads` (farm exporteren).
  - Bij opnieuw publiceren naar dezelfde link: eerst `Artifact` → `read` op die URL doen, dan publiceren met `url`. Laat `capabilities` weg, dan blijven ze behouden.
- **Bron-zip:** `af-farm-planner-src.zip` bevat alles om de pagina opnieuw te bouwen (zie §6).
  - Heb je de zip niet? Dan kun je de gepubliceerde HTML lezen via `Artifact read`. Dat is het volledige gebouwde bestand (~3,3 MB, data en de calculator ingebakken) en dat kun je direct bewerken.

## 2. Wat de planner nu kan (alles werkt en is getest)

### Kaart en weergave
- Scherpe provinciegrenzen en verbindingslijnen.
- Terrein-icoontjes (plains, hills, mountains, forest, urban), aan/uit te zetten.
- AI-units aan/uit. Filter "Show units of" per team of land.
- Optie: AI-legers buiten de twee teams grijs maken.

### Teams
- 5 eigen en 5 vijandige landen, alleen landen met 50 VP.
- Eigen team groen, vijand rood, provincies gekleurd.
- Veroverde provincies kleuren mee op de tijdlijn.
- Teamgenoten tellen als eigen grond: geen ×0,5, havens 3 uur, geen gevecht.

### Beweging
- **Land:**
  - Snelheid = unitsnelheid × terrein × 0,5 op vijandig land.
  - Een provincie is "van jou" zodra de stack het middelpunt haalt.
  - Konvooi over zee, in- en ontschepen 3 uur (eigen of team) of 4,5 uur (anders).
- **Schepen:** alleen zee- en havenverbindingen.
- **Lucht:** rechte lijn plus 30 min opstijgen en landen. Niet bij Rocket en Flying Bomb (die landen niet).
- **Waypoints:** tussenpunten, ook "Move option 2" (punten op de lijnen tussen provincies).
- **Dag-1-regel:** op dag 1 alleen door eigen en team-provincies. Anders wacht de stack aan de grens tot dag 2, 00:00. Uit te zetten onder Assumptions.

### Gevechten
- DTG-simulatormodel, rondes van 1 uur.
- Forced March (+50% snelheid, −5% HP per uur).
- Heal met War Bonds: 10% per klik, prijs 170 × reinforcement-gewicht.

### Range-units
- **Shoot:** de unit loopt tot de rand van zijn range-cirkel het doel raakt en schiet dan, elk half uur.
- Trekt units naar zich toe: altijd blijft er 1 staan; infantry eerst, nooit AC/AA.
- Een kruiser kan units de zee op trekken; die worden dan konvooi.

### Vliegtuigen
- Patrouilles met rode corridor (30° vanaf het vliegveld). Aanval elke 15 min; vliegtuigen en verdedigers doen elk 50% schade.
- "Add extra patrol" met wisseltijd.
- Landen op een Aircraft Carrier: onderscheppen, 30 min tanken, daarna is het schip hun vliegveld.

### Stacks
- Units plaatsen met **Place unit**.
- Split: splitst op de huidige plek op de tijdlijn.
- Stack-picker als units op elkaar staan.
- Een leger met een route kan geen dubbele route krijgen.

### Tijdlijn
- Dag vooruit en terug, afspelen met snelheden, Live-modus (echte klok).

### Farms (opslaan)
- Autosave per farm in het Claude-account (`db`), met localStorage als back-up.
- Meerdere farms: New, Open, Save as copy, Delete.
- Export en Import als `.json`.

### Productie
- Klik een provincie op de kaart (of **Pick province**) → pop-up **Build & produce**.
- Gebouwen per provincietype (stad/platteland/kust), met startlevel.
- Bouwwachtrij en productiewachtrij: 1 tegelijk per provincie.
- Unit-tijd = floor(buildTime / factory-factor van het gebouw), met factor L1=1, L2=1, L3=2, L4=4, L5=8.
- Moraal-straf in % instelbaar.
- **Put on map:** stack verschijnt pas als hij klaar is.

### Research
- Per land, instelbaar aantal slots (standaard 2).
- Per level: `dayAvailable`, en het vorige level moet klaar zijn.
- Waarden per doctrine.

### War Bonds
- Per land: start 10.000, +1.500 per uur (instelbaar).
- Heals en speed-ups (productie, bouw, research) gaan eraf op de tijdlijn. Rood bij tekort.

### Build calculator (tweede tabblad, sinds v88)
- Tabbladen rechtsboven: **🗺 Planner** en **🧮 Build calculator**. De calculator is Mikes eigen "AF Build Calculator" (React), zonder Firebase ingebouwd.
  - Mikes losse versie met Firebase bestaat nog apart; het originele bestand zit in `calc/original_calculator_index.html`. De WAW100-landen (Alberta, Northwest-USA, Colorado) zijn eruit gehaald.
- **Opslag:** de calculator-state (`countries`) staat in `S.calc` en wordt dus per farm opgeslagen.
  - Geen live samenwerking of presence meer.
  - Brug tussen de twee: `window.AFBridge` met `load`, `save`, `info`, `show`, `toPlanner`, `factory`, `cityHas`.
- **Productie:** per productieregel kies je **stad + dag + uur**.
  - `calcSync()` zet calculator-gebouwen en -units in de productiewachtrijen van de planner (`S.prod`, items met `src:'calc'`, `nb` = niet vóór dit uur).
  - Stadsnamen worden gekoppeld via `CITY_ALIAS`, bijvoorbeeld Köln → Cologne.
- **Alleen units uit bestaande fabrieken:**
  - Tank Plant → tanks en AC; Barracks → infantry; Ordnance Foundry → AT, AA en artillerie; Aircraft Factory → vliegtuigen; Naval Base → schepen; Secret Lab → secret units.
  - Dat geldt in de calculator (lijst en stadkeuze) en in de Build & produce-pop-up van de planner.
  - Een fabriek telt als hij daar al staat (startlevel) of in de wachtrij zit.
- **Status per regel:** bijvoorbeeld "on the map from …" of "joins the stack in Madrid at …". Het 🗺-knopje springt naar de kaart.

### Stacks en productie op de kaart (v90–v91)
- **Samenvoegen van productie:** `prodPlace()` zet geproduceerde units op de kaart.
  - **Landunits** sluiten aan bij de landstack van hetzelfde land in die stad: het AI-startleger (dan wordt er automatisch een legerroute gemaakt) of een eigen of eerder geproduceerde stack. Dat gebeurt via `host.adds` met een tijd, zodat het aantal op de tijdlijn meeloopt.
  - Een stack met een route vertrekt pas als de toegevoegde units klaar zijn.
  - **Schepen** (zoals kruisers) en **vliegtuigen** krijgen een eigen stack.
- **Eén stack per land per provincie:** stilstaande landstacks van hetzelfde land in dezelfde provincie worden altijd één stack.
  - Geldt ook voor Place unit en voor oude opgeslagen farms (`joinStack` en `joinAll`).
  - Alleen bewust gesplitste stacks (`r.split`) blijven apart.
- **Hogere unit-levels:** die hebben in de speldata een langere bouwtijd, bijvoorbeeld AC Axis L1 2u45m, L2 8u, L3 21u. Alleen het fabriekslevel verkort dat.

### Moraal op dag 1 (v94)
- Op dag 1 heeft elk land 70% moraal. Bouwen en produceren duren dan **12% langer** (door Mike zo ingesteld). Dit geldt voor alles wat vóór dag 2 00:00 start (`penAt` in `prodCompute`).
- **Uitzondering:** steden met de moraal-War Bond.
  - In de calculator is dat `city.warbond > 0` op de stadskaart.
  - In de planner is dat het vinkje in Build & produce (`pr.wbm`).
- Aan/uit via de instelling "Day 1: 70 % morale…" onder Assumptions.
- In de wachtrij staat dan "+12 % morale".

### Overig (v92–v93)
- Farmkeuze ook in de kopbalk.
- In de calculator kun je een land inklappen met **▾ Done**. De data blijft bewaard (`c.collapsed`).

### Prestaties (v97)
- **Dijkstra:** gebruikt nu een binary heap in plaats van een lineaire zoektocht.
- **Cache:** resultaten worden gecachet in `DJC`. De sleutel is start, route-eigenschappen, `ORIGIN`, teams, `kmf` en het dag-1-venster.
  - Het resultaat is gecontroleerd: 40 willekeurige routes gaven exact dezelfde tijden als v96 (`tests/q99.js`).
- **`variants()`:** wordt gememoiseerd (`VARC`).
- **Effect:** een route selecteren of klikken was ~290 ms per keer, nu ~50 ms.
- **Calculator:** `ResourceBar` crasht niet meer zonder kosten.
- Stippellijnen worden tijdens slepen doorgetrokken lijnen.
- Hertekenen alleen als er iets verandert.

## 3. Belangrijke spelregels en getallen (uit speldata en client-code)

### Snelheid en afstand
- Game-snelheid ×20 = km/u.
- Paginapixels ↔ kaarteenheden: `PX2UNIT = 1/(0.6*0.8077)`.
- Kaartschaal: 1/3 km per eenheid (`S.kmf`).

### Speed-ups
- Prijs = curve `pf` (resterende seconden → prijs, lineair ertussen, maximaal 12 u per speed-up; de planner rekent in blokken van 12 u) × kostfactor.
- Kostfactor: unit feature 71, gebouw feature 42, research 1.
- Curves in War Bonds (offers met `resourceId 22`):
  - productie 22674: 1u = 1.540, 12u = 8.250;
  - bouw 22675;
  - research 22676.

### Doctrine-indexen
- Doctrine 1 = Axis, 2 = Allies, 3 = Comintern, 4 = Pan-Asian.
- Research-ID's per tier komen in blokken: basis, Axis, Allies, Comintern, Pan-Asian. De planner neemt `v[S.doc]`.
- Allies: research ~25% sneller en goedkoper.

### Door Mike in het spel bevestigd
- Infantry research L1 en L2 (Allies):
  - **L1:** 1.450 Food, 300 Steel, 1.900 Money, 5 min.
  - **L2:** 1.550 Food, 350 Steel, 2.150 Money, 3u45m, vanaf dag 2, 4.511 War Bonds om meteen klaar te maken.
- Mike zei "alles klopt".

### Nog niet in het spel gecontroleerd
- Productie-speed-upprijs, bijvoorbeeld Allies Infantry L1 1u45m → 2.283 War Bonds.
- Speed-ups van meer dan 12 uur.

### Bouwtijden (L1…L5)
- Barracks, Tank Plant, Ordnance Foundry: 5 min, 4u, 12u, 24u, 32u.
- Aircraft Factory, Naval Base, Secret Lab: 30 min, 4u, 12u, 24u, 32u.
- Industry: 8u tot 32u.
- Infrastructure: 4, 8, 12u.

### Units zonder fabriek in de data
- Flame Tank, Amphibious Tank, Marines en Sniper staan niet in de productielijst.

## 4. Afspraken en grenzen (belangrijk!)

- **Geen verborgen data:** niet cheaten. Geen verborgen AI-legerposities of fog-of-war-data ophalen. Een eerdere test daarmee is teruggedraaid; Mike was het ermee eens dat dat valsspelen is.
- **Wachtwoorden:** nooit typen. COW_USERNAME en COW_PASSWORD staan alleen als GitHub-secrets.
- **Websites:** static1.bytro.com en www.callofwar.com niet in de browser openen. Mike heeft dat geweigerd.
- **PC van Mike:**
  - bestanden verwijderen mag alleen in de map `cowparser`;
  - geen git-commando's op zijn PC (index.lock-probleem).
- **Workflow:**
  - na elke wijziging bouwen;
  - testen met Playwright (headless Chromium);
  - publiceren naar dezelfde artifact-link;
  - kort in het Nederlands uitleggen wat er veranderd is.
- **Kopie op de PC:** `C:\Users\Gebruiker\Downloads\cowparser-ready\cowparser\farm-planner\farm_planner.html` staat nog op **v69**. Bijwerken als Mikes PC gekoppeld is.

## 5. Open punten en ideeën

- **AI-bewegingen:** Mike wilde de "regels" voor hoe de AI beweegt nog aanleveren. Die zitten niet in de client-code.
- **Unit-codes** zoals in CoW ("E1", "E2"): aangeboden, Mike heeft nog niet gekozen.
- **Research ↔ productie:** waarschuwen als je een unit-level produceert dat nog niet onderzocht is. Aangeboden. De calculator voegt research al automatisch toe, maar die komt nog niet in het Research-blok van de planner.
- **Vliegtuigen bij de landstack:** nu krijgen ze een eigen stack. Aan Mike gevraagd of ze bij de landstack moeten; nog geen antwoord.
- **Doctrine per land:** in het Research-blok (nu één globale doctrine bovenaan). Aangeboden.
- **Speed-ups verifiëren** met een screenshot uit het spel: prijs bij bijvoorbeeld 2u30m resterend, en bij meer dan 12 uur.

## 6. Techniek (voor als je gaat bouwen)

### Bestanden in de zip
- `template.html`: alle UI, CSS en JS.
  - Eén script, één IIFE.
  - Data via `__DATA__`, afbeeldingen via `__BASE__`, `__LOGO__` en `__WB__`.
- `build.py`: bakt de data in de template.
  - Invoer: `data/farm_data.json`, `game_graph2.json`, `dtg_cow_units.json`, `premium_state.json`, `map_geometry.json`, `icons/*.png`, `base_live.jpg` en `logo.png`.
  - Uitvoer: `farm_planner.html`.
  - Draai `python3 build.py` (nodig: Pillow).
- `calc/`: de calculator.
  - `calc_app.jsx` is de bron. `compile.js` compileert die met @babel/standalone naar `calc_app.js`.
  - Verder `calc_scoped.css` (calculator-CSS, gescoped onder `#calcView`) en de React/ReactDOM UMD-bestanden.
  - `build.py` bakt dit allemaal in.
- `tests/q*.js`: Playwright-tests.
  - Serveer de map met `python3 -m http.server 8765` en draai `NODE_PATH=$(npm root -g) node tests/q70.js`.
  - Sommige oudere tests lezen `farm_planner.html` met `setContent`.

### Belangrijke globals en functies in template.html
- **Globals:**
  - `S` (instellingen en state, wordt opgeslagen);
  - `routes`, `act`, `TNOW` (speluren sinds dag 1 00:00);
  - `TRACKS`, `MOVPOS`, `CONQ`, `hidden`.
- **Route:** `computeRoute(r)` gebruikt `edgeW`, `edgeH` en `dijkstra(src,r,t0)` (tijdafhankelijk vanwege de dag-1-regel). Verder `fightsOnRoute`, `buildTrack` en `trackAt`.
- **Tekenen:** `drawMovers`, `renderRoutes` (meerdere keren gewrapt: ranges, versie-bump, WB-herberekening) en `drawBar` (routebalk-knoppen).
- **Range en lucht:** `patrolInfo`, `carrierLanding`, `rangedInfo`, `moveIntoRange`.
- **Stacks:** `splitRoute`, `showSplit`, `planFromArmy`, `showArmy`, `showPicker`, `click` (kaartklik), `openProvPop` (bouw-pop-up).
- **Productie:** `bldHere`, `prodCompute`, `renderProd` (doel `PRODBOX` = 'prod' of 'pop'), `prodSync` (koppelt geproduceerde stacks via `r.born`).
- **Research:** `rsCompute`, `renderRS`.
- **War Bonds:** `spPrice(kind,sec)` (kind: 'u', 'b' of 'r'), `wbEvents`, `wbBal`, `renderWB`.
- **Farms:** `farmSnap`, `farmWrite`, `farmApply`. Db-paden: `data/users/<uid>/f_<farmId>` en `data/users/<uid>/last`.

### Testhooks
- `window.__farmSim`: fighter, simulate, computeRoute, active, dbg, enz.
- `window.__farmT.set(uur)`: tijdlijn zetten.

### Artifact-regels
- Geen externe scripts.
- Browseropslag altijd in try/catch (`store` helper).
- Donker thema met CSS-variabelen.
