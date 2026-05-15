# XI. Vlastní RAG systém s vektorovou databází

## Prerekvizity

> Pro toto cvičení potřebujete:
> - VPS s Dockerem a Codexem ze [cvičení 6](cviceni6.md) a [cvičení 7](cviceni7.md), do kterého se připojujete přes VS Code Remote-SSH
> - Funkční Traefik s HTTPS na `*.aibr.cz` (ze [cvičení 6](cviceni6.md))
> - Subdoménu `llm.<jmeno>.aibr.cz` z [cvičení 10](cviceni10.md) -- zrecyklujeme ji pro RAG, lokální LLM přitom zastavíme
> - `OPENAI_API_KEY` na serveru -- už ho máte v `~/n8n/.env` z [cvičení 8](cviceni8.md), Codex ho odtud vytáhne sám

### 1) Rychlá kontrola

Připojte se na server přes VS Code Remote-SSH a v terminálu ověřte:

```bash
codex --version
docker --version
docker ps  # běží traefik, vaše aplikace, n8n a (zatím) local-llm z cv10
```

Pokud něco nesedí, vraťte se k dřívějším cvičením.

---

## Co je to RAG a kdy ho použít

**RAG** (*Retrieval-Augmented Generation*) je technika, jak nechat LLM odpovídat nad vlastními dokumenty -- bez fine-tuningu, bez přeučování modelu. Pointa je jednoduchá: než LLM pošlete dotaz, najdete v indexu několik nejrelevantnějších úryvků z vašich materiálů a přibalíte je modelu do kontextu. Model pak odpovídá *podle nich*, ne podle své paměti, a může u odpovědi ukázat citace.

Typické use-case:

- Firemní wiki, interní směrnice, manuály.
- Smlouvy, výroční zprávy, právní podklady.
- Studijní materiály, skripta, poznámky.
- Zákaznická podpora nad produktovou dokumentací.
- Osobní "druhý mozek" nad vašimi soubory.

Než si RAG začnete stavět sami, zvažte, jestli vám nestačí hotové řešení:

| Nástroj | Pro koho | Limity |
|---|---|---|
| **ChatGPT Projects / Custom GPT** | jednotlivec, pár desítek souborů | velikost a počet souborů, žádná integrace do vlastní app |
| **Claude Projects** | jednotlivec, dokumenty se vejdou do kontextu | malý objem, bez vlastní logiky |
| **NotebookLM** | studium, výzkum, podcasty z poznámek | jen v Google ekosystému, žádné API pro vlastní integraci |
| **OpenAI Assistants API + file_search** | vývojáři, kteří chtějí managed RAG | menší kontrola nad chunkováním a citacemi, vendor lock-in |
| **Microsoft Copilot Studio** | firmy v M365 ekosystému | licence, integrace cílená na Microsoft stack |

Vlastní RAG má smysl, když potřebujete:

- **Kontrolu nad daty** -- citlivé materiály nesmí ven z vašeho serveru (GDPR, NDA, interní podklady).
- **Integraci do vlastní aplikace nebo API** -- chatbot na webu, automatizace, napojení na n8n.
- **Vlastní logiku** chunkování, rerankování, filtrů nad metadaty (autor, datum, sekce).
- **Kontrolu nákladů u velkých objemů** -- u milionů dokumentů se cena hosted řešení škáluje rychleji, než vlastní stack na vlastním HW.

Naopak pokud chcete **co nejméně práce** a máte jen pár desítek souborů bez integrace, hosted řešení vám dá výsledek za pár minut. V praxi se u větších systémů hodí **hybrid** -- managed nástroj pro rychlý prototyp, vlastní stack ve chvíli, kdy začnete narážet na limity.

Pro učení se vlastní RAG navíc hodí -- v hotových nástrojích je celý proces skrytý, nepoučíte se z něj a to by byla škoda.

### 2) Klíčové pojmy

