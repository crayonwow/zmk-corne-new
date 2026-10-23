# ZMK Firmware для Jorne-совместимой клавиатуры

**Статус**: ✅ **РАБОЧАЯ КОНФИГУРАЦИЯ**

https://nickcoutsos.github.io/keymap-editor/

Полностью функциональная конфигурация ZMK для беспроводной клавиатуры Jorne-WL v3.0.1 на nice!nano v2.

## Быстрый старт

### Требования
- Клавиатура Jorne-WL v3.0.1 с контроллерами nice!nano v2
- GitHub репозиторий с GitHub Actions (для автоматической сборки)
- Или локальное окружение ZMK (west, Zephyr 3.x)

### Шаг 1: Сборка прошивки

**Вариант A: GitHub Actions (рекомендуется)**
1. Форк этого репозитория
2. Пуш в GitHub
3. Перейти на вкладку Actions → ждём сборки
4. Скачать артефакты `firmware.zip`

**Вариант B: Локальная сборка**
```bash
west build -b nice_nano_v2 -d build/jorne_left -- -DSHIELD=jorne_left
west build -b nice_nano_v2 -d build/jorne_right -- -DSHIELD=jorne_right
```

### Шаг 2: Прошивка контроллеров

Для каждой половины клавиатуры:

1. **Подключить USB-C** контроллер к компьютеру
2. **Активировать UF2 bootloader**:
   - Двойной клик кнопку RESET (если есть), или
   - Замкнуть контакты RST и GND на 1 сек
3. **Контроллер появится как USB диск** (NICENANO или аналогичный)
4. **Скопировать `.uf2` файл** на диск:
   - Левая половина: `jorne_left-nice_nano_v2-zmk.uf2`
   - Правая половина: `jorne_right-nice_nano_v2-zmk.uf2`
5. **Диск автоматически отключится** (прошивка завершена)

### Шаг 3: Bluetooth паринг

1. **Включить батарею** на обеих половинах
2. **Система автоматически распознает** устройство "Corne" в Bluetooth
3. **Нажать "Подключить"** в диалоге паринга
4. ✅ Клавиатура готова к использованию

### Проблемы при подключении?

**Если клавиатура не видна в Bluetooth:**

1. **Убедиться, что обе половины прошиты нормальной прошивкой**
   - (не `settings_reset` — в ней Bluetooth отключен)

2. **Сбросить Bluetooth состояние:**
   - Прошить `settings_reset-nice_nano_v2-zmk.uf2` на обе половины
   - Дождаться перезагрузки
   - Прошить нормальную прошивку (`jorne_left` и `jorne_right`)
   - Переподключиться

3. **Удалить старые Bluetooth девайсы (macOS):**
   - System Settings → Bluetooth → Забыть старые "Corne"
   - Переподключиться

4. **Проверить наличие батареи и питание**

## Конфигурация

### Основные параметры
- **ZMK версия**: v0.2.1 (стабильный релиз, перед Zephyr 4.1)
- **Board**: `nice_nano_v2`
- **Shields**: `jorne_left`, `jorne_right`
- **Слои**: 3 (base, symbols/BT, numbers/navigation)

### Раскладка

#### Base layer
```
| ESC |  Q  |  W  |  E  |  R  |  T  |    |  Y  |  U  |  I  |  O  |  P  | BKSP|
| TAB |  A  |  S  |  D  |  F  |  G  |    |  H  |  J  |  K  |  L  |  '  | ' |
|SHFT |  Z  |  X  |  C  |  V  |  B  |    |  N  |  M  |  ,  |  .  |  /  |SHFT|
           | CMD | L1  | SPC |  ENT | L2 | ALT |
```

#### L1 layer (Symbols + Bluetooth)
- `BT_CLR` — очистить все Bluetooth профили
- `BT_SEL 0-4` — выбрать профиль (на левой половине)
- Стрелки, громкость, Cmd+C/V/X

#### L2 layer (Numbers + Navigation)
- Numpad (левая сторона)
- Стрелки, Home/End, Page Up/Down (правая сторона)

### Custom behaviors
- **hml/hmr**: Hold-Tap для левых/правых модификаторов (CTRL, ALT, SHIFT, CMD)
- **tkl/tkr**: Layer toggle (hold для слоя, tap для клавиши)

### Комбо
- `Ctrl+Alt` → Toggle game mode (упрощённый layer без модификаторов)
- `Q+W` → ESC

