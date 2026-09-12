# Y mantis — bryggans firmware

Kompilerade utgåvor för **Y mantis bryggnod**, en ESP32-C3 Super Mini som
ansluter till en Yamaha Ténéré 700 över BLE och talar ICE2 med motorcykelns
CCU. Noden äger fordonsanslutningen själv och behöver ingen telefon för att
förbli ansluten.

Det här repot innehåller bara de byggda binärerna. Källkoden ligger i ett
privat huvudrepo.

## Utgåvor

Varje utgåva under [Releases](../../releases) bär:

| Fil | Innehåll |
| --- | --- |
| `ymantis-bridge.bin` | Bilden som flashas, byggd med ESP-IDF v5.5 |
| `manifest.json` | Version, SHA-256, storlek och byggets commit |
| `CHANGELOG.md` | Vad som ändrats, och vad som är verifierat mot fordonet |

`manifest.json` är det appen läser. SHA-256 kontrolleras innan en enda byte
skickas till noden, så att en avbruten nedladdning aldrig når flashet.

## Installation

Y mantis-appen uppdaterar noden över BLE. Första installationen på en ny nod
görs över USB, eftersom partitionstabellen måste ha två app-platser innan
uppdatering över luft är möjlig.

Bryggan vägrar en bild som byggts tidigare än den som kör. En medveten
nedgradering görs över USB, av någon som ser vad den gör.

## Vad som inte finns här

Ingen dekompilering, inga Yamaha-binärer och inga fordonsreferenser. Binären
avslöjar ingenting som inte redan sitter i en flashad ESP32.
