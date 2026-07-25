---
name: autopilot
description: Use when the user dictates an app, site, bot, or feature to build end-to-end and expects a finished result without reviewing specs, tickets, or code — vibecoding sessions, non-technical users, "собери под ключ", "build it for me", "не задавай лишних вопросов" requests. Also use when the user explicitly invokes /autopilot.
---

# Autopilot

## Overview

Autopilot drives a dictated idea through the mattpocock/skills pipeline — grill → spec → tickets → implement — **in one dialogue**, without making the user approve each stage. The user answers questions once at the start and receives a working project at the end. Core principle: **the order is the product** — code is written only in the last phase, and every ticket is implemented by a separate subagent with a fresh, isolated context.

This skill only orchestrates — the phases below **invoke the pipeline skills and follow their rules**; nothing here restates them. What Autopilot adds: which human gates to remove, how to reach the skills the model is not allowed to invoke, and how to run implementation hands-free.

**Prerequisite:** the pipeline skills must be installed — `grilling`, `to-spec`, `to-tickets`, `implement` (mattpocock/skills pack). If any is missing, install first:

```bash
npx skills add mattpocock/skills
```

## When to Use

- User dictates what to build and expects the finished thing, not a collaboration on process.
- User is non-technical: will not read specs, judge ticket granularity, or review code.
- "Собери под ключ", "just build it", "не задавай лишних вопросов".

**When NOT to use:** the user wants to co-design step by step (use the underlying skills manually); the task is a small single-file change (just do it); the idea is huge and foggy — bigger than one project, destination unclear (run `/wayfinder` first, then return here).

## Phase 0 — Wiring (before asking the user anything)

**Two kinds of pipeline skill.** `grilling`, `tdd` and `code-review` are model-invocable — reach them with the Skill tool as normal. `to-spec`, `to-tickets` and `implement` ship with `disable-model-invocation: true`: **the Skill tool cannot load them, and they are never preloaded into a subagent.** Naming them and hoping is the single most common way this pipeline degrades back into freestyle vibecoding.

For those three, **read the installed `SKILL.md` and follow its body verbatim** — exactly as if the user had typed the slash command. Locate each with Glob, first hit wins:

```
.claude/skills/<name>/SKILL.md            # project install
~/.claude/skills/<name>/SKILL.md          # global install
**/plugins/**/skills/**/<name>/SKILL.md   # Claude Code plugin install
```

Nothing found → the pack isn't installed. Run `npx skills add mattpocock/skills`, confirm the files exist, then start Phase 1.

**Issue tracker: local files, no setup interview.** `to-spec` and `to-tickets` expect `/setup-matt-pocock-skills` to have named a tracker. Do not run it — it is an interview, and a vibecoder has no issue tracker. Unless `docs/agents/issue-tracker.md` already says otherwise, use local markdown and say so when following those skills:

- spec → `.scratch/<feature-slug>/spec.md`
- tickets → `.scratch/<feature-slug>/issues/<NN>-<slug>.md`

**Language.** Everything the user reads — questions, the ticket screen, per-ticket progress, the final report — in the language they dictated in.

## The Pipeline

### Phase 1 — Grill (the only human gate)

Invoke `grilling` (Skill tool) on the dictated idea. Autopilot adds three rules:

- **Blocking unknowns first.** Anything the build depends on but the user hasn't decided (payment provider, hosting, API keys, accounts) goes into the first three questions — never the finish line.
- **Never answer for the user.** No silent assumptions, no fabricated content. Forced to proceed past an unknown → mark it `PLACEHOLDER — уточнить у пользователя`.
- **Cap: 5–8 questions.** Record answers verbatim — Phase 2 synthesizes from this transcript.

### Phase 2 — Spec

Follow `to-spec` (read its file — see Phase 0) on the grilling transcript. Three overrides:

- **No new questions to the user** — anything still open becomes a PLACEHOLDER in the spec, not an interview.
- **Skip the seams check.** `to-spec` asks the user to confirm the test seams; a vibecoder cannot judge seams. Pick them yourself and record them under Implementation Decisions.
- **No git repo yet → `git init` here**, then commit the spec. It is the user's first rollback point, and every later commit needs a repo to land in. A vibecoder's machine usually has no git identity, and `git commit` hard-fails on that — check `git config user.email` and, if empty, set a repo-local one (`git config user.name` / `user.email`, never `--global`) before the first commit.

### Phase 3 — Tickets

Follow `to-tickets` (read its file), with two overrides. First, **one ticket must deliver the run instruction** — the plain-language «как это запустить» file. A vibecoder with working code and no idea how to start it has nothing. Second, **skip the user quiz** — a vibecoder cannot judge granularity or blocking edges. Validate the breakdown yourself against the skill's own slicing rules, then show the user **one screen of plain-language lines** (what each ticket delivers, no technical detail) with a default: «Запускаю через 60 секунд, если не скажешь стоп». Do not wait for explicit approval — waiting is the failure mode this skill exists to remove.

### Phase 4 — Implement (subagent per ticket)

**One ticket = one subagent = one fresh context.** Never two tickets in one context — context pollution is exactly what breaks naive vibecoding.

The subagent **cannot load `implement` itself**, so hand it everything inline: the ticket body, the relevant spec sections, paths to existing code, and the body of `implement`'s `SKILL.md` — plus a note that the `/tdd` and `/code-review` it refers to *are* model-invocable, so the subagent should reach them with the Skill tool.

- **One commit per ticket** — commits are the user's rollback points. A subagent commits **only its own files by path**, never `git add -A`: parallel siblings have half-written files in the tree. On `index.lock` — wait and retry, don't force.
- Unblocked tickets may run in parallel **only when they touch disjoint files**; same files → serialize. `to-tickets` deliberately keeps file paths out of ticket bodies, so **you** hold the ownership map: hand each subagent the exact list of files it owns and the explicit instruction not to touch anything else. Without that map, «disjoint» is a guess and parallel agents overwrite each other.
- After each ticket, report **one plain-language line** («Можно загрузить клиентов из файла — 3 из 8 готово»). No diffs, no jargon.
- Ticket failed → retry **once** in a fresh context with the error attached. Second failure → stop, tell the user in plain language what is blocking and what you need.

### Phase 5 — Finish

Full test suite once, then a final report in the user's language: what was built and the exact command to run it; what was NOT built (the spec's Out of Scope list); open items — placeholders, keys to add, manual steps left.

## Rationalizations — STOP

| Excuse | Reality |
|--------|---------|
| «Скилл называется `/to-spec` — просто вызову его» | `to-spec`, `to-tickets`, `implement` закрыты для модели (`disable-model-invocation`). Вызов не сработает — читай их `SKILL.md` и исполняй тело дословно. |
| «Пользователь сказал не задавать вопросов» | Он сказал не задавать ЛИШНИХ. Решающие вопросы — часть работы, не обсуждение процесса. |
| «KISS — просто собери» | Простой результат даёт порядок, а не пропуск этапов. Без спеки каждая правка — «а я имел в виду другое». |
| «Сделаю заглушку, уточнит потом» | Блокирующие неизвестные (оплата, ключи, хостинг) решаются в grilling, до билда. |
| «И так понятно, что делать» | Понятно тебе — не зафиксировано. Спека — единственная точка сверки. |
| «Быстрее всё сделать в одном контексте» | Быстрее в первый час. Дальше модель ходит кругами и ломает работавшее. |

## Red Flags — start the phase over

- Writing code before the spec exists.
- Running a phase from memory because the Skill tool would not load `to-spec`, `to-tickets` or `implement`.
- Asking the user to review tickets, granularity, seams, or code.
- Two tickets in one subagent context.
- Parallel subagents editing the same files.
- Silent assumption not marked as PLACEHOLDER.
- Payment/keys/hosting first mentioned at the finish line.

**Violating the letter of these rules is violating their spirit.**
