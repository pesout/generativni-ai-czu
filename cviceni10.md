# X. Vlastní AI model na serveru

## Prerekvizity

> Pro toto cvičení potřebujete:
> - VPS s Dockerem a Codexem ze [cvičení 6](cviceni6.md) a [cvičení 7](cviceni7.md), do kterého se připojujete přes VS Code Remote-SSH
> - Funkční Traefik s HTTPS na `*.aibr.cz` (ze [cvičení 6](cviceni6.md))

### 1) Rychlá kontrola

Připojte se na server přes VS Code Remote-SSH a v terminálu ověřte:

```bash
codex --version
docker --version
docker ps  # běží traefik, vaše aplikace, n8n
```

Pokud něco nesedí, vraťte se k dřívějším cvičením.

---

## Proč lokální model

Doteď jsme s LLM komunikovali přes OpenAI API ([cvičení 4](cviceni4.md), [5](cviceni5.md)) nebo přes OpenClaw napojený na OpenAI ([cvičení 9](cviceni9.md)). To znamená, že každý token vaší aplikace vidí někdo třetí a každý měsíc za to platíte. **Lokální model** běží přímo na vašem VPS -- žádná data ven, po stažení žádné průběžné náklady, funguje i bez internetu.

Limit je v kvalitě a rychlosti: model, který vám poběží na 2 vCPU bez GPU, se v žádném případě neblíží ke komerčním cloudovým modelům. Pro chat s pár větami, klasifikaci, extrakci nebo rerankování to ale často stačí. V tomto cvičení si lokální model nasadíme a porovnáme ho s tím, co známe z cloudu.

---

## Kde hledat modely

