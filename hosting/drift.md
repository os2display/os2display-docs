---
title: Drift
layout: default
parent: For driftsleverandører
nav_order: 3
has_children: false
---

# Drift af OS2display

Installationen er kun begyndelsen. [os2display-docker-server](https://github.com/os2display/os2display-docker-server) dækker hele driftslivscyklussen med korte, veldokumenterede `task`-kommandoer, og README'ens omfattende [cookbook](https://github.com/os2display/os2display-docker-server#cookbook) har opskrifter på alle de daglige opgaver. Her er et overblik over den væsentligste funktionalitet for systemadministratorer.

## Opgradering til nye versioner

Opgradering er tre kommandoer — tag altid en backup først, da databasemigreringer ikke kan rulles tilbage:

```bash
task db:backup   # altid før en opdatering
task update      # henter nye images, stopper web-laget, migrerer og genstarter
task env:diff    # viser om den nye version har tilføjet nye konfigurationsnøgler
```

Kør `task templates:install` bagefter, hvis den nye version indeholder nye slide-skabeloner.

## Backup og gendannelse

`task db:backup` lægger en komprimeret dump i `backup/`-mappen med tidsstempel i filnavnet (fx `20260505T140723Z.sql.gz`). Gendannelse sker ved at pipe dumpen tilbage i databasen — se [How do I restore from a backup?](https://github.com/os2display/os2display-docker-server#how-do-i-restore-from-a-backup) i README'en, og vær opmærksom på, at gendannelse overskriver data uden bekræftelse.

## Konfigurationsændringer

Al konfiguration ligger i `.env`-filer med dokumentation for hver indstilling. Når du har ændret en værdi, anvender `task up` ændringen — kun de berørte services genskabes, og kommandoen er en no-op, hvis intet er ændret, så den er altid sikker at køre.


## Logs og fejlsøgning

Alle services' logs kan følges og filtreres fra ét sted:

```bash
task logs                   # følg alle services
task logs S=os2display      # følg en enkelt service
task logs:since T=30m       # alt fra de sidste 30 minutter
task logs:errors            # fejl/kritiske linjer fra den sidste time
task logs:access            # Traefiks access-log, én kompakt linje pr. request
```

## Overvågning og performance

Indbyggede værktøjer giver indsigt i databasens og PHP's sundhedstilstand samt diskforbrug:

```bash
task db:metrics          # driftstællere: forbindelser, locks, buffer pool
task db:processes        # aktive forbindelser og deres queries
task php:opcache         # PHP OPcache-sundhedsrapport
task host:disk           # diskforbrug for media, backup og databasevolumes
task host:disk:tenants   # mediaforbrug pr. tenant, sorteret faldende
task cache:clear         # ryd applikationscachen
```

På en dedikeret server kan `task host:resources` desuden beregne og udskrive anbefalede ressourcegrænser (RAM/CPU) ud fra værtens faktiske kapacitet.

---

**Vil du vide mere?** Alle opskrifter — inkl. ekstern database, egen reverse proxy, tuning af PHP og nginx samt major-opgradering af MariaDB — findes i README'ens [cookbook](https://github.com/os2display/os2display-docker-server#cookbook).
