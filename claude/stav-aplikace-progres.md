# Aplikace „Progres" — stav projektu

**Majitel:** Karel · **Cíl:** iOS/web appka na záznam cvičení, tělesných měr, kalorií a vizualizaci progresu. Zatím pro sebe, App Store možná později.

## 🌐 NASAZENO ŽIVĚ
- **URL:** https://g4m3ch4n63r.github.io/progres/
- **GitHub repo:** https://github.com/G4M3CH4N63R/progres (public, GitHub Pages z větve main, / root). Účet: **G4M3CH4N63R**. **Od 31. 7. 2026 se pushuje přes SSH z Karlova Macu** (remote `git@github.com:G4M3CH4N63R/progres.git`, klíč `karel-macbook`). Stará cesta přes Claude in Chrome je **opuštěná** — viz „Pracovní prostředí" níž.
- Karel má appku na iPhonu (PWA), aktivně testuje a iteruje. Má vlastní Anthropic API klíč s kreditem. **Uživatelé (stav v35): Karel, jeho bratr a jeden kolega** — tři lidé na sdíleném Workeru se stropem $10/měsíc.
- **Nasazení (od 31. 7. 2026, z Macu):** zkopírovat build → bumpnout SW → commit → push.
  ```bash
  cp progres.html repo/index.html
  # v repo/sw.js bumpnout const C na progres-vXX.0 (VLASTNÍ řada, ne číslo z Designu)
  cd repo && git add -A && git commit -m "vXX: popis + SW vXX" && git push
  ```
  Pages přebuildí za ~40 s. **Deploy vyžaduje Karlův výslovný souhlas** — ptát se prostým textem česky, ne přes AskUserQuestion (popup se Karlovi nezobrazuje).
  - 📜 **Historie:** do v38 se nahrávalo přes Claude in Chrome do GitHub UI. Celá sada potíží s tím spojená (klik do pole commit message často minul, minul i klik na „Commit changes", `computer type` psal diakritiku nespolehlivě → commit message se od v30 psaly bez háčků) **je tímhle vyřešená a už neplatí**. Diakritika v commit message je nově v pořádku; kvůli konzistenci se starou historií se ale drží bez ní.
  - ⚠️ **Živé ověření: fetch s query stringem bývá blokovaný.** Cache-busting přes `fetch('sw.js?x='+Date.now())` skončí hláškou „BLOCKED: Cookie/query string data" — použít `fetch('sw.js', {cache:'reload'})`. A pozor: **první načtení `?nocache=vXX` může ještě běžet ze staré SW cache**. Ve v33 to trvalo **dvě** další navigace; ve v34–v38 stačila jedna. Když `swVersion` sedí a funkce ne, prostě navigovat znovu s dalším parametrem.
  - ⚠️ **`String(asstApi)` na živém webu vrací SHIM, ne appku (poučení z v37).** Shim obaluje `window.asstApi` vlastní funkcí (`own(...)`), takže kontrola typu `String(asstApi).indexOf('…')` na živém webu hlásí falešné „chybí". Obsah nasazené appky se ověřuje z **HTML**: `fetch('index.html',{cache:'reload'})` a hledat markery v textu. (Totéž platí pro `aiCall`, `aiFoodSearch`, `save`, `download`, `renderApiKeyBox` — všechno obalené funkce.)
  - 🚨 **Po každé změně těla API požadavku asistenta: JEDNO SKUTEČNÉ volání po nasazení (poučení z v38 — draze).** Mock testy podvrhují `fetch`, takže **nikdy nechytí validaci na straně Anthropicu** — v37 prošlo všech 26 sad a živě pak KAŽDÁ zpráva asistenta padala na `400: temperature is deprecated for this model`. Ověření: na živém webu přes konzoli zavolat `await asstApi([{role:'user',content:[{type:'text',text:'Odpověz přesně jedním slovem: OK. Nic nezapisuj, nevolej žádný nástroj.'}]}])` — nic to nezapíše do chatu ani dat, stojí to pár centů (v38: 7 170 vstupních tokenů) a je to jediný způsob, jak ověřit reálné API. Výsledek zapsat sem.
- Na iPhonu se aktualizuje samo (auto-update SW). Když ne, appku úplně zavřít (app switcher) a otevřít.
- Data per zařízení/prohlížeč (localStorage `progres_data_v1` + IndexedDB fotky). Přenos přes Zálohu → Import (v Nastavení). **Od v31 se API klíč do zálohy nezapisuje** — klíč patří zařízení, ne záloze. **Od v32 se na iPhonu záloha nabízí jako `.txt`** (iOS sdílení `.json` nedovolí), obsah je stejný a Import bere obojí. **Od v33 se smazané tréninky a jídla 14 dní drží v archivu**, takže omylem smazaný záznam už není ztráta. **Od v34 je tu AI asistent** — historie chatu leží zvlášť v `progres_chat_v1` a do zálohy se **nezahrnuje** (data ano, konverzace ne). **Od v35 je celé tohle vysvětlené přímo v appce** — karta „Soukromí a data" v Nastavení pod Zálohou. **Od v36 je konverzací víc** (`progres_chats_v1`, až 30 vláken) a **připomínka zálohy chodí po 5 dnech** místo 14. **Od v37 asistent nikdy nepočítá součty z hlavy** — dostává je hotové z appky. **Od v38 se editované pole drží ve viditelné části obrazovky** (fix iOS klávesnice).

## 🛠️ Pracovní prostředí a skilly

### Kde se pracuje (od 31. 7. 2026)
**Claude Code na Karlově Macu**, složka `~/Desktop/Claude Code/Progres App/`. Repo je trvale na disku, prostředí se **neobnovuje** — to byla dřív povinná úvodní minuta každé session a je pryč.

```
CLAUDE.md            invarianty projektu, Claude Code je čte automaticky
progres.src.html     ZDROJ — jediné místo k editaci (371 832 B)
proxy_shim.js        vrstva sdílené AI, mimo zdroj (10 162 B / 10 098 znaků)
build.py             --src / --out / --no-shim
progres.html         BUILD, needitovat
cd_upload38.html     nasazený build BEZ shimu = báze pro příští diff z Designu
                     955 587 B, MD5 0c89cd6c7f6c955fbb919a3ad4bfae9d
vendor/              chart.js + html5-qrcode (583 882 B) — V GITU
node_modules/        jen npm (Playwright); knihovny buildu sem NEPATRI
tools/check_cut.py   nezávislý oracle: složí index.html bez build.py
repo/                klon z GitHubu = pracovní klon i deploy cíl
claude/              tato dokumentace
.claude/skills/      skilly projektu
```

### 📚 Co vidí projekt na claude.ai (přeuspořádáno 1. 8. 2026)

Projekt je napojený na **veřejné** repo `progres` (nasazený `index.html`, `sw.js`)
a nově i na **`claude/`** — dokumentace se do veřejného repa zrcadlí z toolchainu
(`cp claude/*.md repo/claude/`). Konverzace v projektu tedy vidí aktuální stav
projektu, aniž by Karel cokoli nahrával ručně.

🔴 **Privátní `progres-toolchain` se do projektu připojit NEPODAŘILO.** V dialogu
„Add content from GitHub" se v seznamu vůbec nenabídne, i když má GitHub App
udělený plný přístup k repu. Vyzkoušeno: konfigurace GitHub App, přepnutí na
*All repositories*, reautorizace konektoru — nic. Je to **známá chyba indexace
na straně backendu Anthropicu** (anthropics/claude-code#33875, #79083, #12839;
#33875 zavřen jako „not planned"), veřejný workaround neexistuje. Dokumentace
proto jede oklikou přes veřejné repo. Zdroj (`progres.src.html`) v projektu
tedy **není** — čte se z disku v Cowork session s připojenou složkou.

✅ **Zrcadlo hlídají git hooky** (`tools/hooks/`, zapnuté přes
`git config core.hooksPath tools/hooks`). `pre-commit` zkontroluje `claude/`
na tajemství, zkopíruje změny do `repo/claude/` a commitne je; `pre-push`
zrcadlo odešle dřív než toolchain a při selhání zastaví i ten. Ruční `cp`
tedy odpadá. Bez hooků (cizí klon, `--no-verify`) se zrcadlo **rozchází tiše** —
projekt čte starou verzi a nic nehlásí.

⚠️ **Většina obsahu ze syncu je balast** — z 967 KB `index.html` je 583 KB
minifikovaných knihoven, které konkurují ve výsledcích hledání. Dotazy
formulovat konkrétně (jméno funkce, ne obecné téma). A je to **build se shimem**,
ne zdroj: co je ve zdroji a co v shimu, se pozná jen odsud.

### ✅ Kontrola na začátku session (nahradila obnovu báze)
```bash
git -C repo pull
python3 build.py && cmp -s progres.html repo/index.html && echo "báze OK"
```
Musí projít. Když ne, **zastav se** — pracuješ nad něčím jiným, než co je nasazené.

### Dva skilly + CLAUDE.md
- **`CLAUDE.md`** (kořen projektu) — tvrdá pravidla a pasti. Načítá se samo, netriggeruje se.
- **`uprava-progres`** — Karel popíše změnu **slovy** nebo pošle screenshot chyby. Dnes **běžný případ**.
- **`nasazeni-progres-z-design`** — Karel nahraje `index.html` z Claude Design. Dnes **vzácnější**; Design se vyplatí hlavně na vizuální práci (redesign sekce, vzhled karet, barvy). Za to se platí bajtovým rituálem a rizikem, že export smaže vlastní úpravy.

Skilly leží v `.claude/skills/` **v repu**, ne v účtu — verzují se s kódem a nemůžou se rozejít se stavem projektu. (Staré omezení „lokální zápis do `/root/.claude/skills/` se přepíše" se týkalo efemérního kontejneru a už neplatí.)
⚠️ Název skillu **nesmí obsahovat slovo „claude"**. Pole `description` má limit **1024 znaků** a **nesmí obsahovat lomené závorky** — validátor je čte jako XML tagy (`cd_upload<N>.html` neprojde, `cd_upload_N.html` ano).

### Lokální náhled
```bash
python3 -m http.server 8000   # -> localhost:8000/progres.html
```
Plnohodnotné prostředí: localStorage, IndexedDB, všechno — `make_preview36.py` s paměťovou náhradou localStorage už není potřeba. **Z iPhonu na stejné wifi přes IP Macu** se dá poprvé testovat chování iOS klávesnice před nasazením (dřív šlo ověřit až živě, viz v38).
⚠️ Service worker přes `http://` z jiného zařízení nenaběhne (chce HTTPS nebo localhost) — pro UI a klávesnici to nevadí, pro offline chování ano.

### Git jako pojistka
Commit před změnou i po ní, `git diff` ukáže co se stalo, `git revert` vrátí. **Tohle nahradilo bajtovou kontrolu jako hlavní ochranu proti ztrátě práce** — chrání i proti chybám, které bajtová shoda nezachytí. Experimenty do větve.

### Co zaniklo
`deploy/` a `/mnt/user-data/outputs/progres-web/` existovaly jen kvůli nahrávání do GitHub UI. S `git push` jsou **zbytečné** — build jde rovnou do `repo/index.html`.

**Vlastní opravy nad rámec exportu:** když Karel schválí opravu nebo funkci, kterou Design neposlal (viz fix v28, funkce v30, bezpečnostní oprava v31, sdílení v32, obnova + tři opravy ve v33, dvě opravy ve v34, karta Soukromí a data ve v35, celé várky v36, v37 a v38), postup je: nejdřív ověřit bajtovou shodu s exportem, pak teprve patchnout zdroj, znovu buildnout, spustit testy a do dokumentu zapsat, o kolik se nasazený build liší od schváleného exportu (a proč). Příští diff se pak dělá proti **nasazenému buildu bez shimu** (`cd_upload38.html`).

⚠️ **Když várka NEJDE z Designu (v36, v37, v38), bajtová kontrola odpadá — a s ní hlavní pojistka.** Není proti čemu diffovat, takže jistotu musí nahradit něco jiného: (1) **klikací náhled pro Karla ke schválení** (u čistě „neviditelných" změn jako v37/v38 stačí podrobný popis + testy), (2) plná regrese **dvakrát**, (3) u každé odstraněné funkce `grep` na nulový počet zbylých referencí, (4) ověřit, že sdílené CSS/JS, které mazaná část používala, potřebuje ještě někdo jiný (ve v36 `.ring-card` zůstalo kvůli `#bodyOverview` na Tělu), (5) **u změn API požadavku navíc jedno skutečné volání po nasazení** (viz 🚨 výše). Od 31. 7. 2026 přibyla šestá: **git** — commit před změnou a po ní.

🚨 **Design nevidí vlastní opravy — každý export je může vrátit zpět.** Potvrdilo se to ve v33 v plném rozsahu: export z Designu **smazal celý zálohovací systém z v30–v32** (13 míst) — protože o nich Design neví, žijí jen ve zdroji. Postup je vždy stejný: **nejdřív ověřit bajtovou shodu čistého portu s exportem, teprve pak vrátit své úpravy zpět**. Kontrolní seznam vlastních úprav, které musí přežít každou várku, je v sekcích v28–v38 níž. **Po v36–v38 je toho hodně** — historie chatu, archiv za rozklikem, `BACKUP_WARN_DAYS=5`, vymazané dlaždice Přehledu, **celý přesný počtový aparát asistenta z v37** (`asstDayTotals`, `_den`, `uprav_jidlo`, `dnes`/`souhrn` v `dotaz_data`, retry, pause_turn, pravidla v promptu) a **klávesnicová vrstva z v38** (`kbReveal`, `--kb` v `.chat-bar` a `.modal`) jsou věci, o kterých Design neví. A **POZOR: kdyby export vrátil `temperature` do `asstApi`, znovu ho odstranit — API na něj hází 400.**
✅ **Ve v34 ani v35 se to nestalo** — Design v obou případech stavěl na nasazeném buildu, takže všech 13 obnovených míst i `max_uses:2` přežilo. Neznamená to, že se pravidlo ruší; znamená to, že kontrolu je nutné dělat pokaždé, protože oba scénáře reálně nastaly.

⚠️ **`build.py --no-shim` přepisuje `progres.html`, když mu neřekneš jinak.** Kontrolní build patří do `/tmp` přes `--out`. Když si nejsi jistý, po každém kontrolním buildu **znovu spusť `python3 build.py`** (bez přepínačů) a ověř MD5 proti `repo/index.html`. Taky pozor: **`build.py` tiskne počet znaků, `wc -c` bajty** (v38: 959 809 znaků = 965 769 bajtů; a JS `.length` na živém webu dá ještě třetí číslo — UTF-16 jednotky, emoji za 2). Build je deterministický, dva běhy za sebou dají stejné MD5; když se čísla „nesejdou", je to skoro vždycky znaky vs. bajty. **Shim = 10 162 B / 10 098 znaků** — rozdíl 64 je česká diakritika v hláškách shimu, ne chyba měření.

### 📜 Obnova pracovní báze — historie (od 31. 7. 2026 už není potřeba)
Do 31. 7. 2026 byl kontejner efemérní a `progres.src.html`, `build.py`, `deploy/`, `node_modules/` i testy v nové session **neexistovaly** — musely se obnovit z gitu (~1 minuta). S Claude Code na Macu to odpadlo; zůstává jen kontrola `git pull` + `cmp` na začátku session (viz výš).

> ⛔ **Poučení z v29 (drahé), platí dál v jiné podobě:** ve v27 se tenhle krok přeskočil, protože v kontejneru „něco bylo" — jenže to byla **stará báze v26**, zatímco repo bylo na v28. Commit `298c203` tím přepsal živý web a **smazal dvě verze Karlovy práce**. Opraveno až commitem `8f15684` (v29). **Přítomnost souborů na disku NENÍ důkaz, že jsou aktuální** — proto ta úvodní kontrola. Nově navíc existují dvě možné cesty do repa, takže repo může být napřed.

**Rekonstrukce toolchainu** (kdyby se někdy dělala znovu, ověřeno 31. 7. 2026 — `cmp` prošel napoprvé):
1. Ze `repo/index.html` odstřihnout koncový shim blok (script před `</body>`, obsahuje `__proxy__` a `workers.dev`) → uložit bez obalu jako `proxy_shim.js`.
2. Vyříznout dva inline `<script>` bloky knihoven (Chart.js, html5-qrcode) do `node_modules/`, **odzadu dopředu** ať nesjedou offsety; na jejich místo CDN placeholdery (řádky 14–15: `cdn.jsdelivr.net/npm/chart.js@4.5.1/dist/chart.umd.js`, `unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js`) → `progres.src.html`.
3. U všech tří souborů se ustřihne jeden koncový `\n`, který přidal build. Vkládá se jako `<script>\n` + obsah + `\n</script>\n`.
4. `build.py` aplikuje escape `</script` → `<\/script` (na současných knihovnách no-op, ale příští výměna je krytá).
5. Offset zdroj↔build je **−18 řádků** (patch hunky sednou).

⚠️ **Nejdřív ověř řez, pak teprve piš build.** `tools/check_cut.py` složí `index.html` **bez** `build.py` a porovná bajt po bajtu — tím se oddělí „řízl jsem správně?" od „napsal jsem build.py správně?". Bez toho nevíš, která z těch dvou neznámých je špatně.

Playwright se na Macu instaluje lokálně (`npm i -D playwright && npx playwright install chromium`) — stará hlavička testů s `createRequire` a absolutní cestou do `/home/claude/.npm-global` **už neplatí**, stačí obyčejný import. ⚠️ **Testy zatím na Macu NEJSOU** (stav k 31. 7. 2026), přenášejí se postupně. Nejdřív **runner**, který umí obě konvence hlášení — bez něj vyjde falešné „errors=NA" u sedmi sad. Pak podle toho, co by tiše rozbité stálo nejvíc: **1) `test_v38.mjs`** (temperature), **2) `test_backup31.mjs`** (klíč v záloze), **3) `test_proxy27.mjs`** (sentinel), **4) `test_regress29.mjs`** (obecná regrese). Zbytek postupně. Seznam 27 sad, které existovaly v kontejneru: `test_regress29.mjs` (obecná regrese), `test_v28_live.mjs` (rozpis ingrediencí přes reálné UI; v souhrnném běhu se přeskakuje — chodí na živý web), `test_proxy27.mjs` (10 kontrol sdílené AI), `test_backup30.mjs` (13 kontrol zálohy), `test_backup31.mjs` (10 kontrol, že se klíč nikam nepropíše), `test_share32.mjs` (10 kontrol sdílení zálohy na iOS), `test_v45.mjs` (funkce z v33), `test_asistent.mjs` + `test_asist2.mjs` (asistent), `test_v35.mjs` (10 bloků), `test_v36.mjs` (17 bloků: historie chatu, archiv za rozklikem, Záloha, práh 5 dnů, Přehled bez dlaždic), **`test_v37.mjs` (11 bloků A–K: `asstDayTotals`, `_den` u zápisu/smazání, `dotaz_data` dnes+souhrn, pravidla v promptu, end-to-end tři jídla v jednom kole bez mezisoučtů, `uprav_jidlo`, retry po 529, pause_turn, ABSENCE temperature v těle, počet záznamů bez bílkovin)**, **`test_v38.mjs` (7 bloků A–G: tělo requestu bez `temperature:`/`top_p`, `kbReveal` existuje, `var(--kb)` v `.chat-bar` i `.modal`, fokus pole dole na stránce ho do 650 ms přiscrolluje do horních 55 %, žádný zbytečný skok u už viditelného pole, fokus chat lišty nescrolluje, `--kb` bez klávesnice prázdné)**.

⚠️ **Pozor na id — od v35 se dost změnilo:**
- **Záložka Progres (`tab-data`) NEEXISTUJE.** Navigace je `prehled / trenink / telo / kalorie / asistent`. Statistiky se vtahují do sekcí, archiv sedí u karet.
- **Asistent JE záložka** (`nav button[data-tab="asistent"]`), tlačítko `#asstBtn` v hlavičce **je pryč**. `openAsst()`/`closeAsst()` pořád fungují.
- **`switchTab(name, mode)` má druhý parametr** — `switchTab('x','body')` vykreslí sekci se všemi kartami (bez rozcestníku). `showSolo(sec, '#card-…')` zavrtá do jedné karty; **samo o sobě nestačí ani na screenshoty — bez `switchTab()` před ním zůstane vidět původní sekce**.
- **Statistické karty žijí ve skrytém `#statsPool`** (17 karet) a `showStats(tab)` je do sekce půjčí jako `.solocard` (trénink 6 / tělo 5 / kalorie 8); při odchodu se vrátí do poolu.
- **Nastavení není záložka** — overlay (`openSettings()` / `closeSettings()` / `renderSettings()`).
- **`#themeBtn` má `display:none`** — v testech přepínat přes `toggleTheme()` / `applyTheme('dark')`.
- Karty: `#card-strength/-templates/-newworkout/-foodlog/-favfoods/-foodhist/-today/-ai/-aiplan/-trhist/-steps/-bodyrec/-measures/-bodyhist/-photos` + od v35 `#card-privacy`, `#card-macrosplit/-macrotime/-protein/-fooddow/-fooddays/-topfoods`.
- `renderPlanBox()` a `renderAiBox()` vyžadují `state.profile.apiKey` (od v29 ho zajišťuje sentinel).
- **Od v36:** vlákna chatu v panelu Historie jsou `[data-chat]` (ne `data-chatgo`), přejmenování `[data-chatren]`, smazání `[data-chatdel]`; swipe karty na Přehledu jsou `.swipe-card` (ne `.tile`/`.card`); bubliny chatu `.cmsg` / `.cact`; archiv v Nastavení je `<details id="archAll">` se `<summary id="archAllSum">`.

⚠️ **Zastaralé testy:** `test_func.mjs` (z v15, před rozcestníkem — padá na `#weighPlus`) a `test_redesign.mjs` (padá na `#themeBtn`, který má v obou verzích `display:none`). Ověřeno, že padají **stejně i na nasazené předchozí verzi**, takže o regresi nejde.

⚠️ **Sady hlásí výsledek dvěma způsoby.** Většina tiskne JSON s `"errors": []`, ale `test_backup30/31`, `test_proxy27`, `test_regress29`, `test_share32`, `test_v28_live` a `test_v45` hlásí `CHYBY: N`. Souhrnný běh musí kontrolovat **obě** konvence (plus zbloudilé `PAGEERROR:` / `CONSOLE:` řádky), jinak vyjde falešné „errors=NA" u sedmi sad.

⚠️ **Pozor na falešně procházející testy (poučení z v33).** Chybu v promptu, kterou přinesl export z Designu, regrese **nechytila**, protože v testu byly dvě chyby najednou. Když test kontroluje nepřítomnost něčeho, ověř i **pozitivní** kontrolu. Správně: `JSON.stringify(planContent("", false, true))`.

⚠️ **Mock testy NIKDY nechytí validaci na straně API (poučení z v38).** Podvržený `fetch` odpoví na cokoli — parametr, který server odmítá (`temperature` ve v37), projde všemi sadami a spadne až živě. Proto: každá změna **těla requestu** (`asstApi`, `aiCall`, `aiFoodSearch`) = po nasazení jedno skutečné volání, viz 🚨 v sekci Nasazeno živě.

⚠️ **Mock API musí odpovídat podle POŘADÍ, ne podle času (poučení z v34).** První verze testu asistenta vracela stejnou `tool_use` odpověď „dokud neuplyne 600 ms", jenže všech 8 kol smyčky nástrojů proběhlo dřív → test hlásil 8 zapsaných jídel místo jednoho. Správně je fronta odpovědí (`queue.shift()` nebo počítadlo requestů), viz `test_asist2.mjs` a `test_v37.mjs`.

⚠️ **Nekontroluj tvar návratové hodnoty od oka (poučení z v35).** `foodDays(cut)` **vrací OBJEKT `{ "YYYY-MM-DD": {kcal,p,c,f} }`, ne pole** — test na `Array.isArray` padal, přestože appka byla v pořádku. Stejně tak: canvas grafů jídla se jmenuje **`chDow`**, ne `chFoodDow` (to je jméno *funkce*), a tabulka top jídel je **`#topFoodsTable`**, ne `#card-topfoods .row`. Než se prohlásí regrese, otevřít zdroj a přečíst si tu funkci. **Ve v36 to samé u selektorů** — `[data-chatgo]` a `.tile` byly vymyšlené, v kódu jsou `[data-chat]` a `.swipe-card`. **Ve v38 pozor na komentáře v kódu:** kontrola `String(fn).indexOf('temperature')` chytí i KOMENTÁŘ o temperature — testovat vzor `temperature\s*:` (skutečná vlastnost), ne jen slovo.

⚠️ **Zavřený `<details>` má v Chromiu nenulovou výšku (v36).** `offsetHeight > 0` tedy neznamená „je otevřený". Správné kontroly: `details.offsetHeight <= summary.offsetHeight + 4` a `box.checkVisibility() === false`.

⚠️ **`#backupNudge` sedí v `#tab-prehled`, ne v Nastavení.** Test na připomínku zálohy musí nejdřív `closeSettings(); switchTab('prehled')`, jinak se nic nezobrazí. A stav je v **`state.lastBackup`**, ne `state.profile.lastBackup`.

**Testování importu:** soubor se do `#importFile` dostane přes `new DataTransfer()` → `dt.items.add(new File([json],'z.json',{type:'application/json'}))` → `inp.files = dt.files` → `inp.dispatchEvent(new Event('change',{bubbles:true}))`. Sdílecí cesta iOS se testuje podvržením `navigator.canShare = () => true` a `navigator.share = async d => { got = d }`. Klasické stažení se odchytí obalením `window.Blob`.

**Testování AI funkcí bez sítě:** `aiAnalyzeFood` je klasická `function` deklarace, takže se dá přepsat přes `window.aiAnalyzeFood = async () => ({…})`. **Pro asistenta se osvědčilo něco lepšího: `page.route('**/*', …)` funguje i na `file://` stránce** — dá se tak odchytit každé volání na `workers.dev` / `api.anthropic.com`, přečíst hlavičky, změřit velikost těla a podstrčit odpověď. Jednodušší varianta (v37): přepsat `window.fetch` uvnitř `page.evaluate` a odpovídat podle počítadla requestů. **Pozor:** `pol`, `polSel`, `polTot` i `paintPol` jsou **lokální uvnitř obsluhy `#aiGo`** — testovat přes reálné UI (`test_v28_live.mjs`). `ensure()` v shimu je privátní (IIFE) — sentinel se v testu vynutí zavoláním `save()`.

**Klikací náhled pro Karla — od 31. 7. 2026 jednoduše:** `python3 -m http.server 8000` a poslat Karlovi `localhost:8000/progres.html`, případně IP Macu pro iPhone na stejné wifi. Prostředí je plnohodnotné (localStorage i IndexedDB fungují), takže odpadá `make_preview36.py` s paměťovou náhradou localStorage i podvrhováním `window.fetch`.
📜 *Historicky:* náhled se posílal jako soubor do konverzace, proto se stavěl přes `build.py --no-shim --out /tmp/preview_base.html` (aby se `APP_TOKEN` nedostal ven) a proháněl kontrolou na `['workers.dev','APP_TOKEN','x-app-token','__proxy__','sk-ant']`. ⚠️ Kdyby se ta kontrola někdy dělala znovu: **`sk-ant` v seznamu hlásí planý poplach** — je to jen `placeholder="sk-ant-…"` u `#apiKeyInput`.

## Aktuální verze: v45 (2. 8. 2026), SW cache `progres-v45.0`, commit `8e80c01`

Self-contained `index.html` (972 519 B / 966 588 znaků, MD5 `1a99a9d55bffc5d07d2ddf89bb61c85a`). Báze pro příští diff z Designu: `cd_upload45.html` (962 393 B, MD5 `240276aed8e8be7141936910eb243fac`, bez shimu).

### ⚡ v45 — profil rozdělen mezi Tělo a Jídlo, váha sjednocena s vážením
**Druhý krok reorganizace nastavení (plán v sekci v44).** Karta „Cíl kalorií & profil" v Nastavení mísila tři různé věci; rozdělena podle toho, kde uživatel výsledek sleduje.

**Nové rozložení:**
- **Tělo → „Můj profil"** (`#card-myprofile` / `renderMyProfile()`): pohlaví, věk, výška, cílová váha.
- **Jídlo → „Cíl a výpočet"** (`#card-goal` / `renderGoalCard()`): aktivita, cíl, výsledek BMR/TDEE/cíl, přepínače „připočítat kroky/tréninky", tlačítko Uložit. Přepočítává se **živě** při změně aktivity i cíle, ještě před uložením.
- **Trénink → „Nastavení"**: přibyl **denní cíl kroků** (`#pstepgoalInput`) ke čtyřem polím z v44.
- **Nastavení (gear)**: zbylo vzhled, **oslovení** (`#unameInput` / `#saveUname`), AI klíč, archiv, záloha, soukromí.

🔴 **Opravena chyba: `profile.weight` se z ručního vážení NEAKTUALIZOVAL.** Vážení přes `#weighSave` zapsalo jen do `state.body`, ale `stepKcal()`, `workoutKcal()` i BMR četly `profile.weight` — **výdej z kroků a tréninků tedy jel na váze zadané kdysi v profilu**. Asistent to dělal správně (`zapis_telo`), ruční cesta ne.
- 🆕 **`curWeight()`** — jediná pravda o aktuální váze: poslední vážení z `state.body`, fallback `profile.weight` (kvůli starým zálohám). Používají ji `stepKcal`, `workoutKcal` i výpočet cíle.
- `#weighSave` navíc drží `profile.weight` v souladu, aby fallback nelhal. Pole „Váha" z profilu **zmizelo z UI**, v datovém modelu zůstalo.
- 🆕 **`calcFrom(profil)`** nahradilo `calc()` — je to čistá funkce nad profilem. Muselo to tak být: údaje o těle sedí na Těle, aktivita a cíl na Jídle, takže z jednoho formuláře už je přečíst nejde. `#formCalc` a `calc()` zanikly.

**Drobnosti nalezené při kontrole (díky Karlovi):**
- ⚠️ Text v Mém profilu tvrdil, že se váha bere „z posledního vážení **nahoře**" — jenže karta se otevírá přes zkratku a `section.soloon` skryje vše ostatní, takže nad ní nic není. Přepsáno; přibylo tlačítko **⚖️ Zvážit se** (`#myProfWeigh` → `fabGo` na `#weighHero`).
- ⚠️ **Sekce se jmenuje „Jídlo", ne „Kalorie"** (id je `tab-kalorie`, ale `page-title` říká Jídlo). Odkazy v textech opraveny na **čtyřech** místech — dvě byly nové z v45, **dvě tam byly dávno**: prázdný graf kalorií a prázdný graf kroků. Ten druhý navíc posílal do špatné sekce úplně (kroky jsou v **Tréninku**).
- ⚠️ **Dlaždice Rychlých akcí se na Těle dotýkaly karet nad sebou.** `renderBodyOverview` vykresloval `.ring-cards` s **inline** `style=margin-bottom:0`, který přebíjel CSS pravidlo 14 px. Trénink i Jídlo mají nad hubem `.wk-hero` s toutéž hodnotou — po odstranění inline stylu mají všechny tři sekce 14 px (změřeno testem).

🆕 **`test_v45.mjs`** (41 kontrol): rozmístění karet a zkratek, BMR na kontrolním příkladu (80 kg / 180 cm / 30 let / muž = 1780), **že se výdej z kroků po změně váhy skutečně změní** (jádro opravy), živý přepočet cíle, přepínače, oslovení včetně propsání do promptu asistenta, a tři bloky proti opakování nalezených drobností: že karta netvrdí „nahoře", **že žádný text neodkazuje na neexistující název sekce** (porovnává se s `page-title`) a **že mezera nad Rychlými akcemi je ve všech třech sekcích stejná**.
- ⚠️ Regrese odhalila skutečnou závadu: `test_v39` (oslovení) sahal na zaniklý `#formCalc` — aktualizován na `#unameInput` / `#saveUname`.
- Regrese 11 sad **2× po sobě, 0 chyb**.

### ⚡ v44 — AI plán pryč, jeho nastavení zůstalo v Tréninku
**Várka mimo Design, zadání od Karla: „Vymazat ten AI plán, když už je tam asistent."** První krok větší reorganizace nastavení (plán níž).

**Odstraněno** (~100 řádků, build −6,6 kB): `renderPlanBox`, `planContent`, `salvagePlan`, `generatePlan`, `renderPlan`, `savePlanDay`, karta `#card-aiplan` s `#planBox`, zkratka v rozcestníku Tréninku a pole **`profile.plan`** z `DEFAULT`.

**Zachováno a přestěhováno** — v kartě AI plánu byla **jediná obrazovka**, kde šlo nastavit `daysPerWeek`, `experience`, `equipment` a `exPerDay`. Asistent na ně vidí přes `dotaz_data` s `co:profil`, takže smazat je spolu s kartou by mu vzalo podklad pro plán — přesně to, čím se AI plán nahrazuje.
- 🆕 **`#card-trsettings` + `renderTrainSettings()`** — nová karta „Nastavení tréninku" v sekci Trénink, zkratka `data-card=#card-trsettings` s ikonou posuvníků. Ukládá `#trSetSave`, hodnoty se ořezávají do rozsahu (dny 1–7, cviků 3–7).

**Shim** — `renderPlanBox` byla jeho kotva. Odstraněna z obou seznamů v `proxy_shim.js` (obalování i startovní překreslení). ⚠️ Shim by to ustál i bez zásahu (`if (typeof o !== "function") return` a `try/catch` u startu), ale zůstal by mrtvý kód.
- ⚠️ **`renderApiKeyBox` shim NEobaluje přes `ensure()`** jako ostatní — má vlastní logiku s `_rKey`, `own()`, `heading()` a `routeNote()`. Kontrola typu „obsahuje ensure" na něm hlásí falešný poplach (narazilo se na to při psaní `test_v44`).
- 🔤 Na **třech místech** se slibovalo „pro foto jídla a **plán tréninku**" (hint u AI klíče, text pod ním, dvakrát `heading()` v shimu). Po odstranění plánu by to lhalo — všude přepsáno na asistenta.

🆕 **`test_v44.mjs`** (29 kontrol): že po plánu nezůstala stopa (funkce, DOM, zkratka, `profile.plan`, mrtvý kód ve zdroji), že nová karta funguje a ukládá, **že shim dál drží** (sentinel `__proxy__`, obalení `renderChat`/`renderAiBox`/`renderApiKeyBox`) a **že asistent na ta čtyři pole dál vidí** přes `dotaz_data` — a že mu přitom nevydá `apiKey` (bezpečnostní oprava z v34). Regrese 10 sad **2× po sobě, 0 chyb**.
- ⚠️ `test_deleni.mjs` měl `function renderPlanBox` v seznamu kotev shimu — vyřazeno, funkce zanikla.

### 🗺️ Plán reorganizace nastavení (schválil Karel 2. 8. 2026)
Karta „Cíl kalorií & profil" v Nastavení mísí tři různé věci. Rozdělení podle toho, kde uživatel výsledek sleduje:

| Kam | Co |
|---|---|
| **Tělo** → „Můj profil" | pohlaví, věk, výška, cílová váha |
| **Trénink** → „Nastavení" ✅ hotovo ve v44 | tréninků týdně, zkušenost, vybavení, cviků na trénink (+ cíl kroků ve v45) |
| **Kalorie** → „Cíl a výpočet" | aktivita, cíl (cut/maintain/bulk), výsledek BMR/TDEE, přičítat kroky, přičítat tréninky |
| **Nastavení** (gear) | vzhled, AI klíč, archiv, záloha, soukromí — jen technika a data |

Oslovení (`profile.name`) zůstává v Nastavení — je to nastavení chování asistenta, ne údaj o těle.

🔴 **Nález k vyřešení ve v45: `profile.weight` se z vážení na Těle NEAKTUALIZUJE.** Ruční vážení přes `#weighSave` zapíše jen do `state.body`; `profile.weight` zůstane ten z profilu. Přitom z něj počítá `stepKcal()`, `workoutKcal()` i BMR — **výdej z kroků a tréninků tedy jede na zastaralé váze**. Asistent to dělá správně (`zapis_telo` profil aktualizuje), ruční cesta ne. Karel schválil **sjednocení**: `profile.weight` se bude brát z posledního vážení a jako pole z profilu zmizí.

### ⚡ v43 — rozložení maker po dnech, swipe mezi nimi
**Várka mimo Design (skill `uprava-progres`), zadání od Karla: „Ve statistikách mít koláč makra i z předchozích dnů (alespoň týden) a swipem moci koláče porovnávat (okna vedle sebe)."**

Karta „Rozložení maker" (`#card-macrosplit`) měla **jeden** koláč za celé období. Jednotlivé dny se porovnat nedaly.

- 🆕 **Swipe pás `#macroSwipe` + tečky `#macroDots`** místo canvasu `#chMacroSplit` (ten je **odstraněn**) a bloku `#macroSplitInfo`. Recykluje se hotová mechanika z Přehledu — CSS `.swipe` / `.swipe-card` / `.swipe-dots` a varianta `.card .swipe-card` už v appce byly.
- **Karta 0 = průměr za celé období** (původní chování, výchozí pohled), pak **posledních `MACRO_DNU` = 7 dnů s daty** od nejnovějšího. Popisky „Dnes" / „Včera" / datum, v nadpisu vždy kcal daného dne — bez toho by se dny porovnávaly jen poměrem, ne objemem.
- 🆕 **`macroPieData(p,s,f)`** — pomocník, který z gramů udělá kalorie (B a S ×4, T ×9) a procenta.
- Grafy `chMacroPie0..N`, legenda jen u první karty (u malých koláčů by zabrala půl plochy).
- ⚠️ **Podíl se počítá z KALORIÍ, ne z gramů.** Při 100 g od každého makra to není 33/33/33, ale **24/24/53 %**. `test_v43.mjs` to hlídá konkrétními čísly a navíc explicitní podmínkou, že třetiny vyjít nesmí.
- ⚠️ **Staré instance se ruší při každém překreslení** (`charts` klíče s prefixem `chMacroPie`). Bez toho by se s každou změnou období hromadily. Test kreslí 3× po sobě a počítá, kolik jich visí.
- ⚠️ **Průměr se počítá ze VŠECH dnů období, ne jen ze sedmi zobrazených** — jinak by při delším období tiše lhal. Hlídá blok F.
- 🆕 **`test_v43.mjs`** (21 kontrol) + guard, který na staré verzi čitelně nahlásí, že swipe pás neexistuje, místo pádu. **Ověřeno negativně proti nasazené v42.** Regrese 9 sad **2× po sobě, 0 chyb**.

### ⚡ v42 — zpětná úprava historie u jídla i tréninku
**Várka mimo Design (skill `uprava-progres`), zadání od Karla: „Upravovat zpětně makro z historie, stejně tak i u tréninků možnost upravovat historii."**

Do v41 šel v UI změnit jen **název** (přejmenování). Kalorie, makra, datum ani série u tréninku se z historie opravit nedaly — jedině přes asistenta (`uprav_jidlo` / `uprav_trenink`), tedy oklikou přes AI za peníze.

**Jídlo — modál `#foodEditBg`:**
- 🆕 **`openFoodEdit(id)` / `closeFoodEdit()` / `foodEditSave()`** + pomocník `feNum()`. Pole `#feName`, `#feKcal`, `#feP`, `#feC`, `#feF`, `#feDate`.
- Tlačítko v historii se změnilo z `data-renh` (přejmenovat) na **`data-edith`** (upravit) — přejmenování je teď podmnožina editace, tlačítek nepřibylo.
- Prázdné pole u makra uloží `null`, ne nulu. Změna data přesune záznam pod jiný den.

**Trénink — formulář jako editor (varianta B, Karel vybral):**
- 🆕 **`editSessId`** + **`startSessEdit(id)`**, **`endSessEdit()`**, **`sessEditBar(on,popis)`**. Tlačítko `data-edit-sess` s novou ikonou **`ICON_SLID`** (posuvníky).
- Klik načte trénink do `#card-newworkout` **se vším** — cviky, série, váhy — a nahoře naskočí pruh `#sessEditBar` („Upravuješ trénink…") s tlačítkem Zrušit. Text `#sessSubmit` se změní na „✎ Uložit změny".
- 🆕 **`fillSessionForm(sess, keepMeta)`** — druhý parametr. Bez něj (předloha) se nastaví dnešní datum a prázdná poznámka jako dosud; s ním se **zachová původní datum a poznámka**, jinak by úprava záznam přesunula na dnešek.
- Uložení **přepíše** záznam na místě (`state.sessions[i] = rec`) a **zachová `title`**, aby se neztratilo přejmenování. Ošetřeny obě větve submitu — Fitko i Kardio.
- ⚠️ **Rekordy se nerozbijí** — `bestBefore(name, id)` vynechává z porovnání záznam se stejným `id`, a úprava `id` zachovává. Kdyby se při úpravě generovalo nové `id`, trénink by se porovnával sám se sebou.
- `⧉` („Použít jako předlohu") a tužka (přejmenovat) u tréninku zůstávají beze změny.

**Ověření:**
- 🆕 **`test_v42.mjs`** (38 kontrol). Blok 0 hlásí čitelně, když editační API vůbec chybí, místo pádu. Těžiště je na **duplikátech**: počet záznamů u jídla i tréninku, zachování `title`, že nevyplněné pole nepřepíše původní hodnotu, a že předloha dál vytváří **nový** záznam s dnešním datem. **Ověřeno negativně proti nasazené verzi — padá na tom, že API neexistuje.**
- Regrese 8 sad **2× po sobě, 0 chyb**. Tělo API requestu beze změny.

### 🖼️ v42 — vrácen klikací náhled do konverzace (`tools/make_preview.py`)
**Karel se ptal, proč musí náhled spouštět přes lokální server, když dřív chodil rovnou do Coworku.** Měl pravdu — postup ve skillu popisoval jen `python3 -m http.server`, což je cesta pro Claude Code. V Coworku jde hotové HTML poslat přes `SendUserFile` a proklikat vedle chatu.

`tools/make_preview.py` to dělá na jeden příkaz:
1. `build.py --no-shim` → APP_TOKEN se do souboru vůbec nedostane
2. vypne registraci service workeru
3. naseeduje ukázková data (6 jídel, 3 tréninky) a otevře Kalorie
4. prožene výsledek kontrolou na 5 vzorů tajemství — při nálezu soubor **smaže** a skončí chybou

⚠️ **localStorage se řešit nemusí** — appka má vlastní fallback (`lsGet`/`lsSet` + `memStore`), takže v prostředí bez úložiště běží z paměti. Starý `make_preview36.py` kvůli tomu vkládal paměťovou náhradu; dnes je to zbytečné.

### ⚡ v41 — mikrofon už nezamrzne appku
**Várka mimo Design (skill `uprava-progres`), hlášení od Karla: „když u asistenta v chatu kliknu na mikrofon, tak se aplikace zasekne" — a upřesněno: mikrofon svítí, appka nereaguje, musí se úplně zavřít a otevřít.**

**Příčina, dvě vrstvy:**
1. Na iOS `webkitSpeechRecognition` v `window` **existuje**, takže podmínka `if(mic&&SR)` prošla a tlačítko se zobrazilo — ale samotné rozpoznávání umí zatuhnout celý WebView.
2. **`r.start()` selhává ASYNCHRONNĚ**, takže `try/catch` kolem něj nic nezachytí. Když `onerror` nedorazil, stav „nahrávám" neměl kdo uklidit — žádný timeout tam nebyl.

**Oprava:**
- 🆕 **`window.micBlocked`** — detekce iOS (`/iP(hone|od|ad)/` v UA, plus iPadOS maskovaný jako `MacIntel` s `maxTouchPoints > 1`). Na iOS se tlačítko **nezobrazí vůbec**; podmínka je `if(mic&&SR&&!IOS)`. Funkce se neztrácí — diktovat jde ze systémové klávesnice (mikrofon vedle mezerníku), a to spolehlivě.
- 🆕 **8s pojistka na ostatních platformách** (`micT` + `micOff()`, exportované jako `window.micOff`): po `start()` se nastaví timeout, který `onstart` zruší. Když `onstart` nedorazí, timeout zavolá `abort()`, uklidí třídu `rec` a řekne „Diktování se nespustilo". Tlačítko tedy nemůže svítit donekonečna.
- `onend` i `onerror` teď vedou přes společné `micOff()`, takže se timeout uklidí vždy.
- 🆕 **`test_v41.mjs`** (13 kontrol, psaný poruchou napřed): A iPhone UA → tlačítko `display:none` a `micBlocked===true`; B desktop UA → `display:grid` (funkce se nesmí ztratit); C bez `onstart` je po 8 s stav uklizený a `abort()` zavolaný; D s `onstart` pojistka běžící diktování **NEsmí** utnout; E druhý klik zastaví a uklidí. **Ověřeno negativně: proti nasazené v40 sada padá 6× přesně na Karlově symptomu.**
- Regrese 7 sad **2× po sobě, 0 chyb**. Tělo API requestu beze změny → skutečné volání netřeba. MD5 v40 byla `bae408e7c30ea33df57b68adcee33755`.
- ⚠️ **Playwright na Macu ≠ prostředí mostu.** Testy se z Coworku v cloudu nedají spustit přes `device_bash` — Linux VM mostu nemá stažený prohlížeč a nemá síť, aby si ho stáhla. Řešení: nastage soubory do cloudového kontejneru a spustit tam (má vlastní Chromium). Pozor, verze buildu Chromia se nemusí shodovat s tím, co čeká nainstalovaný Playwright.

### 🔧 1. 8. 2026 — dokumentace teče zdola nahoru (appka beze změny)

**Změna zapojení, ne appky.** `index.html`, `sw.js` ani ikony se nezměnily —
diff obsahuje jen `claude/*.md` a prázdný `.nojekyll`.

Cíl byl propojit projekt na claude.ai s pracovní složkou, aby se soubory
nemusely posílat ručně. Rozpadlo se to na dvě různé věci:

- **Cowork sessions** — vyřešeno připojením složky přes „Add folder" v desktopové
  appce. Session pak čte `progres.src.html`, `CLAUDE.md`, testy i git historii
  přímo z disku. Nic se nenahrává.
- **Chat konverzace v projektu** — potřebují sync z GitHubu, a ten **na privátní
  `progres-toolchain` nefunguje** (viz „Co vidí projekt na claude.ai" výš).
  Řešení: dokumentace se zrcadlí do veřejného `progres` do složky `claude/`.

Co se u toho našlo a spravilo:
- 🔐 **Obě kopie `proxy-ai-sdilena.md` v projektu měly APP_TOKEN natvrdo**
  v tabulce proměnných — hodnotu z v29, tou dobou už dvakrát rotovanou
  (ověřeno proti `token.txt`, takže neplatnou). Ve verzi v toolchainu bylo
  opraveno dřív, do projektu se oprava nikdy nepropsala. Kopie smazány,
  zbylá přepsána čistou verzí.
- 📄 **V projektu ležely dvě verze stavového dokumentu** — v kořeni v40, ve
  `claude/` v38. Ta stará pořád popisovala deploy přes Claude in Chrome,
  efemérní kontejner a povinnou obnovu báze. Konverzace z ní mohla číst návody
  neplatné od 31. 7. Smazána.
- 🧹 Opraveno v tomto dokumentu: cesta ke složce (je
  `~/Desktop/Claude Code/Progres App/`, ne „Claude Code - Progress App"),
  knihovny buildu jsou ve `vendor/`, ne v `node_modules/`.
- ➕ `repo/.nojekyll` — vypíná Jekyll, aby přidané `.md` nemohly shodit build
  Pages.

### ⚡ v40 — klávesnicová vrstva v38 přepsaná podle skutečného chování iOS
**Várka mimo Design, tři iterace podle Karlova testování na iPhonu přes lokální server (`http://IP-Macu:8000/progres.html`) — nasazovalo se až schválené chování.** Symptom z v39: při psaní kg/opakování v Novém tréninku obsah poskočil a pole zmizelo; po první opravě zůstal poskok u pole těsně nad klávesnicí; po druhé se ukázalo, že reveal přebíjí i ruční scroll uživatele.

**Nová pravidla `reveal()` (nahrazují 38% linku z v38):**
- zasáhne JEN když pole není celé vidět (rezerva 12 px) — iOS si pole typicky pokládá pár px nad klávesnici a jakákoli „pohodlná rezerva" u spodní hrany znamená rvačku dvou scrollů
- scrolluje MINIMÁLNĚ (pole k nejbližší hraně), žádné tahání na 38 %
- je „ozbrojený" jen ~1,2 s po fokusu (kolem otevírání klávesnice); `vv.resize` mimo to okno reveal nespouští — v Safari chodí resize i při ručním scrollu (sbalení lišty) a stahoval uživatele zpátky k poli
- `touchmove`/`wheel` reveal odzbrojí úplně — řízení přebírá uživatel až do dalšího fokusu

`--kb` vrstva (chat lišta, modály) beze změny. Tělo API requestů beze změny (skutečné volání netřeba). MD5 v39 byla `2cfe2852…`.

- 🆕 **`test_v40.mjs`** (7 kontrol, každá iterace psaná poruchou napřed: A viditelné pole 287 px → 0; A2 pole nad klávesnicí 461 px → 0; D stažení po ručním scrollu 642 px → 0; B/C/E hlídají, že skrytá pole se pořád odkrývají).
- ⚠️ **Známé slabé místo (nehlášeno, neopravováno):** `#pickerSearch` sedí ve fixed `.modal-bg` — když `.modal` není přetečený, `scrollable()` vrátí null a reveal by scrolloval stránku ZA overlayem. Kdyby někdo hlásil poskoky při hledání cviku, začít tady.

### ⚡ v39 — oslovení asistenta z profilu (konec natvrdo „Karel") + druhá rotace APP_TOKEN
**Várka mimo Design (skill `uprava-progres`), zadání od Karla: „AI v aplikaci i bratra oslovuje Karle, mohl bys to opravit?"** Položka z roadmapy (personalizace z hloubkové kontroly v37).

- 🆕 **`profile.name`** (default `""`) — **oslovení, které uživatel píše sám v 5. pádě** (appka česky neskloňuje, žádný skloňovač). Pole „Oslovení v chatu (nepovinné)" na začátku karty Cíl kalorií & profil, input `name="uname"` (⚠️ ne `name="name"` — koliduje s `form.name`), ukládá ho „Uložit cíl" (nápověda v placeholderu na to upozorňuje). Jde do zálohy jako běžný údaj profilu.
- **Systémový prompt**: místo „Uživatel je Karel." podmíněné „Uživatel si přeje oslovení „X" — oslovuj ho tak." (bez vyplnění nic). **Uvítací bublina**: „Ahoj X." / neutrální „Ahoj." — oslovení jde přes `esc()` (XSS).
- **Staré zálohy bez pole**: `mergeState` dá `""` (DEFAULT + Object.assign), v promptu nikdy „undefined" — hlídá blok D testu.
- 🆕 **`test_v39.mjs`** (16 kontrol) + na Macu obnovená pojistka: runner `tools/run_tests.mjs` (obě konvence hlášení, nenaparsovaný výstup = chyba, maskování tokenu) a sady `test_v38` (temperature), `test_deleni` (proxy/kotvy), `test_backup31` (klíč+data v záloze), `test_regress29` (22 kontrol následků). Vše psáno „poruchou napřed".
- 🔐 **Druhá rotace APP_TOKEN (němá)** — token z v38.1 unikl do konverzace výstupem `check_cut.py` (před doplněním maskování); nový vygenerován bez vypsání kamkoli, žije jen v `token.txt` a Cloudflare Secrets. Starý token vrací 401 (ověřeno).
- **Živě ověřeno** (změna obsahu `system` = povinné skutečné volání): `asstApi` → `end_turn`, model `claude-sonnet-4-5-20250929`, text „OK", 7 161 vstupních / 4 výstupní tokeny, 2,6 s. SW `progres-v39.0`, MD5 živého HTML = build (`2cfe2852…`).
- 📣 **Uživatelé**: po aktualizaci si každý vyplní oslovení v Nastavení (Karel „Karle", bratr/kolega svoje). Do té doby asistent zdraví neutrálně.

### ⚡ v38.1 (31. 7. 2026), SW cache `progres-v38.1`, commit `abc2110`

Self-contained `index.html` (965 769 B / 959 809 znaků, MD5 `1aebe5df7ebc6f057e3e7cb1039c0ccd`: inline Chart.js + html5-qrcode/ZXing), česky, mobile-first, PWA, offline. Návrh z Claude Design (2a tmavý + 2b světlý), gear vpravo nahoře = Nastavení, Asistent je pátá záložka v dolní navigaci.

### ⚡ v38.1 — rotace APP_TOKEN (žádná změna funkcí)
**Jediná změna proti v38 je hodnota tokenu v shimu** — kód appky, shim i knihovny jsou bajtově totožné, velikost souboru se nemění (starý i nový token mají 44 znaků). Diff v38→v38.1 = 1 řádek v `index.html` (`var TOKEN`), 1 řádek v `sw.js` (cache `progres-v38.1`). MD5 v38 byla `77ef640690c04ba136e452306987e883`.

Důvod: starý token žil od v29 v Claude kontejnerech, konverzacích a docs; nový vznikl `openssl rand -hex 20` na Karlově Macu a existuje jen v `token.txt` (mimo git, práva 600) a v Cloudflare Secrets. Pořadí nasazení: deploy → ověření živě (MD5 živého HTML == nový build) → přepnutí secretu v Cloudflare (Karel ručně) → skutečné API volání. Výpadek AI jen mezi ověřením a přepnutím secretu.

- Živě ověřeno: `index.html` MD5 `1aebe5df…` (bajt na bajt), starý token v HTML 0 výskytů, SW `progres-v38.1` na třetí poll (~45 s po pushi).
- **Skutečné API volání po rotaci: OK.** `asstApi` s jedinou zprávou vrátila `end_turn`, model `claude-sonnet-4-5-20250929`, text „OK", 7 168 vstupních / 4 výstupní tokeny, 1,6 s. `/health` s novým tokenem 200 (vše nastaveno), se starým i bez tokenu 401. Pozor při příští rotaci: (1) po uložení secretu chvíli trvá propagace (~10 s — první test hned po Save vrátil 401 a vypadal jako neúspěch), (2) schránka není spolehlivý přenos hodnoty — cokoli mezitím zkopíruješ ji přepíše; při prvním pokusu se do secretu vložil text rotačního příkazu místo tokenu a obě strany pak měly 401.
- Uživatelům (bratr, kolega) je potřeba říct, ať appku úplně zavřou a otevřou.

### 🔧 31. 7. 2026 — vývoj přesunut do Claude Code na Macu (appka beze změny)
**Změna pracovního postupu, ne appky.** Nasazený build zůstává v38, MD5 `77ef640690c04ba136e452306987e883`. Jediný commit toho dne je `b8ec529` — přidaný popisný řádek do `README.md`, ověřený zkušební push. Diff `ed94775..b8ec529` obsahuje **jediný soubor, `README.md`**; `index.html`, `sw.js`, ikony ani manifest se nezměnily.

Co se ověřilo:
- **Čerstvý klon** souhlasí s dokumentem: commit `ed94775`, `progres-v38.0`, MD5 `77ef6406…`, 965 769 B.
- **Toolchain zrekonstruován a `cmp` prošel napoprvé.** Ruční složení bez `build.py` (`tools/check_cut.py`) dalo bajtově identický soubor, teprve pak vznikl `build.py`. Build je deterministický (dva běhy = stejné MD5).
- **`cd_upload38.html`** uložen jako báze pro příští diff z Designu: 955 587 B, MD5 `0c89cd6c7f6c955fbb919a3ad4bfae9d`. Rozdíl proti plnému buildu 10 182 B = shim 10 162 + obal `<script>\n` … `\n</script>\n`.
- **SSH autentizace na GitHub** funguje, remote přepnut z HTTPS na SSH, zkušební push prošel. Git identita nastavená **jen lokálně** pro tenhle klon (`G4M3CH4N63R`), systémový git netknutý.

Co z toho plyne pro příště: odpadá obnova báze, odpadá deploy přes GitHub UI a s ním celá sada potíží s klikáním, přibyl lokální náhled včetně testování z iPhonu na wifi, a `repo/` přestalo být read-only reference — je to pracovní klon i deploy cíl. **Zbývá:** přenést testy a zálohovat toolchain (viz Roadmapa).

### ⚡ v38 — hotfix `temperature` (API 400) + iOS klávesnice nezakrývá editované pole
**Hotfix várka mimo Design, ~hodinu po v37.** Karel poslal screenshot: každá zpráva asistentovi padala na **„API chyba 400: `temperature` is deprecated for this model"** — parametr `temperature:0.4` přidaný ve v37 aktuální modely odmítají. Mock testy to zachytit nemohly (podvržený fetch serverovou validaci nevidí). Druhá věc z téže zprávy: **kdekoli v appce se po ťuknutí do textového pole obrazovka odscrollovala tak, že pole zmizelo** (za klávesnicí / mimo obraz).

Nasazený build MD5 `77ef640690c04ba136e452306987e883` (v37 byl `0d13f20d36d5d2408cc11f7196956a41`). `node --check` OK, plná regrese **26 běžících sad 2× po sobě, 0 chyb**, nový `test_v38.mjs` (7 bloků). **Živě ověřeno skutečným API voláním:** `asstApi` s jedinou zprávou vrátila `end_turn`, model `claude-sonnet-4-5-20250929`, text „OK", 7 170 vstupních / 4 výstupní tokeny — 400 je pryč.

- 🔴 **`temperature` ODSTRANĚNO z `asstApi` a NESMÍ SE VRACET.** Komentář přímo v kódu + `test_v37.mjs` blok J a `test_v38.mjs` bloky A/B hlídají vzor `temperature\s*:` v `asstApi` i `aiCall`. Přesnost počtů drží aparát z v37 (hotové součty), ne sampling.
- 🆕 **Klávesnicová vrstva (IIFE za Chart.defaults, PŘED `initSeg()` — schválně MIMO blok `if("serviceWorker" in navigator…)`, aby běžela i na `file://` v testech).** Obsah:
  - `reveal()` (exportované jako **`window.kbReveal`** pro testy): pokud má fokus INPUT/TEXTAREA/SELECT, spočítá cíl = horních ~38 % **visual viewportu** (`vv.offsetTop + min(vv.height*0.38, max(90, vv.height−120))`) a přiscrolluje: najde nejbližšího scrollovatelného předka (modály mají `overflow:auto`) a posune jeho `scrollTop`, jinak `window.scrollBy`. Tolerance ±28 px, aby to necukalo. **Pole v `.chat-bar` se přeskakuje** — lišta je sticky a zvedá ji `--kb`.
  - Spouštění: `focusin` → `plan(380)` (čeká na animaci klávesnice), `visualViewport.resize` → `kbVar()` + `plan(120)`, `visualViewport.scroll` → jen `kbVar()` (žádný reveal — nesmí se prát s ručním scrollem uživatele).
  - `kbVar()`: `--kb = max(0, innerHeight − vv.height − vv.offsetTop)`, práh 60 px (šum vs. klávesnice), zaokrouhleno.
  - CSS: `.chat-bar{bottom:calc(8px + var(--safe-bottom) + var(--kb,0px))}` a `.modal{padding-bottom:calc(20px + var(--safe-bottom) + var(--kb,0px))}` — lišta chatu i spodek modálu se zvednou nad klávesnici. Na desktopu se `vv.height` nemění → `--kb` zůstane 0 → nulový dopad.
  - ⚠️ **Netestovatelné z cloudu:** skutečné chování iOS klávesnice se ověří jen na Karlově iPhonu. Kdyby hlásil, že se pole pořád schovává, chtít vědět KDE (sekce/karta) — pravděpodobná příčina by byla další scrollovatelný kontejner, který `scrollable()` nenajde.
- 📌 Viewport meta už `maximum-scale=1` měla (auto-zoom na fokusu nebyl příčina) — příčinou bylo, že appka nechávala scroll po otevření klávesnice čistě na iOS.

### ⚡ v37 — asistent už nepočítá z hlavy: hotové součty, oprava jídla, retry, pause_turn
**Druhá várka mimo Claude Design, celá ve zdroji appky (kromě dat — žádná změna UI).** Podnět: Karel poslal screenshot, kde asistent **podruhé** špatně sečetl den — po zapsání tří jídel ohlásil 3597 kcal / 201 g bílkovin místo správných 2197 / 132. **Příčina (systémová):** model zapsal jídla nástroji, a pak k součtu, který si pamatoval z dřívějška konverzace, ta samá jídla přičetl ještě jednou — přestože systémový kontext už obsahoval přepočítaný správný součet. Data v appce byla vždy správně; lhal jen výpočet v hlavě modelu. Řešení: **model nesmí mít důvod cokoli sčítat.**

Nasazený build MD5 `0d13f20d36d5d2408cc11f7196956a41` (v36 byl `3c904e6950379c4b420a4ad8eb40ec47`). `node --check` OK, **plná regrese 25 běžících sad 2× po sobě, 0 chyb**, `test_v37.mjs` 11 bloků. Diff v36→v37 = 10 bloků, jen JS (žádné CSS/HTML markup změny).

**Přesnost počtů (jádro):**
- 🆕 **`asstDayTotals(d)`** — hotový denní součet: `{datum, pozn, kcal_celkem, bilkoviny_celkem, sacharidy_celkem, tuky_celkem, zapisu_za_den, cil_kcal, zbyva_kcal, cil_bilkovin, zbyva_bilkovin}` + `zapisu_bez_udaje_bilkovin` (jen když > 0 — ať model ví, že skutečné bílkoviny jsou nejspíš vyšší). Pro dnešek se cíl bere z `effTarget()` (vč. příplatků za kroky/trénink), pro jiné dny z `targetKcal`.
- 🆕 **`zapis_jidlo`, `smaz_jidlo` a `uprav_jidlo` vracejí interní marker `_den`** a smyčka v `asstSend` ho po **dokončení VŠECH nástrojů daného kola** nahradí polem **`den`** = `asstDayTotals(datum)`. Klíčové pořadí: kdyby se součet počítal hned v nástroji, tři zápisy v jednom kole by daly tři různé mezisoučty a model by zase dopočítával. Takhle dostane třikrát totéž konečné číslo. (`_den` se maže, ven nikdy nejde — hlídá test.)
- 🆕 **`dotaz_data` má dvě nové volby:** `co:'dnes'` (hotový součet dne + položky; jiný den přes `od`) a `co:'souhrn'` (od/do, výchozí 30 dní: kcal a bílkoviny celkem i průměr na den, počet tréninků, objem celkem i na trénink, kroky celkem i průměr). Průměry za týden/měsíc už taky nepočítá model.
- 🆕 **Systémový prompt:** odstavec „POČÍTÁNÍ — pravidlo, které nesmíš porušit": NIKDY nesčítat z hlavy, nikdy nepřičítat právě zapsané jídlo k dřívějšímu číslu, hlásit výhradně čísla z pole `den`; vlastní čísla z dřívějších zpráv jsou **ZASTARALÁ**; když uživatel číslo **zpochybní**, nehádat se a zavolat `dotaz_data`. Řádek „Dnes zatím sněděno" v kontextu nově výslovně říká, že je to HOTOVÝ součet přepočítaný právě teď (vč. zápisů z této konverzace), a přibylo „do cíle bílkovin zbývá X g".

**Užitečnost a odolnost (hloubková kontrola na Karlovu žádost):**
- 🆕 **Nástroj `uprav_jidlo`** — 13. vlastní nástroj (`ASST_TOOLS` = 13 + web_search). Najde záznam podle `id` nebo `nazev`(+`datum`), přepíše jen vyplněná pole (`novy_nazev`, `kcal`, `bilkoviny`, `sacharidy`, `tuky`, `nove_datum`), vrátí `_den`. Bublina „✏️ Opraveno: …". Prompt: opravu zapsaného jídla dělat tímhle nástrojem, ne druhým zápisem.
- ~~`temperature: 0.4`~~ → 🔴 **VRÁCENO ve v38** — aktuální modely parametr odmítají (API 400 „deprecated for this model"). Do těla asistentových requestů se `temperature` NIKDY nevrací.
- 🆕 **Retry:** `asstApi` při síťové chybě, 429, 500, 502, 503 a 529 **jednou tiše zopakuje** požadavek (pauza 1,8 s). Když selže i podruhé, hlásí česky: 429 „Moc dotazů za sebou…", 503/529 „AI služba má zrovna nával…". Retry běží uvnitř originální funkce, takže shim (který na dobu volání podvrhuje `window.fetch` směrem na Worker) pokryje i opakované pokusy. **400 se NEopakuje** — správně, chyba požadavku není přechodná (ověřeno živě na temperature průšvihu).
- 🆕 **`pause_turn` fix:** `web_search` může vrátit `stop_reason:"pause_turn"` — smyčka dřív skončila a odpověď zůstala useknutá. Teď `if(stop_reason==="pause_turn" && !uses.length) continue` (assistant bloky se pošlou zpět a API pokračuje). Strop 8 kol platí dál.
- 🆕 **Prompt navíc:** při zmínce bolesti/zranění nedoporučovat trénink přes bolest (šetrná varianta nebo pauza).

**Co se kontrolovalo a záměrně NEměnilo:** datumy jedou z lokálního času (`todayISO` — zápis po půlnoci padne do správného dne ✓); klíč se do kontextu neposílá (výjimka v `dotaz_data profil` z v34 drží ✓); rozsahy kontextu (tréninky 120 d / součty 90 d / položky 21 d) a `web_search max_uses:5` beze změny — Karlovo rozhodnutí o nákladech; model `claude-sonnet-4-5` beze změny (Worker má stejně whitelist a nepovolený model tiše přepíše na výchozí; živě reálně jede `claude-sonnet-4-5-20250929`).

⚠️ **Poznámky pro příští várky:** (1) Text `pozn` v `asstDayTotals` je schválně **bez diakritiky** (jde do JSON tool_result). (2) `chatMsgsFromView` posílá jen bubliny `u`/`a` — akční bubliny (`act`) model v historii nevidí, jeho znalost dat jde vždy ze systémového kontextu nebo z nástrojů. (3) Kdyby Design poslal export postavený na v36, **celý v37+v38 aparát zmizí** — vracet podle těchto sekcí.

### ⚡ v36 — historie chatu v levém panelu, archiv až po rozkliku, záloha po 5 dnech, čistší Přehled
**První várka, která NEPŘIŠLA z Claude Design** — celé zadání dal Karel přímo („Ano přejmenuj na jen Záloha… Připomenutí zálohy bych zkrátil na 5 dní… Přidal bych i historii chatu… Archiv nastav k zobrazení až po rozkliku" + později „v Přehledu smaž ty dvě okna nahoře"). Odpadá tím bajtová kontrola proti exportu, takže jistotu nahradil **klikací náhled ke schválení** a **dvakrát spuštěná plná regrese: 24 sad, 0 chyb** (`test_v28_live.mjs` se záměrně přeskakuje — chodí na živý web). `node --check` OK. Nasazený build MD5 `3c904e6950379c4b420a4ad8eb40ec47`, předchozí v35 byl `ac14b37fcf3683b728eac8ee4e894531`.

**Co je nového:**
- 🆕 **Historie chatu — starší konverzace v levém panelu.** Nový model: vedle `progres_chat_v1` (jedna aktivní konverzace, kvůli zpětné kompatibilitě) přibyl **`progres_chats_v1`** = `{v:1, cur, list:[{id,title,ts,renamed,msgs[]}]}`, **max 30 vláken** (`CHAT_MAX`) po **80 zprávách** (`CHAT_MSGS`). V asistentovi je lišta `.asst-bar` s **☰ Historie** (`#asstHistBtn`), **＋ Nový** (`#asstNew`) a **Vymazat chat** (`#asstClear`). Panel je vysouvací drawer zleva (`.drawer` z-index 45, backdrop `.dw-bg` 44 — tedy **pod** modály na 50 a FAB na 55). Řádek vlákna nese název, relativní čas (`chatWhen`) a počet zpráv (`plurZprava`), plus ikonky **přejmenovat** (`ICON_PEN`) a **smazat** (`ICON_TRASH`). Název se odvozuje z první zprávy (`chatTitleFrom`), po ručním přejmenování se drží (`renamed:true`). Prázdné konverzace kromě otevřené se neuchovávají.
  - 🔴 **Chyba nalezená a opravená při testování:** `chatSave()` nastavovalo `c.ts = Date.now()` **vždy**, takže **pouhé otevření staršího vlákna mu přepsalo datum na „dnes" a vyhodilo ho na první místo** — přesně to, co mělo zadání („abych se k nim mohl vrátit") umožnit. Stejně tak přejmenování nebo smazání *jiného* vlákna posunulo čas tomu **právě otevřenému**. Řešení: **`chatSave(touch)`** — při `touch === false` se `ts` nechá být. `chatOpen`, `chatDelete`, `chatRename` a první volání v `chatNew` volají `chatSave(false)`; čas se mění jen tehdy, když do konverzace opravdu přibude zpráva. Hlídá to blok `datumPriCteni` v `test_v36.mjs`.
- 🆕 **Archiv smazaných až po rozkliku.** V Nastavení je `#card-archive` obalený do `<details id="archAll">` se `<summary id="archAllSum">`, který podle obsahu píše **„Zobrazit smazané · N"** nebo **„Archiv je prázdný"** (a v tom případě se `open` násilím drží na `false`). Archivy u jednotlivých karet z v35 zůstaly beze změny.
- 🔤 **Tlačítko `#exportBtn` se jmenuje prostě „Záloha"** (bylo „Záloha (JSON)"). Vyřešená drobnost z roadmapy — název si odporoval s `.txt` variantou na iPhonu.
- ⏱ **`BACKUP_WARN_DAYS = 5`** (bylo 14). Připomínka na Přehledu i oranžová tečka u stavu v Nastavení naskočí dřív.
- 🧹 **Přehled bez dvou kruhových dlaždic.** `<div class="ring-cards" id="progressRings">` (Váha + Konzistence) je pryč a s ním celá funkce `renderProgressRings()` a její dvě volání v `renderDashboard()` a `renderData()`. Přehled teď jde: nadpis → připomínka zálohy → swipe karty → Kalendář tréninků. **Informace se neztratila** — váha, trend i odhad k cíli jsou ve swipe kartě „Váha" a na Tělu, konzistence zůstává jako heatmapa „posledních 20 týdnů" (`#card-heat`) v Tréninku pod statistikami.
  - ⚠️ **CSS `.ring-cards` / `.ring-card` se NESMÍ smazat** — používá ho `#bodyOverview` na záložce Tělo (test `teloMaKruhy` hlídá, že tam kroužky pořád jsou). Funkce `ringSVG`, `isoAddDays` a `mondayOf` taky zůstávají, volají je jiná místa.

### ⚡ v35 — konec záložky Progres, nové grafy jídla, přejmenování, archiv u karet + karta Soukromí a data
Export z Designu (jeho `sw.js` = v47.0), diff proti v34 bez shimu = **32 hunků / 1 080 řádků patche**. **Čistý port ověřen bajtovou shodou s exportem** (`cd_upload35.html`, MD5 `80372a2a34d94809250fd540c9e2d1ab`), teprve pak přišla vlastní karta Soukromí. `node --check` OK, **24 testovacích sad bez jediné chyby**, živě ověřeno (`progres-v35.0`, karta i všechny nové funkce na první navigaci). Nasazený build: MD5 `ac14b37fcf3683b728eac8ee4e894531`.

**Co přinesl Design:**
- **Záložka „Progres" zmizela, statistiky se přestěhovaly do sekcí.** `#tab-data` je pryč, navigace je **prehled / trenink / telo / kalorie / asistent**. Statistické karty žijí ve skrytém **`#statsPool`** (17 karet) a `showStats(tab)` je do sekce vloží jako `.solocard` — trénink 6, tělo 5, kalorie 8. Při odchodu se karty vrátí do poolu (ověřeno testem, ať se neztratí).
- **`switchTab(name, mode)`** — nový druhý parametr. `switchTab('trenink','body')` vykreslí sekci rovnou se všemi kartami (bez rozcestníku), `showSolo(sec, cardSel)` zavrtá do jedné karty.
- **Asistent je plnohodnotná záložka** místo bubliny v hlavičce; `#asstBtn` je odstraněn. `openAsst()` / `closeAsst()` zůstaly.
- **Pět nových pohledů na jídlo** — rozpad maker (`chartMacroSplit` → `chMacroSplit`), makra v čase (`chartMacroTime` → `chMacroTime`), bílkoviny (`chartProtein` → `chProtein`), dny v týdnu (`chartFoodDow` → **`chDow`**), tabulka dnů (`renderFoodDaysTable` → `#foodDaysTable`) a **top jídla** (`renderTopFoods` → `#topFoodsTable`). Karty `#card-macrosplit/-macrotime/-protein/-fooddow/-fooddays/-topfoods`. Podkladová funkce **`foodDays(cut)` vrací objekt klíčovaný datem** `{kcal,p,c,f}`.
- **Přejmenování čehokoli** — vlastní dialog `openPrompt(title, label, val, cb, okText)` nad `#promptBg` / `#promptTitle` / `#promptLabel` / `#promptInput` / `#promptOk`. Háčky: `data-tplren` (šablona), `data-ren-sess` (trénink — nové pole **`s.title`**), `data-renrec` (recept), `data-renfood` (jídlo), `data-renh` (položka historie).
- **Archiv smazaných je u karet, ne v jedné hromadě** — `renderArchiveBlock(elId, kinds, label)` + `renderCardArchives()` plní `#archTr` / `#archFood` / `#archTpl` / `#archFav` / `#archRcp` / `#archBody`. ⚠️ **Ty prvky samy JSOU `<details class="archd">`** — v testech a screenshotech se otevírají `document.getElementById('archFav').open = true`, ne hledáním vnořeného `<details>`.

**Vlastní přídavek nad rámec exportu (zadal Karel doslova):**
- 🆕 **Karta „Soukromí a data" (`#card-privacy`) v Nastavení, hned pod Zálohou.** Vždy viditelný souhrn: „🔒 Tvoje data zůstávají v tomhle telefonu. Nemám k nim přístup a na žádný server se nekopírují. Ven odchází jen to, co sám pošleš AI." Pod ním rozklikávací `<details id="privDet">` **(Podrobnosti ▾)** se sedmi oddíly (`.flabel`): *Kde data jsou · Co odchází ven, když použiješ AI · Co se neposílá nikdy · Jak to funguje bez tvého klíče · Chat s asistentem · Záloha · Smazání*, plus odkaz na `anthropic.com/legal/privacy`. Vlastní CSS třída `.priv`.
- 🔐 **Odstavec o směrování AI píše shim, ne zdroj appky.** Zdroj má jen prázdné kotvy **`#aiRouteLabel`** a **`#aiRouteNote`**; text dopisuje `routeNote(mine)` v `proxy_shim.js` — s vlastním klíčem „Kdo za AI platí / jde to přímo na tvůj klíč", ve sdíleném režimu „Jak to funguje bez tvého klíče / jde to přes můj přeposílací server, který dotazy neukládá". **Slovo „proxy", `workers.dev` ani sentinel `__proxy__` se ve zdroji appky nevyskytují** — hlídá to test `deleniZnalosti`.

### ⚡ v34 — AI asistent: chat, který zapisuje i radí
Export z Designu (jeho `sw.js` = v46.0), diff proti v33 bez shimu = **6 hunků**. **Čistý port ověřen bajtovou shodou s exportem** (MD5 `6bed2681d7c99886fbb67fc70ae4d5ef`), teprve pak přišly vlastní opravy. `node --check` OK, **21 testovacích sad bez jediné chyby**, živě ověřeno.

⚠️ **Nové a důležité: export z Designu už obsahuje shim.** Design staví na *nasazeném* souboru, takže jeho `index.html` nese na konci i můj proxy shim. Obyčejný `diff progres.html cd_uploadN.html` proto dal shim řádky do koncového kontextu posledního hunku a `patch` **selhal**. Správný postup:
1. `python3 build.py --no-shim --out /tmp/progres_noshim.html`
2. z uploadu v Pythonu **odstřihnout přesně ten samý shim blok** a ověřit `up.count(block) == 1` — což zároveň dokazuje, že Design se shimu nedotkl
3. `diff -U 6` těch dvou souborů **bez shimu** → `patch` (offset −18, žádné `.rej`)
4. teprve pak `python3 build.py` **se shimem** a `cmp` proti původnímu uploadu = bajtová shoda

**Co přinesl Design:**
- **Sekce `#tab-asistent`** (ve v34 skrytá sekce, **od v35 řádná záložka**) — `openAsst()` / `closeAsst()`, sdílí `_prevTab` s Nastavením. `syncFab()` schová FAB i v asistentovi.
- **Chat** — `#chatLog`, `#chatStatus`, návrhy `#chatSug` (`CHAT_SUG`), lišta `#chatInput` / `#chatSend` / `#chatPhoto` / `#chatMic` / `#chatFile`, akční bubliny `.cact` (modré) a `.cact.warn` (chyba). Historie v **`localStorage['progres_chat_v1']`**, ořezaná na **80 zpráv** (`chatLoad` / `chatSave` / `chatView`), tlačítko **Vymazat chat** (`#asstClear`). **Od v36 je konverzací víc — viz `progres_chats_v1`.**
- **Nástroje** (ve v34: 12 vlastních + serverový `web_search_20250305`; **od v37: 13 vlastních — přibyl `uprav_jidlo`**): `zapis_jidlo`, `smaz_jidlo`, `zapis_trenink`, `uprav_trenink`, `smaz_trenink`, `uloz_sablonu`, `uloz_recept`, `pridej_oblibene`, `zapis_telo`, `zapis_kroky`, `nastav_cile`, `dotaz_data`. Vykonává je `runAsstTool(nazev, vstup)`, smyčka `asstSend` jede **max 8 kol** nástrojů na jednu zprávu. Mazání jde přes `trashPush`, takže i to, co smaže asistent, je 14 dní v Archivu.
- **Kontext modelu** — `asstSystem()` skládá: profil a cíle, tréninky 120 dní, denní součty jídla 90 dní, položky jídla 21 dní, míry 30 záznamů, rekordy, šablony, oblíbená jídla, recepty a katalog cviků. Starší nebo podrobnější data si model doptá nástrojem `dotaz_data`.
- **Fotka a hlas** — až **3 fotky** (`resizeImage(f,1024,0.8)` → JPEG base64, jdou jako `image` bloky v posledním `user` message), náhledy `#chatImgs .thumb` s křížkem; diktování přes `SpeechRecognition` s `lang="cs-CZ"`.

**Vlastní opravy nad rámec exportu (schválil Karel):**
- 🔴 **`asstApi` obcházel proxy — asistent by pro Karla vůbec nefungoval.** Volal `fetch` na `api.anthropic.com` napřímo, takže by ve sdíleném režimu poslal sentinel `__proxy__` jako `x-api-key` → **401 při každé zprávě**. Vyřešeno **sekcí 3d v `proxy_shim.js`** — na dobu toho jednoho volání se `window.fetch` přesměruje na Worker (s `x-app-token`) a hned zase vrátí zpátky, i při výjimce. **Tělo se posílá beze změny**, takže `system`, `tools` i `web_search` projdou. (Od v37 tím projdou i opakované pokusy retry — fetch je podvržený po celou dobu volání.)
- 🔐 **`dotaz_data` s `co:"profil"` posílalo do konverzace API klíč.** Opraveno na kopii bez klíče. **Kdyby to Design v exportu vrátil zpět, opravit znovu.** (Ve v35–v38 zůstalo opravené.)

**Peníze (rozhodl Karel: „uvidíme podle útraty, zatím to nech jak to je"):**
- `web_search` má u asistenta `max_uses: 5` (u hledání jídla je od v33 `2`). Necháno záměrně.
- Změřeno: na demo datech `asstSystem()` = **11 309 znaků**, tělo požadavku 16 878 B. Na plných datech (69 tréninků / 480 jídel) = **44 155 znaků / 51 235 B**, tedy zhruba **5–6 centů za odeslání** — a každé kolo nástrojů to násobí (ukecanější konverzace ~20 centů). Živě změřeno ve v38: prázdná zpráva bez dat na desktopu = 7 170 vstupních tokenů.
- **Připravená úspora, kdyby útrata rostla:** zkrátit okna na tréninky 60 d, součty jídla 30 d, položky 7 d ⇒ **−43 %** kontextu prakticky bez ztráty. Druhá varianta: `cache_control` (prompt caching) a počítat `asstSystem()` jednou za `asstSend`, ne v každém kole.

### ⚡ v33 — archiv smazaných, zvýraznění nevyplněného, foto jídla, AI hledání nutričních hodnot
Export z Designu (jeho `sw.js` = v45.0), diff proti v32 bez shimu = 804 řádků / 31 hunků. **Čistý port ověřen bajtovou shodou s exportem**, teprve pak přišly vlastní opravy.

**Co přinesl Design:**
- **Archiv smazaných (14 dní)** — `state.trash[{tid,kind,at,item}]`, `TRASH_DAYS = 14`, `trashPush(kind,item)` (max 400 záznamů), `pruneTrash()` (běží při načtení). **Od v35 se nezobrazuje v jedné kartě, ale u jednotlivých karet** (`renderCardArchives`), **od v36 je souhrnný archiv v Nastavení navíc schovaný za rozklik.**
- **Zvýraznění toho, co ještě chybí** — `refreshNeedFill()` obtáhne modře nevybrané partie, prázdný název cviku a nevyplněnou sérii v Novém tréninku.
- **Vyfotit jídlo přímo z Talíře** — `#foodPhotoBtn`.
- **Varování u zpětného data** — `#plateDateWarn` + `fdateTouch`, `renderPlateDateWarn()`, `plateDateSync()`, `dayCheck()` (hlídá i přechod přes půlnoc).
- **Hledání jídla přes AI s přístupem na web** — `aiFoodSearch(q)` s nástrojem `web_search_20250305`; při selhání fallback `runOffSearch()` na Open Food Facts.
- **Držení tlačítek +/−** — `holdRepeat(btn,fn)`. ⚠️ **Reaguje na `pointerdown` a `click` schválně potlačuje** — v testech posílat `new PointerEvent('pointerdown',{bubbles:true,cancelable:true})`. Krok u KG je **0,5**.
- **FAB „+" se schovává v otevřené kartě** — `syncFab()`. ⚠️ Komentář nad funkcí v exportu tvrdí opak toho, co dělá kód; platí kód.

**Vlastní opravy nad rámec exportu (schválil Karel):**
- 🔴 **Obnoveno celé zálohování z v30–v32** — export ho smazal, viz varování „Design nevidí vlastní opravy" výše. Všech 13 míst vráceno.
- 🔴 **Oprava chyby v exportu — úsporná verze promptu AI plánu.** V `planContent(focus,usePhoto,compact)` byla vnořená šablona v jednoduchých uvozovkách → doslovný `${…}` v promptu. **Přesně ta chyba, kterou jsme opravovali už ve v28.**
- 🔴 **`aiFoodSearch` obcházel proxy.** Vyřešeno **sekcí 3c v `proxy_shim.js`**.
- 💰 **`max_uses` sníženo ze 4 na 2** (rozhodl Karel). **Pokud to příští export vrátí na 4, snížit zpátky na 2.** (Ve v34–v38 hodnota `2` zůstala.)

### ⚡ v32 — záloha jde na iPhonu opravdu do sdílecí nabídky
**Vlastní oprava mimo export z Designu.** Sdílecí nabídka z v30 se na iPhonu **nikdy neukázala**. Dvě nezávislé příčiny, obě opravené. `test_share32.mjs` 10/10.
- 🔴 **Příčina 1 — `.json` se sdílet nesmí.** Web Share API level 2 má **pevný seznam povolených přípon**. Řešení: zkusit **nejdřív `.json` a pak `.txt`** se stejným obsahem — `backupCandidates(text)` vyrobí oba, `shareableBackup(text)` vrátí první přijatý.
- 🔴 **Příčina 2 (latentní) — `await` sežral „user activation".** Řešení: **`doBackup()` už není `async` a před `navigator.share()` není žádné `await`**; pole `photos` se předehřívá v `openSettings()`.
- **Import bere i `.txt`** — `accept="application/json,text/plain,.json,.txt"`.
- **Co se záměrně NEzměnilo:** `backupName()` vrací dál `.json` a fallback `download()` posílá `application/json` — mění se **jen sdílený** soubor.

### ⚡ v31 — API klíč se nikdy nezapisuje do zálohy
**Bezpečnostní oprava mimo export z Designu.** **Skutečný `sk-ant-…` klíč se zapisoval do každé zálohy** — shim ve v29/v30 čistil jen sentinel. `test_backup31.mjs` 10/10.
- **`backupText()` klíč vynuluje** — jediné místo, kde vzniká obsah zálohy. ⚠️ `aiModel` se **záměrně nestripuje**.
- **Import: klíč patří zařízení, ne záloze.** Logika je **v obsluze importu, ne v `mergeState`**.
- 🔐 **Shim: `scrub()` čistí libovolný klíč** — `/("apiKey"\s*:\s*)"[^"]*"/g`.
- **Peníze i model:** vlastní klíč a sdílený Worker **stojí totéž** — Worker jede na Karlově klíči jako Cloudflare Secret. Rozdíly: vlastní klíč jede na `state.profile.aiModel`, Worker má natvrdo `claude-sonnet-4-5`.

### ⚡ v30 — záloha: stav, připomínka a uložení přes sdílecí nabídku
**Vlastní funkce mimo export z Designu** (Karel vybral A + C; varianta B — synchronizace přes Cloudflare Worker — je **odložená, ne zrušená**). `test_backup30.mjs` 13/13.
- **`state.lastBackup`** (`"YYYY-MM-DD"` nebo `null`) v `DEFAULT` i `mergeState`. ⚠️ **Je to pole na nejvyšší úrovni, ne v `profile`.**
- **Stav v Nastavení** — `#backupStatus` → „Poslední záloha: dnes / včera / před N dny / zatím nikdy", tečka zelená do prahu, pak oranžová. ⚠️ Třída `.bkp` / `.bkp.stale` sedí **na `#backupStatus` samotném**.
- **Připomínka na Přehledu** — `#backupNudge`, jen když je záloha starší než `BACKUP_WARN_DAYS` (**od v36 = 5**, původně 14) **a** `hasAnyData()`.
- **Soubor nese datum svého vzniku**; `AbortError` `lastBackup` vrátí zpět.
- **Poznámka k iOS:** sedmidenní limit ITP se **netýká** webových appek na ploše. Reálná rizika: ztráta telefonu, ruční vymazání dat webů, nedostatek místa, migrace na nový telefon.

### ⚡ v29 — sdílená AI přes proxy (bez API klíče) + obnova v28
Detailní dokumentace proxy a Workeru je v **`claude/proxy-ai-sdilena.md`**.
- **Uživatel nemusí nic nastavovat.** V Nastavení je banner „✅ AI je zapnutá" a tlačítko **„Použít vlastní klíč"**.
- **Architektura — shim mimo Design zdroj.** Proxy vrstva žije v samostatném `proxy_shim.js` a `build.py` ji připojuje až do hotového buildu před `</body>`.
  - ⚠️ **Diff z Claude Design se dělá proti výstupu `build.py --no-shim`** — a od v34 navíc **z exportu shim odstřihnout**.
- **Sentinel místo klíče.** `state.profile.apiKey = "__proxy__"`, takže všechny původní podmínky `if(!apiKey)` projdou beze změny. Sentinel se **nikdy nezapíše do localStorage ani do zálohy**.
- **Kotvy, na kterých shim stojí** (kdyby je Design přejmenoval, shim tiše přestane fungovat — vždy zkontrolovat): `aiCall`, `save`, `download`, `backupText`, `aiFoodSearch` (v33), `asstApi` (v34), `renderApiKeyBox`, `renderAiBox`, `renderPlanBox`, `renderChat` (v34), `#apiKeyBox`, `#chatLog` (v34), **`#aiRouteLabel` + `#aiRouteNote` (v35)**, `state.profile.apiKey`. **Ve v36–v38 se žádná kotva neměnila, shim zůstal beze změny (10 162 B / 10 098 znaků).**
- **Trik s DOM:** originální `renderApiKeyBox()` vykreslí vstup i s obsluhou, shim ho pak `appendChild`em přesune do skrytého bloku.
- **Peníze:** jediná tvrdá pojistka je **měsíční strop $10 v Anthropic Console**. Worker má `MAX_TOKENS_CAP = 8192` a přeposílá tělo požadavku beze změny.
- **App Store:** sdílený token v nativním obalu nebude bezpečnější — reálné vydání bude chtít přihlášení a kvótu na účet. Rozvaha je v **`claude/appstore-rozvaha.md`**.

### ⚡ v28 — AI foto rozepíše ingredience + oprava promptu AI plánu
- **Rozpis ingrediencí z fotky** — prompt žádá rozepsat KAŽDOU složku zvlášť včetně maker; `max_tokens` 1100 → **1600**. Pole `pol[{name,amount,kcal,p,c,f,on}]`.
- **Seznam „Nalezené ingredience"** — `#aiPolList`, karty `.aiitem`, `#aiPolSum`. `polSel()`, `polTot()`, `paintPol()` jsou **lokální uvnitř obsluhy `#aiGo`**.
- **Dvě akce nad výběrem** — `#aiPolPlate` a `#aiPolEat`.
- 🔧 **Vlastní oprava mimo export:** fallback věta v `planContent(...,compact=true)` byla v jednoduchých uvozovkách → doslovný `${…}`. **Ve v33 to Design vrátil zpět a muselo se to opravit znovu.**

### ⚡ v27 — objem cviku, detail šablony, datum na talíři, Sníst/Na talíř
`#exSwipe` (Síla / Objem), `dailyVol`, `chartExVolume`; šablony s detailem (`tplExDetail`, `data-tplload`, `data-tpldel`); `plateDate()`, `addFood(o)`; `#portionEat` / `#portionAdd`; prstenec „Bílkoviny dnes"; AI foto až 3 fotky; `profile.exPerDay` (3–7).
### ⚡ v26 — „Talíř" + historie jídel + statistiky v sekci + koš s potvrzením
### ⚡ v25 — swipe karty na Přehledu (`#statSwipe`) + FAB „+" (`FABMENU`) + týdenní přehledy v hubech
### ⚡ v24 — velká DB jídel (`FOODDB` ~313) + online Open Food Facts (`offSearch`)
### ⚡ v23 — redesign Tréninku: Fitko/Kardio/Jiné, kardio dial, série se slidery, pauza u cviku
### ⚡ v22 — přehlednější Jídlo + výběr cviku jen přes picker (1. běh skillu)
### ⚡ v21 — oblíbená jídla + makro koláč + steppery měr · v20 série karty · v19 recepty · v18 rozcestník hub · v17 5 obrazovek dle předlohy · v14–v16 základ

### Funkce (souhrn)
**Navigace (od v35):** pět záložek — **Přehled · Trénink · Tělo · Kalorie · Asistent**. Samostatná záložka „Progres" už neexistuje; statistiky se otevírají tlačítkem přímo v příslušné sekci a archiv smazaného sedí u karty, ke které patří.
**Asistent (záložka):** chat česky, který **umí zapisovat do appky** — jídlo (zapsat, **od v37 i opravit**, smazat), trénink (i úprava a smazání), šablona, recept, oblíbené, míry, kroky, cíle — a odpovídat na dotazy k datům. Vidí historii tréninků, jídla, měr a rekordů, umí **hledat na webu**, přijímá **až 3 fotky** a **diktování hlasem** (⚠️ od v41 **jen na desktopu** — na iOS tlačítko mizí, protože zamrzávalo appku; diktuje se systémovou klávesnicí). Každý zápis potvrdí modrou bublinou. **Od v37 denní součty a souhrny za období nepočítá sám** — appka mu je po každém zápisu vrací hotové (a v promptu má zákaz počítat z hlavy), takže hlášená čísla sedí na data. **Od v36 se konverzace neztrácejí** — lišta ☰ Historie / ＋ Nový / Vymazat chat, starší vlákna ve vysouvacím panelu zleva (až 30, s názvem, datem a počtem zpráv, dají se přejmenovat i smazat). Historie chatu je jen v zařízení a **není v záloze**.
**Přehled:** **od v36 bez kruhových dlaždic Váha a Konzistence** — jen swipe karty (Kalorie / Bílkoviny / Váha / Makro) + kalendář tréninků + **připomínka zálohy starší než 5 dnů**. **FAB „+"** = rychlé přidání.
**Trénink:** **od v44 bez AI plánu** — plán sestaví asistent (má nástroj `uloz_sablonu`); zkratka Nastavení drží, jak trénuješ (týdně, zkušenost, vybavení, cviků), a asistent z toho čte. **Od v42 jde zpětně upravit zaznamenaný trénink** — ikona posuvníků v historii načte záznam do formuláře včetně sérií a vah, uložení přepíše původní. Dál: typ Fitko / Kardio / Jiné; týdenní přehled + Statistiky (síla, objem cviku, rekordy, objem partií, **heatmapa konzistence za 20 týdnů**). Fitko = partie + cviky se sériemi (modré zvýraznění nevyplněného; steppery s držením, krok u kg 0,5; pauza; koš s potvrzením); výběr cviku přes picker; databáze + vlastní; **šablony s detailem, načtením a přejmenováním**; kroky; AI plán; historie **s možností pojmenovat trénink**.  Kardio = aktivita + kruhový číselník + slider metriky.
**Tělo:** **od v45 zkratka Můj profil** (pohlaví, věk, výška, cílová váha; aktuální váha se bere z posledního vážení). Dál: přehled váhy + míry (kruhové dlaždice `#bodyOverview`) + Statistiky; hero váha; míry-seznam; detailní záznam; fotky + porovnání.
**Kalorie (Talíř):** **od v43 jde rozložení maker prohlížet po jednotlivých dnech** (swipe pás: průměr za období + posledních 7 dní, v nadpisu kcal daného dne). **Od v42 jde zpětně upravit zapsané jídlo** — název, kalorie, makra i datum. Dál: slož jídlo z ingrediencí a „Sníst" (i zpětně na jiný den — s varováním a tlačítkem Dnes); databáze ~313 + online OFF + hledání přes AI s přístupem na web; sken kódu; AI foto — až 3 fotky, popis, jistota odhadu a rozpis ingrediencí; Recepty; Oblíbené; Dnešní makro; Historie jídel; **Statistiky nově včetně rozpadu maker, maker v čase, bílkovin, dnů v týdnu, tabulky dnů a top jídel**.
**Nastavení (gear):** cíl+profil, krokový cíl, AI (od v29 zapnutá rovnou, volitelně vlastní klíč), téma, **záloha se stavem a uložením přes sdílecí nabídku** (tlačítko se od v36 jmenuje prostě **„Záloha"**; od v31 bez API klíče, od v32 na iPhonu jako `.txt`), **karta „Soukromí a data" (v35)**, **archiv smazaných za rozklikem (v36)**, import (klíč zůstává ten zdejší, bere `.json` i `.txt`), CSV, .ics, ukázková data, smazání.
**Celoappkové (v38):** editované textové pole se po otevření klávesnice drží ve viditelné části obrazovky; chat lišta a modály se zvedají nad klávesnici.

### Technika
- Model: `sessions[{id,date,title,muscles[],type("Fitko"/"Kardio"/"Jiné"),dur,notes,exercises[{n,unit,sets:[{w,r,sec,done}],rest,dist,distUnit,mkind,pr}]}]` (**`title` přibylo ve v35**; starý `{w,s,r}` přes `exSets`), `body[]`, `foods[{id,date,name,kcal,protein,carbs,fat}]`, `steps{}`, `templates[]`, `customExercises[]`, `foodDb[]`, `recipes[…]`, `favorites[…]`, `trash[{tid,kind,at,item}]` (max 400, 14 dnů), `lastBackup:"YYYY-MM-DD"|null`, `profile{…,stepGoal,exPerDay,daysPerWeek,experience,equipment,apiKey,aiModel,plan}`. Fotky zvlášť v IndexedDB (db `progres`, store `photos`, keyPath `id`). **Historie chatu je mimo `state`** — `progres_chat_v1` (aktivní konverzace, max 80 zpráv) a **od v36 `progres_chats_v1`** = `{v:1,cur,list:[{id,title,ts,renamed,msgs[]}]}`, max **30** vláken; do zálohy nejde ani jedno. Vestavěná DB jídel = konst. `FOODDB` (~313). V paměti: `batch[]`, `batchServings`, `batchName`, `offResults`, `wkOff`/`selDay`/`foodWeekAll`, `photos[]`, `chatView`/`chatPend`/`chatBusy`, **`chats[]`/`curChat`**.
- Klíčové (v45): **`curWeight()`** (aktuální váha z posledního vážení), **`calcFrom(profil)`** (čistý výpočet; `calc()` a `#formCalc` ZANIKLY), **`#card-myprofile`/`renderMyProfile`/`#myProfWeigh`** (Tělo), **`#card-goal`/`renderGoalCard`/`#saveGoal`/`#gactInput`/`#ggoalInput`** (Jídlo), **`#unameInput`/`#saveUname`** (oslovení v Nastavení), **`#pstepgoalInput`** (cíl kroků v Tréninku). Z v44: **`#card-trsettings`/`renderTrainSettings`/`#trSetSave`** (nastavení tréninku; `renderPlanBox`, `planContent`, `salvagePlan`, `generatePlan`, `renderPlan`, `savePlanDay`, `#card-aiplan`, `#planBox` a `profile.plan` ODSTRANĚNY). Z v43: **`#macroSwipe`/`#macroDots`/`macroPieData`/`MACRO_DNU`/`chMacroPie0..N`** (rozpad maker po dnech; `#chMacroSplit` a `#macroSplitInfo` ODSTRANĚNY). Z v42: **`openFoodEdit`/`closeFoodEdit`/`foodEditSave`/`feNum`/`#foodEditBg`/`data-edith`** (úprava jídla) a **`editSessId`/`startSessEdit`/`endSessEdit`/`sessEditBar`/`#sessEditBar`/`#sessSubmit`/`data-edit-sess`/`ICON_SLID`** + **`fillSessionForm(sess,keepMeta)`** (úprava tréninku). Z v41: **`window.micBlocked`** (detekce iOS — na iOS se tlačítko mikrofonu nezobrazí) a **`window.micOff`** (úklid stavu diktování + 8s pojistka `micT`). Z v38: **`window.kbReveal`** + `--kb` (klávesnicová vrstva; IIFE mezi `Chart.defaults` a `initSeg()`, MIMO serviceWorker blok). Z v37: **`asstDayTotals(d)`** (hotové denní součty; pro dnešek cíl z `effTarget()`), marker **`_den`** → pole **`den`** v tool_result (dopočítává se **až po všech nástrojích kola** v `asstSend`), **`uprav_jidlo`**, **`dotaz_data co:'dnes'/'souhrn'`**, **retry 1× na síť/429/5xx/529** (pauza 1,8 s, české hlášky pro 429 a 503/529; **bez temperature — viz v38**), **`pause_turn` → continue** (jen když nejsou client tool_use). Z v36: **`chats`/`curChat`/`chatSave(touch)`/`chatOpen`/`chatNew`/`chatDelete`/`chatRename`/`chatClear`/`chatNewThread`/`chatTitleFrom`/`chatWhen`/`chatCount`/`renderChatList`/`openChatDrawer`/`closeChatDrawer`/`#asstHistBtn`/`#asstNew`/`.asst-bar`/`.drawer`/`.dw-bg`**, **`#archAll`/`#archAllSum`**, **`BACKUP_WARN_DAYS=5`**, `plurZprava`, `ICON_PEN`; z v35 **`switchTab(name,mode)`**, `showSolo`, **`#statsPool`/`showStats`**, **`openPrompt`/`#promptBg`**, **`renderArchiveBlock`/`renderCardArchives`/`#archTr…#archBody`**, **`foodDays`/`chartMacroSplit`/`chartMacroTime`/`chartProtein`/`chartFoodDow`/`renderFoodDaysTable`/`renderTopFoods`**, **`#card-privacy`/`#privDet`/`#aiRouteLabel`/`#aiRouteNote`**; dál `#tab-asistent`/`openAsst`/`asstSend`/`asstApi`/`asstSystem`/`ASST_TOOLS`/`runAsstTool`/`renderChat`, `#statSwipe`, `#backupNudge`/`#backupStatus`, `#exSwipe`/`#chExVol`, `#fab`/`FABMENU`/`syncFab`, `trashPush`/`pruneTrash`, `refreshNeedFill`, `holdRepeat`, `#foodPhotoBtn`, `#plateDateWarn`/`plateDateSync`/`dayCheck`, `aiFoodSearch`/`runOffSearch`, `foodUnit`/`LIQUID_RE`, `confirmDel`/`ICON_TRASH`, `plateDate`, `tplExDetail`, `dailyVol`/`chartExVolume`, `#aiPolList`/`paintPol`, `doBackup`/`backupText`/`backupStale`/`backupCandidates`/`shareableBackup`.
- **Odstraněno ve v36:** `renderProgressRings()` a `#progressRings`. **`ringSVG`, `isoAddDays`, `mondayOf` i CSS `.ring-cards`/`.ring-card` ZŮSTÁVAJÍ** — používá je `#bodyOverview` na Tělu a heatmapa. **Odstraněno ve v38:** `temperature` z těla `asstApi` (API 400) — NEVRACET.
- **Vrstvení (z-index):** `.chat-bar` 12, **drawer historie 45 / jeho backdrop 44**, `.modal-bg` 50, `.fab-bg` 55 — panel se tedy vždycky schová pod modál i pod FAB menu.
- **`apiKey` je jediné pole, které se ze stavu do zálohy nepropisuje** (v31) — a od v34 ho nevydá ani `dotaz_data` s `co:"profil"`. Kdo by přidával další citlivé pole, přidá ho do `backupText()`, do `scrub()` v shimu **i do té výjimky v `runAsstTool`**.
- ⚠️ **`chatSave()` bez parametru posouvá `ts` na teď.** Kdykoli se ukládá z důvodu, který není „přišla nová zpráva" (otevření, přejmenování, smazání jiného vlákna), volat **`chatSave(false)`** — jinak se historie přerovná a data vláken lžou (v36).
- ⚠️ **`doBackup()` nesmí být `async` a nesmí před `navigator.share()` nic `await`ovat** (v32).
- ⚠️ **`holdRepeat` tlačítka reagují na `pointerdown` a `click` schválně ruší** (v33) — testy musí posílat `PointerEvent`.
- ⚠️ **`planContent(focus, usePhoto, compact)` má compact jako TŘETÍ parametr a vrací POLE bloků** — v testech `JSON.stringify()`.
- ⚠️ **`foodDays(cut)` vrací OBJEKT klíčovaný datem, ne pole** (v35).
- ⚠️ **`todayISO` je `const todayISO=()=>…`, ne `function`** — kontrola existence helperů přes `grep "function todayISO"` hlásí falešný poplach. Datum je z **lokálního času** (žádný UTC posun po půlnoci) — ověřeno ve v37.
- ⚠️ **Znalost o proxy patří VÝHRADNĚ do `proxy_shim.js`.** Ve zdroji appky nesmí být `__proxy__`, `workers.dev`, `x-app-token` ani slovo „přeposílací" — zdroj má jen kotvy, text dopisuje shim (`routeNote`). Hlídá to test `deleniZnalosti` v `test_v35.mjs`.
- Zdroj: `progres.src.html` (CDN placeholdery, export z Designu + vlastní schválené úpravy) + `proxy_shim.js` (vrstva sdílené AI, mimo zdroj; sekce 3b `backupText`, 3c `aiFoodSearch`, 3d `asstApi`, `routeNote` pro kartu Soukromí) + `build.py` (inline knihoven + append shimu → `progres.html`; `--no-shim`, `--src`, `--out`) + `tools/check_cut.py` (nezávislý oracle) + testy `*.mjs` (Playwright, lokální instalace). Deploy jde přímo do `repo/index.html` a `git push`; `make_preview36.py` i složky `deploy/` a `progres-web/` **zanikly** — náhled řeší `python3 -m http.server`. **Vždy editovat `progres.src.html`, pak `python3 build.py`, pak `node --check`.**
- **Když Karel pošle úpravu z Claude Design:** viz skill `nasazeni-progres-z-design`. (V nové session **nejdřív `git -C repo pull` a ověřit `cmp`**.) `diff` proti výstupu `build.py --no-shim` **a proti uploadu s odstřiženým shimem** → ověřit helpery **i kotvy shimu** → `patch -U 6` do `progres.src.html` (offset −18) → build → **cmp/MD5 shoda** → **vrátit vlastní úpravy, které export smazal** (v37+v38 aparát!) → testy (0 chyb) → deploy. SW vlastní řadou (v14…v38), ne číslem z CD sw.js (teď v47.0).

## Roadmapa
- 🔴 **Testy neexistují.** 27 sad zaniklo s kontejnerem, na Macu zatím žádná.
  Do té doby se nasazuje bez pojistky. Pořadí: **runner** (musí umět obě
  konvence hlášení — `"errors": []` i `CHYBY: N` — jinak vyjde falešné
  „errors=NA" u sedmi sad), pak `test_v38` (nepřítomnost `temperature`),
  `deleniZnalosti` z `test_v35` (proxy oddělená od zdroje), `test_backup31`
  (klíč v záloze), `test_regress29`. Zbytek postupně.
- **Druhá rotace APP_TOKEN — neřešit zvlášť.** Token z v38.1 unikl do
  konverzace výstupem `check_cut.py` (před doplněním maskování). Riziko
  nízké (soukromá konverzace vs. starý token ve veřejném repu od v29).
  Zrotovat němě při příštím nasazení — maskování už je hotové.
- **Odložené, ne zrušené: varianta B zálohy** — automatická synchronizace do cloudu přes Cloudflare Worker (KV/R2). Karel ji odložil při zadání v30 a znovu při v32. Pozor: jakmile appku začnou používat testeři, ukládání jejich zdravotních dat na Karlův účet je otázka GDPR (souhlas, správce údajů, mazání). **Karta „Soukromí a data" z v35 dnes tvrdí, že data zůstávají v telefonu — kdyby se varianta B zapnula, ten text se MUSÍ přepsat.**
- ~~Chování klávesnice po v38~~ → **zpětná vazba přišla a je zapracovaná ve v40** (tři iterace, viz sekce v40). Sledovat, jestli se poskoky neobjeví jinde — první podezřelý je `#pickerSearch` (fixed modál).
- **Sledovat útratu v Anthropic Console (strop $10/měsíc) — po v34 je to hlavní věc k hlídání.** Uživatelé jsou tři (Karel, bratr, kolega). Účtuje se: běžné AI volání, `web_search` zvlášť (~$10 za 1 000 hledání; hledání jídla `max_uses:2` ≈ 2 centy, asistent `max_uses:5` ≈ až 5 centů) a hlavně **velký kontext asistenta** (~5–6 centů za odeslání při plných datech; v38.1 ověřovací volání 7 168 tokenů ≈ 2 centy). Karel rozhodl **zatím nic neškrtat a řídit se skutečnou útratou**. Připravená úspora: zkrátit okna kontextu (⇒ −43 %) a/nebo prompt caching.
- **Sledovat, jestli se chyby v počtech asistenta po v37 opakují.** Kdyby ano, další krok je přidat `den` součty i do dalších nástrojů (`zapis_trenink` → týdenní objem apod.) nebo zkrátit historii konverzace posílanou modelu.
- Testování Karlem a jeho testery, doladění dle zpětné vazby.
- Nápad k asistentovi: chat historie (až 30 vláken) není v záloze. Kdyby ji Karel chtěl přenášet mezi zařízeními, je to malá změna v `backupText()` + import (a řádek v kartě Soukromí).
- ~~Personalizace oslovení~~ → **hotovo ve v39** (`profile.name`, uživatel píše 5. pád sám).
- Odložený úkol: bezplatný SwiftUI/HealthKit spike na Macu jako průzkum před rozhodnutím o App Storu.
- Nápady: nativní SwiftUI (HealthKit, Watch, iCloud) → App Store (tam bude potřeba přihlášení + kvóta na uživatele místo sdíleného tokenu).

Splněno 31. 7. 2026 (odstraněno z roadmapy): záloha toolchainu (privátní
repo `progres-toolchain`), `.gitignore` na `.DS_Store` (kořen i `repo/`,
obojí v gitu), instalace Node, **maskování tajemství ve výstupu nástrojů**
(`check_cut.py` i `check_js.py` nahrazují hodnotu z `token.txt` i cokoli
ve tvaru tokenu — prefix + 40 hex — za `<APP_TOKEN>`; ověřeno negativně — umělá neshoda
uvnitř `var TOKEN` ukázala masku, ne hodnotu; předtím za 31. 7. vytekl
token do výstupu třikrát a pokaždé to zachytila pozornost, ne mechanismus).
