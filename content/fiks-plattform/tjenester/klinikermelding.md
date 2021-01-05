---
title: Klinikermelding hos FHI
date: 2020-12-14
---

### Formål
Tjeneste for å sende klinikermeldinger til FHI.

### Produktbeskrivelse
KS har sammen med Folkehelseinstituttet (FHI) utviklet et API som gjør det mulig å sende MSIS klinikermelding direkte fra smittesporingsverktøyet eller EPJ til MSIS ved FHI. Tjenesten heter Fiks klinikermelding. API-et vil bli tatt i bruk av Fiks smittsporing og ReMin så snart som mulig, og er tilgjengelig for kommuner som benytter andre løsninger til smittesporing enn de nevnte.

#### Brukeradministrasjon og tilgangsstyring
En må ha en gyldig HelseID innlogging for å sende klinikermeldinger. 
Kommunen må også gi tilgang til applikasjonen sin integrasjon i Fiks platformen, samt alle sluttbrukere som skal kunne sende klinikermeldinger.

### Sikkerhet
Kommunikasjonen vil være kryptert med TLS. Autentisering med HelseID og Fiks integrasjon.

## Endepunkt

Denne tjenesten er ikke ferdig enda, vi har en foreløpig apispec som kan indikere funksjonalitet i endelig api. [(api-spec)](https://editor.swagger.io/?url=https://ks-no.github.io/api/klinikermelding-api-v1.json)

For alle endepunktene hentes autentisert bruker fra access token i "Authorization"-headeren på requesten. Dette tokenet 
må være utstedt av HelseID med aud-verdi ks:fiks.

IntegrasjonId- og IntegrasjonPassord-headere må også være satt med verdier fra Fiks-integrasjonen opprettet av kommunen.


