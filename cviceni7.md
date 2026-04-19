# VII. Úvod do n8n, instalace a první workflow

## Prerekvizity

> Pro toto cvičení potřebujete:
> - Funkční VPS s Dockerem a Traefikem z [cvičení 6](cviceni6.md)
> - Pokud máte aplikaci běžící na vaší subdoméně `<jmeno>.aibr.cz`, jste pravděpodobně připraveni

### 1) Rychlá kontrola prostředí z cvičení 6

Než začneme, ověřte si na serveru, že všechno z minula běží. Připojte se přes SSH a projděte tento checklist:

```bash
ssh root@<IP_ADRESA>
```

```bash
# Docker a Docker Compose
docker --version
docker compose version

# Běžící kontejnery (frontend, backend, traefik)
docker ps

# Aplikace odpovídá na HTTPS
curl -I https://<jmeno>.aibr.cz
```

Pokud něco nesedí, vraťte se k [cvičení 6](cviceni6.md) a opravte to předtím, než budete pokračovat.

---

## Motivace -- proč n8n

**n8n** je nástroj pro automatizace, které si skládáte v prohlížeči z hotových dílů. Místo psaní skriptu přetahujete uzly (nodes) na plátno a propojujete je -- jeden uzel stáhne email, druhý zavolá AI model, třetí zapíše výsledek do tabulky. Pro většinu běžných služeb (Gmail, Google Sheets, Slack, Notion, OpenAI a stovky dalších) je už integrace hotová, takže se nemusíte starat o přihlašování, knihovny ani detaily API.

Pro nás je n8n zajímavé hlavně ze dvou důvodů: rychle v něm postavíte reálnou automatizaci a snadno do ní zapojíte LLM.

### Příklady použití

- **Příchozí emaily -> strukturovaná data** -- nová faktura v emailu se automaticky přepíše do Google Sheets, PDF se uloží na Drive
- **Formulář -> CRM + notifikace** -- poptávka z webu se zapíše do tabulky a majitel dostane zprávu na Telegram nebo Slack
- **AI zpracování obsahu** -- nová hlasovka, článek nebo přepis schůzky projde přes LLM (shrnutí, extrakce úkolů, tagy) a uloží se zpět
- **Denní reporty** -- každé ráno v 8:00 stáhni data z API, nech LLM napsat shrnutí a pošli ho na email
- **Propojení služeb** -- nová událost v Kalendáři -> úkol v Notion; nový zákazník ve Stripe -> řádek v tabulce a uvítací email

Dnes n8n jen nainstalujeme a vyzkoušíme si, že všechno funguje. Složitější workflow s LLM si ukážeme příště.

---

## Pohodlná editace souborů na serveru -- VS Code Remote-SSH

V [cvičení 6](cviceni6.md) jsme soubory na serveru upravovali v terminálu přes `nano`. Od teď to budeme dělat pohodlněji, tedy přímo z VS Code. Plugin **Remote-SSH** umožňuje otevřít vzdálený adresář tak, jako by byl lokální. Uvidíte strukturu projektu v postranním panelu, soubory editujete normálně ve VS Code a v témže okně máte i terminál, který běží na serveru.

> **Poznámka:** Pokud používáte **Cursor** nebo **Antigravity**, postup je stejný -- oba editory jsou forky VS Code a mají identický plugin marketplace.

### 2) Instalace rozšíření Remote-SSH

1. Otevřete VS Code (lokálně na svém počítači)
2. V levém panelu klikněte na ikonu **Extensions** (čtyři čtverečky), nebo stiskněte `Ctrl+Shift+X`
3. Vyhledejte **Remote - SSH** (vydavatel Microsoft) a klikněte na **Install**

### 3) Připojení k serveru

1. Stiskněte `F1` (nebo `Ctrl+Shift+P`) -- otevře se Command Palette
2. Začněte psát `Remote-SSH: Connect to Host...` a potvrďte
3. Vyberte **+ Add New SSH Host...**
4. Zadejte příkaz pro připojení:

   ```
   ssh root@<IP_ADRESA>
   ```

