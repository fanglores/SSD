# ADR: JWT TTL + Refresh

## Title

JWT TTL + Refresh

## Status

Proposed  
(Утверждает: Security Team, дата предложения: 2025-10-11)

## Context

Risk: R-01 “Перехват/reuse JWT/токенов” (L=4, I=4, Score=16)  
DFD: Edge Internet→API (JWT, API-token)  
NFR: NFR-003  
Assumptions:

- Используется стандарт JWT (RFC 7519) для авторизации REST API через gateway/service.
- В инфраструктуре отсутствует MTLS для клиентов.
- Требования: минимизировать окно компрометации, легко поддерживать и катить изменения.
- Применимо для всех публичных утилитных и B2B-эндпоинтов.

## Decision

Выбран вариант сокращения времени жизни (TTL) access-токена JWT до 15 минут и внедрения обязательного refresh-token механизма.

- Param/Policy: `access_token_TTL=15m` (scope: gateway/service)
- Param/Policy: поддержка отдельного refresh endpoint (scope: сервис авторизации)
- Param/Policy: обязательная проверка срока действия токена на каждом входящем вызове (gateway)
- Notes: Покрывает все публичные API, не требует изменения схемы хранения токенов.

## Alternatives

- Alt A: Ротация signing ключей/JWKS - отказано как более трудоёмкое по DevOps части, требует отдельной инфраструктуры Key Management.
- Alt B: mTLS для всех клиентов - не подходит для внешних интегрируемых пользователей, слишком высок порог сложности для начального этапа.

## Consequences

- Снижение риска компрометации: даже при утечке окно использования токена - ≤15 минут.
- Быстрое внедрение (config-driven), минимальные дополнительные зависимости.

* Требуется поддержка refresh-потока (больше логики в клиенте).
* Возможны частые re-login при ошибках реализации refresh.

## DoD / Acceptance

Given валидный access/refresh-token  
When access-token истёк, а refresh-token в силе  
Then клиент обязан запросить новый access-token через refresh endpoint

Checks:

- test: автотесты на истечение срока жизни access-токена и корректную работу refresh потока
- log: появление записей о refresh-операциях, отсутствие длинносрочных access-token’ов
- scan/policy: политика - все живые токены ≤15m
- metric/SLO: не более 5% ошибок из-за некорректных refresh-операций, среднее время refresh ≤5s

## Rollback / Fallback

- Можно безопасно увеличить TTL обратно через конфиг без пересборки сервисов.
- Откат refresh endpoint и возврат к текущей схеме доступен после удаления политики TTL.

## Trace

- DFD: Edge Internet→API (JWT, API-token)
- STRIDE: Spoofing row по JWT
- Risk scoring: S04_risk_scoring.md (R-01, Top-5)
- NFR: NFR-003
- Issues: внедрение новой политики TTL/refresh (issue-XX)

## Ownership & Dates

Owner: Security Team  
Reviewers: Backend Lead, DevOps  
Date created: 2025-10-11  
Last updated: 2025-10-11

## Open Questions

- Нужно ли ограничивать TTL refresh-token?
- Требуется ли логировать все refresh-операции для аудита?
- Как кэшировать токен в API gateway (при коротких TTL) без увеличения latency?

## Appendix (optional)

Пример конфига:

```
auth:
    access_token_ttl: 15m
    refresh_token_ttl: 7d
    refresh_endpoint: "/api/auth/refresh"
```
