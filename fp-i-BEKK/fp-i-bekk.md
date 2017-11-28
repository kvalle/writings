# Funksjonell programmering i BEKK

Funksjonell programmering (FP) er et gammelt paradigme, som har fått et stadig sterkere fotfeste de siste årene. Vi har lenge sett funksjonelle konsepter snike seg inn i de vanligste imperative språkene våre, slik som Java, C# og JavaScript. Og på frontend-siden har tanken om å skrive views på en deklarativ måte blitt normen med [React](https://reactjs.org/).

Men det er også flere rent funksjonelle språk som begynner å bli sterke kandidater, både i frontend og backend. I backend har F# og Scala vært mulige valg, på henholdsvis .NET og JVM-en, og i frontend har det dukket opp en myriade av fuksjonelle språk som kompilerer til JavaScript de siste årene. Det mest populære av disse i BEKK-sammenheng er for øyeblikket [Elm](http://elm-lang.org/), men det er også andre sterke kandidater der ute, slik som [Fable](http://fable.io), [PureScript](http://www.purescript.org/) og [Reason](https://reasonml.github.io/).

## Interesse i BEKK

Vi har lenge hatt folk med interesse for FP i BEKK, men det er tydelig at interssen er økende. 2017 markerte det første året hvor vi hadde en egen faggruppe for funksjonell programmering (fokuset har vært noe bredere på programmeringsspråk generelt tidligere inkarnasjoner av gruppen), og vi hadde også en faggruppe med et initiativ dedikert til å finne ut hvordan Elm som språk passet i BEKK-prosjekter. I tillegg til dette var BEKKere sentrale i å starte [Oslo Elm Meetup](https://www.meetup.com/oslo-elm-meetup/) og å arrangere [Oslo Elm Day](https://osloelmday.no/).


## FP i BEKK anno 2017

Men selv om det er interesse for en teknolgi, så betyr ikke det automatisk at den blir tatt i bruk på kundeprosjekter. Det er fort å tenke at det er lett å prate om Elm og Scala, men at det i praksis er JavaScript og Java som er de reelle kandidatene når det kommer til stykket og man skal lage noe ute hos kunden.

For å finne ut av hvordan tilstanden faktisk er der ute bestemte jeg meg for å undersøke litt. Jeg har vært i kontakt med teknisk ansvarlige hos de ulike utviklingsprosjektene våre, og vil her dele noen av de resultatene jeg fant.

Jeg var i kontakt med 37 forskjellige prosjekter, fordelt hos 26 ulike kunder. Fordelingen så overordnet ut slik:

     FP  |=====     14 (38%)
    ¬FP  |========= 23 (62%)


### Funksjonelle nivåer

For å skille litt på hvordan FP brukes rundt om delte jeg prosjektene opp i ulike nivåer med FP-bruk:

- Nivå 0 🍼: Bruker ikke FP
- Nivå 1 🎓: FP til verktøystøtte, o.l.
- Nivå 2 🎩: FP på interne eller lite viktige systemer
- Nivå 3 👑: FP i produksjon på viktige systemer

Hvis vi fordeler de 14 prosjektene som bruker FP per nivå, ser resultatet slik ut:

    Nivå 1 🎓 |====         4
    Nivå 2 🎩 |             0
    Nivå 3 👑 |==========  10

Hele 10 av 37 prosjekter, dvs 27%, har faktisk FP i produksjon på systemer som er viktige hos kunden!

Det er også gøy å se at de prosjektene som tar i bruk FP faktisk gjør det på viktige systemer, og at det ikke ender som små eksperimenter på perifere løsninger med liten viktighet.

Dette er riktignok en sannhet med en liten modifikasjon: Jeg har her inkludert noen prosjekter som ikke var helt i produksjon da jeg gjore undresøkelsen, men disse er nære nok til at de forhåpentligvis er ute før denne bloggposten ser dagens lys.

En del av prosjektene på nivå 3 brukte FP også på nivå 1 eller 2. Tar vi hensyn til dette ser fordelingen slik ut:

      Nivå 1 🎓 |====== 6
      Nivå 2 🎩 |=== 3
      Nivå 3 👑 |========== 10

### Valg av språk

Vender vi blikket til hvilke språk vi velger, så har vi denne fordelingen:

    F#     |========== 5
    Scala  |========== 5
    Elm    |========== 5
    Kotlin |==== 2

De fire språkene som brukes vil neppe sjokkere noen. F# og Scala er etter hvert modne alternativ i .NET og på JVM-en, Elm har vært veldig populært de siste par årene, og Kotlin, med JetBrains i ryggen, har fått momentum som et enklere alternativ til Scala i det siste.

Men det hjelper ikke å bare vite hvilke språk som brukes, det er også interessant å se hva de brukes til. Hvis vi starter med språkene som brukes på nivå 1, til verktøystøtte, finner vi følgende:

    F#     |======== 4
    Scala  |===== 3
    Elm    |
    Kotlin |

Det er Scala og F# som er favorittene her, og det ser ut til at det er to årsaker til dette: [Fake](https://fake.build/) og [Gatling](https://gatling.io/). Fake er et byggesystem à la Make i F#, og er populært på flere av prosjektene våre, og Gatling er et ytelsestestverktøy der man skriver testene i Scala.

På nivå 2 var det ikke like mange prosjekter, men vi ser litt bruk av Elm og F#.

    F#     |== 1
    Scala  |
    Elm    |==== 2
    Kotlin |

Disse løsningene er typisk lite forretningskritiske løsninger, som gjerne bare brukes av utviklere og en liten gruppe interne hos kundene. Et eksempel på en slik er en løsning hos Statens Vegvesen som gir brukerstøtte tilgang til å slå opp hendelser i selvbetjeningssaker når folk ringer inn og lurer på ting.

På nivå 3, de viktigere løsnignene, er alle fire språkene representert, men Elm er den klart vanligste.

    F#     |==== 2
    Scala  |==== 2
    Elm    |========== 5
    Kotlin |==== 2

At Elm er populær er kanskje ikke overraskende, men det er betryggende å se at det vi prater om i faggrupper, meetups og på konferanse også brukes ute hos kundene.

### Offentlig vs privat

En artig observasjon fra undersøkelsen er fordelingen mellom bruk av FP hos offentlige og private kunder.

Det offentlige er gjerne stemplet som litt tregt og traust, og det er lett å tenke at det private gjerne er mer fremoverlent. Det er imidlertid en antagelse som får beina sparket bort under seg når vi oppdager at hele 75% av kundene som bruker funksjonelle språk er offentlige.

## Oppsummert

At 38% av BEKK sine prosjekter bruker funksjonell programmering, og ikke minst at 27% brukes til systemutvikling på viktige prosjekter, burde være en øyeåpner for dem som tror at FP ikke er et reelt alternativ.

Det blir dessverre ikke plass til å trekke frem så mange konkrete eksempler på prosjekter i denne bloggposten. Tanken her var heller å dele litt om i hvilken grad vi bruker funksjonell programmering, hva det brukes til, og hvilke språk vi velger. Vi vil fremover trekke frem noen konkrete prosjekter i egne bloggposter, for å dele mer konkret om hvilke erfaringer man har gjort seg. So stay tuned!
