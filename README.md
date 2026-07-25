# Autopilot

[![skills.sh](https://skills.sh/b/nick-vels/autopilot)](https://skills.sh/nick-vels/autopilot)

**Скилл для Claude Code, который доводит продиктованную идею до работающего проекта за один диалог.**

Ты говоришь, что нужно собрать. Агент задаёт 5–8 решающих вопросов — и дальше сам проходит весь инженерный пайплайн: спека → тикеты → реализация. Каждый тикет пишет отдельный субагент со свежим контекстом. Тебе не нужно читать спеку, оценивать декомпозицию и ревьюить код.

Autopilot — оркестратор поверх [mattpocock/skills](https://github.com/mattpocock/skills). Он ничего не изобретает заново: он убирает из проверенного пайплайна человеческие «подтверди-ка» и заменяет их автоматикой.

## Установка

```bash
npx skills add nick-vels/autopilot
```

Autopilot требует пак Мэтта Пакока — поставь его тоже (нужны `grilling`, `to-spec`, `to-tickets`, `implement`, а также `tdd` и `code-review`):

```bash
npx skills add mattpocock/skills
```

Оба пака ставятся в тот проект, где ты запускаешь команду. Хочешь, чтобы скилл был доступен во всех проектах — добавь флаг `-g`:

```bash
npx skills add nick-vels/autopilot -g
npx skills add mattpocock/skills -g
```

### Второй способ: плагин Claude Code

Если не хочешь держать файл скилла у себя в проекте — поставь его как плагин, он обновляется сам:

```
/plugin marketplace add nick-vels/autopilot
/plugin install autopilot@nick-vels
```

Разница простая: через `npx skills` файл скилла копируется к тебе в проект, и его можно править под себя. Плагин — это подписка: файлы остаются read-only и подтягивают обновления. Скилл при установке плагином вызывается как `/autopilot:autopilot` (имя плагина + имя скилла), через `npx skills` — просто `/autopilot`.

## Как пользоваться

Открой Claude Code в пустой папке будущего проекта и продиктуй задачу:

```
/autopilot Собери телеграм-бота, который принимает голосовые и присылает текстовую расшифровку
```

Дальше от тебя нужны только ответы на вопросы в первой фазе. Всё остальное агент сделает сам и в конце скажет, чем запускать проект.

Скилл сработает и без слэш-команды — если ты пишешь «собери под ключ», «просто сделай», «не задавай лишних вопросов», Claude подхватит его сам.

## Что происходит под капотом

| Фаза | Что делает | Твоё участие |
|------|-----------|--------------|
| **0. Проводка** | Находит скиллы пайплайна, назначает трекер задач в локальные файлы | — |
| **1. Grilling** | 5–8 вопросов по существу; блокирующие неизвестные (оплата, ключи, хостинг) идут первыми | **Отвечаешь** |
| **2. Спека** | Синтезирует спеку из твоих ответов, `git init`, первый коммит | — |
| **3. Тикеты** | Режет работу на вертикальные срезы с зависимостями, показывает одним экраном человеческим языком | Можешь сказать «стоп» |
| **4. Реализация** | Один тикет = один субагент = свежий контекст. Один коммит на тикет | — |
| **5. Финиш** | Прогоняет тесты, отчитывается: что собрано, чем запускать, что осталось | — |

Ключевой принцип: **порядок и есть продукт**. Код пишется только в последней фазе — до этого агент выясняет, что именно ты имел в виду, и фиксирует это в документе, к которому можно вернуться.

## Что даёт этот порядок

- **Вопросы решаются в начале, а не в конце.** Классический провал вайбкодинга — узнать про платёжку и ключи на финише, когда всё уже собрано не так.
- **Свежий контекст на каждый тикет.** Модель не ходит кругами и не ломает то, что уже работало.
- **Коммит на тикет.** Любой шаг можно откатить.
- **Ничего не додумывается молча.** Если решение за тобой, а тебя не спросили — в спеке останется явный `PLACEHOLDER`.

## Когда НЕ использовать

- Ты хочешь проектировать вместе, шаг за шагом → используй скиллы Мэтта напрямую (`/grill-me`, `/to-spec`, `/to-tickets`, `/implement`).
- Задача — правка в один файл → просто попроси сделать её.
- Идея огромная и туманная, больше одного проекта → сначала `/wayfinder`, потом сюда.

## Совместимость

Скилл написан под [Claude Code](https://code.claude.com) и опирается на его модель приглашения скиллов: `to-spec`, `to-tickets` и `implement` помечены `disable-model-invocation: true`, то есть модель не может вызвать их сама. Autopilot это учитывает — он читает их инструкции напрямую и исполняет дословно. В других агентах (Codex, Cursor, OpenCode) скилл поставится через `npx skills`, но поведение фаз может отличаться.

---

## English

**A Claude Code skill that turns a dictated idea into a working project in a single dialogue.**

You say what to build. The agent asks 5–8 decisive questions, then runs the whole engineering pipeline on its own — spec → tickets → implementation — with one fresh subagent per ticket. You never have to read a spec, judge ticket granularity, or review code.

Autopilot is an orchestrator on top of [mattpocock/skills](https://github.com/mattpocock/skills). It invents nothing: it strips the human approval gates out of a proven pipeline and automates them away.

### Install

```bash
npx skills add nick-vels/autopilot
npx skills add mattpocock/skills   # required: grilling, to-spec, to-tickets, implement, tdd, code-review
```

Add `-g` to install globally instead of into the current project.

Or install it as a Claude Code plugin — read-only, self-updating:

```
/plugin marketplace add nick-vels/autopilot
/plugin install autopilot@nick-vels
```

Installed via `npx skills` the skill is `/autopilot`; installed as a plugin it is namespaced as `/autopilot:autopilot`.

### Use

```
/autopilot Build me a Telegram bot that transcribes voice messages
```

Answer the questions in phase 1. Everything after that is hands-free; the final report tells you the exact command to run the project.

The skill also triggers without the slash command on requests like "just build it" or "don't ask me unnecessary questions".

### Pipeline

1. **Wiring** — locate the pipeline skills, set the issue tracker to local files.
2. **Grill** — 5–8 questions; blocking unknowns (payments, keys, hosting) go first. *The only human gate.*
3. **Spec** — synthesized from your answers, `git init`, first commit.
4. **Tickets** — vertical slices with blocking edges, shown as one plain-language screen.
5. **Implement** — one ticket = one subagent = one fresh context, one commit per ticket.
6. **Finish** — full test suite, then a report: what was built, how to run it, what is left.

The core principle: **the order is the product.** Code is written only in the last phase.

### Note on Claude Code specifics

`to-spec`, `to-tickets` and `implement` ship with `disable-model-invocation: true`, so a skill cannot invoke them through the Skill tool. Autopilot handles this by reading their `SKILL.md` files and following them verbatim. The skill installs into other agents via `npx skills`, but phase behaviour may differ there.

## License

MIT
