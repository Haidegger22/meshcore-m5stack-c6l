# MeshCore для M5Stack Unit C6L

Прошивка MeshCore v1.14.1 (Companion Radio BLE) для [M5Stack Unit C6L](https://docs.m5stack.com/en/unit/Unit_C6L) — компактного LoRa-узла на базе ESP32-C6 + SX1262.

## Характеристики устройства

| Параметр | Значение |
|----------|----------|
| MCU | ESP32-C6 (RISC-V 160 МГц) |
| Flash | 16 МБ |
| LoRa | SX1262, 868–923 МГц, до +22 dBm TX |
| Дисплей | SSD1306 OLED 64×48 |
| Интерфейс | USB-C, BLE, Wi-Fi 6 |
| Дополнительно | NeoPixel LED, Buzzer, кнопка |

## Содержимое репозитория

```
meshcore-m5stack-c6l/
├── README.md
└── firmware/
    ├── m5stack_unit_c6l_companion_radio_ble.bin     ← стандартная прошивка
    └── m5stack_unit_c6l_companion_radio_ble_lp.bin  ← энергосберегающая (LP)
```

Оба файла — **merged binary**, прошиваются одной командой с адреса `0x0`.

## Варианты прошивок

### Стандартная (`companion_radio_ble`)

Оригинальная сборка v1.14.1 от TheRealHaoLiu без изменений.
- Экран горит 30 сек после включения и загорается при каждом сообщении
- Кнопка будит экран на 30 сек
- TX до 20 dBm, RX boost ON

### Энергосберегающая LP (`companion_radio_ble_lp`)

Собрана из исходников (v1.16-dev) с модификациями для экономии энергии.

| Что | Стандартная | LP |
|-----|------------|-----|
| Экран при сообщениях | ✅ загорается | ❌ не загорается |
| Кнопка → экран | на 30 сек | на **5 сек** |
| Экран при старте | 30 сек | 30 сек |
| TX мощность | до 20 dBm | **до 22 dBm** |
| RX boost | ON | ON |
| BLE | работает | работает |
| CPU-сон | нет | нет |

**Основная экономия:** OLED не загорается при каждом входящем/исходящем сообщении (~20 мА экономии). CPU не спит (BLE требует постоянного обслуживания).

**Изменения в исходном коде:**
- `UITask.cpp` — сообщения не будят дисплей в режиме `POWER_SAVING`, таймер кнопки 5 сек
- `ESP32Board.h` — добавлена поддержка light sleep для ESP32-C6, метод `sleepMs()`
- `platformio.ini` — новый env `m5stack_unit_c6l_companion_radio_ble_lp`

**Сборка LP из исходников:**
```bash
git clone https://github.com/TheRealHaoLiu/MeshCore --branch main-m5stack-unit-c6l
cd MeshCore
# Применить патчи из meshcore-m5stack-c6l/patches/
pio run -e m5stack_unit_c6l_companion_radio_ble_lp
```

## Как прошить

### 1. Установите esptool

```bash
pip install esptool
# или используйте PlatformIO:
# ~/.platformio/packages/tool-esptoolpy/esptool.py
```

### 2. Переведите C6L в режим прошивки

1. Подключите C6L к компьютеру через USB-C
2. Зажмите кнопку **RESET** (боковая) на **2–3 секунды** — светодиод сменит цвет с зелёного на **красный**
3. Отпустите RESET — светодиод снова загорится зелёным, но устройство уже в режиме загрузки
4. Устройство будет доступно как `/dev/ttyACM0` (Linux) или `COMx` (Windows)

> **Примечание для Raspberry Pi / Linux:** если после отпускания RESET устройство отключается (пропадает `/dev/ttyACM0`), используйте альтернативный метод:
> 1. Отключите USB-C
> 2. Зажмите **BOOT** и держите
> 3. Подключите USB-C (не отпуская BOOT)
> 4. Отпустите BOOT

### 3. Прошейте

```bash
esptool.py --chip esp32c6 --port /dev/ttyACM0 --before default_reset --after hard_reset \
  write_flash 0x0 firmware/m5stack_unit_c6l_companion_radio_ble.bin
```

### 4. Подключение через приложение

После прошивки C6L перезагрузится. На OLED-дисплее отобразится **случайный PIN-код** для BLE-подключения.

> 💡 Если нужен постоянный PIN — измените его в приложении MeshCore: **Settings → BLE PIN Code**.

Установите приложение MeshCore:
- 📱 [Android](https://play.google.com/store/apps/details?id=com.liamcottle.meshcore.android)
- 📱 [iOS](https://apps.apple.com/us/app/meshcore/id6742354151?platform=iphone)
- 🌐 [Web App](https://app.meshcore.nz)

## Источник прошивки

### Автор порта под C6L

**Hao Liu ([TheRealHaoLiu](https://github.com/TheRealHaoLiu))** — автор порта MeshCore для M5Stack Unit C6L.

### Исходный код

- **Форк MeshCore:** [TheRealHaoLiu/MeshCore](https://github.com/TheRealHaoLiu/MeshCore/tree/main-m5stack-unit-c6l) (ветка `main-m5stack-unit-c6l`)
- **Предсобранные бинарники:** [TheRealHaoLiu/MeshCore-M5Burner-UnitC6L](https://github.com/TheRealHaoLiu/MeshCore-M5Burner-UnitC6L)
- **Официальный MeshCore:** [meshcore-dev/MeshCore](https://github.com/meshcore-dev/MeshCore)

### Версия

- **MeshCore:** v1.14.1 (апрель 2026)
- **Бинарник взят из:** [`TheRealHaoLiu/MeshCore-M5Burner-UnitC6L`](https://github.com/TheRealHaoLiu/MeshCore-M5Burner-UnitC6L/tree/main/firmware_companion_radio) — директория `firmware_companion_radio/`

### Изменения

- **Стандартный бинарник** — без изменений, as-is из репозитория TheRealHaoLiu
- **LP-вариант** — собран из исходников с модификациями (патчи в `patches/`) для энергосберегающего поведения дисплея

## Сборка из исходников (для версии 1.16+)

Если вы хотите собрать свежую версию MeshCore из исходников:

```bash
git clone https://github.com/TheRealHaoLiu/MeshCore --branch main-m5stack-unit-c6l
cd MeshCore
pio run -e m5stack_unit_c6l_companion_radio_ble
```

Собранный бинарник будет в `.pio/build/m5stack_unit_c6l_companion_radio_ble/`.

### Известные проблемы при сборке

- **DNS для `packages.platformio.org` не резолвится** на некоторых конфигурациях (Tailscale DNS). Решение: добавить в `/etc/hosts`:
  ```
  88.198.170.159 packages.platformio.org
  ```
- **Тулчейн RISC-V** (~139 МБ) качается медленно с PlatformIO CDN. Альтернатива — скачать напрямую с [espressif/crosstool-NG](https://github.com/espressif/crosstool-NG/releases) и установить вручную в `~/.platformio/packages/toolchain-riscv32-esp/`.

## Двойная прошивка (Dual OTA)

C6L поддерживает хранение **двух прошивок** одновременно и переключение между ними удержанием кнопки BOOT при включении:

| Слот | Адрес | Размер |
|------|-------|--------|
| ota_0 (app0) | `0x10000` | 1920 КБ |
| ota_1 (app1) | `0x1F0000` | 1920 КБ |

Для прошивки во второй слот используйте `firmware_0x10000.bin` (не merged binary):
```bash
esptool.py --chip esp32c6 --port /dev/ttyACM0 write_flash 0x1F0000 firmware_0x10000.bin
```

При загрузке с зажатым BOOT (≥500 мс):
- 🎵 Два восходящих сигнала = переключение успешно
- 🎵 Три коротких гудка = второй слот пуст

## Лицензия

Проект MeshCore распространяется под лицензией **MIT**.

## Благодарности

- **Hao Liu** ([TheRealHaoLiu](https://github.com/TheRealHaoLiu)) — порт MeshCore для M5Stack Unit C6L
- **Liam Cottle** ([meshcore-dev](https://github.com/meshcore-dev)) — создатель MeshCore
- **M5Stack** — производитель устройства Unit C6L
