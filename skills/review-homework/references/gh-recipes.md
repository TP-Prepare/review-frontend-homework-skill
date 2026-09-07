# Рецепты gh

Везде `R` — репозиторий потока, `N` — номер PR.

```bash
R=frontend-park-mail-ru/homework_2026_2
```

## Сбор

### Метаданные PR

```bash
gh pr view N -R $R --json number,title,author,baseRefName,headRefName,labels,assignees,isCrossRepository,changedFiles,additions,deletions,url
```

`baseRefName` даёт вариант. `isCrossRepository: true` — норма, студенты форкают.

### Изменённые файлы

```bash
gh api "repos/$R/pulls/N/files" -q '.[] | .filename + "  +" + (.additions|tostring) + " -" + (.deletions|tostring)'
```

Красный флаг: у файла число удалений примерно равно числу строк файла, а
добавлений — чуть больше. Значит файл переписан целиком, обычно из-за смены
переносов строк (CRLF ↔ LF). Проверка — в `checklist.md`, пункт A5.

### Diff

```bash
gh pr diff N -R $R
```

### Статус CI

```bash
gh pr checks N -R $R
```

Если печатает `no checks reported` — проверить, не ждёт ли запуск одобрения:

```bash
gh run list -R $R --branch <headRefName> --json conclusion,databaseId
```

`conclusion: action_required` (длительность `0s`) — это гейт GitHub для
первого контрибьютора с форка: запуск создан, но ждёт одобрения мейнтейнера
во вкладке Actions репозитория. Это не вина студента, но стоит упомянуть в
сводке: автопроверки не прогонялись, запуск можно одобрить во вкладке Actions,
а до этого `eslint` и тесты нужно просмотреть глазами внимательнее.

При падении — лог упавшего шага:

```bash
gh run list -R $R --branch <headRefName> --limit 3 --json databaseId,conclusion,headSha
gh run view <databaseId> -R $R --log-failed
```

### Задание варианта

```bash
gh api "repos/$R/contents/README.md?ref=variant-<N>" -q '.content' | base64 -d | sed -n '/#### Задание/,/### Файлы/p'
```

### Переносы строк

Перенос в конце файла в диффе **не виден**. Признак обратный: git дописывает
`\ No newline at end of file`, только если переноса нет.

```bash
gh api "repos/$R/pulls/N/files" -q '.[] | .filename + " ::: " + (.patch | split("\n") | .[-2:] | join(" ⏎ "))'
```

Маркера нет — перенос на месте.

Проверка CRLF/LF, до любого совета по A5: смотрим последние байты файла в
**базовой ветке**, а не в PR.

```bash
gh api "repos/$R/contents/test/<функция>.js?ref=variant-N" -q .content | base64 -d | tail -c 16 | xxd
```

`0d0a` в конце — репозиторий на CRLF (так и есть в `homework_2026_2`), `0a` —
на LF. Советовать студенту переключить переносы можно только против того, что
реально лежит в базовой ветке.

### Существующие комментарии

Обязательно перед повторным ревью.

```bash
gh api "repos/$R/pulls/N/comments" -q '.[] | "[\(.id)] [\(.user.login)] \(.path):\(.line // .original_line)\n\(.body)\n---"'
gh api "repos/$R/issues/N/comments" -q '.[] | "[\(.user.login)]\n\(.body)\n---"'
gh pr view N -R $R --json reviews -q '.reviews[] | "\(.author.login): \(.state)"'
```

`.id` из первого рецепта — это `comment_id` для ответа в существующем треде,
рецепт «Ответ в существующем треде» в секции «Постинг».

## Постинг

Только после подтверждения ментора.

### Ассайн ментора

```bash
gh pr edit N -R $R --add-assignee @me
```

**После ассайна перечитать лейблы.** Автоматика репозитория сама вешает
«На проверке», когда на PR появляется ассайни. Если ты потом ставишь
«Нужны исправления», статусных лейблов окажется два.

```bash
gh pr view N -R $R --json labels -q '[.labels[].name] | join(", ")'
```

### Ревью с инлайн-комментариями

Собрать payload в файл — так надёжнее, чем передавать вложенные объекты флагами.
Писать во временную директорию, не в репозиторий.

