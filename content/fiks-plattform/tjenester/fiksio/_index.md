---
title: Fiks IO
date: 2021-12-17
aliases: [/fiks-platform/tjenester/fiksio]
---

Fiks IO er en kanal for sikker maskin-til-maskin integrasjon. Denne kanalen kan benyttes for å bygge prosesser på tvers av systemer og organisasjoner, for eksempel når et fagsystem skal arkivere et dokument i et arkivsystem eller spørre om informasjon som er lagret i et annet system.

Fiks IO tilbyr:

* _Rask leveranse_: ved hjelp av kø-basert arkitektur ([AMQP)](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) kan Fiks IO levere meldinger raskt, stort sett på under ett sekund plus eventuelt overføring av data. Dette oppnås ved at meldinger nå sendes til det mottakende fagystemet i det øyeblikket de ankommer (push), i stede for at fagsystemet må hente meldingen (pull).
* _Svar på melding_: Fiks IO lar en mottaker svare på en spesifikk melding, f.eks. for å svare på en spørring eller å bekrefte at en forespurt handling er utført.
* _Sikker kommunikasjon med ende-til-ende kryptering_: Fiks IO tilbyr ende-til-ende kryptering av meldinger. Merk at dette ikke gjelder alle bruks-scenarioer, se "Sikkerhet" for detaljer.
* _Sikker identifisering av avsender_: Bruk av standard for kryptografisk signatur [(ASiC-E)](https://github.com/difi/asic) for meldinger gjør at man kan være sikker på identiteten til avsender.
* _Levetid på meldinger_: En melding har en brukerdefinert levetid. Avsender vil få beskjed hvis en melding ikke har blitt lest av mottaker innen levetidens utløp.
* _Sending av store filer_: Fiks IO integrerer mot [Fiks Dokumentlager]({{< ref "dokumentlager.md" >}}) for å støtte sending av store filer, helt opp til dokumentlagers grense på fem gigabyte. 

### Hvordan tar man i bruk Fiks IO?
Fiks IO baserer seg på at organisasjoner benytter Fiks Konfigurasjon for å opprette en eller flere _kontoer_. Andre organisasjoner kan så sende meldinger til disse kontoene, som mottaker kan lese ved å koble seg til io.fiks.ks.no. 

For å gjøre det lettere å finne en konto, og vite hvilke meldingstyper systemet som benytter kontoen kan håndtere kan man konfigurere _adresser_. Adresser består av en eller flere organisasjonsnummer, meldingsprotokoller og sikkerhetsnivåer. Avsendere kan så slå opp i Fiks IOs katalogtjeneste for å finne f.eks. hvilken konto som skal motta digisos meldinger med sikkerhetsnivå fire for Bergen kommune. 

Funksjonalitet for å sende og lese meldinger vil typisk bli tilbudt av fagsystemleverandører eller offentlig virksomheter, som når Fiks Digisos formidler sosialsøknader fra Nav til kommunens sosialsystem, men kommuner og andre organisasjoner må fortsatt ha et forhold til hvilke kontoer de har, hvilke meldingstyper disse håndterer, og ikke minst hvem som har tilgang til å lese inkommende meldinger.

![em fiks-io konto](/images/fiksiokonto.png)

I eksempelet over ser man en konto opprettet for å formidle Digisos meldinger til en kommunes fagsystem for behandlig av sosialsøknader. Det er opprettet en adresse på kontoen, "Digisos", man har konfigurert støtte for meldingsprotokollen "no.nav.digisos.fagsystem.v1", og sagt at dette systemet kan håndtere meldinger for både sikkerhetsnivå tre og fire. 

### Forhold til SvarUt og SvarInn
Fiks IO er en selvstendig kanal, og er ikke bygget for å være en erstatning for SvarUt/SvarInn, som begge vil bli videreført i sin nåværende form. Bruksområdene til tjenestene kan overlappe, og dette gjør at det noen ganger kan være tvil om SvarUt/SvarInn eller Fiks IO er riktig verktøy for et problem.

Hovedsakelig bør man benytte svarut/svarinn for "post": meldinger hvor payload er menneske-lesbar (f.eks. en pdf), hvor automatisk håndtering begrenses til f.eks. opprettelse av sak i et arkivsystem, og hvor sending til Altinn eller som brevpost er gode alternativer hvis den automatiske håndteringen feiler. Andre faktorer kan være:

#### Er dette en maskin-til-maskin integrasjon? 
Benytt Fiks IO, denne leverer utelukkende til spesifisert mottakerkonto og benytter ingen alternative kanaler, som for eksempel printet post.

#### Skal meldingen sikres gjennom ende-til-ende kryptering? 
Benytt Fiks IO, SvarUt/SvarInn meldinger kan bare krypteres med Fiks-plattformens nøkkel, ikke mottakers. Se "Sikkerhet" for deltajer om hvordan man oppnår trygg ende-til-ende kryptering gjennom Fiks-io

#### Trenger man rask levering? 
Benytt Fiks IO for å få leveranse på sekunder, ved bruk av SvarUt/SvarInn kan levering ta lang tid, siden SvarUt prøver flere kanaler etter tur og defaulter til print ved leveranseproblemer.

#### Vil man at fiks-plattformen skal garantere at meldingen blir levert?
Benytt SvarUt, ved Fiks IO forsendelser risikerer man at meldingen ikke blir håndtert eller avvist av mottaker. Man vil få melding om dette, men man må håndtere evt. retry eller sending via alternative kanaler selv.

Økonomiske aspekter kan også spille inn her, Fiks IO meldinger koster vesentlig mindre pr. stykk enn SvarUt forsendelser.

### Hvordan virker Fiks IO
![fiks_io](https://www.lucidchart.com/publicSegments/view/23b3e542-6059-471f-96d6-70f57da44f17/image.png)
En Fiks organisasjon oppretter en Fiks IO _konto_. Andre kontoer kan nå sende til denne kontoen gjennom Fiks IOs REST api ved å spesifisere kontoens _KontoId_ som mottaker. Organisasjonen får meldingene ved å etablere en AMQP kobling til io.fiks.ks.no.

Fiks IO tar i utgangspunktet ikke stilling til hva payloaden i meldingen består i. Metadataformat, filformat, kryptering og lignende er opp til brukeren, men Fiks plattformen tilbyr tjenester og verktøy for å etablere format for integrasjoner:

* _Fiks-IO Kontokatalog_: Gir mulighet for å registrere adresser bestående av organisasjonsnummer, sikkerhetsnivå, og hvilke meldingsprotokoller en gitt konto støtter. Andre kan så gjøre oppslag mot denne katalogen for å finne riktig konto å sende en melding til.
* _Fiks-IO Protokollkatalog_: Tilbyr en katalog over registrerte meldingsprotokoller. En protokoll er en oversikt over hvilke meldingstyper som inngår i en kommunikasjon, gjerne også med skjema som spesifiserer syntax for de enkelte typene.
* _Fiks-IO java klient_: [Java klient](https://github.com/ks-no/fiks-io-klient-java) som tilbyr funksjonalitet for å bygge, signere, kryptere, og sende meldinger som ASiC-E pakker, samt mottak og dekryptering på andre siden. 
* _Fiks-IO .net klient_: [.net core](https://github.com/ks-no/fiks-io-client-dotnet) implementasjon av samme funksjonalitet som klienten over.

### Sikkerhet
Autentisering av klienter mot REST service for sending av meldinger og AMQP service for leveranse av meldinger skjer gjennom virksomhetssertifikat-basert maskinporten autentisering. 

I utgangspunktet legger ikke Fiks IO føringer på hvordan (eller om) en melding sendt over plattformen sikres, men alle klienter som utvikles av KS, og alle protokoller som Fiks spesifiserer, vil benytte signerte og krypterte meldinger gjennom [ASIC-E](https://github.com/difi/asic) containere. Denne standarden benyttes også av DIFI i forbindelse med integrasjonspunktet. I klientene skjer signering med samme sertifikat som blir benyttet for autentisering mot maskinporten, kryptering skjer med sertifikatet som mottaker har publisert i Fiks IO Kontokatalog.

Merk at man for å oppnå reell ende-til-ende kryptering, i betydningen at KS ikke har noen mulighet til å lese den overførte meldingen, bør innhente mottakers sertifikater gjennom egne kanaler og selv verifisere disse. Fiks-IO tilbyr automatisk oppslag i katalogen som en tjeneste for å lette dette arbeidet, men man bør være klar over at det å benytte denne gir noe svekket sikkerhet.  

### Kontokatalogen
For å sende en melding i Fiks IO må man kjenne konto-id'en til mottakeren, men for å støtte mer dynamisk bruk tilbys Fiks IO Kontokatalog. Dette er et register over Fiks IO kontoer og tilknyttede adresser. En adresse består av:

* _Identifikator_: Autoriserende identifikator for adressen. Dette vil i første omgang være begrenset til organisasjonsnummer, men kan senere utvides. For organisasjonsnummer benyttes Altinn for autorisering: personen som legger til adressen må ha rollen "post/arkiv" på den aktuelle organisasjonen.
* _Protokoll_: Spesifiserer hvilken protokoll adressen gjelder for. En protokoll vil som regel omfatte flere meldingstyper.
* _Sikkerhetsnivå_: Spesifiserer hvilket sikkerhetsnivå meldingen skal ha. Typisk benyttes nivå 3 for ikke sensitive og nivå 4 for sensitive meldinger.

Adresser opprettes og forvaltes gjennom Fiks Konfigrasjon. Katalogen gir også mulighet for å laste opp en offentlig nøkkel i form av et X509 sertifikat. Dette sertifikatet blir benyttet for å kryptere meldinger sendt til kontoen, slik at de kan dekrypteres med den private delen av nøkkelen ved mottak.

Fiks Organisasjoner kan gjøre oppslag i registrerte adresser gjennom [katalog-api'et](https://editor.swagger.io/?url=https://ks-no.github.io/api/fiksio-katalog-api-v1.json). 

Merk at bruk av dette api'et i stede for manuell innhenting av sertifikater kan gi noe redusert sikkerhet. Se "Sikkerhet" for detaljer.

### Protokollkatalogen
Som nevnt over har Fiks IO i utgangspunktet ingen formening om hva innholdet i en melding er, men det er i mange integrasjon-scenarioer nyttig å ha et felles repository med kontrakter for hvordan meldinger skal bygges opp. Protokollkatalogen tilbyr en slik oversikt, og man kan her referere til spesifikasjoner for meldingstypene som inngår i protokollen, og hvordan disse skal benyttes. 

Digisos bruker som nevnt over meldingsprotokollen "no.nav.digisos.fagsystem.v1", som er beskrevet i mer detalj under [Fiks Digisos](https://ks-no.github.io/fiks-plattform/tjenester/digisos/#fiks-io-meldingsprotokoll).

Ta kontakt med fiks@ks.no om du ønsker å etablere eller gjøre endringer i en protokoll.

### Standardmeldingstyper

#### Tidsavbrudd (under utfasing)
I tillegg til meldinger definert i protokollkatalogen det er også definert en standard meldingstype som forteller at meldingens ttl har utløpt uten at mottaker har mottat den. Meldingstypen er `no.ks.fiks.kvittering.tidsavbrudd`. Denne meldingen blir sendt til avsender når originalmeldingen kommer fremst i køen til mottaker. Dette betyr at meldingen ikke trenger å komme når ttl utløper men kan komme en stund senere.
Hvis mottaker er nede og ikke leser meldinger og første melding har ttl på 2 timer. Vil ikke meldinger bak denne trigge tidsavbrudd melding før 2 timer meldingen som er først utløper. 

Disse meldingene inneholder ingen body, men kun headere deriblant `svar-til` som vil være en referanse til den opprinnelige meldingen (melding-id), `svar-til-type` som inneholder den originale typen på meldingen som har utløpt.

__NB! Siden utløp av meldinger ikke vil fungere godt om det ligger flere meldinger på kø vil denne funksjonaliteten fases ut snart. I stedet vil man få en [serverfeil](#serverfeil) når en melding er avvist tre ganger__

#### Serverfeil
Ved serverfeil hos mottaker skal det sendes en `no.ks.fiks.io.feilmelding.serverfeil` tilbake til sender. Json [schema](https://github.com/ks-no/fiks-io-client-dotnet/blob/master/KS.Fiks.IO.Client/Schema/no.ks.fiks.kvittering.serverfeil.v1.schema.json) følger med i .net pakken for Fiks-IO-client.


### Håndtering av store filer
Fiks IO støtter sending av store filer ved at alle meldinger større enn 5 megabyte mellomlagres i Fiks Dokumentlager, i en dedikert konto som opprettes sammen med Fiks IO kontoen. En referanse til denne lagrede filen blir så sendt over AMQP. Filer sendt på slik måte får en time-to-live i dokumentlager lik time-to-live for meldingen + 24 timer. Etter dette vil de automatisk slettes.

Hvis man benytter java eller .net klienten utviklet av KS vil denne mellomlagringen oppleves sømløst - klienten streamer automatisk filer fra dokumentlageret om dette er benyttet.

### Klienter
For å gjøre integrasjon lettere vil KS utvikle klienter som benyttes for både sending og mottak av meldinger fra Fiks IO. 

Klientene er tilgjengelig i [Java](https://github.com/ks-no/fiks-io-klient-java), og [.net core](https://github.com/ks-no/fiks-io-client-dotnet). Andre språk vil vurderes, og vi vil gjerne høre fra deg om du skriver klienter for andre språk. 

### Headere
Følgende headere er definert i meldingsutvekslingen.

| Header | Beskrivelse |
|---|---|
| avsender-id | Avsenders Fiks IO-konto |
| melding-id | Meldingens ID |
| avsender-navn | Navnet på avsenderkonto |
| type | Meldingstype |
| dokumentlager-id | ID til eventuell fil lagret i Dokumentlager |
| svar-til | Hvis meldingen er svar på en tidligere Fiks IO-melding, vil svar-til referere til den |
| svar-til-type | På `no.ks.fiks.kvittering.tidsavbrudd` meldinger legges typen til den opprinnelige meldingen |
