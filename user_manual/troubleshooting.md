---
title: Troubleshooting
layout: default
parent: Sådan bruger du OS2display
nav_order: 13
has_children: false
---

# Troubleshooting

### Fejl ved indlæsning af klient, før der er oprettet netværksforbindelse 

Visse enheder kan have problemer med at indlæse klienten korrekt, hvis netværksforbindelsen først bliver oprettet efter indlæsning af klienten er påbegyndt. 
I Aarhus anvendes et fix, hvor alle maskiner starter op med en online check-side, som først videresender til den korrekte URL, når der er opnået forbindelse til netværket. 

[History for public/client/online-check/index.html - os2display/os2display · GitHub](https://github.com/os2display/os2display/commits/main/public/client/online-check/index.html)

### Sådan finder du versionsnummer på OS2display  
 
Du kan undersøge hvilken version af OS2display som er installeret, ved at kigge i filen release.json som er tilgængelig via browseren. 
 
Adressen er URL til admin-grænsefladen efterfulgt af /release.json 
 
For København er det f. eks.   
https://kkos2display.dk/admin/release.json 

### Skærmen viser fallback-billedet

Fallback-billedet vises, når skærmen ikke har noget indhold at vise lige nu. Det skyldes typisk en af disse ting:

- Skærmen har ingen spillelister, eller spillelisterne er tomme.
- Spillelister eller slides er ikke udgivet lige nu, eller en planlægning på en spilleliste forhindrer, at de vises.
- En kampagne tilsidesætter skærmens normale spillelister.
- Skærmen kan ikke hente indhold fra serveren (netværk, login eller serverfejl).

> **Ændringer slår ikke igennem med det samme.** Skærmen henter som standard nyt indhold hvert 10. minut.
> Vent, eller genindlæs skærmens browser, før du konkluderer, at en rettelse ikke virkede.

> **Viser skærmen en fejlkode** (ER101–ER106 eller ER201) i stedet for fallback-billedet, er det et andet problem
> (login/token, udgivelse eller skabelon). Notér koden, og kontakt support.

#### Trin 1: Tjek i Admin (redaktør)

1. **Skærmstatus** (vises kun, hvis det er slået til i installationen). Find skærmen i skærmlisten, og se kolonnen
   "Status":
   - "+ Tilkobl": Skærmen er ikke tilkoblet en enhed.
   - Grønt flueben: Enheden er tilkoblet og kører seneste version.
   - Gul cirkel: Enheden kører ikke seneste version.
   - Rød trekant: Enheden har ikke kontaktet serveren den seneste time, eller dens login er udløbet. Gå til trin 2.
2. **Skærmens spillelister.** Tjek, at mindst én spilleliste er tilknyttet en region i skærmens layout.
3. **For hver spilleliste på skærmen:**
   - Tjek, at spillelisten indeholder mindst ét slide.
   - Tjek, at spillelisten er udgivet nu (udgivelse fra/til).
   - Tjek, at der ikke er en planlægning (Planlægning) på spillelisten, som forhindrer den i at blive vist nu.
   - For hvert slide i spillelisten: Tjek, at slidet er udgivet nu (udgivelse fra/til).
4. **Kampagner.** En kampagne har forrang for alle andre spillelister og fylder hele skærmen. Tjek, om der er en aktiv
   kampagne på skærmen eller på en gruppe, som skærmen er medlem af. Hvis ja, så gennemgå kampagnen på samme måde som
   spillelisterne i punkt 3.
5. **Vent.** Har du rettet noget, så vent op til 10 minutter, eller få skærmens browser genindlæst.

#### Trin 2: Tjek på skærmen (tekniker)

Kræver adgang til skærmens browser.

1. Åbn browserens udviklerværktøjer, og gå til fanen Netværk.
2. Genindlæs siden, så skærmen henter alt indhold forfra.
3. Kig efter fejlede kald (røde linjer) til `https://[HOST]/v2/...`:
   - **401 eller 403:** Skærmens login er udløbet eller afvist. Skærmen skal tilkobles igen.
   - **429:** Serveren begrænser antallet af kald. Det kan ske, når mange skærme deler samme IP-adresse.
   - **500–504:** Fejl på serveren.
   - **Intet svar:** Netværksproblem på enheden.
4. Find kald til `https://[HOST]/v2/playlists/[PLAYLIST_ID]/slides`, og tjek, at der er slides i svaret.
   - Er svaret tomt, er det et indholdsproblem. Gå tilbage til trin 1.
   - Kommer der slet ingen af disse kald, har skærmen ingen spillelister (trin 1), eller den henter ikke indhold.
5. Kan siden ikke genindlæses: Tilføj et slide til en af skærmens spillelister i Admin, og vent op til 10 minutter.
   Kommer der stadig ingen kald, henter skærmen ikke indhold. Tjek netværk og login på enheden.
6. Tjek, at enhedens ur og tidszone er korrekte. Forkert tid kan give forkert udgivelse og planlægning.

**Tip:** Har du ikke adgang til den fysiske skærm, kan du bruge forhåndsvisning af skærmen i Admin (hvis det er slået
til). Vises indholdet dér, men ikke på skærmen, ligger problemet på enheden.

#### Kontakt support

Hvis problemet ikke er løst, så send:

- Skærmens navn og tidspunktet for fejlen.
- Hvad du har tjekket i trin 1 og 2.
- Et skærmbillede af fanen Netværk med de fejlede kald.
- Eventuel fejlkode på skærmen.
