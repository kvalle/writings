# Funksjonell programmering i BEKK

Funksjonell programmering (FP) er et gammelt paradigme som har hatt en liten renesanse de siste årene, med stadig sterkere fotfeste i bransjen. Vi har lenge sett funksjonelle konsepter snike seg inn i de vanligste imperative språkene våre, slik som Java, C# og JavaScript. Og på frontend-siden har tanken om å skrive views på en deklarativ måte blitt normen med [React](https://reactjs.org/).

Det er også flere rent funksjonelle språk som begynner å bli sterke kandidater. I backend har F# og Scala lenge vært aktuelle valg, på henholdsvis .NET og JVM-en, og i frontend har det dukket opp en myriade av fuksjonelle språk som kompilerer til JavaScript i de siste årene. Det mest populære av disse i BEKK-sammenheng er for øyeblikket [Elm](http://elm-lang.org/), men det er også andre sterke kandidater der ute, slik som [Fable](http://fable.io), [PureScript](http://www.purescript.org/) og [Reason](https://reasonml.github.io/).

## Interesse i BEKK

Vi har lenge hatt folk med interesse for FP i BEKK, men det er tydelig at interessen er økende. 2017 markerte det første året hvor vi hadde en egen faggruppe for funksjonell programmering (temaet har vært programmeringsspråk generelt i tidligere inkarnasjoner av gruppen), og vi hadde også et initiativ dedikert til å finne ut hvor godt Elm passet i BEKK-prosjekter. I tillegg til dette var BEKKere sentrale i å starte [Oslo Elm Meetup](https://www.meetup.com/oslo-elm-meetup/) og å arrangere [Oslo Elm Day](https://osloelmday.no/).


## FP i BEKK anno 2017

Men selv om konsulenter er interessert i en teknolgi, så betyr ikke det automatisk at den blir tatt i bruk på kundeprosjekter. Det er fort å tenke at det er lett å prate om Elm og Scala, men at det i praksis er JavaScript og Java som er de reelle kandidatene når det kommer til stykket og man skal lage noe ute hos kunden.

For å finne ut av hvordan tilstanden faktisk er der ute bestemte jeg meg for å undersøke litt, og tok kontakt med de teknisk ansvarlige hos de ulike utviklingsprosjektene våre. Jeg vil her dele noen av de resultatene jeg fant.

Totalt har jeg kartlagt 37 forskjellige prosjekter, fordelt hos 26 ulike kunder. Helt overordnet så fordelingen som bruker FP slik ut:

     FP ▕█████     14 (38%)
    ¬FP ▕█████████ 23 (62%)

Mer enn ett av tre prosjekter bruker altså FP i en eller annen grad. Det er altså ikke normen å bruke FP helt enda, men det er også langt fra obskurt.

### Hva regner vi som funksjonelt?

_TODO_

### Funksjonelle nivåer

Etter å ha sett hvor mange som bruker FP er det spennende å se _hvordan_ det brukes rundt om. For å kvantifisere dette skiller vi prosjektene opp i ulike nivåer:

- Nivå 0 🍼: Bruker ikke FP
- Nivå 1 🎓: FP til verktøystøtte, o.l.
- Nivå 2 🎩: FP på interne eller lite viktige systemer
- Nivå 3 👑: FP i produksjon på viktige systemer

Hvis vi fordeler de 14 prosjektene som bruker FP per nivå, ser resultatet slik ut:

    Nivå 3 👑 ▕████████████████████ 10
    Nivå 2 🎩 ▕ 0
    Nivå 1 🎓 ▕████████ 4

Hele 10 av 37 prosjekter, dvs 27%, har faktisk FP i produksjon på systemer som er viktige hos kunden!

Det er også morsomt å se at de prosjektene som tar i bruk FP faktisk gjør det på viktige systemer, og at det ikke bare ender som små eksperimenter på perifere løsninger med liten viktighet.

Dette er riktignok en sannhet med en liten modifikasjon: Jeg har her inkludert noen prosjekter som ikke var helt i produksjon da jeg gjore undresøkelsen, men disse er nære nok til at de forhåpentligvis er ute før denne bloggposten ser dagens lys.

En del av prosjektene på nivå 3 bruker FP også på nivå 1 eller 2. Tar vi hensyn til dette ser den justerte fordelingen slik ut:

    Nivå 3 👑 ▕████████████████████ 10
    Nivå 2 🎩 ▕██████ 3
    Nivå 1 🎓 ▕████████████ 6

### Valg av språk

Vender vi blikket til hvilke språk vi velger, så har vi denne fordelingen:

    F#     ▕██████████ 5
    Scala  ▕██████████ 5
    Elm    ▕██████████ 5
    Kotlin ▕████ 2

De fire språkene som brukes vil neppe sjokkere noen. F# og Scala er etter hvert modne alternativ i .NET og på JVM-en. Elm har vært veldig populært de siste par årene. Og Kotlin, med JetBrains i ryggen, har fått momentum som et enklere alternativ til Scala i det siste.

Men utover hvilke språk som brukes er det også interessant å se hva de brukes til. Hvis vi starter med språkene som brukes på nivå 1, altså til verktøystøtte, finner vi følgende:

    F#     ▕████████ 4
    Scala  ▕█████ 3
    Elm    ▕
    Kotlin ▕

Det er Scala og F# som er favorittene her, og det ser ut til at det er to årsaker til dette: [Fake](https://fake.build/) og [Gatling](https://gatling.io/). Fake er et byggesystem à la Make i F#, og er populært på flere av prosjektene våre, og Gatling er et mye brukt ytelsestestverktøy der man skriver testene i Scala.

På nivå 2 var det ikke like mange prosjekter, men de få vi har bruker Elm og F#.

    F#     ▕██ 1
    Scala  ▕
    Elm    ▕████ 2
    Kotlin ▕

Disse løsningene er typisk lite forretningskritiske løsninger, gjerne bare brukt av utviklere og/eller en liten gruppe interne hos kunden. Et eksempel er en løsning vi har utviklet hos Statens Vegvesen, der brukerstøtte har tilgang til å slå opp hendelsesforløpet i selvbetjeningssaker slik at de kan svare folk som ringer og lurer på ting.

På nivå 3, de viktigere løsnignene, er alle fire språkene representert, dog med Elm som det klart vanligste.

    F#     ▕████ 2
    Scala  ▕████ 2
    Elm    ▕██████████ 5
    Kotlin ▕████ 2

At Elm er populær er kanskje ikke overraskende, men det er betryggende å se at det vi prater om i faggrupper, meetups og på konferanse også brukes ute hos kundene.

### Offentlig vs privat

En artig observasjon fra undersøkelsen er fordelingen mellom bruk av FP hos offentlige og private kunder.

Det offentlige er gjerne stemplet som litt tregt og traust, og det er lett å tenke at det private gjerne er mer fremoverlent. Det er imidlertid en antagelse som punkterer raskt når vi oppdager at hele 75% av kundene som bruker funksjonelle språk er offentlige.

## Oppsummert

At 38% av BEKK sine prosjekter bruker funksjonell programmering, og ikke minst at det i 27% brukes til systemutvikling på viktige løsninger, burde være en øyeåpner for dem som tror at FP ikke er et reelt alternativ i dag.

Det blir dessverre ikke plass til å trekke frem så mange konkrete eksempler i denne bloggposten. Det vil vi heller gjøre fremover i egne bloggposter, der vi deler erfaringer fra ulike prosjekter. So stay tuned!
