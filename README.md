# SecDev Lite - Отчетный репозиторий студента

> Здесь живут **итоговые артефакты**: `GRADING/TM.md`, `GRADING/DV.md`, `GRADING/DS.md`.  

---

## Команда

### Проект: [SSD-Project](https://github.com/fanglores/SSD-project)

### Студент 1
- **ФИО:** Цатурьян Константин Артурович
- **Группа:** МСПИН241
- **Контакт:** [@fanglores](https://t.me/fanglores)

### Студент 2
- **ФИО:** Дандамаев Гаджи
- **Группа:** МСПИН241
- **Контакт:** [@dandamev](https://t.me/dandamaev)

---

## Что мы сдаем (три артефакта)
- **TM - Требования + Модель угроз + ADR:** `GRADING/TM.md`
- **DV - Мини-проект «DevOps-конвейер»:** `GRADING/DV.md`
- **DS - Отчёт «DevSecOps-сканы и харднинг»:** `GRADING/DS.md`

**Минимальная структура репозитория:**

- `README.md`
- `SEMINARS/`
- `EVIDENCE/`
- `GRADING/` → `TM.md`, `DV.md`, `DS.md`

## Рубрики:
- TM: <https://github.com/hse-secdev-2025-fall/secdev-lite-docs/blob/main/rubrics/rubric_TM.md>
- DV: <https://github.com/hse-secdev-2025-fall/secdev-lite-docs/blob/main/rubrics/rubric_DV.md>
- DS: <https://github.com/hse-secdev-2025-fall/secdev-lite-docs/blob/main/rubrics/rubric_DS.md>

---

## Контрольные точки по содержанию
- **После S05** - заполнен черновик `TM.md` (архитектура/DFD+STRIDE, Top-5, NFR, 2×ADR, трассировка).
- **После S08** - заполнен черновик `DV.md` (one-liner локальной сборки, Dockerfile/запуск, базовый CI, артефакты, гигиена секретов).
- **После S12** - заполнен черновик `DS.md` (SBOM/SCA, SAST+Secrets, DAST или Policy, ≥2 меры харднинга с доказательствами, пороги+триаж+до/после).

---

## Правила артефактов
- **Все доказательства** - в `EVIDENCE/` (давайте точные ссылки из TM/DV/DS).
- **Не коммить секреты.** Пользуйся секретами репозитория/организации; не печатай значения в логах.
- **Честность.** Подлог/чужие скрины без явной пометки ⇒ **0 по критерию**.

---

## Самопроверка перед сдачей
- **TM:**
  - [x] есть DFD/границы;
  - [x] STRIDE закрыт;
  - [x] Top-5 с обоснованием;
  - [x] NFR (GWT) + ≥2 ADR;
  - [x] таблица трассировки и план проверок.
- **DV:**
  - [ ] one-liner локальной сборки;
  - [ ] образ собирается/запускается;
  - [ ] CI build+test зелёный;
  - [ ] артефакты в `EVIDENCE/`;
  - [ ] секреты - через masked.
- **DS:**
  - [ ] SBOM/SCA отчёты;
  - [ ] SAST+Secrets;
  - [ ] DAST **или** Policy;
  - [ ] ≥2 меры харднинга с доказательствами;
  - [ ] пороги/триаж;
  - [ ] хотя бы один «до/после».
