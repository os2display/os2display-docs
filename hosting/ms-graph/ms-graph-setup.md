# Vejledning: Opsætning af Microsoft Graph til OS2display

Denne vejledning beskriver, hvordan en systemadministrator konfigurerer en Microsoft Entra-app, så OS2display kan vise kalenderbookinger for mødelokaler. Den gennemgår også, hvordan app-oplysningerne indtastes i OS2display, og hvordan lokaler kan afgrænses med lokalelister.

## Overblik

OS2display bruger en app-registrering i Microsoft Entra ID til at hente data fra Microsoft Graph. Integrationen bruger **app-only authentication** med client credentials. Der er derfor ikke en bestemt bruger, som skal være logget ind, når kalenderdata hentes.

Appen bruges til at:

- hente lokalelister fra Microsoft Graph
- hente lokaler, enten alle tenantens lokaler eller lokaler fra valgte lokalelister
- hente kalenderbookinger for de lokaler, der vælges til en OS2display-feed

Der bruges ikke miljøvariabler til app-oplysningerne. De indtastes som hemmeligheder på datakilden i OS2display.

## 1. Opret en app-registrering

1. Log på [Microsoft Entra admin center](https://entra.microsoft.com).
2. Gå til **Identity → Applications → App registrations**.
3. Vælg **New registration**.
4. Giv appen et sigende navn, for eksempel `OS2display Microsoft Graph`.
5. Under **Supported account types** vælges normalt **Accounts in this organizational directory only**.
6. Lad **Redirect URI** stå tom. Integrationen bruger ikke brugerlogin eller redirect.
7. Vælg **Register**.

På appens **Overview**-side skal du gemme:

- **Directory (tenant) ID**
- **Application (client) ID**

**App ID og Client ID er det samme:** brug værdien fra **Application (client) ID**.

## 2. Tilføj Microsoft Graph-tilladelser

Appen skal have **Application permissions**. Vælg ikke **Delegated permissions**.

1. Åbn appens **API permissions**.
2. Vælg **Add a permission → Microsoft Graph → Application permissions**.
3. Tilføj:

   | Tilladelse | Bruges til |
   |---|---|
   | `Place.Read.All` | Læse lokaler og lokalelister. |
   | `Calendars.Read` | Læse kalenderbookinger for lokalerne. |

4. Vælg **Grant admin consent** for organisationens tenant.
5. Kontrollér, at tilladelserne vises som **Granted**.

Tilladelserne skal være tilføjet på den samme app-registrering, hvis **Application (client) ID** indtastes i OS2display.

### Begræns adgang til postkasser

`Calendars.Read` som application permission kan give appen adgang til kalendere i organisationen. Begræns derfor appens adgang til de relevante lokale-postkasser efter organisationens sikkerhedspolitik.

Exchange-administratorer kan bruge en Exchange Online application access policy til at begrænse adgangen. Kontrollér den aktuelle Microsoft-vejledning for den anbefalede metode og organisationens Exchange-konfiguration. En begrænsning, der udelukker lokale-postkasserne, kan medføre, at kalenderkald returnerer `403 Forbidden`.

## 3. Opret en client secret

1. Gå til appens **Certificates & secrets**.
2. Under **Client secrets** vælg **New client secret**.
3. Angiv en beskrivelse og en udløbsperiode, som følger organisationens politik.
4. Vælg **Add**.
5. Kopiér hemmelighedens **Value** med det samme, og opbevar den sikkert.

Indtast **Value** i OS2display—not **Secret ID**. Værdien kan normalt ikke vises igen, efter du har forladt siden. Planlæg fornyelse inden udløb; når en secret er udløbet, kan OS2display ikke hente et nyt adgangstoken.

## 4. Opret en datakilde i OS2display

I OS2display oprettes en datakilde (feed source), som gemmer forbindelsesoplysningerne til Microsoft Graph:

1. Åbn **Datakilder**.
2. Opret en ny datakilde af typen **MS Graph kalender**.
3. Giv datakilden et sigende navn.
4. Indtast disse værdier:

   | Felt i OS2display | Værdi fra Microsoft |
   |---|---|
   | **Directory (tenant) ID** | **Directory (tenant) ID** fra appens Overview-side. |
   | **Application (client) ID** | **Application (client) ID** fra appens Overview-side. |
   | **Client Secret** | **Value** fra den oprettede client secret. |

5. Gem datakilden.

Af sikkerhedshensyn vises Tenant ID, Client ID og Client Secret ikke som almindelige gemte værdier, når datakilden åbnes igen. At felterne ser tomme eller skjulte ud, betyder ikke nødvendigvis, at værdierne ikke er gemt. Hvis du ændrer andre indstillinger, skal du normalt ikke indtaste hemmelighederne igen, medmindre du vil ændre dem.

## 5. Lokaler og lokalelister

### Hvad er en lokaleliste?

En lokaleliste er en gruppering af lokaler i Exchange, ofte efter bygning eller adresse. Den er ikke påkrævet for at hente lokaler. Den bruges som et **valgfrit filter** i OS2display.

Graph-endpointet for lokalelister returnerer lokalelister, ikke de individuelle lokaler. Lokaler hentes separat.

Hvis organisationen bruger lokalelister, skal de være oprettet og indeholde de relevante lokale-postkasser i Exchange Online. Exchange-administratoren kan kontrollere eller oprette lokalelister med Exchange Online PowerShell. Microsoft Graph viser kun de lokalelister og lokaler, som tenantens Exchange-konfiguration eksponerer.

### Feltet “Lokationer” på datakilden

Efter datakilden er gemt, kan feltet **Lokationer (lokalelister)** vise lokalelister hentet fra Microsoft Graph.

- **Ingen lokalelister valgt:** Alle lokaler, som Graph returnerer for tenantens app, kan vælges senere på den enkelte feed.
- **En eller flere lokalelister valgt:** OS2display begrænser den senere liste af valgbare lokaler til lokaler, der tilhører de valgte lokalelister.
- **Listen er tom:** Det betyder typisk, at Graph ikke returnerer lokalelister for tenantens opsætning. Det forhindrer ikke brug af lokaler; lad feltet stå tomt for at gøre alle tilgængelige lokaler valgbare.

Hvis Exchange-administratoren netop har oprettet eller ændret lokalelister, kan OS2display vise cachede oplysninger i op til en time, før listen opdateres.

### Vælg de lokaler, der skal vises

Når datakilden er gemt, kan en redaktør oprette en feed, vælge datakilden og vælge et eller flere lokaler i feedens lokalevælger. Det er disse lokaler, OS2display henter bookinger for.

Hvis der er valgt lokalelister på datakilden, indeholder lokalevælgeren kun lokaler fra de valgte lister. Hvis ingen lokalelister er valgt, indeholder den alle lokaler, Graph returnerer.

En feed kan vise bookinger for flere valgte lokaler; layoutet bestemmer, hvordan indholdet præsenteres på skærmen.

## 6. Hvilke kalenderdata vises?

OS2display henter kalenderdata fra de lokaler, der er valgt på feeden, for perioden:

> Fra midnat i dag til midnat syv dage senere.

Gentagne møder udvides til de forekomster, der ligger i perioden. Aflyste møder udelades.

Af hensyn til privatliv ændres titlen til **“Optaget”**, hvis Graph angiver mødet som privat, eller hvis emnet indeholder ordet `Privat`. Hvis et lokale er konfigureret til at fjerne mødeemner, kan titlen også vises som “Optaget”.

## 7. Cache og opdateringer

OS2display cacher data for at reducere antallet af kald til Microsoft Graph:

| Data | Cache |
|---|---|
| Adgangstoken | Genbruges indtil kort før udløb, typisk omkring 55 minutter. |
| Lokaler og lokalelister | En time. |
| Kalenderbookinger | Indstilles pr. feed i minutter; standard er fem minutter. |

Der findes også en separat cache for den samlede feed. Derfor kan ændringer i Exchange eller kalenderen tage lidt tid, før de ses på skærmen. Cachetiden for bookinger ændres på feeden; den ændrer ikke den faste cachetid for lokaler og lokalelister.

## 8. Fejlsøgning

| Symptom | Kontroller |
|---|---|
| Ingen lokalelister vises | Kontrollér, om der er oprettet Exchange-lokalelister. Hvis ikke, lad **Lokationer** stå tomt og vælg lokaler direkte på feeden. |
| Ingen lokaler vises | Kontrollér `Place.Read.All`, admin consent og at appen peger på den korrekte tenant. Husk, at lokaleoplysninger caches i op til en time. |
| HTTP 401 ved lokalekald | Kontrollér, at adgangstokenet er gyldigt, og at appen er konfigureret korrekt. |
| HTTP 403 ved lokalekald | Kontrollér application permissions og admin consent. Kontrollér også, at eventuelle Exchange-begrænsninger tillader de relevante data. |
| Bookinger vises ikke | Kontrollér, at det korrekte lokale er valgt, at bookingen ligger inden for de næste syv dage, og at bookingen findes i lokalets egen kalender. Kontrollér `Calendars.Read` og eventuelle Exchange-begrænsninger. |
| Titlen vises som “Optaget” | Mødet kan være markeret som privat, emnet kan indeholde `Privat`, eller lokale-postkassen kan være konfigureret til at fjerne emner. |
| Ændringer slår ikke straks igennem | Vent på, at relevante cacheperioder udløber, og kontrollér derefter igen. |

## Referencer

- [Registrér en app til app-only authentication](https://learn.microsoft.com/graph/tutorials?tabs=http)
- [Microsoft Graph permissions reference](https://learn.microsoft.com/graph/permissions-reference)
- [List places, rooms and room lists](https://learn.microsoft.com/graph/api/place-list)
- [Microsoft Graph calendarView](https://learn.microsoft.com/graph/api/calendar-list-calendarview)
- [Limit app access to specific mailboxes](https://learn.microsoft.com/graph/auth-limit-mailbox-access)



# MS Graph feed: endpoints and workflow

The browser calls our backend; the backend makes all requests to Microsoft. The app registration supplies the identity and permissions—it is not itself an API endpoint.

## Endpoints and when they are called

All Graph paths below use the base URL `https://graph.microsoft.com/v1.0`.

| Request | Purpose | Where in the workflow |
|---|---|---|
| **POST** `https://login.microsoftonline.com/{tenantId}/oauth2/v2.0/token` | Obtain an app-only access token using the saved Client ID and Client Secret. | Before any Graph request, if no cached token is available. |
| **GET** `/places/microsoft.graph.roomlist` | Get room lists—the optional groupings shown as “Lokationer”. | When opening an existing feed source’s configuration form. The browser requests `/v2/feed-sources/{id}/config/locations`. |
| **GET** `/places/microsoft.graph.room` | Get all rooms. | When the slide’s room selector loads, if no locations are selected on the feed source. The browser requests `/v2/feed-sources/{id}/config/resources`. |
| **GET** `GET /places/{roomListEmail}/microsoft.graph.roomlist/rooms` | Get rooms belonging to a selected room list. | Instead of the all-rooms request, when locations are selected. One lookup per selected room list. |
| **GET** `/users/{roomEmail}/calendarView` | Get bookings for an individual room. | When a preview or screen requests `/v2/feeds/{id}/data` and the feed cache needs refreshing. One calendar lookup per selected room. |

The requests are implemented in [ApiClient.php](src/Feed/SourceType/MsGraph/ApiClient.php).

## Calendar request details

For the example room, the endpoint is:

```text
GET https://graph.microsoft.com/v1.0/users/ress1023%40sonderborg.dk/calendarView
```

It sends:

- `startDateTime`: midnight today, converted to UTC.
- `endDateTime`: midnight seven days later, converted to UTC.
- `$select=id,subject,sensitivity,start,end,isCancelled`
- `$orderby=start/dateTime`
- `$top=100`
- Header `Prefer: outlook.timezone="UTC"`

Recurring meetings are expanded through `calendarView`; cancelled meetings are removed from the output.

Before fetching bookings, `getData()` in [MsGraphFeedType.php](src/Feed/MsGraphFeedType.php) also looks up room metadata to obtain friendly room names. That can trigger the same room-listing requests used by the dropdown, although they will usually be cached. A failed name lookup falls back to the room’s email address.

## How caching affects actual requests

A UI action does not necessarily cause an external request:

| Data | Cache duration |
|---|---|
| Access token | Reported token lifetime minus five minutes—typically about 55 minutes. |
| Room lists and rooms | One hour. |
| Bookings per room | The feed’s `cache_expire_minutes`, default five minutes. |
| Complete feed response | A separate outer cache, default five minutes, handled by [FeedService.php](src/Service/FeedService.php). |

Room discovery uses pages of 100, following `@odata.nextLink` or explicitly requesting `$skip=100`, `$skip=200`, etc. when Graph omits the link. Calendar requests follow `@odata.nextLink` when bookings span multiple pages.

Simply selecting a room or choosing the “one room” layout does not fetch its bookings. Bookings are fetched when the preview/client requests the feed data. The layout changes presentation, not the Graph endpoint or date window.

## No room list is returned
Room lists are entirely optional. You can have meeting rooms without having any room lists at all.

In Microsoft 365 / Exchange:
- A room is an individual room mailbox.
- A room list is an optional grouping of room mailboxes.
- Creating a room mailbox does not automatically require adding it to a room list.
- A tenant can have rooms but zero room lists.
If you want rooms grouped by location, your Exchange administrator needs to configure Room Lists containing the relevant room mailboxes.


## How room lists are used

Room lists are an optional filter, not a requirement for discovering rooms.

### Without room lists

```text
GET /places/microsoft.graph.room
→ Returns all rooms Microsoft Graph exposes in the tenant
→ User selects specific rooms in the slide’s feed configuration
→ Bookings are fetched for those selected rooms only
```

The `/places/microsoft.graph.room` endpoint does not need room IDs or room-list names as input. Each returned room includes an email address, which we then use to request its calendar:

```text
GET /users/{roomEmail}/calendarView
```

### With room lists selected

```text
Selected room lists on the feed source
→ Fetch rooms belonging to those lists
→ Only those rooms appear in the slide’s room selector
→ Fetch bookings for the rooms the user selects
```

## Two levels of selection

| Setting | Purpose |
|---|---|
| **Lokalelister on the feed source** | Optionally restrict which rooms editors can choose from. No selection means all discovered rooms. |
| **Lokaler on the slide’s feed** | Select the actual rooms whose bookings should be displayed. |

## Data cached by the MS Graph integration

The integration caches mapped calendar objects, not the complete JSON responses from Microsoft Graph. These are PHP data objects, not Doctrine/database entities.

## 1. Room lists → `Location` objects

**Cached value:** an array of [Location](src/Feed/OutputModel/Calendar/Location.php) objects, keyed by room-list email address.

| Property | Stored value |
|---|---|
| `id` | Room-list email address—not Microsoft’s place ID. |
| `displayName` | Microsoft’s `displayName`, falling back to the email address. |

Example, shown as JSON for readability:

```json
{
  "building@example.com": {
    "id": "building@example.com",
    "displayName": "Building A"
  }
}
```

**Cache scope:** all room lists for the credentials used by the feed source.

**Duration:** one hour. An empty array is also cached.

Other Graph properties, such as address and geographic coordinates, are not retained.

## 2. Rooms → `Resource` objects

**Cached value:** an array of [Resource](src/Feed/OutputModel/Calendar/Resource.php) objects, keyed by room email address.

| Property | Stored value |
|---|---|
| `id` | Room email address. |
| `name` | Room email address. |
| `locationId` | Room-list email address if fetched through a selected room list; otherwise an empty string. |
| `displayName` | Microsoft’s `displayName`, falling back to the room email address. |

Example:

```json
{
  "room@example.com": {
    "id": "room@example.com",
    "name": "room@example.com",
    "locationId": "building@example.com",
    "displayName": "Meeting room 1"
  }
}
```

**Cache scope:** separate entries for:

- All rooms.
- Rooms belonging to each queried room list.

**Duration:** one hour.

Room capacity, equipment, building, floor and other Graph metadata are not retained. When fetching all rooms, `locationId` is empty—the integration does not infer room-list membership.

## 3. Bookings → `CalendarEvent` objects

**Cached value:** a list of [CalendarEvent](src/Feed/OutputModel/Calendar/CalendarEvent.php) objects.

| Property | Stored value |
|---|---|
| `id` | A ULID generated by our backend—not Microsoft’s event ID. |
| `title` | Trimmed meeting subject, or `"Optaget"` if empty. |
| `startTimeTimestamp` | Start time as a Unix timestamp in seconds. |
| `endTimeTimestamp` | End time as a Unix timestamp in seconds. |
| `resourceId` | Room email address. |
| `resourceDisplayName` | Room email address at this stage—not the friendly room name. |

Example:

```json
[
  {
    "id": "01HZX0P8Y1N5V4S3Q2R1T0W9ZA",
    "title": "Project meeting",
    "startTimeTimestamp": 1719820800,
    "endTimeTimestamp": 1719824400,
    "resourceId": "room@example.com",
    "resourceDisplayName": "room@example.com"
  }
]
```

**Cache scope:** credentials + individual room + requested start/end window. The window is midnight today through midnight seven days later.

**Duration:** `cache_expire_minutes` from the feed, default five minutes.

Cancelled events and events whose dates cannot be parsed are excluded before caching. Attendees, organiser, meeting body and online-meeting links are not retained.

## 4. Access token → a string

**Cached value:** only the access-token string returned by Entra ID.

**Cache scope:** the credentials used by the feed source.

**Duration:** `expires_in` minus five minutes, with a minimum cache duration of 60 seconds.

The cache does not store a separate token object with `expires_in`, permissions or expiry properties. Expiration is managed as cache metadata. Token claims may be inspected separately where readable, but the token itself should not be printed.

## 5. An additional outer cache: the assembled feed

[FeedService](src/Service/FeedService.php) also caches the final response in the separate `feeds.cache` pool, keyed by feed ID:

```json
[
  {
    "id": "01HZX0P8Y1N5V4S3Q2R1T0W9ZA",
    "title": "Project meeting",
    "startTime": 1719820800,
    "endTime": 1719824400,
    "resourceTitle": "Meeting room 1",
    "resourceId": "room@example.com"
  }
]
```

Here, the backend has resolved the friendly room name and renamed the timestamp properties for the API output. It also generates a new output ID.

**Duration:** default five minutes, or the feed’s `cache_expire` setting in seconds. On a fetch failure, an empty response is cached for 30 seconds instead.

## Cache implementation

All four Microsoft-specific cache entries are implemented in [ApiClient](src/Feed/SourceType/MsGraph/ApiClient.php).

Their keys include a credentials fingerprint, so changing credentials causes subsequent lookups to use different cache entries.

## Aarhus gør det anderledes

### To Datakilder (eksempel fra Aarhus Kommune)

- Ressourcer og lokationer (fx SharePoint)
- Events fra Exchange via Microsoft Graph

Eksempel på ressource og lokation (opdigtede data)

| Type | Eksempel | Data |
|---|---|---|
| Ressource | Mødelokale 1 | `moedelokale1@kommune.dk`, visningsnavn “Mødelokale 1” |
| Lokation | Rådhuset | Adresse, by, postnummer og eventuelt koordinater |