# ir2mqtt
ir to mqtt firmware for the generic-bk7231n-qfn32-tuya to allow it to pass ir remote input and output directly to home assistant

it is designed to be used with cheap tuya ir devices such as the following

https://www.aliexpress.com/item/1005007335703208.html?spm=a2g0o.productlist.main.17.1cb1jicgjicgar&algo_pvid=cfe127f9-f796-4d04-8767-1ff65614c814&algo_exp_id=cfe127f9-f796-4d04-8767-1ff65614c814-8&pdp_ext_f=%7B%22order%22%3A%22797%22%2C%22eval%22%3A%221%22%7D&pdp_npi=4%40dis%21GBP%216.92%213.18%21%21%2161.67%2128.37%21%40211b629217403230884477924eaae3%2112000040313459127%21sea%21UK%216065768981%21X&curPageLogUid=HlkDNUhrevUK&utparam-url=scene%3Asearch%7Cquery_from%3A

https://www.aliexpress.com/item/1005008188667566.html?spm=a2g0o.productlist.main.33.1cb1jicgjicgar&algo_pvid=cfe127f9-f796-4d04-8767-1ff65614c814&algo_exp_id=cfe127f9-f796-4d04-8767-1ff65614c814-16&pdp_ext_f=%7B%22order%22%3A%2219%22%2C%22eval%22%3A%221%22%7D&pdp_npi=4%40dis%21GBP%218.08%214.93%21%21%219.93%216.06%21%40211b629217403230884477924eaae3%2112000044172442642%21sea%21UK%216065768981%21X&curPageLogUid=KzWr64rAd9H7&utparam-url=scene%3Asearch%7Cquery_from%3A

this firmware lets you use them with home assistant (the self hosted smart home controller) and allows you to control IR stuff, but also use old IR remotes to control your smart home crap, which isn't something most of these things support 

Prerequisites & Tools

    Hardware:
        A Tuya IR device with a BK7321N chipset
        A computer or Raspberry Pi for running the Tuya CloudCutter exploit
        A USB-to-UART adapter (if needed for kickstart flashing)
    Software:
        Tuya CloudCutter (see digiblurDIY’s guide)
        https://digiblur.com/2023/08/19/updated-tuya-cloudcutter-with-esphome-bk7231-how-to-guide/

        ESPHome (v10.yml firmware to be compiled and flashed)
        Home Assistant (for device integration)

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

https://pictogrammers.com/library/mdi/
a library for icons for UI cards

Be sure to edit the name in substitutions:, globals:, esphome:, the name must be unique for each ir remote, also be sure to set mqtt_username, mqtt_password, and mqtt_broker ip address in your secrets.yaml 
