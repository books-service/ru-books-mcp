# RU Books MCP

Публичный MCP-сервер для поиска и подбора русскоязычных книг.

- Имя в MCP Registry: `io.github.books-service/ru-books`
- Транспорт: Streamable HTTP
- Endpoint: `https://knigochit.ru/mcp`
- Аутентификация: не требуется (без API key/токена)
- Health: `https://knigochit.ru/health`

## Для кого эта страница

- Для пользователей и агентов, которым нужно подключиться к серверу и вызывать инструменты.
- Здесь нет внутренних планов, деплой-заметок и другой внутренней документации разработки.

## Возможности

- Поиск книг по свободному тексту: `books_search`
- Browse-сценарии по явным атрибутам: `books_by_genre`, `books_by_author`, `books_by_language`, `books_by_format`, `books_without_series`
- Детальная карточка книги: `book_details`
- Кандидаты для ранжирования агентом: `recommendation_candidates`

## Установка

Отдельная установка сервера не требуется: сервис уже развернут публично.

Нужен только MCP-клиент/агент с поддержкой Streamable HTTP.

## Подключение

Минимальные параметры:

- Base URL: `https://knigochit.ru/mcp`
- Transport: `streamable-http`
- Аутентификация: не требуется

Если ваш клиент использует JSON-конфиг MCP-серверов, обычно достаточно такого фрагмента:

```json
{
  "mcpServers": {
    "ru-books": {
      "url": "https://knigochit.ru/mcp",
      "transport": "streamable-http"
    }
  }
}
```

Примечание: формат конфигурации может немного отличаться в разных клиентах, но `url` и transport всегда те же.

## Быстрый старт для пользователей

### Вариант 1: попросить агента подключить MCP

Можно дать агенту такой текст:

```text
Подключи MCP-сервер:
- URL: https://knigochit.ru/mcp
- transport: streamable-http
- аутентификация не требуется

После initialize вызови tools/list и используй его как источник истины для этой сессии.
```

## Как выбирать инструмент

- Любые 5 книг по жанру: `books_by_genre`
- Любые 5 книг по автору: `books_by_author`
- Любые 5 книг по языку: `books_by_language`
- Любые 5 книг по формату: `books_by_format`
- Любые 5 книг без серии: `books_without_series`
- Поиск по текстовому запросу: `books_search`
- Подробная карточка книги: `book_details`

Важно для агентов:

- После `initialize` всегда выполняйте `tools/list` и используйте его как источник истины для текущей сессии.
- Для browse-задач не используйте `books_search`.
- Не используйте `recommendation_candidates` для простого «дай N книг по жанру».
- `books_search.filters` в production отключен.

## Ограничения и поведение

- Неподдерживаемые аргументы/фильтры возвращаются как явная ошибка в `tools/call` (`isError: true`).
- `limit` должен быть `>=1`, `offset` — `>=0`; иначе возвращается явная ошибка валидации.
- Для `recommendation_candidates.filters` конфликт `min_price > max_price` возвращается как явная ошибка валидации.
- `books_by_author` приоритизирует точные/префиксные совпадения автора; для неоднозначных фамилий сначала показывает доминирующего автора.
- `books_search` работает в 2 фазы: strict-поиск + fallback по токенам длинного запроса (если strict пустой).
- В strict-фазе `books_search` добавлен typo-tolerant matching (trigram) для запросов с опечатками.
- `recommendation_candidates` работает в 2 фазы (strict -> fallback по токенам), если strict-результатов недостаточно.
- В ответе `recommendation_candidates` есть `diagnostic`; при `items: []` он объясняет причину пустой выдачи и дает подсказки.
- Для `recommendation_candidates` в `diagnostic` возвращаются `status`, `strict_candidates`, `fallback_candidates`, чтобы агент видел, какой этап подбора сработал.
- `books_search` и browse-инструменты возвращают `items`, `has_more`, `next_offset` (без полного `total`).
- `filters.popularity` и другие нестандартные фильтры не поддерживаются.

## Проверка подключения

### Health

```bash
curl https://knigochit.ru/health
```

### MCP initialize

```bash
curl -X POST https://knigochit.ru/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-12-11","capabilities":{},"clientInfo":{"name":"manual-client","version":"0.1.0"}}}'
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