Hlavním rozcestníkem je [**Hugging Face**](https://huggingface.co) -- registr otevřených modelů s vyhledáváním, *model cards* (popisem, jak model trénovali, na čem, s jakými limity), licencemi a komunitními benchmarky. Vedle něj existují i menší alternativy ([Ollama Library](https://ollama.com/library), Kaggle Models), ale Hugging Face je 95 % všeho, co potřebujete znát.

### 2) Co sledovat při výběru modelu

- **Počet parametrů** (typicky `1B`, `3B`, `8B`, ... -- B = miliardy). Větší model = chytřejší, ale potřebuje víc RAM.
- **Kvantizace** (`Q4_K_M`, `Q5_K_M`, `Q8_0`, ...). Kvantizace znamená kompresi vah na nižší přesnost. Model "zaokrouhlíte" tak, aby byl násobně menší a vlezl se i do skromné RAM, za cenu malé ztráty kvality. Model označený `Q4_K_M` je často zlatý střed (zhruba 4× menší soubor, v běžném chatu kvalitu skoro nepoznáte), vyšší číslo = větší a kvalitnější, nižší = menší a hloupější. Soubory v této podobě mají příponu `.gguf`.
- **Licence.** `Apache 2.0` a `MIT` jsou bezstarostné. `Llama Community License` (Meta) povoluje jen omezené komerční použití. Některé modely mají vlastní omezení, která jsou popsána v licenci.
- **Jazyky.** Drtivá většina open modelů má v *model card* uvedený seznam oficiálně podporovaných jazyků. Pozor -- bývá kratší, než se zdá: Llama 3.2 oficiálně uvádí 8 jazyků (`en, de, fr, it, pt, hi, es, th`), čeština mezi nimi **není**. Model byl trénovaný na širším korpusu a česky obstojně rozumí, ale generuje slabší než v podporovaných jazycích. Pro silnější češtinu sahejte po **Qwen2.5** nebo **Gemma 2** -- mají explicitnější multilingual pokrytí.
- **Aktuálnost.** Datum poslední úpravy a počet stažení = jak moc je model živý/používaný. Modely starší než rok jsou v 2026 obvykle překonané.

### 3) HW požadavky podle velikosti modelu

Velmi orientační, pro `Q4_K_M` kvantizaci a CPU inference:

| Velikost | RAM (Q4) | Vhodné pro |
|---|---|---|
| ~1B | ~1 GB | malý VPS, zařízení  typu Raspberry Pi 5 |
| ~3B | ~2-3 GB | běžný cloud VPS |
| ~7-8B | ~5-6 GB | desktop / silnější VPS |
| ~13B | ~9-10 GB | workstation |
| ~30B+ | 20+ GB | dedikovaný HW nebo GPU |

> Bez GPU počítejte s rychlostí jednotek tokenů za sekundu. Pro chat a krátké úlohy stačí, na batch processing nebo dlouhé reporty se to nehodí.

### 4) Jak zjistit HW vašeho serveru

```bash
lscpu              # procesor: počet jader, architektura, frekvence
free -h            # RAM: total, used, available
lspci | grep -i vga  # GPU (na typickém VPS jen virtuální)
df -h              # volné místo na disku (model si stáhne ~1-2 GB)
```

### 5) Doporučený model pro typický server kurzu

Hetzner VPS, který v kurzu používáme, má **2 vCPU, 3.7 GB RAM, žádné GPU**. S běžící aplikací a n8n vám zbývá zhruba 2 GB volné RAM. Doporučení:

> [`bartowski/Llama-3.2-1B-Instruct-GGUF`](https://huggingface.co/bartowski/Llama-3.2-1B-Instruct-GGUF) v kvantizaci `Q4_K_M` (~0.8 GB na disku, ~1 GB v RAM). Veřejný GGUF mirror Meta Llama 3.2 1B, bez tokenu, default tag pro Ollama.

Alternativy:

- **Qwen2.5 1.5B Instruct** (Apache 2.0, ~1.2 GB Q4) -- silnější v matematice, kódu a v češtině.
- **Phi-3.5 mini** (3.8B, ~2.5 GB Q4) -- chytřejší, ale potřebuje uvolnit zdroje (sekce 6).
- **TinyLlama 1.1B** -- ještě menší, ale citelně slabší kvalita.

---

## (Volitelné) Uvolnění zdrojů serveru

### 6) Zastavení aplikace a n8n

> Tato sekce je **volitelná**. Pokud chcete pustit větší model (Phi-3.5 mini, Llama 3.2 3B) nebo prostě zrychlit inference s víc RAM, dočasně zastavte aplikaci a n8n. Po cvičení je zase spustíte.

Nejjednodušší cesta je říct si o to Codexu na serveru:

```
Dočasně zastav moji aplikaci a n8n, ať se uvolní RAM pro lokální LLM.
Cesty zjisti z ~/AGENTS.md (nebo z `docker ps`). Po stopu mi ukaž
`free -h` a vypiš příkaz, kterým je oba zase nahodím zpátky.
```

> Pro zajímavost -- ruční alternativa:
>
> ```bash
> cd ~/apps/<vase-aplikace> && docker compose stop
> cd ~/n8n && docker compose stop
> free -h  # ověřte, kolik RAM se uvolnilo
> ```
>
> Restart zpět:
>
> ```bash
> cd ~/apps/<vase-aplikace> && docker compose start
> cd ~/n8n && docker compose start
> ```

> **Pozor:** stopnutím aplikace shodíte i web na `<jmeno>.aibr.cz`.

---

## Instalace lokálního LLM přes Codex

Použijeme **[Ollama](https://ollama.com)** jako runtime pro inference (stáhne a spustí GGUF modely) a **[Open WebUI](https://github.com/open-webui/open-webui)** jako chat UI. Oba běží v Dockeru, oba se napojí do existující Traefik sítě, takže chat dostanete na `https://llm.<jmeno>.aibr.cz` přes HTTPS bez další ruční konfigurace.

Ollama umí pullovat GGUF modely přímo z Hugging Face přes syntaxi `hf.co/<user>/<repo>:<kvantizace>` -- propojení na HF tedy zůstává a nemusíte zvlášť konvertovat soubory.

### 7) Codex prompt -- nasazení Ollama + Open WebUI

Na serveru ve VS Code Remote-SSH připravte adresář a spusťte Codex:

```bash
mkdir -p ~/local-llm
cd ~/local-llm
codex
```

Prompt:

```
V aktuálním adresáři (~/local-llm) nasaď lokálně provozovaný open-source
LLM, kterému budu posílat zprávy přes chat UI v prohlížeči. Cíl: po
dokončení mám v prohlížeči chat na vlastní subdoméně, který odpovídá
lokálním modelem -- bez OpenAI API a bez internetu.

Postup si vezmi z aktuální dokumentace (ollama.com/docs,
github.com/ollama/ollama, github.com/open-webui/open-webui,
huggingface.co/docs/hub/ollama). Verze a flagy se mezi vydáními mění,
takže si je ověř.

Tvrdá omezení:

- Ollama i Open WebUI běží v Dockeru přes docker-compose v ~/local-llm,
  ne přímo na hostiteli.
- Připoj kontejnery do existující Traefik sítě -- chat UI vystav
  přes HTTPS na sub-doméně llm.<jmeno>.aibr.cz. Konkrétní jméno serveru
  (<jmeno>) zjisti z ~/AGENTS.md nebo z Traefik labelů jiných služeb,
  netipuj.
- Stáhni model bartowski/Llama-3.2-1B-Instruct-GGUF v kvantizaci Q4_K_M.
  Ollama pullne přímo z Hugging Face přes syntaxi
  hf.co/<user>/<repo>:<kvantizace>. Pull dělej až po startu kontejneru
  proti běžícímu Ollama (např. docker compose exec).
- Login: spolehni se na vestavěnou autentizaci Open WebUI (první
  registrovaný účet je admin, další jsou pending dokud je admin
  nepustí). Zveřejněnou registraci vypni přes ENABLE_SIGNUP=False
  v ~/local-llm/.env (chmod 600), ať si nikdo cizí nezaloží účet.
  Žádné Basic Auth navíc nepotřebuju.
- Persistence: stažený model i historie konverzací přežijí restart
  serveru (named volumes pro Ollama models a Open WebUI data).
- Memory limit kontejnerům nastav konzervativně (Ollama + Open WebUI
  dohromady ~2.5 GB), ať to nezabije ostatní běžící služby.

Po dopsání mi ukaž:
1. `docker compose ps` + posledních ~20 řádků logů obou kontejnerů,
2. obsah ~/local-llm/docker-compose.yml a strukturu ~/local-llm/.env
   (jen názvy proměnných, ne hodnoty),
3. end-to-end test -- v UI nebo přes Ollama HTTP API pošli zprávu
   "Ahoj, představ se v jedné větě česky." a vrať mi odpověď modelu,
4. doplň ~/AGENTS.md o nové LLM kontejnery: jaká subdoména, jaký model,
   kde jsou data, jak přidat další model, jak restartovat.
```

### 8) Otestujte to

Otevřete `https://llm.<jmeno>.aibr.cz`. Při prvním přístupu si vytvoříte účet -- první registrace dostane admin práva. Další registrace jsou zablokované (`ENABLE_SIGNUP=False`), takže pokud byste chtěli pustit dovnitř někoho jiného, založíte mu účet v *Settings -> Admin Panel*.

Vyzkoušejte:

- "Ahoj, pozdrav mě."
- "Napiš mi v Pythonu funkci, která vrátí faktoriál čísla."
- "Vysvětli rozdíl mezi `let` a `const` v JavaScriptu, krátce."
- Stejné dotazy zkuste položit v ChatGPT nebo Claude a porovnejte rychlost a kvalitu.

Při prvním dotazu může model "rozjíždět" pár sekund -- načítá se do paměti. Další dotazy jdou rychleji.

### 9) Přidání dalšího modelu

V Open WebUI je přepínač modelů v UI. Další model stáhnete přes `docker compose exec` v adresáři `~/local-llm` (název služby si Codex zvolil v compose souboru -- typicky `ollama`):

```bash
cd ~/local-llm
docker compose exec ollama ollama pull hf.co/bartowski/Qwen2.5-1.5B-Instruct-GGUF:Q4_K_M
docker compose exec ollama ollama list  # ukáže nainstalované modely
```

V UI se po refreshi nový model objeví v dropdownu.

Nebo o to klasicky poproste Codex 🙂

---

## Kdy lokální model dává smysl

| Důvod | Lokální model | Cloudové API |
|---|---|---|
| Citlivá data (GDPR, zdravotní, interní) | nic neopouští server | citlivá data v cloudu |
| Cena při velkém objemu | jen HW | platba za token |
| Offline / air-gapped prostředí | funguje bez internetu | nefunguje |
| Komplexní reasoning, dlouhý kontext | slabší | bez problému |
| Multimodalita (obrázky, audio) | omezená | dostupná |
| Latence pro veřejný chatbot | bez GPU pomalé | rychlé |

V praxi se hodí **hybridní cesta** -- pro klasifikaci, extrakci nebo rerankování pustíte malý lokální model, pro náročné generování saháte po cloudu.

---

## Shrnutí

Po dokončení tohoto cvičení byste měli mít:

1. Přehled o tom, jak na Hugging Face vybrat model podle velikosti, kvantizace a licence.
2. Schopnost zjistit HW vlastního serveru (`lscpu`, `free -h`, `lspci`).
3. Lokálně běžící Ollama + Open WebUI s Llama 3.2 1B (nebo ekvivalentem) na `llm.<jmeno>.aibr.cz`.
4. Aktualizovaný `~/AGENTS.md` o LLM stack a postup pro přidání dalších modelů.
