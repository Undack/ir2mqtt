# ir2MQTT

IR2MQTT is a custom firmware for **generic BK7231N-QFN32 Tuya IR devices**, enabling direct **IR input and output** communication with **Home Assistant** via **MQTT**. This allows control of IR appliances and enables existing IR remotes to interact with smart home devices—functionality most Tuya IR hubs lack.

## Features

- Manage all IR codes easily.
- Receive and store IR codes automatically.
- Copy MQTT topics for use in external applications.
- Send IR codes directly to hardware.

### Supported Devices

This firmware is compatible with budget-friendly Tuya IR blasters like:

- [Device 1](https://www.aliexpress.com/item/1005007335703208.html)
- [Device 2](https://www.aliexpress.com/item/1005008188667566.html)

## Prerequisites & Tools

### Hardware:

- Tuya IR device with a **BK7231N chipset**
- A **computer or Raspberry Pi** (to run Tuya CloudCutter)
- A **USB-to-UART adapter** (if needed for kickstart flashing)

### Software:

- [Tuya CloudCutter](https://digiblur.com/2023/08/19/updated-tuya-cloudcutter-with-esphome-bk7231-how-to-guide/) (for unlocking the device)
- **ESPHome** (to compile and flash the firmware)
- **Home Assistant** (for integration)

## Installation Guide

### Step 1: Unlock Device & Install Kickstart Firmware

1. Run Tuya CloudCutter on your Linux machine or Raspberry Pi.
2. Select:
   ```txt
   2) Flash third-party firmware
   2) Select by firmware device and name
   2.0.0 - BK7231N / oem_bk7231n_irbox_mol_ty
   ```
3. Once the **kickstart firmware** is installed, the device will create a **temporary Wi-Fi access point**. Connect to it.

## Step 2: Flash the ESPHome Firmware

### ESPHome Firmware Builder

1. Create a **new ESPHome device** in Home Assistant.
2. Place the `v10.yml` file in the ESPHome configuration.
3. Edit `secrets.yaml`:
   ```yaml
   mqtt_broker: "your_broker_ip"
   mqtt_username: "your_username"
   mqtt_password: "your_password"
   ```
4. Compile and download the firmware:
   ```sh
   esphome run v10.yml
   ```
5. Access the **kickstart firmware’s web interface** and upload the compiled **HEX file**.

## Step 3: Configure Home Assistant

Once the ESPHome firmware is installed, the device will:

- Automatically create binary sensors and buttons to receive and send codes from any detected IR remote in home assistant, these can be found via the event log, when the code is detected, or in the entities tab, under the parent device.

### Example Home Assistant UI (Lovelace YAML)

```yaml
type: vertical-stack
title: Big Candle IR Remote
cards:
  - type: button
    name: Big Candle ON
    icon: mdi:fire
    tap_action: {action: toggle}
  - type: button
    name: Big Candle 6H Timer
    icon: mdi:clock-outline
    tap_action: {action: toggle}
  - type: button
    name: Big Candle OFF
    icon: mdi:fire-off
    tap_action: {action: toggle}
```

```txt
```

## Additional Resources

- **Icon Library for UI Cards:** [MDI Icons](https://pictogrammers.com/library/mdi/)

## Notes & Tips

- Each IR remote **must have a unique name** (`substitutions`, `globals`, `esphome` fields in `v10.yml`).
- Ensure correct **MQTT credentials** are set in `secrets.yaml`.
- The **kickstart firmware** will NOT brick the device if the process fails—it can be retried.

This guide ensures a smooth transition from a Tuya-locked IR device to an open-source, MQTT-enabled smart home solution!



The kickstart firmware will NOT brick the device if the process fails—it can be retried.

This guide ensures a smooth transition from a Tuya-locked IR device to an open-source, MQTT-enabled smart home solution!


