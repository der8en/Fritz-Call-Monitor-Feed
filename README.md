# FRITZ!Box Call Monitor-Feed
**Home Assistant Automatisierung um die Anrufliste der Fritz!Box im Dashboard als Karte anzuzeigen.**

Die Automatisierung bassiert auf der von [MeinedigitaleWelt](https://youtu.be/w062jGwby_4). Vielen Dank dafür!

![der8en-fritz-feed-card](https://github.com/der8en/Fritz-Call-Monitor-Feed/assets/19150585/70f41327-5564-4da9-a3bf-d1fcd3a147b6)

## Optimierungen
- farbige Symbole im Feed für ausgehende, ankommende bzw. verpasste Anrufe
- unerwünschte Anrufer von Blocklisten (z.B. von Phoneblock.net) werden nicht als verpasste Anrufer angezeigt
- keine doppelten Einträge mehr im Feed

## Benötigt werden:
- AVM FRITZ!Box Call Monitor Integration
- HACS -> [Lovelace Home Feed Card](https://github.com/gadgetchnnel/lovelace-home-feed-card)
- 2 Helfer vom Typ "Text" (Maximale Länge 250)
  - `telefon_anrufer`
  - `telefon_feed`
- 1 Helfer vom Typ "Schalter"
  - `telefon_klingelt`
 
## Automatisierung
Inhalt der [telefon-anrufliste.yaml](/telefon-anrufliste.yaml) kopieren und als neue Automatisierung im YAML-Mode einfügen.
Der Teil "sensor.fritz_box_7490_anrufmonitor_telefonbuch" muss überall an den eigenen Entitätsnamen angepast werden!

Der Trigger "klingelt" mit ID 2 hat eine Mindestklingeldauer von 2 Sekunden. Wird diese Zeit reduziert/entfernt, werden geblockte Anrufer als verpasster Anruf angezeigt.  
Der Nachteil ist, wenn man das Telefon in unter 2 Sekunden abnimmt, wird dieser Anruf nicht getrackt! _(Kommt bei mir nie vor)_  

Wer keine Blockliste hat, kann die Zeit auch auf null setzen.

## Beispiel Home Feed Card
```
type: custom:home-feed-card
title: Anrufe
card_id: main_feed
show_empty: false
show_icons: false
id_filter: ^home_feed_.*
history_days_back: 5
entities:
  - entity: input_text.telefon_feed
    name: Telefon-Feed
    include_history: true
    max_history: 3
    content_template: '{{state}}'

```
