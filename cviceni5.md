# V. Praktická práce s LLM API

## Teorie -- jak funguje volání LLM API

Když aplikace komunikuje s velkým jazykovým modelem (LLM), probíhá to přes HTTP API. Klient (váš backend) odešle HTTP POST request s JSON tělem na endpoint poskytovatele (např. OpenAI) a dostane zpět JSON odpověď s vygenerovaným textem.

### Základní princip

```
Vaše aplikace  ──HTTP POST + JSON──►  API endpoint poskytovatele
                                              │
                                      zpracování modelem
                                              │
Vaše aplikace  ◄──JSON odpověď──────  vygenerovaný výstup
```

Každý request typicky obsahuje:

- **Model** -- který model má odpovědět (např. `gpt-4o-mini`, `gpt-4o`)
- **Vstupní zprávy / prompt** -- text, na který má model reagovat
- **Parametry generování** -- temperature, max_tokens apod.

### Autentizace

API používá **Bearer token** -- API klíč se posílá v HTTP hlavičce:

```
Authorization: Bearer sk-...váš-klíč...
```

Klíč identifikuje váš účet a určuje, ke kterým modelům máte přístup. Nikdy ho nevkládejte do kódu -- používejte environment variables (`.env`).

### Temperature a další parametry

| Parametr | Co ovlivňuje |
|---|---|
| `temperature` | Kreativita odpovědi |
| `max_tokens` / `max_output_tokens` | Maximální délka odpovědi v tokenech |
| `stream` | Streamování odpovědi po částech (token po tokenu) |

### Chat Completions vs. Responses API

OpenAI nabízí dva hlavní endpointy pro generování textu:

- **Chat Completions** (`/v1/chat/completions`) -- starší, široce podporovaný endpoint. Vstupem je pole zpráv s rolemi (`system`, `user`, `assistant`).
- **Responses** (`/v1/responses`) -- novější endpoint s podporou built-in nástrojů (web search, code interpreter aj.). Doporučený pro nové projekty.

### Další modality

Kromě generování textu nabízí OpenAI API endpointy pro práci s dalšími modalitami:

- **Speech-to-Text** -- přepis audia do textu (modely Whisper, GPT-4o Transcribe)
- **Text-to-Speech** -- generování řeči z textu
- **Images** -- generování a editace obrázků z textového popisu
- **Embeddings** -- vektorové reprezentace textu pro vyhledávání a porovnávání

## Praktická část -- OpenAI API pískoviště

Pro praktické zkoušení API endpointů použijeme interaktivní Swagger UI rozhraní: [openai-api-swagger.html](data/openai-api-swagger.html).

Soubor si stáhněte a otevřete v prohlížeči. Obsahuje předpřipravené příklady pro všechny klíčové endpointy OpenAI API.

### Příprava -- autorizace

1. Klikněte na tlačítko **Authorize** v horní části stránky
2. Zadejte svůj OpenAI API klíč (formát `sk-...`)
3. Potvrďte -- klíč se uloží pro všechny následující requesty

### 1) Chat Completions -- generování textu

**Endpoint:** `POST /v1/chat/completions`

Klasický endpoint pro generování textu. Vstupem je pole zpráv s rolemi:

- `system` -- instrukce pro model (jak se má chovat)
- `user` -- zpráva od uživatele
- `assistant` -- předchozí odpověď modelu (pro konverzaci)

#### Příklady k vyzkoušení

**Jednoduchý dotaz** -- základní otázka s výchozím nastavením:

```json
{
  "model": "gpt-4o-mini",
  "messages": [
    { "role": "system", "content": "Jsi užitečný asistent. Odpovídej česky." },
    { "role": "user", "content": "Co je to REST API? Vysvětli ve 2 větách." }
  ]
}
```

**Nízká teplota** -- deterministická, faktická odpověď:

```json
{
  "model": "gpt-4o-mini",
  "messages": [
    { "role": "system", "content": "Jsi přesný technický asistent." },
    { "role": "user", "content": "Jaký je rozdíl mezi HTTP metodami GET a POST?" }
  ],
  "temperature": 0.2,
  "max_tokens": 300
}
```

**Vysoká teplota** -- kreativní generování:

```json
{
  "model": "gpt-4o",
  "messages": [
    { "role": "system", "content": "Jsi kreativní spisovatel. Piš barvitě a originálně." },
    { "role": "user", "content": "Napiš krátký příběh o robotovi, který se naučil vařit." }
  ],
  "temperature": 1.2,
  "max_tokens": 500
}
```

**Vision** -- analýza obrázku (multimodální vstup):

