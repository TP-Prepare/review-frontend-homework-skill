# review-homework

Скилл для Claude Code: ревью студенческих домашек по фронтенду в репозиториях
`frontend-park-mail-ru/homework_*`.

## Что делает

1. Собирает контекст PR: метаданные, diff, статус CI, задание варианта,
   предыдущие комментарии
2. Проверяет по чек-листу из трёх уровней — оформление PR, код, тесты
3. Печатает ментору черновик ревью: инлайн-заметки, сводку, предлагаемый вердикт
4. **После подтверждения** постит ревью, ставит лейбл, ассайнит

Формулировки — сократические, собраны из 2903 реальных комментариев менторов
прошлых потоков. Скилл спрашивает, а не приказывает.

## Установка

Репозиторий — плагин Claude Code и одновременно собственный маркетплейс.
Две команды в Claude Code:

```
/plugin marketplace add TP-Prepare/review-frontend-homework-skill
/plugin install review-homework@tp-prepare
```

Дальше Claude Code сам следит за версией и обновлениями — `/plugin update`.

### Без плагина

Если плагины не подходят, скилл можно положить руками. Он самодостаточен:

```bash
git clone https://github.com/TP-Prepare/review-frontend-homework-skill.git /tmp/rh \
  && mkdir -p ~/.claude/skills \
  && cp -r /tmp/rh/skills/review-homework ~/.claude/skills/
```

На Windows, в PowerShell:

```powershell
git clone https://github.com/TP-Prepare/review-frontend-homework-skill.git $env:TEMP\rh
New-Item -ItemType Directory -Force $HOME\.claude\skills
Copy-Item -Recurse $env:TEMP\rh\skills\review-homework $HOME\.claude\skills\
```

Либо в конкретный проект — скопировать `skills/review-homework` в
`.claude/skills/` его репозитория.

## Использование

```
отревьюй PR 4 в homework_2026_2
```

```
проверь домашку в PR 7
```

Скилл дойдёт до черновика и остановится. Прочитай, поправь формулировки при
необходимости, скажи «постим» — и он опубликует.

## Чего не делает

- Не ставит `5/5` — это финальный балл старшего ментора
- Не ставит `Списано` и не проверяет на списывание
- Не мержит PR и не пушит в ветку студента
- Не постит ничего без подтверждения

## Требования

- `gh` CLI, авторизованный с правами на репозиторий курса
- Права ментора: назначать лейблы и ассайны

## Структура

| Файл | Что внутри |
|---|---|
| `skills/review-homework/SKILL.md` | Рабочий процесс из четырёх фаз. Точка входа |
| `skills/review-homework/references/checklist.md` | 22 проверки: `A1`–`A6` блокирующие, `B1`–`B11` код, `C1`–`C5` тесты |
| `skills/review-homework/references/comment-bank.md` | Сократические формулировки под каждый пункт чек-листа |
| `skills/review-homework/references/variants.md` | 19 вариантов задания с подводными камнями |
| `skills/review-homework/references/gh-recipes.md` | Команды `gh`: «Сбор» только читает, «Постинг» под подтверждением |
| `.claude-plugin/plugin.json` | Манифест плагина |
| `.claude-plugin/marketplace.json` | Манифест маркетплейса `tp-prepare` |

Справочники грузятся по необходимости: при ревью `variant-12` не нужны
подводные камни `variant-15`.

## Откуда взялись проверки

`docs/specs/review-homework.md` — спека: разбор 2903 ревью-комментариев из
`homework_2026_1` и `homework_2024_2`, частота тем, обоснование каждого
уровня проверок. `docs/plans/review-homework.md` — план реализации.

Если хочешь поспорить с каким-то пунктом чек-листа — начни со спеки,
там видно, на скольких реальных ревью он основан.
