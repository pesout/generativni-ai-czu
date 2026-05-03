# VIII. n8n workflows, integrace s aplikací a MCP servery

## Prerekvizity

> Pro toto cvičení potřebujete:
> - Funkční n8n na `n8n.<jmeno>.aibr.cz` z [cvičení 7](cviceni7.md)
> - Vaši aplikaci běžící na `<jmeno>.aibr.cz` z [cvičení 6](cviceni6.md)
> - VS Code (Cursor / Antigravity) připojený k serveru přes Remote-SSH
> - OpenAI API klíč ze [cvičení 4](cviceni4.md)

### 1) Rychlá kontrola

```bash
docker ps
curl -I https://<jmeno>.aibr.cz
curl -I https://n8n.<jmeno>.aibr.cz
```

Aplikace by měla vrátit `HTTP/2 200`, n8n `HTTP/2 401` (Basic Auth) nebo `200`.

---

## Praktické příklady workflow

### Příklad 1 -- Ranní AI brief (osobní)

**Cíl:** Každé ráno v 7:00 vám přijde na e-mail krátké shrnutí dne -- počasí, dnešní události z kalendáře a tři novinky z RSS, vše zpracované LLM do jednoho čtivého odstavce.

**Uzly v pořadí:**

1. **Schedule Trigger** -- spouští workflow podle Cron výrazu (`0 7 * * *`)
2. **HTTP Request** -- volá API počasí
3. **Google Calendar -- Get Many Events** -- dnešní události
4. **HTTP Request** -- stáhne RSS feed
5. **Merge** -- spojí výstupy do jednoho objektu
6. **OpenAI** (chat completion) -- LLM napíše ranní brief podle systémového promptu
7. **Send Email** -- odešle text na vaši adresu

**Použité techniky:** Cron / Schedule trigger, HTTP Request pro veřejná REST API, paralelní větve a Merge, prompt engineering, data summarization přes LLM, hotové integrace (Google Calendar, Gmail) bez ručního OAuth.

### Příklad 2 -- Zpracování poptávky (firemní)

**Cíl:** Když někdo na webu vyplní poptávkový formulář, workflow ho automaticky klasifikuje (zájem / spam / mimo nabídku), vygeneruje koncept odpovědi, zapíše do Google Sheets a obchodníkovi pošle upozornění na Slack.

**Uzly v pořadí:**

1. **Webhook Trigger** -- veřejná URL volaná při odeslání formuláře
2. **Edit Fields (Set)** -- normalizace vstupu (trim, lowercase e-mailu, timestamp)
3. **OpenAI** -- LLM zařadí zprávu (`zajem` / `spam` / `mimo`)
4. **Switch** -- větvení podle klasifikace
5. **OpenAI** -- pro `zajem` vygeneruje koncept odpovědi
6. **Google Sheets -- Append Row** -- zápis řádku do tabulky leadů
7. **Slack -- Send Message** -- notifikace obchodníkovi se shrnutím a odkazem

**Použité techniky:** webhook trigger, validace a normalizace vstupu, LLM jako klasifikátor, Switch pro podmíněné větvení, strukturovaný prompt pro generování textu, persistence do Sheets, human-in-the-loop (LLM nepošle odpověď přímo, ale obchodníkovi ke schválení).

> Druhý příklad si dnes v jednodušší podobě (bez klasifikace a Slacku, výstup do CSV) sami postavíte v sekci 3.

---

## Základy n8n UI

V [cvičení 7](cviceni7.md) jste si zkusili Manual Trigger a Set node. Doplníme zbytek konceptů, které dnes využijete.

### 2) Triggery

**Trigger** je vždy první uzel a určuje, kdy workflow běží. V postranním panelu (přes **Add first step** nebo **+** na plátně) najdete v záložce **On a trigger**:

- **Trigger manually** -- ručně, vhodné při ladění
- **On a schedule** -- Cron / interval
- **On webhook call** -- veřejná HTTPS URL spustí workflow HTTP požadavkem
- **On form submission** -- n8n vyrobí formulář na své doméně
- Triggery uvnitř integrací -- *On new email (Gmail)*, *On new row (Sheets)*, *On new message (Slack)* atd.

### 3) Přidávání a propojení uzlů

