IR2MQTT: IR to MQTT Firmware for Tuya BK7231N Devices

Overview

IR2MQTT is a custom firmware for the generic BK7231N-QFN32 Tuya IR devices, enabling direct IR input and output communication with Home Assistant via MQTT. This allows you to control IR appliances and use existing IR remotes to interact with smart home devices—functionality that most Tuya IR hubs lack.

Supported Devices

This firmware is compatible with budget-friendly Tuya IR blasters like:

Device 1

Device 2

Prerequisites & Tools

Hardware:

Tuya IR device with a BK7231N chipset

A computer or Raspberry Pi (to run Tuya CloudCutter)

A USB-to-UART adapter (if needed for kickstart flashing)

Software:

Tuya CloudCutter (for unlocking the device)

ESPHome (to compile and flash the firmware)

Home Assistant (for integration)

Installation Guide

Step 1: Unlock Device & Install Kickstart Firmware

Run Tuya CloudCutter on your Linux machine or Raspberry Pi.

When prompted, select:

Option 2: Flash third-party firmware.

Option 2: Select by firmware device and name.

Firmware version: 2.0.0 - BK7231N / oem_bk7231n_irbox_mol_ty.

After the kickstart firmware is installed, the device will create a temporary Wi-Fi access point. Connect to it to proceed.

Step 2: Flash the ESPHome Firmware

Option 1: ESPHome Firmware Builder

In Home Assistant, create a new ESPHome device.

Place the v10.yml file into the ESPHome device’s YAML configuration.

Edit secrets.yaml with:

mqtt_broker IP

mqtt_username

mqtt_password

Compile and download the firmware:

esphome run v10.yml

Access the kickstart firmware’s web interface via its Wi-Fi network, and upload the compiled firmware (HEX file).

Option 2: Flash with 3rd-Party Tools

Use USB-to-UART or LT Chip Tool for flashing.

Follow standard procedures for your preferred method.

Step 3: Configure Home Assistant

Once the ESPHome firmware is installed, the device will:

Detect and log IR remotes via its receiver.

Create MQTT sensors and buttons in Home Assistant.

Example Home Assistant UI (Lovelace YAML)

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

Additional Resources:

Icon Library for UI Cards: MDI Icons

Notes & Tips

Each IR remote must have a unique name (substitutions, globals, esphome fields in v10.yml).

Ensure correct MQTT credentials are set in secrets.yaml.

The kickstart firmware will NOT brick the device if the process fails—it can be retried.

This guide ensures a smooth transition from a Tuya-locked IR device to an open-source, MQTT-enabled smart home solution!


