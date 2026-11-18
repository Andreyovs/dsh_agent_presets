# dsh_agent_presets

Пресеты агентов для [DeepSeek Harness (DSH)](https://www.npmjs.com/package/@deepseek-ai/dsh) в формате, совместимом с [agent-skills](https://github.com/Andreyovs/agent-skills).

## Пресеты

| Пресет | Роль | Описание |
|--------|------|----------|
| `dev-lead` | Руководитель разработки (Development Lead) | Staff-инженер: архитектура, декомпозиция задач, качество кода, технические риски, менторство |
| `project-lead` | Руководитель проекта (Project Lead) | Старший лидер delivery: планирование, статусы, риски и блокеры, контроль скоупа, коммуникация со стейкхолдерами |
| `pravo-lawyer` | Юрист (pravo.gov.ru) | Законодательство РФ: КонсультантПлюс (актуальные редакции, комментарии, практика), pravo.gov.ru и sudact.ru (браузер), базы знаний LightRAG, markitdown; правовые заключения, регламенты |
| `pravo-judge-assistant` | Помощник судьи (pravo.gov.ru) | Поиск применимых норм, анализ судебной практики (картотека дел — первичный источник; КС РФ и высшие суды), проекты судебных актов |
| `pravo-advocate` | Адвокат (pravo.gov.ru) | Изучение нормативной базы для защиты доверителей, процессуальные документы (ходатайства, жалобы, пояснения, отзывы), анализ правовых позиций судов |
| `pravo-notary` | Нотариус (pravo.gov.ru) | Проверка действующих норм при нотариальных действиях (доверенности, завещания, нотариальные соглашения, заверение), консультирование клиентов |

Общие атрибуты `dev-lead` и `project-lead`:

- **Уровень достоверности 90%** — утверждать как факт только то, в чём уверенность ≥ 90% (проверено в коде/документах/данных); иначе — проверить или пометить как предположение с уровнем уверенности. Выдумывать факты, API и цифры — запрещено.
- **Уровень навыков — супер-профессионал** — persona на уровне staff/senior-эксперта, высочайший стандарт качества.

Общий атрибут пресетов `pravo-*` — **дисциплина ссылок**: правовая позиция, которую нельзя сверить по подключённым источникам, — не правовая позиция; если источник не подтверждает — прямо сказать об этом, а не выдумывать.

### Пресеты `pravo-*`: юридический домен (РФ)

Четыре роли работают по одному рабочему конвейеру — «найти норму → проверить актуальность → понять практику → собрать позицию → скорректно сослаться» — и используют четыре источника:

1. **КонсультантПлюс** (MCP `consultant`) — актуальные систематизированные редакции (база LAW), комментарии, экспертные материалы (CJI), формы (PAP), судебная практика (ARB/AP), картотека дел;
2. **LightRAG** (MCP, графовый RAG по документам пользователя) — базы Основная/Законодательство/Договоры;
3. **Playwright** (MCP, headless Chrome) — pravo.gov.ru (официальные данные опубликования) и sudact.ru (картотека судебных актов);
4. **markitdown** (MCP) — документы (PDF/DOCX/…) в markdown, включая сканы (OCR).

КонсультантПлюс на машине одна разделяемая сессия (одна лицензия, запросы сериализуются): если вызов вернул `queued`, вызов повторяется, а не дублируется.

## Состав

```
presets/
  dev-lead.patch.yml            # готовая вставка в patch профиля DSH (полный пресет: persona + все инструменты standard)
  project-lead.patch.yml        # то же для руководителя проекта
  pravo-lawyer.patch.yml        # то же для Юриста
  pravo-judge-assistant.patch.yml
  pravo-advocate.patch.yml
  pravo-notary.patch.yml
agents/
  dev-lead.md                   # persona в формате agents/*.md из agent-skills (frontmatter + scope + rules + Composition)
  project-lead.md               # то же для project-lead
  pravo-lawyer.md               # то же для Юриста
  pravo-judge-assistant.md
  pravo-advocate.md
  pravo-notary.md
```

`presets/*.patch.yml` — полные декларации `@deepseek-ai/dsh-agent-preset` на базе композиции пресета `standard` (bash, fs, web, subagents, workflow, plan mode, compaction и т.д.), унаследованной из `dsh-web-app`, с заменённой persona. `agents/*.md` — те же роли в переносимом формате agent-skills (можно класть в `agents/` плагина agent-skills или использовать как system prompt).

## Установка в DSH

Пресеты добавляются **patch'ем профиля** (live-reload, рестарт не нужен). Скопируйте содержимое нужных файлов в конец patch-файла профиля, например `~/.dsh/profiles/web/cordis.patch.yml`:

```bash
cat presets/dev-lead.patch.yml presets/project-lead.patch.yml >> ~/.dsh/profiles/web/cordis.patch.yml
# либо юридические:
cat presets/pravo-*.patch.yml >> ~/.dsh/profiles/web/cordis.patch.yml
```

После этого в GUI DSH в списке agent presets появятся `Development Lead (руководитель разработки)`, `Project Lead (руководитель проекта)` и/или `Юрист (pravo.gov.ru)`, `Помощник судьи (pravo.gov.ru)`, `Адвокат (pravo.gov.ru)`, `Нотариус (pravo.gov.ru)` — выберите нужный в новой сессии.

Откат — удалить добавленные блоки (маркеры `# Agent preset <id>:` в начале каждого блока).

> Пресеты `order: 5`–`8` — после встроенных (standard=1, ptc=2, minimal=3, cordis=4). Если id `dev-lead`, `project-lead`, `pravo-lawyer`, `pravo-judge-assistant`, `pravo-advocate`, `pravo-notary` уже заняты в вашем профиле, поменяйте `config.id` в нужных файлах (и `preset-` id строки).

> Пресетам `pravo-*` на целевой машине нужны MCP-серверы `consultant`, `lightrag` (3 инстанции), `playwright` и `markitdown`; без них персона описывает источники, которых нет.

## Установка в agent-skills (опционально)

```bash
cp agents/*.md <path>/agent-skills/agents/
```

После этого персоны доступны как подагенты (`subagent_type: dev-lead` / `project-lead` / `pravo-lawyer` / …) и могут добавляться в fan-out команды по правилам `docs/agents.md` того репозитория.

## История изменений

- 2026-11-17 — добавлены юридические пресеты `pravo-lawyer`, `pravo-judge-assistant`, `pravo-advocate`, `pravo-notary` (4 источника: КонсультантПлюс, LightRAG, Playwright, markitdown). Из `dev-lead.patch.yml` / `project-lead.patch.yml` удалён `suffix: Your working directory is {{cwd}}.` — плагин `@deepseek-ai/dsh-persona` регистрирует только `provider` и `model`, неизвестная переменная `{{cwd}}` вызывала ошибку сборки prompt'а (`unknown prompt variable "{{cwd}}"`); рабочая директория модели уже передаётся runtime-контекстом.
