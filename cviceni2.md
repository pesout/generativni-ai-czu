# II. Prompt engineering, context engineering, architektura projektu

## Porovnávací aplikace se dvěma okny

- [http://aibr.pesout.net/dual-prompt.php](http://aibr.pesout.net/dual-prompt.php)

## Ukázkové páry promptů -- srovnání formulací

### 1) Vágní vs. konkrétní

**Prompt A:**

```
Řekni mi něco o fotosyntéze.
```

**Prompt B:**

```
Vysvětli proces fotosyntézy pro studenta střední školy. Popiš vstupní látky,
výstupní produkty a kde v rostlině probíhá. Odpověz v 5 bodech.
```

### 2) Bez role vs. s rolí

**Prompt A:**

```
Jak funguje inflace?
```

**Prompt B:**

```
Jsi zkušený učitel ekonomie na gymnáziu. Vysvětli studentům, jak funguje inflace,
použij jednoduchý příklad z běžného života (např. cena chleba) a uveď hlavní
příčiny inflace.
```

### 3) Bez formátu vs. s požadovaným formátem

**Prompt A:**

```
Jaké jsou výhody a nevýhody práce z domova?
```

**Prompt B:**

```
Vytvoř přehlednou tabulku ve formátu markdown se dvěma sloupci: "Výhody"
a "Nevýhody" práce z domova. Uveď alespoň 5 položek v každém sloupci.
```

### 4) Jednoduchý dotaz vs. krok za krokem

**Prompt A:**

```
Kolik je 47 × 83? Napiš výsledek.
```

**Prompt B:**

```
Kolik je 47 × 83? Ukaž celý postup výpočtu krok za krokem a na konci napiš
finální výsledek.
```

### 5) Bez příkladu vs. s příkladem

**Prompt A:**

```
Z následujících popisů vypiš klíčové informace:

Nyní zpracuj:
1. Jan Novák, 32 let, pracuje jako programátor v Praze.
2. Marie Dvořáková, 45 let, učitelka z Brna.
3. Petr Horák, 28 let, designér, žije v Ostravě.
```

**Prompt B:**

```
Z následujících popisů vypiš klíčové informace:

Příklad:
Vstup: "Eva Malá, 38 let, pracuje jako lékařka v Plzni."
Výstup: "Malá, E. | 38 | lékařka | Plzeň"

Nyní zpracuj:
1. Jan Novák, 32 let, pracuje jako programátor v Praze.
2. Marie Dvořáková, 45 let, učitelka z Brna.
3. Petr Horák, 28 let, designér, žije v Ostravě.
```

## Context engineering

Analýza/výpočty nad dvěma CSV soubory pomocí nástroje codex přímo v CLI.

Soubory:

- [sales_clean.csv](data/sales_clean.csv)
- [sales_messy.csv](data/sales_messy.csv)

## Implementační plán projektu

Studenti si vyberou svůj projekt (viz [příklady projektů z cvičení 1](cviceni1.md)) a založí GitHub repozitář (viz [postup z cvičení 1](cviceni1.md#1-registrace-na-github--testovací-repozitář)).

Cílem je vyplnit šablonu implementačního plánu a poté ji po částech kopírovat do AI agenta. Každá část = jeden prompt pro agenta. Postup: zkopírovat → nechat implementovat → otestovat → opravit → pokračovat další částí.

### Šablona -- části ke kopírování

Zde je potřeba vyplnit `...` svými odpověďmi a smazat nerelevantní sekce. V případě nejistoty je možné se na jednotlivé koncepty, technické pojmy nebo best practices doptat cvičících nebo nějakého LLM.

Struktura je pouze doporučená a není pevná.

---

**Část 1 -- Popis projektu a AGENTS.md** (zkopírovat do agenta jako první prompt)

```
Vytvoř soubor AGENTS.md s následujícím obsahem a zároveň vytvoř README.md
s názvem projektu a krátkým popisem. Commitni a pushni.

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

TECH STACK:
- Jazyk / framework: ... (např. Next.js, Nest.js, Python/Flask, SvelteKit)
- Databáze: ... (např. SQLite pro jednoduchost, PostgreSQL pro produkci, žádná pokud stačí soubory/localStorage)
- Další knihovny: ... (např. Tailwind CSS, Prisma, OpenAI SDK, Tesseract.js)
- Runtime: ... (např. Node.js, Python 3.12, Bun)

KONVENCE PRO VÝVOJ:
- Po změnách vše zapisuj do AGENTS.md, aby to vždy odpovídalo aktuálnímu stavu projektu
- Test-first přístup -- nejdřív test, pak implementace.
- Každou logickou změnu commitni zvlášť s popisným commit message.
- Kód piš čistě, s konzistentním stylem a bez zbytečných komentářů.
- Pokud si nejsi jistý, zeptej se -- neimplementuj na základě domněnek.
- Všechny texty v UI budou v: ... (čeština / angličtina)
```

---

**Část 2 -- Inicializace projektu** (zkopírovat po dokončení části 1)

```
Podle AGENTS.md inicializuj projekt:

- Vytvoř adresářovou strukturu projektu.
- Přidej konfigurační soubory (.gitignore, linter, formatter, package.json
  nebo requirements.txt).
- Nastav dev server tak, aby šel spustit jedním příkazem.
- Přidej do README.md sekci "Spuštění" s příkazy pro instalaci závislostí
  a start dev serveru.
- Pokud projekt používá databázi, připrav schéma / migrace.
- Commitni a pushni.
```

---

**Část 3 -- Uživatelské rozhraní a datový model** (vyplnit a zkopírovat)

```
OBRAZOVKY / STRÁNKY APLIKACE:
1. ... (např. "Hlavní stránka -- přehled výdajů za aktuální měsíc")
2. ... (např. "Detail účtenky -- seznam položek z jedné účtenky")
3. ... (např. "Nahrání účtenky -- formulář pro upload fotky")
4. ...
5. ...

NAVIGACE MEZI STRÁNKAMI:
... (např. "Spodní navigační lišta s ikonami: Přehled, Nahrát, Historie")

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

Na základě výše uvedeného:
1. Vytvoř datový model (entity, atributy, vztahy).
2. Vytvoř základní kostru komponent / stránek / rout -- zatím bez logiky.
3. Napiš testy pro datový model.
4. Commitni a pushni.
```

---

**Část 4 -- Implementace funkcí** (kopírovat zvlášť pro každou funkci)

Pro každou funkci z MVP seznamu vyplnit a zkopírovat jako samostatný prompt:

```
Implementuj funkci: ... (např. "Nahrání a zpracování fotky účtenky")

POPIS:
... (Co přesně tato funkce dělá z pohledu uživatele?)

VSTUP:
... (Co uživatel zadá nebo nahraje? Jaký formát? Jaká omezení --
např. max velikost souboru, povolené formáty?)

VÝSTUP:
... (Co uživatel uvidí po dokončení? Jaký formát výsledku?)

KDE V APLIKACI SE POUŽÍVÁ:
... (Na které stránce / obrazovce? Po jaké akci se spustí?)

ZÁVISLOSTI:
... (Potřebuje tato funkce externí API? Databázi? Výstup jiné funkce?)

EDGE CASES:
- Co když je vstup prázdný: ...
- Co když je vstup ve špatném formátu: ...
- Co když externí služba neodpoví: ...
- Co když už taková data existují (duplicita): ...

POSTUP:
1. Napiš testy pro tuto funkci (happy path + edge cases).
2. Implementuj backend logiku / API endpoint.
3. Implementuj frontend komponentu.
4. Ověř, že testy prochází.
5. Commitni a pushni.
```

---

**Část 5 -- Testování a dokončení** (zkopírovat na závěr)

```
Závěrečné testování a dokončení projektu:

1. Spusť všechny testy a oprav případné chyby.
2. Projdi kompletní uživatelský flow od začátku do konce:
   - Registrace / přihlášení (pokud existuje)
   - Hlavní funkce aplikace
   - Zobrazení výsledků
   - Chybové stavy
3. Ověř, že dev server jde spustit jedním příkazem po čistém git clone.
4. Zkontroluj, že README.md obsahuje:
   - Popis projektu
   - Návod k instalaci a spuštění
   - Použité technologie
5. Finální commit a push.
```

