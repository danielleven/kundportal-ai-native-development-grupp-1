# Kravspecifikation – Kundserviceportalen

Status: första förslag för gruppens granskning, 2026-10-23. Dokumentet beskriver planerad funktionalitet, inte en färdig implementation.

## 1. Syfte och målgrupp

Kundserviceportalen ska ge handläggare en gemensam överblick över kundärenden. En handläggare ska kunna registrera ett ärende, läsa dess innehåll och följa det genom att ändra status.

Den primära användaren är en handläggare. Ärenden registreras manuellt. En separat kundvy eller kundinloggning ingår inte.

## 2. Uppdragskrav och föreslagna beslut

Uppdraget kräver registrering med validering, beständig lagring, lista och detaljvy, statusändring, begriplig felhantering, automatiserade tester och instruktioner för körning och demonstration. Minst en kompletterande funktion ska specificeras före implementation.

Följande är gruppens föreslagna preciseringar och behöver fastställas innan implementation:

- Fältreglerna, kategorierna och prioriteterna i avsnitt 4.
- Fyra statusar och fria övergångar mellan dessa enligt avsnitt 5.
- Filtrering på status som kompletterande funktion enligt K8.
- Felhantering och sortering enligt acceptanskriterierna i avsnitt 6.

Ändrade beslut dokumenteras här och i verifieringsplanen. Uppdraget föreskriver inte de exakta längdgränserna eller kategorierna.

## 3. Omfattning

### Ingår

- Ett användargränssnitt, applikationslogik och beständig lagring.
- Registrering, ärendelista, ärendedetaljer och statusändring.
- Filtrering av ärendelistan på en status åt gången.
- Automatiserad verifiering av centrala beteenden och kompletterande manuella kontroller.
- Lokal installation, körning och demonstration med dokumenterade instruktioner.

### Ingår inte

- Produktionsdrift eller molninfrastruktur.
- Inloggning, behörighetsroller eller separat kundportal.
- Integration med e-post, chatt eller andra ärendekanaler.
- AI-funktioner i produkten.
- Bilagor, interna anteckningar, tilldelning eller ärendehistorik.
- Fritextsökning, flera samtidiga filter eller redigering av andra ärendefält efter registrering.
- Radering av ärenden.

Endast påhittade uppgifter får användas i applikation, testdata och demonstration. Lösningen ska kunna bedömas utan ett personligt betalkonto.

## 4. Ärendemodell och validering

| Fält | Regel |
| --- | --- |
| Ärende-ID | Unikt, skapas automatiskt och ändras inte. Exakt format väljs med tekniken. |
| Rubrik | Obligatorisk. 1–100 tecken efter trimning. |
| Beskrivning | Obligatorisk. 1–5 000 tecken efter trimning. |
| Kategori | Obligatoriskt val: Faktura, Teknisk support eller Övrigt. |
| Prioritet | Obligatoriskt val: Låg, Normal eller Hög. |
| Status | Nytt, Pågående, Väntar på kund eller Avslutat. Sätts automatiskt till Nytt vid registrering. |
| Skapad | Sätts automatiskt när ärendet skapas. Kan inte ändras av användaren och bevaras vid statusändring. |

Inledande och avslutande blanksteg tas bort från rubrik och beskrivning före validering och lagring. Text som enbart innehåller blanksteg räknas som tom. Inre blanksteg och radbrytningar i beskrivningen bevaras. Längd räknas efter trimning; den tekniska definitionen av tecken fastställs med teknikvalet och används konsekvent i validering och tester.

Kategori och prioritet väljs bland tillåtna värden. Ett tomt val eller ett värde utanför listan avvisas. Ärende-ID, startstatus och skapandetid bestäms av applikationslogiken.

Tidpunkter lagras entydigt, exempelvis i UTC med tidszon, och visas som ett läsbart datum och klockslag. Exakt format dokumenteras med teknikvalet.

Reglerna ska upprätthållas i applikationslogiken även om gränssnittets validering kringgås. Gränssnittet kan ge tidigare återkoppling med samma regler.

## 5. Statusflöde

| Status | Betydelse |
| --- | --- |
| Nytt | Ärendet är registrerat och ännu inte påbörjat. |
| Pågående | En handläggare arbetar med ärendet. |
| Väntar på kund | Arbetet väntar på kompletterande information från kunden. |
| Avslutat | Ärendet är färdigbehandlat. |

Alla övergångar mellan de fyra giltiga statusarna tillåts. Ett avslutat ärende kan därför återöppnas. Att välja aktuell status igen är tillåtet och lämnar ärendets uppgifter oförändrade.

Motivering: ett enkelt flöde ger handläggaren flexibilitet och håller implementationen avgränsad. Automatiska övergångar och regler om vem som får ändra status ingår inte.

## 6. Funktionella krav och acceptanskriterier

<a name="k1"></a>

### K1 – Registrera ärende

- Med giltig rubrik, beskrivning, kategori och prioritet sparas exakt ett nytt ärende per lyckad registreringsoperation.
- Sparade uppgifter följer fältreglerna. Ärendet får ett unikt ID, status Nytt och skapandetid.
- Användaren får en bekräftelse efter lyckad lagring och kan hitta ärendet i listan och öppna dess detaljer.

<a name="k2"></a>

### K2 – Validera inmatning

- Tomma obligatoriska fält, text med enbart blanksteg, för långa texter och ogiltiga val avvisas enligt avsnitt 4.
- Inget ärende sparas när valideringen misslyckas.
- Gränssnittet visar ett begripligt fel som identifierar fältet och hur felet rättas. Redan ifyllda uppgifter finns kvar.
- Exempel: ”Ange en rubrik” och ”Rubriken får innehålla högst 100 tecken”.