```json
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "user",
      "content": [
        { "type": "text", "text": "Co vidíš na tomto obrázku? Popiš česky." },
        { "type": "image_url", "image_url": { "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/Van_Gogh_-_Starry_Night_-_Google_Art_Project.jpg/300px-Van_Gogh_-_Starry_Night_-_Google_Art_Project.jpg" } }
      ]
    }
  ],
  "max_tokens": 300
}
```

**Vícekroková konverzace** -- simulace chatu s historií:

```json
{
  "model": "gpt-4o-mini",
  "messages": [
    { "role": "system", "content": "Jsi programátorský mentor. Odpovídej stručně česky." },
    { "role": "user", "content": "Co je to API?" },
    { "role": "assistant", "content": "API (Application Programming Interface) je rozhraní, přes které spolu komunikují dva programy." },
    { "role": "user", "content": "Dej mi konkrétní příklad." }
  ],
  "temperature": 0.7
}
```

#### Co zkoušet měnit

- **`model`** -- porovnejte odpovědi `gpt-4o-mini` (rychlejší, levnější) vs. `gpt-4o` (kvalitnější) vs. `gpt-4.1` / `gpt-5.2` / `gpt-5.4`
- **`temperature`** -- pošlete stejný dotaz s hodnotami 0, 0.5, 1.0 a 1.5 a porovnejte výstupy
- **`max_tokens`** -- zkuste nízkou hodnotu (např. 50) a sledujte, jak model zkrátí odpověď
- **`system` zpráva** -- změňte roli modelu (učitel, básník, programátor) a pozorujte vliv na styl odpovědi
- **Obrázky** -- v příkladu Vision zkuste zaměnit URL za jiný obrázek

### 2) Responses -- novější agentní endpoint

**Endpoint:** `POST /v1/responses`

Novější endpoint doporučený pro nové projekty. Hlavní rozdíly oproti Chat Completions:

- Vstup může být jednoduchý string (nemusíte vytvářet pole zpráv)
- Systémové instrukce se zadávají přes pole `instructions` (ne jako zpráva s rolí `system`)
- Podpora built-in nástrojů: `web_search`, `file_search`, `code_interpreter`, `computer_use`, `mcp`

#### Příklady k vyzkoušení

**Jednoduchý dotaz** -- stačí zadat string:

```json
{
  "model": "gpt-4o-mini",
  "input": "Řekni mi vtip o programátorech."
}
```

**S web search** -- model může vyhledat aktuální informace na internetu:

```json
{
  "model": "gpt-4o-mini",
  "input": "Jaký je aktuální kurz CZK/EUR?",
  "tools": [{ "type": "web_search" }]
}
```

**S instrukcemi a nízkou teplotou** -- přesný překlad:

```json
{
  "model": "gpt-4o-mini",
  "input": "Přelož do angličtiny: Dnes je krásný den.",
  "instructions": "Jsi profesionální překladatel. Odpovídej pouze překladem, bez vysvětlení.",
  "temperature": 0.1,
  "max_output_tokens": 100
}
```

**Kreativní generování** -- vysoká teplota:

```json
{
  "model": "gpt-4o",
  "input": "Vymysli 3 originální názvy pro mobilní aplikaci na sdílení receptů.",
  "instructions": "Buď kreativní a hravý. Názvy by měly být chytlavé a zapamatovatelné.",
  "temperature": 1.6,
  "max_output_tokens": 200
}
```

**Obrázek (vision)** -- multimodální vstup v Responses API:

```json
{
  "model": "gpt-4o",
  "input": [
    {
      "role": "user",
      "content": [
        { "type": "input_text", "text": "Co je na tomto obrázku? Popiš česky." },
        { "type": "input_image", "image_url": "https://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/Van_Gogh_-_Starry_Night_-_Google_Art_Project.jpg/300px-Van_Gogh_-_Starry_Night_-_Google_Art_Project.jpg" }
      ]
    }
  ]
}
```

**Vícekroková konverzace:**

```json
{
  "model": "gpt-4o-mini",
  "instructions": "Jsi učitel programování. Vysvětluj jednoduše česky.",
  "input": [
    { "role": "user", "content": "Co je to proměnná?" },
    { "role": "assistant", "content": "Proměnná je pojmenované místo v paměti, kam si program ukládá hodnoty." },
    { "role": "user", "content": "A jaký je rozdíl mezi let a const v JavaScriptu?" }
  ],
  "temperature": 0.5
}
```

#### Co zkoušet měnit

