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

Administration af eksisterende eksterne brugere er beskrevet i brugermanualen under [Brugere](../user_manual/brugere.html).


## Områder og brugere via kommandolinje (ikke anbefalet)

Det er muligt at oprette områder og brugere direkte fra kommandolinjen på serveren med interaktive prompts — bemærk, at områder kaldes *tenants* i task-menuen:

```bash
task tenant:add   # spørger om tenant-id, titel og beskrivelse
task user:add     # spørger om e-mail, kodeord, rolle og tenant
```

Kommandolinjen er praktisk til den allerførste administrator og til testmiljøer, men til daglig brugeroprettelse anbefales Single Sign-On. Se også [Drift](drift.html) og README'ens [cookbook](https://github.com/os2display/os2display-docker-server#cookbook).
