# FRITZ!Box Call Monitor-Feed
**Home Assistant Automatisierung um die Anrufliste der Fritz!Box im Dashboard als Karte anzuzeigen.**

Die Automatisierung bassiert auf der von [MeinedigitaleWelt](https://youtu.be/w062jGwby_4). Vielen Dank dafür!

## Optimierungen
- farbige Symbole im Feed für ausgehende, ankommende bzw. verpasste Anrufe
- unerwünschte Anrufer von Blocklisten (z.B. von Phoneblock.net) werden nicht mehr als verpasste Anrufer angezeigt
- keine doppelten Einträge mehr im Feed

## Benötigt werden:
- AVM FRITZ!Box Call Monitor Integration
- HACS -> Lovelace Home Feed Card
- 2 input_text Helfer
  - `telefon_klingelt`
  - `telefon_feed`
