# XIAO Smart IR Mate — Seeed IR learning + Home Assistant IR Proxy

A **dual-mode ESPHome configuration** for the **Seeed Studio XIAO Smart IR Mate (ESP32-C3)**, combining the original Seeed local IR-learning interface with the native ESPHome IR proxy for Home Assistant and **HAIR**.

**Status (November 19, 2026): field-tested on actual hardware.** The maintainer successfully learned and replayed a signal with Seeed controls, received and transmitted IR through HAIR, then replayed the previously stored `Signal0` after HAIR control. This verifies the specific sequence, not long-term stress testing or every IR protocol.

## Features

| Seeed local controls | Native IR proxy |
| --- | --- |
| Learn and store up to 10 IR commands (`Signal0`–`Signal9`) | Expose an IR proxy transmitter in Home Assistant |
| Replay a stored command via the HA **Send** button | Expose an IR proxy receiver for HAIR learning |
| Choose a signal slot in Home Assistant | Send and receive codes without reflashing ESPHome |
| Touch sensor replay, vibration, RGB Wi-Fi indicator | Shares the same physical IR transmitter/receiver |
| Restart/factory reset controls | Works alongside stored slots in tested sequence |

### Hardware mapping

| Function | GPIO |
| --- | --- |
| IR transmitter | GPIO3 |
| IR receiver (inverted) | GPIO4 |
| Touch pad | GPIO5 |
| Vibration motor | GPIO6 |
| WS2812 RGB LED | GPIO7 |
| Reset button | GPIO9 |

Both IR systems **share the same** `remote_transmitter` (`my_transmitter`) and `remote_receiver` (`rcvr`) components. Do not add a second GPIO3/GPIO4 definition. The Seeed side uses `Flash_comp` from the upstream Seeed Studio repository to store learned IR samples.

## Requirements

- Seeed Studio **XIAO Smart IR Mate** with ESP32-C3.
- ESPHome Device Builder supporting the `infrared` / `ir_rf_proxy` integration (use a current ESPHome release that exposes it).
- Home Assistant with the ESPHome integration.
- **HAIR** for the tested dynamic IR remote workflow (optional for Seeed-only learning and playback).
- An available Wi-Fi network and ESPHome `secrets.yaml`.

## Installation

1. Create a device in **ESPHome Device Builder** or use an existing XIAO Smart IR Mate configuration. Back up the working YAML before replacing anything.
2. Copy [`xiao_smart_ir_mate.yaml`](xiao_smart_ir_mate.yaml) into ESPHome's configuration directory. The sample device name is `irproxy-sb`; change it to avoid conflicts on your network.
3. Copy the entries from [`secrets.example.yaml`](secrets.example.yaml) into your **local** ESPHome `secrets.yaml` and supply a real Wi-Fi SSID/password, API encryption key and fallback AP password. **Do not upload `secrets.yaml` to GitHub.**
4. In a VPN or network without functional mDNS, optionally uncomment `wifi.use_address` and set it to the device's reachable LAN IP; this affects ESPHome's connection address, **not** DHCP assignment. Prefer a DHCP reservation if you need a stable address.
5. Run **Validate** in ESPHome. Compile and install via USB initially or OTA if you already have a compatible firmware installed.
6. Add or reconfigure the device in **Home Assistant → Settings → Devices & services → ESPHome**, supplying the API encryption key when requested.
7. Confirm that the Seeed controls and the two IR proxy entities appear. Entity IDs may differ by installation.

> **Migration warning:** A change to the ESPHome API encryption key may require reconfiguring the Home Assistant integration. The example does not force an ESPHome OTA password; if your existing device does, preserve your existing OTA settings when updating. Do not blindly flash a copy over an established configuration.

## Using the Seeed local learning features

1. In Home Assistant, select **Signal0** (or another `Signal0`–`Signal9` slot).
2. Press **Learn**, then press the desired source remote button toward the XIAO Smart IR Mate.
3. Verify **Is Learned Signal?** indicates a stored sample.
4. Press **Send**, or use the device's touch sensor, while pointing the device toward the controlled appliance.
5. Change the selected slot to learn or play a different command.

