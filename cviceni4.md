# IV. Integrace LLM API a chatbot

## Kontrolní seznam z minulých cvičení

Než začnete, ověřte si stav svého projektu z minulých cvičení:

1. Projekt existuje v Lovable
2. Projekt je propojený s GitHub repozitářem
3. Repozitář je naklonovaný lokálně
4. Struktura repozitáře obsahuje složky `frontend/` a `backend/`
5. Frontend se spustí lokálně (`cd frontend && npm install && npm run dev`)
6. Backend se spustí lokálně (pokud už je implementovaný)
7. Databáze je nakonfigurovaná a dostupná (pokud ji projekt používá)
8. `AGENTS.md` existuje v kořeni repozitáře
9. `AGENTS.md` existuje ve složce `frontend/`
10. `AGENTS.md` existuje ve složce `backend/`

> Pokud některý bod chybí, vraťte se ke [cvičení 2](cviceni2.md) a doplňte ho. Zejména `AGENTS.md` soubory jsou důležité -- AI agent (Codex) z nich čerpá kontext o projektu.

## 1) OpenAI API klíč

Pro práci s LLM API budete potřebovat OpenAI API klíč:

1. Přihlaste se na [platform.openai.com](https://platform.openai.com)
2. Přejděte do sekce **API keys**
3. Vytvořte nový klíč (tlačítko **Create new secret key**)
4. Klíč si zkopírujte -- zobrazí se pouze jednou

Klíč přidejte do `.env` souboru v `backend/` složce:

```
OPENAI_API_KEY=sk-...váš-klíč...
```

> **Bezpečnostní pravidlo:** Soubor `.env` nesmí být nikdy commitnutý do Gitu. Ověřte, že `backend/.gitignore` obsahuje `.env`. Pokud ne, přidejte ho tam.

## 2) LLM integrace s abstrakční vrstvou

Cílem tohoto kroku je vytvořit v backendu abstrakční vrstvu pro komunikaci s LLM. Díky ní půjde v budoucnu snadno vyměnit poskytovatele (např. OpenAI → Anthropic → lokální model) bez změny zbytku aplikace. Pokud váš projekt už LLM volání obsahuje, budou zároveň refaktorována tak, aby tuto vrstvu využívala.

Spusťte Codex v kořenové složce projektu:

```bash
cd <repozitar>
codex
```

### Prompt pro Codex

```
Vytvoř v backendu abstrakční vrstvu (LLM service / adapter) pro komunikaci
s LLM. Jako výchozího poskytovatele použij OpenAI API.

Požadavky na abstrakční vrstvu:
- Zbytek aplikace volá pouze LLM service -- nikdy přímo OpenAI SDK
- API klíč, název modelu, temperature a max_tokens se načítají z environment
  variables (.env souboru v backendu)
- Pokud bych chtěl v budoucnu vyměnit OpenAI za jiného poskytovatele, stačí
  změnit pouze implementaci tohoto service
- Přidej ošetření chyb (error handling) pro případ, že API neodpoví nebo
  vrátí chybu

Dále projdi celý projekt a zkontroluj, zda už existuje kód, který volá
OpenAI API nebo jakékoli jiné LLM API:
- Pokud ano: vypiš, kde se volání nacházejí, zkontroluj, že API klíč není
  hardcoded, a refaktoruj volání tak, aby procházela přes novou abstrakční
  vrstvu. Zachovej veškerou existující funkcionalitu.
- Pokud ne: zatím nic nepřidávej, abstrakční vrstva bude využita v později
  pro jiné implementace.

Aktualizuj AGENTS.md soubory, aby popisovaly architekturu LLM vrstvy.
Commitni změny s popisnou commit message.
```

### Cílová architektura

Po dokončení by architektura měla vypadat přibližně takto:

```
Controller / Route
    ↓
Business Service (např. RecipeService, AnalysisService)
    ↓
LLM Service (abstrakce -- definuje rozhraní)
    ↓
OpenAI Provider (konkrétní implementace)
```

Klíčové je, že business logika nezná detaily o tom, jaký LLM provider se používá.

### Volitelné: přidání dalších LLM funkcí

Pokud váš projekt má use-case pro LLM (např. generování textů, analýzu vstupů, doporučování), ale zatím není implementovaný, použijte následující prompt. Upravte ho podle svého konkrétního projektu:

```
Přidej do aplikace LLM funkci pro [POPIŠTE SVŮJ USE-CASE,
např. "generování popisků k produktům", "analýzu sentimentu recenzí",
"doporučování receptů na základě ingrediencí"].

Použij existující LLM service / abstrakční vrstvu. Přidej potřebné API
endpointy na backendu a ve frontendu nahraď mock data / placeholdery
skutečnými voláními nových endpointů. Aktualizuj AGENTS.md.

Commitni změny.
```

## 3) Konfigurace chatbotu -- soubor s instrukcemi

Než začnete implementovat chatbot, připravte si konfigurační soubor, který definuje, jak se má chatbot chovat. Vytvořte soubor `backend/chatbot-instructions.md`.

Obsah souboru upravte podle svého projektu. Nebojte se napsat jen základ a nechat Codex, aby intrukce upravil v kontextu vašeho projektu. Zde je šablona:

```markdown
# Chatbot instrukce

## Identita
Jsi pomocný asistent aplikace [NÁZEV VAŠÍ APLIKACE].
Tvé jméno je [JMÉNO ASISTENTA, např. "Asistent", "Pepa", "AI Pomocník"].

## Hlavní úkol
Pomáháš uživatelům s používáním aplikace -- odpovídáš na dotazy o funkcích,
naváděíš je krok za krokem a pomáháš řešit problémy.

## Jazyk a tón komunikace
- Komunikuj v češtině (pokud uživatel nepíše v jiném jazyce -- pak odpověz
  v jeho jazyce)
- Piš přátelsky, ale profesionálně
- Používej stručné a jasné odpovědi -- ne zbytečně dlouhé texty
- Pokud je potřeba delší vysvětlení, strukturuj ho do očíslovaných kroků
  nebo odrážek

## Pravidla chování
- Odpovídej pouze na témata související s aplikací a její doménou
- Pokud se uživatel ptá na něco mimo scope aplikace, zdvořile ho přesměruj:
  "Na toto bohužel nejsem schopen odpovědět. Mohu vám pomoci s [konkrétní
  funkce aplikace]."
- Pokud si nejsi jistý odpovědí, řekni to: "Tím si nejsem zcela jistý.
  Doporučuji [alternativní zdroj / akce]."
- Nikdy si nevymýšlej informace, které nemáš k dispozici
- Nikdy neprozrazuj technické detaily implementace (API klíče, názvy
  databázových tabulek, interní endpointy apod.)
- Nikdy neposkytuj rady, které by mohly vést k poškození dat uživatele

## Znalosti o aplikaci

### Hlavní funkce
- [Popište funkci 1 a jak ji uživatel používá]
- [Popište funkci 2 a jak ji uživatel používá]
- [Popište funkci 3 a jak ji uživatel používá]

### Omezení aplikace
- [Popište, co aplikace neumí nebo nepodporuje]
- [Popište známé limity, např. maximální velikost souboru]

### Časté problémy a řešení
- [Problém 1]: [Řešení]
- [Problém 2]: [Řešení]

## Příklady interakcí

Uživatel: "Jak přidám nový záznam?"
Asistent: "[Vaše odpověď popisující postup krok za krokem]"

Uživatel: "Nefunguje mi přihlášení."
Asistent: "[Vaše odpověď s konkrétními kroky pro řešení]"

Uživatel: "Můžeš mi napsat esej o historii Prahy?"
Asistent: "Na toto bohužel nejsem schopen odpovědět -- jsem asistent
aplikace [NÁZEV]. Mohu vám pomoci například s [funkce 1] nebo [funkce 2]."

## Formátování odpovědí
- Pro postupy používej očíslované kroky (1., 2., 3.)
- Pro výčty používej odrážky
```

> Tento soubor se načítá při startu backendu jako system prompt pro chatbot. Když chcete změnit chování chatbotu, stačí upravit tento soubor a restartovat backend.

Dále přidejte do `backend/.env` konfiguraci pro chatbot:

```
OPENAI_API_KEY=sk-...váš-klíč...
LLM_MODEL=gpt-4o-mini
LLM_TEMPERATURE=0.7
LLM_MAX_TOKENS=1024
```

### Konfigurace teploty (temperature)

- **0.0** -- deterministický výstup, vždy nejpravděpodobnější token. Ideální pro fakta, kód, klasifikaci.
- **0.1 až 0.3** -- minimální variabilita, stále velmi konzistentní. Dobrý kompromis pro většinu produkčních úloh (sumarizace, extrakce, překlad).
- **0.4 až 0.7** -- vyvážený režim. Více kreativity, stále koherentní. Konverzační chatboti, copywriting.
- **0.8 až 1.0** -- výrazně kreativnější, méně předvídatelný. Brainstorming, generování příběhů, poezie.
- **více než 1.0** -- roste pravděpodobnost i málo pravděpodobných tokenů. Často nesouvislý, halucinující výstup. Zřídka užitečné.

## 4) Implementace chatbotu -- backend

### Prompt pro Codex

```
Implementuj backend část chatbotu pro aplikaci. Vyhledej aktuální
dokumentaci použitých knihoven (OpenAI SDK, framework) a OpenAI API.

Požadavky:
1. Vytvoř API endpoint pro chat -- navrhni vhodnou strukturu requestu
   a response podle best practices (endpoint přijímá zprávu od uživatele
   a konverzační historii, vrací odpověď chatbotu)

2. System prompt se načítá ze souboru backend/chatbot-instructions.md
   při startu aplikace (ne při každém requestu)

3. Konfigurace modelu (název modelu, temperature, max_tokens) se načítá
   z environment variables (.env)

4. Použij existující LLM service / abstrakční vrstvu -- pokud ještě
   neexistuje, vytvoř ji (zbytek aplikace nesmí volat OpenAI SDK přímo)

5. Konverzační historie se posílá z frontendu -- backend ji neukládá,
   pouze předává do LLM spolu se system promptem

6. Přidej validaci vstupu a ošetření chyb s vhodnými HTTP status kódy
   a uživatelsky čitelnými chybovými hláškami

7. Aktualizuj AGENTS.md -- přidej dokumentaci chatbot endpointu

8. Otestuj implementaci pomocí requestu na API endpoint pro chat.

Commitni změny.
```

## 5) Implementace chatbotu -- frontend

### Prompt pro Codex

```
Implementuj frontend část chatbotu jako chat widget.

Požadavky:
1. Vytvoř chatovací komponentu (chat widget), která se zobrazí jako plovoucí
   okno v pravém dolním rohu aplikace na všech stránkách

2. Widget má dva stavy:
   - Zavřený: zobrazí se jen ikona chatu (bublina / ikona zprávy)
   - Otevřený: zobrazí se chatovací okno s:
     - Hlavičkou s názvem (např. "Asistent") a tlačítkem pro zavření
     - Oblastí pro zprávy (scrollovatelná)
     - Vstupním polem a tlačítkem pro odeslání

3. Zprávy se zobrazují jako bubliny -- uživatelské zprávy vpravo,
   odpovědi asistenta vlevo, vizuálně odlišené barvami

4. Po odeslání zprávy:
   - Zpráva se okamžitě zobrazí v chatu
   - Zobrazí se indikátor načítání (např. "Asistent píše...")
   - Odešle se POST request na backend endpoint s textem zprávy a celou
     konverzační historií
   - Po přijetí odpovědi se zobrazí odpověď asistenta a zruší se indikátor

5. Konverzační historie se drží v paměti komponenty (React state) --
   neukládá se do databáze ani local storage

6. Ošetření chyb: pokud API vrátí chybu, zobraz uživatelsky přívětivou
   hlášku přímo v chatu (např. "Omlouvám se, něco se pokazilo.
   Zkuste to prosím znovu.")

7. Widget by měl být responzivní -- na mobilních zařízeních zabere
   celou šířku obrazovky

8. Použij existující design systém projektu (barvy, fonty, komponenty)

Commitni změny.
```

## 6) Testování a ladění

Po implementaci backendu i frontendu ověřte, že vše funguje:

Spusťte backend:

```bash
cd backend
# podle tech stacku: npm run start:dev / npm run dev / python app.py
```

Spusťte frontend:

```bash
cd frontend
npm run dev
```

Otevřete aplikaci v prohlížeři a otestujte chatbot:
  - Klikněte na ikonu chatu
  - Napište testovací zprávu
  - Ověřte, že přijde odpověď
  - Napište navazující zprávu -- ověřte, že chatbot si pamatuje kontext konverzace
  - Zkuste odeslat prázdnou zprávu -- nemělo by být povoleno

Dále můžete experimentovat se změnami konfigurace v souboru `backend/chatbot-instructions.md` a s různým nastavením v `.env` (změna teploty nebo modelu).

### Prompt pro opravu problémů (pokud něco nefunguje)

```
Chatbot nefunguje správně. Problém: [POPIŠTE KONKRÉTNÍ PROBLÉM,
např. "frontend posílá request, ale dostává 500 error",
"chatbot neodpovídá v češtině", "konverzační historie se neodesílá"].

Oprav problém a commitni.
```

## 7) Volitelné: rozšíření a individuální úpravy

Následující rozšíření jsou volitelná. Vyberte si ta, která dávají smysl pro váš projekt, a upravte prompty podle potřeby.

### A) Chatbot s přístupem k datům aplikace

Pokud chcete, aby chatbot mohl odpovídat na dotazy o datech uživatele (např. "Kolik jsem utratil tento měsíc?", "Jaké recepty mám uložené?"):

```
Rozšiř chatbot endpoint tak, aby měl přístup k datům aktuálně
přihlášeného uživatele.

Před odesláním dotazu do LLM:
1. Načti relevantní data uživatele z databáze
2. Přidej je jako součást kontextu (system promptu nebo user message)
   ve strukturovaném formátu
3. LLM pak může odpovídat na dotazy o konkrétních datech uživatele

Dej pozor na velikost kontextu -- neposílej celou databázi, jen relevantní
data. Aktualizuj chatbot-instructions.md s informací o tom, jaká data
má chatbot k dispozici.

Commitni změny.
```

### B) Více LLM funkcí v aplikaci

Pokud chcete přidat další LLM-poháněné funkce mimo chatbot:

```
Přidej do aplikace funkci [POPIŠTE FUNKCI, např. "automatické generování
shrnutí", "kategorizace vstupů", "překlad obsahu"].

Použij existující LLM service / abstrakční vrstvu. Přidej nový endpoint
na backendu a propoj ho s frontendovým UI. Aktualizuj AGENTS.md.

Commitni změny.
```

### C) Výměna LLM poskytovatele

Pro ověření, že abstrakční vrstva funguje správně, zkuste vyměnit poskytovatele:

```
Vytvoř alternativní implementaci LLM service pro [Anthropic Claude API /
lokální Ollama / jiný provider]. Obě implementace (OpenAI i nová) by měly
být v kódu a přepínání mezi nimi by mělo být řízené environment variable,
např. LLM_PROVIDER=openai nebo LLM_PROVIDER=anthropic.

Commitni změny.
```

## 8) Průběžné pushování na GitHub

Po dokončení každého kroku pushujte změny:

```
Commitni všechny změny s popisným commit message a pushni na GitHub.
```

Nebo ručně:

```bash
git add -A
git commit -m "popis změny"
git push
```