5. Vyberte konfigurační soubor (typicky `~/.ssh/config` ve vašem domovském adresáři) -- VS Code do něj zápis uloží a příště stačí hosta jen vybrat ze seznamu
6. Znovu otevřete Command Palette -> `Remote-SSH: Connect to Host...` -> vyberte svůj server
7. Otevře se nové okno VS Code. Při prvním připojení se VS Code zeptá na typ platformy -- vyberte **Linux**. Pokud máte SSH klíč s passphrase nebo používáte heslo, VS Code si je vyžádá.
8. V levém dolním rohu uvidíte zelený indikátor `SSH: <IP_ADRESA>` -- jste připojení

---

## Instalace n8n v Dockeru

n8n nainstalujeme **vedle** vaší aplikace -- do samostatného adresáře a s vlastním `docker-compose.yml`. Výhodou je, že se n8n neplete s kódem vaší aplikace (není ve vašem gitu) a můžete ho kdykoli spustit nebo zastavit nezávisle. Zároveň se připojí do stejné Docker sítě jako Traefik, takže o HTTPS certifikát i směrování na sub-subdoménu `n8n.<jmeno>.aibr.cz` se postará Traefik úplně sám.

### 4) Příprava adresáře a konfigurace pomocí Codexu

V terminálu VS Code (připojeném k serveru) vytvořte adresář pro n8n a spusťte v něm Codex:

```bash
mkdir -p ~/n8n
cd ~/n8n
codex
```

Zadejte následující prompt:

```
V aktuálním adresáři (~/n8n) připrav instalaci n8n jako samostatné Docker služby,
která se napojí na existující Traefik z ~/apps/<repozitar>.

Nejdřív si zjisti název externí Docker sítě, kterou používá Traefik
(podívej se do ~/apps/<repozitar>/docker-compose.yml) -- tu stejnou síť
použijeme i zde jako external network.

Vytvoř:

1. docker-compose.yml se službou n8n:
   - image: docker.n8n.io/n8nio/n8n (latest stable)
   - container_name: n8n
   - restart: always
   - volume: pojmenovaný volume n8n_data -> /home/node/.n8n
   - připojení do existující Traefik sítě (external: true)
   - NEmapuj port 5678 na host (bude dostupný jen přes Traefik)

2. Environment variables ve službě:
   - N8N_HOST=n8n.<jmeno>.aibr.cz
   - N8N_PROTOCOL=https
   - N8N_PORT=5678
   - WEBHOOK_URL=https://n8n.<jmeno>.aibr.cz/
   - GENERIC_TIMEZONE=Europe/Prague
   - TZ=Europe/Prague
   - N8N_SECURE_COOKIE=true
   - N8N_BASIC_AUTH_ACTIVE=true
   - N8N_BASIC_AUTH_USER a N8N_BASIC_AUTH_PASSWORD načítej z .env
     (nepiš hodnoty přímo do docker-compose.yml)

3. Traefik labels pro směrování a HTTPS:
   - traefik.enable=true
   - router pro Host `n8n.<jmeno>.aibr.cz` na entrypoint websecure
   - TLS přes stejný Let's Encrypt resolver, který už používá frontend/backend
     (ověř si jeho název v ~/apps/<repozitar>/docker-compose.yml)
   - traefik.http.services.n8n.loadbalancer.server.port=5678

4. .env soubor (reálné hodnoty) se dvěma proměnnými:
   - N8N_BASIC_AUTH_USER
   - N8N_BASIC_AUTH_PASSWORD
   Zatím nech obě hodnoty prázdné -- doplním je ručně ve VS Code.

5. Aktualizuj ~/AGENTS.md:
   - přidej do "Nasazené aplikace" záznam o n8n
     (subdoména, cesta k docker-compose, volume, způsob přihlášení)

Zatím NESPOUŠTĚJ kontejner -- jen priprav konfiguraci, ať si ji můžu projít.

Subdoména aplikace: [DOPLŇTE svou subdoménu, např. novak.aibr.cz]
```

> Nahraďte `<jmeno>` svou skutečnou subdoménou (stejnou jako u aplikace z cvičení 6) a `<repozitar>` názvem adresáře s aplikací.

### 5) Vyplnění přihlašovacích údajů

V postranním panelu VS Code otevřete soubor `~/n8n/.env` a doplňte uživatelské jméno a heslo, kterým se budete do n8n přihlašovat:

```
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=<silne-heslo>
```

Uložte (`Ctrl+S`).

> Heslo si bezpečně uložte (např. do správce hesel) -- budete ho potřebovat při každém přihlášení.

