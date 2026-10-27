Рабочая конфигурация WT99P4C5-S1 (ESP32-P4 + ESP32-C5)
📋 Краткое описание
Данный репозиторий содержит проверенную конфигурацию для платы Wireless-Tag WT99P4C5-S1 (ESP32-P4 + ESP32-C5) с работой через ESPHome. Конфигурация включает дисплей GC9B72 (SPI), аудиокодек ES8311, микрофон, wake word и голосового ассистента.

Ключевая особенность: ESP32-C5 используется как Wi-Fi сопроцессор через ESP-Hosted (SDIO), а не как AT-модем.

🔧 Аппаратная конфигурация
Компонент	Модель / Параметры
Основной чип	ESP32-P4 rev v1.0 (pre-rev3)
Сопроцессор	ESP32-C5 rev v1.0
Flash	16 MB (P4), 4 MB (C5)
PSRAM	32 MB, 200 MHz
Дисплей	GC9B72, SPI (360×360)
Аудиокодек	ES8311 (I2C + I2S)
Микрофон	I2S (GPIO11)
Усилитель	GPIO53
📦 Версии сред и совместимость
Компонент	Версия	Примечание
ESPHome	2026.9.1	Компиляция и прошивка
ESP-IDF (P4)	v5.5.2	Сборка ESPHome
ESP-IDF (C5)	v5.5.2	Сборка ESP-Hosted slave
ESP-Hosted (C5)	2.12.12	Должна совпадать с версией хоста
ESP-Hosted (P4 хост)	2.12.12	Встроена в ESPHome
⚠️ Критически важные моменты совместимости
Версия ESP-Hosted на C5 должна совпадать с версией на P4 (хосте). В ESPHome 2026.9.1 используется 2.12.12, поэтому C5 должен быть прошит именно этой версией.

ESP32-P4 rev v1.0 требует engineering_sample: true в конфиге ESPHome (иначе прошивка не загрузится из-за ошибки MSPI-749).

ESP32-C5 должен быть прошит ESP-Hosted slave, а не ESP-AT. AT-прошивка несовместима с ESPHome.

📁 Структура репозитория
text
/
├── README.md                          # Этот файл
├── esphome/
│   └── wireless-tag-p4.yaml          # Конфиг ESPHome (P4)
├── c5_firmware/
│   ├── flasher_args.json             # Адреса для прошивки C5
│   ├── bootloader.bin                # Загрузчик C5
│   ├── partition-table.bin           # Таблица разделов C5
│   ├── ota_data_initial.bin          # OTA data C5
│   └── network_adapter.bin           # ESP-Hosted slave (2.12.12)
└── docs/
    ├── P4_FLASH.md                   # Инструкция по прошивке P4
    └── C5_FLASH.md                   # Инструкция по прошивке C5
🛠 Сборка и прошивка
P4 (ESP32-P4) — ESPHome
Установи ESPHome 2026.9.1.

Скопируй esphome/wireless-tag-p4.yaml в папку конфигов ESPHome.

Настрой secrets.yaml (wifi_ssid, wifi_password, ha_ip).

Собери и прошей через ESPHome Dashboard или CLI:

bash
esphome run wireless-tag-p4.yaml
C5 (ESP32-C5) — ESP-Hosted slave
Сборка:

bash
idf.py create-project-from-example "espressif/esp_hosted=2.12.12:slave"
cd slave
idf.py set-target esp32c5
idf.py build
Прошивка (адреса из flasher_args.json):

bash
esptool.py --chip esp32c5 -p COM3 write_flash \
  --flash_mode dio --flash_size 4MB --flash_freq 80m \
  0x2000 bootloader/bootloader.bin \
  0xc000 partition_table/partition-table.bin \
  0x16000 ota_data_initial.bin \
  0x20000 network_adapter.bin
⚠️ Важно: при прошивке C5 обязательно притяни EN P4 к земле (замкни на GND), иначе SDIO-помехи повредят запись.

⚙️ Ключевые настройки ESPHome
Блок esp32 (P4)
yaml
esp32:
  board: esp32-p4-evboard
  variant: esp32p4
  flash_size: 16MB
  engineering_sample: true  # ОБЯЗАТЕЛЬНО для rev v1.0
  framework:
    type: esp-idf
    advanced:
      enable_idf_experimental_features: yes
Блок esp32_hosted (SDIO к C5)
yaml
esp32_hosted:
  variant: esp32c5
  active_high: true
  reset_pin: GPIO54
  cmd_pin: GPIO19
  clk_pin: GPIO18
  d0_pin: GPIO14
  d1_pin: GPIO15
  d2_pin: GPIO16
  d3_pin: GPIO17
Блок esp_ldo
yaml
esp_ldo:
  - channel: 4
    voltage: 2.5V
(Канал 3 закомментирован — на данной плате PSRAM работает через канал 4)

📝 Известные проблемы и решения
Проблема	Причина	Решение
sdmmc_init_ocr: send_op_cond returned 0x107	C5 прошит ESP-AT или несовместимой версией ESP-Hosted	Прошить C5 ESP-Hosted slave версии 2.12.12
P4 уходит в цикл POWERON / SW_SYS_RESET	Errata MSPI-749 для P4 v1.0	Добавить engineering_sample: true в конфиг ESPHome
invalid header: 0xffffffff на C5	Прошивка по неправильному адресу	Использовать адреса из flasher_args.json
H_SDIO_DRV: Dropping packet(s) from stream	Переполнение RX-очереди SDIO	Увеличить CONFIG_ESP_HOSTED_SDIO_RX_Q_SIZE или снизить частоту SDIO
✅ Проверенная рабочая конфигурация
Лог успешной загрузки C5:

text
I (509) co-pro-main: ESP-Hosted-MCU Slave FW version :: 2.12.12
I (533) SDIO_SLAVE: Using SDIO interface
I (550) co-pro-main: host reset handler task started
I (1723) co-pro-main: Start Data Path
I (14264) slave_wifi_std: Sta mode connected
Лог успешной загрузки P4 (ESPHome):

text
[I][app:117]: setup() finished successfully!
[I][wifi:1486]: - 'ASUS123' (4A:22:54:4F:33:8C) ▂▄▆█
[I][wifi:1626]: Connected
📚 Ссылки
ESP-Hosted репозиторий

ESPHome ESP32-P4 support

Wireless-Tag WT99P4C5-S1

ESP32-P4 Errata (MSPI-749)

📄 Лицензия
MIT (или укажи свою)

🙏 Благодарности
Конфигурация собрана на основе официальных примеров Espressif, ESPHome и документации Wireless-Tag. Отдельная благодарность сообществу ESPHome за помощь в отладке SDIO.