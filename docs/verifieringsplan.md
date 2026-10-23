# Verifieringsplan – Kundserviceportalen

Status: planerad verifiering, 2026-10-23. Inga tester eller manuella produktkontroller har genomförts. Repot innehåller vid dokumentets tillkomst ingen applikationskod eller testsvit.

Krav och acceptanskriterier finns i [kravspecifikationen](kravspecifikation.md). Installation, körning och demonstrationsflöde finns i [README](../README.md).

## 1. Syfte och urval

Verifieringen ska visa att ärenden skapas med rätt uppgifter, att felaktig inmatning inte sparas, att uppgifter och statusändringar finns kvar efter omstart samt att listor, detaljer och statusfilter visar rätt ärenden.

Urvalet utgår från risken att felaktiga uppgifter sparas, att lagrad information försvinner, att fel ärende visas eller ändras och att en misslyckad operation presenteras som lyckad. Antal tester eller täckningsprocent är inte i sig ett mått på om acceptanskriterierna är uppfyllda.

## 2. Testnivåer och arbetssätt

- **Enhetstester av applikationslogik:** validering, standardvärden och regler för status. Ger snabb och precis återkoppling på regler och gränsfall.
- **Integrationstester med verklig testlagring:** registrering, hämtning, statusändring, sortering och filtrering samt återöppning av lagring. Verifierar samspelet mellan logik och den lagringsteknik som faktiskt används.
- **Automatiserade tester med simulerat lagringsfel:** verifierar hur applikationslogiken rapporterar en misslyckad skrivning. Kompletterar integrationstesterna men bevisar inte hantering av alla verkliga disk- eller databasfel.
- **Manuella kontroller av gränssnittet:** verifierar synlig återkoppling, navigation, formulärbeteende och demonstration enligt README.

Enhetstester ska inte vara den enda verifieringen av lagringen. Integrationstester använder en separat temporär databas eller motsvarande testlagring som återställs mellan tester. Tester ska inte ändra utvecklarnas sparade ärenden eller bero på körordning.

För beständighet stängs lagringsanslutningen och nya applikations-/lagringsinstanser öppnas mot samma testdata. Ett separat manuellt test stoppar och startar hela applikationen. Att endast läsa från samma minnesobjekt räcker inte.

Teknik och testverktyg är ännu inte valda. Testnivåerna kan anpassas efter arkitekturen, men kontrollerna och motiveringen ska behållas eller uppdateras uttryckligen.

## 3. Planerade automatiserade tester

Alla rader nedan har status **Planerat**. Test-ID:n är dokumentationsreferenser; det finns ännu inga testfiler att länka till. När tester implementeras ska kolumnen Testlänk ersättas med en relativ länk till relevant testfil eller testgrupp.

| ID | Typ | Krav | Beteende och förväntat resultat | Föreslagen nivå | Testlänk |
| --- | --- | --- | --- | --- | --- |
| T01 | Happy path | K1 | Giltig registrering sparar ett ärende med rätt trimmade fält, unikt ID, status Nytt och giltig skapandetid. Två registreringar får olika ID:n. | Integration | Saknas |
| T02 | Error path | K2 | Saknad eller tom rubrik respektive beskrivning avvisas. Fel identifierar berört fält och antalet sparade ärenden är oförändrat. | Logik och integration | Saknas |
| T03 | Gränsfall | K2 | Rubrik eller beskrivning med enbart blanksteg avvisas. Inget ärende sparas. | Logik och integration | Saknas |
| T04 | Gränsfall | K1, K2 | Rubrik på 1 och 100 tecken godkänns; 101 avvisas. Beskrivning på 1 och 5 000 tecken godkänns; 5 001 avvisas. Längd räknas efter trimning. | Logik | Saknas |
| T05 | Error path | K2 | Tom eller okänd kategori respektive prioritet avvisas i applikationslogiken även utan gränssnittets validering. Inget ärende sparas. | Logik och integration | Saknas |
| T06 | Happy path | K3 | Ett registrerat ärende finns kvar med samma uppgifter när lagringen stängs och öppnas av nya instanser. | Integration | Saknas |
| T07 | Happy path | K4, K5 | Flera ärenden listas med angivna listfält i fallande skapandetid. Hämtning med ett ID ger samtliga fält från rätt ärende. | Integration | Saknas |
| T08 | Gränsfall | K4, K8 | Tom lagring ger en tom ärendelista utan fel. Ett giltigt filter utan träffar ger en tom resultatlista. | Integration | Saknas |
| T09 | Error path | K5, K6 | Hämtning eller statusändring för ett okänt ID ger ett hanterat fel. Inget ärende skapas och befintliga uppgifter ändras inte. | Integration | Saknas |
| T10 | Happy path | K3, K6 | Tillåtna statusändringar sparas och finns kvar efter återöppnad lagring. Övriga fält är oförändrade. Återöppning från Avslutat till Pågående ingår. | Logik och integration | Saknas |
| T11 | Error path | K6 | Tom eller okänd status avvisas och tidigare lagrad status samt övriga ärendefält förblir oförändrade. | Logik och integration | Saknas |
| T12 | Gränsfall | K6 | Att välja aktuell status igen lyckas utan att ändra några ärendeuppgifter. | Integration | Saknas |
| T13 | Error path | K7 | Simulerat lagringsfel vid registrering ger ett hanterat misslyckat resultat och inget sparat ärende. | Logik med simulerat fel | Saknas |
| T14 | Error path | K7 | Simulerat lagringsfel vid statusändring ger ett hanterat misslyckat resultat och lämnar tidigare lagrad status oförändrad. | Logik med simulerat fel | Saknas |
| T15 | Kompletterande funktion / happy path | K8 | Bland ärenden med olika status returnerar varje statusfilter endast rätt ärenden i rätt sortering. Alla returnerar samtliga. Lagrade uppgifter ändras inte. | Integration | Saknas |
| T16 | Gränsfall | K1, K2 | Inledande och avslutande blanksteg tas bort före lagring, medan inre blanksteg och radbrytningar i beskrivningen bevaras. | Logik och integration | Saknas |

