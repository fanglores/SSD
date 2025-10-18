# DV - Мини-проект «DevOps-конвейер»

> Этот файл проверяют по **rubric_DV.md** (5 критериев × {0/1/2} → 0-10).
> Все доказательства/скрины кладутся в **EVIDENCE/**.

---

## 0) Мета

- **Проект**: [SSD-Project](https://github.com/fanglores/SSD-project)
- **Версия (commit/date):** `v1` / `2025-12-10`  
- **Кратко (1-2 предложения):** микросервис на Flask; реализована безопасная аутентификация, контейнеризация и CI.

---

## 1) Воспроизводимость локальной сборки и тестов (DV1)

- **Одна команда для сборки/тестов:**

  ```bash
  python -m venv .venv && . .venv/bin/activate \
  && pip install -r requirements.txt \
  && python scripts/init_db.py \
  && pytest -q --junitxml=EVIDENCE/S06/test-report.xml
  ```

  См. [test-report.xml](../EVIDENCE/S06/test-report.xml)

- **Версии инструментов (фиксация):**

  ```bash
  python --version
  pip freeze > requirements.txt
  ```

  См. [requirements.txt](../EVIDENCE/S06/requirements.txt)

- **Описание шагов (кратко):**
  1. `python -m venv .venv && . .venv/bin/activate` - создание и активация виртуального окружения.  
  2. `pip install -r requirements.txt` - Установка зависимостей.  
  3. `python scripts/init_db.py` - Инициализация БД
  4. `pytest -q --junitxml=EVIDENCE/S06/test-report.xml` - Запуск тестов с генерацией отчёта

- **Доказательства:**  
  - Отчет по тестам: [test-report.xml](../EVIDENCE/S06/test-report.xml)
  - Фиксированная среда venv: [requirements.txt](../EVIDENCE/S06/requirements.txt)

---

## 2) Контейнеризация (DV2)

- **Dockerfile:** `./Dockerfile` - базовый образ `python:3.12-slim`, добавлены healthcheck и non-root.  
- **Сборка/запуск локально:**

  ```bash
  docker build -t secdev-seed:latest .
  docker compose up --build --wait
  ```

  Healthcheck возвращает HTTP 200, user - не root.

- **docker-compose:** `./docker-compose.yml` - сервис `web` + `db`.

- **Доказательства:**  
  - [Dockerfile](../EVIDENCE/S07/Dockerfile)  
  - [docker-compose.yml](../EVIDENCE/S07/docker-compose.yml)  
  - [image-size.txt](../EVIDENCE/S07/image-size.txt)  
  - [compose-up.log](../EVIDENCE/S07/compose-up.log)  
  - [ps.txt](../EVIDENCE/S07/ps.txt)  
  - [http_root_code.txt](../EVIDENCE/S07/http_root_code.txt)  
  - [http_root_last_error.txt](../EVIDENCE/S07/http_root_last_error.txt)  

---

## 3) CI: базовый pipeline и стабильный прогон (DV3)

- **Платформа CI:** GitHub Actions  
- **Файл конфига CI:** `.github/workflows/ci.yml`  
- **Стадии (минимум):** checkout → deps → **build** → **test** (артефакты выгружаются в отдельных экшенах)

- **Фрагмент конфигурации (ключевые шаги):**
  [ci.yml](https://github.com/fanglores/SSD-project/blob/main/.github/workflows/ci.yml)

- **Стабильность:** последние 3 запуска зелёные; кэш pip сработал.  
- **Ссылка/копия лога прогона:**  
  - [ci_run.png](../EVIDENCE/S08/ci_run.png)  
  - [GitHub Actions (Project repo)](https://github.com/fanglores/SSD-project/actions/)

---

## 4) Артефакты и логи конвейера (DV4)

| Артефакт/лог                    | Путь в `EVIDENCE/`                                    | Комментарий |
|---------------------------------|--------------------------------------------------------|--------------|
| Лог успешной сборки/тестов (CI) | [S08/ci_tests_run.txt](../EVIDENCE/S08/ci_tests_run.txt) |  |
| Локальный лог сборки            | [S06/local_tests_run.txt](../EVIDENCE/S06/local_tests_run.txt) |  |
| Логи запуска контейнера         | [S07/ci_docker_log.txt](../EVIDENCE/S07/ci_docker_log.txt) |  |
| Freeze/версии инструментов      | [S06/requirements.txt](../EVIDENCE/S06/requirements.txt)  | воспроизводимость окружения |
| Отчёт тестов                    | [S06/test-report.xml](../EVIDENCE/S06/test-report.xml) |  |

---

## 5) Секреты и переменные окружения (DV5 - гигиена)

- **Шаблон окружения:**  
  [.env.example](../EVIDENCE/S06/.env.example).  
  Секреты не коммитятся в git, есть проверка [pre-commit](../EVIDENCE/S06/.pre-commit-config.yaml).  

- **Хранение и передача в CI:**  
  - Секреты проверяются в [pre-commit](../EVIDENCE/S06/.pre-commit-config.yaml).  
  - На CI проводится проверка на токены в коде [secrets_ci.png](../EVIDENCE/S08/secrets_ci.png).  
  - В логе CI не печатаются значения (`::add-mask::`) [secrets_ci.png](../EVIDENCE/S08/secrets_ci.png).  

- **Доказательства:**  
  - [pre-commit](../EVIDENCE/S06/.pre-commit-config.yaml)
  - [secrets_ci.png](../EVIDENCE/S08/secrets_ci.png)

- **Памятка по ротации:** в проект добавлен [SECURITY.md](../EVIDENCE/S06/SECURITY.md).

---

## 6) Индекс артефактов DV

| Тип     | Файл в `EVIDENCE/`                                     | Дата/время         | Коммит/версия | Runner/OS    |
|---------|----------------------------------------------------------|--------------------|---------------|--------------|
| CI-log | [S08/ci_tests_run.txt](../EVIDENCE/S08/ci_tests_run.txt) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | gha-ubuntu |
| Local log | [S06/local_tests_run.txt](../EVIDENCE/S06/local_tests_run.txt) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | win11 24h2 |
| Freeze | [S06/requirements.txt](../EVIDENCE/S06/requirements.txt) | 2025-11-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | - |
| Dcoker log | [S07/ci_docker_log.txt](../EVIDENCE/S07/ci_docker_log.txt) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | ubuntu |
| Tests report | [S06/test-report.xml](../EVIDENCE/S06/test-report.xml) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | ubuntu |
| Secrets check | [S06/ci_precommit.txt](../EVIDENCE/S06/ci_precommit.txt) | 2025-18-10  | `dbb2c10d367246c8169f40ba8590820a69da4035` | local/ubuntu |
| Dockerfile | [Dockerfile](../EVIDENCE/S07/Dockerfile) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | ubuntu |
| docker-compose | [docker-compose.yml](../EVIDENCE/S07/docker-compose.yml) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | ubuntu |
| docker image size | [image-size.txt](../EVIDENCE/S07/image-size.txt) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | ubuntu |
| compose log | [compose-up.log](../EVIDENCE/S07/compose-up.log) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | ubuntu |
| docker ps | [ps.txt](../EVIDENCE/S07/ps.txt) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | ubuntu |
| healt check | [http_root_code.txt](../EVIDENCE/S07/http_root_code.txt) | 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | ubuntu |
| healt check | [http_root_last_error.txt](../EVIDENCE/S07/http_root_last_error.txt)| 2025-10-10 | `8f63fe435045a592a0a55715721cfc50c732f474` | ubuntu |

---

## 7) Связь с TM и DS (hook)

- **TM:** управление рисками DevOps - секреты, воспроизводимость, прозрачность артефактов.  
- **DS:** артефакты CI и контейнеризации служат базой для гейтов безопасности в `DS.md`.

---

## 8) Самооценка по рубрике DV (0/1/2)

- **DV1. Воспроизводимость локальной сборки и тестов:** [ ] 0 [ ] 1 [x] 2  
- **DV2. Контейнеризация (Docker/Compose):** [ ] 0 [ ] 1 [x] 2  
- **DV3. CI: базовый pipeline и стабильный прогон:** [ ] 0 [ ] 1 [x] 2  
- **DV4. Артефакты и логи конвейера:** [ ] 0 [ ] 1 [x] 2  
- **DV5. Секреты и конфигурация окружения (гигиена):** [ ] 0 [ ] 1 [x] 2 

**Итог DV (сумма):** **10/10**

А ещё у нас все сделано через PR и совместные коммиты в данном репозитории!  
А [программный проект](https://github.com/fanglores/SSD-project) максимально настроен, работа через PR, настроен CI с прекоммитом, всеми проверками.
