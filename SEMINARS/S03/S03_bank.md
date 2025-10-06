# Адаптированные NFR требования

## NFR-001 - Хранение API-токенов
**User Story:** US-001  
**Category:** Security-Secrets  
**Requirement:** API-токены хранятся только в хэшированном виде; plaintext-токены отображаются только при создании  
**Rationale:** Предотвращение утечки токенов при компрометации БД  
**Acceptance:**  
```gherkin
Given база данных с таблицей api_tokens
When выполняется сканирование хранимых данных
Then все токены в колонке token_value хэшированы, plaintext-версии отсутствуют
```
**Evidence:** Ревизия БД, unit-тест валидации хэширования  
**Trace:** TBA  
**Owner:** Information Security Department  
**Status:** Draft  
**Priority:** P1  
**Severity:** S1  
**Tags:** secrets, api-tokens  

## NFR-002 - Лимиты запросов для API
**User Story:** US-001, US-003  
**Category:** RateLimiting  
**Requirement:** Эндпойнты `/api/tokens` и `/api/integrations/*` ограничены 1000 запросов/час на токен; превышение → 429 + Retry-After: 3600  
**Rationale:** Защита от злоупотребления и DoS-атак  
**Acceptance:**
```gherkin
Given валидный API-токен
When выполняется 1001 запрос к /api/tokens за 1 час
Then 1001-й запрос возвращает 429 с заголовком Retry-After: 3600
```
**Evidence:** Нагрузочный тест  
**Trace:** TBA  
**Owner:** DevOps  
**Status:** Draft  
**Priority:** P2  
**Severity:** S2  
**Tags:** rate-limiting, api-security, dos-protection  

## NFR-003 - Изоляция данных между пользователями
**User Story:** US-001  
**Category:** Security-AuthZ/RBAC  
**Requirement:** Пользователь может управлять только собственными токенами; попытка доступа к чужим токенам → 404  
**Rationale:** Принцип наименьших привилегий, изоляция данных для сокрытия данных между пользователями  
**Acceptance:**
```gherkin
Given пользователь A с токеном TokenA и пользователь B
When пользователь B запрашивает GET /api/tokens/TokenA
Then возвращается 404 без утечки информации о существовании токена
```
**Evidence:** Интеграционные тесты авторизации, security review  
**Trace:** TBA  
**Owner:** Backend Team  
**Status:** Draft  
**Priority:** P1  
**Severity:** S1  
**Tags:** authorization, rbac, api-tokens  

## NFR-004 - Надежность внешних интеграций
**User Story:** US-003  
**Category:** Timeouts/Retry/CircuitBreaker  
**Requirement:** Вызовы внешних API: timeout=5s, retry=2 с экспоненциальным backoff; circuit breaker открывается при ≥50% ошибок за 2 минуты  
**Rationale:** Устойчивость к недоступности внешних сервисов, снижение общей нагрузки на систему за счет circuit-breaker  
**Acceptance:**
```gherkin
Given внешний API vendor-x недоступен
When сервис вызывает GET /api/integrations/vendor-x/health
Then выполняется не более 2 ретраев с backoff, общее время ≤15s, после чего circuit breaker размыкается
```
**Evidence:** Интеграционные тесты  
**Trace:** TBA  
**Owner:** Backend Team  
**Status:** Draft  
**Priority:** P1  
**Severity:** S2  
**Tags:** timeouts, circuit-breaker, retry  

## NFR-005 - Трассируемость операций
**User Story:** US-001, US-002, US-003  
**Category:** Observability/Logging  
**Requirement:** Все запросы логируются; уведомления и внешние вызовы содержат исходный id запроса  
**Rationale:** Возможность трассировки полного цикла операций, расследование инцидентов  
**Acceptance:**
```gherkin
Given запрос с ID=123
When обработка завершена
Then все логи содержат ID=123, включая вызовы внешних сервисов
```
**Evidence:** Лог-файлы, дашборд трассировки, пример цепочки запросов  
**Trace:** TBA  
**Owner:** DevOps  
**Status:** Draft  
**Priority:** P2  
**Severity:** S3  
**Tags:** observability, tracing, logging  

## NFR-006 - Единый формат ошибок API
**User Story:** US-001, US-002, US-003  
**Category:** API-Contract/Errors  
**Requirement:** Все ошибки возвращаются в формате RFC 7807; 4xx/5xx не содержат stack trace; обязательные поля: type, title, status, instance  
**Rationale:** Стандартизация клиентской обработки ошибок, безопасность  
**Acceptance:**
```gherkin
Given невалидный запрос к любому API endpoint
When сервис возвращает ошибку
Then Content-Type=application/problem+json, тело соответствует RFC 7807, stack trace отсутствует
```
**Evidence:** Контракт-тесты API, примеры ошибок в документации  
**Trace:** TBA  
**Owner:** Backend Team  
**Status:** Draft  
**Priority:** P2  
**Severity:** S3  
**Tags:** api-design, error-handling, standards  

## NFR-007 - Аудит критических операций
**User Story:** US-001  
**Category:** Auditability  
**Requirement:** Создание/отзыв API-токенов логируются в аудит-журнал с метками: actor_id, action, token_id, timestamp, ip_address  
**Rationale:** Соответствие требованиям безопасности, расследование инцидентов  
**Acceptance:**
```gherkin
Given пользователь создает новый API-токен
When операция завершается успешно
Then в аудит-журнал записывается событие с полями actor_id, action=CREATE, token_id, timestamp, ip_address
```
**Evidence:** Аудит-логи, политика хранения журналов  
**Trace:** TBA  
**Owner:** Security Team  
**Status:** Draft  
**Priority:** P1  
**Severity:** S1  
**Tags:** audit, compliance, security, api-tokens  

## NFR-008 - Производительность внешних вызовов
**User Story:** US-003  
**Category:** Performance  
**Requirement:** P95 латентности вызовов `/api/integrations/vendor-x/*` ≤ 500ms при нагрузке 50 RPS в течение 10 минут  
**Rationale:** Обеспечение отзывчивости сервиса при зависимостях от внешних API  
**Acceptance:**
```gherkin
Given сервис развернут и здоров
When на /api/integrations/vendor-x/data подается 50 RPS в течение 10 минут
Then P95 латентности ≤ 500ms, доля ошибок ≤ 2%
```
**Evidence:** Нагрузочное тестирования, графики латентности в мониторинге  
**Trace:** TBA  
**Owner:** DevOps  
**Status:** Draft  
**Priority:** P2  
**Severity:** S2  
**Tags:** performance, latency, load-testing  

## NFR-009 - Валидация входных данных
**User Story:** US-001, US-002  
**Category:** Security-InputValidation  
**Requirement:** Размер тела запросов к API ≤ 1MB; email в уведомлениях валидируется по RFC 5322; webhook URL должен быть валидным HTTPS  
**Rationale:** Защита от переполнения и инъекций  
**Acceptance:**
```gherkin
Given запрос с телом размером 2MB к /api/notifications/send
When выполняется валидация
Then возвращается 413 с телом ошибки в формате RFC 7807
```
**Evidence:** Тесты валидации, статический анализ кода  
**Trace:** TBA  
**Owner:** Backend Team  
**Status:** Draft  
**Priority:** P2  
**Severity:** S3  
**Tags:** validation, security, input-sanitization  