- **Embedding** -- text se převede na vektor čísel (typicky 1536 nebo 3072 dimenzí), který zachycuje *význam*. Texty s podobným významem mají blízké vektory v prostoru. Embedding je deterministický: stejný text -> stejný vektor.
- **Chunk** -- dokument se rozseká na kratší úseky (~300--800 tokenů). Příliš malé chunky ztrácejí kontext ("...a proto je to nelegální" bez okolí nic neznamená), příliš velké rozmlží relevanci (model dostane půl stránky kvůli jedné větě).
- **Overlap** -- sousední chunky se mezi sebou částečně překrývají (~10--20 % délky). Důvod: aby se neztratil kontext na rozhraní -- důležitá věta nemá padnout přesně mezi dva chunky.
- **Vektorová databáze** -- ukládá embeddingy spolu s metadaty (zdroj, pozice, původní text). Umí rychle najít *k nejbližších* vektorů k zadanému dotazu (typicky cosinová podobnost).
- **Top-k retrieval** -- po embeddingu dotazu se z databáze vytáhne *k* nejbližších chunků (typicky 3--5). Tyhle chunky jdou jako kontext do LLM.
- **System prompt** -- pevná instrukce modelu, jak s dodanými chunky pracovat. Typicky: *"Odpovídej výhradně podle dodaných úryvků. Pokud v nich odpověď není, řekni, že to nevíš."* Bez toho si model rád dovymýšlí.
- **Citace** -- u odpovědi ukážeme, ze kterého souboru (a kterého úryvku) model čerpal. Bez citací RAG ztrácí půlku své hodnoty -- nejde ověřit původ.

### 3) Náš tech stack