Närliggande fall får grupperas och parametriseras. Exakta feltexter behöver inte låsas i logiktester om feltypen och kopplingen till fältet kan verifieras. Begripligheten i den text som användaren faktiskt ser bedöms manuellt.

## 4. Planerade manuella kontroller

Alla kontroller har status **Planerat**. Datum, utförare och utfall fylls i efter faktisk körning.

| ID | Krav | Genomförande och förväntat resultat |
| --- | --- | --- |
| M01 | K1, K4, K5 | Registrera ett giltigt ärende i gränssnittet. Kontrollera bekräftelse, listfält, detaljfält och navigation tillbaka till listan. |
| M02 | K2 | Skicka ett ofullständigt formulär och överskrid en längdgräns. Kontrollera att felmeddelandet är begripligt, pekar på rätt fält och att inmatade uppgifter finns kvar. |
| M03 | K3, K6 | Skapa ett ärende och ändra status. Stoppa hela applikationen, starta igen och kontrollera att rätt ärende och status visas i både lista och detaljvy. |
| M04 | K4, K8 | Kontrollera tom lista, statusfilter med och utan träffar samt återställning med Alla. Kontrollera att respektive besked är begripligt. |
| M05 | K7 | Framkalla ett kontrollerat skrivfel i en separat testmiljö. Kontrollera begripligt fel, ingen lyckad bekräftelse, bevarad formulärinmatning och att misslyckad statusändring inte visas som sparad. Dokumentera hur felet framkallades. |
| M06 | Q1, Q4 | Följ README från en ren klon, installera, starta, kör testerna och genomför demonstrationsflödet. Dokumentera eventuella saknade steg. |
| M07 | K5, K6 | Begär ett ärende som inte finns via den åtkomstväg implementationen erbjuder. Kontrollera ett begripligt besked och att användaren kan fortsätta använda applikationen. |

## 5. Köra testerna

Exakta kommandon saknas eftersom språk, ramverk och testverktyg inte är valda. Detta avsnitt är inte en körbar testinstruktion ännu.

Före verifiering och inlämning ska följande fyllas i här eller länkas till i README:

1. Nödvändiga verktyg och versioner.
2. Kommando för installation av beroenden.
3. Hur separat testlagring skapas, konfigureras och återställs.
4. Exakta kommandon för att köra samtliga automatiserade tester från repots rot.
5. Hur ett lyckat respektive misslyckat testresultat känns igen.
6. Eventuella förberedelser och hur testdata städas bort.

Se [README:s körinstruktioner](../README.md#installation-och-körning).

## 6. Faktiska resultat

Inga automatiserade tester har körts och inga manuella produktkontroller har genomförts per 2026-10-23. Det finns därför inga godkända produktresultat att redovisa.

Efter varje relevant verifieringstillfälle registreras verkliga resultat enligt följande mall. Ersätt platshållarna först efter genomförd körning.

| Uppgift | Ska dokumenteras |
| --- | --- |
| Datum och utförare | När kontrollen kördes och vem som körde den. |
| Version | Commit-SHA för den testade koden samt eventuella lokala ändringar. |
| Miljö | Operativsystem, runtime, databas och relevanta versioner. |
| Automatiserade tester | Exakt kommando, test-ID:n, faktiskt antal godkända/misslyckade/överhoppade tester samt länk till testutdata eller CI-körning. |
| Manuella kontroller | M-ID:n, genomförda steg, observerat resultat och godkänt/underkänt. Redovisas separat från automatiserade tester. |
| Avvikelser | Berörda krav, länk till felrapport/uppgift och resultat av eventuell omtestning. |

Planerade, överhoppade eller ännu inte implementerade kontroller får inte redovisas som godkända.

## 7. Begränsningar

- Planen är inte bevis för att kraven är uppfyllda. Testlänkar och verkliga körresultat återstår.
- Logik- och integrationstester visar inte i sig att gränssnittets texter, formulär eller navigation fungerar. Manuella kontroller kompletterar dessa.
- Återöppnad lagring verifierar beständighet på integrationsnivå. Fullständig omstart av applikationen kontrolleras separat.
- Simulerade skrivfel verifierar den valda felvägen, inte alla verkliga lagringsfel. Vid implementation ska det dokumenteras var felet injiceras och vilken återställning testet faktiskt visar.
- Belastning, produktionsdrift, avancerad säkerhet, samtidiga uppdateringar från flera handläggare och externa integrationer ingår inte i denna verifieringsplan.
- Ingen viss testtäckningsprocent utlovas. Slutsatser ska kopplas till verifierade beteenden och kvarvarande risker.

## 8. Ansvar och spårbarhet

Varje utvecklingsuppgift kopplas till ett eller flera krav-ID:n och relevanta test-ID:n. Den som implementerar beteendet ansvarar för att dess verifiering och dokumentation uppdateras. Testförfattare, granskare och faktisk utförare dokumenteras i uppgiften eller PR:n.

Varje student ska kunna förklara varför testerna för det egna bidraget är relevanta, vad de visar och vilka begränsningar de har. Individuella redovisningar hanteras enligt kursinstruktionerna och läggs inte i repot.
