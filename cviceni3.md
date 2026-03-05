# III. Vibe-coding: frontend v Lovable, pokračování v Codexu

> Předpoklad: máte hotový komplexní popis projektu z [cvičení 2](cviceni2.md).

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
Vytvoř prázdnou složku backend/. Commitni s popisným commit message.
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
   requirements.txt, základní server, AGENTS.md)
2. Další chunky = jednotlivé funkce / API endpointy, každý se dá implementovat
   a otestovat nezávisle
3. Jeden z chunků = propojení frontendu s backendem (nahrazení mock dat
   skutečnými API voláními)
4. Poslední chunk = závěrečné testování a dokončení

Pro každý chunk uveď:
- Název a stručný popis
- Co přesně má agent udělat (konkrétní instrukce)
- Jaké soubory bude vytvářet / upravovat
- Jak ověřit, že chunk je hotový (testovací kritéria)
- Závislosti na předchozích chunkách

Formát výstupu: očíslované chunky, každý jako hotový prompt ke zkopírování
do agenta. Každý prompt konči instrukcí "Commitni a pushni změny."

DŮLEŽITÉ:
- Piš v češtině
- Každý chunk musí být soběstačný -- agent nemá kontext z předchozích chunků
- Odkazuj na existující strukturu frontendu kde je to relevantní

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
