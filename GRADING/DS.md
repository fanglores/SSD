# DS - Отчёт «DevSecOps-сканы и харднинг»

> Этот файл - **индивидуальный**. Его проверяют по **rubric_DS.md** (5 критериев × {0/1/2} → 0-10).
> Подсказки помечены `TODO:` - удалите после заполнения.
> Все доказательства/скрины кладите в **EVIDENCE/** и ссылайтесь на конкретные файлы/якоря.

---

## 0) Мета

- **Проект**: [SSD-Project](https://github.com/Dandamaev/ssd-project-s09-s12)
- **Версия (commit/date):** `v1` / `2025-19-10`
- **Кратко (1-2 предложения):** FastAPI-приложение с демонстрацией DevSecOps-практик, предназначенное для изучения безопасности и управления зависимостями.

---

## 1) SBOM и уязвимости зависимостей (DS1)

- **Инструмент/формат:** CycloneDX 1.6, Grype v0.101.1 для SCA

- **Как запускал:**

  ```bash
  # Generate SBOM with Syft
  scan dir:. -o cyclonedx-json > EVIDENCE/S09/sbom.json

  # Run SCA with Grype
  grype sbom:EVIDENCE/S09/sbom.json --fail-on high -o json > EVIDENCE/S09/sca_report.json
  ```

- **Отчёты:**

  - SBOM: `EVIDENCE/S09/sbom.json`
  - SCA Report: `EVIDENCE/S09/sca_report.json`
  - CI/CD: [GitHub Actions Job](https://github.com/Dandamaev/ssd-project-s09-s12/actions/runs/18632150076/job/53118368830)

- **Выводы (кратко):**

  - Найдены уязвимости в Jinja2 3.1.4:
    - 2 Medium (GHSA-q2x7-8rv6-6q7h, GHSA-cpwx-vrp4-4pq7) — sandbox breakout через format method
    - 1 High (CVE-2024-56201) — RCE через malicious filenames
  - Ключевые зависимости:
    - fastapi=0.115.0
    - jinja2=3.1.4 (требует обновления до 3.1.6)
    - uvicorn=0.30.1
    - httpx=0.27.2
    - pydantic=2.9.2
    - pytest=8.3.2

- **Действия:**

  - Требуется обновление jinja2 до версии 3.1.6 для устранения всех найденных уязвимостей
  - Остальные зависимости актуальны и не содержат известных уязвимостей
  - Добавлен автоматический CI check через GitHub Actions

- **Гейт по зависимостям:**
  - Critical=0
  - High≤1 (временно допускается одна High в jinja2 до планового обновления)

---

## 2) SAST и Secrets (DS2)

### 2.1 SAST

- **Инструмент/профиль:** Semgrep OSS v1.138.0 с профилем p/ci

- **Как запускал:**

  ```bash
  semgrep --config p/ci --severity=high --error --sarif --output EVIDENCE/S10/semgrep.sarif
  ```

- **Отчёт:** `EVIDENCE/S10/semgrep.sarif`
- **Выводы:**
  - Сканирование успешно выполнено
  - Критических уязвимостей и проблем с качеством кода не обнаружено
  - Профиль p/ci покрывает основные паттерны безопасности Python

### 2.2 Secrets scanning

- **Инструмент:** Gitleaks (последняя версия)
- **Как запускал:**

  ```bash
  gitleaks detect --no-git --report-format json --report-path EVIDENCE/S10/gitleaks.json
  ```

- **Отчёт:** `EVIDENCE/S10/gitleaks.json`
- **Выводы:**
  - Сканирование успешно выполнено
  - Секретов и чувствительных данных в репозитории не обнаружено
  - CI/CD: [GitHub Actions Job](https://github.com/Dandamaev/ssd-project-s09-s12/actions/runs/18635149386)

---

## 3) DAST **или** Policy (Container/IaC) (DS3)

> Для «1» достаточно одного из классов; на «2» - желательно оба **или** один глубже (настроенный профиль/таргет).

### Вариант A - DAST (лайт)

- **Инструмент/таргет:** OWASP ZAP (ZAP Baseline Scan) — ZAP v2.16.1
- **Как запускал (локально через Docker):**

```bash
# запущено локально через docker (user)
docker run --rm -v %cd%:/zap/wrk \
  zaproxy/zap-stable \
  zap-baseline.py -t http://host.docker.internal:8080 \
  -r EVIDENCE/S11/zap_baseline.html \
  -J EVIDENCE/S11/zap_baseline.json -d
```

- **Отчёты / Артефакты:**

  - HTML: `EVIDENCE/S11/zap_baseline.html`
  - JSON: `EVIDENCE/S11/zap_baseline.json`
  - Scan config used (baseline job): `EVIDENCE/S11/zap.yaml`

- **Краткая сводка (из zap_baseline.json / html):**

  - High: 0
  - Medium: 2
  - Low: 3
  - Informational: 3

- **Основные Medium‑alerts (рекомендуемые исправления):**

  - Content Security Policy (CSP) Header Not Set — pluginId 10038 — instances: 4  
    → Action: добавить / настроить Content-Security-Policy (backend / web server). Evidence: `EVIDENCE/S11/zap_baseline.json#10038` / `EVIDENCE/S11/zap_baseline.html#10038`
  - Missing Anti-clickjacking Header (X-Frame-Options / frame-ancestors) — pluginId 10020 — instances: 3  
    → Action: добавить X-Frame-Options: DENY или CSP frame-ancestors (backend). Evidence: `EVIDENCE/S11/zap_baseline.json#10020` / `EVIDENCE/S11/zap_baseline.html#10020`

- **Выводы (кратко):**

  - Сканы выполнены успешно локально; основные находки — отсутствие/недостаток security headers. Это типичные конфигурационные замечания для demo‑приложения.
  - Нет критичных (High) уязвимостей по результатам базового скана.
  - Рекомендуется исправить security headers и повторно запустить ZAP для валидации.

- **Действия / Owner / Проверка:**

  - Owner: Backend Team
  - Действие: внедрить CSP, X-Frame-Options / frame-ancestors, X-Content-Type-Options; затем повторный ZAP baseline.
  - Проверка: re-run ZAP baseline; compare `EVIDENCE/S11/zap_baseline.json` до/после.

- **CI/CD Actions:** [GitHub Actions Job](https://github.com/Dandamaev/ssd-project-s09-s12/actions/runs/18658048855)

### Вариант B - Policy / Container / IaC

- **Инструмент(ы):**

  - Trivy v0.48.0 (container/config scanning для образов и IaC)

- **Как запускал:**

  ```powershell
  # Config scan
  docker run --rm -v "${PWD}:/workdir" -w /workdir aquasec/trivy:latest config `
    --format table `
    --severity HIGH,CRITICAL `
    --output EVIDENCE/S11/trivy-config.txt .

  # Image scan
  docker run --rm -v "${PWD}:/workdir" -v /var/run/docker.sock:/var/run/docker.sock -w /workdir `
    aquasec/trivy:latest image `
    --format table `
    --severity HIGH,CRITICAL `
    --output EVIDENCE/S11/trivy-image.txt myapp:latest
  ```

- **Отчёты:**

  - Config scan: `EVIDENCE/S12/trivy-config.txt`
  - Image scan: `EVIDENCE/S12/trivy-image.txt`

- **Выводы:**

  1. Container/Image findings (HIGH):

     - CVE-2024-47874 в starlette 0.37.2 (DoS via multipart/form-data)
     - Dockerfile: нет USER (non-root) директивы
     - Отсутствует HEALTHCHECK

  2. K8s configuration findings (HIGH):

     - readOnlyRootFilesystem не установлен
     - Используется root пользователь (securityContext.runAsUser: 0)
     - Отсутствуют resource limits
     - Используется latest tag
     - Нет liveness/readiness probes
     - NetworkPolicy не определена

  3. Actions:
     - Обновить starlette до 0.40.0
     - Добавить non-root USER в Dockerfile
     - Настроить SecurityContext в k8s (non-root, readonly fs)
     - Добавить resource limits и probes
     - Использовать фиксированные теги образов
     - Определить NetworkPolicy

- **Owner/Status:**
  - Container/Image fixes: Backend Team (open)
  - K8s configuration: DevOps Team (open)
  - Priority: HIGH (критичные находки по безопасности контейнеров)

---

## 4) Харднинг (доказуемый) (DS4)

Отметьте **реально применённые** меры, приложите доказательства из `EVIDENCE/`.

- [ ] **Контейнер non-root / drop capabilities** → Evidence: `EVIDENCE/policy-YYYY-MM-DD.txt#no-root`
- [ ] **Rate-limit / timeouts / retry budget** → Evidence: `EVIDENCE/load-after.png`
- [ ] **Input validation** (типы/длины/allowlist) → Evidence: `EVIDENCE/sast-YYYY-MM-DD.*#input`
- [ ] **Secrets handling** (нет секретов в git; хранилище секретов) → Evidence: `EVIDENCE/secrets-YYYY-MM-DD.*`
- [ ] **HTTP security headers / CSP / HTTPS-only** → Evidence: `EVIDENCE/security-headers.txt`
- [ ] **AuthZ / RLS / tenant isolation** → Evidence: `EVIDENCE/rls-policy.txt`
- [ ] **Container/IaC best-practice** (минимальная база, readonly fs, …) → Evidence: `EVIDENCE/trivy-YYYY-MM-DD.txt#cfg`

> Для «1» достаточно ≥2 уместных мер с доказательствами; для «2» - ≥3 и хотя бы по одной показать эффект «до/после».

---

## 5) Quality-gates и проверка порогов (DS5)

- **Пороговые правила (словами):**  
  Примеры: «SCA: Critical=0; High≤1», «SAST: Critical=0», «Secrets: 0 истинных находок», «Policy: Violations=0».
- **Как проверяются:**

  - Ручной просмотр (какие файлы/строки) **или**
  - Автоматически: (скрипт/job, условие fail при нарушении)

    ```bash
    SCA: grype ... --fail-on high
    SAST: semgrep --config p/ci --severity=high --error
    Secrets: gitleaks detect --exit-code 1
    Policy/IaC: trivy (image|config) --severity HIGH,CRITICAL --exit-code 1
    DAST: zap-baseline.py -m 3 (фейл при High)
    ```

- **Ссылки на конфиг/скрипт (если есть):**

  ```bash
  GitHub Actions: .github/workflows/security.yml (jobs: sca, sast, secrets, policy, dast)
  или GitLab CI: .gitlab-ci.yml (stages: security; jobs: sca/sast/secrets/policy/dast)
  ```

---

## 6) Триаж-лог (fixed / suppressed / open)

| ID/Anchor     | Класс | Severity | Статус     | Действие | Evidence                            | Ссылка на фикс/исключение       | Комментарий / owner / expiry       |
| ------------- | ----- | -------- | ---------- | -------- | ----------------------------------- | ------------------------------- | ---------------------------------- |
| CVE-2024-XXXX | SCA   | High     | fixed      | bump     | `EVIDENCE/deps-YYYY-MM-DD.json#CVE` | `commit abc123`                 | -                                  |
| ZAP-123       | DAST  | Medium   | suppressed | ignore   | `EVIDENCE/dast-YYYY-MM-DD.pdf#123`  | `EVIDENCE/suppressions.yml#zap` | FP; owner: ФИО; expiry: 2025-12-31 |
| SAST-77       | SAST  | High     | open       | backlog  | `EVIDENCE/sast-YYYY-MM-DD.*#77`     | issue-link                      | план фикса в релизе N              |

> Для «2» по DS5 обязательно указывать **owner/expiry/обоснование** для подавлений.

---

## 7) Эффект «до/после» (метрики) (DS4/DS5)

| Контроль/Мера | Метрика                 |   До |  После | Evidence (до), (после)                             |
| ------------- | ----------------------- | ---: | -----: | -------------------------------------------------- |
| Зависимости   | #Critical / #High (SCA) | TODO | 0 / ≤1 | `EVIDENCE/deps-before.json`, `deps-after.json`     |
| SAST          | #Critical / #High       | TODO | 0 / ≤1 | `EVIDENCE/sast-before.*`, `sast-after.*`           |
| Secrets       | Истинные находки        | TODO |      0 | `EVIDENCE/secrets-*.json`                          |
| Policy/IaC    | Violations              | TODO |      0 | `EVIDENCE/checkov-before.txt`, `checkov-after.txt` |

---

## 8) Связь с TM и DV (сквозная нитка)

- **Закрываемые угрозы из TM:** TODO: T-001, T-005, … (ссылки на таблицу трассировки TM)
- **Связь с DV:** TODO: какие сканы/проверки встроены или будут встраиваться в pipeline

---

## 9) Out-of-Scope

- TODO: что сознательно не сканировалось сейчас и почему (1-3 пункта)

---

## 10) Самооценка по рубрике DS (0/1/2)

- **DS1. SBOM и SCA:** [ ] 0 [ ] 1 [ ] 2
- **DS2. SAST + Secrets:** [ ] 0 [ ] 1 [ ] 2
- **DS3. DAST или Policy (Container/IaC):** [ ] 0 [ ] 1 [ ] 2
- **DS4. Харднинг (доказуемый):** [ ] 0 [ ] 1 [ ] 2
- **DS5. Quality-gates, триаж и «до/после»:** [ ] 0 [ ] 1 [ ] 2

**Итог DS (сумма):** \_\_/10
