# ADR: Centralized PII Masking in Logging Layer

## Title

Centralized PII Masking in Logging Layer

## Status

Proposed  
(Утверждает: Security Team, дата предложения: 2025-10-11)

## Context

Risk: R-03 “Утечка PII/email через логи, ошибки и input” (L=3, I=5, Score=15)  
DFD: Edge/Node: Request, DTO, Logs, Errors  
NFR: NFR-006, NFR-009  
Assumptions:

- Используются централизованные логгеры и storage для сервисных логов.
- В сервисах хранятся и обрабатываются персональные данные: email, webhook для уведомлений.
- Требования: обеспечить предотвращение утечки PII через логи ошибок и сервисные события, соответствовать GDPR/152-ФЗ и внутреннему privacy policy.
- Логи хранятся минимум 30 дней для аудита/разбора инцидентов.

## Decision

Внедряется централизованная политика маскирования (masking) PII/email/webhook на уровне логгера:

- Param/Policy: включить masking-фильтр в централизованном логгере обработки ошибок, запросов, событий.
- Param/Policy: определить список полей PII, обязательных к маскированию (`email`, `webhook_url` и др.).
- Param/Policy: добавить alerter (Sentry/Prometheus) для выявления необработанного PII в логах.
- Doc: обновить policy логирования (logging policy).

## Alternatives

- Alt A: Retention/автоочистка сырых PII в логах ≤30 дней - снизит долговременный ущерб, но не защитит от мгновенной утечки (сразу после события инцидента).
- Alt B: Построить allowlist-логи: логировать только whitelisted-поля, без PII вообще - требует перепроектирования процессинга логов и DTO, затратно по времени.

## Consequences

- Немедленное снижение риска массовой утечки персональных данных через логи.
- Простой и обратимый rollout - можно включать/отключать фильтр через конфиг.
- Соответствие privacy-политике (GDPR/152-ФЗ).

* Возможность потери точности данных для отладки edge-cases (PII скрыта).
* Требуется отдельный alert на случаи, когда PII всё же появилось не маскированным.

## DoD / Acceptance

Given логгер с включённым masking-фильтром  
When сервис пишет ошибку или событие, содержащее PII/email/webhook  
Then в логах появляется только маскированная версия поля

Checks:

- test: автотесты - маскирование PII/email/webhook в логах, edge-cases с невалидными значениями
- log: при попытке записать открытый email/webhook - алерт, запись с маской
- scan: регулярные скрипты для поиска не-маскированных PII в логах
- audit: ручная проверка логов аудиторами безопасности

## Rollback / Fallback

- Masking можно выключить через параметр логгера (без деплоя).
- Возврат к прежней политике без потери совместимости.

## Trace

- DFD: Edge/Node: Request, DTO, Logs, Errors
- STRIDE: Info Disclosure
- Risk scoring: S04_risk_scoring.md (R-03, Top-5)
- NFR: NFR-006, NFR-009
- Issues: внедрение masking policies (issue-YY)

## Ownership & Dates

Owner: Security Team  
Reviewers: Backend Lead, Privacy Officer  
Date created: 2025-10-11  
Last updated: 2025-10-11

## Open Questions

- Какие дополнительные типы данных помимо email/webhook считать PII и включить в маскирование?
- Нужно ли логировать “оригинал” для супер-админов с audit доступа?
- Какой формат mask использовать (например: first char + domain для email)?

## Appendix (optional)

Пример masking-политики:

```
logging:
  masking:
    enabled: true
    pii_fields: [email, webhook_url]
    mask_format:
      email: "***@****.com"
      webhook_url: "https://***.domain.com"
```
