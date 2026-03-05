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

## Výběr tématu a komplexní popis projektu

Studenti si vyberou svůj projekt (viz [příklady projektů z cvičení 1](cviceni1.md)). Cílem je vytvořit podrobný popis celé aplikace, který bude později sloužit jako vstup pro všechny další kroky:

- extrakce popisu frontendu pro [Lovable](https://lovable.dev)
- extrakce implementačních částí pro lokálního AI agenta Codex

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
- Backend framework: ... (např. Node.js/Express, Python/Flask, Python/FastAPI)
- Databáze: ... (např. SQLite pro jednoduchost, PostgreSQL, žádná pokud stačí soubory)
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

## Přehled celého vývojového flow

Kompletní popis projektu bude v dalších krocích (viz [cvičení 3](cviceni3.md)) sloužit jako vstup pro:

1. **Extrakci popisu frontendu** -- pomocí LLM vytáhneme z popisu semi-technickou specifikaci uživatelského rozhraní
2. **Vytvoření frontendu v Lovable** -- specifikaci použijeme v [Lovable](https://lovable.dev) pro vygenerování funkčního UI a iterujeme přímo v Lovable
3. **Propojení s GitHubem** -- výsledný projekt z Lovable propojíme s GitHub repozitářem a přesuneme lokálně do složky `frontend/`
4. **Extrakce implementačních částí** -- z původního popisu pomocí LLM vytvoříme oddělené části pro lokálního AI agenta (Codex)
5. **Iterativní implementace v Codexu** -- každou část zkopírujeme do Codexu: zkopírovat → nechat implementovat → otestovat → opravit promptováním → pokračovat další částí
6. **Průběžné pushování na GitHub** -- pravidelně pushujeme změny


