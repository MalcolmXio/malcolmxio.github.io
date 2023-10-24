# Data Storage Options

## Key-Value Storage (SharedPreferences)

Для хранений используется XML файл или бинарный файл.

Позволяет хранить пары ключ-значение. Хорошо подходит для простых неструктурированных, нечувствительных данных. (настройки, флаги итп)

Плюсы:

- легко использовать. Есть встроенный API

Минусы:

- не безопасно (но есть EncryptedSharedPreferences)
- не подходит для хранения больших объемов данных
- нет возможности запрашивать данных как в БД
- нет поддержки миграций
- плохая производительность

## Database/ORM (sqlite/Room/Realm)

Хорошо подходит для больших наборов данных, где нужна сложная логика выборки.

Плюсы:

- объеткно реляционный маппинг
- поддержка запросов (query) и schema
- поддержка миграций

Минусы:

- сложная настройка
- не безопасны (есть надстройки)
- занимают большой объем памяти

cursor feed example:

```Kotlin
item_id:        String
author_id:      String
title:          String
description:    String
likes:          Int
comments:       Int
attachments:    String # comma separated list
created_at:     Date   # also used for sorting
cursor_next_id: String # points to the next cursor page
cursor_prev_id: String # points to the prev cursor page
```

## Custom/Binary (DataStore)

Низкоуровневое хранение данных.

Плюсы:

- высокая кастомизируемость
- производительность

Минусы:

- нет поддержки миграции
- нет схемы
- большое количество настроек

## On-Device Secure storage (KeyStore)

Зашифрованное хранилище для хранения ключей шифрования и пар ключ-значение.

Плюсы:

- безопасность

Минусы:

- нет поддержки миграции
- нет схемы
- плохой перформанс
- не оптимизировано для хранения других типов данных, кроме ключей шифрования