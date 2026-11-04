---
title: Logins, brugere og områder
layout: default
parent: For driftsleverandører
nav_order: 4
has_children: true
---

# Logins, brugere og områder

Via OS2display-brugergrænsefladen er det **ikke** muligt at oprette ordinære brugere. Brugere kan i stedet tilknyttes på tre måder:

1. **Medarbejdere og områder via Single Sign-On (Azure/OS2Faktor)** — den anbefalede metode
2. **Eksterne brugere via MitID** — til personer, der ikke er ansat i kommunen
3. **Kommandolinje på serveren** — muligt, men ikke anbefalet

## Medarbejdere og områder via Single Sign-On (anbefalet)

Best practice er at knytte medarbejdere til OS2display via Single Sign-On mod Azure eller OS2Faktor. Både medarbejdere og deres tilhørende områder styres via SSO-konfigurationen: Medarbejderne logger ind med deres organisationskonto, og både adgang og områdetilknytning følger automatisk organisationens brugerstyring.

Se opsætningsguiden: [SSO Setup](sso_setup/sso_setup.html).

## Eksterne brugere via MitID

Eksterne brugere er personer, der ikke er ansat i kommunen og derfor ikke kan tilknyttes via Medarbejder Single Sign-On — fx frivillige i et medborgerhus. De logger i stedet ind med deres MitID, og første gang de logger ind, skal de angive en aktiveringskode, som de har fået tilsendt.

### Opret aktiveringskode til ny ekstern bruger

1. Under **Aktiveringskoder** tryk på **Ny**.
2. Vælg mellem **ekstern bruger** og **ekstern brugeradministrator**. Begge får redaktør-adgang til området, men brugeradministratoren kan desuden tilføje nye aktiveringskoder og dermed give adgang til andre eksterne brugere.
3. En aktiveringskode genereres. Den giver adgang til ét område i OS2display. Koden er **aktiv i to døgn** fra udstedelse og skal manuelt videredistribueres til den person, som skal have ekstern adgang.
4. Den eksterne bruger logger ind i OS2display via MitID og indtaster den unikke kode — inden for to døgn fra oprettelsen af aktiveringskoden.

![Ekstern bruger logger ind første gang](../assets/ekstern-bruger-aktiveringskode.png)

Administration af eksisterende eksterne brugere er beskrevet i brugermanualen under [Brugere](../user_manual/brugere.html).

## Områder og brugere via kommandolinje (ikke anbefalet)

Det er muligt at oprette områder og brugere direkte fra kommandolinjen på serveren med interaktive prompts — bemærk, at områder kaldes *tenants* i task-menuen:

```bash
task tenant:add   # spørger om tenant-id, titel og beskrivelse
task user:add     # spørger om e-mail, kodeord, rolle og tenant
```

Kommandolinjen er praktisk til den allerførste administrator og til testmiljøer, men til daglig brugeroprettelse anbefales Single Sign-On. Se også [Drift](drift.html) og README'ens [cookbook](https://github.com/os2display/os2display-docker-server#cookbook).
