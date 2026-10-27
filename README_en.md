# WT99P4C5-S1 Working Configuration

Проверенная конфигурация для платы **Wireless-Tag WT99P4C5-S1** (ESP32-P4 + ESP32-C5) с ESPHome.

## Hardware

| Component | Model |
|---|---|
| Main chip | ESP32-P4 rev **v1.0** (pre-rev3) |
| Coprocessor | ESP32-C5 rev v1.0 |
| Flash | 16 MB (P4), 4 MB (C5) |
| PSRAM | 32 MB, 200 MHz |
| Display | GC9B72, SPI (360×360) |
| Audio codec | ES8311 (I2C + I2S) |
| Microphone | I2S (GPIO11) |
| Amplifier | GPIO53 |

## Versions

| Component | Version | Notes |
|---|---|---|
| ESPHome | 2026.9.1 | Compilation & flashing |
| ESP-IDF (P4) | v5.5.2 | ESPHome build |
| ESP-IDF (C5) | v5.5.2 | ESP-Hosted slave build |
| ESP-Hosted (C5) | **2.12.12** | Must match host version |
| ESP-Hosted (P4 host) | 2.12.12 | Built into ESPHome |

### ⚠️ Critical compatibility notes

1. **ESP-Hosted version on C5 must match P4 (host).** ESPHome 2026.9.1 uses 2.12.12.
2. **ESP32-P4 rev v1.0** requires `engineering_sample: true` (otherwise bootloop due to MSPI-749 errata).
3. **ESP32-C5** must be flashed with **ESP-Hosted slave**, not ESP-AT.

## Repository structure
/
├── README.md
├── esphome/
│ └── wireless-tag-p4.yaml
├── c5_firmware/
│ ├── flasher_args.json
│ ├── bootloader.bin
│ ├── partition-table.bin
│ ├── ota_data_initial.bin
│ └── network_adapter.bin
└── docs/
├── P4_FLASH.md
└── C5_FLASH.md

text

## Flashing

### P4 (ESP32-P4) — ESPHome

1. Install ESPHome 2026.9.1.
2. Copy `esphome/wireless-tag-p4.yaml` to ESPHome config folder.
3. Configure `secrets.yaml` (wifi_ssid, wifi_password, ha_ip).
4. Build and flash:
   ```bash
   esphome run wireless-tag-p4.yaml
C5 (ESP32-C5) — ESP-Hosted slave
Build:

bash
idf.py create-project-from-example "espressif/esp_hosted=2.12.12:slave"
cd slave
idf.py set-target esp32c5
idf.py build
Flash (addresses from flasher_args.json):

bash
esptool.py --chip esp32c5 -p COM3 write_flash \
  --flash_mode dio --flash_size 4MB --flash_freq 80m \
  0x2000 bootloader/bootloader.bin \
  0xc000 partition_table/partition-table.bin \
  0x16000 ota_data_initial.bin \
  0x20000 network_adapter.bin
⚠️ Important: when flashing C5, pull EN P4 to ground (short to GND), otherwise SDIO interference will corrupt the write.

Known issues
Problem	Cause	Solution
sdmmc_init_ocr: send_op_cond returned 0x107	C5 flashed with ESP-AT or incompatible ESP-Hosted	Flash C5 with ESP-Hosted slave 2.12.12
P4 bootloop POWERON / SW_SYS_RESET	MSPI-749 errata for P4 v1.0	Add engineering_sample: true
invalid header: 0xffffffff on C5	Flashed at wrong address	Use addresses from flasher_args.json
H_SDIO_DRV: Dropping packet(s) from stream	SDIO RX queue overflow	Increase CONFIG_ESP_HOSTED_SDIO_RX_Q_SIZE or lower SDIO frequency
Verified working logs
C5:

text
I (509) co-pro-main: ESP-Hosted-MCU Slave FW version :: 2.12.12
I (533) SDIO_SLAVE: Using SDIO interface
I (1723) co-pro-main: Start Data Path
I (14264) slave_wifi_std: Sta mode connected
P4 (ESPHome):

text
[I][app:117]: setup() finished successfully!
[I][wifi:1626]: Connected
Links
ESP-Hosted

ESPHome ESP32-P4

Wireless-Tag WT99P4C5-S1

ESP32-P4 Errata (MSPI-749)

License
MIT

text

---

## 📄 `docs/P4_FLASH.md`

```markdown
# Flashing ESP32-P4 (ESPHome)

## Prerequisites

- ESPHome 2026.9.1
- ESP-IDF v5.5.2 (used by ESPHome internally)

## Steps

1. Copy `esphome/wireless-tag-p4.yaml` to your ESPHome config directory.
2. Create `secrets.yaml`:
   ```yaml
   wifi_ssid: "YourSSID"
   wifi_password: "YourPassword"
   ha_ip: "192.168.1.100"
Build and flash:

bash
esphome run wireless-tag-p4.yaml
Critical config points
engineering_sample: true — required for P4 rev v1.0

esp_ldo channel 4 — PSRAM power

esp32_hosted — SDIO pins to C5

Troubleshooting
If P4 bootloops with POWERON, check:

engineering_sample: true is present

execute_from_psram is NOT enabled

esp_ldo has channel 4 enabled

text

---

## 📄 `docs/C5_FLASH.md`

```markdown
# Flashing ESP32-C5 (ESP-Hosted slave)

## Prerequisites

- ESP-IDF v5.5.2
- ESP-Hosted **2.12.12** (must match P4 host)

## Build

```bash
idf.py create-project-from-example "espressif/esp_hosted=2.12.12:slave"
cd slave
idf.py set-target esp32c5
idf.py build
Flash
⚠️ Before flashing: pull EN P4 to GND (short to ground). This disables P4 and prevents SDIO interference.

Connect to J5 pins: EN, TXD, RXD, GND, BOOT.

Enter bootloader mode: hold BOOT, tap EN, release EN, release BOOT.

Flash using addresses from build/flasher_args.json:

bash
esptool.py --chip esp32c5 -p COM3 write_flash \
  --flash_mode dio --flash_size 4MB --flash_freq 80m \
  0x2000 bootloader/bootloader.bin \
  0xc000 partition_table/partition-table.bin \
  0x16000 ota_data_initial.bin \
  0x20000 network_adapter.bin
Verify
Monitor C5:

bash
idf.py -p COM3 monitor
Expected output:

text
I (509) co-pro-main: ESP-Hosted-MCU Slave FW version :: 2.12.12
I (533) SDIO_SLAVE: Using SDIO interface
I (1723) co-pro-main: Start Data Path
