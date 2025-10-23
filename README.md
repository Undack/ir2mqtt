# ir2MQTT

IR2MQTT is a custom firmware for **generic BK7231N-QFN32 Tuya IR devices**, enabling direct **IR input and output** communication with **Home Assistant** via **MQTT**. This allows control of IR appliances which many projects can do, but also enables existing IR remotes to interact with smart home devices—functionality most IR hubs lack, Finally, the project will automatically enroll codes into home assistant, making deployment easy.

<img src="images/WhatsApp Image 2025-02-28 at 11.40.20.jpeg" alt="Remotes, various, previously e-waste can now be used as cheap wireless controllers" width="200"/><img src="images/WhatsApp Image 2025-02-28 at 11.40.23(1).jpeg" alt="Remotes, various, previously e-waste can now be used as cheap wireless controllers" width="200"/><img src="images/WhatsApp Image 2025-02-28 at 11.40.23.jpeg" alt="Remotes, various, previously e-waste can now be used as cheap wireless controllers" width="200"/>

*Figure 1: Remotes, various, previously e-waste can now be used as cheap wireless controllers*

<img src="images/WhatsApp Image 2025-03-01 at 00.22.50.jpeg" alt="Various cheap led devices and most AV equipment can be controlled via IR remote" width="200"/><img src="images/WhatsApp Image 2025-03-01 at 00.22.50(1).jpeg" alt="Various cheap led devices and most AV equipment can be controlled via IR remote" width="200"/><img src="images/WhatsApp Image 2025-03-01 at 00.22.51.jpeg" alt="Various cheap led devices and most AV equipment can be controlled via IR remote" width="200"/>

*Figure 2: Many cheap led devices and most AV equipment can be controlled via IR remote*

## Features

- Receive IR codes from remotes and publish them to mqtt
- Receive IR codes from mqtt and transmit them via IR
- Automatically add binary sensors for receiving and button for transmitting each detected code to home assistant

### Supported Devices

This firmware is compatible with cheap Tuya wifi IR blasters, it has been tested on the following model specifically but would likely work with similar esphome flashable devices with minimal reconfiguration

<img src="images/WhatsApp Image 2025-02-28 at 11.40.24.jpeg" alt="The Tuya wifi IR blaster." width="600"/>
*Figure 3: The Tuya wifi IR blaster.*


Commonly whitelabeled and resold, it can be found on aliexpress for around £5:

