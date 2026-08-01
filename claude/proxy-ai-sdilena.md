# Sdílená AI přes proxy (Cloudflare Worker) — jak to funguje

Stav: **NASAZENO ŽIVĚ ve verzi v29** (27. 7. 2026, SW cache `progres-v29.0`). Ověřeno naživo: AI přes proxy odpovídá z `g4m3ch4n63r.github.io`, banner „AI je zapnutá" v Nastavení, sentinel se nedostane do localStorage.

## K čemu to je

Testeři měli původně vkládat vlastní Anthropic API klíč (BYOK) a platit si dotazy sami — což nechtěli. Řešení: malý server (proxy), který drží **Karlův** klíč. Appka volá proxy místo Anthropicu, takže AI (foto jídla, čtení etikety, AI plán tréninku) funguje **hned a zdarma pro testera**. Kdo chce, může si v Nastavení dál vložit vlastní klíč a jet na svůj účet.

Žádné předplatné (Claude Pro/Max, ChatGPT Plus, Gemini) nejde použít k placení API volání třetí aplikace — ověřeno v dokumentaci Anthropicu. Proto proxy.

## Proxy — Cloudflare Worker

- URL: `https://progres-ai.chuanito-bonito.workers.dev`
- Kód: `/home/claude/proxy/worker.js` (verze 2.0), návod `/home/claude/proxy/nastaveni-proxy.md`
- Free tier: 100 000 dotazů/den

Proměnné v Cloudflare (Settings → Variables and Secrets) — **ověřený stav přes `/health` 27. 7. 2026**:

| Jméno | Typ | Hodnota |
|---|---|---|
| `ANTHROPIC_KEY` | Secret | `sk-ant-…` — **zadává výhradně Karel**, Claude ho nikdy nevidí ✅ nastaveno |
| `APP_TOKEN` | Secret | (hodnota jen v Cloudflare Secrets, nikde jinde) ✅ nastaveno |
| `ALLOWED_ORIGIN` | Text | `https://g4m3ch4n63r.github.io` ✅ |
| `MAX_TOKENS_CAP` | Text | `8192` ✅ (Karel opravil ze 4096; AI plán si říká o 8000 — při 4096 se odpověď usekla) |
| `IP_PER_MIN` | Text | `8` (best effort, viz níže) |

Endpoint `GET /health` (vyžaduje `x-app-token`) vrátí, co je nastavené, bez prozrazení hodnot. Ověřeno naživo, že request s `max_tokens: 8000` projde.

**Strop útraty v Anthropic Console: 10 $/měsíc** (Karel potvrdil). To je jediná tvrdá pojistka — viz níže.

## Co proxy chrání a co ne

Funguje spolehlivě: kontrola Originu (403 mimo appku v prohlížeči), sdílený token (401), strop délky odpovědi, whitelist modelů.

**Nefunguje spolehlivě: limit dotazů za minutu.** Ověřeno naživo — 11 dotazů rychle za sebou prošlo všech 11. Důvody: Cloudflare KV cachuje čtení 60 s (počítadlo se v rámci minuty neprojeví) a Worker běží v efemérních izolátech, takže ani počítadlo v paměti nepřežije. Oficiální Rate Limiting binding jde nastavit jen přes soubor `wrangler.toml`, ne přes dashboard. Řešení, kdyby došlo ke zneužití: Durable Objects.

**Jediná tvrdá pojistka peněženky je měsíční strop útraty v Anthropic Console** (Billing → Limits), nastaven na **10 $/měsíc**. Token je čitelný ve zdroji appky a Origin jde mimo prohlížeč podvrhnout — strop útraty ne.

## Zapojení do appky — proxy vrstva (shim)

Klíčová věc: **proxy kód není v `progres.src.html`.** Je v samostatném `/home/claude/proxy_shim.js` a `build.py` ho vkládá až do hotového `progres.html` před `</body>`. Díky tomu zůstává workflow s Claude Design nedotčené — příští export z Designu se dá pořád ověřit bajt na bajt.

