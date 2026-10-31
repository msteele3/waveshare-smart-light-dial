# Smart Light Dial

A reusable ESPHome starter for the [Waveshare ESP32-S3 Knob Touch LCD 1.8](https://www.waveshare.com/wiki/ESP32-S3-Knob-Touch-LCD-1.8). Turn the dial to choose one of three rooms, tap to see lighting presets, then pause for three seconds to apply one. Home Assistant handles the actual lights and scenes.

This repository owns the knob display and Home Assistant application, including the current bedroom/bathroom/living-room labels, nighttime screen sleep, and wake-on-interaction behavior. Network credentials, the API key, and the Home Assistant health-check address stay in ignored `secrets.yaml`. Review the entity IDs for your installation. No secrets or generated firmware are included.

## Hardware and software

- [Waveshare ESP32-S3 Knob Touch LCD 1.8](https://www.waveshare.com/product/esp32-s3-knob-touch-lcd-1.8.htm)
- Suitable USB-C cable and 5 V power source per Waveshare's documentation
- A Command Strip to attach it
- A 2.4 GHz Wi-Fi network
- [Home Assistant](https://www.home-assistant.io/) with light entities for the rooms you want to control
- [ESPHome Device Builder or CLI](https://esphome.io/install/) (use a version that supports encrypted OTA, currently 2026.9 or later)

## How it works

The dial runs ESPHome on its ESP32-S3. Its rotary pulses select a zone or preset; a tap switches between the room and preset views. After a three-second pause, it publishes a request through its encrypted ESPHome API. A Home Assistant automation translates the request into a scene or a light-off action. Home Assistant sends the selected mode and each light's on/off state back to the display.

The device does not contain a Home Assistant token or issue arbitrary service calls. It uses an ESPHome `select` entity as a narrow request channel.

## Setup

1. Clone this repository. Copy `secrets.yaml.example` to `secrets.yaml`, then fill in your **own** 2.4 GHz Wi-Fi SSID/password and a unique 32-byte base64 API key. Keep `secrets.yaml` private.
2. Edit the six `entity_id` entries near the end of `smart-light-dial.yaml` to match your own Home Assistant `input_select` helpers and `light` entities. The sample expects three helpers, one for each room. You can rename the three room labels everywhere in the YAML if you prefer. Do not put private details into a public fork.
3. Create the helpers from [`home-assistant/input_select.example.yaml`](home-assistant/input_select.example.yaml), or make equivalent helpers in Home Assistant's UI. Each helper needs the five exact options shown in the file.
4. Create scenes for each room/preset pair. The sample automation expects names such as `scene.bedroom_morning` and `scene.living_room_about_to_sleep`. Adjust those mappings if your scenes use other IDs.
5. Add [`home-assistant/automations.example.yaml`](home-assistant/automations.example.yaml) to your Home Assistant automations. Update the trigger's dial `select` entity ID if Home Assistant assigns a different one. Reload automations and helpers, or restart Home Assistant as appropriate.
6. From the repo directory, run `esphome config smart-light-dial.yaml` to validate the configuration. Connect the dial over USB and run `esphome run smart-light-dial.yaml` to compile and install. The first install needs a serial connection; later updates can use encrypted OTA.
7. Attach the dial with a Command Strip after routing the cable. Clean the surfaces and follow the strip's instructions.
8. Add the device to Home Assistant using the [ESPHome integration](https://www.home-assistant.io/integrations/esphome/). Use the API encryption key from your private `secrets.yaml` when prompted. Verify that turning the knob changes the zone, tapping opens presets, and waiting three seconds applies a scene.

If this is an already-flashed dial, read [ESPHome's OTA encryption migration steps](https://esphome.io/components/ota/esphome/) before changing its OTA configuration. The example here is intended for a new USB flash.

## Customize

The three room names, five preset names, and three-second dwell are defined directly in `smart-light-dial.yaml`. The light styles are defined by your Home Assistant scenes. Start by changing the Home Assistant entity IDs and automation mappings, then change the display labels if desired.

## Keep your private setup private

- Never commit `secrets.yaml`, `.esphome/`, generated firmware, backups, or Home Assistant exports. Firmware binaries can contain your credentials.
- Use a unique API key for each device. If credentials were previously exposed in a public repo, rotate them; deleting a commit does not reliably remove a secret from history.
- Check `git status` and `git diff --cached` before publishing your own changes. This repo's `.gitignore` covers common generated and secret files, but you still need to review new files.

See [ESPHome security best practices](https://esphome.io/guides/security_best_practices/) for current guidance.

## Reference

- [Waveshare hardware wiki](https://www.waveshare.com/wiki/ESP32-S3-Knob-Touch-LCD-1.8)
- [ESPHome CLI](https://esphome.io/guides/cli/)
- [Home Assistant ESPHome integration](https://www.home-assistant.io/integrations/esphome/)

## License

MIT. See [LICENSE](LICENSE).

## Coordinated CSI builds

[WiFiSensing](https://github.com/msteele3/wifi-sensing) owns the CSI external component and paired firmware releases. Its builder checks out a pinned commit of this application and merges a local CSI package. This application remains independently buildable without CSI; do not copy the CSI C++ implementation into this repository.

Use ESPHome **2026.10.1**. Set `ha_health_url` in `secrets.yaml` to your Home Assistant manifest URL. The display starts dark until Pacific time is available and uses the nighttime sleep/wake behavior from the deployed local application.
