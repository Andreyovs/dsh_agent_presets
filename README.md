# dsh_agent_presets

Пресеты агентов для [DeepSeek Harness (DSH)](https://www.npmjs.com/package/@deepseek-ai/dsh) в формате, совместимом с [agent-skills](https://github.com/Andreyovs/agent-skills).

## Пресеты

| Пресет | Роль | Описание |
|--------|------|----------|
| `dev-lead` | Руководитель разработки (Development Lead) | Staff-инженер: архитектура, декомпозиция задач, качество кода, технические риски, менторство |
| `project-lead` | Руководитель проекта (Project Lead) | Старший лидер delivery: планирование, статусы, риски и блокеры, контроль скоупа, коммуникация со стейкхолдерами |

Общие атрибуты обоих пресетов:

- **Уровень достоверности 90%** — утверждать как факт только то, в чём уверенность ≥ 90% (проверено в коде/документах/данных); иначе — проверить или пометить как предположение с уровнем уверенности. Выдумывать факты, API и цифры — запрещено.
- **Уровень навыков — супер-профессионал** — persona на уровне staff/senior-эксперта, высочайший стандарт качества.

## Состав

```
presets/
  dev-lead.patch.yml      # готовая вставка в patch профиля DSH (полный пресет: persona + все инструменты standard)
  project-lead.patch.yml  # то же для руководителя проекта
agents/
  dev-lead.md             # persona в формате agents/*.md из agent-skills (frontmatter + role + rules + Composition)
  project-lead.md         # то же для project-lead
```

`presets/*.patch.yml` — полные декларации `@deepseek-ai/dsh-agent-preset` на базе композиции пресета `standard` (bash, fs, web, subagents, workflow, plan mode, compaction и т.д.), унаследованной из `dsh-web-app`, с заменённой persona. `agents/*.md` — те же роли в переносимом формате agent-skills (можно класть в `agents/` плагина agent-skills или использовать как system prompt).

## Установка в DSH

Пресеты добавляются **patch'ем профиля** (live-reload, рестарт не нужен). Скопируйте содержимое обоих файлов в конец patch-файла профиля, например `~/.dsh/profiles/web/cordis.patch.yml`:

```bash
cat presets/dev-lead.patch.yml presets/project-lead.patch.yml >> ~/.dsh/profiles/web/cordis.patch.yml
```

После этого в GUI DSH в списке agent presets появятся `Development Lead (руководитель разработки)` и `Project Lead (руководитель проекта)` — выберите их в новой сессии.

Откат — удалить добавленные блоки (маркеры `# Agent preset dev-lead:` / `# Agent preset project-lead:`).

> Пресеты `order: 5` и `order: 6` — после встроенных (standard=1, ptc=2, minimal=3, cordis=4). Если id `dev-lead`/`project-lead` уже заняты в вашем профиле, поменяйте `config.id` в обоих файлах.

## Установка в agent-skills (опционально)

```bash
cp agents/*.md <path>/agent-skills/agents/
```

После этого персоны доступны как подагенты (`subagent_type: dev-lead` / `project-lead`) и могут добавляться в fan-out команды по правилам `docs/agents.md` того репозитория.
