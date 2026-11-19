# Upstream attribution

The combined YAML adapts the following upstream projects:

1. **Seeed Studio — XIAO ESPHome Projects, XIAO Smart IR Mate:** https://github.com/Seeed-Studio/xiao-esphome-projects/blob/main/projects/xiao_smart_ir_mate/xiao_smart_ir_mate.yaml — GPL-3.0. Includes `Flash_comp` referenced as an external component.
2. **ESPHome — Infrared Proxies, XIAO IR Mate:** https://github.com/esphome/infrared-proxies/blob/main/xiao-ir-mate/xiao-ir-mate.yaml — Apache-2.0.

Changes from upstream: merged the IR proxy platform into Seeed's configuration using existing `my_transmitter` and `rcvr` hardware components, replaced credentials with `!secret` references, supplied shared setup documentation and testing notes. Source projects are independent and no endorsement is implied.

If redistributing upstream code, preserve required upstream notices; this repository is GPL-3.0 for the combined derivative.
