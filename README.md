<p align="center"><img src="buildroot/share/pixmaps/logo/marlin-outrun-nf-500.png" height="250" alt="MarlinFirmware's logo" /></p>

<h1 align="center">Marlin 3D Printer Firmware</h1>
<h2 align="center">Самосборный дрыгостол 310×310 — MKS TinyBee</h2>

<p align="center">
    <a href="/LICENSE"><img alt="GPL-V3.0 License" src="https://img.shields.io/github/license/marlinfirmware/marlin.svg"></a>
    <a href="https://github.com/MarlinFirmware/Marlin/graphs/contributors"><img alt="Contributors" src="https://img.shields.io/github/contributors/marlinfirmware/marlin.svg"></a>
    <a href="https://github.com/MarlinFirmware/Marlin/releases"><img alt="Last Release Date" src="https://img.shields.io/github/release-date/MarlinFirmware/Marlin"></a>
    <a href="https://github.com/MarlinFirmware/Marlin/actions/workflows/ci-build-tests.yml"><img alt="CI Status" src="https://github.com/MarlinFirmware/Marlin/actions/workflows/ci-build-tests.yml/badge.svg"></a>
</p>

---

## О проекте

Кастомная конфигурация Marlin 2.1 для самосборного 3D-принтера с движущимся столом (дрыгостол).  
Конфигурация является отправной точкой — **шаговые двигатели, направления осей и смещения датчика необходимо настроить под своё железо.**

---

## Железо

| Компонент | Описание |
|---|---|
| Плата управления | MKS TinyBee (ESP32) |
| Драйверы | TMC2209 (режим Standalone) |
| Размер стола | 310 × 310 мм |
| Тип принтера | Cartesian, движущийся стол (дрыгостол) |
| Ходовой винт Z | T8, шаг 8 мм, однозаходный |
| Датчик автолевелинга | Индукционный (FIX_MOUNTED_PROBE) |
| Дисплей | LCD12864 (RAMPS, ST7920) с энкодером |
| Датчик филамента | Есть (FIL_RUNOUT_SENSOR) |

---

## Что настроено

- ✅ Плата: `BOARD_MKS_TINYBEE`
- ✅ Размер стола: 310 × 310 мм
- ✅ Шаги Z: 400 steps/mm (T8 8мм + 16 микрошагов)
- ✅ Автолевелинг: Bilinear 7×7
- ✅ Z Safe Homing (хоминг по центру стола)
- ✅ Индукционный датчик: FIX_MOUNTED_PROBE
- ✅ Дисплей: REPRAP_DISCOUNT_FULL_GRAPHIC_SMART_CONTROLLER
- ✅ SD карта, бипер, энкодер
- ✅ Датчик окончания филамента
- ✅ EEPROM (M500/M501/M502)
- ✅ Thermal Runaway Protection
- ✅ NOZZLE_PARK_FEATURE
- ✅ Преднагрев: PLA / PETG / ABS

---

## ⚠️ Что нужно настроить под себя

### 1. Шаги/мм шаговых двигателей
```cpp
// Configuration.h
#define DEFAULT_AXIS_STEPS_PER_UNIT { 80, 80, 400, 500 }
//                                    ^    ^    ^    ^
//                                    X    Y    Z    E  ← E нужно откалибровать!
```
X и Y = 80 (GT2 ремень + 20T шкив + 16 микрошагов)  
Z = 400 (T8 8мм + 16 микрошагов) — менять не нужно  
**E — обязательно откалибровать под свой экструдер командой M92 E**

### 2. Направление осей
```cpp
#define INVERT_X_DIR false   // поменяй на true если ось едет не туда
#define INVERT_Y_DIR true
#define INVERT_Z_DIR false
#define INVERT_E0_DIR false
```
Проверь при первом запуске командами G1 X10, G1 Y10 и т.д.

### 3. Смещение датчика от сопла
```cpp
#define NOZZLE_TO_PROBE_OFFSET { 10, 10, -1 }
//                               ^    ^    ^
//                               X    Y    Z  ← измерь линейкой своё!
```

### 4. PID автотюн (после сборки)
```
M303 E0 S200 C8     — автотюн хотэнда при 200°C
M303 E-1 S60 C8     — автотюн стола при 60°C
M500                — сохранить результат в EEPROM
```

### 5. Концевики
Убедись что логика NC/NO совпадает с твоими концевиками:
```cpp
#define X_MIN_ENDSTOP_INVERTING true   // true = NO, false = NC
#define Y_MIN_ENDSTOP_INVERTING true
#define Z_MIN_ENDSTOP_INVERTING true
```
Рекомендуется использовать NC концевики — безопаснее при обрыве провода.

---

## Первый запуск — чеклист

- [ ] Проверить направление всех осей (G1 X10, Y10, Z5)
- [ ] Проверить срабатывание концевиков (M119)
- [ ] Проверить срабатывание датчика зонда (M119 при поднесении металла)
- [ ] Выставить смещение датчика (M851 X Y Z)
- [ ] Выполнить G28, затем G29 (автолевелинг)
- [ ] Откалибровать E шаги (M92 E, M500)
- [ ] Выполнить PID автотюн (M303)
- [ ] Сохранить все настройки: M500

---

## Сборка прошивки

Прошивка собирается через **PlatformIO** в **Visual Studio Code**.

1. Установи [VS Code](https://code.visualstudio.com/)
2. Установи расширение **PlatformIO IDE**
3. Установи расширение **Auto Build Marlin**
4. Открой папку с прошивкой в VS Code
5. В панели Auto Build Marlin нажми **Build** или **Upload**

Target платформы для MKS TinyBee: `mks_tinybee`

---

## Полезные G-коды

| Команда | Описание |
|---|---|
| `G28` | Хоминг всех осей |
| `G29` | Автолевелинг (bilinear) |
| `M500` | Сохранить настройки в EEPROM |
| `M501` | Загрузить настройки из EEPROM |
| `M502` | Сброс к заводским настройкам |
| `M503` | Показать текущие настройки |
| `M92 E___` | Установить шаги экструдера |
| `M851 Z___` | Установить Z-offset датчика |
| `M303 E0 S200 C8` | PID автотюн хотэнда |
| `M303 E-1 S60 C8` | PID автотюн стола |
| `M119` | Статус концевиков и датчиков |
| `M412 S1` | Включить датчик филамента |

---

## Ссылки

- [Marlin Documentation](https://marlinfw.org)
- [MKS TinyBee Wiki](https://github.com/makerbase-mks/MKS-TinyBee)
- [Marlin Discord](https://discord.com/servers/marlin-firmware-461605380783472640)
- [Конфигурации Marlin](https://github.com/MarlinFirmware/Configurations)

---

## Лицензия

Marlin публикуется под лицензией [GPL](/LICENSE). Все изменения конфигурации в этом репозитории также распространяются свободно.
