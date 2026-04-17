# Multi-Agent Orchestration на Claude Code

Production-система мультиагентной оркестрации внутри Claude Code CLI. Превращает описание фичи на естественном языке в готовый, провалидированный код, соответствующий конвенциям проекта (Next.js, React, TypeScript, 5-слойная архитектура).

Вместо одного LLM-вызова с гигантским промптом — пайплайн из 8+ изолированных суб-агентов, каждый на подходящей модели (Opus / Sonnet / Haiku) и в подходящем когнитивном режиме. Коммуникация через файлы в `.claude/sessions/` — без LangChain, очередей, БД и внешних сервисов.

## Зачем

Один LLM-вызов на всю задачу = размытие фокуса, confirmation bias на собственном коде, переплата за модель и взрыв контекста. Решение — когнитивное разделение труда:

- **Одна роль на агента** (аналитик, архитектор, разработчик, валидатор, консультант)
- **Минимальный контекст** — только то, что нужно для задачи
- **Подходящая модель** — Opus для решений, Sonnet для исполнения, Haiku для рутины
- **Изоляция** — агенты общаются только через файлы, нет contamination контекста

## Структура

```
.claude/
├── commands/
│   ├── dev.md                  # Оркестратор (thin dispatcher)
│   └── dev-steps/              # Step-файлы (инструкции на шаг)
├── agents/                     # Identity-файлы (analyst, architect, developer, validator)
├── skills/                     # Переиспользуемые шаблоны кода
├── rules/                      # Конвенции проекта (TS, SCSS, GraphQL, Redux)
└── sessions/
    └── dev-YYYYMMDD-HHMMSS/    # Артефакты конкретного запуска
```

Четыре уровня абстракции:

| Тип | Назначение |
|-----|-----------|
| **Rules** | КАК писать код — ограничения и конвенции, читают все агенты |
| **Skills** | ЧТО написать для конкретного паттерна — шаблоны, читает только разработчик |
| **Agents** | Identity роли (кто ты, как ведёшь себя) |
| **Steps** | Инструкции для конкретной фазы пайплайна |

## Агенты

| Роль | Модель | Mode | Задача |
|------|--------|------|--------|
| Analyst | Sonnet | MEDIUM | Исследует код, формулирует вопросы, собирает context-pack |
| Architect | Opus | HIGH | Проектирует план, ревьюит, консультирует, диагностирует тупики |
| Developer | Sonnet | LOW | Пишет код строго по плану, исправляет замечания |
| Validator | Haiku | LOW | Запускает линтеры, парсит вывод, генерит отчёт |
| Consultant | Opus | HIGH | Отвечает на блокирующие вопросы pre/mid-flight |

## Пайплайн

```
Step 1  CLARIFY    (Sonnet)  → questions.md, context-pack.md
Step 2  PLAN       (Opus)    → plan.md
Step 2.5 PREFLIGHT (Sonnet)  → проверка плана на выполнимость
        └ blocked → CONSULT (Opus) → PATCH или re-plan
Step 3  IMPLEMENT  (Sonnet)  → код, context-pack.delta.md
        └ blocker → mid-flight CONSULT (Opus, max 1x)
Step 4  REVIEW     (3x Opus, parallel) → arch / TS / SCSS
Step 5  FIX        (Sonnet)  → если есть must-fix
Step 6  VALIDATE   (Haiku)   → typescript / lint:ts / lint:scss vs baseline
Step 7  FIX LOOP   (Sonnet, max 3 iterations)
Step 7B ESCALATE   (Opus, max 1x) → root cause при тупике
Step 8  PRESENT    (Haiku)   → summary.md
```

Маршрутизация — по содержимому файлов: оркестратор grep-ает `STATUS:` / `PLAN_STATUS:`, считает must-fix.

## Ключевые механики

**Context Pack + Delta** — Step 1 собирает snapshot всех затронутых файлов (`context-pack.md`), Step 3/5/7 ведут append-only delta. Все агенты читают pack+delta вместо повторного Read с диска. Сокращает tool-calls на 60–80%.

**Self-contained Plan** — `plan.md` содержит точные пути, Import Map, Naming Map, SCSS structure. Разработчик не принимает архитектурных решений.

**Execution Mode** — инструкция в промпте (LOW / MEDIUM / HIGH), калибрующая поведение модели независимо от её мощности. Sonnet в LOW ≠ Sonnet в MEDIUM.

**Baseline Diff** — валидатор сравнивает ошибки с состоянием `origin/main`. В отчёт попадают только regressions, а не существующий техдолг.

**Circuit breakers** — hard caps на каждый цикл: 3 fix iterations, 1 escalation, 1 mid-flight consult, 1 re-plan. Никаких бесконечных loop.

**PLAN_STATUS** — каждый ответ консультанта содержит сигнал: `OK` / `PLAN_PATCH` / `PLAN_BROKEN`. Три уровня реакции — продолжить, точечно поправить, перепланировать.

## Файловый протокол

Все артефакты — markdown в `.claude/sessions/dev-YYYYMMDD-HHMMSS/`. Без JSON, без shared memory, без API.

- **Debuggable** — `cat plan.md` показывает что задумал архитектор, `cat review-ts.md` — что нашёл ревьюер
- **Reproducible** — session-директория = полный snapshot, можно перезапустить шаг с теми же inputs
- **Cache-friendly** — Reading Order стабилен, общий префикс попадает в Claude Code prompt cache (TTL ~5 мин)

## Без внешней инфраструктуры

**Используется:** Claude Code CLI, файловая система, git, yarn, markdown.

**Не используется:** LangChain / LangGraph, CrewAI / AutoGen, vector DB, Redis, custom backend, Docker.

Преимущества:
- Zero setup — `git clone`, и пайплайн работает
- Version controlled — step/agent/rules — обычные markdown в репозитории
- Modifiable — добавить шаг = создать markdown + обновить роутер
- Portable — заменить `rules/` и `skills/` → пайплайн работает на другой кодовой базе

## Стоимость

| Роль | Модель | Обоснование |
|------|--------|-------------|
| Analyst | Sonnet | Широкий поиск, deep reasoning не нужен |
| Architect | Opus | Решения с последствиями |
| Developer | Sonnet | Механическое исполнение по плану |
| Validator | Haiku | Запуск команд + парсинг |
| Consultant | Opus | Не слабее планировщика |

Полностью на Opus — ~10x дороже. Полностью на Sonnet — деградация планов и ревью, больше итераций fix loop, парадоксально дороже.

## Использование

```
/dev <описание задачи>
```

Два user checkpoint-а в потоке:
1. Подтверждение понимания задачи (ответы на вопросы аналитика)
2. Подтверждение плана (или request на изменения)

Дальше — автоматический пайплайн до summary.

## Когда применимо

Команды с жёсткими конвенциями, большой кодовой базой и повторяющимися паттернами (компоненты, хуки, страницы). Чем больше конвенций в `rules/` и шаблонов в `skills/` — тем выше ROI.