```
python3 build.py              # build včetně proxy vrstvy (nasazuje se tenhle)
python3 build.py --no-shim    # čistý build bez proxy — pro porovnání s exportem z Designu
python3 build.py --src X --out Y
```

Ověřeno: `build.py --no-shim` dává soubor **bajt na bajt shodný** s exportem v28 z Claude Design. Shim je čistý append `<script>…</script>` před `</body>` (+6 071 znaků), takže jeho odebrání vrací přesně původní soubor — to se dá kdykoli ověřit řetězcovou náhradou.

Jak shim funguje, aby nemusel duplikovat velké render funkce:

- Do `state.profile.apiKey` se vloží sentinel `"__proxy__"`, takže všechny původní podmínky `if(!apiKey)` projdou beze změny.
- `own(k)` = klíč začínající `sk-` a různý od sentinelu → uživatel má vlastní klíč.
- Přepsané `aiCall`: vlastní klíč → přímo na Anthropic (jako dřív), jinak → POST na proxy s hlavičkou `x-app-token`. Hezké hlášky pro 429 a 401.
- Obal `save()` sentinel před zápisem vymaže a po zápisu vrátí → do localStorage se nikdy nedostane. `ensure()` na začátku `save()` pokrývá i import / reset / ukázková data, kde se `state` přepíše.
- Obal `download()` ho vyčistí i z JSON zálohy.
- Obal `renderApiKeyBox()` nechá originál vykreslit vstup pro klíč (i s jeho vlastní obsluhou), pak jeho uzly **přesune** (`appendChild` obsluhu zachová) do skrytého bloku a nahoru dá banner „✅ AI je zapnutá" + tlačítko „Použít vlastní klíč". Nadpis karty se přepíše z „AI klíč" na „AI".
- `renderAiBox` / `renderPlanBox` jen doplní sentinel a zavolají originál.

Shim je závislý jen na těchto kotvách: `aiCall`, `save`, `download`, `renderApiKeyBox`, `renderAiBox`, `renderPlanBox`, `#apiKeyBox` a `state.profile.apiKey`. Před nasazením nové várky z Designu stačí ověřit, že tyhle existují (v v28 ověřeno — beze změny).

## Testy

`/home/claude/test_proxy27.mjs` — 10 kontrol: sentinel není v localStorage ani v záloze, banner a rozbalovací panel v Nastavení, bloky jídla i plánu už nehlásí „potřebuje klíč", `aiCall` bez klíče jde na proxy s tokenem a bez `x-api-key`, s vlastním klíčem přímo na Anthropic, odebrání klíče vrátí sdílený režim, hlášky 429/401. **0 chyb** (ověřeno i na bázi v28).

`/home/claude/test_regress29.mjs` — regrese v27/v28 s aktivním shimem (všech 5 záložek, objem cviku, šablony, datum na talíři). `/home/claude/test_v28_live.mjs` — rozpis ingrediencí end-to-end (podstrčí falešnou odpověď přes `window.aiAnalyzeFood`, nahraje `testfood.jpg` do `#aiFile`, klikne `#aiGo`) → 3 položky, odškrtnutí přepočítá souhrn, „na talíř" vloží vybrané do `batch`. **0 chyb.**

Naživo ověřeno proti běžícímu Workeru: `/health` OK, bez tokenu 401, cizí Origin 403, špatný token 401, platný dotaz 200 s odpovědí, CORS preflight z `g4m3ch4n63r.github.io` OK, `max_tokens: 8000` projde.

## Až na App Store

Předplatné ani proxy na tom nic nemění co do Applu: PWA potřebuje nativní obal (Capacitor) a nějakou nativní funkci (HealthKit, kamera, notifikace). Vývojářský účet ~99 $/rok. Kdyby se za appku nebo AI vybíraly peníze, musí to jít přes In-App Purchase (15 % pro malé vývojáře pod 1 M $, jinak 30 %).

Pozn.: v nativním obalu už token v klientovi nebude o nic bezpečnější než teď — pořád platí, že hlavní ochranou je strop útraty. Při reálném vydání by se to mělo nahradit přihlášením uživatele (účet + kvóta na uživatele), ne sdíleným tokenem.