- **[Qdrant](https://qdrant.tech)** -- open-source vektorová DB v Rustu. Rychlá, jednoduchá REST/gRPC API, běží v Dockeru (`qdrant/qdrant:latest`, persistentní storage v `/qdrant/storage`).
- **OpenAI Embeddings API** (`text-embedding-3-small`, 1536 dimenzí) -- multilingual včetně češtiny, výborná kvalita, cena ~$0.02 / 1M tokenů (řádově centy za běžnou kolekci).
- **PDF parsing** -- doporučení: [`pypdf`](https://pypdf.readthedocs.io) (čistě Python, BSD licence). Konkrétní knihovnu necháme vybrat Codex; pokud sáhne po `PyMuPDF`, hlídejte si, že má AGPL licenci (problém pro komerční projekty). Plaintext (`.txt`, `.md`) se čte přímo bez parseru.
- **[Chainlit](https://chainlit.io)** -- Python framework na chat UI s vestavěnou podporou citací (přes `cl.Text` elements) a streamování odpovědí. Píše se v něm jen pár desítek řádků. Od května 2025 je community-maintained, vývoj pokračuje.
- **OpenAI Chat Completions** (`gpt-4o-mini`) -- generátor odpovědí. Levný, rychlý, česky umí dobře.
- Všechno v Dockeru, napojené do existující Traefik sítě, HTTPS na sub-doméně bez další ruční konfigurace.

> Embeddingy i chat běží přes OpenAI API. Pokud chcete plně lokální RAG (kvůli citlivým datům), můžete později vyměnit OpenAI za lokální embedding model (`bge-m3`, `nomic-embed-text`) a lokální chat model z [cvičení 10](cviceni10.md). V tomto cvičení jdeme cestou nejmenšího odporu.

---

## Uvolnění subdomény z cvičení 10

V [cvičení 10](cviceni10.md) jsme `llm.<jmeno>.aibr.cz` přiřadili Open WebUI. Pro RAG ji zrecyklujeme -- Open WebUI dočasně zastavíme. Pokud lokální LLM nepoužíváte, můžete kontejnery i smazat, ale na to přijde čas až později. Výhoda je, že tím uvolníme systémové prostředky.

### 4) Zastavení lokálního LLM

V Codexu na serveru:

```
Dočasně zastav Ollama + Open WebUI z cvičení 10, ať se uvolní subdoména
llm.<jmeno>.aibr.cz a RAM pro nový RAG stack. Cesty zjisti z ~/AGENTS.md
(typicky ~/local-llm). Po stopu mi ukaž `free -h`, `docker ps` a vypiš
příkaz, kterým je zase nahodím zpátky, kdybych je chtěl vrátit.
```

> Stopnutím Open WebUI ztratíte chat z cvičení 10 -- subdoménu zabere nový RAG. Konverzace ale zůstávají v named volume, takže po případném restartu Open WebUI o nic nepřijdete.

---

## Nahrání vlastních dokumentů

### 5) Příprava adresáře `docs/`

Na serveru ve VS Code Remote-SSH si připravte pracovní adresář:

```bash
mkdir -p ~/rag/docs
```

Do `~/rag/docs/` nahrajte své PDF a/nebo `.txt` soubory. Nejjednodušší cesta: ve VS Code Remote-SSH otevřete složku `~/rag/docs` v Exploreru a soubory do ní jednoduše přetáhněte z lokálního disku.

Doporučení k obsahu pro první test:

- Vlastní studijní materiály, skripta nebo poznámky.
- Manuály nebo dokumentaci k něčemu, s čím pracujete.
- Smlouvy, výroční zprávy, regulační texty.
- Cokoli, kde víte, co byste se chtěli zeptat a snadno ověříte správnost odpovědi.

Pro první test stačí 2--5 souborů, ať indexace trvá vteřiny, ne minuty.

> **Pozor na scanované PDF.** Pokud je PDF jenom obrázek (scan papíru bez OCR), parser z něj text nedostane. Otestujete tak, že si v PDF prohlížeči zkusíte text **označit a zkopírovat** -- pokud to nejde, je to scan a do RAGu se v této podobě nehodí. Buď ho proženete OCR (např. `ocrmypdf`), nebo ho z první iterace vynecháte.
>
> POZOR: Pracujeme záměrně jen s PDF a plaintextem (`.txt`, `.md`). Word, Excel, HTML a další formáty by šly přidat, ale komplikovaly by parsing -- nechte si je na pozdější iterace.

---

## Nasazení RAG stacku přes Codex

### 6) Codex prompt -- celý RAG

Na serveru ve VS Code Remote-SSH:

```bash
cd ~/rag
codex
```

Prompt:

```
V aktuálním adresáři (~/rag) postav RAG systém, který bude odpovídat na
dotazy nad mými dokumenty v ~/rag/docs/. Cíl: po dokončení mám
v prohlížeči na https://llm.<jmeno>.aibr.cz chat, kde se zeptám a dostanu
odpověď postavenou nad mými PDF/textovými soubory, s citacemi na
zdrojové soubory.

Stack je daný:
- Qdrant jako vektorová databáze (image qdrant/qdrant)
- OpenAI Embeddings API (text-embedding-3-small) pro embeddingy
- Chainlit jako chat UI v Pythonu
- OpenAI Chat Completions (gpt-4o-mini) jako generátor odpovědí

Konkrétní knihovnu na PDF parsing, verze balíčků a přesné API volání si
vyber sám podle aktuální dokumentace (qdrant.tech/documentation,
docs.chainlit.io, platform.openai.com/docs/guides/embeddings,
platform.openai.com/docs/api-reference). Verze a flagy se mění, ověř si
je.

Tvrdá omezení:

- Všechno přes docker-compose v ~/rag: jeden kontejner Qdrant, jeden
  kontejner s Chainlit aplikací (Python). Žádný proces přímo na hostiteli.
- Připoj kontejnery do existující Traefik sítě a chat UI vystav přes HTTPS
  na sub-doméně llm.<jmeno>.aibr.cz. Konkrétní jméno serveru (<jmeno>)
  zjisti z ~/AGENTS.md nebo z Traefik labelů jiných služeb, netipuj.
  Předtím ověř, že subdoména už není obsazená Open WebUI z cv10 (mělo by
  již být zastaveno, případně zastav).
- Qdrant nevystavuj ven, jen na interní docker síti (žádné `ports:`
  mapování 6333/6334 na hostitele). Jeho data persisti přes named volume
  připojený na /qdrant/storage, ať reindexace přežije restart.
- ~/rag/docs/ bind-mountni do app kontejneru jako read-only.
- OPENAI_API_KEY už existuje na serveru v ~/n8n/.env (z cvičení 8).
  Zkopíruj ho do ~/rag/.env (chmod 600), ať mám klíče oddělené
  per-služba a nemusím nic ručně vyplňovat. Pokud v ~/n8n/.env klíč
  z jakéhokoliv důvodu chybí, řekni mi to a zastav -- nevymýšlej.
  Do ~/rag/.env dále vygeneruj náhodný CHAINLIT_AUTH_SECRET (Chainlit
  ho vyžaduje pro podpis session tokenů) a admin heslo do chatu.
  V repu/compose nikdy plaintext hodnoty.
- Ingestion: při startu app kontejneru projdi ~/rag/docs/, naparsuj PDF
  a plaintext soubory, rozsekej text na chunky (cca 600 tokenů, overlap
  ~100 tokenů), spočítej embeddingy přes OpenAI a ulož je do Qdrant
  kolekce spolu s metadaty (název souboru, pořadí chunku, původní text).
  Pokud kolekce už existuje a soubory se od poslední indexace nezměnily
  (porovnej hash), neindexuj znovu, ať při restartu neplatím embeddingy
  podruhé.
- Query workflow v Chainlitu: dotaz -> embedding -> top-5 nejbližších
  chunků z Qdrantu -> složení promptu (system instrukce + nalezené chunky
  + dotaz) -> OpenAI chat -> odpověď. K odpovědi v UI ukaž citace --
  doporučená cesta je vytvořit cl.Text elementy se zdrojovým textem
  (display="side") a předat je do cl.Message(elements=...); přesnou
  cestu ale zvol podle aktuálního Chainlit API.
- System prompt měj v samostatném souboru ~/rag/prompts/system.md, ať ho
  můžu upravit bez rebuild image. Výchozí znění: model odpovídá výhradně
  podle dodaných úryvků; pokud v nich odpověď není, otevřeně řekne, že
  to v dokumentech není.
- Login do Chainlitu: použij vestavěnou password authentikaci
  (@cl.password_auth_callback) s jedním admin účtem, credentials a
  CHAINLIT_AUTH_SECRET z .env. Žádné Basic Auth navíc nepotřebuju.
- Memory limit kontejnerům nastav konzervativně (Qdrant + app dohromady
  ~1.5 GB), ať to nezabije aplikaci a n8n.

Po dopsání mi ukaž:
1. `docker compose ps` a posledních ~20 řádků logů obou kontejnerů,
   včetně logu ingestion (kolik souborů, kolik chunků, kolik embeddingů).
2. Obsah ~/rag/docker-compose.yml, strukturu ~/rag/.env (jen názvy
   proměnných, ne hodnoty), klíčové části app kódu (ingestion + query)
   a obsah ~/rag/prompts/system.md.
3. End-to-end test: pošli do chatu konkrétní dotaz, na který je odpověď
   v jednom z mých dokumentů, a vrať mi odpověď i s citací.
4. Doplň ~/AGENTS.md o RAG stack: jaká subdoména, kde jsou docs, jak
   přidat dokument a reindexovat, kde se mění system prompt, jak vyměnit
   model.
```

Indexace se spustí při startu app kontejneru. Pokud máte v `docs/` jen pár souborů, doběhne během vteřin; u větší kolekce počítejte s minutami. Sledujte logy (`docker compose logs -f`), v nich uvidíte, kolik chunků se vytvořilo a kolik embeddingů odešlo.

---

## Jak to celé funguje

### 7) Tok ingestion

Spustí se jednou při startu app kontejneru (a znovu po přidání nového souboru a restartu):

- `docs/` -> načtení seznamu souborů
- PDF -> text (přes parser), plaintext -> načtený rovnou
- text -> rozsekání na chunky (~600 tokenů) s overlapem (~100 tokenů)
- každý chunk -> embedding přes OpenAI API (`text-embedding-3-small`)
- vektor + metadata (zdroj, pozice, původní text chunku) -> Qdrant kolekce

Výsledkem je naplněná vektorová databáze, ve které vektor reprezentuje význam každého úryvku.

### 8) Tok user query

Spustí se pokaždé, když uživatel pošle zprávu:

- uživatel napíše dotaz v Chainlit UI
- dotaz -> embedding přes OpenAI (stejný model jako u ingestion)
- Qdrant vrátí top-5 nejbližších chunků (podle cosinové podobnosti)
- aplikace složí prompt: **system instrukce** (z `prompts/system.md`) + **nalezené chunky** (s názvy zdrojů) + **dotaz uživatele**
- OpenAI Chat Completions vrátí odpověď
- Chainlit zobrazí odpověď v UI a pod ní citace -- názvy zdrojových souborů a krátké úryvky, ze kterých model čerpal

Klíčový moment je system prompt -- ten určuje, jestli model bude poctivě říkat "v dokumentech to není", nebo si bude dovymýšlet. Proto je v samostatném souboru, ať s ním můžete experimentovat bez rebuildu.

---

## Otestujte to

### 9) Testovací tipy

Otevřete `https://llm.<jmeno>.aibr.cz`, přihlaste se admin účtem z `.env`. Zkuste:

- **Konkrétní dotaz, jehož odpověď víte, že v dokumentech je.** Zkontrolujte, že citace ukazuje na správný soubor a že odpověď sedí. Toto je základní test, jestli retrieval funguje.
- **Dotaz, na který v dokumentech odpověď *není*.** Dobře nastavený model řekne *"v dodaných materiálech to není"*. Pokud si přesto vymyslí odpověď, doladěte system prompt v `~/rag/prompts/system.md` (např. přidejte *"Nikdy nehádej. Pokud odpověď není doslova v úryvcích, řekni, že to nevíš."*) a restartujte app kontejner.
- **Stejný dotaz dvakrát s drobně jiným zněním** ("co je X" vs. "vysvětli mi X"). Mělo by vrátit podobné chunky -- pokud ne, embedding nebo retrieval má problém.
- **České i anglické dotazy** (pokud máte i takové dokumenty). `text-embedding-3-small` je multilingual a měl by je propojit.
- **Přidání dalšího PDF** -- nahrajte ho do `~/rag/docs/`, restartujte app kontejner (`docker compose restart app` nebo o to požádejte Codex) a sledujte v logu, kolik chunků z něj vzniklo.

Když odpovědi neodpovídají očekávání:

- Zkontrolujte v logu ingestion, jestli se soubory opravdu naparsovaly a kolik chunků z nich vzniklo. Nula chunků = parser z PDF nedostal text (typicky scan bez OCR).
- Zkuste upravit chunkování (větší/menší chunky, větší/menší overlap) a porovnejte kvalitu citací. U strukturovaných dokumentů (kapitoly, sekce) chce parser splittovat po sekcích, u volného textu stačí pevná délka.
- Zkuste navýšit `top-k` z 5 na 8-10, pokud model dostává příliš málo kontextu. Naopak pokud začne "halucinovat z příliš mnoha zdrojů", snižte.
- Podívejte se v UI na citace -- jestli sedí, retrieval funguje a problém je v generaci; jestli nesedí, problém je v retrievalu (embedding model, chunky, overlap).

---

## Shrnutí

Po dokončení tohoto cvičení byste měli mít:

1. Přehled, co je RAG, kdy ho stavět vlastní a kdy stačí hosted řešení jako NotebookLM nebo ChatGPT Projects.
2. Pochopení pojmů embedding, chunk, overlap, top-k retrieval a system prompt -- a proč na nich záleží.
3. Běžící RAG stack (Qdrant + Chainlit + OpenAI) na `llm.<jmeno>.aibr.cz` s vlastními PDF a textovými dokumenty z `~/rag/docs/`, s citacemi v odpovědích.
4. Aktualizovaný `~/AGENTS.md` o RAG kontejnery, postup pro přidání dokumentu, reindexaci a změnu system promptu.
