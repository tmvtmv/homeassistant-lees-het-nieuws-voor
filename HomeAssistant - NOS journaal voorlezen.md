# HomeAssistant - Lees het nieuws voor

## Aanleiding

Ik vind het fijn als het NOS journaal door een mens wordt voorgelezen. Na wat zoekwerk bleek dat er tegenwoordig bijna geen streams meer  beschikbaar zijn. Op de website van [NPO Radio 1](https://www.nporadio1.nl/) zag ik onderin de knop "**Laatste journaal**" staan. Die heb ik samen met [Claude Code](https://claude.com/product/claude-code) i.c.m. [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp/) uitgeplozen want dat gaat gewoon lekker snel ;-)

![nporadio1](HomeAssistant - NOS journaal voorlezen.assets/nporadio1.png)

Mijn [Home Assistant Voice](https://www.home-assistant.io/voice-pe/) leest nu het meest recente journaal voor zodra ik erom vraag.

## Benodigd

Zorg ervoor dat deze integration is geinstalleerd in jouw HomeAssistant

* https://www.home-assistant.io/integrations/rest_command/
* Eén of meerdere [Home Assistant Voice](https://www.home-assistant.io/voice-pe/)

## Configureer het REST-commando

Open `configuration.yaml` en voeg in sectie `rest` onderstaande YAML-code toe.
Deze code haalt de tijdelijke URL op 

```yaml
rest:
    - resource: https://www.nporadio1.nl/api/latestNews
      headers:
        Referer: https://www.nporadio1.nl/
        User-Agent: Mozilla/5.0
      scan_interval: 0
      sensor:
        - name: "NPO Radio 1 Latest News"
          value_template: "OK"
          json_attributes_path: "$.player.parameters[0]"
          json_attributes:
            - value
```



## Automation

Maak een nieuwe automation aan. In dit geval reageert Home Assistant Voice op de vragen "Wat is het nieuws?" en "Wat is het laatste nieuws?".

```yaml
alias: Lees het laatste nieuws voor
description: ""
triggers:
  - trigger: conversation
    command:
      - Wat is het nieuws
      - Wat is het laatste nieuws
conditions: []
actions:
  - set_conversation_response: ""
  - action: homeassistant.update_entity
    target:
      entity_id: sensor.npo_radio_1_latest_news
  - delay:
      seconds: 1
  - action: media_player.play_media
    data:
      media:
        media_content_id: "{{ state_attr('sensor.npo_radio_1_latest_news', 'value') }}"
        media_content_type: music
    target:
      entity_id: >-
        media_player.{{ trigger.satellite_id | replace('assist_satellite.', '')
        | replace('_assist_satellite', '') }}
mode: single

```

> **Toelichting** `media_player`:
> Om ervoor te zorgen dat de juiste Home Assistant Voice (degene die getriggered is door de vraag) de  journaal-mp3 gaat afspelen moest ik de waarde van variabele `trigger.satellite` ontdoen van: `assist_satellite` en `_assist_satellite`.  Zo kom ik tot de juiste entity_id van de media_player.
>  In mijn geval is dat:
> `media_player.voiceassistant_studiekamer` of `media_player.voiceassistant_woonkamer`. 
>
> Als je erg afwijkt van de standaard naamgevingsconventie kan het zijn dat je hier nog iets in moet aanpassen.



## Trace

De trace van deze automatisering ziet er dan zo uit:

![nieuws-trace](HomeAssistant - NOS journaal voorlezen.assets/nieuws-trace.png)

