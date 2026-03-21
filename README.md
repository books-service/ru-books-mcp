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

- `books_search`: `query`, `limit`, `offset`, `filters.author`, `filters.genre`, `filters.language`, `filters.min_price`, `filters.max_price`
- `recommendation_candidates`: `user_query`, `limit`, `filters.author`, `filters.genre`, `filters.language`, `filters.min_price`, `filters.max_price`
- `book_details`: `book_id`

Сейчас не поддерживается:

- `filters.formats`, `filters.without_series`, `filters.popularity` (и другие нестандартные поля)

## Инструменты

- `books_search` — поиск книг по запросу и опциональным фильтрам.
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

## Примечания

- Сервер возвращает данные поиска и наборы кандидатов.
- Финальная рекомендация для пользователя формируется агентом/клиентом.