```bash
cat > "${TMPDIR:-/tmp}/review.json" <<'JSON'
{
  "body": "Привет! Посмотрел решение. В целом хорошо, оставил несколько вопросов и замечаний — глянь, пожалуйста.",
  "event": "COMMENT",
  "comments": [
    {
      "path": "source/flatten.js",
      "line": 13,
      "side": "RIGHT",
      "body": "Что произойдёт, если в функцию передать не массив — например, `null` или строку? Давай добавим проверку входных данных и тесты на неё."
    }
  ]
}
JSON

gh api "repos/$R/pulls/N/reviews" --method POST --input "${TMPDIR:-/tmp}/review.json"
```

Правила по `line`:

- `line` — номер строки в **новой** версии файла, `side` всегда `RIGHT`
- Строка обязана присутствовать в diff, иначе API вернёт `422`
- Для комментария к удалённой строке — `side: "LEFT"`
- Для диапазона — добавить `start_line` и `start_side`

`event` держать `COMMENT`, пока есть замечания. `APPROVE` — только когда
замечаний не осталось. `REQUEST_CHANGES` в этом курсе менторы почти не
используют, роль «нужны правки» играет лейбл.

Если API вернул `422 Unprocessable Entity` — почти всегда строка вне diff.
Найти ближайшую изменённую строку и перепривязать комментарий.

### Лейблы

Ставить ровно один статусный лейбл, предыдущий снимать.

```bash
# взял в работу
gh pr edit N -R $R --add-label "На проверке"

# отправил замечания
gh pr edit N -R $R --remove-label "На проверке" --add-label "Нужны исправления"

# всё хорошо, передаём старшему ментору
gh pr edit N -R $R --remove-label "Нужны исправления" --remove-label "На проверке" --add-label "ОК от ментора"
gh pr edit N -R $R --add-assignee DPeshkoff

# PR нацелен в master, или студент работал в форкнутом мастере (A1)
gh pr edit N -R $R --add-label "Ошибочный"
```

Лейблы `5/5` и `Списано` не ставим никогда.

### Одиночный инлайн-комментарий

Когда ревью уже отправлено, а ментор захотел добавить одну заметку. Через
`pulls/N/reviews` это дало бы **второе ревью со второй сводкой** в PR —
студенту непонятно, какая из них актуальная. Одиночный коммент заводит один
тред и ничего больше.

```bash
SHA=$(gh pr view N -R $R --json headRefOid -q .headRefOid)

cat > "${TMPDIR:-/tmp}/one.json" <<JSON
{
  "path": "source/<функция>.js",
  "line": 13,
  "side": "RIGHT",
  "commit_id": "$SHA",
  "body": "текст заметки"
}
JSON

gh api "repos/$R/pulls/N/comments" --method POST --input "${TMPDIR:-/tmp}/one.json"
```

`commit_id` обязателен и должен быть головным коммитом PR. Правила по `line` —
те же, что для ревью выше.

### Ответ в существующем треде

Использовать вместо нового инлайн-комментария на той же строке — особенно
когда студент уже ответил вопросом. `comment_id` — это `.id` из рецепта
«Существующие комментарии» в секции «Сбор».

```bash
gh api "repos/$R/pulls/N/comments" --method POST -f body="текст" -F in_reply_to=<comment_id>
```

### Правка уже опубликованного комментария

Когда переформулировал замечание после того, как оно ушло. `comment_id` — тот же
`.id` из рецепта «Существующие комментарии».

```bash
cat > "${TMPDIR:-/tmp}/patch.json" <<'JSON'
{
  "body": "новый текст замечания"
}
JSON

gh api "repos/$R/pulls/comments/<comment_id>" --method PATCH --input "${TMPDIR:-/tmp}/patch.json"
```

GitHub пометит комментарий как отредактированный и сохранит историю правок —
студент увидит, что текст менялся. Это нормально для уточнения формулировки.

Если замечание оказалось ошибочным целиком, честнее удалить его и написать
новое, чем незаметно подменить смысл под старым:

```bash
gh api "repos/$R/pulls/comments/<comment_id>" --method DELETE
```

### Просто комментарий без ревью

Только после подтверждения, как и всё в этой секции.

```bash
gh pr comment N -R $R --body "текст"
```
