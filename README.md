# holidays.json

Файл `holidays.json` содержит список праздничных дат на один календарный
год в формате JSON.
Он может использоваться в боте (например, Discord-боте) или любом другом
приложении, которое должно:

-   определять, есть ли сегодня праздник,
-   показывать описание текущего праздника,
-   считать, сколько дней осталось до ближайшего праздника.

## Структура файла

Корневая структура:

``` json
{
  "year": 2026,
  "holidays": [
    {
      "name": "Новый Год",
      "description": "...",
      "start": "01.01",
      "end": "09.01"
    }
  ]
}
```

### Поля

#### `year`

-   Тип: число
-   Назначение: календарный год, для которого составлен список
    праздников.
-   Пример: `2026`

#### `holidays`

-   Тип: массив объектов
-   Каждый объект описывает один праздник и имеет поля:

``` json
{
  "name": "Новый Год",
  "description": "**Новый год в России** – государственный праздник, ...",
  "start": "01.01",
  "end": "09.01"
}
```

##### `name`

-   Тип: строка
-   Назначение: название праздника.

##### `description`

-   Тип: строка
-   Назначение: расширенное описание праздника.
-   Поддерживает **Markdown**.

##### `start` и `end`

-   Тип: строка
-   Формат: `ДД.ММ`
-   Указывают диапазон праздника **включительно**.

## Принцип работы с файлом в приложении / боте

### Пример кода на Python

``` python
import json
from datetime import date, datetime

with open("holidays.json", "r", encoding="utf-8") as f:
    data = json.load(f)

year = data["year"]
holidays = data["holidays"]

def _parse_dm(dm: str, year: int) -> date:
    day, month = map(int, dm.split("."))
    return date(year, month, day)

def get_holidays_for_date(target_date: date):
    result = []
    for h in holidays:
        start = _parse_dm(h["start"], target_date.year)
        end = _parse_dm(h["end"], target_date.year)
        if start <= target_date <= end:
            result.append(h)
    return result

def get_next_holiday(target_date: date):
    upcoming = []
    for h in holidays:
        start = _parse_dm(h["start"], target_date.year)
        if start >= target_date:
            upcoming.append((start, h))
    if not upcoming:
        return None
    upcoming.sort(key=lambda x: x[0])
    return upcoming[0]  # (date, holiday_dict)

today = date.today()
today_holidays = get_holidays_for_date(today)
next_holiday = get_next_holiday(today)
```

Использование:

-   если `today_holidays` не пустой --- сегодня праздник;
-   если пустой --- берём `next_holiday` и считаем количество дней до
    него.

## Правила редактирования

1.  Сохраняйте JSON-формат.
2.  Всегда должны быть поля `year` и `holidays`.
3.  Праздники указываются в формате `ДД.ММ`.
4.  Описание (`description`) можно форматировать Markdown.

## Локализация

-   Кодировка файла --- **UTF-8**.
-   По умолчанию используется русский язык.
-   При желании можно создать отдельные JSON-файлы для разных языков.

## Возможные сценарии использования

-   отправка автоматических сообщений о праздниках в Discord/Telegram;
-   расчёт дней до ближайшего праздника;
-   использование в календарях и виджетах;
-   генерация праздничных embed-ов.
