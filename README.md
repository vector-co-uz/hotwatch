# 🔥 HotWatch — Система мониторинга котла отопления

[![Status](https://img.shields.io/badge/Status-Этап%202%20%28Прошивка%29-blue)]()
[![License](https://img.shields.io/badge/License-MIT-green)]()
[![ESPHome](https://img.shields.io/badge/ESPHome-2024.11%2B-orange)]()

**HotWatch** — микроконтроллерная система мониторинга и безопасности самодельного котла отопления с открытой системой. Базируется на **ESP32-C3**, **ESPHome** и **Home Assistant**.

> ⚡ **Локальная безопасность:** пламя, газ и перегрев срабатывают **моментально** — даже без интернета!

---

## 🎯 Основные возможности

| Функция | Компонент | Статус |
|---------|-----------|--------|
| 🔥 Обнаружение пламени | KY-026 ИК датчик | ✅ |
| 🌡️ Контроль температуры | DS18B20 ×2 (подача + обратка) | ✅ |
| 💧 Влажность и T° | DHT11 модуль | ✅ |
| 💨 Датчик дыма | MQ-2 газовый сенсор | ✅ |
| 🌍 Датчик газа | MQ-5 газовый сенсор | ✅ |
| 🚪 Открытие дверки | YL-63 ИК датчик | ✅ |
| 🔔 Локальная тревога | Баззер + LED | ✅ |
| 📡 WiFi + Home Assistant | Native API | ✅ |
| 📊 Дашборд | HA Sections/Tile | 🟡 (Этап 4) |
| 💬 Telegram уведомления | HA интеграция | 🟡 (Этап 3) |

---

## 📦 Требования

### Аппаратное обеспечение
- **Микроконтроллер:** ESP32-C3 SuperMini
- **Питание:** 2×18650 Li-ion (7.4 В) + DC-DC 5 В + регулятор 3.3 В
- **Датчики:** см. [BOM.md](BOM.md) (все куплены ✅)

### Программное обеспечение
- **Python 3.9+**
- **ESPHome 2024.11+**
- **Home Assistant 2024.11+** (опционально)

---

## 🚀 БЫСТРЫЙ СТАРТ

### 1️⃣ Собрать схему
```bash
# Используй BOM.md и схему питания из PROJECT_OVERVIEW.md
# Все компоненты уже куплены ✅
```

### 2️⃣ Установить ESPHome
```bash
pip install esphome
```

### 3️⃣ Клонировать репозиторий и настроить
```bash
git clone <этот_репозиторий> hotwatch && cd hotwatch
cp secrets.yaml.example secrets.yaml
# Отредактировать secrets.yaml (WiFi SSID, пароль и т.д.)
```

### 4️⃣ Загрузить прошивку
```bash
esphome run hotwatch.yaml
```

### 5️⃣ Протестировать
```bash
# Открыть веб-интерфейс:
# http://<IP_ESP32>:80

# Ожидаемые результаты:
# ✓ Зелёный LED мигает (статус ОК)
# ✓ Красный LED не горит (нет тревоги)
# ✓ Датчики показывают значения
```

👉 **Подробная инструкция:** см. [QUICKSTART.md](QUICKSTART.md)

---

## 📋 ДОКУМЕНТАЦИЯ

| Файл | Описание |
|------|---------|
| [BOM.md](BOM.md) | 📦 Полный список компонентов и схемы |
| [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) | 📋 Описание проекта, этапы разработки, ограничения |
| [QUICKSTART.md](QUICKSTART.md) | 🚀 Быстрый старт (5 шагов) |
| [hotwatch.yaml](hotwatch.yaml) | 🔧 Основная конфигурация ESPHome |
| [secrets.yaml.example](secrets.yaml.example) | 🔐 Пример переменных WiFi и API |

---

## 🔌 GPIO распределение (ESP32-C3)

```
GPIO0  → 1-Wire (DS18B20: подача + обратка)
GPIO1  → DHT11 DATA
GPIO2  → Кнопка сброса тревоги (вход pull-up)
GPIO3  → Кнопка тест (вход pull-up)
GPIO4  → MQ-2 аналоговый (ADC1, делитель 10k-10k)
GPIO5  → (резерв, недоступен при Wi-Fi)
GPIO6  → YL-63 ИК датчик (вход)
GPIO7  → 🔴 LED тревога (выход)
GPIO10 → 🟢 LED статус (выход)
GPIO20 → 🔊 Баззер (выход)
GPIO21 → MQ-5 DO дискретный (вход)
GPIO22 → I2C SCL (резерв)
GPIO23 → I2C SDA (резерв)
```

---

## ⚡ ПИТАНИЕ

```
2×18650 (7.4 В) 
    ↓ DC-DC LM2596 (7.4 В → 5.0 В)
    ├─→ Все датчики (5 В)
    ├─→ Баззер (5 В)
    └─→ Регулятор AMS1117 (5 В → 3.3 В)
            ↓
        ESP32-C3 (3.3 В)
        LED (3.3 В)
```

**Ограничения:**
- ⚠️ На входы ESP32: **макс 3.3 В** (выше = повреждение)
- ✅ 5 В датчики → делитель напряжения 10k-10k

---

## 🧪 ТЕСТИРОВАНИЕ

### Локальная безопасность (работает БЕЗ интернета)

```bash
# Тест 1: Кнопка "Тест"
# Результат: RED LED ON + Баззер ON на 5 сек ✓

# Тест 2: Датчик огня (KY-026 + зажигалка)
# Результат: RED LED ON + Баззер ON (пока есть огонь) ✓

# Тест 3: Датчик газа (MQ-2 + дым)
# Результат: RED LED ON + Баззер ON (когда ADC > 400) ✓

# Тест 4: Кнопка "Сброс тревоги"
# Результат: RED LED OFF + Баззер OFF ✓

# Тест 5: WiFi Watchdog (отключить WiFi на 5+ мин)
# Результат: RED LED ON + Баззер ON (нет связи) ✓
```

---

## 🏛️ АРХИТЕКТУРА

```
┌─────────────────────────────────────────────────────────────┐
│                     HOTWATCH СИСТЕМА                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  УРОВЕНЬ 1: МИКРОКОНТРОЛЛЕР (ESP32-C3)                     │
│  ├─ Датчики (7 шт): огонь, T°, газ, ИК, влажность        │
│  ├─ Локальная безопасность: LED + Баззер (АВТОНОМНАЯ)     │
│  ├─ Логирование и фильтрация                               │
│  └─ WiFi подключение                                        │
│       ↓                                                      │
│  УРОВЕНЬ 2: HOME ASSISTANT (опционально)                   │
│  ├─ Telegram уведомления                                    │
│  ├─ Дашборд с графиками                                    │
│  ├─ Автоматизации и правила                               │
│  └─ История данных (InfluxDB)                              │
│       ↓                                                      │
│  УРОВЕНЬ 3: МОБИЛЬНОЕ ПРИЛОЖЕНИЕ                           │
│  └─ Уведомления на телефон (через Telegram)               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔐 БЕЗОПАСНОСТЬ

### Локальная безопасность
✅ Тревога срабатывает **даже без интернета** (автономная)  
✅ Пламя, газ, потеря WiFi → немедленная сигнализация  
✅ Баззер и LED включаются напрямую (не зависят от HA)

### Защита данных
🔐 WiFi пароль в `secrets.yaml` (не в репозитории)  
🔐 API encryption key для Home Assistant  
🔐 `.gitignore` исключает все секреты

### Рекомендации
1. Не коммитьте `secrets.yaml` в git
2. Используйте сильный WiFi пароль
3. Регулярно обновляйте ESPHome
4. Тестируйте локальную безопасность ежемесячно

---

## 📈 ЭТАПЫ РАЗРАБОТКИ

| Этап | Описание | Статус |
|------|---------|--------|
| 1 | Проектирование, BOM, GPIO, схемы | ✅ Завершён |
| 2 | Прошивка ESPHome, локальная безопасность | 🟡 **Текущий** |
| 3 | Home Assistant, Telegram алерты | 🔴 Планируется |
| 4 | Дашборд HA с графиками | 🔴 Планируется |
| 5 | Тестирование, финальная документация | 🔴 Планируется |

---

## 🐛 ИЗВЕСТНЫЕ ПРОБЛЕМЫ

| Проблема | Решение | Статус |
|----------|---------|--------|
| WiFi не подключается | Проверить SSID/пароль в secrets.yaml | ⏳ |
| Датчики не показывают значения | Проверить напряжение питания (5 В / 3.3 В) | ⏳ |
| Баззер не пикает | Проверить GPIO20 питание, тестировать кнопкой | ⏳ |
| Red LED горит без причины | Обычно = потеря WiFi (Watchdog) | ⏳ |

---

## 🤝 КАК ПОМОЧЬ

1. **Баги:** Open issue с логами (`esphome logs hotwatch.yaml`)
2. **Улучшения:** Fork → branch → commit → PR
3. **Документация:** Любые добавления приветствуются

---

## 📚 ССЫЛКИ И РЕСУРСЫ

- 🌐 **ESPHome:** https://esphome.io/
- 🏠 **Home Assistant:** https://www.home-assistant.io/
- 📘 **ESP32-C3 docs:** https://docs.espressif.com/projects/esp-idf/en/latest/esp32c3/
- 🔧 **DS18B20 datasheet:** https://www.maximintegrated.com/en/products/ibutton/ibutton_legacy/DS18B20.html
- 🎓 **DHT11 справка:** https://lastminuteengineers.com/dht11-temperature-humidity-sensor-arduino-tutorial/

---

## 📄 ЛИЦЕНЗИЯ

MIT License — используй как угодно, но с указанием авторства.

```
Copyright (c) 2026 Vector

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

---

## 👤 АВТОР И АССИСТЕНТ

| Роль | Имя | Дата |
|------|-----|------|
| **Инженер** | Vector | 2026-11-13 |
| **Ассистент AI** | Claude Haiku 4.5 | 2026-11-13 |
| **Язык** | Русский (RU) | |
| **Проект** | HotWatch v1.0 | |

---

## 📞 БЫСТРЫЕ ССЫЛКИ

- 🚀 [Быстрый старт](QUICKSTART.md)
- 📦 [BOM (компоненты)](BOM.md)
- 📋 [Обзор проекта](PROJECT_OVERVIEW.md)
- 🔧 [Конфигурация ESPHome](hotwatch.yaml)
- 🔐 [Пример секретов](secrets.yaml.example)

---

**Версия:** 1.0  
**Последнее обновление:** 2026-11-13  
**Статус:** 🟡 Готово к прошивке (Этап 2)

> 🎉 **Поздравляем!** Ты готов к загрузке прошивки на ESP32-C3.  
> Следуй инструкции в [QUICKSTART.md](QUICKSTART.md) 🚀
