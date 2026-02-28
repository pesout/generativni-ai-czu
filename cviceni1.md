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

### 2) Připojení na server, SSH klíč pro GitHub

PowerShell/Terminál:

```bash
# připojení na server
ssh user@example.com

# vygenerování SSH klíče
ssh-keygen -t ed25519 -C "mail@domena.cz" -f ~/.ssh/id_ed25519

# spuštění SSH agenta
eval "$(ssh-agent -s)"

# přidání klíče do agenta
ssh-add ~/.ssh/id_ed25519
```

### 3) Vložit SSH klíč na GitHub

- zavolat příkaz a zkopírovat výstup -- `cat ~/.ssh/id_ed25519.pub`
- GitHub -- Settings → SSH and GPG keys → New SSH key → vložit → Add SSH key
- ověřit na serveru -- `ssh -T git@github.com`

### 4) Instalace node, nvm, npm

- Node Version Manager -- [https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating](https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating)
- instalace Node.js + npm -- `nvm install --lts`
- ověření -- `node -v && npm -v`

### 5) Instalace OpenAI Codex

- instalace -- `npm i -g @openai/codex`
- ověření -- `codex --version`

### 6) Lokální instalace VS Code, připojení vzdáleného serveru

- [https://code.visualstudio.com](https://code.visualstudio.com/)
- instalace doplňku -- Remote SSH (Microsoft)
- `Ctrl+Shift+P` → Remote-SSH: Add New SSH Host → zadat `ssh user@example.com`
- `Ctrl+Shift+P` → Remote-SSH: Connect to Host... → vybrat hosta → zadat heslo

### 7) Vyklonovat repozitář

- repozitář → Code → SSH
- na serveru -- `git clone git@github.com:xxxx/xxxx.git`
- přejít do projektu -- `cd <složka>`

### 8) Demonstrace správného nastavení

- propojit s OpenAI pomocí kódu -- `codex login --device-auth`
- spustit codex ve složce s projektem -- `codex`
- povolit změny bez schvalování
- vybrat model pomocí `/model`
- založit AGENTS.md pomocí `/init`
- zadat prompt: *Create template (headings) in AGENTS.md file, add project description to README.md. Commit and push all changes.*
- zkontrolovat výsledek na GitHubu

### 9) AGENTS.md pro správu serveru

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