- **`tools`** -- přidejte `[{ "type": "web_search" }]` k libovolnému dotazu a sledujte, jak model využije aktuální informace z internetu
- **`instructions`** -- porovnejte odpovědi se a bez systémových instrukcí
- **Formát vstupu** -- porovnejte jednoduchý string vs. pole zpráv pro stejný dotaz

### 3) Speech-to-Text -- transkripce audia

**Endpoint:** `POST /v1/audio/transcriptions`

Přepíše audio soubor do textu. Nahrajte audio soubor (mp3, wav, m4a, webm aj.) a získejte přepis.

#### Parametry

| Parametr | Popis |
|---|---|
| `file` | Audio soubor (max 25 MB) |
| `model` | `whisper-1` (levnější), `gpt-4o-transcribe` (vyšší kvalita), `gpt-4o-transcribe-diarize` (identifikace mluvčích) |
| `language` | ISO kód jazyka, např. `cs` pro češtinu -- zvyšuje přesnost |
| `response_format` | `json`, `text`, `srt`, `vtt` (titulky s časovými značkami), `verbose_json` |
| `prompt` | Volitelný kontext -- pomáhá s rozpoznáním technických termínů (např. `"Kubernetes, NestJS, Docker"`) |
| `temperature` | 0 = nejpřesnější přepis |

#### Co zkoušet měnit

- **`model`** -- porovnejte přesnost `whisper-1` vs. `gpt-4o-transcribe`
- **`language`** -- zkuste přepis bez a s uvedením jazyka `cs`
- **`response_format`** -- vyzkoušejte `srt` nebo `vtt` pro titulky s časovými značkami
- **`prompt`** -- u technických nahrávek přidejte očekávané termíny

**Endpoint:** `POST /v1/audio/translations`

Přepíše audio v libovolném jazyce a přeloží do angličtiny (podporuje pouze `whisper-1`).

### 4) Text-to-Speech -- generování řeči

**Endpoint:** `POST /v1/audio/speech`

Převede text na mluvené audio. Po odeslání requestu se ve Swagger UI zobrazí audio přehrávač.

#### Parametry

| Parametr | Popis |
|---|---|
| `model` | `gpt-4o-mini-tts` (nejnovější, podporuje prompting stylu), `tts-1`, `tts-1-hd` |
| `input` | Text k přečtení |
| `voice` | Hlas -- `alloy`, `ash`, `ballad`, `coral`, `echo`, `fable`, `onyx`, `nova`, `sage`, `shimmer`, `verse`, `marin`, `cedar` |
| `instructions` | Instrukce pro styl řeči (jen `gpt-4o-mini-tts`) |
| `speed` | Rychlost řeči (0.25 -- 4.0, výchozí 1.0) |
| `response_format` | `mp3`, `opus`, `aac`, `flac`, `wav`, `pcm` |

#### Příklady k vyzkoušení

**Základní syntéza:**

```json
{
  "model": "gpt-4o-mini-tts",
  "input": "Ahoj, vítej na workshopu o OpenAI API. Dnes se naučíme pracovat s různými endpointy.",
  "voice": "nova"
}
```

**S instrukcemi pro styl řeči:**

```json
{
  "model": "gpt-4o-mini-tts",
  "input": "Dnes máme pro vás skvělou novinku. Naše aplikace nyní podporuje hlasové příkazy!",
  "voice": "marin",
  "instructions": "Mluv nadšeně a energicky, jako moderátor ranní show."
}
```

**Pomalé diktování:**

```json
{
  "model": "gpt-4o-mini-tts",
  "input": "kubectl apply minus f deployment dot yaml",
  "voice": "cedar",
  "instructions": "Mluv pomalu a zřetelně, diktuj příkaz slovo po slovu.",
  "speed": 0.75
}
```

#### Co zkoušet měnit

- **`voice`** -- vyzkoušejte různé hlasy na stejném textu (doporučené: `marin`, `cedar`, `nova`)
- **`instructions`** -- změňte styl řeči: "šeptej", "mluv jako profesor", "říkej to jako pohádku pro děti"
- **`speed`** -- porovnejte 0.5 (pomalé) vs. 2.0 (zrychlené)
- **Jazyk** -- zkuste anglický text a porovnejte kvalitu s českým

### 5) Images -- generování obrázků

**Endpoint:** `POST /v1/images/generations`

Vygeneruje obrázek z textového popisu.

#### Parametry

| Parametr | Popis |
|---|---|
| `model` | `gpt-image-1` |
| `prompt` | Textový popis požadovaného obrázku |
| `size` | `1024x1024`, `1536x1024` (landscape), `1024x1536` (portrait), `auto` |
| `quality` | `low`, `medium`, `high` |
| `n` | Počet obrázků (1--10) |
| `output_format` | `png`, `jpeg`, `webp` |

