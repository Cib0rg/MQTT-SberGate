# MQTT-SberGate

Агент-прослойка между облаком **Sber Smart Home** и **Home Assistant**. Забирает сущности из HA, регистрирует их в облаке Салюта и синхронизирует состояния в обе стороны через MQTT и REST API.

## Содержание

- [Поддерживаемые устройства](#поддерживаемые-устройства)
- [Установка и настройка](#установка-и-настройка)
- [Запуск в Docker](#запуск-в-docker)
- [Веб-интерфейс](#веб-интерфейс)
- [Отладка](#отладка)
- [Ссылки](#ссылки)

---

## Поддерживаемые устройства

Агент автоматически обнаруживает сущности HA и присваивает им категорию Sber. Категорию можно переопределить вручную в [веб-интерфейсе](#веб-интерфейс) через dropdown — изменение применяется немедленно без перезапуска.

| Категория Sber | Автоопределяется из HA | Описание |
|---|---|---|
| `relay` | `switch`, `script`, `button` | Реле, выключатель |
| `light` | `light` | Освещение |
| `sensor_temp` | `sensor` с `device_class=temperature` | Датчик температуры |
| `scenario_button` | `input_boolean` | Кнопка сценария (click / double_click) |
| `hvac_ac` | `climate` | Кондиционер |
| `hvac_radiator` | `hvac_radiator` | Радиатор отопления |
| `hvac_underfloor_heating` | переопределяется вручную | Тёплый пол |
| `intercom` | переопределяется вручную | Домофон |

### Кондиционер (`hvac_ac`)

Поддерживаемые функции: включение/выключение, целевая температура, текущая температура, **режим работы**.

Маппинг режимов между Sber и Home Assistant:

| Sber (`hvac_work_mode`) | Home Assistant (`hvac_mode`) |
|---|---|
| `cooling` | `cool` |
| `heating` | `heat` |
| `auto` | `auto` |
| `dehumidification` | `dry` |
| `ventilation` | `fan_only` |
| `fast_cooling` | `cool` |
| `fast_heating` | `heat` |
| `turbo` | `cool` |
| `eco` | `auto` |
| `comfortable_sleep` | `heat` |
| `air_purification` | `fan_only` |
| `self_cleaning` | `fan_only` |

### Тёплый пол (`hvac_underfloor_heating`)

Подключается как `climate`-сущность в HA. Для активации выбери категорию `hvac_underfloor_heating` в dropdown веб-интерфейса. При управлении через Салют всегда отправляет `hvac_mode: heat` в HA.

### Домофон (`intercom`)

Подключается как `switch`-сущность в HA. Для активации выбери категорию `intercom` в dropdown веб-интерфейса. Голосовая команда «открой домофон» отправляет `switch/turn_on` в HA.

---

## Установка и настройка

### 1. Регистрация в Sber Studio

1. Зарегистрируйся в [Sber Studio](https://developers.sber.ru/studio/workspaces/)
2. Создай интеграцию и получи `sber-mqtt_login` и `sber-mqtt_password`

### 2. Токен Home Assistant

В профиле пользователя HA создай долгосрочный токен доступа (`ha-api_token`). Рекомендуется завести отдельного пользователя.

### 3. Параметры конфигурации

| Параметр | По умолчанию | Описание |
|---|---|---|
| `ha-api_url` | `http://ha_server_ip:8123` | Адрес REST API Home Assistant |
| `ha-api_token` | — | Долгосрочный токен HA |
| `sber-mqtt_broker` | `mqtt-partners.iot.sberdevices.ru` | MQTT брокер Сбера |
| `sber-mqtt_broker_port` | `8883` | Порт брокера |
| `sber-mqtt_login` | — | Логин из Sber Studio |
| `sber-mqtt_password` | — | Пароль из Sber Studio |
| `sber-http_api_endpoint` | `https://mqtt-partners.iot.sberdevices.ru` | HTTP API Сбера |
| `log_level` | `info` | Уровень логирования (`trace`/`debug`/`info`/`warning`/`error`) |

---

## Запуск в Docker

```yaml
services:
  mqtt-sber-gate:
    image: mqtt-sber-gate
    ports:
      - "9123:9123"
    volumes:
      - ./data:/data
    environment:
      - PYTHONUNBUFFERED=1
    restart: unless-stopped
```

Конфигурация читается из `/data/options.json`:

```json
{
  "ha-api_url": "http://192.168.1.10:8123",
  "ha-api_token": "your_long_lived_token",
  "sber-mqtt_broker": "mqtt-partners.iot.sberdevices.ru",
  "sber-mqtt_broker_port": 8883,
  "sber-mqtt_login": "your_sber_login",
  "sber-mqtt_password": "your_sber_password",
  "sber-http_api_endpoint": "https://mqtt-partners.iot.sberdevices.ru",
  "log_level": "info"
}
```

---

## Веб-интерфейс

Доступен на порту `9123`. Открой `http://<host>:9123` в браузере.

Функции:
- Список всех устройств с текущими состояниями
- Включение/выключение устройств для синхронизации с Сбером (чекбокс **Включено**)
- **Dropdown выбора категории** — позволяет переопределить тип устройства, отображаемый в Sber Smart Home, для каждого устройства отдельно. Категория по умолчанию определяется автоматически из типа сущности HA
- Удаление базы устройств

---

## Отладка

Для включения подробного логирования JSON-сообщений (входящие команды от Сбера, исходящие payload в HA REST API) добавь переменную окружения:

```yaml
environment:
  - DEBUG=true
  - PYTHONUNBUFFERED=1
```

При `DEBUG=true` уровень логирования принудительно устанавливается в `debug`, вне зависимости от `log_level` в конфиге.

Лог доступен в файле `SberGate.log` — ссылка для скачивания есть в веб-интерфейсе.

---

## Ссылки

- [Sber Studio — регистрация](https://developers.sber.ru/studio/workspaces/)
- [Создание интеграции в Studio](https://developers.sber.ru/docs/ru/smarthome/mqtt-diy/create-mqtt-diy-integration-project)
- [Авторизация контроллера в облаке Sber](https://developers.sber.ru/docs/ru/smarthome/mqtt-diy/controller-authorization)
- [Как работает интеграция Sber](https://developers.sber.ru/docs/ru/smarthome/mqtt-diy/integration-scheme)
- [Категории устройств Sber C2C](https://developers.sber.ru/docs/ru/smarthome/c2c/devices)
- [hvac_ac — функции кондиционера](https://developers.sber.ru/docs/ru/smarthome/c2c/hvac_ac)
- [hvac_work_mode — режимы работы](https://developers.sber.ru/docs/ru/smarthome/c2c/hvac_work_mode)
- [intercom — функции домофона](https://developers.sber.ru/docs/ru/smarthome/c2c/intercom)
- [HA REST API](https://developers.home-assistant.io/docs/api/rest)
- [HA WebSocket API](https://developers.home-assistant.io/docs/api/websocket)
- [Eclipse Paho MQTT Python Client](https://github.com/eclipse/paho.mqtt.python)
- [Telegram-чат](https://t.me/+k_w9uO0h73FkNjJi)
