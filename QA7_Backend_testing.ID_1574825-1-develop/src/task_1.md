# Сравнение публичных REST API Т-Банка и Wildberries

## Адреса документации API

| Сервис | Ссылка на документацию |
|------|------------------------|
| Т-Банк | [https://developer.tbank.ru/](https://developer.tbank.ru/) |
| Wildberries | [https://dev.wildberries.ru/](https://dev.wildberries.ru/) |

---

## Сравнительная таблица API

### 1. Основные эндпоинты и поддерживаемые методы

| Характеристика | Т-Банк (T-Bank API) | Wildberries (WB API) |
|----------------|-------------------------|--------------------------|
| Основное назначение | Финансовые операции, платежи, эквайринг, инвестиции | Управление маркетплейсом: товары, заказы, остатки, цены |
| Базовый URL | `https://securepay.tinkoff.ru/v2/` (платежи) <br>`https://api-invest.tinkoff.ru/openapi/` (инвестиции) | `https://suppliers-api.wildberries.ru` <br>`https://suppliers-stats.wildberries.ru/api/v1/` (статистика) |
| Популярные эндпоинты | • `POST /Init` — инициировать платеж <br>• `POST /AddAccountQr` — привязать счет через СБП <br>• `POST /userinfo/userinfo` — получить данные пользователя | • `GET /api/v1/supplier/orders` — заказы <br>• `GET /api/v1/supplier/stocks` — остатки на складе <br>• `POST /api/v2/orders` — обновление статуса заказов <br>• `POST /content/v2/get/cards/list` — список товаров |
| Методы | В основном `POST`, реже `GET` | Широкий спектр: `GET`, `POST`, `PUT`, `DELETE` |

---

### 2. Форматы запроса и ответа

| Характеристика | Т-Банк (T-Bank API) | Wildberries (WB API) |
|----------------|-------------------------|--------------------------|
| Формат данных | `application/json` | `application/json` |
| Пример запроса (платёж) | ```json { "TerminalKey": "TBankTest", "Amount": 10000, "OrderId": "ORDER_001", "Token": "signature" } ``` | ```json { "sort": {}, "filter": { "tagIDs": [null], "objectIDs": [null], "brands": [null] }, "cursor": {} } ``` |
| Пример ответа (платёж/привязка) | ```json { "TerminalKey": "TBankTest", "Data": "https://sub.nspk.ru/...", "Success": true, "ErrorCode": "0", "Message": "OK" } ``` | ```json { "data": [ { "id": 123, "name": "Товар", "price": 1000 } ] } ``` |
| Формат даты/времени | `YYYY-MM-DDTHH24:MI:SS+GMT` | RFC3339 (например, `2019-06-20T00:00:00Z`) |

---

### 3. Особенности авторизации и версии API

| Характеристика | Т-Банк (T-Bank API) | Wildberries (WB API) |
|----------------|-------------------------|--------------------------|
| Способ авторизации | `Authorization: Bearer {TOKEN}` | Для статистики: параметр `key` в URL <br>Для остальных: `Authorization: {token}` <br>Для сервисов: `X-Client-Secret` + `Authorization` |
| Типы токенов | • Токен для песочницы (sandbox)<br>• Токен для Production (3 месяца жизни) | • Персональный токен (Personal)<br>• Базовый токен (Base)<br>• Сервисный токен (Service) <br>Срок действия — 180 дней |
| Где получить токен | В настройках Т-Инвестиций | В разделе "Профиль → Настройки → Доступ к API" |
| Версионирование | Через URL: `/v2/Init`, `/v2/AddAccountQr` | Через URL и заголовки: `/api/v1/`, `/api/v2/`, `/api/v3/`, `/content/v2/` |
| Песочница (Sandbox) |  Есть (`/openapi/sandbox/`) |  Есть, изолированная среда |

---

### 4. Отличия в подходах к дизайну и структуре

| Характеристика | Т-Банк (T-Bank API) | Wildberries (WB API) |
|----------------|-------------------------|--------------------------|
| Архитектурный стиль | Классический REST + элементы RPC (много POST-методов) | RESTful с ресурсно-ориентированными URL |
| Подпись запросов |  Требуется `Token` — подпись запроса для безопасности |  Не требуется, только авторизационный токен |
| Наличие чеков (Фискализация) |  Встроенная поддержка чеков (параметр `Receipt` с тегами ФФД) |  Не предусмотрено (через маркетплейс автоматически) |
| Целевая аудитория | Разработчики интернет-магазинов, финтех-проекты, торговые роботы | Продавцы на Wildberries, SaaS-партнёры |
| Модель монетизации | Комиссия за транзакции (стандартный эквайринг) | Pay-as-you-go (плата за запросы после превышения лимитов) |
| Лимиты запросов | Не публикуются открыто | Алгоритм Token Bucket, заголовки `X-Ratelimit-Retry` |
| Поддержка OAuth 2.0 |  Есть (Tinkoff ID) |  Для партнёров в Каталоге решений |
| Swagger/OpenAPI |  Есть (инвестиционное API) |  Есть (`/swagger.yaml`) |

---

## Анализ JSON-структуры на примере API Wildberries

### Выбранный эндпоинт

Ссылка на документацию: [https://dev.wildberries.ru/](https://dev.wildberries.ru/) → API управления товарами → `POST /content/v2/get/cards/list` (получение списка товаров)

### Пример JSON-ответа

```json
{
  "data": [
    {
      "id": 1234567,
      "vendorCode": "ТОВАР-001",
      "title": "Смартфон XYZ",
      "description": "Мощный смартфон с отличной камерой",
      "brand": "BrandName",
      "price": 19990,
      "discountPrice": 14990,
      "sizes": [
        {
          "name": "Стандарт",
          "price": 19990,
          "skus": ["SKU1234567890"]
        }
      ],
      "photos": [
        {
          "url": "https://images.wb.ru/...",
          "default": true
        }
      ],
      "needKiz": false,
      "kizMarked": false
    }
  ],
  "cursor": {
    "total": 42,
    "limit": 100,
    "nextCursor": "cursor_value_for_pagination"
  }
}

```

## Структура JSON: ключи и типы данных

| Ключ | Тип данных | Описание |
|------|------------|----------|
| `data` | Array | Массив объектов товаров |
| `data[].id` | Number | Уникальный идентификатор товара |
| `data[].vendorCode` | String | Артикул продавца |
| `data[].title` | String | Название товара (максимум 50 символов) |
| `data[].description` | String | Описание товара |
| `data[].brand` | String | Бренд товара |
| `data[].price` | Number | Цена товара в рублях |
| `data[].discountPrice` | Number | Цена со скидкой (если есть) |
| `data[].sizes` | Array | Массив размеров/вариантов товара |
| `data[].sizes[].name` | String | Название размера |
| `data[].sizes[].price` | Number | Цена для данного размера |
| `data[].sizes[].skus` | Array of String | Массив SKU-кодов товара |
| `data[].photos` | Array | Массив фотографий товара |
| `data[].photos[].url` | String | URL изображения |
| `data[].photos[].default` | Boolean | Признак основного изображения |
| `data[].needKiz` | Boolean | Требуется ли маркировка (КиЗ) |
| `data[].kizMarked` | Boolean | Подтверждена ли маркировка |
| `cursor` | Object | Объект пагинации |
| `cursor.total` | Number | Общее количество товаров |
| `cursor.limit` | Number | Лимит записей в ответе |
| `cursor.nextCursor` | String | Курсор для получения следующей страницы |