## Структура репозитория

```
.
├── config/
│   ├── jorne.conf          # ZMK конфигурация (BT, sleep и т.д.)
│   ├── jorne.keymap        # Раскладка (3 слоя)
│   └── west.yml            # Версия ZMK (v0.2.1)
├── build.yaml              # GitHub Actions матрица сборки
├── .github/workflows/
│   └── build.yml           # Конфиг для CI/CD
├── docs/
│   ├── deep-research.md    # Анализ аппаратной части
│   ├── configuration-summary.md  # Финальная конфигурация
│   └── README.md           # Этот файл
└── firmware/               # Бинарные прошивки (результаты сборки)
```

## Файлы конфигурации

### `config/jorne.conf`
Минимальная конфигурация для стабильного BT:
```ini
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y  # Увеличить мощность Bluetooth
# CONFIG_ZMK_RGB_UNDERGLOW=y     # RGB (если есть)
# CONFIG_ZMK_DISPLAY=y           # OLED дисплей (если есть)
```

**Почему минимальная?**
- Pointing device отключен (требует hardware)
- Экспериментальные BLE опции отключены (стабильность)
- Sleep отключён (не нужен для большинства)

### `config/jorne.keymap`
Раскладка для Jorne (44 позиции):
- Row 1 начинается с `&none` (фантомная клавиша, физически отсутствует)
- Custom behaviors для удобного использования модификаторов
- 3 слоя для всех нужных функций

### `config/west.yml`
```yaml
revision: v0.2.1  # ZMK версия (важно для совместимости)
```

**Почему v0.2.1, а не main?**
- main ZMK включает Zephyr 4.1 с изменениями в board definitions
- v0.2.1 — последний стабильный релиз, где `nice_nano_v2` работает корректно
- Гарантирует совместимость с прошивкой продавца

## Редактирование раскладки

### Добавить новую клавишу
Найти нужную позицию в первом ряду нужного слоя и заменить:
```
&kp ESC → &kp ENTER  # ESC → ENTER
```

### Изменить поведение при удержании
Использовать `&mt` (mod-tap) или `&lt` (layer-tap):
```
&mt LEFT_CTRL A  # Ctrl при удержании, A при тапе
&lt 1 SPACE      # Layer 1 при удержании, Space при тапе
```

### Добавить новый слой
1. Скопировать существующий слой в keymap
2. Переименовать (например, `layer_4`)
3. Добавить в keymap block

## Обновление и техническое обслуживание

### Обновить версию ZMK
```yaml
# config/west.yml
revision: v0.3.0  # или main
```
**Внимание**: При переходе на main (Zephyr 4.1) нужно изменить:
- `board: nice_nano_v2` → `board: nice_nano`
- Переименовать shields (если требуется)

### Добавить опции (RGB, display, sleep)
1. Раскомментировать в `config/jorne.conf`
2. Пересобрать
3. Протестировать

### Сбросить Bluetooth
```bash
# Прошить settings_reset на обе половины
# Потом прошить нормальную прошивку
```

## Известные ограничения

- ⚠️ Pointing device не поддерживается (нет hardware)
- ⚠️ RGB Underglow закомментирован (нет hardware)
- ⚠️ OLED Display закомментирован (нет hardware)
- ⚠️ ZMK v0.2.1 — старая версия, но стабильная

## Ссылки и ресурсы

- **ZMK Документация**: https://zmk.dev
- **Официальный Jorne конфиг**: https://github.com/joric/jorne-zmk-config
- **ZMK GitHub**: https://github.com/zmkfirmware/zmk
- **Bluetooth паринг troubleshooting**: https://zmk.dev/docs/troubleshooting/connection-issues

## История версий

**v1.0 (2026-02-15)** — Финальная рабочая конфигурация
- ZMK v0.2.1 (стабильный)
- Jorne shields (jorne_left, jorne_right)
- nice_nano_v2
- 3 слоя, custom behaviors, combos
- ✅ Bluetooth паринг работает
- ✅ Все клавиши функционируют

## Контакты и вопросы

При возникновении проблем:
1. Посмотреть раздел "Проблемы при подключении?" выше
2. Проверить `docs/configuration-summary.md` для деталей
3. Проверить официальную документацию ZMK

---

**Последнее обновление**: 15 февраля 2026

🤖 Собрано с помощью Claude Code