### 6) Spuštění a ověření

V Codexu (stále v `~/n8n`) zadejte:

```
Spusť n8n pomocí docker compose up -d v aktuálním adresáři. Poté ověř:

1. Kontejner n8n běží a nerestartuje se v cyklu (docker ps, sloupec STATUS)
2. Logy n8n neobsahují chyby (docker logs n8n)
3. Traefik vidí nový router pro n8n (docker logs <traefik-kontejner> | grep -i n8n)
4. https://n8n.<jmeno>.aibr.cz odpovídá (curl -I) -- očekáváme
   HTTP 401 (Basic Auth) nebo 200, rozhodně ne 404 ani 502
5. HTTPS certifikát je platný (vydaný Let's Encrypt pro n8n.<jmeno>.aibr.cz)

Pokud cokoliv nefunguje, zjisti příčinu z logů (Traefik i n8n) a oprav.
```

> První vydání certifikátu od Let's Encrypt může trvat i minutu. Pokud vidíte chybu certifikátu, chvíli počkejte a zkuste znovu.

---

## První workflow v n8n

### 7) Přihlášení do n8n

Otevřete v prohlížeči:

```
https://n8n.<jmeno>.aibr.cz
```

Prohlížeč se nejdřív zeptá na Basic Auth -- zadejte uživatelské jméno a heslo z `.env`. Potom vás n8n provede krátkým úvodním setupem (jméno vlastníka instance, email, heslo k účtu v rámci n8n). Vyplňte a pokračujte až na hlavní obrazovku.

### 8) Vytvoření prvního workflow

Cílem je jen ověřit, že se workflow dá sestavit a spustit. Složitější věci s LLM a integracemi si ukážeme příště.

1. V levém menu klikněte na **Workflows** -> **Create Workflow** (vpravo nahoře). Otevře se prázdné plátno.
2. Uprostřed plátna klikněte na **Add first step**. Otevře se postranní panel se seznamem triggerů.
3. Vyberte **Trigger manually** (v n8n se tento uzel zobrazuje jako *When clicking 'Execute workflow'*). Uzel se přidá na plátno.
4. Najeďte myší na pravý okraj triggeru -- objeví se **+**. Klikněte na něj a v postranním panelu vyhledejte a vyberte **Edit Fields (Set)**.
5. V konfiguraci uzlu **Edit Fields**:
   - **Mode** nechte na **Manual Mapping**
   - V sekci **Fields to Set** klikněte na **Add Field** a vyplňte:
     - **Name:** `zprava`
     - **Type:** `String`
     - **Value:** `Ahoj z n8n!`
   - Panel uzlu zavřete křížkem vpravo nahoře
6. Vpravo nahoře klikněte na **Save** (nebo `Ctrl+S`) a dejte workflow libovolný název.
7. Dole uprostřed plátna klikněte na **Execute workflow**. Oba uzly by se měly orámovat zeleně.
8. Klikněte na uzel **Edit Fields** -- v pravé části obrazovky (panel **OUTPUT**) uvidíte výstup:

   ```json
   [
     {
       "zprava": "Ahoj z n8n!"
     }
   ]
   ```

Pokud workflow doběhlo a výstup sedí, máte n8n připravené pro příští cvičení.

> **Tip:** Každý uzel jde spouštět samostatně -- při otevřeném uzlu kliknete na **Execute step** a n8n spustí workflow jen po tento bod. To se hodí při ladění delších workflow, která budeme dělat příště.

---

## Shrnutí

Po dokončení tohoto cvičení byste měli mít:

1. VS Code připojený k serveru přes Remote-SSH -- pohodlná editace souborů i terminál v jednom okně
2. n8n běžící v samostatném adresáři `~/n8n` jako nezávislá Docker služba vedle vaší aplikace
3. n8n dostupné přes HTTPS na `n8n.<jmeno>.aibr.cz` díky sdílené Traefik síti (bez ručního řešení certifikátu)
4. Perzistentní volume `n8n_data`, takže workflow a credentials přežijí restart kontejneru
5. Přihlašovací údaje uložené v `~/n8n/.env`
6. Aktualizovaný `~/AGENTS.md` se záznamem o n8n
7. Hotové první workflow -- příští týden navážeme reálnými automatizacemi s LLM
