# Progres na App Store — rozvaha (27. 7. 2026)

Podklad k rozhodnutí, jestli a kdy přenést PWA „Progres" do App Store. Doplňuje `claude/stav-aplikace-progres.md` (technický stav) a `claude/proxy-ai-sdilena.md` (sdílená AI).

## Nejdřív rozplést tři různé cíle

Často splývají do jedné otázky, ale každý má jinou odpověď:

1. **„Chci HealthKit, Watch, sync mezi zařízeními."** → potřebuješ **nativní appku**, ale **ne nutně App Store**.
2. **„Chci to dát kamarádům, ať to zkusí."** → tohle už umíš dneska, stačí URL. App Store k tomu není potřeba.
3. **„Chci to pustit ven cizím lidem / zpeněžit."** → teprve tohle je App Store, a je to samostatný projekt (podpora, platby, GDPR, marketing).

## Tři technické cesty

### A) Obal webu (Capacitor / WKWebView)
Současný `index.html` se zabalí do nativního kontejneru. Nejrychlejší (dny), ale nejmenší přínos. Apple to hlídá **guidelinem 4.2 (Minimum Functionality)** — appka musí mít „features, content, and UI that elevate it beyond a repackaged website". Odmítá se browser-like UI, navigace jen z webu, žádné nativní schopnosti. Aby prošla, musí se dodělat nativní tab bar, push notifikace, offline obrazovka, splash screen.

⚠️ **Závěr: nedoporučuju.** PWA přidaná na plochu iPhonu dneska vypadá i chová se skoro stejně (ikona, fullscreen, offline). Obal přidá práci a roční poplatek, ale HealthKit/Watch stejně pořádně neotevře. Nejhorší poměr práce/přínos ze všech tří.

### B) Nativní SwiftUI přepis
Nejvíc práce (měsíce, ne dny), ale jediná cesta, která reálně něco přidá: HealthKit, Apple Watch, widgety, Live Activities, iCloud/CloudKit sync, plnohodnotné notifikace.

### C) Hybrid (nativní skořápka + postupné přepisování obrazovek)
V praxi obvykle nejhorší z obou — dvojí údržba a pořád ne úplně nativní pocit.

## Co to obnáší prakticky (bez ohledu na cestu)

- **Apple Developer Program — $99/rok.** Obnovuje se; když se nezaplatí, appka z App Store zmizí.
- **Mac s Xcode.** Bez Macu to jde (Xcode Cloud — 25 hodin/měsíc je v členství, MacStadium, MacinCloud, Capawesome Cloud), ale interaktivní ladění a simulátor chtějí skutečné macOS.
- **App Store Connect:** ikony, screenshoty ve víc velikostech, popis, klíčová slova, věková klasifikace, **privacy nutrition labels**.
- **Zásady ochrany osobních údajů** na veřejné URL — povinné.
- **Review u každé verze.** První schválení typicky dny. Konec dnešního „nahraju a za minutu je to živě".
- **Když přibudou účty:** povinné mazání účtu přímo v aplikaci; Sign in with Apple, pokud se nabízí přihlášení přes Google/Facebook.

## Dvě věci specifické pro tuhle appku

### 1. Sdílená AI na Karlově klíči se do App Store v současné podobě nedá
Token je dneska v souboru — na webu i v nativní binárce ho jde vytáhnout a pálit cizí kredit. Ve větším měřítku to $10 strop nezachrání, jen to dřív vypne AI všem. Před vydáním je potřeba jedno z:
- přihlášení uživatele + kvóta na účet (vlastní backend, ne jen Worker s jedním tokenem),
- předplatné přes **In-App Purchase** (Apple si bere **30 %**, resp. **15 %** pro Small Business Program a u předplatných po prvním roce),
- nebo vlastní klíč každého uživatele (což je přesně to, čemu jsme se ve v29 vyhýbali).

### 2. Guideline 5.1.2(i) — sdílení dat s AI třetích stran (od 13. 11. 2025)
Apple doplnil: *„You must clearly disclose where personal data will be shared with third parties, including with third-party AI, and obtain explicit permission before doing so."* Fotky jídla, váha a míry jsou osobní (a zdravotní) data a jedou na Anthropic → v appce musí být souhlasná obrazovka a v privacy policy popis toho, co se posílá a komu.

## Výhody přechodu (nativní cesta)

- **HealthKit** — váha a kroky se natáhnou samy, tréninky se zapíšou do Zdraví. Pro tuhle appku největší výhra.
- **Apple Watch** — zapisovat série přímo od stroje, tep u kardia.
- **iCloud/CloudKit sync** — dnes data žijí v localStorage jednoho prohlížeče. Přemazání telefonu nebo úklid Safari = konec, pokud není záloha.
- **Live Activities** — odpočet pauzy mezi sériemi na zamčené obrazovce. Přesně na to je to dělané.
- **Widgety**, plnohodnotné notifikace, plynulejší chod, kamera bez omezení web API.
- Důvěryhodnost a objevitelnost, kdyby to šlo ven.

## Nevýhody

- **Ztráta rychlé iterace** — dnes minuta od exportu k živé verzi u všech; pak build → upload → review → čekání → uživatelé musí aktualizovat.
- $99/rok napořád (+ případně cloud build).
- Mac na pořádné ladění.
- Podíl Applu z plateb a jeho měnící se pravidla.
- Dvojí údržba, pokud web poběží dál.
- **Čas** — nativní přepis jsou měsíce, během kterých se appka funkčně neposune.

## Doporučení

**Teď ne.** Zůstat u PWA, sbírat zpětnou vazbu od testerů (od v29 už bez nutnosti vlastního API klíče), nechat funkce ustálit.

**Naléhavější než App Store je záloha dat.** Data jsou v localStorage jednoho prohlížeče a stačí jedna nešťastná událost. Automatický export nebo jednoduchý sync by měl přijít dřív než jakákoli úvaha o nativní appce.

**Až přijde čas, jít rovnou nativně (cesta B), obal přeskočit.**

## Spouštěče — podle čeho poznat, že je čas

- Ručně zadávaná váha/kroky, které Zdraví už zná, začnou vadit → HealthKit → nativní.
- Někdo přijde o data → sync (řešitelné i na webu, nativně líp).
- Funkce se tři měsíce nemění a chceš to pustit cizím lidem → App Store.
- Chceš to zpeněžit → moment, kdy se $99 vrátí; ale nejdřív musí být vyřešená AI kvóta.

## Zdroje
- Apple Developer Program — [Membership Details](https://developer.apple.com/programs/whats-included/)
- Guideline 4.2 — [App Store Review Guidelines: Will Your Webview App Be Rejected?](https://www.mobiloud.com/blog/app-store-review-guidelines-webview-wrapper)
- Guideline 5.1.2(i) / AI — [TechCrunch, 13. 11. 2025](https://techcrunch.com/2025/11/13/apples-new-app-review-guidelines-clamp-down-on-apps-sharing-personal-data-with-third-party-ai), [Apple Developer News](https://developer.apple.com/news/?id=ey6d8onl)
- Build bez Macu — [Capawesome](https://capawesome.io/blog/how-to-build-and-deploy-ios-apps-without-a-mac/)
