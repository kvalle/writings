---
title: Hvor funksjonelle er vi i 2018?
date: 2018-12-19
external: 
description: "Det har gått et år siden sist vi stilte spørsmålet, og mye kan skje på et år. Er funksjonell på vei inn eller ut? Og er Elm fortsatt det mest populære valget? Rett og slett: Hvor funksjonelle er Bekk i 2018?"
---

## Hvor funksjonelle er vi i 2018?

Det har gått et år siden [sist vi stilte dét spørsmålet](https://blogg.bekk.no/hvor-funksjonelle-er-vi-7414246ca1ae), og mye har skjedd både i Bekk og i bransjen for øvrig. Vi har på ny hatt en runde og undersøkt ståa ute i de ulike prosjektene våre, og det er på tide å finne ut hvor funksjonelle vi er i Bekk ved utgangen av 2018.

![Illustration photo which really has nothing to do with FP](https://images.unsplash.com/photo-1534988333262-c455b9332e52?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1566&q=80)



### Hva legger vi i FP?

Først: Definisjoner! For å vite hvor mye FP vi bruker, må vi først si noe om hva vi i denne sammenhengen legger i det begrepet.

For å kunne sammenlikne resultatene med fjoråret har vi forsøkt å følge samme definisjon på hva vi legger i _funksjonell programmering_ i år også. For å gjøre det enkelt å samle inn data (og slippe å diskutere hva FP _egentlig_ er med alle) har vi gjort skillet mellom språk som tilbyr mekanismer for å gjøre FP på en enkel måte, og språk hvor dette er noe man må jobbe litt ekstra for å få til.

Språk som Java, C# og JavaScript teller ikke som FP i denne undersøkelsen. Språk som F#, Scala og Elm er derimot på den funksjonelle siden.

Det kanskje mest omstridte valget vi har gjort er legge Kotlin blant de funksjonelle språkene. Kotlin er et språk som har lagt seg tett opp mot Java, men selv om Kotlin kanskje mangler noen av de mer avanserte funksjonelle featurene, så ligger likevel mye til rette for å bruke Kotlin funksjonelt. Med funksjoner som first-class citizens, mulighet til å definere funksjoner uten å måtte knytte dem til en klasse, algebraiske datatyper og immutability som default har man de verktøyene man trenger for å kunne starte å kode i en funksjonell stil.


### FP på prosjekt

Vi har i denne runden gått enda grundigere til verks enn sist, og har fått inn svar fra hele 52 ulike team/prosjekter fordelt på engasjement hos 25 kunder.

La oss se på det mest åpenbare tallet først:

     FP ▕█████████████████████████████  29 (55%)
    ¬FP ▕███████████████████████        23 (45%)

Det er altså nå blitt flere prosjekter som rapporterer at de bruker funksjonell programmering enn dem som ikke gjør det i det hele tatt! Til sammenlikning var det 38% som brukte FP i fjor. Det er en markant økning.

### FP, really?

Når vi ser på hvilke språk som er representert vil vi (spoiler alert!) se at Kotlin har hatt en markant fremgang. Og som vi nevnte over er det ikke gitt at alle som bruker Kotlin koder i en funksjonell stil. 

Vi har faktisk fått kommentarer fra noen prosjekter som beskriver at selv om de har tatt i bruk Kotlin, så har de ikke virkelig fått tatt i bruk de funksjonelle aspektene enda. På en annen side har vi fått vel så mange kommentarer fra prosjekter som bruker Java og JavaScript om at de virkelig føler de bruker FP til tross for at vi ikke regner språkene som funksjonelle her.

Vi tror derfor at tallene kan være ganske representative, til tross for at det er mange nyanser som ikke kommer med når vi gjør en slik forenkling.

Det er også verdt å merke seg at de 29 prosjektene som rapporterer bruk av FP ikke _utelukkende_ bruker funksjonelle språk, men at de har minst ett funksjonelt språk i bruk.


### Hva brukes det til?

Vi skiller her på 3 ulike områder med bruk. 

- 🚀 Å ha FP i bruk i reelle systemer i produksjon.
- 🛷 Å bruke FP til systemutvikling på mindre viktige systemer. Dette vil typisk si interne applikasjoner, eller ting som er veldig lite forretningskritisk og ikke skal brukes av sluttbrukere.
- 🛠 Å bruke funksjonelle språk til tooling, slik som testing eller deploy pipeline.

Merk at et gitt prosjekt gjerne befinner seg innen flere av disse samtidig.

Prosjektene fordeler seg som følger:

     🚀	███████████████████████  23
     🛷	███████████              11
     🛠	███████████              11

Vi ser altså at de fleste som tar i bruk funksjonelle språk bruker dem til reell systemutvikling på viktige systemer. Dette stemmer ganske godt med resultatene fra fjorårets undersøkelse.


### Hvilke språk brukes?

Årets fordeling ser slik ut:

	Kotlin	██████████████████	18
	   Elm	██████	             6
	    F#	██████               6
	 Scala	██	                 2


#### Kotlin 📈

Her har Kotlin virkelig tatt et byks fremover. På samme tid i fjor hadde vi kun 2 prosjekter som hadde tatt i bruk Kotlin!

Det er tydelig at fremgangen på antall prosjekter med FP i hovedsak skyldes en enorm fremgang på bruk av Kotlin. Mange av disse prosjektene er riktignok i startgropa med tanke på å ta i bruk de riktig funksjonelle aspektene av språket, slik som [Arrow](https://arrow-kt.io/).

Den enorme fremgangen til Kotlin kommer ikke som en komplett overraskelse. Det har vært ganske høyt aktivitetsnivå rundt Kotlin i Bekk det siste året. Noe som blant annet har resultert i at vi fra 2019 får en ny faggruppe nettopp for dette språket. Det ser ut til at denne gruppen vil få mer enn nok å ta for seg.

#### Elm

Elm har stått relativt stille fra i fjor. Dette er litt overraskende med tanke på hvor mye som skjer i Elm-miljøet om dagen. Siste eksempel på dette er [julekalenderen med bloggposter](https://elm.christmas) som Elm-faggruppa produserte nå i desember. I tillegg er [Meetup-en](https://www.meetup.com/oslo-elm-meetup/) fortsatt ledet av Bekk-ere, og flere har også engasjert seg i [Oslo Elm Day](https://osloelmday.no/) som våkner til nytt liv i 2019. Likevel har det altså ikke kommet til så mange nye prosjekter på denne fronten.

Men at Elm ikke har den samme enorme fremgangen som Kotlin er kanskje heller ikke helt overraskende. Kotlin kan brukes side ved side med Java, og legger opp til å gjøre overgangen så enkel som overhodet mulig. Elm er på sin side en radikal endring fra JavaScript, der en må forholde seg til et helt nytt paradigme og blir tvunget ut av komfortsonen og utfordret til å lære nye måter å jobbe på fra starten av. Vi ser likevel at det er potensiale for å omsette mer av engasjementet rundt Elm til reelle prosjekter ute hos kunde.


#### F# og Scala

Bruken av F# har, som Elm, holdt seg forholdsvis uendret fra i fjor. Det er fortsatt en god blanding av språk for verktøystøtte (blant annet med byggverktøyet [Fake](https://fake.build/)) og bruk i reelle systemer.

Scala har på sin side gått kraftig bakover. Det ser ut som dette skyldes at vi har sluttet å bruke Scala til tooling. Begge Scala-prosjektene er på 🚀-nivå, mens alle prosjektene som i fjor rapportere bruk av Scala til ting som ytelsestesting med [Gatling](https://gatling.io/) har falt fra i år.

### Privat mot offentlig

I fjor så vi at det var en stor overvekt på offentlige kunder blant de Bekk-prosjektene som brukte FP. I årets resultater er denne trenden sterkt redusert, og fordelingen er langt jevnere:

	Offentlig	████████████████ 	16
	   Privat	█████████████	    13

I den totale fordelingen blant alle prosjektene er til sammenlikning 25 offentlige og 27 hos private kunder.


### Hva med dem som ikke bruker FP?

I år undersøkte vi også stemningen blant dem som ikke bruker FP i prosjekt per i dag. Er det fordi de er interessert men ikke har fått muligheten, eller de rett og slett ikke interessert? Prosjektene fordelte seg slik:

	Interessert i FP  ████████████
	Ikke interessert  ████████
	Vet ikke          ██

Den absolutt vanligste utfordringen for dem som rapporterer at de gjerne skulle tatt FP i bruk er mangel på erfaring og kompetanse med funksjonelle språk i teamet.

Det er også en del som nevner at prosjektet eller kunden har standardisert på Java/JavaScript/andre språk, og at det er vanskelig å utfordre dette.

Og til sist er det noen som rapporterer at selv om de gjerne skulle brukt et funksjonelt språk, så er det rett og slett ikke det riktige verktøyet for det de holder på med for øyeblikket.


### Oppsummert

Funksjonell programmering ser altså ut til å ha hatt en merkbar fremgang i Bekk, mye på grunn av en enorm økning i bruk av Kotlin.

Det er nok mange av prosjektene med Kotlin som i realiteten ikke er veldig funksjonelle, og det er nok en del prosjekter med språk vi her ikke har regnet som funksjonelle som faktisk bruker en del konsepter fra FP. Dette er noe vi bør undersøke nærmere i neste iterasjon av denne undersøkelsen.

Vi synes også det er oppløftende at den vanligste grunnen til at folk som ønsker å bruke FP ikke gjør det i dag er at de ønsker mer kompetanse. Det betyr at faggruppene for funksjonell programmering og Elm har en viktig rolle i året som kommer.
