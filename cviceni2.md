# II. Vibe-coding: výběr projektu, architektura a vývoj

## Rychlá kontrola nastavení z cvičení 1

Než začneme, ověřte si v terminálu, že máte vše z [cvičení 1](cviceni1.md) správně nastavené:

1. `git --version` -- Git je nainstalovaný
2. `ssh -T git@github.com` -- SSH klíč je propojený s GitHubem
3. `node -v` -- Node.js je nainstalovaný
4. `npm -v` -- npm je nainstalovaný
5. `codex --version` -- OpenAI Codex CLI je nainstalovaný
6. Editor (VS Code / Cursor / Antigravity) je nainstalovaný a funkční
7. Testovací repozitář je naklonovaný a přístupný
8. V repozitáři se nachází vygenerovaný `AGENTS.md soubor`

## Výběr tématu a komplexní popis projektu

Studenti si vyberou svůj projekt (viz [příklady projektů z cvičení 1](cviceni1.md)). Cílem je vytvořit podrobný popis celé aplikace, který bude později sloužit jako vstup pro všechny další kroky:

- extrakce popisu frontendu pro AI tool [Lovable](https://lovable.dev)
- extrakce implementačních částí pro AI agenta Codex

### Šablona komplexního popisu projektu

Vyplňte `...` svými odpověďmi a smažte nerelevantní sekce. V případě nejistoty je vhodné se na jednotlivé koncepty, technické pojmy nebo best practices doptat AI (LLM) nebo cvičících.

```
NÁZEV PROJEKTU: ...
JEDNOVĚTÝ POPIS: ... (např. "Aplikace pro sledování výdajů z fotografií účtenek")

PROBLÉM, KTERÝ ŘEŠÍ:
...

CÍLOVÝ UŽIVATEL:
... (např. student, rodina, malý podnikatel, učitel)

HLAVNÍ FUNKCE (MVP) -- co aplikace musí umět v první verzi:
1. ...
2. ...
3. ...
4. ...
5. ...

CO APLIKACE V MVP NEBUDE UMĚT (out of scope):
- ...
- ...

JAK SE PROJEKT LIŠÍ OD EXISTUJÍCÍCH ŘEŠENÍ:
... (např. "Na rozdíl od Splitwise se zaměřuje čistě na účtenky z českých obchodů")

STRUKTURA REPOZITÁŘE:
- frontend/  -- uživatelské rozhraní (bude vytvořeno v Lovable, pak upravováno lokálně)
- backend/   -- serverová logika a API (bude implementováno lokálně pomocí Codexu)

TECH STACK:
- Frontend: ... (bude vytvořen v Lovable -- typicky React + Vite + Tailwind)
- Backend framework: ... (např. NestJS/Express, Python/Flask)
- Databáze: ... (např. SQLite pro jednoduchost, MySQL, Supabase, žádná pokud stačí soubory)
- Další knihovny / API: ... (např. OpenAI SDK, Tesseract.js, Weather API)

OBRAZOVKY / STRÁNKY APLIKACE:
1. ... (např. "Hlavní stránka -- přehled výdajů za aktuální měsíc")
2. ... (např. "Detail účtenky -- seznam položek z jedné účtenky")
3. ... (např. "Nahrání účtenky -- formulář pro upload fotky")
4. ...
5. ...

NAVIGACE MEZI STRÁNKAMI:
... (např. "Spodní navigační lišta s ikonami: Přehled, Nahrát, Historie")

BRANDING:
- Název aplikace (jak se zobrazuje v UI): ...
- Logo: ... (např. "Jednoduché textové logo", "Ikona + text", "Zatím bez loga")
- Tón komunikace v aplikaci: ... (např. "Přátelský a neformální", "Profesionální", "Hravý s emojis")
- Claim / slogan (nepovinné): ... (např. "Vaše účtenky pod kontrolou")

BAREVNÉ SCHÉMA:
- Primární barva: ... (např. "#4F46E5 -- indigo", "modrá", "zelená -- příroda")
- Sekundární barva: ... (např. "#10B981 -- zelená pro úspěch")
- Barva pozadí: ... (např. "Bílá / světle šedá", "Tmavý režim")
- Barva textu: ... (např. "Tmavě šedá #1F2937")
- Accent / CTA barva: ... (např. "Oranžová pro hlavní tlačítka")
- Styl: ... (např. "Moderní a minimalistický", "Barevný a hravý", "Korporátní")

TYPICKÝ PRŮCHOD UŽIVATELE (user flow):
1. Uživatel otevře aplikaci a vidí: ...
2. Klikne na: ...
3. Vyplní / nahraje: ...
4. Systém zpracuje a zobrazí: ...
5. Uživatel může dále: ...

AUTENTIZACE A UŽIVATELÉ:
- Potřebuje aplikace přihlášení? ... (ano / ne)
- Způsob přihlášení: ... (např. email + heslo, Google OAuth, žádné -- single user)
- Uživatelské role: ... (např. admin + běžný uživatel, pouze jedna role, N/A)
- Co vidí nepřihlášený uživatel: ... (např. nic -- redirect na login, veřejná landing page)

DATA, KTERÁ UŽIVATEL ZADÁVÁ:
... (např. fotka účtenky, název obchodu, kategorie nákupu)

DATA, KTERÁ APLIKACE UKLÁDÁ:
... (např. rozparsované položky z účtenky, celková cena, datum nákupu, kategorie)

EXTERNÍ ZDROJE DAT:
... (např. OpenAI API pro OCR účtenek, Weather API pro počasí, žádné)

FORMÁT VÝSTUPU PRO UŽIVATELE:
... (např. tabulka, graf, notifikace, exportovaný CSV soubor)

CHYBOVÉ STAVY:
- Chybný vstup od uživatele: ... (např. "Zobrazit validační hlášku u formuláře")
- Výpadek externího API: ... (např. "Zobrazit chybovou hlášku a nabídnout retry")
- Prázdný stav (žádná data): ... (např. "Zobrazit ilustraci s textem 'Zatím nemáte žádné účtenky'")
- Pomalé načítání: ... (např. "Zobrazit skeleton loading")

POPIS JEDNOTLIVÝCH FUNKCÍ:

Pro každou funkci z MVP seznamu vyplňte:

Funkce 1: ... (název, např. "Nahrání a zpracování fotky účtenky")
- Popis: ... (Co přesně tato funkce dělá z pohledu uživatele?)
- Vstup: ... (Co uživatel zadá nebo nahraje? Jaký formát? Jaká omezení --
  např. max velikost souboru, povolené formáty?)
- Výstup: ... (Co uživatel uvidí po dokončení? Jaký formát výsledku?)
- Kde v aplikaci se používá: ... (Na které stránce / obrazovce? Po jaké akci se spustí?)
- Závislosti: ... (Potřebuje tato funkce externí API? Databázi? Výstup jiné funkce?)
- Edge cases:
  - Co když je vstup prázdný: ...
  - Co když je vstup ve špatném formátu: ...
  - Co když externí služba neodpoví: ...
  - Co když už taková data existují (duplicita): ...

Funkce 2: ...
(stejná struktura)

Funkce 3: ...
(stejná struktura)
```

## 1) Extrakce popisu frontendu pomocí LLM

Z komplexního popisu projektu necháme LLM vytáhnout semi-technickou specifikaci frontendu, kterou pak použijeme v Lovable.

### Instrukce pro LLM (zkopírovat do ChatGPT / Claude / jiného LLM)

```
Mám komplexní popis webové aplikace (viz níže). Vytáhni z něj semi-technickou
specifikaci frontendu pro nástroj Lovable (AI website builder).

Specifikace by měla obsahovat:
- Seznam všech obrazovek / stránek a co na nich uživatel vidí
- Navigaci mezi stránkami
- Pro každou obrazovku: rozložení prvků (layout), formuláře, tlačítka,
  seznamy, karty, modální okna atd.
- Typický průchod uživatele (user flow) krok za krokem
- Vizuální styl (barvy, font, světlý/tmavý režim) -- pokud není specifikován,
  navrhni moderní a čistý design
- Responzivitu (mobilní vs. desktopová verze)
- Chybové stavy a prázdné stavy z pohledu UI
- Kde frontend volá backend API -- u těchto míst napiš "TODO: napojit na API"
  a použij mock data / placeholder

DŮLEŽITÉ:
- Frontend zatím NEBUDE obsahovat skutečnou backend logiku -- jen UI a mock data
- Specifikaci piš v češtině
- Výstup by měl být přímo použitelný jako prompt pro Lovable

Zde je kompletní popis projektu:
<ZDE VLOŽTE SVŮJ KOMPLEXNÍ POPIS PROJEKTU>
```

## 2) Vytvoření frontendu v Lovable

1. Otevřít [Lovable](https://lovable.dev) a vytvořit nový projekt
2. Vložit semi-technickou specifikaci frontendu jako úvodní prompt
3. Nechat Lovable vygenerovat první verzi UI
4. Iterovat přímo v Lovable pomocí dalších promptů -- opravit rozložení, doplnit chybějící stránky, upravit vizuální styl atd.
5. Ověřit, že všechny obrazovky a navigace fungují s mock daty

## 3) Propojení Lovable s GitHubem a klonování

1. V Lovable propojit projekt s GitHub repozitářem (Lovable → Settings → GitHub)
2. Lovable vytvoří repozitář s vygenerovaným kódem
3. Naklonovat repozitář lokálně a spustit Codex:

```bash
git clone git@github.com:<uzivatel>/<repozitar>.git
cd <repozitar>
codex
```

4. Přesunout obsah do složky `frontend/` a vytvořit složku `backend/` -- v Codexu zadat prompt:

```
Přesuň všechny soubory a složky (kromě .git) do nové složky frontend/.
Vytvoř prázdnou složku backend/. Commitni s popisnou commit message.
```

Ruční alternativa v terminálu:

```bash
mkdir frontend
git mv $(ls -A | grep -v -E '^(\.git|frontend)$') frontend/
mkdir backend
git add -A && git commit -m "Reorganize: move Lovable frontend into frontend/ folder, create backend/"
```

5. Ověřit, že frontend stále funguje -- v Codexu zadat prompt:

```
Nainstaluj závislosti ve složce frontend/ a spusť dev server.
Ověř, že aplikace běží bez chyb.
```

Ruční alternativa v terminálu:

```bash
cd frontend
npm install
npm run dev
```

## 4) Extrakce implementačních částí pro Codex

Z původního komplexního popisu projektu necháme LLM vytvořit oddělené části (chunky), které půjdou postupně kopírovat do Codexu.

### Instrukce pro LLM (zkopírovat do ChatGPT / Claude / jiného LLM)

```
Mám komplexní popis webové aplikace (viz níže). Rozděl ho na samostatné
implementační části (chunky) pro lokálního AI coding agenta (OpenAI Codex).

Každá část bude zkopírována do agenta jako samostatný prompt. Agent pracuje
v repozitáři se strukturou:
- frontend/  -- hotový frontend (React + Vite), vygenerovaný v Lovable
- backend/   -- zatím prázdná složka pro serverovou logiku

Požadavky na chunky:
1. První chunk = inicializace backendu (struktura projektu, package.json /
   requirements.txt, základní server)
2. Druhý chunk = vytvoření AGENTS.md souborů pro celý projekt:
   - Kořenový AGENTS.md -- přehled projektu, struktura repozitáře, konvence,
     návod jak spouštět a testovat, bezpečnostní pravidla (nikdy necommitovat
     secrets, .env soubory přidávat do .gitignore, používat environment variables)
   - frontend/AGENTS.md -- tech stack frontendu, jak spustit dev server,
     struktura komponent, konvence pojmenování, kde jsou mock data
   - backend/AGENTS.md -- tech stack backendu, jak spustit server, struktura
     API endpointů, databázové schéma, jak přidávat nové endpointy
   - Každý AGENTS.md musí obsahovat instrukci: "Pokud přidáš novou funkci,
     endpoint nebo změníš strukturu projektu, aktualizuj příslušný AGENTS.md"
3. Další chunky = jednotlivé funkce / API endpointy, každý se dá implementovat
   a otestovat nezávisle
4. Jeden z chunků = propojení frontendu s backendem (nahrazení mock dat
   skutečnými API voláními)
5. Poslední chunk = závěrečné testování a dokončení

Pro každý chunk uveď:
- Název a stručný popis
- Co přesně má agent udělat (konkrétní instrukce)
- Jaké soubory bude vytvářet / upravovat
- Jak ověřit, že chunk je hotový (testovací kritéria)
- Závislosti na předchozích chunkách

Formát výstupu: očíslované chunky, každý jako hotový prompt ke zkopírování
do agenta. Každý prompt konči instrukcí "Commitni a pushni změny."

Zde je kompletní popis projektu:
<ZDE VLOŽTE SVŮJ KOMPLEXNÍ POPIS PROJEKTU>
```

## 5) Iterativní implementace v Codexu

Postup pro každý chunk:

1. Zkopírovat chunk do Codexu jako prompt
2. Nechat agenta implementovat
3. Otestovat výsledek (spustit server, ověřit endpoint, zkontrolovat kód)
4. Pokud něco nefunguje -- opravit promptováním přímo v Codexu
5. Pokračovat dalším chunkem

```bash
# spustit codex ve složce projektu
cd <repozitar>
codex
```

Užitečné příkazy v Codexu viz [přehled z cvičení 1](cviceni1.md#10-přehled-užitečných-příkazů-v-codexu).

## 6) Průběžné pushování na GitHub

Po dokončení každého chunku (nebo i častěji) pushujte změny. Můžete využít přímo Codex s promptem:

```
Commitni všechny změny s popisným commit message a pushni na GitHub.
```

Nebo ručně:

```bash
git add -A
git commit -m "popis změny"
git push
```

## 7) Pokračování práce doma

Rozpracujte projekt dále dle libosti s využitím postupů ze cvičení.
