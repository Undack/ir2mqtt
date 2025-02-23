# ir2mqtt
ir to mqtt firmware for the generic-bk7231n-qfn32-tuya to allow it to pass ir remote input and output directly to home assistant

Prerequisites & Tools

    Hardware:
        A Tuya IR device with a BK7321N chipset
        A computer or Raspberry Pi for running the Tuya CloudCutter exploit
        A USB-to-UART adapter (if needed for kickstart flashing)
    Software:
        Tuya CloudCutter (see digiblurDIY’s guide)
        https://digiblur.com/2023/08/19/updated-tuya-cloudcutter-with-esphome-bk7231-how-to-guide/
        LT Chip Tool (for reading/adjusting device configuration)
        ESPHome (v10.yml firmware to be compiled and flashed)
        Home Assistant (for device integration)
        A web browser to reference icons from Pictogrammers MDI Library

Step 1: Preparing Your Device & Installing Kickstart Firmware

    Run Tuya CloudCutter:
    Use your Raspberry Pi or Linux computer to run Tuya CloudCutter. This tool will exploit the device over the air without any soldering.
        Follow the on-screen instructions until you reach the firmware flash options.
    Select the Correct Kickstart Firmware:
    When prompted, choose the kickstart firmware installers for the BK7321N. Important:
        Use the v2.0.0 installer that is the second one down in the group.
        Note that if you try any other installer, the process will simply fail (without bricking the device), so double-check before proceeding.
    Flash the Kickstart Firmware:
    Complete the flashing process. Once successful, your device will boot into a temporary “Kickstart” mode (typically broadcasting its own access point). Connect to this access point, then proceed with your network setup as guided.

Step 2: Flashing the v10.yml ESPHome Firmware

    Prepare Your ESPHome Configuration:
    Download or open your v10.yml file—the custom configuration designed for your IR device. Make sure it contains the appropriate settings for your device’s hardware (GPIOs for IR transmission/reception, relays, sensors, etc.).

    Compile the Firmware:
    Use the ESPHome dashboard or command-line tool to compile the firmware:

    esphome run v10.yml

    This process builds the binary (UF2 or BIN file, depending on your chosen update method).

    Flash the ESPHome Firmware:
    There are two main methods:
        OTA Update: If your device (now running kickstart firmware) supports OTA, upload the new binary through its web interface (accessed via the device’s temporary IP, e.g. http://192.168.4.1).
        Direct Flashing: Alternatively, use your preferred method (USB-to-UART or the LT Chip Tool) to flash the binary.

    Verify Successful Flash:
    Once updated, the device should connect to your home network. You can verify by checking your router’s DHCP leases or by using the ESPHome dashboard.

Step 3: Configuring Home Assistant for IR Control

After the v10.yml firmware is installed, the device automatically creates a sensor and button for each IR remote detected by the receiver. To control IR transmissions via Home Assistant, you can add a UI card. For example, here’s a YAML snippet for a vertical-stack card:

type: vertical-stack
title: Big Candle Ir Remote
cards:
  - type: button
    name: big candle on
    icon: mdi:fire
    show_name: true
    show_icon: true
    tap_action:
  - type: button
    name: big candle 6H
    icon: mdi:clock-outline
    show_name: true
    show_icon: true
    tap_action:
  - type: button
    name: big candle off
    icon: mdi:fire-off
    show_name: true
    show_icon: true
    tap_action:

Notes:

    Replace $devicename with your device’s MQTT topic or name as configured in your ESPHome YAML.
    Visit Pictogrammers MDI Library to choose and verify icons.
