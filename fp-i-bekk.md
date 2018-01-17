---
title: Hvor funksjonelle er vi?
date: 2017-12-20
external: https://blogg.bekk.no/hvor-funksjonelle-er-vi-7414246ca1ae
description: "*In norwegian.* Funksjonell programmering har hatt et oppsving de siste årene. Men hvor mye brukes det egentlig? Vi tar en titt på bruken av FP i prosjektene til BEKK."
---

# Hvor funksjonelle er vi?

Funksjonell programmering (FP) er et gammelt paradigme som har hatt en liten renesanse de siste årene, med stadig sterkere fotfeste i bransjen. Vi har lenge sett funksjonelle konsepter snike seg inn i de vanligste imperative språkene våre, slik som Java, C# og JavaScript. Og på frontend-siden har tanken om å skrive views på en deklarativ måte blitt normen med [React](https://reactjs.org/).

Det er også flere rent funksjonelle språk som begynner å bli sterke kandidater. I backend har F# og Scala lenge vært aktuelle valg, på henholdsvis .NET og på JVM-en, og i frontend har det dukket opp en myriade av fuksjonelle språk som kompilerer til JavaScript de siste årene. Det mest populære av disse i BEKK-sammenheng er for øyeblikket [Elm](http://elm-lang.org/), men det er også andre sterke kandidater der ute, slik som [Fable](http://fable.io), [PureScript](http://www.purescript.org/) og [Reason](https://reasonml.github.io/).

## Interesse i BEKK