1. Najeďte na pravý okraj uzlu -- objeví se **+**
2. Klik -- otevře se panel s vyhledáváním
3. Začněte psát (např. `openai`, `http`, `set`, `merge`, `if`)
4. Vyberte uzel -> propojí se čarou s předchozím

Uzly přetahujete, mažete (`Delete`), kopírujete (`Ctrl+C`/`V`). Větvení vznikne tak, že z jednoho uzlu vedete `+` do dvou různých následníků.

### 4) Konfigurace uzlu

Klik na uzel otevře jeho detail. Většina uzlů má:

- **Credentials** -- přihlašovací údaje (uloží se jednou, používáte opakovaně)
- **Parameters** -- co uzel dělá (URL, model, název listu...)
- **Options** -- doladění (timeout, retry...)

**Přístup k datům z předchozích uzlů:** v textovém poli přepněte na **Expression** mode. Hodnoty referencujete přes `{{ $json.nazevPole }}` nebo `{{ $('Nazev Uzlu').item.json.pole }}`. n8n napravo ukazuje živý náhled.

### 5) Spuštění a ladění

- **Execute step** (na otevřeném uzlu) -- spustí workflow jen po tento uzel
- **Execute workflow** (dole na plátně) -- celé workflow od triggeru
- Po běhu: zelený rámeček = OK, červený = chyba; v panelu **OUTPUT** vidíte data uzlu
- **Executions** v levém menu -- historie všech běhů s detailem

### 6) Aktivace

Workflow s reálným triggerem (Webhook, Schedule) musíte aktivovat přepínačem **Inactive / Active** vpravo nahoře. Bez aktivace n8n volání ignoruje. Manual trigger aktivaci nepotřebuje.

> Webhook má dvě URL: **Test** (jen když je editor otevřený a kliknete *Listen for test event*) a **Production** (jen když je workflow Active). Při ladění Test, na ostro Production.

---

## Integrace s vaší aplikací -- vše přes Codex

Tato integrace je pouze příklad. Zvolte si vlastní podle potřeb vaší aplikace.

Cíl: na webu přibude kontaktní formulář, jeho odeslání pošle data do n8n, n8n nechá LLM napsat návrh odpovědi a uloží řádek do CSV souboru. Vše ve dvou promptech přes Codex, bez ručního klikání v UI.

> **Architektura:** Frontend volá backend (`POST /api/contact`), backend přepošle požadavek na n8n webhook. Backend slouží jen jako tenká proxy -- veškerou logiku (LLM, CSV) drží n8n. Formulář tak nikdy nevolá cizí doménu, čímž se vyhneme problémům s CORS.

### 7) Prompt 1 -- kontaktní formulář v aplikaci

Na serveru v adresáři aplikace:

```bash
cd <vase-aplikace>
codex
```

Prompt:

```
V této aplikaci přidej kontaktní formulář. Nejdřív si projdi strukturu
projektu a vizuální styl, ať to nově přidané vypadá a chová se jako
zbytek aplikace -- nehádej tech stack, zjisti si ho.

1. Frontend -- nová stránka /kontakt s formulářem (jméno, e-mail,
   zpráva). Po odeslání ukaž rozumnou hlášku o úspěchu/chybě. Přidej
   odkaz do hlavní navigace.

2. Backend -- endpoint POST /api/contact, který přijme JSON, validuje
   a přepošle ho jako JSON na URL z proměnné prostředí
   N8N_CONTACT_WEBHOOK_URL. Žádná další logika, všechno řeší n8n.
   Backend vrátí 200 při úspěchu, 502 když n8n nedostupný.

3. Přidej N8N_CONTACT_WEBHOOK_URL do .env.example (s prázdnou
   hodnotou a komentářem). Hodnotu zatím v .env nech prázdnou,
   doplníme ji později.

4. Lokálně restartuj aplikaci a zkontroluj v prohlížeči, že
   stránka /kontakt funguje a formulář vypadá v pořádku.

5. Commitni s popisnou commit message a pushni na GitHub.
```

Po skončení si v prohlížeči lokálně otevřete `/kontakt` a zkontrolujte vzhled. Pokud něco nesedí, doinstruujte krátkým follow-up promptem.

