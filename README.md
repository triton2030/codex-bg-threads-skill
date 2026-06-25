# Codex Skill: Фоновые Треды

Неофициальный Codex skill для фоновых тредов: главный тред остаётся
оркестратором, воркеры работают отдельно, присылают `WORKER_DONE`, а при
нужде их результаты сводит merge-owner.

Не для обычных same-thread subagents.

## Когда Нужен

- несколько независимых пишущих агентов;
- долгие задачи с heartbeat/watchdog;
- воркер должен сам вызвать subagents;
- worktree-воркеры потом сливаются в один результат.

## Установка

```sh
mkdir -p ~/.codex/skills
git clone https://github.com/triton2030/codex-bg-threads-skill.git ~/.codex/skills/1codex-bg-threads
```

Запрос:

```text
Use $1codex-bg-threads for explicit background workers with heartbeat and WORKER_DONE stop signals.
```

## Как Это Работает

```mermaid
flowchart TD
  A["Пользовательская задача"] --> B["Оркестратор"]
  B --> C["Фоновый worker thread"]
  C --> D{"Нужны subagents?"}
  D -- "нет" --> E["Worker делает свою часть"]
  D -- "да" --> F["Worker вызывает subagents"]
  F --> G["Worker сводит их ответы"]
  E --> H["WORKER_DONE"]
  G --> H
  H --> I["Оркестратор проверяет evidence"]
  I --> J{"Нужен merge?"}
  J -- "нет" --> K["Принять результат"]
  J -- "да" --> L["Merge-owner собирает финальный diff"]
```

## Выбор Режима

```mermaid
flowchart TD
  A["Задача"] --> B{"Нужен отдельный thread?"}
  B -- "нет" --> C["Обычный Codex flow"]
  B -- "да" --> D{"Есть пересечения по файлам?"}
  D -- "да" --> E["Sequential writer"]
  D -- "нет" --> F{"Нужна изоляция?"}
  F -- "нет" --> G["Light worker"]
  F -- "да" --> H["Worktree worker"]
  G --> I{"Долго?"}
  H --> I
  I -- "да" --> J["Heartbeat + watchdog"]
  I -- "нет" --> K["WORKER_DONE"]
```

## Слияние Worktree

```mermaid
sequenceDiagram
  participant O as Оркестратор
  participant W1 as Worktree worker A
  participant W2 as Worktree worker B
  participant M as Merge-owner

  O->>W1: отдельный scope
  O->>W2: отдельный scope
  W1-->>O: WORKER_DONE + diff
  W2-->>O: WORKER_DONE + diff
  O->>M: принятые пакеты
  M-->>O: итоговый diff + checks
```

## Правила

- Один worker: один outcome и один write scope.
- Parallel только при независимых scope.
- Worktree и merge-owner: инструменты безопасности, не ритуал.
- Heartbeat доказывает жизнь, не качество.
- `WORKER_DONE` принимается только после diff, checks и evidence.

## Файлы

- `SKILL.md`: короткий router и контракт.
- `references/templates.md`: prompt-блоки.
- `references/codex-facts.md`: факты Codex и ссылки.
- `references/worktree-merge.md`: fanout -> merge-owner.
- `agents/openai.yaml`: опциональные метаданные.

## Лицензия

MIT.
