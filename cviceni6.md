# VI. Nasazení aplikace -- Docker na VPS

## Prerekvizity

> Pro toto cvičení potřebujete:
> - Funkční aplikaci (frontend + backend) v GitHub repozitáři z předchozích cvičení
> - Přidělený Hetzner VPS s Ubuntu a subdoménu `*.aibr.cz` -- IP adresy a subdomény najdete ve sdíleném [Google Sheetu](https://docs.google.com/spreadsheets/d/1vZOATUBXVZJbRnIsglXyWlC8TZbvt5mJeXB8dqAQ5-0/edit?gid=0#gid=0)

---

## Konfigurace serveru

### 1) Přihlášení na server

Připojte se ke svému VPS přes SSH. IP adresu najdete ve sdíleném [Google Sheetu](https://docs.google.com/spreadsheets/d/1vZOATUBXVZJbRnIsglXyWlC8TZbvt5mJeXB8dqAQ5-0/edit?gid=0#gid=0).

```bash
ssh root@<IP_ADRESA>
```

> **Poznámka:** Pracujeme přímo jako `root`, což se na skutečných serverech nedělá -- normálně se vytváří běžný uživatel a `root` přístup se zakazuje. Pro náš kurz to ale zjednodušuje práci, tak to necháme.

### 2) Základní aktualizace systému

```bash
apt update && apt upgrade -y
```

### 3) Instalace Gitu

Na Ubuntu je git obvykle předinstalovaný. Ověřte a případně nainstalujte:

```bash
git --version
# pokud chybí:
apt install git -y
```

### 4) SSH klíč pro GitHub

Vygenerujte SSH klíč na serveru a přidejte ho na GitHub -- stejný postup jako v [cvičení 1](cviceni1.md), tentokrát na Linuxovém serveru:

```bash
# vygenerování SSH klíče
ssh-keygen -t ed25519 -C "mail@domena.cz" -f ~/.ssh/id_ed25519

# spuštění SSH agenta
eval "$(ssh-agent -s)"

# přidání klíče do agenta
ssh-add ~/.ssh/id_ed25519
```

Zkopírujte veřejný klíč a vložte ho na GitHub (Settings -> SSH and GPG keys -> New SSH key):

```bash
cat ~/.ssh/id_ed25519.pub
```

Ověřte připojení:

```bash
ssh -T git@github.com
```

> **Poznámka:** Na běžném serveru se SSH klíč s push přístupem do repozitáře nedává -- nasazení řeší CI/CD (např. GitHub Actions), které po pushnutí do větve automaticky nasadí novou verzi. My klonujeme přímo na server, abychom mohli pohodlně pracovat s Codexem.

### 5) Instalace Node.js, nvm, npm

```bash
# instalace nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# načtení nvm do aktuální session
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# instalace Node.js LTS
nvm install --lts

# ověření
node -v && npm -v
```

### 6) Instalace OpenAI Codex a přihlášení

```bash
npm i -g @openai/codex
codex --version
```

Přihlašte se ke svému OpenAI účtu:

```bash
codex login --device-auth
```

> **Poznámka:** Na ostrých serverech se AI coding agenti nespouští -- vývoj a automatizace probíhají lokálně nebo v CI/CD. Codex na serveru používáme čistě pro výuku, protože nám umožňuje konfigurovat server a nasazovat aplikaci interaktivně pomocí promptů.

---

## AGENTS.md pro server

### 7) Vytvoření AGENTS.md v domovském adresáři

Spusťte Codex v domovském adresáři a nechte ho vytvořit kontextový soubor pro správu serveru:

```bash
cd ~
codex
```

Zadejte následující prompt:

```
V domovském adresáři vytvoř soubor AGENTS.md s tímto obsahem:

# Server Management -- AGENTS.md

## Systém
- OS: Ubuntu 24.04 LTS
- Uživatel: root (výukové prostředí)
- Shell: bash
- Hostname: <moje subdoména>.aibr.cz

## Nainstalované nástroje
- git
- nvm, Node.js (LTS), npm
- OpenAI Codex CLI (@openai/codex)

## SSH
- Klíč: ~/.ssh/id_ed25519
- GitHub: připojený (ssh -T git@github.com)

## Struktura serveru
- ~/            -- domovský adresář, AGENTS.md
- ~/apps/       -- nasazené aplikace (každá ve vlastním adresáři)

## Nasazené aplikace
(zatím žádné -- po nasazení sem přidej záznam s názvem, cestou, subdoménou a porty)

## Docker
(zatím nenainstalován -- po instalaci aktualizuj tuto sekci)

## Traefik (reverse proxy)
(zatím nenainstalován -- po instalaci aktualizuj tuto sekci)

## Pravidla pro agenta
1. Před destruktivní operací (smazání souborů, restart služby, změna konfigurace) se zeptej uživatele.
2. Po instalaci nového nástroje nebo služby aktualizuj sekci "Nainstalované nástroje".
3. Po naklonování nebo nasazení aplikace aktualizuj sekci "Nasazené aplikace".
4. Po instalaci a konfiguraci Dockeru aktualizuj sekci "Docker" (verze, cesta k docker-compose, běžící kontejnery).
5. Po konfiguraci Traefiku aktualizuj sekci "Traefik" (cesta ke konfiguraci, směrovací pravidla).
6. Používej docker compose (ne docker-compose) -- novější syntaxe.
7. Při práci s .env soubory nezobrazuj jejich obsah -- obsahují citlivé údaje.
8. Před úpravou konfiguračních souborů vytvoř zálohu originálu.
9. Po každé změně ověř, že služby běží (docker ps, curl, kontrola logů).
10. Při chybě nejdřív zkontroluj logy (docker logs, journalctl) a teprve pak navrhuj opravu.
11. Vždy vysvětli, co děláš a proč -- uživatel se učí.
```

> Tento AGENTS.md funguje jako stálý kontext pro Codex. Kdykoli ho spustíte v domovském adresáři, automaticky ho načte a bude vědět, co na serveru je. Důležité je, že agent si soubor sám průběžně aktualizuje -- po každé instalaci nebo konfiguraci přidá záznam, takže slouží jako živá dokumentace serveru.

---

## Klonování repozitáře

### 8) Naklonování aplikace na server

Vytvořte adresář pro aplikace a naklonujte svůj repozitář. V Codexu (spuštěném v `~`) zadejte prompt:

```
Vytvoř adresář ~/apps/ (pokud neexistuje) a naklonuj do něj repozitář
git@github.com:<uzivatel>/<repozitar>.git přes SSH.
Po naklonování ověř, že složka obsahuje očekávanou strukturu (frontend/, backend/).
Aktualizuj AGENTS.md -- přidej záznam o naklonované aplikaci do sekce
"Nasazené aplikace".
```

> Nahraďte `<uzivatel>/<repozitar>` skutečným názvem svého GitHub repozitáře.

---

## Nasazení aplikace

### 9) Instalace Dockeru a příprava Docker konfigurace

Přesuňte se do adresáře aplikace a spusťte Codex:

```bash
cd ~/apps/<repozitar>
codex
```

Zadejte prompt, který nainstaluje Docker a připraví kompletní konfiguraci:

```
Zkontroluj, jestli je na serveru nainstalovaný Docker a Docker Compose.
Pokud ne, nainstaluj Docker z oficiálního repozitáře (ne přes snap)
a ověř instalaci (docker --version, docker compose version).
Aktualizuj ~/AGENTS.md -- přidej Docker do nainstalovaných nástrojů.

Poté pro tuto aplikaci vytvoř Docker konfiguraci:

1. Dockerfile pro backend (v backend/) -- multi-stage build:
   - build stage: nainstaluj závislosti a sestav aplikaci
   - production stage: kopíruj jen build artefakty, spusť na příslušném portu
2. Dockerfile pro frontend (v frontend/) -- multi-stage build:
   - build stage: nainstaluj závislosti a sestav produkční build
   - production stage: servíruj přes nginx
3. docker-compose.yml v kořeni projektu:
   - služba backend s příslušným portem
   - služba frontend s příslušným portem
   - síť pro komunikaci mezi kontejnery
   - restart: always pro obě služby
   - environment variables z .env souboru
4. Vytvoř .env.example soubor se všemi potřebnými proměnnými prostředí
   (API klíče, databázové připojení apod.) -- s prázdnými hodnotami
   a komentáři co kam patří.
5. Přidej .env do .gitignore (pokud tam ještě není).

Zatím NESPOUŠTĚJ kontejnery -- jen připrav konfiguraci.
```

> Codex zná tech stack vašeho projektu z AGENTS.md a ze zdrojového kódu -- nemusíte ho specifikovat v promptu.

Po dokončení vytvořte `.env` soubor s reálnými hodnotami ručně:

```bash
cp .env.example .env
nano .env
```

> Proměnné prostředí (API klíče, hesla k databázi apod.) vyplňujte ručně -- je lepší je nezadávat do promptu pro AI agenta.

### 10) Konfigurace Traefiku a HTTPS

Traefik je reverse proxy -- prostředník, který přijímá požadavky z internetu a posílá je správnému kontejneru. Zároveň automaticky zajistí HTTPS certifikát přes Let's Encrypt, takže se o šifrování nemusíte starat ručně. V Codexu zadejte:

```
Přidej do docker-compose.yml službu Traefik jako reverse proxy:

1. Traefik konfigurace:
   - entrypoints: web (port 80) a websecure (port 443)
   - automatický redirect HTTP -> HTTPS
   - Let's Encrypt certifikáty (ACME, HTTP challenge)
   - certifikáty ukládej do souboru /letsencrypt/acme.json (volume)
   - provider: docker (automatická detekce služeb přes labels)

2. Uprav služby frontend a backend:
   - přidej Traefik labels pro směrování
   - frontend dostupný na: <moje-subdomena>.aibr.cz
   - backend dostupný na: <moje-subdomena>.aibr.cz/api
     (nebo api.<moje-subdomena>.aibr.cz -- zvol vhodný přístup dle projektu)
   - odeber mapování portů na host (porty budou dostupné jen přes Traefik)

3. Ujisti se, že:
   - všechny služby mají restart: always (poběží i po restartu serveru)
   - Docker je nastavený na automatický start (systemctl enable docker)
   - Traefik síť je sdílená mezi všemi službami

Subdoména: [DOPLŇTE svou subdoménu, např. novak.aibr.cz]
Email pro Let's Encrypt: [DOPLŇTE svůj email]

Aktualizuj ~/AGENTS.md -- přidej Traefik konfiguraci a směrovací pravidla.
```

### 11) Spuštění a ověření nasazení

Nechte Codex vše spustit a ověřit:

```
Spusť všechny Docker kontejnery pomocí docker compose up -d.
Poté ověř:
1. Všechny kontejnery běží (docker ps)
2. Žádný kontejner se nerestartuje v cyklu (zkontroluj uptime)
3. Logy neobsahují chyby (docker logs pro každou službu)
4. Frontend odpovídá na HTTPS (curl -I https://<moje-subdomena>.aibr.cz)
5. Backend API odpovídá (curl https://<moje-subdomena>.aibr.cz/api)
6. HTTP automaticky přesměrovává na HTTPS

Pokud něco nefunguje, zjisti příčinu z logů a oprav.
```

### 12) Kontrola dostupnosti na subdoméně

Zadejte Codexu svou subdoménu k finální kontrole:

```
Ověř, že aplikace běží na adrese https://<moje-subdomena>.aibr.cz:
- frontend se načte a vrací HTTP 200
- backend API odpovídá
- HTTPS certifikát je platný
- aplikace přežije restart (docker compose down && docker compose up -d)

Pokud cokoliv nefunguje, zjisti proč (logy, DNS, firewall, konfigurace)
a oprav to. Poté znovu ověř.
```

### 13) Commit a push změn

Jakmile vše běží a je ověřené, uložte Docker konfiguraci do repozitáře:

```
Zkontroluj, že .env soubor NENÍ v gitu (je v .gitignore).
Poté přidej do gitu všechny nové a změněné soubory týkající se nasazení:
- Dockerfile (frontend i backend)
- docker-compose.yml
- nginx konfigurace (pokud existuje)
- .dockerignore soubory (pokud existují)
- aktualizované AGENTS.md soubory
- .env.example (šablona bez skutečných hodnot -- pokud neexistuje, vytvoř ji)

Commitni s popisnou commit message a pushni na GitHub.
```

> **Poznámka:** Na reálných projektech se z produkčního serveru nepushuje. Změny jdou opačným směrem -- vývojář pushne do repozitáře a CI/CD pipeline automaticky nasadí na server. Ruční push ze serveru děláme jen v rámci kurzu, abychom si nechali Docker konfiguraci uloženou v gitu.

---

## Shrnutí

Po dokončení tohoto cvičení byste měli mít:

1. Nakonfigurovaný VPS s nainstalovaným Gitem, Node.js, Codexem a Dockerem
2. AGENTS.md na serveru jako živou dokumentaci nainstalovaných nástrojů a služeb
3. Aplikaci nasazenou v Docker kontejnerech (frontend + backend)
4. Traefik jako reverse proxy s automatickým HTTPS (Let's Encrypt)
5. Aplikaci dostupnou na vaší subdoméně `<jmeno>.aibr.cz`
6. Docker konfiguraci uloženou v Git repozitáři
7. Služby nastavené tak, aby přežily restart serveru
