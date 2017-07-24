# Løsninsbeskrivelse gjenkjenning av eIDAS-brukere i ID-porten

## Fase A: arkitekturvisjon

Visjonen er henta fra et av målene fra ID-porten sin strategi:

> *Fysiske personer med europeisk e-ID kan autentiseres i ID-porten.*

Denne utredninga ser på alternativet der gjenkjenningsfunksjoanlitet er plassert i ID-porten

## Fase B: Virksomhetsarkitektur



<div class="mermaid">
graph LR
subgraph Skatteeaten
  dsf[Folkeregisteret]
end
subgraph gjenkjenningsfunksjonalitet
  gjenkjenningskomponent
  gjenkjenningsdatabase(gjenkjenningsdatabase)
end
  utland[Utanlandsk eIDAS Node]
  subgraph Difi
      eidas[Norsk eIDAS Node]
      proxy[eIDAS tilpassingskomponent]
      idp[ID-porten]
  end
  sp[Norske tjenester]
  eidas --- utland
  eidas --- proxy
  proxy --- idp
  idp --- sp
  proxy --- gjenkjenningskomponent
  gjenkjenningskomponent --- gjenkjenningsdatabase
  gjenkjenningskomponent --- Andre_attributt_leverandør
  gjenkjenningskomponent --- dsf
</div>

På virksomhetsnivå vil følgende forretningsprosesser være relevante for  eIDAS-pålogginger:



#### Infrastruktur-prosesser

|ID|Namn|Status|Skildring|Kommentar|
|-|-|-|-|-|
|1|Som eIDAS-bruker ønsker jeg å kunne logge inn til norske offentlige tjenster|OK, implementert i ID-porten | Basis-flyt, der en eiDAS-bruker blir logget inn i norske tjenester gjennom ID-porten. Medfører at ID-porten og norsk eIDAS-node og utenlandske eIDAS Noder må integreres. Det er ingen gjennkjenning i denne brukerhistorien - eIDAS-bruker må identifiseres i tjenestene vha. eIDAS-identifikatoren.| "0-alternativet" i eIDAS-prosjektet |


#### Gjenkjenningstjenester

|ID|Namn|Status|Skildring|Kommentar|
|-|-|-|-|-|
|2|Som offentlig tjeneste ønsker jeg at eIDAS-brukere blir entydig identifisert med F/D-nummer, slik at jeg slipper å endre mine fagsystemer|Ikke implementert|Når ID-porten mottar en pålogging fra en eIDAS-bruker, vil det blir foretatt en gjenkjenningsprosess i samhandling med Folkeregisteret for å finne D/F-nummer, og eventuelt berike innloggingen med denne.|(*Kun "best-effort" (gjenkjenning basert på navn+fødselsdato) er implementert idag.*)|
|3|Som offentlig tjeneste ønsker jeg at eIDAS-brukere som ikke blir gjenkjent, skal bli vist enkel feilside i ID-porten, slik at jeg slipper å lage egen feil-behandling | OK, implementert | eIDAS-bruker som ikke blir gjenkjent, får opp en side "Du har ikke D-nummer. Ta kontakt med Folkeregisteret dersom dette ikke stemmer." |
|4|Som offentlig tjeneste ønsker jeg at bare eIDAS-brukere som er gjenkjent med F/D-nummer kan logge inn i min tjeneste, slik at jeg slipper å endre mine fagsystemer|OK, implementert i ID-porten| Tjenester som er prekonfigurert for dette, vil _kun_ motta pålogginger der eIDAS-brukeren er gjenkjent. Dersom eIDAS-brukeren ikke bli gjenkjent, se #3. Noen tjenester kan håndtere brukere både med og uten D-nummer, og de klarer seg med #2. **Merk:** denne brukerhistorien omhandler ikke selve gjenkjeninngsprosessen (#2).|Behovsanalysen viser at majoriteten av tjenesteeiere ønsker slik oppførsel.|
|5|Som norsk eIDAS-Node skal jeg be utenlandsk eIDAS Node i samarbeidsland  om å sende TIN-nummer som ekstra attributt til innloggingen, slik at vi kan gjenkjenne eIDAS-bruker fra de land der eIDAS-identifikatoren ikke er meningsbærende.  | Ikke implementert | Flere land vil ikke sende en meningsbærende eIDAS-identifikator. Men med enkelte av disse kan det på bilateral basis inngås avtaler om utveksling av ytterligere, meningsbærende informasjon, som feks. TIN eller  personidentifikator.  ID-porten må vite hvilke land dette er snakk om, og be om ytterligere informasjon når det er relevant.  | Må vurdere om dette skal skje hver gang, eller kun _dersom_ gjenkjenning mislykkes.  Kan feks. skje i kombinasjon med #3 slik: *"Jo, jeg har D-nummer og ønsker at norske myndigheter henter ytterligere informasjon fra mitt hjemland, for å se om dere finner meg igjen i norske registre."*|
|6|Som gjenkjenningskomponent skal jeg slå opp i ESSII-databasen for å kvalitetssikre kobling mellom D-nummer og utenlansk TIN-nummer| Dersom eIDAS-bruker ikke ble gjenkjent mot Folkeregisteret, men

#### Administrasjonstjenester

|ID|Namn|Status|Skildring|Kommentar|
|-|-|-|-|-|
|5|Som saksbehandler av rekvireringssøknader skal jeg lagre identifikator og land fra utenlandsk id-bevis i Folkeregisteret, slik at eIDAS-brukere kan gjenkjennes automatisk ved første gangs innlogging.|ukjent?|Gjennkjenningskomponent kan finne utenlandsk person i Folkeregisteret dersom |Må gjenkjenningskomponent vite _hvilken_ identifikator det er snakk om (TIN, passnr., personnummer, andre)
|z|Som ikke-gjenkjent eIDAS-bruker som vet at jeg har D/F-nummer, ønsker jeg å kunne oppgi mitt D/F-nummer frivillig.|ikke implementert |Bruker skriver inn sitt D-nummer selv. Lagres i gjenkjenningskomponenten for senere gjenkjenning. Utleveres til tjenestene merket med identifikatorstyrke "selvdeklarert". | Gir sannsynligvis for å dårlig koblings-styrke.  Risikovurderes?
|z|Som ikke-gjenkjent eIDAS-bruker som vet at jeg har D/F-nummer, ønsker jeg å bruke norsk eID til å oppdatere Folkeregisteret med eIDAS-identifikator| ikke implementert |Bruker logger inn med både eIDAS og norsk eID (nivå4?). eIDAS-identifikatoren lagres i gjenkjenningskomponenten (og evt. Folkeregisteret), slik at bruker vil gjenkjennes ved senere innlogginger (se #2). | Antar at mange av disse ikke besitter norsk eID (lengre). Liten gevinst ? |
|z| Som "Relevant" departement skal jeg inngå bilateral avtale med samarbeidsland om å utveksle nødvendig tilleggsinformasjon for å sikre entydig gjenkjenning av eIDAS-brukere|Ikke implementert|Manuell prosess basert på møtevirksomhet og brev.|Difi og Folkeregisteret må informeres/være med i prosessen.




## Fase C: Applikasjonsarkitektur


sdette er markdown

```
dette er noko kode
```

<div class="mermaid">
graph TD
  p--> b
</div>
