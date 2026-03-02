# I. Příprava prostředí, principy generativní AI

## Příklady projektů pro inspiraci

**1) Aplikace na počítání výdajů za nákupy** -- do aplikace fotím účtenky z nákupů a dostávám měsíční přehledy a nejčastěji nakupované položky.

**2) Podpůrná platforma pro výuku předmětu** -- na základě osnovy připraví studijní texty, chatbota pro doplňující otázky a automaticky vygeneruje test na ověření znalostí.

**3) Tip na zítřejší výlet** -- aplikace, která podle aktuální polohy, počasí a dne v týdnu dá 1 konkrétní tip na výlet v okolí na zítřek. Aplikace si bude pamatovat předchozí tipy, aby vždy nabízela něco nového.

## Příprava prostředí pro vývoj

> Je potřeba, aby uživatelé mohli testovat svou appku během vývoje (v dev módu) v lokálním prohlížeči -- musí být tedy dostupná zvenku aspoň pomocí tunelu.

### 1) Registrace na Github + testovací repozitář

- registrovat na [https://github.com/signup](https://github.com/signup) (možné přes účet Google)
- vytvořit repozitář [https://github.com/new](https://github.com/new)

### 2) Instalace Gitu (Windows)

> Na Linuxu a Macu je git obvykle předinstalovaný. Na Windows je potřeba nainstalovat Git, který obsahuje **Git Bash** — terminál s unixovým prostředím.

- stáhnout a nainstalovat z [https://git-scm.com/install/windows](https://git-scm.com/install/windows)
- při instalaci ponechat výchozí nastavení
- po instalaci otevřít **Git Bash** — všechny další příkazy na Windows zadávat zde

### 3) SSH klíč pro GitHub

V terminálu (Linux / Mac) nebo Git Bash (Windows):

```bash
# vygenerování SSH klíče
ssh-keygen -t ed25519 -C "mail@domena.cz" -f ~/.ssh/id_ed25519

# spuštění SSH agenta
eval "$(ssh-agent -s)"

# přidání klíče do agenta
ssh-add ~/.ssh/id_ed25519
```

### 4) Vložit SSH klíč na GitHub

- zavolat příkaz a zkopírovat výstup -- `cat ~/.ssh/id_ed25519.pub`
- GitHub -- Settings → SSH and GPG keys → New SSH key → vložit → Add SSH key
- ověřit v terminálu -- `ssh -T git@github.com`

### 5) Instalace node, nvm, npm

- Node Version Manager
    - Linux/Mac -- [https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating](https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating)
    - Windows -- [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases)
- instalace Node.js + npm -- `nvm install --lts`
- ověření -- `node -v && npm -v`

### 6) Instalace OpenAI Codex

- instalace -- `npm i -g @openai/codex`
- ověření -- `codex --version`

### 7) Lokální instalace VS Code nebo alternativního editoru

- [VS Code](https://code.visualstudio.com)
- [Cursor](https://cursor.com/download)
- [Google Antigravity](https://antigravity.google)

### 8) Vyklonovat repozitář

- repozitář → Code → SSH
- v terminálu -- `git clone git@github.com:xxxx/xxxx.git`
- přejít do projektu -- `cd <složka>`

### 9) Demonstrace správného nastavení

- propojit s OpenAI pomocí kódu -- `codex login --device-auth`
- spustit codex ve složce s projektem -- `codex`
- povolit změny bez schvalování
- vybrat model pomocí `/model`
- založit AGENTS.md pomocí `/init`
- zadat prompt: *Create template (headings) in AGENTS.md file, add project description to README.md. Commit and push all changes.*
- zkontrolovat výsledek na GitHubu

### 10) Přehled užitečných příkazů v Codexu

Příkazy se zadávají přímo do promptu Codexu (začínají lomítkem `/`).

| Příkaz | K čemu slouží |
|---|---|
| `/model` | Přepne aktivní model (např. gpt-5.3-codex / gpt-5.1-codex-mini). |
| `/plan` | Přepne do plánovacího režimu — Codex nejdřív navrhne plán a teprve po odsouhlasení implementuje. Hodí se pro větší úpravy. |
| `/permissions` | Změní úroveň oprávnění (Auto / Read Only apod.) uprostřed práce, bez restartu. |
| `/diff` | Ukáže git diff včetně netreackovaných souborů — rychlá kontrola, co se změnilo. |
| `/review` | Spustí review pracovního stromu na provedené změny, typicky po dokončení úprav. |
| `/mention` | Připne konkrétní soubor do konverzace, aby na něj Codex cíleně odkazoval. |
| `/compact` | Shrne dosavadní konverzaci a uvolní kontext — užitečné po dlouhých sezeních, kdy se blíží limit okna. |
| `/status` | Zobrazí stav sezení: aktuální model, oprávnění, využití tokenů / kapacitu kontextu. |
| `/new` | Začne nový thread ve stejném CLI okně (reset kontextu bez nutnosti restartovat). |
| `/resume` | Obnoví předchozí uložené sezení — Codex nabídne výběr z historie. |
| `/fork` | Vytvoří kopii aktuální konverzace do nového threadu — hodí se na zkoušení alternativních přístupů bez ztráty původní konverzace. |
| `/mcp` | Vypíše dostupné MCP nástroje (pokud používáte MCP servery). |

---

### 11) AGENTS.md pro správu serveru (zatím přeskakujeme)

V domovském adresáři (`~`) vytvořit soubor `AGENTS.md`, který bude sloužit jako kontext pro codex při práci se serverem. Spustit `codex` v `~` a zadat následující prompt:

```
Vytvoř soubor ~/AGENTS.md s následujícím obsahem:

# Server -- správa a kontext

## Systém
- OS: Ubuntu 24
- Uživatel: <moje uživatelské jméno>
- Shell: bash

## Nainstalované nástroje
- nvm, Node.js (LTS), npm
- OpenAI Codex CLI (@openai/codex)
- git

## SSH
- Klíč: ~/.ssh/id_ed25519
- GitHub: připojený (ssh -T git@github.com)

## Git
- Repozitáře klonovány v ~/
- Konvence: popisné commit messages, commitovat logické celky

## Správa serveru
- Balíčky: apt update && apt upgrade
- Služby: systemctl start/stop/restart/status <služba>
- Firewall: ufw status / ufw allow <port>
- SSH konfigurace: /etc/ssh/sshd_config
- Logy: journalctl -u <služba> --since "1 hour ago"

## Pravidla pro agenta
- Před destruktivní operací (smazání, restart, změna konfigurace) se vždy zeptej.
- Po instalaci nového nástroje aktualizuj sekci "Nainstalované nástroje".
- Po naklonování nového repozitáře aktualizuj sekci "Git".
- Příkazy vyžadující sudo spouštěj s sudo, ne jako root.
```
