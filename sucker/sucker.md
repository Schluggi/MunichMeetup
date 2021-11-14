# Ein Staubsauger und seine Probleme (und wie man sie löst)
In diesem Vortag möchte ich über die Erfahrungen und Probleme die ich mit meinem Roborock S50 hatte sprechen. 

- [Vortrag auf YouTube](https://www.youtube.com/watch?v=Gz9_gbex1FI)
- [Download der Präsentation](https://github.com/Schluggi/MunichMeetup/blob/main/sucker/2021-03.Ein%20Staubsauger%20und%20seine%20Probleme%20(und%20wie%20man%20sie%20l%C3%B6st).pdf)

## Inhaltsverzeichnis
- [Kaufentscheidung](#kaufentscheidung)
- [Probleme](#probleme)
  - [Die Cloud muss Weg](#die-cloud-muss-weg)
  - [Dunkle Teppiche](#dunkle-teppiche)
  - [Random Resets](#random-resets)
- [Home Assistant & Automatisierungen](#home-assistant--automatisierungen)
  - [Integration via MQTT (empfohlen)]()
  - [Integration via Miio]()
  - [Automatisierungen](#automatisierungen)
- [Link-Sammlung](#link-sammlung)

> **Update Nov. 2021:** Mitlerweile hat sich [Valetudo von Hypfer](https://github.com/Hypfer/Valetudo/) so schnell weiterentwickelt, dass meine Empfehlung klar bei diesem liegt. Außerdem benötigt man für die Map-Darstellung nun keinen mapper, sondern kann dies direkt über die entsprechende [Card](https://github.com/TheLastProject/lovelace-valetudo-map-card) tun.

## Kaufentscheidung
Gut, günstig und ohne Cloud. Das waren die Anforderungen an meinen Saugroboter. Gerade der letzte Punkt schien zur damaligen Zeit (August 2019) unmöglich. 
Zwar gab es 2-3 Sauger die auch ohne Cloud auskamen, aber diese hatten dann meist nur eine Fernbedienung und schnitten in dem meisten Tests nicht sonderlich gut ab.
Via Zufall bin ich bei der Recherche bei YouTube auf [dieses Video](https://youtu.be/uhyM-bhzFsI) des CCC gestoßen. Zwei Hacker zeigen dort, wie sie einen Roborock geknackt 
und ihn Cloud-los bekommen haben. Am Ende des Videos wird auf das [DustCloud-Projekt](https://github.com/dgiese/dustcloud) hingewiesen, das aus ihren Arbeiten heraus entschaden ist.
An diesem Punkt war für mich die Entscheidung gefallen und ich kaufte mir ein Roborock S50 für ~480€ auf Amazon.

## Probleme
### Die Cloud muss Weg
Wie Dennis Giese und Daniel Wegemer auf der [34C3 zeigten](https://media.ccc.de/v/34c3-9147-unleash_your_smart-home_devices_vacuum_cleaning_robot_hacking), gibt es gute Gründe seinen Staubsauger mit einer OpenSource
Firmware auszustatten. Das im Video angesprochene [DustCloud-Projekt](https://github.com/dgiese/dustcloud) war leider nicht ganz so, wie ich mir das gedacht hatte. Einfach nicht sinnvoll nutzbar. 
Allerdings fand ich ein anderes, offenbar komplett unabhängig von DustCloud entwickeltes, Projekt namens [Valetudo](https://github.com/Hypfer/Valetudo/). Ich flashte es und war begeistert davon, wie einfach es war. 
Später bin ich aufgrund einiger Fehler und der langsamen Entwicklung auf den [Fork von rand256] (https://github.com/rand256/valetudo) umgestiegen. Mittlerweile unterstützen beide Projekte mehr als nur einen Sauger. Aus Bequemlichkeit bin ich noch immer bei dem Fork, obwohl Hypfer mittlerweile aktiv an dem Projekt weitergearbeitet hat.

### Dunkle Teppiche
Wie ich feststellen musste, kommt der Roborock nicht so gut mit dunklen Teppichen klar (er saugt sie nicht), da die Klippen-Sensoren, den Teppich fälschlicher Weise als Abgrund erkennen. 
Der Klippen-Sensor sendet ein Infrarot-Signal aus und schaut, ob dieses reflektiert wird. Empfängt er keine Reflektion, geht er von einer Stufe / einem Abgrund aus.
Da dunkle Teppiche das Licht und somit auch das Infrarot-Signal recht gut absorbieren, kommt es zur Fehleinschätzung. 
Solltest du keine Stufen in deiner Wohnung haben, wo der Roboter herunterfallen kann, ist die Lösung recht simpel: Sensoren Abkleben.
Falls du zugriff auf einen 3D Drucker hast, kannst du dir auch [Abdeckungen drucken](https://www.thingiverse.com/thing:3103343). 

### Random Resets
Sollte es vorkommen, dass sich dein Staubsauger nach einiger Zeit (bei mir waren es meist so 2-3 Monate) zurücksetzt, kann dies mit dem Watchdog zusammenhängen.
Sowohl in der Firmware von Hypfer als auch in der von rand256 sollte das Problem mittlerweile behoben sein.
Solltest du dennoch Probleme haben, kann dir [dieser Github-Kommentar](https://github.com/Hypfer/Valetudo/issues/206#issuecomment-498132355) vermutlich weiterhelfen.

## Home Assistant & Automatisierungen 
### Integration via MQTT (empfohlen)
(folgt, siehe Aufzeichnung)

### Integration via Miio
Das Integrieren des Roborocks in Home Assistant ist sehr leicht. Hierzu wird der Token des Saugers benötigt, welchen wir im Webfrontend unter
`Settings -> Info -> Token` finden. Die Konfiguration sieht dann wie folgt aus:
```yaml
vacuum:
  - platform: xiaomi_miio
    name: sucker
    host: 192.168.13.4
    token: 748293425487239402342346
```
Nun sollte eine Entität namens `vacuum.sucker` auftauchen. 
Damit das ganze in Lovelace auch schön aussieht, noch flott die [Custom Card](https://github.com/benct/lovelace-xiaomi-vacuum-card) geladen und eingerichtet.
Es gibt zudem eine Custom Card, um die verschiedene Zonen anfahren zu können. Diese findest du [hier](https://github.com/PiotrMachowski/lovelace-xiaomi-vacuum-map-card).

### Automatisierungen
Da wir unseren unsere eigenen Automatisierungen nutzen wollen, schalten wir als erstes alle Zeitpläne im Webfrontend des Saugers ab.
Anschließend erstellen wir einen `input_boolean` in Home Assistant.
```yaml
input_boolean:
  suck_my_flat:
    name: Suck my Flat
    initial: off
    icon: mdi:robot-vacuum
```
Nun überlegen wir uns einen Zeitplan, wie regelmäßig ca. gesaugt werden soll und bauen uns eine Automatisierung, die den gerade angelegten `input_boolean` entsprechend einschaltet.
```yaml
alias: Sucker (routine)
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
    target:
      entity_id: input_boolean.suck_my_flat
mode: single
```
In diesem Beispiel soll also jeden Dienstag und Freitag um 9Uhr gesaugt werden.
<br><br>
Damit er auch wirklich saugt, legen wir nun eine weitere Automatisierung an, die jede Stunde schaut, ob gesaugt werden soll. 
Außerdem soll nur gesaugt werden, wenn wir mindestens bereits eine Stunde nicht mehr zuhause waren.
Dies soll verhindern, dass der Sauger nicht direkt anspringt, wenn wir z.B. den Müll raus bringen oder nur kurz nach der Post sehen.
```yaml
alias: Sucker (suck my flat)
trigger:
  - platform: time_pattern
    hours: '*'
    minutes: '0'
    seconds: '0'
condition:
  - condition: state
    entity_id: person.lukas
    state: not_home
    for: '01:00:00'
  - condition: time
    after: '10:30:00'
    before: '20:00:00'
  - condition: state
    entity_id: input_boolean.suck_my_flat
    state: 'on'
action:
  - service: input_boolean.turn_off
    target:
      entity_id: input_boolean.suck_my_flat
  - service: vacuum.start
    entity_id: vacuum.sucker
mode: single
```
Eine time condition soll verhindern, dass Nachts gesaugt wird.
Mit dem Starten des Saugvorgangs schalten wir zudem unser `input_boolean` wieder aus.
<br><br>
Aktuell kann es noch passieren, dass wir nach einem harten Tag nach Hause kommen und der Sauger (je nach Zeitplan) gerade saugt (super nervig -.-).
Daher habe basteln wir uns noch flott eine eine dritte Automatisierung, welche das Saugen abbricht und später wiederholt, sobald wir nach Hause kommen:
```yaml
alias: Sucker (abort)
trigger:
  - platform: state
    entity_id: person.lukas
    to: home
condition:
  - condition: state
    entity_id: vacuum.sucker
    state: cleaning
action:
  - service: input_boolean.turn_on
    target:
      entity_id: input_boolean.suck_my_flat
  - service: vacuum.stop
    entity_id: vacuum.sucker
  - delay: '00:00:03'
  - service: vacuum.return_to_base
    entity_id: vacuum.sucker
mode: single
```
Das Delay ist notwendig, da der Sauger immer nur ein Input verarbeiten kann.
Ohne Delay würde er also nur aufhören zu saugen, aber nicht zurück in die Dock fahren.

## Link-Sammlung
- Vortrag von Dennis Giese und Daniel Wegemer auf der 34C3
  - Beitrag und Video des CCC: https://media.ccc.de/v/34c3-9147-unleash_your_smart-home_devices_vacuum_cleaning_robot_hacking
  - YouTube (englisches Original): https://youtu.be/uhyM-bhzFsI
  - YouTube (deutsche Übersetzung): https://youtu.be/wKVZqkXUypo
  - DustCloud: https://github.com/dgiese/dustcloud
- Roborock Vacuum Cleaner 2 (S50) bei Amazon: https://www.amazon.de/gp/product/B07GQN4VM8
- Klippensensorenabdeckung zum selber drucken: https://www.thingiverse.com/thing:3103343
- Lösung für die random Resets: https://github.com/Hypfer/Valetudo/issues/206#issuecomment-498132355
- Open-Source Firmware
  - Von Hypfer
    - Firmware: https://github.com/Hypfer/Valetudo/
    - Map-Converter: https://github.com/Hypfer/ICantBelieveItsNotValetudo (wird in der aktuellen Version nicht mehr benötigt)
    - MQTT-Doku: https://valetudo.cloud/pages/integrations/mqtt.html
  - Fork von rand256
    - Firmware: https://github.com/rand256/valetudo
    - Map-Converter: https://github.com/rand256/valetudo-mapper
    - MQTT-Doku: https://github.com/rand256/valetudo/wiki/MQTT-API
- Home Assistant
  - Integration via Miio: https://www.home-assistant.io/integrations/xiaomi_miio/
  - Custom Lovelace Card (Vacuum Card): https://github.com/denysdovhan/vacuum-card
  - Custom Lovelace Card (XIAOMI Vacuum Card): https://github.com/benct/lovelace-xiaomi-vacuum-card
  - Custom Lovelace Card (XIAOMI Vacuum Map Card): https://github.com/PiotrMachowski/lovelace-xiaomi-vacuum-map-card
  - Custom Lovelace Card (Valetudo Map Card): https://github.com/TheLastProject/lovelace-valetudo-map-card
 - MQTT-Explorer: https://mqtt-explorer.com/