The `on_raw` automation saves a sample to Seeed's flash-backed storage when learning is active. The implementation contains a `filter_size` threshold of 10 timing entries; not all IR remotes/protocols are guaranteed to work. The `Send` script uses a 38 kHz carrier for locally stored raw samples.

## Using the native IR proxy with HAIR

1. Confirm both **IR Proxy Receiver** and **IR Proxy Transmitter** are available in Home Assistant.
2. In **HAIR**, add a new remote/device using the XIAO's IR proxy receiver, following your installed HAIR version's workflow.
3. Capture a remote command through the native proxy and use the corresponding proxy transmitter to control the appliance.
4. Return to the Seeed **Send** button and replay `Signal0` to verify that the stored command still functions.

No additional `remote_receiver` or `remote_transmitter` declaration is necessary for HAIR; the `infrared: ir_rf_proxy` entries reference existing IDs.

## Field verification

The following were **observed by the project maintainer on November 19, 2026**, on a configuration with the same IR logic and Seeed hardware:

| Check | Result |
| --- | --- |
| ESPHome firmware installed and running with Home Assistant | PASS |
| Seeed learning into a selected slot | PASS |
| Seeed command replay | PASS |
| HAIR learning using native IR proxy receiver | PASS |
| HAIR appliance control using native IR proxy transmitter | PASS |
| `Signal0` Seeed replay *after* HAIR control | PASS |
| Standalone raw IR receiver logs | Not separately tested |
| Cold-reboot persistence, concurrent captures, long-duration load | Not tested |

## Architecture and compatibility notes

- `esp32.framework: arduino` is intentional: it matches Seeed's custom `Flash_comp` implementation. The upstream ESPHome standalone proxy example uses ESP-IDF; this combined setup has been field-tested using Arduino.
- The Seeed `on_raw` handler and the proxy receiver both subscribe to `rcvr`. The reported sequence demonstrates interoperability but does not establish concurrent capture behavior under every load.
- `remote_transmitter.non_blocking: true`, `idle: 65500us` and `clock_resolution: '500000'` match the tested setup. These are deliberately **not** retuned without a regression test.
- `external_components` follows the Seeed `main` branch; upstream changes can affect future builds. Pin the source to a known commit when creating a reproducible release.
- `project.version: '2.0'` is inherited from Seeed's YAML metadata; it is **not** this repository's release version.

## Troubleshooting

**Device connects but HAIR sees no receiver/transmitter**: confirm that your installed ESPHome build supports `infrared: - platform: ir_rf_proxy`, then inspect entity availability in the ESPHome integration. Check the HAIR integration's selected receiver/transmitter.

**Seeed Learn doesn't capture**: verify that the correct slot is selected, press **Learn** first, check ESPHome logs, and ensure the IR source is within range. Some remotes may use unsupported timing/protocols.

**Local Seeed Send fails**: confirm `Is Learned Signal?` is on for the selected slot; check appliance line-of-sight and whether the appliance uses the expected carrier/protocol.

**Can't reach the ESPHome device over a VPN**: `wifi.use_address` sets a reachable address for ESPHome management; it does not convert a DHCP client into a static-IP host. Ensure the ESPHome API and OTA ports are reachable.

**After changing credentials**: retain existing valid API/OTA settings and reconfigure Home Assistant if the encryption key changes. Restore your backed-up YAML if an update regresses behavior.

## Credits and license

This work is a documented combination of upstream configurations:

- [Seeed Studio — XIAO Smart IR Mate ESPHome YAML](https://github.com/Seeed-Studio/xiao-esphome-projects/blob/main/projects/xiao_smart_ir_mate/xiao_smart_ir_mate.yaml), including its external `Flash_comp` component — **GPL-3.0**.
- [ESPHome — Infrared Proxy XIAO IR Mate YAML](https://github.com/esphome/infrared-proxies/blob/main/xiao-ir-mate/xiao-ir-mate.yaml) — **Apache-2.0**.

This combined repository is distributed under the **GNU General Public License v3.0** (see [`LICENSE`](LICENSE)). Retain the attribution and upstream notices. The Apache-2.0 source is compatible with GPLv3 combination; upstream repositories and their original code remain under their respective licenses. The `Flash_comp` source is fetched from Seeed at build time, not vendored here.

This is a community configuration, not an official release from Seeed Studio, ESPHome, HAIR or Home Assistant.
