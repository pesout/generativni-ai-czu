# III. Prompt engineering, context engineering

## Výukový materiál od Alžběty Pokorné

- [github.com/Alzpeta/prompt_engineering](https://github.com/Alzpeta/prompt_engineering)

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
