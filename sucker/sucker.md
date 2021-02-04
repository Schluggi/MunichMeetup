# Ein Staubsauger und seine Probleme (und wie man sie löst)
- Vortrag auf YouTube: (folgt)
- Download der Präsentation: (folgt)

## Inhaltsverzeichnis
- Einleitung
- Kaufentscheidung
- Probleme
- Home Assistant & Automatisierungen
- Link-Sammlung

## Einleitung 

## Kaufentscheidung

## Probleme
### Die Cloud muss Weg
### Dunkle Teppiche
### Random Resets


## Home Assistant & Automatisierungen 
### Integration
Das Integrieren des Roborocks in Home Assistant ist sehr leicht. Hierzu wird der Token des Saugers benötigt, welchen wir im Webfrontend unter
`Settings -> Info -> Token` finden. Die Konfiguration sieht dann wie folgt aus:
```yaml
vacuum:
  - platform: xiaomi_miio
    name: sucker
    host: 192.168.13.4
    token: 748293425487239402342346
```
Nun solltest eine Entität namens `vacuum.sucker` auftauchen. 

### Automatisierungen
Da wir unseren unsere eigenen Automatisierungen nutzen wollen, schalten wir als erstes alle Zeitpläne im Webfrontend des Saugers ab.
Anschließend erstellen wir einen `input_boolean` in Home Assistant.
```yaml
input_boolean:
  sucker_scheduling:
    name: Suck my Flat
    initial: off
    icon: mdi:robot-vacuum
```
Nun überlegen wir uns einen Zeitplan, wann gesaugt werden soll und bauen uns eine Automatisierung, die den gerade angelegten `input_boolean` entsprechend einschaltet.
```yaml
  - alias: Sucker (routine)
    trigger:
      - platform: time
        at: '9:00:00'
    condition:
      - condition: time
        weekday:
          - tue
          - fri
    action:
      - service: input_boolean.turn_on
        entity_id: input_boolean.sucker_scheduling
```
In diesem Beispiel soll also jeden Dienstag und Freitag um 9Uhr gesaugt werden.
<br><br>
Damit er auch wirklich saugt, legen wir nun eine weitere Automatisierung an, die jede Stunde schaut, ob gesaugt werden soll. 
Außerdem soll nur gesaugt werden, wenn wir mindestens bereits eine Stunde nicht mehr zuhause waren.
Dies soll verhindern, dass der Sauger nicht direkt anspringt, wenn wir z.B. den Müll raus bringen oder nur kurz nach der Post sehen.
```yaml
 - alias: Sucker (suck my flat)
    trigger:
      - platform: time_pattern
        hours: '*'
        minutes: 0
        seconds: 0
    condition:
      - condition: state
        entity_id: binary_sensor.presence_lukas
        state: 'off'
        for: '1:00:00'
      - condition: time
        after: '10:30:00'
        before: '22:00:00'
      - condition: state
        entity_id: input_boolean.sucker_scheduling
        state: 'on'
    action:
      - service: input_boolean.turn_off
        entity_id: input_boolean.sucker_scheduling
      - service: vacuum.start
        entity_id: vacuum.sucker
```
Eine time condition soll verhindern, dass Nachts gesaugt wird.
Mit dem Starten des Saugvorgangs schalten wir zudem unser `input_boolean` wieder aus.
<br><br>
Aktuell kann es noch passieren, dass wir nach einem harten Tag nach Hause kommen und der Sauger (je nach Zeitplan) gerade saugt (super nervig -.-).
Daher habe basteln wir uns noch flott eine eine dritte Automatisierung, welche das Saugen abbricht und später wiederholt, sobald wir nach Hause kommen:
```yaml
  - alias: Sucker (abort) 
    trigger:
      - platform: state
        entity_id: binary_sensor.presence_lukas
        to: 'on'
    condition:
      - condition: state
        entity_id: vacuum.sucker
        state: 'cleaning'
    action:
      - service: input_boolean.turn_on
        entity_id: input_boolean.sucker_scheduling
      - service: vacuum.stop
        entity_id: vacuum.sucker
      - delay: '00:00:03'
      - service: vacuum.return_to_base
        entity_id: vacuum.sucker
```
Das Delay ist notwendig, da der Sauger immer nur ein Input verarbeiten kann.
Ohne Delay würde er also nur aufhören zu saugen, aber nicht zurück in die Dock fahren.

## Link-Sammlung
- Vortrag von Dennis Giese und Daniel Wegemer auf der 34C3
  - https://media.ccc.de/v/34c3-9147-unleash_your_smart-home_devices_vacuum_cleaning_robot_hacking
  - YouTube (englisches Original): https://youtu.be/uhyM-bhzFsI
  - YouTube (deutsche Übersetzung): https://youtu.be/wKVZqkXUypo
- Roborock Vacuum Cleaner 2 (S50) bei Amazon
  - https://www.amazon.de/gp/product/B07GQN4VM8
- Open-Source Firmware
  - Original von Hypfer: https://github.com/Hypfer/Valetudo/
  - Fork von rand256: https://github.com/rand256/valetudo
- Klippensensorenabdeckung zum selber drucken
  - https://www.thingiverse.com/thing:3103343
- Lösung für die random Resets
  - https://github.com/Hypfer/Valetudo/issues/206#issuecomment-498132355
- Home Assistant
  - Integration: https://www.home-assistant.io/integrations/xiaomi_miio/
  - Custom Lovelace Card für den Sauger: https://github.com/benct/lovelace-xiaomi-vacuum-card
  - Custom Lovelace Card für die Zonen: https://github.com/PiotrMachowski/lovelace-xiaomi-vacuum-map-card