- [Device 1](https://www.aliexpress.com/item/1005008188667566.html)
- [Device 2](https://www.aliexpress.com/item/1005007588675238.html)

## Prerequisites & Tools

### Hardware:

- Tuya IR device with a **BK7231N chipset**
- A **computer or Raspberry Pi** (to run Tuya CloudCutter)

### Software:

- [Tuya CloudCutter](https://digiblur.com/2023/08/19/updated-tuya-cloudcutter-with-esphome-bk7231-how-to-guide/) (for unlocking the device)
- **Home Assistant** (for integration)
- **HA ESPHome integration** (to compile and flash the firmware)
- **HA MQTT integration** (for communications)

## Installation Guide

### Step 1: Unlock Device & Install Kickstart Firmware

1. Run Tuya CloudCutter on your Linux machine or Raspberry Pi.
2. Select:
   ```txt
   2) Flash third-party firmware
   2) Select by firmware device and name
   2.0.0 - BK7231N / oem_bk7231n_irbox_mol_ty
   ```
3. Once the **kickstart firmware** is installed, the device will create a **temporary Wi-Fi access point**.
   
- The **kickstart firmware** will NOT brick the device if the process fails—it can be retried.

## Step 2: Flash the ESPHome Firmware

### ESPHome Firmware Builder

1. Create a **new ESPHome device** in Home Assistant.
2. Copy the contents of the `v12.yml` file to the ESPHome configuration.
3. Each IR remote must have a unique name (substitutions, globals, esphome fields in v12.yml).
4. Edit `secrets.yaml`:
   ```yaml
   mqtt_broker: "your_broker_ip"
   mqtt_username: "your_username"
   mqtt_password: "your_password"
   ```
5. Compile and download the firmware by selecting manual download and then uf2 (reccommended)

6. Access the **kickstart firmware’s wifi network** and **kickstart firmware’s web interface** and upload the compiled **UF2 file**.

## Step 3: Configure Home Assistant

Once the ESPHome firmware is installed, the device will:

Automatically create a new device with the name as entered in the yaml in the mqtt integration, and will automatically populate with new codes as button and binary sensor entities as they are detected. It can be helpful to rename the entities at this stage, be sure to rename both the button and the sensor to avoid confusion later.

Simply point at the device and press a button on a remote in order to save it or the devices it controls for automation, if no remote is available for a device it can be possible to find a suitable controller with one of the many ir universal remote apps available 

<img src="images/Screenshot 2025-03-08 174647.png" alt="Automatically created binary sensor and button for detected code, with a renamed pair above" width="600"/>
*Figure 4: Automatically created binary sensor and button for detected code, with a renamed pair above.*



### Example Home Assistant UI (Lovelace YAML)

```yaml
type: vertical-stack
title: Big Candle IR Remote
cards:
  - type: button
    name: Big Candle ON
    icon: mdi:fire
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: your_entity_id
  - type: button
    name: Big Candle 6H Timer
    icon: mdi:clock-outline
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: your_entity_id
  - type: button
    name: Big Candle OFF
    icon: mdi:fire-off
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: your_entity_id
```

placing these elements within a horizontal stack can be used to recreate broader remotes with smaller buttons to make better use of available ui space.

<img src="images/Screenshot 2025-02-28 130632.png" alt="In the lovelace UI" width="200"/>
*Figure 5: In the lovelace UI*

### Example Home Assistant Automation

```yaml
alias: Example Automation
trigger:
  - platform: state
    entity_id: binary_sensor.example_sensor
    to: "on"
  - platform: state
    entity_id: binary_sensor.example_sensor
    to: "off"
action:
  - choose:
      - conditions:
          - condition: state
            entity_id: binary_sensor.example_sensor
            state: "on"
        sequence:
          - service: light.turn_on
            target:
              entity_id: light.example_light
      - conditions:
          - condition: state
            entity_id: binary_sensor.example_sensor
            state: "off"
        sequence:
          - service: light.turn_off
            target:
              entity_id: light.example_light
mode: single
```

## Finding RX and TX pins on devices

Some batches of this module use different pins for RX (receive) and TX (transmit). To find which pins your device actually uses, we provide two helper scripts: rx_finder and tx_finder.

### RX Finder

1. Copy the contents of the `rx_finder.yml` file to the ESPHome configuration (remember to ensure that each IR remote has a unique name as in step 2 of initial flashing, no changes to secrets.yaml are required for finding pins).
2. Compile and write the firmware to the device, either wirelessly, or manually via the device's local page.
3. Open the device’s local web page — either via its IP/hostname (reccommended, check your router dhcp allocation if you cannot find) or Home Assistant’s esphome device list.
4. Point any IR remote at the device and press a button.

<img src="images/Untitled.jpg" alt="Point any IR remote at the device and press a button" width="600"/>

5. On the web interface, on the left side table You’ll see the pin changing state, and on the right you'll see the same reported to the console.

<img src="images/Screenshot 2025-10-23 095155.png" alt="Detecting in the UI" width="600"/>

6. Note the pin name (e.g. P7, P6, P26).
7. Copy that pin into the remote_receiver section of your main v12.yaml, under the field named number (make sure it is within the quotes, like "P9").
8. Reflash the v12.yml script with the rx (and/or tx pins changed), the device should automatically report codes to homeassistant, and you should see code publishing reported in the console on the device's local page.

### TX Finder

1. Copy the contents of the tx_finder.yml` file to the ESPHome configuration (steps 1, 2, and 3 here are identical to rx_finder).
2. Compile and write the firmware to the device, either wirelessly, or manually via the device's local page.
3. Open the device’s local web page — either via its IP/hostname or Home Assistant’s esphome device list.

<img src="images/Screenshot 2025-10-23 102834.png" alt="The TX finder UI" width="600"/>

4. Grab a phone camera or similar (IR LEDs are invisible to the naked eye).
5. Press each button on the web page (or in Home Assistant) one by one, Watch through your camera — when you see the LEDs come on, you’ve found your TX pin.

<img src="images/Blinky.jpg" alt="The IR blaster with LEDs lit" width="600"/>

6. Note the pin name (e.g. P7, P6, P26).
7. Copy that pin into the remote_transmitter section of your main v12.yaml, under the field named pin (no quotes are needed here).
8. Reflash the v12.yml script with the tx (and/or rx pins changed), the device should transmit any codes sent from homeassistant, and you should see code reception and transmission reported in the console on the device's local page.

### If Nothing Works

If neither script detects any pins, your hardware revision may use nonstandard pins not listed in the finder scripts.
You can manually edit the pin lists in rx_finder and tx_finder to test the remaining available GPIOs.
On the generic-bk7231n-qfn32-tuya board you have these usable GPIOs to cycle through:
```
P0, P1, P6, P7, P8, P9, P10, P11, P14, P15, P16, P17, P20, P21, P22, P23, P24, P26, P28
```
[libretiny wiki](https://docs.libretiny.eu/boards/generic-bk7231n-qfn32-tuya/#usage)
If that still fails — it’s probably an incompatible module. Consider it unsupported (for now).

## Additional Resources

- Icon Library for UI Cards and IR remote icons: [MDI Icons](https://pictogrammers.com/library/mdi/)
- Included is the file remote_label_v9, for the custom remote in figure 1, this is for printing on a 62mm sticker printer, and should have the dotted corners rounded before being attached to the following remote [Remote](https://www.aliexpress.com/item/1005005923594477.html) or similar
- https://esphome.io/components/libretiny.html
- https://github.com/tuya-cloudcutter/tuya-cloudcutter
- The gateway works best when laying flat pointed toward the ceiling, so that the IR light can bounce back down evenly 

## Potential future work (No promises)

- IRdb integration to automtically group codes within a device, create UI cards automatically, suggest related codes etc...: [IRdb](https://github.com/probonopd/irdb)

