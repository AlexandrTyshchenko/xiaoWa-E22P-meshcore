# xiaoWa E22P — FakeSolar з модулем Ebyte E22P-433M30S

Та сама плата й сенсори, що й FakeSolar (Faketec / ProMicro nRF52840, AHT20, BMP280, INA3221 @0x42, PCF8563, GPS 38400, захист GPS від спуфінгу). Інший радіомодуль: **E22P-433M30S** (SX1268 + PA + LNA, до 30 dBm) замість E22-400M22S.

Відмінності цієї плати від FakeSolar:
- INA3221 @0x42: шунти **0.1 / 0.1 / 0.1 Ом**, батарея на CH2;
- дільник батареї **1 МОм / 1 МОм**: `ADC_MULTIPLIER=1.758`, час вибірки АЦП 40 мкс.

Радіо: 433.650 МГц / 62.5 кГц / SF8 / CR8, як у FakeSolar.

## Файли

| Файл | Роль |
|---|---|
| `Faketec_E22P_repeater.*` | repeater, ім'я `E22P Repeater`, пароль адміністратора `password` |
| `Faketec_E22P_sensor.*` | sensor, ім'я `E22P Sensor`, пароль `password` |
| `Faketec_E22P_companion_radio_ble.*` | companion BLE, PIN `123456` |

`.uf2` — подвійний reset і копіювання на диск; `.zip` — DFU по USB або OTA (`start ota`).

## Потужність — головна відмінність

Підсилювач модуля дає ~18 дБ: за графіком з паспорта Ebyte (VCC 5 В, 25 °C):

| Налаштування чипа (`set tx`) | Вихід модуля |
|---|---|
| 12 dBm | ~30 dBm (1 Вт) — **максимум** |
| 8 dBm | ~28 dBm |
| 5 dBm | ~25.5 dBm |
| 2 dBm | ~22.5 dBm |

- За замовчуванням `LORA_TX_POWER=12`.
- `LORA_TX_POWER_MAX=12` — жорстке обмеження: `set tx` понад 12 відповідає `Error, max is 12`, а збережене раніше значення (наприклад `tx 22` після прошивки FakeSolar) обрізається до 12 і в налаштуваннях, і в драйвері радіо. Companion не дає виставити понад 12 через застосунок.
- **Не прошивайте на плату з E22P звичайні прошивки FakeSolar/ProMicro** — вони ставлять чипу 22 dBm і можуть спалити підсилювач.

## Живлення

- Модуль: 3.3–5.25 В, понад 5.5 В — ризик пошкодження. Повні 30 dBm — лише при VCC ≥ 5 В; від 3.3 В вихід буде нижчим.
- Струм передачі **600–670 мА** при 30 dBm, прийом ≈ 18 мА, сон ≈ 2 мкА. Живлення й конденсатори біля модуля мають витримувати такі імпульси; від стабілізатора ProMicro 3.3 В модуль не живити.
- Для економії батареї: `set tx 8` (~28 dBm) майже не втрачає дальності, а струм помітно менший.

## Підключення

Таке саме, як у E22-400M22S на Faketec:
- RXEN — пін 2 nRF (керує прошивка), TXEN — з'єднаний з DIO2 модуля (`SX126X_DIO2_AS_RF_SWITCH`);
- TCXO 32 МГц живиться від DIO3, 1.8 В;
- таблиця перемикача (паспорт): TXEN=1/RXEN=0 — TX, TXEN=0/RXEN=1 — RX, 0/0 — вимкнено.

## Збірка

Цей репозиторій — MeshCore з історією upstream, змінами FakeSolar (SX1268 у ProMicro, INA3221, GPS, захист від спуфінгу) і комітом xiaoWa E22P зверху. Збірки — у `variants/fakesolar_sx1268/platformio.ini` (секція `[Faketec_E22P]` і три `env:Faketec_E22P_*`), обмеження потужності — `LORA_TX_POWER_MAX` у `src/helpers/radiolib/RadioLibWrappers.cpp`, `src/helpers/CommonRadioPrefs.cpp`, `src/helpers/CommonCLI.cpp`.

```
pio run -e Faketec_E22P_repeater -e Faketec_E22P_sensor -e Faketec_E22P_companion_radio_ble
python bin/uf2conv/uf2conv.py .pio/build/<env>/firmware.hex -c -f 0xADA52840 -o <env>.uf2
```

Паспорт модуля: https://www.cdebyte.com/products/E22P-433M30S (Manual, `pdf-down.aspx?id=4333`).
