# RU Books MCP

Публичный MCP-сервер для поиска и подбора русскоязычных книг.

- Имя в MCP Registry: `io.github.books-service/ru-books`
- Транспорт: Streamable HTTP
- Endpoint: `https://knigochit.ru/mcp`
- Аутентификация: не требуется (без API key/токена)
- Health: `https://knigochit.ru/health`

## Подключение агента

Минимально необходимое:

- Base URL MCP-сервера: `https://knigochit.ru/mcp`
- Аутентификация: не требуется (API key/токен не нужны)

Дополнительно (если клиент спрашивает transport):

- `streamable-http`

Поддерживаемые параметры (кратко):

- `books_search`: `query`, `limit`, `offset`
- `books_by_genre`: `genre`, `limit`, `offset`
- `books_by_author`: `author`, `limit`, `offset`
- `books_by_language`: `language`, `limit`, `offset`
- `books_by_format`: `format` (`text`/`audio`/`book`/`ebook`), `limit`, `offset`
- `books_without_series`: `limit`, `offset`
- `recommendation_candidates`: `user_query`, `limit`, `filters.author`, `filters.genre`, `filters.language`, `filters.min_price`, `filters.max_price`, `filters.without_series`, `filters.formats`
- `book_details`: `book_id`

Сейчас не поддерживается:

- `filters.popularity` (и другие нестандартные поля)

Важно:

- Неподдерживаемые аргументы инструментов возвращаются как явная ошибка в `tools/call` (`isError: true`).
- Неподдерживаемые фильтры возвращаются как явная ошибка в `tools/call` (`isError: true`).
- `books_search.filters` не входит в публичный контракт `tools/list` и доступен только в legacy-режиме совместимости.
- В текущем production-профиле legacy-режим `books_search.filters` отключен.
- Browse-инструменты (`books_by_*`, `books_without_series`) возвращают `items`, `has_more`, `next_offset` и не возвращают полный `total` (для быстрого ответа на большом каталоге).
- Для browse-запросов используйте профильные инструменты:
  - жанр: `books_by_genre`
  - автор: `books_by_author`
  - формат: `books_by_format`
  - язык: `books_by_language`
  - без серии: `books_without_series`

## Как выбирать инструмент

- Любые книги по жанру: `books_by_genre`
- Любые книги по автору: `books_by_author`
- Любые книги по языку: `books_by_language`
- Любые книги по формату: `books_by_format`
- Любые книги без серии: `books_without_series`
- Свободный текстовый поиск (название/описание/метаданные): `books_search`
- Детальная карточка книги: `book_details`

Анти-паттерны:

- Не используйте `books_search` для browse-задач по жанру/автору/формату/языку.
- Не используйте `recommendation_candidates` для простого «дай N книг по жанру» (для этого есть `books_by_genre`).
- Не ожидайте работу `books_search.filters` в production: этот legacy-режим отключен.

## Инструменты

- `books_search` — текстовый поиск по запросу.
- `books_by_genre` — витринный просмотр книг по жанру без текстового запроса.
- `books_by_author` — витринный просмотр книг по автору.
- `books_by_language` — витринный просмотр книг по языку.
- `books_by_format` — витринный просмотр книг по формату.
- `books_without_series` — витринный просмотр книг без серии.
- `book_details` — получение подробной карточки по `book_id`.
- `recommendation_candidates` — получение кандидатов для ранжирования на стороне агента.

## Быстрая проверка

### Health

```bash
curl https://knigochit.ru/health
```

### MCP initialize

```bash
curl -X POST https://knigochit.ru/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-11-05","capabilities":{},"clientInfo":{"name":"manual-client","version":"0.1.0"}}}'
```

### MCP tools/list

```bash
curl -X POST https://knigochit.ru/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'
```

### Пример: «любые 5 книг жанра фантастика»

```bash
curl -X POST https://knigochit.ru/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"books_by_genre","arguments":{"genre":"фантастика","limit":5}}}'
```

## Примечания

- Сервер возвращает данные поиска и наборы кандидатов.
- Финальная рекомендация для пользователя формируется агентом/клиентом.
