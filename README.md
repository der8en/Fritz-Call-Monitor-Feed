
# FRITZ!Box Call Monitor-Feed
**Home Assistant Automatisierung um die Anrufliste der Fritz!Box im Dashboard als Karte anzuzeigen.**

<img width="643" height="530" alt="Fritz Call Monitor Feed" src="https://github.com/user-attachments/assets/edadf948-34cc-4916-82cc-68f8c4bcfe75" />

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
- 2 Helfer vom Typ "Text" (Maximale Länge 250)
  - `telefon_anrufer`
  - `telefon_feed`
- 1 Helfer vom Typ "Schalter"
  - `telefon_klingelt`
- folgendes Template in der `configuration.yaml`:
```
template:
  - trigger:
      - platform: state
        entity_id: input_text.telefon_feed
        # Ignoriere Neustarts von Home Assistant (Wechsel von 'unavailable')
        not_from:
          - "unavailable"
        # Ignoriere, wenn der Status ZU unbekannt/leer wechselt.
        not_to:
          - "unknown"
          - "unavailable"
          - ""
    sensor:
      - name: "Telefon Feed Verlauf"
        unique_id: telefon_feed_verlauf
        state: "{{ trigger.to_state.state | truncate(250) }}"
        attributes:
          verlauf: >
            {% set neu_wert = trigger.to_state.state %}
            {% set neu_zeit = now().isoformat() %}
            {% set alt_verlauf = this.attributes.get('verlauf', []) %}
            
            {% set ns = namespace(liste=[]) %}
            
            {# Alte Einträge übernehmen #}
            {% for eintrag in alt_verlauf %}
              {% if eintrag is mapping and 'zeit' in eintrag %}
                {% set ns.liste = ns.liste + [eintrag] %}
              {% endif %}
            {% endfor %}
            
            {# Neuen Eintrag oben anfügen, als JSON speichern und auf 20 Einträge begrenzen #}
            {{ ([{"zeit": neu_zeit, "wert": neu_wert}] + ns.liste)[:20] | to_json }}
```
In der letzten Zeile kann über [:20] die maximale Anzahl der gespeicherten Einträge festgelegt werden. Standardmäßig sind 20 Einträge vorgesehen; zu hohe Werte können sich jedoch negativ auf die Ladezeit auswirken.

![Helfer](https://github.com/der8en/Fritz-Call-Monitor-Feed/assets/19150585/1fe21548-7a7a-4ac1-a208-17dc2261dc9a)

## Grundlagen
Inhalt der [telefon-anrufliste.yaml](/telefon-anrufliste.yaml) kopieren und als neue Automatisierung im YAML-Mode einfügen.
Der Teil "sensor.fritz_box_7590_ax_anrufmonitor_telefonbuch" muss überall an den **eigenen Entitätsnamen angepasst** werden!
Am besten in einem Texteditor mit "Suchen und ersetzen" erledigen.

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

## Markdown Karte
In Zeile 3 lässt sich die maximale Anzahl der angezeigten Einträge festlegen. Dieser Wert kann unabhängig von den im Template gespeicherten Einträgen nach Bedarf auch niedriger gewählt werden. Voreingestellt sind 20.
```
type: markdown
content: |-
  {%- set max_eintraege = 20 -%}{# maximale Anzahl der Einträge #}
  {%- set wochentage = ['So', 'Mo', 'Di', 'Mi', 'Do', 'Fr', 'Sa'] -%}
  {%- set anrufe = state_attr('sensor.telefon_feed_verlauf', 'verlauf') -%} 

  {%- if anrufe -%}
    {%- set ns = namespace(zaehler=0) -%}
    {%- for anruf in anrufe -%}
      {%- if ns.zaehler < max_eintraege -%}
        {%- if anruf.wert not in ['unknown', 'unavailable', ''] -%}
          {%- set ns.zaehler = ns.zaehler + 1 -%}
          {%- set anruf_ts = as_timestamp(anruf.zeit) -%}
          {%- set jetzt_ts = as_timestamp(now()) -%}
          
          {# Zeitdifferenzen und Datum für den Heute-Vergleich ermitteln #}
          {%- set diff_minuten = ((jetzt_ts - anruf_ts) / 60) | int -%}
          {%- set diff_tage = ((jetzt_ts - anruf_ts) / 86400) | int -%}
          {%- set heute_datum = jetzt_ts | timestamp_custom('%Y-%m-%d') -%}
          {%- set anruf_datum = anruf_ts | timestamp_custom('%Y-%m-%d') -%}
          
          {{ anruf.wert | trim }}<br>
          
          {%- if diff_minuten < 60 and diff_minuten >= 0 -%}
            {%- if diff_minuten == 1 -%}vor 1 Minute
            {%- elif diff_minuten == 0 -%}gerade eben
            {%- else -%}vor {{ diff_minuten }} Minuten
            {%- endif -%}
          {%- else -%}
            {%- set wochentag = wochentage[anruf_ts | timestamp_custom('%w') | int] -%}
            
            {# 3-stufige Abfrage für Heute, 7-Tage und Älter #}
            {%- if heute_datum == anruf_datum -%}
              um {{ anruf_ts | timestamp_custom('%-H:%M') }} Uhr
            {%- elif diff_tage <= 6 -%}
              am {{ wochentag }}, {{ anruf_ts | timestamp_custom('%-H:%M') }} Uhr
            {%- else -%}
              am {{ wochentag }}, {{ anruf_ts | timestamp_custom('%-d.%-m. %-H:%M') }} Uhr
            {%- endif -%}
            
          {%- endif -%}
          
          {%- if ns.zaehler < max_eintraege and not loop.last -%}
          <hr>
          {%- endif -%}
        {%- endif -%}
      {%- endif -%}
    {%- endfor -%}
    
    {%- if ns.zaehler == 0 %}
      Keine gültigen Anrufe.
    {%- endif %}
  {%- else %}
    Keine Anrufe.
  {%- endif %}
```

### Optional:
[card-mod](https://github.com/thomasloven/lovelace-card-mod)

Beispiel, wie sich die Farbe und der Abstand der Trennlinie anpassen lassen sowie die Höhe der Karte begrenzt wird, damit der Inhalt innerhalb der Karte scrollbar ist.
```
card_mod:
  style:
    ha-markdown $: |
      hr {
        border-top: 1px solid var(--primary-text-color) !important; /* Farbe Trennlinie */
        margin: 10px 0 !important; /* Abstand Trennlinie */
      }
      p {
        margin: 0; /* Abstand erste Trennlinie */
      }
    .: |
      ha-card {
        max-height: 15rem; /* maximale Höhe */
        overflow-y: auto;
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