<a name="k3"></a>

### K3 – Bevara ärenden

- Ett sparat ärende finns kvar med samma ID och uppgifter när applikationen stoppas och startas igen med samma lagring.
- En sparad statusändring finns också kvar efter omstart.
- Minneslagring som töms vid omstart uppfyller inte kravet.

<a name="k4"></a>

### K4 – Visa ärendelista

- Listan visar ID, rubrik, kategori, prioritet, status och skapandetid för sparade ärenden.
- Senast skapade ärenden visas först. Ärenden med samma skapandetid får visas i valfri inbördes ordning.
- Om inga ärenden finns visas ett tydligt tomläge, exempelvis ”Det finns inga ärenden ännu”.

<a name="k5"></a>

### K5 – Visa ärendedetaljer

- Ett ärende kan öppnas från listan. Alla fält i ärendemodellen visas och tillhör det valda ärendet.
- Om ett efterfrågat ID inte finns visas ett begripligt besked, exempelvis ”Ärendet kunde inte hittas”.
- Användaren kan återgå till ärendelistan.

<a name="k6"></a>

### K6 – Ändra status

- En giltig statusändring på ett befintligt ärende sparas och visas i både detaljvy och lista.
- Ärendets ID, rubrik, beskrivning, kategori, prioritet och skapandetid ändras inte av statusoperationen.
- Ett ogiltigt statusvärde avvisas utan att den tidigare statusen ändras.
- Ett okänt ärende-ID ger ett begripligt fel och skapar inte ett nytt ärende.
- Statusändringen finns kvar efter omstart enligt K3.

<a name="k7"></a>

### K7 – Hantera lagringsfel

- Om lagring misslyckas vid registrering eller statusändring visas ett begripligt fel, exempelvis ”Ändringen kunde inte sparas. Försök igen”.
- Ingen lyckad bekräftelse visas. Inmatade formulärvärden behålls vid misslyckad registrering.
- En misslyckad skrivoperation lämnar den tidigare beständiga datan oförändrad. Gränssnittet får inte presentera ändringen som sparad.

<a name="k8"></a>

### K8 – Filtrera på status, kompletterande funktion

- Ärendelistan har valen Alla, Nytt, Pågående, Väntar på kund och Avslutat.
- Alla är förvalt och visar alla ärenden.
- Ett statusval visar endast ärenden med exakt den statusen och behåller sorteringen enligt K4.
- Om inga ärenden matchar visas ett tydligt besked, exempelvis ”Inga ärenden matchar vald status”.
- När användaren väljer Alla återställs listan utan att lagrade ärenden ändras.

Motivering: filtret hjälper handläggaren att hitta ärenden som behöver hanteras och kräver inga ytterligare fält i ärendemodellen.

## 7. Kvalitetskrav och leverans

| ID | Krav | Acceptanskriterium |
| --- | --- | --- |
| <a name="q1"></a>Q1 | Lokal och reproducerbar körning | README beskriver faktiska förutsättningar, installation, lagringskonfiguration och startkommandon. En annan person kan följa instruktionerna och starta lösningen lokalt. |
| <a name="q2"></a>Q2 | Automatiserad verifiering | Centrala beteenden verifieras mot acceptanskriterier med happy paths, relevanta error paths och gränsfall. Testkommandon och testlänkar dokumenteras. |
| <a name="q3"></a>Q3 | Spårbar verifiering | Verifieringsplanen skiljer planerade tester från genomförda kontroller, automatiserat från manuellt och dokumenterar faktiska resultat samt begränsningar. |
| <a name="q4"></a>Q4 | Demonstration och eget bidrag | README beskriver ett demonstrationsflöde. Varje medlem kan visa sitt bidrag och förklara dess verifiering genom länkade uppgifter, PR:er och tester. |

## 8. Beslut att fastställa före implementation

| Beslut | Nuvarande läge |
| --- | --- |
| Språk, ramverk och klientplattform | Inte valt. Välj efter gruppens kompetens. |
| Databas eller annan beständig lagring | Inte valt. Ska klara K3 och kunna köras lokalt. |
| Testverktyg och exakta testkommandon | Inte valt. Dokumenteras i README och verifieringsplanen. |
| Fältregler och statusflöde | Föreslagna i detta dokument; ska fastställas av gruppen. |
| Kompletterande funktion | Statusfilter föreslaget; ska fastställas före implementation. |
| Teknisk definition av textlängd och tidsformat | Fastställs tillsammans med teknikvalet. |
| Teammedlemmar och ansvar | Ansvarsfördelningen finns i [README](../README.md#ansvarsfördelning). Konkreta uppgifter kopplas till ansvarig medlem och relevanta krav-ID:n. |

## 9. Föreslagen arbetsordning

1. Fastställ avgränsning, regler, teknik och testverktyg.
2. Skapa körbar projektstruktur och uppdatera installationsinstruktionerna.
3. Bygg registrering, validering och lagring med tester för K1–K3.
4. Bygg lista och detaljvy med tester för K4–K5.
5. Bygg statusändring och felhantering med tester för K6–K7.
6. Bygg statusfiltret med tester för K8.
7. Genomför manuella kontroller, verifiera körning från en ren klon och dokumentera resultaten.

Varje utvecklingsuppgift bör ange krav-ID, ansvarig, acceptanskriterier och hur ändringen ska verifieras. Uppdatera dokumentationen när ett beslut ändras.
