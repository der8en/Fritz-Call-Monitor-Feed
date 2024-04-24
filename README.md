# FRITZ!Box Call Monitor-Feed
**Home Assistant Automatisierung um die Anrufliste der Fritz!Box im Dashboard als Karte anzuzeigen.**

![der8en-fritz-feed-card](https://github.com/der8en/Fritz-Call-Monitor-Feed/assets/19150585/70f41327-5564-4da9-a3bf-d1fcd3a147b6)

Die Idee zur Automatisierung basiert auf der von [MeinedigitaleWelt](https://youtu.be/w062jGwby_4). Vielen Dank dafür!

## Optimierungen
- farbige Symbole im Feed für ausgehende, ankommende bzw. verpasste Anrufe etc.
- Anrufbeantworter nach X Sekunden
- Anrufbeantworter in bestimmten Zeiten
- unerwünschte Anrufer von Blocklisten (z.B. von Phoneblock.net) werden nicht als verpasste Anrufer angezeigt
- keine doppelten Einträge für den gleichen Anruf
- wiederholte Anrufer werden auch mehrfach angezeigt

## Voraussetzungen
- [AVM FRITZ!Box Call Monitor Integration](https://www.home-assistant.io/integrations/fritzbox_callmonitor/) _(`#96*5*` nicht vergessen!)_
- HACS -> [Lovelace Home Feed Card](https://github.com/gadgetchnnel/lovelace-home-feed-card)
- 2 Helfer vom Typ "Text" (Maximale Länge 250)
  - `telefon_anrufer`
  - `telefon_feed`
- 1 Helfer vom Typ "Schalter"
  - `telefon_klingelt`

![Helfer](https://github.com/der8en/Fritz-Call-Monitor-Feed/assets/19150585/1fe21548-7a7a-4ac1-a208-17dc2261dc9a)

## Grundlagen
Inhalt der [telefon-anrufliste.yaml](/telefon-anrufliste.yaml) kopieren und als neue Automatisierung im YAML-Mode einfügen.
Der Teil "sensor.fritz_box_7490_anrufmonitor_telefonbuch" muss überall an den **eigenen Entitätsnamen angepasst** werden!
Am besten im Texteditor mit "Suchen und ersetzen" erledigen.

Der Trigger "klingelt" mit ID 2 hat eine Mindestklingeldauer von 2 Sekunden. Der Nachteil ist, wenn man das Telefon in unter 2 Sekunden abnimmt, wird dieser Anruf nicht getrackt! _(Kommt bei mir nie vor)_  

Wird diese Zeit reduziert/entfernt, werden geblockte Anrufer als verpasster Anruf angezeigt. Wer keine Blockliste hat, kann die Zeit auch auf null setzen.

### ohne Anrufbeantworter
Das ist die Voreinstellung in der YAML. Hier muss nichts weiter angepasst werden.

### AB nach X Sekunden
Man kann leider nicht feststellen, ob jemand eine Nachricht hinterlässt, da der AB als normales Telefonat gewertet wird. Aber man kann nachverfolgen, ob es überhaupt lange genug geklingelt hat, damit der Anrufer den AB hört.
- den Trigger "Zeit für AB nach X Sekunden (abzgl. 1 Sekunde)" aktivieren (im 3-Punkte Menü)
- im Trigger die Zeit, nachdem der eigene AB aktiviert, weniger 1 Sekunde, eintragen (unten bei "Für")  
Beispiel: Euer AB aktiviert sich nach 50 Sekunden, dann tragt ihr hier 0:00:49 ein.

### AB außerhalb bestimmter Zeiten
z.B. Geschäftszeiten.
- unter "Dann mache"
  - "V1: Ankommend (ohne Geschäftszeiten)" deaktivieren (im 3-Punkte Menü)
  - "V2: Ankommend (innerhalb Geschäftszeiten)" aktivieren
  - "V2: AB (außerhalb Geschäftszeiten)" aktivieren
- die Zeiten/Tage anpassen
  - "V2: Ankommend (innerhalb Geschäftszeiten)" aufklappen -> Wenn -> Testen, ob eine von 2 Bedingungen zutrifft 
  - die gleichen Zeiten, nur umgekehrt, unter "V2: AB (außerhalb Geschäftszeiten)" eintragen
 
Damit diese Variante gemeinsam mit den 2 Sekunden Mindestklingeldauer funktioniert, sollte die Anrufannahme in der Fritz!Box auf 5 Sekunden oder mehr eingestellt sein!

## Home Feed Card
Wichtige Einstellungen:
- show_icons: false _(damit nicht zwei Icons angezeigt werden)_
- history_days_back: 5 _(max. vergangene Tage)_
- include_history: true
- remove_repeats: false _(damit wiederholende Anrufer angezeigt werden)_
- max_history: 3 _(max. Anzahl Einträge)_

Beispiel:
```
type: custom:home-feed-card
title: Anrufe
card_id: main_feed
show_icons: false
id_filter: ^home_feed_.*
history_days_back: 5
entities:
  - entity: input_text.telefon_feed
    name: Telefon-Feed
    include_history: true
    remove_repeats: false
    max_history: 3
    content_template: '{{state}}'
```

## Benachrichtigungen
sind nicht enthalten, können aber leicht hinzugefügt werden.  
Unter "Dann mache" die Aktion heraussuchen, bei der benachrichtigt werden soll.
1. Aktion hinzufügen
2. Benachrichtigung
3. Typ/Gerät auswählen
4. im 3-Punkte Menü -> Als YAML bearbeiten
5. data-Code einfügen

evtl. noch Text anpassen

Beispiel für verpasste Anrufe:
```
service: notify.deine_entität
data:
  message: Verpasster Anruf von {{states("input_text.telefon_anrufer")}}
```

