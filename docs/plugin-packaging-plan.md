# Упаковка скилла в плагин Claude Code — план реализации

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Превратить репозиторий в плагин Claude Code, который ставится одной командой `/plugin install review-homework@tp-prepare`, сохранив скилл работоспособным и его внутренние контракты целыми.

**Architecture:** Репозиторий становится одновременно маркетплейсом и плагином — приём, которым пользуется `superpowers`: в `.claude-plugin/marketplace.json` единственная запись с `"source": "./"`, рядом `.claude-plugin/plugin.json`. Скилл переезжает из корня в `skills/review-homework/`, потому что Claude Code ищет скиллы плагина именно там. Автотестов нет: деливерабл — манифесты и документация, проверки структурные.

**Tech Stack:** Markdown (Claude Code skill format с YAML-фронтматтером), JSON-манифесты по схеме `https://anthropic.com/claude-code/marketplace.schema.json`, `git`, `python` для валидации JSON.

**Spec:** [`plugin-packaging-design.md`](plugin-packaging-design.md)

## Global Constraints

- Репозиторий: `F:\Github\TP-Prepare\review-frontend-homework-skill`, ветка `feat/review-homework-skill` (в ней открыт PR #1 в `main`)
- Имя маркетплейса: `tp-prepare`
- Имя плагина и скилла: `review-homework` — обязано совпадать с `name` во фронтматтере `SKILL.md`
- Версия: `1.0.0`
- Лицензия: MIT
- Скилл после переезда лежит ровно в `skills/review-homework/`
- Все файлы заканчиваются пустой строкой (скилл сам придирается к этому)
- Контракты скилла обязаны пережить переезд: 22 идентификатора `### A1`–`C5` в `checklist.md` совпадают с `## `-заголовками в `comment-bank.md`; 19 секций `## variant-` в `variants.md`; ноль пишущих команд в секции «Сбор» файла `gh-recipes.md`
- **Перевод репозитория в public не входит в объём** — это делает владелец руками
- Никаких изменений в содержании самого скилла: переезд файлов не должен менять ни байта внутри `SKILL.md` и `references/*.md`, кроме случаев, явно описанных в задачах

---

## File Structure

| Файл | Ответственность | Задача |
|---|---|---|
| `skills/review-homework/SKILL.md` | Точка входа скилла, переезжает из корня без изменения содержимого | 1 |
| `skills/review-homework/references/*.md` | Четыре справочника, переезжают без изменения содержимого | 1 |
| `docs/design.md` | Дизайн скилла. Правится дерево архитектуры и фраза про корень репы | 1 |
| `docs/implementation-plan.md` | План сборки скилла. Правится историческая справка, утверждающая, что скилл в корне | 1 |
| `.claude-plugin/plugin.json` | Манифест плагина: имя, описание, версия, автор, лицензия | 2 |
| `.claude-plugin/marketplace.json` | Манифест маркетплейса: одна запись с `"source": "./"` | 2 |
| `LICENSE` | Текст лицензии MIT | 2 |
| `README.md` | Установка через `/plugin install` + запасной вариант, обновлённая таблица структуры | 3 |

Разделение по ответственности: Задача 1 двигает файлы и чинит устаревшие ссылки, Задача 2 добавляет манифесты, Задача 3 переписывает вход для пользователя и проверяет всё вместе на свежем клоне. Ревьюер может отклонить формулировки README, приняв манифесты, — поэтому это разные задачи.

---

### Task 1: Переезд скилла в `skills/review-homework/`

**Files:**
- Move: `SKILL.md` → `skills/review-homework/SKILL.md`
- Move: `references/checklist.md` → `skills/review-homework/references/checklist.md`
- Move: `references/comment-bank.md` → `skills/review-homework/references/comment-bank.md`
- Move: `references/gh-recipes.md` → `skills/review-homework/references/gh-recipes.md`
- Move: `references/variants.md` → `skills/review-homework/references/variants.md`
- Modify: `docs/design.md` — секция «Архитектура»
- Modify: `docs/implementation-plan.md` — блок «Историческая справка»

**Interfaces:**
- Consumes: ничего
- Produces: скилл по пути `skills/review-homework/SKILL.md` со ссылками на `references/checklist.md`, `references/comment-bank.md`, `references/gh-recipes.md`, `references/variants.md` — пути внутри `SKILL.md` относительные и после переезда остаются верными, менять их не нужно. Задача 2 опирается на существование каталога `skills/review-homework/`.

- [ ] **Step 1: Зафиксировать контрольные суммы до переезда**

Содержимое скилла меняться не должно. Снимаем отпечаток, чтобы в конце доказать это.

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
md5sum SKILL.md references/*.md | sort -k2 > /tmp/skill-before.txt
cat /tmp/skill-before.txt
```
Expected: пять строк — `SKILL.md` и четыре файла из `references/`.

- [ ] **Step 2: Переместить файлы через `git mv`**

`git mv`, а не `cp` + `rm` — так git запишет перемещение и дифф в PR останется читаемым.

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
mkdir -p skills/review-homework
git mv SKILL.md skills/review-homework/SKILL.md
git mv references skills/review-homework/references
git status --short
```
Expected: строки вида `R  SKILL.md -> skills/review-homework/SKILL.md` и по одной `R` на каждый файл справочников.

- [ ] **Step 3: Проверить, что содержимое не изменилось**

Относительные пути после `cd` в каталог скилла те же самые (`SKILL.md`,
`references/*.md`), поэтому вывод сравнивается напрямую, без правки путей.

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill/skills/review-homework"
md5sum SKILL.md references/*.md | sort -k2 > /tmp/skill-after.txt
diff /tmp/skill-before.txt /tmp/skill-after.txt && echo "СОДЕРЖИМОЕ ИДЕНТИЧНО"
```
Expected: печатает `СОДЕРЖИМОЕ ИДЕНТИЧНО`. Если diff непустой — переезд что-то испортил, откатись через `git checkout -- .` и разберись до продолжения.

- [ ] **Step 4: Проверить, что контракты скилла целы**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill/skills/review-homework/references"
diff <(grep -oE '^### [ABC][0-9]+' checklist.md | sed 's/### //') \
     <(grep -oE '^## [ABC][0-9]+' comment-bank.md | sed 's/## //') && echo "22 идентификатора: OK"
echo "вариантов: $(grep -c '^## variant-' variants.md)"
echo "пишущих команд в Сборе: $(awk '/^## Сбор/,/^## Постинг/' gh-recipes.md | grep -cE 'method (POST|PATCH|PUT|DELETE)|pr edit|pr comment')"
```
Expected: `22 идентификатора: OK`, `вариантов: 19`, `пишущих команд в Сборе: 0`.

- [ ] **Step 5: Проверить, что ссылки внутри `SKILL.md` разрешаются**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill/skills/review-homework"
for f in $(grep -oE 'references/[a-z-]+\.md' SKILL.md | sort -u); do
  printf "%-30s %s\n" "$f" "$([ -f "$f" ] && echo OK || echo MISSING)"
done
```
Expected: четыре строки, все `OK`.

- [ ] **Step 6: Поправить дерево архитектуры в `docs/design.md`**

Заменить блок от строки с ``` перед `SKILL.md` до абзаца про корень репозитория включительно.

Было:

````markdown
```
SKILL.md                — рабочий процесс, 4 фазы
references/
  checklist.md          — проверки по приоритетам
  comment-bank.md       — сократические формулировки + ссылки
  variants.md           — 19 вариантов: функция, подводные камни, тест-кейсы
  gh-recipes.md         — команды gh для сбора и постинга
```

При ревью `variant-12` не нужны подводные камни `variant-15` — грузится только
нужная секция.

Скилл занимает корень этого репозитория, поэтому папку под него выбирает тот,
кто клонирует: `git clone <url> ~/.claude/skills/review-homework`.
````

Стало:

````markdown
```
skills/review-homework/
  SKILL.md              — рабочий процесс, 4 фазы
  references/
    checklist.md        — проверки по приоритетам
    comment-bank.md     — сократические формулировки + ссылки
    variants.md         — 19 вариантов: функция, подводные камни, тест-кейсы
    gh-recipes.md       — команды gh для сбора и постинга
```

При ревью `variant-12` не нужны подводные камни `variant-15` — грузится только
нужная секция.

Репозиторий упакован как плагин Claude Code, поэтому скилл лежит в `skills/` —
именно там Claude Code ищет скиллы плагина. Как это устроено — в
[`plugin-packaging-design.md`](plugin-packaging-design.md).
````

- [ ] **Step 7: Поправить историческую справку в `docs/implementation-plan.md`**

Справка сейчас прямо утверждает, что скилл лежит в корне. После переезда это ложь.

Было:

```markdown
> **Историческая справка.** Это план, по которому скилл был собран 2026-09-07.
> Пути вида `F:\Github\TP-Prepare\skills\review-homework\…` указывают на рабочую
> директорию сборки, а не на этот репозиторий: здесь скилл лежит в корне, без
> каталога `review-homework/`. Пути оставлены как есть, чтобы план честно
> описывал то, что действительно исполнялось.
```

Стало:

```markdown
> **Историческая справка.** Это план, по которому скилл был собран 2026-09-07.
> Пути вида `F:\Github\TP-Prepare\skills\review-homework\…` указывают на рабочую
> директорию сборки, а не на этот репозиторий: здесь скилл лежит в
> `skills/review-homework/`, потому что репозиторий упакован как плагин. Пути
> оставлены как есть, чтобы план честно описывал то, что действительно
> исполнялось.
```

- [ ] **Step 8: Убедиться, что нигде не осталось ссылок на старую раскладку**

`README.md` в этот список попадёт — его чинит Задача 3, здесь он ожидаемо в выводе.

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
grep -rn 'skills/review-homework' docs/ | head
grep -n 'скилл лежит в корне\|занимает корень' docs/*.md README.md
```
Expected: первая команда печатает свежие упоминания в `docs/design.md` и `docs/implementation-plan.md`; вторая не печатает ничего.

- [ ] **Step 9: Проверить пустую строку в конце изменённых файлов**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
for f in docs/design.md docs/implementation-plan.md skills/review-homework/SKILL.md; do
  printf "%-45s " "$f"; [ "$(tail -c 1 "$f" | xxd -p)" = "0a" ] && echo OK || echo FAIL
done
```
Expected: три строки `OK`.

- [ ] **Step 10: Коммит**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
git add -A
git commit -m "refactor: скилл переезжает в skills/review-homework под упаковку в плагин"
```

---

### Task 2: Манифесты плагина и маркетплейса

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`
- Create: `LICENSE`

**Interfaces:**
- Consumes: каталог `skills/review-homework/` из Задачи 1
- Produces: маркетплейс с именем `tp-prepare`, содержащий плагин `review-homework` версии `1.0.0` с `"source": "./"`. Задача 3 использует эти два имени в командах установки в README: `/plugin marketplace add TP-Prepare/review-frontend-homework-skill` и `/plugin install review-homework@tp-prepare`.

- [ ] **Step 1: Создать `.claude-plugin/plugin.json`**

```bash
mkdir -p "F:/Github/TP-Prepare/review-frontend-homework-skill/.claude-plugin"
```

Записать в `.claude-plugin/plugin.json`:

```json
{
  "name": "review-homework",
  "description": "Ревью студенческих домашек по фронтенду в репозиториях frontend-park-mail-ru/homework_*: собирает контекст PR, проверяет по чек-листу и готовит ментору черновик сократического ревью",
  "version": "1.0.0",
  "author": {
    "name": "Yaroslav Mihalev"
  },
  "homepage": "https://github.com/TP-Prepare/review-frontend-homework-skill",
  "repository": "https://github.com/TP-Prepare/review-frontend-homework-skill",
  "license": "MIT",
  "keywords": [
    "code-review",
    "education",
    "javascript",
    "mentoring"
  ]
}
```

- [ ] **Step 2: Создать `.claude-plugin/marketplace.json`**

`"source": "./"` означает «плагин лежит в корне этого же репозитория» — приём из `superpowers`.

Записать в `.claude-plugin/marketplace.json`:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "tp-prepare",
  "description": "Скиллы для менторов Технопарка VK",
  "owner": {
    "name": "TP-Prepare"
  },
  "plugins": [
    {
      "name": "review-homework",
      "description": "Ревью студенческих домашек по фронтенду в репозиториях frontend-park-mail-ru/homework_*",
      "version": "1.0.0",
      "source": "./",
      "author": {
        "name": "Yaroslav Mihalev"
      },
      "category": "development",
      "homepage": "https://github.com/TP-Prepare/review-frontend-homework-skill"
    }
  ]
}
```

- [ ] **Step 3: Создать `LICENSE`**

Записать в `LICENSE`:

```
MIT License

Copyright (c) 2026 Yaroslav Mihalev

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 4: Проверить, что оба JSON валидны**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
python -m json.tool .claude-plugin/plugin.json > /dev/null && echo "plugin.json: валиден"
python -m json.tool .claude-plugin/marketplace.json > /dev/null && echo "marketplace.json: валиден"
```
Expected: обе строки про валидность.

- [ ] **Step 5: Проверить согласованность имён между манифестами и скиллом**

Имя плагина в обоих манифестах, имя скилла во фронтматтере и имя каталога обязаны совпадать.

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
python - <<'PY'
import json, re, pathlib
plugin = json.load(open('.claude-plugin/plugin.json', encoding='utf-8'))
market = json.load(open('.claude-plugin/marketplace.json', encoding='utf-8'))
entry = market['plugins'][0]
skill = pathlib.Path('skills/review-homework/SKILL.md').read_text(encoding='utf-8')
front = re.search(r'^---\n(.*?)\n---', skill, re.S).group(1)
skill_name = re.search(r'^name:\s*(\S+)', front, re.M).group(1)

checks = {
    'plugin.json name == marketplace entry name': plugin['name'] == entry['name'],
    'plugin.json version == marketplace entry version': plugin['version'] == entry['version'],
    'имя плагина == имя скилла во фронтматтере': plugin['name'] == skill_name,
    'имя плагина == имя каталога скилла': plugin['name'] == 'review-homework',
    'source указывает на корень репы': entry['source'] == './',
    'маркетплейс называется tp-prepare': market['name'] == 'tp-prepare',
    'в маркетплейсе ровно один плагин': len(market['plugins']) == 1,
}
for k, v in checks.items():
    print(('OK   ' if v else 'FAIL ') + k)
raise SystemExit(0 if all(checks.values()) else 1)
PY
```
Expected: семь строк `OK`, код возврата 0.

- [ ] **Step 6: Проверить, что скилл виден по пути, на который указывает `source`**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
test -f ./skills/review-homework/SKILL.md && echo "скилл на месте относительно source"
```
Expected: `скилл на месте относительно source`.

- [ ] **Step 7: Пустая строка в конце `LICENSE`**

JSON-файлы `python -m json.tool` не проверяет на перенос, а `LICENSE` — обычный текст.

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
for f in LICENSE .claude-plugin/plugin.json .claude-plugin/marketplace.json; do
  printf "%-38s " "$f"; [ "$(tail -c 1 "$f" | xxd -p)" = "0a" ] && echo OK || echo FAIL
done
```
Expected: три строки `OK`. Если `FAIL` — дописать перенос: `printf '\n' >> <файл>`.

- [ ] **Step 8: Коммит**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
git add .claude-plugin LICENSE
git commit -m "feat: манифесты плагина и маркетплейса tp-prepare"
```

---

### Task 3: README под установку плагином и проверка на свежем клоне

**Files:**
- Modify: `README.md` — секции «Установка» и «Структура»

**Interfaces:**
- Consumes: имена `tp-prepare` и `review-homework` из манифестов Задачи 2; раскладку `skills/review-homework/` из Задачи 1
- Produces: конечный деливерабл, дальше задач нет

- [ ] **Step 1: Заменить секцию «Установка» в `README.md`**

Было:

````markdown
## Установка

Персонально, для всех проектов — клонировать сразу в папку скиллов:

```bash
git clone https://github.com/TP-Prepare/review-frontend-homework-skill.git \
  ~/.claude/skills/review-homework
```

На Windows, в PowerShell:

```powershell
git clone https://github.com/TP-Prepare/review-frontend-homework-skill.git `
  $HOME\.claude\skills\review-homework
```

Обновиться потом — `git pull` в этой папке.

Либо в конкретный проект — клонировать в `.claude/skills/review-homework`
его репозитория.

Имя папки важно: Claude Code берёт имя скилла из неё, а `SKILL.md` лежит
в корне этого репозитория.
````

Стало:

````markdown
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
````

- [ ] **Step 2: Заменить таблицу в секции «Структура»**

Было:

```markdown
| Файл | Что внутри |
|---|---|
| `SKILL.md` | Рабочий процесс из четырёх фаз. Точка входа |
| `references/checklist.md` | 22 проверки: `A1`–`A6` блокирующие, `B1`–`B11` код, `C1`–`C5` тесты |
| `references/comment-bank.md` | Сократические формулировки под каждый пункт чек-листа |
| `references/variants.md` | 19 вариантов задания с подводными камнями |
| `references/gh-recipes.md` | Команды `gh`: «Сбор» только читает, «Постинг» под подтверждением |
```

Стало:

```markdown
| Файл | Что внутри |
|---|---|
| `skills/review-homework/SKILL.md` | Рабочий процесс из четырёх фаз. Точка входа |
| `skills/review-homework/references/checklist.md` | 22 проверки: `A1`–`A6` блокирующие, `B1`–`B11` код, `C1`–`C5` тесты |
| `skills/review-homework/references/comment-bank.md` | Сократические формулировки под каждый пункт чек-листа |
| `skills/review-homework/references/variants.md` | 19 вариантов задания с подводными камнями |
| `skills/review-homework/references/gh-recipes.md` | Команды `gh`: «Сбор» только читает, «Постинг» под подтверждением |
| `.claude-plugin/plugin.json` | Манифест плагина |
| `.claude-plugin/marketplace.json` | Манифест маркетплейса `tp-prepare` |
```

- [ ] **Step 3: Проверить, что в README не осталось старых путей установки**

Старый однострочник клонировал прямо в `~/.claude/skills/review-homework`, а
новый запасной вариант копирует в `~/.claude/skills/` — поэтому наличие первой
строки означает, что старый текст остался.

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
echo "новые пути в README:"; grep -c 'skills/review-homework' README.md
echo "остатки старой установки (должно быть 0):"
grep -cE 'claude/skills/review-homework|SKILL\.md` лежит' README.md
```
Expected: первое число больше нуля, второе — ровно `0`. `grep -c` при отсутствии совпадений печатает `0` и возвращает код 1 — это нормально, ориентируйся на число.

- [ ] **Step 4: Проверить сквозную согласованность имён README и манифестов**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
python - <<'PY'
import json, pathlib
market = json.load(open('.claude-plugin/marketplace.json', encoding='utf-8'))
readme = pathlib.Path('README.md').read_text(encoding='utf-8')
name, plugin = market['name'], market['plugins'][0]['name']
cmd = f"/plugin install {plugin}@{name}"
checks = {
    f'README содержит "{cmd}"': cmd in readme,
    'README содержит команду добавления маркетплейса':
        '/plugin marketplace add TP-Prepare/review-frontend-homework-skill' in readme,
}
for k, v in checks.items():
    print(('OK   ' if v else 'FAIL ') + k)
raise SystemExit(0 if all(checks.values()) else 1)
PY
```
Expected: две строки `OK`, код возврата 0.

- [ ] **Step 5: Финальная проверка структуры репозитория**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
find . -type f -not -path './.git/*' -not -path './.idea/*' | sort
```
Expected ровно этот список:
```
./.claude-plugin/marketplace.json
./.claude-plugin/plugin.json
./.gitattributes
./.gitignore
./LICENSE
./README.md
./docs/design.md
./docs/implementation-plan.md
./docs/plugin-packaging-design.md
./docs/plugin-packaging-plan.md
./skills/review-homework/SKILL.md
./skills/review-homework/references/checklist.md
./skills/review-homework/references/comment-bank.md
./skills/review-homework/references/gh-recipes.md
./skills/review-homework/references/variants.md
```

- [ ] **Step 6: Проверить пустую строку в конце всех markdown-файлов**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
for f in $(find . -name '*.md' -not -path './.git/*' -not -path './.idea/*'); do
  [ "$(tail -c 1 "$f" | xxd -p)" = "0a" ] && echo "OK   $f" || echo "FAIL $f"
done
```
Expected: все строки начинаются с `OK`.

- [ ] **Step 7: Проверить установку на свежем клоне**

Главная проверка задачи: собрать репозиторий так, как его увидит Claude Code, и убедиться, что от корня, куда указывает `"source": "./"`, скилл находится.

```bash
rm -rf /tmp/rh-check
git clone -q --branch feat/review-homework-skill \
  "F:/Github/TP-Prepare/review-frontend-homework-skill" /tmp/rh-check
cd /tmp/rh-check
python - <<'PY'
import json, pathlib, re
market = json.load(open('.claude-plugin/marketplace.json', encoding='utf-8'))
root = pathlib.Path(market['plugins'][0]['source'])
manifest = root / '.claude-plugin' / 'plugin.json'
skill = root / 'skills' / 'review-homework' / 'SKILL.md'
print('OK   plugin.json найден' if manifest.is_file() else 'FAIL plugin.json не найден')
print('OK   SKILL.md найден' if skill.is_file() else 'FAIL SKILL.md не найден')
front = re.search(r'^---\n(.*?)\n---', skill.read_text(encoding='utf-8'), re.S)
print('OK   фронтматтер разобран' if front else 'FAIL фронтматтер битый')
refs = sorted(p.name for p in (skill.parent / 'references').glob('*.md'))
print('OK   справочники:', refs) if len(refs) == 4 else print('FAIL справочников', len(refs))
PY
cd - > /dev/null
rm -rf /tmp/rh-check
```
Expected: четыре строки, все начинаются с `OK`, последняя перечисляет `['checklist.md', 'comment-bank.md', 'gh-recipes.md', 'variants.md']`.

- [ ] **Step 8: Коммит**

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
git add README.md
git commit -m "docs: установка через /plugin install вместо клонирования"
```

---

## После плана

Ветка `feat/review-homework-skill` уже несёт открытый PR #1 в `main`, поэтому эти три коммита доедут туда же — отдельный PR не нужен. Пуш:

```bash
cd "F:/Github/TP-Prepare/review-frontend-homework-skill"
git push https://github.com/TP-Prepare/review-frontend-homework-skill.git feat/review-homework-skill
```

`origin` в репозитории настроен на SSH, а ключ в текущем окружении не проходит, поэтому пушим по HTTPS явным URL.

Две вещи остаются за владельцем репозитория, агент их не делает:

1. **Перевести репозиторий в public** — до этого `/plugin marketplace add` сработает только у тех, у кого есть доступ
2. **Смержить PR #1**

## Самопроверка плана

**Покрытие спеки:**

| Требование спеки | Задача |
|---|---|
| Скилл переезжает в `skills/review-homework/` | 1 |
| Содержимое скилла не меняется | 1, шаги 1 и 3 |
| Контракты скилла целы | 1, шаг 4 |
| `docs/design.md` — дерево и фраза про корень | 1, шаг 6 |
| `docs/implementation-plan.md` — историческая справка | 1, шаг 7 |
| `.claude-plugin/plugin.json` | 2, шаг 1 |
| `.claude-plugin/marketplace.json` с `"source": "./"` | 2, шаг 2 |
| Лицензия MIT | 2, шаг 3 |
| Имя маркетплейса `tp-prepare`, плагина `review-homework`, версия `1.0.0` | 2, шаги 1, 2, 5 |
| README на `/plugin install` | 3, шаг 1 |
| Запасная установка без плагина | 3, шаг 1 |
| README — таблица структуры | 3, шаг 2 |
| Манифесты парсятся | 2, шаг 4 |
| Фронтматтер `name: review-homework` | 2, шаг 5; 3, шаг 7 |
| Ни одной ссылки на старые пути | 1, шаг 8; 3, шаг 3 |
| Проверка на свежем клоне | 3, шаг 7 |
| Перевод репы в public — вне объёма | нигде, как и задумано |

**Согласованность имён:** `review-homework` фигурирует как имя плагина в обоих манифестах, имя скилла во фронтматтере, имя каталога и правая часть `@`-адреса в README — сверяется программно в Задаче 2 шаг 5 и Задаче 3 шаг 4. `tp-prepare` — имя маркетплейса в `marketplace.json` и в команде README, сверяется там же. Версия `1.0.0` дублируется в `plugin.json` и в записи маркетплейса, сверяется в Задаче 2 шаг 5.

**Плейсхолдеры:** отсутствуют, каждый шаг содержит итоговый текст файла либо команду с ожидаемым выводом.
