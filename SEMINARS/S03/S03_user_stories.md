# S03 — Перечень User Stories

## US-001 - Управление API-токенами

- **Роль-Цель-Ценность:** Как разработчик, я хочу выпускать и отзывать персональные API-токены, чтобы интегрироваться с сервисом.
- **Кратко:** Список токенов, маскирование, отзыв.
- **API/Endpoints:** `GET/POST/DELETE /api/tokens`
- **NFR hooks:** `Security-Secrets`, `Auditability`, `RateLimiting`, `API-Contract/Errors`

## US-002 - Уведомления (email/webhook)

- **Роль-Цель-Ценность:** Как пользователь, я хочу получать уведомления о важных событиях, чтобы не пропускать изменения.
- **Кратко:** Настройки каналов, отправка, ретраи вебхуков.
- **API/Endpoints:** `POST /api/notifications/send`, `POST /api/webhooks/{id}`
- **NFR hooks:** `Timeouts/Retry`, `Observability/Logging`, `RateLimiting`

## US-003 - Интеграция с внешним API

- **Роль-Цель-Ценность:** Как система, я хочу запрашивать данные у внешнего API, чтобы обогощать информацию.
- **Кратко:** Пулы соединений, таймауты, кэш, деградации.
- **API/Endpoints:** `GET /api/integrations/vendor-x/*`
- **NFR hooks:** `Timeouts/Retry`, `CircuitBreaker`, `Performance`, `Observability/Logging`
