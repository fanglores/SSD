# S04 — STRIDE per element (матрица)

## Легенда STRIDE

- S - Spoofing: подмена идентичности/токена.
- T - Tampering: изменение данных/запросов/конфигурации.
- R - Repudiation: отрицание действий (нет аудита/трассировки).
- I - Information disclosure: утечка конфиденциальных данных (PII/секреты).
- D - Denial of service: отказ в обслуживании (ресурсное истощение/«залипание»).
- E - Elevation of privilege: повышение привилегий/обход RBAC/тенант-изоляции.

## Матрица

| Element                      | Data/Boundary                   | Threat (S/T/R/I/D/E) | Description                                                 | NFR link (ID)    | Mitigation idea (ADR later)                              |
| ---------------------------- | ------------------------------- | -------------------- | ----------------------------------------------------------- | ---------------- | -------------------------------------------------------- |
| Edge: Internet → API         | JWT, API-token                  | S (Spoofing)         | Подмена или reuse украденного токена (JWT, API-token)       | NFR-003          | Проверка подлинности токена, короткий TTL, rate limiting |
| Edge: Internet → API         | Request, PII                    | I (Info Disclosure)  | Утечка email/webhook через запросы или логи                 | NFR-009, NFR-006 | Валидация input, маскирование ошибок, logging ENVs       |
| Edge: Internet → API         | Requests (Bulk)                 | D (Denial)           | Flood запросами → истощение ресурса/DoS                     | NFR-002          | Троттлинг и rate limiting на эндпоинты                   |
| Edge: API → Service          | DTO, PII                        | I (Info Disclosure)  | Утечка PII/невалидные DTO при передаче между API и сервисом | NFR-009, NFR-006 | Валидация DTO, маскирование полей, secure transport      |
| Node: API/Controller         | API-Responses                   | T (Tampering)        | Модификация/инъекции тела запроса или params                | NFR-009          | Строгая валидация payload, input sanitizer               |
| Node: API/Controller         | Logs, Errors                    | I (Info Disclosure)  | Утечка stacktrace, внутренних данных или токена в error/log | NFR-006          | RFC7807 ошибки, без stacktrace, логирование без секретов |
| Node: Service                | AuthZ token processing          | E (Elevation)        | Попытка обхода RBAC, отзыв чужих токенов                    | NFR-003          | Явная проверка subject, RBAC enforcement                 |
| Node: Service                | Audit log                       | R (Repudiation)      | Операции не попадают в аудит, нельзя отследить действия     | NFR-007          | Явный аудит журнал, atomарность логирования              |
| Node: Service                | Outbound webhooks               | D (Denial)           | Блокировка на ретраях, зависание HTTP-потока без таймаута   | NFR-004          | Таймауты, ретраи, circuit breaker                        |
| Edge: Service → DB           | API-tokens table                | T (Tampering)        | Изменение или кража токенов не через бизнес-логику          | NFR-001          | Хэширование, только через сервис, ревизия                |
| Edge: Service → DB           | Audit table                     | R (Repudiation)      | Удаление или подмена audit событий                          | NFR-007          | Только append log, права только на создание              |
| Edge: Service → External API | Outbound request (MITM)         | S (Spoofing)         | Подмена внешнего API / MITM                                 | NFR-010          | TLS, проверка сертификата, mTLS, pinned certs            |
| Edge: Service → External API | API responses                   | I (Info Disclosure)  | Утечка конфиденциальных данных через third-party интеграцию | NFR-005          | Фильтрация полей, только необходимые данные              |
| Edge: Service → External API | Outbound request (timeout/slow) | D (Denial)           | Залипание при external API outage/slow downstream           | NFR-004          | Таймауты, circuit breaker                                |
| Node: DB                     | API-tokens table                | I (Info Disclosure)  | Хранение токенов в plaintext                                | NFR-001          | Только хэшированное хранение                             |
| Node: DB                     | Notifications                   | R (Repudiation)      | Нет аудита операций с нотификациями (утеряны события)       | NFR-007          | Аудит на каждое событие                                  |
| Node: DB                     | User tokens + meta              | E (Elevation)        | Возможен доступ из другой «тенант»-зоны                     | NFR-003          | Изоляция данных, проверки на subject                     |
