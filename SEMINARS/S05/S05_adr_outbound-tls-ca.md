# ADR: Outbound TLS + CA for External API Calls

## Title

Outbound TLS + CA for External API Calls

## Status

Proposed  
(Утверждает: Security Team, дата предложения: 2025-10-11)

## Context

Risk: R-07 “MITM/подмена внешнего API (Spoofing)” (L=3, I=5, Score=15)  
DFD: Edge: Service → External API (Outbound request, MITM)  
NFR: NFR-010  
Assumptions:

- Интеграция с внешними API происходит через HTTPS.
- Требуется гарантия подлинности партнерских API, снижение риска MITM.
- Есть корпоративный CA и штатный TLS stack, mTLS невозможен для SaaS интеграций.
- Требуется оценка соответствия стандартам безопасности при взаимодействии с третьими сторонами.

## Decision

Внедрить обязательную проверку TLS и CA для всех outbound external API вызовов:

- Param/Policy: Включение проверки CA (root/trusted) для каждого внешнего endpoint.
- Param/Policy: Ошибка connect при невалидном TLS/CA.
- Doc: Обновить техническую документацию по требованиям TLS для интеграций.

## Alternatives

- Alt A: Внедрить mTLS для всех trusted endpoints — избыточно для SaaS, требует согласования с партнёрами.
- Alt B: Certificate Pinning (список известных fingerprint/key) — сложнее администрировать, требует обновления при ротации сертификатов.

## Consequences

- Значительно снижается риск атак MITM/подмены внешнего API при правильно настроенном CA.
- Лёгкая реализация через стандартные библиотеки и флаг/конфиг.
- Повышается доверие к outbound интеграциям, соответствие корпоративной политике.

* Возможность блокировки операций при ошибке CA (например, экспирация сертификата).
* Требуется мониторинг CA expiration и актуальность trust list.

## DoD / Acceptance

Given outbound запрос к внешнему API  
When производится TLS handshake и CA validation  
Then соединение устанавливается только при валидном сертификате

Чек-лист:

- test: автотесты — выполняется TLS/CA проверка на outbound вызовах
- log: все неудачные handshakes с ошибками CA фиксируются в log/error
- scan/policy: анализ trust list, все endpoints внесены по актуальному CA
- metric: нет успешных запросов при ошибках handshake/CA

## Rollback / Fallback

- Отключить CA enforcement через конфиг, временно разрешить non-verified connect по траблшутингу.

## Trace

- DFD: Edge: Service → External API
- STRIDE: Spoofing
- Risk scoring: S04_risk_scoring.md (R-07, Top-5)
- NFR: NFR-010
- Issues: внедрение TLS CA policy (issue-ZZ)

## Ownership & Dates

Owner: Security Team  
Reviewers: Backend Lead, DevOps, Vendor Integration Lead  
Date created: 2025-10-11  
Last updated: 2025-10-11

## Open Questions

- Как обновлять trust list/CA при ротации сертификатов у партнерских API?
- Нужно ли логировать handshake details для всех outbound операций или только ошибки?
- Возможна ли интеграция с мониторингом expiration CA у большого числа endpoints?

## Appendix (optional)

Пример конфига:

```
external_api:
  tls:
    ca_verification: true
    trust_list:
      - "VendorX_CA.pem"
      - "PartnerY_CA.pem"
```