Vi har lenge hatt folk med interesse for FP i BEKK, men det er tydelig at interessen er økende. 2017 markerte det første året hvor vi hadde en egen faggruppe for funksjonell programmering (temaet har vært programmeringsspråk generelt i tidligere inkarnasjoner av gruppen), og vi hadde også et initiativ dedikert til å finne ut hvor godt Elm passet i BEKK-prosjekter. I tillegg til dette var BEKKere sentrale i å starte [Oslo Elm Meetup](https://www.meetup.com/oslo-elm-meetup/) og å arrangere [Oslo Elm Day](https://osloelmday.no/).


## FP i BEKK anno 2017

Selv om konsulenter er interessert i en teknolgi, så betyr ikke det automatisk at den blir tatt i bruk på kundeprosjekter. Det er mye enklere å prate om Elm og Scala, enn det er å utfordre status quo med JavaScript og Java.

Er det fortsatt OOP som er det reelle alternativet når det kommer til stykket og man skal lage noe ute hos kunden, eller begynner FP endelig å få fotfeste? For å få oversikt over tilstanden i BEKK bestemte jeg meg for å undersøke litt, og tok kontakt med de teknisk ansvarlige hos de ulike utviklingsprosjektene våre. Jeg vil her dele noen av de resultatene jeg fant.

Totalt har jeg kartlagt 37 forskjellige prosjekter, fordelt hos 26 ulike kunder. Av disse prosjektene viser det seg at 14 bruker funksjonell programmering:

     FP ▕█████     14 (38%)
    ¬FP ▕█████████ 23 (62%)

FP er fortsatt ikke det vanligste valget, men med bruk i over en tredjedel av prosjektene er det også langt fra obskurt.


### Hva regner vi som funksjonelt?

Det er verdt å merke seg at skillet mellom hva som er og ikke er funksjonell programmering er vanskelig å definere. Mange tradisjonelt imperative språk begynner å absorbere flere funksjonelle konsepter. Jeg har likevel valgt å ikke regne prosjektene som bruker disse som funksjonelle i denne undersøkelsen. Samtidig er språk som tilbyr god støtte for både FP og OOP, slik som Kotlin og Scala, regnet som funksjonelle (med mindre prosjektene fokuserer på å bruke de objektorienterte aspektene).


### Funksjonelle nivåer

Det kan være interessant å vite at 38% av prosjektene våre bruker FP. Men det er langt mer spennende å vite _hvordan_ de bruker det. For å kunne kvantifisere dette deler vi prosjektene opp i fire ulike nivåer:

- Nivå 0 🍼: Bruker ikke FP
- Nivå 1 🎓: FP til verktøystøtte, o.l.
- Nivå 2 🎩: FP på interne eller lite viktige systemer
- Nivå 3 👑: FP i produksjon på viktige systemer

Nivå 0 representerer altså de 23 som ikke bruker FP. Ser vi nærmere på de 14 prosjektene hvor FP er i bruk, fordelt på nivå, ser resultatet slik ut:

    Nivå 3 👑 ▕████████████████████ 10
    Nivå 2 🎩 ▕ 0
    Nivå 1 🎓 ▕████████ 4

Hele 10 av 37 prosjekter, dvs 27%, har faktisk FP i produksjon på systemer som er viktige hos kunden! Det er morsomt å se at de prosjektene som tar i bruk FP faktisk gjør det på viktige systemer, og at det ikke bare ender som små eksperimenter på perifere løsninger med liten viktighet.

Det er verdt å nevne at en del av prosjektene på nivå 3 også bruker FP på nivå 1 eller 2. Tar vi hensyn til dette ser den justerte fordelingen slik ut:

    Nivå 3 👑 ▕████████████████████ 10
    Nivå 2 🎩 ▕██████ 3
    Nivå 1 🎓 ▕████████████ 6

### Valg av språk

Vendes blikket til hvilke språk vi velger, så har finner vi denne fordelingen:

    F#     ▕██████████ 5
    Scala  ▕██████████ 5
    Elm    ▕██████████ 5
    Kotlin ▕████ 2

De fire språkene som brukes vil neppe sjokkere noen. F# og Scala er etter hvert modne alternativ i .NET og på JVM-en. Elm har vært veldig populært de siste par årene. Og Kotlin, med JetBrains i ryggen, har fått momentum som et enklere alternativ til Scala i det siste.

Utover hvilke språk som brukes er det også interessant å se hva de brukes til. Hvis vi starter med språkene som brukes på nivå 1, altså til verktøystøtte, finner vi følgende:

    F#     ▕████████ 4
    Scala  ▕█████ 3
    Elm    ▕
    Kotlin ▕

Det er Scala og F# som er favorittene her, og det ser ut til at det er to årsaker til dette: [Fake](https://fake.build/) og [Gatling](https://gatling.io/). Fake er et byggesystem à la Make i F# som er populært på flere av prosjektene våre, og Gatling er et mye brukt ytelsestestverktøy der man skriver testene i Scala.

På nivå 2 var det ikke like mange prosjekter, men de få vi har bruker Elm og F#.

    F#     ▕██ 1
    Scala  ▕
    Elm    ▕████ 2
    Kotlin ▕

Disse løsningene er typisk lite forretningskritiske løsninger, gjerne bare brukt av utviklere og/eller en liten gruppe interne hos kunden. Et eksempel er en løsning vi har utviklet hos Statens Vegvesen, der brukerstøtte har tilgang til å slå opp hendelsesforløpet i selvbetjeningssaker slik at de kan svare folk som ringer og lurer på ting.

På nivå 3, de viktigere løsningene, er alle fire språkene representert, dog med Elm som det klart vanligste.

    F#     ▕████ 2
    Scala  ▕████ 2
    Elm    ▕██████████ 5
    Kotlin ▕████ 2

At Elm er populær er kanskje ikke overraskende, men det er betryggende å se at det vi prater om i faggrupper, på meetups og på konferanse også brukes ute hos kundene.

### Offentlig vs privat

En artig observasjon fra undersøkelsen er fordelingen mellom bruk av FP hos offentlige og private kunder.

Det offentlige er gjerne stemplet som litt tregt og traust, og det er lett å tenke at det private kan være mer fremoverlent. Det er imidlertid ikke støtte for dette i resultatene fra denne undresøkelsen. Faktisk er hele 75% av kundene der vi bruker funksjonelle språk offentlige.

## Oppsummert

At 38% av BEKK sine prosjekter bruker funksjonell programmering, og ikke minst at det i 27% brukes til systemutvikling på viktige løsninger, burde være en øyeåpner for dem som tror at FP ikke er et reelt alternativ i dag.

FP har lenge vært en god mulighet i backend, med etter hvert godt modne språk på mest vanlige platformene, uten at det helt har klart å gjøre FP mainstream. En funksjonell bølge er nå i ferd med å skylle inn også på frontend-siden i form av nye språk og bibliotiek/rammeverk i JS. Jeg liker utviklingen, og er veldig spent på hvordan dette utvikler seg i tiden fremover.

Det blir dessverre ikke plass til å trekke frem så mange konkrete eksempler i denne bloggposten. Det vil vi heller gjøre fremover i egne bloggposter, der vi deler erfaringer fra ulike prosjekter. So stay tuned!