### 8) Prompt 2 -- n8n workflow + e2e test

Druhý prompt poběží na serveru. Než ho spustíte, vyzvedněte si dva klíče:

- **n8n API klíč:** v n8n klikněte na svůj avatar / iniciály v levém menu (úplně dole) -> **Settings** -> **n8n API** -> **Create an API key**. Zkopírujte (zobrazí se jen jednou).
- **OpenAI API klíč:** ten už máte ze [cvičení 4](cviceni4.md).

Otevřete `~/n8n/.env` ve VS Code a přidejte:

```
N8N_API_KEY=<vas-n8n-api-klic>
OPENAI_API_KEY=<vas-openai-klic>
```

V terminálu na serveru (z VS Code Remote-SSH):

```bash
cd ~
codex
```

Prompt:

```
Cíl: propojit moji aplikaci s n8n. Aplikace už má /kontakt, který přes
backend posílá JSON na N8N_CONTACT_WEBHOOK_URL. Workflow má webhook
přijmout, nechat OpenAI napsat zdvořilý český návrh odpovědi a připojit
nový řádek do CSV souboru. Veškerý setup udělej skriptovaně, žádné
klikání v n8n UI.

Než začneš, projdi si ~/AGENTS.md, ~/n8n/ a ~/apps/<repozitar>, ať máš
kontext. Citlivé klíče máš v ~/n8n/.env, nikdy je nepiš do logů
ani do commitů.

1. Vytvoř ~/data/contact-form.csv s rozumnou hlavičkou (datum, jméno,
   e-mail, zpráva, navržená odpověď). Soubor vytvoříš teď ty, n8n
   bude jen připisovat řádky. Pozor: n8n kontejner běží jako uživatel
   s UID 1000, takže ~/data i CSV musí být zapisovatelné pro UID 1000
   (typicky chown -R 1000:1000 ~/data po vytvoření).

2. Bind-mountni ~/data jako /data uvnitř n8n kontejneru (úprava
   ~/n8n/docker-compose.yml), restartuj n8n a ověř, že soubor je
   vidět a zapisovatelný zevnitř (docker exec).

3. Vytvoř v n8n credential pro OpenAI (z OPENAI_API_KEY) a workflow
   "Contact form -> CSV + AI draft". Workflow má tři uzly: webhook
   (path "contact-form", method POST), OpenAI pro generování českého
   návrhu odpovědi a uzel, který do /data/contact-form.csv připíše
   nový řádek (datum, jméno, e-mail, zpráva, návrh odpovědi).
   Workflow pak aktivuj.

   Pro samotný setup zvol nejspolehlivější přístup -- buď n8n REST API
   (https://n8n.<jmeno>.aibr.cz/api/v1/, header X-N8N-API-KEY z
   ~/n8n/.env), nebo n8n CLI uvnitř kontejneru přes
   docker exec n8n n8n import:workflow / import:credentials z JSON
   souborů, který si připravíš. Konkrétní názvy uzlů, model OpenAI
   a způsob zápisu do CSV zvol podle toho, co aktuální verze n8n
   stabilně umí (pozor, append mode u některých file uzlů má bugy --
   pokud narazíš, použij alternativu, např. shellový append přes
   Execute Command).

   Výstup OpenAI musí být v češtině, krátký, zdvořilý, reagovat
   na obsah zprávy.

4. Doplň webhook Production URL do .env aplikace a restartuj backend.

5. End-to-end test (přes curl, žádné klikání):
   - pošli testovací POST na /api/contact
   - ověř, že do CSV přibyl správný řádek s neprázdnou AI odpovědí
   - ověř v n8n executions, že workflow proběhl OK
   - pošli druhý test s jinými daty, ať máš jistotu

6. Aktualizuj ~/AGENTS.md o info o /kontakt, webhook URL, CSV souboru
   a o vytvořeném workflow.

Subdoména: [DOPLŇTE, např. novak.aibr.cz]
Repozitář aplikace: [DOPLŇTE název adresáře v ~/apps/]
```

Po dokončení otevřete v UI workflow a podívejte se, jak ho Codex postavil. **Executions** ukáže oba testovací běhy.

---

## MCP servery

