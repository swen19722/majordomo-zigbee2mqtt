# Zigbee2MQTT для MajorDoMo

Модуль получает метаданные устройств из сохранённого брокером сообщения `bridge/devices` и показания из обычных тем Zigbee2MQTT. В карточке доступны модель, производитель, бренд, описание, IEEE-адрес, питание и возможности. При переименовании в Zigbee2MQTT имя в MajorDoMo сохраняется, а темы показаний и команд обновляются по IEEE-адресу. Для изображений используется каталог модуля. Старый ответ `bridge/log` с типом `devices` также поддерживается.

Проект основан на модуле [directman66](https://github.com/directman66/majordomo-zigbee2mqtt). Лицензия [Apache 2.0](LICENSE). Прежняя инструкция доступна в [исходной версии](https://github.com/directman66/majordomo-zigbee2mqtt/blob/29195aa2afeac9ed6796c42662dc47f96cd9e196/readme.md).

## Установка и обновление

1. На чистой установке MajorDoMo сначала установите Zigbee2MQTT через Маркет дополнений, чтобы MajorDoMo зарегистрировал модуль и создал его таблицы. Для уже установленного модуля сразу переходите к резервной копии.
2. Остановите только цикл `cycle_zigbee2mqtt.php` штатным способом вашей установки. В закрытом каталоге вне веб-корня сохраните `modules/zigbee2mqtt`, `templates/zigbee2mqtt`, `scripts/cycle_zigbee2mqtt.php` и `img/modules/zigbee2mqtt.png`. Сделайте дамп таблиц модуля. Укажите имя базы MajorDoMo вместо `DB_NAME` и используйте локальные настройки подключения MySQL.

   ```sh
   umask 077
   mysqldump DB_NAME zigbee2mqtt zigbee2mqtt_devices zigbee2mqtt_devices_list \
     zigbee2mqtt_devices_command zigbee2mqtt_bind zigbee2mqtt_grouplist zigbee2mqtt_log \
     > zigbee2mqtt-before.sql
   ```

   Если в базе есть таблица `zigbee2mqtt_queue`, включите её в дамп.

3. В базе MajorDoMo однократно добавьте поля в таблицу модуля. Если часть полей уже существует, добавьте только отсутствующие по результату `SHOW COLUMNS FROM zigbee2mqtt_devices`.

   ```sql
   ALTER TABLE zigbee2mqtt_devices
     ADD COLUMN Z2M_TOPIC VARCHAR(255) NOT NULL DEFAULT '',
     ADD COLUMN Z2M_DESCRIPTION TEXT NULL,
     ADD COLUMN Z2M_EXPOSES TEXT NULL;
   ```

4. Скопируйте из архива версии каталоги `modules`, `templates`, `scripts` и `img` в корень MajorDoMo с сохранением структуры и прав чтения. Для действующей установки после сверки с резервной копией достаточно заменить класс, цикл и шаблоны `devices.html` и `edit_device.html`. Используйте штатные настройки MQTT модуля. Для базовой темы `zigbee2mqtt` фильтр имеет вид `zigbee2mqtt/#`. Запустите цикл снова и откройте карточку датчика.

## Откат и восстановление

Для отката остановите цикл, верните файлы из резервной копии и снова запустите цикл. Добавленные поля можно оставить. Если требуется вернуть данные на момент обновления, сначала сохраните текущие данные, затем восстановите таблицы из дампа с учётом изменений после обновления. Команда удаления модуля в MajorDoMo удаляет его таблицы.

После переустановки MajorDoMo установите модуль по шагам выше. Чтобы перенести прежние устройства и привязки, восстановите сохранённые данные модуля и соответствующие объекты MajorDoMo из своей локальной резервной копии. Архив проекта содержит исходники, цикл, шаблоны и изображения.
