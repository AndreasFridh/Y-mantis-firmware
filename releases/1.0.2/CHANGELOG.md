# Ändringar i bryggans firmware

Versionen som står i varje logg­rad och som appen läser över C207 kommer ur
binärens egen beskrivning. Ett bygge från ett orört träd får firmwarekatalogens
commit­hash; ett bygge med lokala ändringar får `dev-` plus ett avtryck av
källorna, så att två testbyggen går att skilja åt.

Släpp taggas `fw-v<version>`. Bygget sker i GitHub Actions med **ESP-IDF v5.5**,
samma version som den hårdvaruverifierade imagen. En binär byggd med en annan
ESP-IDF är en annan binär — annan BLE-stack — och ska inte behandlas som samma
sak även om koden är identisk.

Bryggan vägrar en bild som byggts tidigare än den som kör. En medveten
nedgradering görs över USB, av någon som ser vad den gör.

## 1.0.2 — 2026-09-12

### Belasta inte buffertarna i onödan
- Telemetrin till telefonen har tak. Ticken går var tjugonde millisekund, vilket
  gav upp till femtio notifieringar i sekunden som konkurrerade med
  fordonsskrivningarna om samma buffertpool. Varje post behåller sitt senaste
  värde och markeras smutsig på nytt vid nästa ändring, så en långsammare takt
  tappar ingenting utom upprepningar som ändå hade skrivits över.
- Reglagen omfattas inte av taket. Ett knapptryck går fram med en gång, och
  ett värdtest håller den gränsen.
- Notifieringar till telefonen står tillbaka medan en fordonsskrivning är på
  väg ut, och när buffertpoolen börjar sina. Fordonet går före.

## 1.0.1 — 2026-09-12

### Rättat
- En tillfällig buffertbrist rev inte längre en godkänd fordonssession. NimBLE
  har ett begränsat antal utgående paket, och med telefonen ansluten — som
  prenumererar på både logg och händelseström — kan de ta slut för ett ögonblick.
  Noden svarade med `ble_gap_terminate` och kostade trettio sekunders uppkoppling
  för ett fel som var borta vid nästa tick. Mätt på hårdvara: `status=6`,
  `BLE_HS_ENOMEM`, följt av frånkoppling 534.
- Ramen kastas nu i stället för länken. Nästa tick bygger en ny med färsk
  klocka; en sekund gammal telefonstatus är inget värt att spara.
- Samma bräckliga mönster låg kopierat på sex ställen. Det ligger nu i
  `submit_vehicle_write`, som skiljer tillfälligt fel från verkligt.

## Ej släppt

### Skydd
- Uppdateringen läser den inkommande bildens beskrivning ur de första 288 byten
  och vägrar en bild byggd tidigare än den körande. Versionen är en git-hash och
  säger ingenting om vad som är nyare; byggtidpunkten gör det. Jämförelsen
  ligger i `bridge_version.c` och testas på värddatorn.

## 2026-09-12 — fristående nod

Den här milstolpen gjorde ESP32:an till en självständig ICE2-klient i stället
för en genomlysningsbrygga.

### Fungerar, verifierat mot motorcykeln
- Noden ansluter, bondar, autentiserar och förblir ansluten utan telefon.
  510 sekunder, 7 636 giltiga ramar, noll ogiltiga, MTU 247, stabil heap.
- Tändningscykel på 90 sekunder: frånkoppling, skanning efter 2 sekunder,
  autentisering 1,9 sekunder efter återupptäckt. Ingen manuell ombondning.
- Notifieringar syns i instrumentets lista. Klasserna 4 och 5 fungerar var för
  sig sedan applikationsplatserna först annonserats med `A6/058B`.

### Grundorsaken till den tidigare tystnaden
- MTU-utbytet saknades. Standard-MTU 23 rymmer 20 nyttolastbytes medan `AA` är
  60, så ramen kunde trunkeras — och ATT-kvittensen var inget bevis för att
  hela ramen kommit fram. Ordningen är nu: exakt sparat CCU-namn, kryptering,
  tjänster och verklig CCCD, bekräftad prenumeration, MTU-utbyte, **en** `AA`
  per anslutning, strikt svarskontroll.

### Nytt appgränssnitt
- `C207` running version, byggtid och partition.
- `C208` versionerade CAN- och reglagehändelser.
- `C209` versionerade kommandon: stopp, telefonstatus, telefon- och
  mediastatus, notifieringar, applikationsplatser, navigation.
- `C20A` räknare för poster, reglage, tappade och okända.

### Fordonsdata
- Växel avkodas ur `0x027C` byte 0 skiftad fem steg. Följde hela sekvensen i
  båda riktningarna och är säker.
- Varvtal ur `0x0512` byte 1 och motorstatus ur `0x02A3` byte 1 bit 3 är
  **obekräftade**: rimliga men inte jämförda med instrumentet.
- Blinkers och ljustuta gav inget separerbart utslag i familj `0x6C`.

### Begränsningar
- Grenen som raderar en avvisad gammal bond är inte verifierad mot hårdvara.
- Heldagstur, samtidig apptrafik och uppdatering över BLE från den nya appen är
  inte testade.
- Att kryptering lyckas bevisar inte att gamla nycklar raderats.