Workflow v n8n a **MCP server** se na první pohled můžou plést, ale jsou to dvě různé věci:

- **n8n workflow** je *autonomní automatizace* -- spustí se na základě definovaných pravidel (Cron, webhook atd.) a udělá konkrétní činnosti v daném pořadí.
- **MCP server** je *nástroj pro AI agenta* -- agent (Codex, Claude, Cursor a další) se k němu připojí a získá nové schopnosti. Agent rozhoduje, kdy je potřeba ho spustit.

Workflow je tedy sada jasných pravidel spouštěných při konkrétní akci. MCP server přináší nástroje pro agenta, které může použít, když něco potřebuje vykonat.

> **MCP** = *Model Context Protocol*, otevřený standard od Anthropicu pro propojení LLM klientů s externími nástroji. Codex, Claude Desktop i většina moderních AI editorů ho umí.

### 9) Prompt -- najdi a nastav vhodný MCP

Cíl: vyzkoušet si, k čemu MCP server může sloužit a jak ho agent použije. Konkrétní nástroj nehraje roli.

V adresáři vaší aplikace lokálně:

```bash
cd <vase-aplikace>
codex
```

Prompt:

```
Projdi si tuhle aplikaci a zjisti, čím se zabývá. Pak mi:

1. Doporuč jeden third-party MCP server, který by mi v této aplikaci
   pomohl při dalším vývoji. Vyber takový, který si dokážeš sám
   nainstalovat a otestovat -- bez složitého onboarding, bez OAuth
   přes prohlížeč, bez placených klíčů. Buď ať běží lokálně bez klíčů,
   nebo ať si potřebné údaje vytáhneš z mého .env (a zeptáš se mě).

2. Krátce mi vysvětli, proč právě tenhle a co s ním zvládnu, co předtím ne.

3. Nainstaluj ho a zaregistruj v Codex CLI. Otestuj, že se Codex
   připojí a vidí jeho nástroje.

4. Navrhni mi 3 prompty, které si v Codexu sám zkusím, abych viděl,
   co MCP umí a jak ho Codex použije.
```

> **Tip:** Po nastavení napište v Codexu `/mcp` -- uvidíte připojené MCP servery a jejich nástroje.

Po dokončení vyzkoušejte alespoň jeden z navržených promptů. Sledujte, jak Codex volá MCP nástroje -- to je ten okamžik, kdy LLM sahá ven za hranice konverzace.

---

## Další zamyšlení -- integrace OpenClaw (nebo podobné alternativy)

Tématem k zamyšlení do [cvičení 9](cviceni9.md) jsou osobní AI agenti, kteří běží trvale na vašem počítači (nebo VPS) a mluví s vámi přes Telegram, WhatsApp, Discord a další chaty. Mají perzistentní paměť, ovládají soubory, prohlížeč i příkazovou řádku.

- [**OpenClaw**](https://openclaw.ai) -- open-source agent, lokálně na Mac/Windows/Linux, 100+ skillů, libovolný LLM (Claude, OpenAI, lokální).
- [**NanoClaw**](https://nanoclaw.dev) -- bezpečnější odlehčená alternativa OpenClaw, každý agent v izolovaném Docker kontejneru, postavená nad Anthropic Claude Agent SDK.
- [**Hermes Agent**](https://hermes-agent.nousresearch.com) (Nous Research) -- self-improving agent s perzistentní pamětí napříč konverzacemi, učí se z používání nové skilly
- [**Manus**](https://manus.im) -- plně autonomní cloudový agent, dostane vysokoúrovňový cíl a sám si rozplánuje a odpracuje kroky bez dohledu.

**Zamyslete se:** která z vašich rutinních úloh by se hodila spíš na **n8n workflow** a která spíš na **osobního agenta** (mluvíte s ním a on dělá věci za vás)?

---

## Shrnutí

Po dokončení tohoto cvičení byste měli mít:

1. Přehled o struktuře typického n8n workflow a jeho technikách
2. Jistotu v základech n8n UI -- triggery, propojování, expressions, ladění, aktivace
3. Integrované workflow v rámci vaší aplikace
4. Aktualizovaný `~/AGENTS.md`
5. Funkční third-party MCP server v Codexu a osahaný koncept MCP

