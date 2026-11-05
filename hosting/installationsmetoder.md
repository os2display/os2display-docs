---
title: Alternative installationsmetoder
layout: default
parent: For driftsleverandører
nav_order: 5
has_children: false
---

# Installationsmetoder og krav

Fra version 3.0 er OS2display én samlet Symfony-applikation, [os2display/os2display](https://github.com/os2display/os2display), der indeholder API'et, administrations-GUI på `/admin` og skærmklienten på `/client`. Ingen admin- eller klientapplikationer skal hostes seperat.


## Fire måder at installere på

### 1. os2display-docker-server (anbefalet)

[os2display-docker-server](https://github.com/os2display/os2display-docker-server) er et færdigt docker compose-setup med de publicerede images, en valgfri medfølgende MariaDB og Traefik samt tasks til at installere, opdatere og drifte det samlede setup. Denne metode er beskrevet trin for trin på [installationssiden](./).

### 2. Publicerede images i eget container-setup

Kør de to images i jeres eget container-setup:

- `ghcr.io/os2display/os2display`: PHP-FPM-applikationen.
- `ghcr.io/os2display/os2display-nginx`: nginx, der serverer statiske filer og videresender PHP-requests.

Images tagges med release-versionen (fx `3.0.0`); `latest` er den nyeste 3.x-release og `develop` udviklingsbranchen. Prerelease-tags (fx `3.0.0-RC1`) flytter ikke `latest`.

### 3. Release-tarball

Hver [release](https://github.com/os2display/os2display/releases) har en færdigbygget `os2display-<version>.tar.gz` med PHP-afhængigheder og den byggede frontend, til et klassisk PHP + nginx-setup.

### 4. Checked-out repository

Klon repositoriet og byg og kør selv koden.

## Krav

- PHP 8.4 (fpm) med de extensions, der er angivet i `composer.json`, herunder **imagick**.
- MariaDB (10.11 eller 11.4).
- En cache-backend: Redis som standard, alternativt memcached, databasen, filsystemet eller APCu (`CACHE_ADAPTER`).
- nginx (eller en anden webserver) foran PHP-FPM.
- Uden de publicerede images: en ImageMagick-sikkerhedspolitik, der kun tillader de anvendte billedformater, fx den medfølgende [`imagemagick-policy.xml`](https://github.com/os2display/os2display/blob/main/infrastructure/os2display/imagemagick-policy.xml). Uden den kan ondsindede uploads misbruge ImageMagicks SVG-, MVG- og URL-læsere.

De publicerede container-images indeholder alt undtagen databasen og cachen. Se [Hosting requirements](https://github.com/os2display/os2display#hosting-requirements) for detaljerne.
