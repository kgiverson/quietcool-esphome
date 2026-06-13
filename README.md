# QuietCool ESPHome Control

> [!NOTE]
> The QuietCool Smart Attic Fan Control works with both whole house fans and attic fans.  In my application, I used it for a whole house fan.  This project requires having a Smart Attic Fan Control, which can be purchased separately from QuietCool and is a drop-in replacement for the RF controller.

This repository contains ESPHome configurations for controlling QuietCool whole house fans and attic fans. QuietCool fans can be automated and integrated into your smart home using ESP32 devices flashed with these configurations.

The project provides a web-based installation interface using [ESP Web Tools](https://esphome.github.io/esp-web-tools/), making it easy to flash your ESP device directly from your browser.

For the tested M5Stack ATOM Lite setup, confirmed V2 BLE protocol, known
limitations, and session resume notes, see [SESSION_HANDOFF.md](SESSION_HANDOFF.md).

## Configurations

### quietcool-smart-attic-fan-control.yaml

This configuration file provides smart control for QuietCool attic fans. Features include:

- Manual two- or three-speed control
- Configurable timer control
- Integration with Home Assistant
- Real-time temperature and humidity monitoring
- BLE pairing and status monitoring

### M5Stack ATOM Lite

The `quietcool-m5stack-atom-lite.yaml` configuration runs this project as an
external BLE-to-Wi-Fi bridge on an M5Stack ATOM Lite. It does not replace or
require wiring to the QuietCool controller. Place the ATOM Lite within reliable
Bluetooth range of the fan controller.

This configuration enables three-speed control and uses the numeric JSON API
required by newer QuietCool controller firmware.

To install it with the ESPHome Device Builder in Home Assistant:

1. Connect the ATOM Lite to the Home Assistant host with a data-capable USB-C
   cable.
2. Create a new ESPHome device and choose the connected USB serial port.
3. Replace the generated configuration with `quietcool-m5stack-atom-lite.yaml`.
4. Set `mac_address` to the Bluetooth MAC address shown by the QuietCool app.
5. Keep the generated `wifi`, `api`, and `ota` credentials if you want encrypted
   API access and a configured Wi-Fi network.
6. Install over USB. Later updates can be installed wirelessly.
7. Add the device to Home Assistant, put the QuietCool controller into pairing
   mode with its app, and press the device's **Pair BLE** button in Home
   Assistant.

The pairing ID is an arbitrary 16-character hexadecimal string. The default is
valid, but you can change it using the **Pair ID** entity before pairing.

### Home Assistant controls

- **QuietCool Fan** is the sole on/off control. Turning it on starts the selected
  Operating Mode; turning it off stops the fan.
- **Manual Speed** provides an explicit Low, Medium, or High dropdown.
- On V2 three-speed controllers, Manual Run Medium uses a 23-hour-59-minute
  timer fallback because the continuous-run speed command only accepts Low and
  High. Re-select Medium to restart that period.
- **Operating Mode** selects Manual Run, Timer Run, or Smart Temperature &
  Humidity. Changing it while off only changes the next mode; changing it while
  on switches modes immediately.
- **Timer Hours** and **Timer Minutes** configure the Timer Run duration.
- **Smart Temperature & Humidity** uses the temperature and humidity thresholds
  currently stored in the QuietCool controller. Applying a preset in the app
  writes its values to these active thresholds; the controller does not appear
  to reference a named preset while running.
- A **Pair State** value of `No` is normal after pairing; it means the controller
  is not currently accepting new pairings.

## Getting Started

1.  Pair your phone with the Smart Attic Fan Control.
2.  Retrieve the MAC address of your controller from the QuietCool app.
3.  Flash an ESP32 using this template.  See [Packages](https://next.esphome.io/components/packages) for details on how to use this repository in your ESPHome configuration.  The gist is that you can simply add the following YAML to your configuration:

``` yaml
substitutions:
    mac_address: "00:00:00:00:00:00" # Replace with your Smart Attic Fan Control's MAC address

packages:
  remote_package_shorthand: github://awkaplan/quietcool-esphome/quietcool-smart-attic-fan-control.yaml@main
```

4.  Add the ESPHome device to Home Assistant and update the pairing ID with a unique, 16-digit hex string.
5.  Use the QuietCool app to put the controller in pairing mode.
6.  Press the ESPHome device's pair button in Home Assistant.
7.  Forget the bluetooth device on your phone (optional).

## Credits

This project builds upon the research and development done by [@emerose](https://github.com/emerose/quietcool), who created a Python-based BLE client for the QuietCool Wireless RF Control Kit. Their work was instrumental in understanding the QuietCool BLE protocol and making this ESPHome integration possible.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
