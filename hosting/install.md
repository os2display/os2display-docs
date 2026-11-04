---
title: Installation
layout: default
parent: For driftsleverandører
nav_order: 2
has_children: false
---
# Installér OS2display på jeres egen server

Med den officielle installationspakke [os2display-docker-server](https://github.com/os2display/os2display-docker-server) får I en produktionsklar OS2display-installation op at køre på en enkelt Linux-server. Alt hvad I skal bruge, er:

- [Docker](https://docs.docker.com/engine/install/) med [Compose v2](https://docs.docker.com/compose/)
- [Task](https://taskfile.dev/)

## Fra nul til kørende infoskærme

Installationen er bygget som en interaktiv guide, der spørger om det nødvendige (domæne, Let's Encrypt-e-mail, admin-bruger) og selv genererer resten:

```bash
git clone https://github.com/os2display/os2display-docker-server.git
cd os2display-docker-server

task env:init     # 1. Opretter alle konfigurationsfiler og genererer secrets
task env:traefik  # 2. Opsætter HTTPS og dashboard-adgang
task install      # 3. Starter hele stakken, kører migreringer og opretter admin-bruger
```

Når `task install` er færdig, kører jeres løsning på `https://<jeres-domæne>/` med administrationen på `/admin/` og skærmklienten på `/client/`. HTTPS-certifikater udstedes automatisk via Let's Encrypt — eller I kan bruge jeres egne certifikater.

## Hvad får I med i pakken?

- **Hele OS2display v3 i ét image** — API, administration og skærmklient følges ad og opgraderes samlet.
- **Komplet infrastruktur** — Traefik som reverse proxy med automatisk HTTPS, MariaDB og Redis følger med, men kan slås fra, hvis I har egen database eller proxy i forvejen.
- **Fornuftige produktionsstandarder** — PHP-, nginx- og cache-indstillinger er tunet til produktionsdrift fra start. Alle konfigurationsmuligheder er dokumenteret direkte i `.env`-filerne.
- **Sikker netværksarkitektur** — applikation, database og proxy er isoleret på interne Docker-netværk; kun det nødvendige er eksponeret mod internettet.

## Vil I bare prøve det af først?

Kør hele stakken lokalt på jeres egen maskine — uden offentligt domæne og med selvsigneret certifikat — med `task dev:install`. Se [Local dev quick start](https://github.com/os2display/os2display-docker-server#local-dev-quick-start) i README'en.

---

**Klar til at gå i gang?** Alle detaljer om forudsætninger, konfiguration, netværkstopologi og drift finder du i den fulde dokumentation:

👉 **[README for os2display-docker-server](https://github.com/os2display/os2display-docker-server#readme)**