#### Příklady k vyzkoušení

**Základní generování:**

```json
{
  "model": "gpt-image-1",
  "prompt": "Minimalistické logo pro aplikaci na zaznamenávání hlasových poznámek, pastelové barvy, čisté linie",
  "size": "1024x1024",
  "quality": "medium"
}
```

**Vysoká kvalita, landscape:**

```json
{
  "model": "gpt-image-1",
  "prompt": "Fotorealistický západ slunce nad Karlovým mostem v Praze, zlatá hodinka, odraz ve Vltavě",
  "size": "1536x1024",
  "quality": "high"
}
```

**Více variant najednou:**

```json
{
  "model": "gpt-image-1",
  "prompt": "Ikona pro mobilní aplikaci na počasí, flat design, modrá a bílá",
  "n": 3,
  "size": "1024x1024",
  "quality": "low"
}
```

#### Co zkoušet měnit

- **`prompt`** -- zkuste vygenerovat logo nebo ikonu pro svůj semestrální projekt
- **`quality`** -- porovnejte `low` vs. `high` pro stejný prompt (vliv na detaily i cenu)
- **`size`** -- zkuste různé poměry stran: čtverec, landscape, portrait
- **`n`** -- vygenerujte více variant a porovnejte

**Endpoint:** `POST /v1/images/edits`

Úpravy existujícího obrázku podle textového popisu (model `dall-e-2`). Vyžaduje nahrání zdrojového obrázku ve formátu PNG.

### 6) Embeddings -- vektorové reprezentace

**Endpoint:** `POST /v1/embeddings`

Převede text na číselný vektor (embedding), který zachycuje jeho význam. Embeddingy se používají pro:

- **Sémantické vyhledávání** -- najdi dokumenty podobné dotazu
- **Clustering** -- seskup podobné texty
- **RAG** (Retrieval-Augmented Generation) -- doplň kontext pro LLM z vlastních dat
- **Klasifikace** a **doporučování**

#### Příklady k vyzkoušení

**Jeden text:**

```json
{
  "model": "text-embedding-3-small",
  "input": "OpenAI API workshop pro vývojáře"
}
```

**Dávka textů pro porovnání:**

```json
{
  "model": "text-embedding-3-small",
  "input": [
    "Jak vařit svíčkovou na smetaně",
    "Recept na tradiční český guláš",
    "Instalace Node.js na Linux"
  ]
}
```

**Menší dimenze (úspora storage):**

```json
{
  "model": "text-embedding-3-large",
  "input": "Sémantické vyhledávání v dokumentech",
  "dimensions": 256
}
```

#### Co zkoušet měnit

- **`model`** -- `text-embedding-3-small` (rychlejší, levnější) vs. `text-embedding-3-large` (přesnější, až 3072 dimenzí)
- **`dimensions`** -- zkuste snížit počet dimenzí (např. 256) -- menší vektory, levnější storage
- **`input`** -- pošlete sémanticky podobné a odlišné texty a v odpovědi porovnejte výsledné vektory

### 7) Models -- výpis dostupných modelů

**Endpoint:** `GET /v1/models`

Vrátí seznam všech modelů dostupných pro váš API klíč. Užitečné pro ověření, ke kterým modelům máte přístup.

**Endpoint:** `GET /v1/models/{model_id}`

Vrátí detail konkrétního modelu -- zkuste zadat např. `gpt-4o-mini`.

## Experimenty k vyzkoušení

1. **Porovnání temperature** -- pošlete stejný dotaz přes Chat Completions s `temperature` 0, 0.7 a 1.5. Jak se liší odpovědi? Zaznamenejte si pozorování.

2. **System prompt** -- v Chat Completions vytvořte system prompt, který změní chování modelu (např. "odpovídej jako obchodník na tržišti"). Ověřte, že model dodržuje instrukce.

3. **Responses API s web search** -- použijte Responses endpoint s nástrojem `web_search` a položte otázku na aktuální událost. Porovnejte odpověď s a bez web search.

4. **Generování obrázku** -- vygenerujte logo nebo ikonu pro svůj semestrální projekt. Experimentujte s různými prompty a nastaveními `quality` / `size`.

5. **Text-to-Speech** -- převeďte krátký text do řeči s různými hlasy a styly. Zkuste měnit `instructions` a `speed`.

6. **Embeddings** -- vygenerujte embeddingy pro 3 sémanticky podobné věty a 1 odlišnou. Zamyslete se, jak by se daly embeddingy využít ve vašem projektu.
