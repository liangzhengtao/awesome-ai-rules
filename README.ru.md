[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)

<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# 🧠 Awesome AI Rules

<div align="center">

### Хватит повторять одно и то же ИИ. Напишите правила один раз.

**20 шаблонов продакшен-конфигураций для Cursor, Claude Code, Kimi Code, Copilot и других.**

Скопируйте файл правил → ваш ИИ-ассистент мгновенно пишет лучший код.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## До / После

**Без правил** — ИИ генерирует общий, несогласованный код:
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// No types. No error handling. No loading state. No tests.
```

**С правилами `react.md`** — ИИ генерирует продакшен-код:
```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['user'],
  queryFn: () => api.getUser(),
});
if (isLoading) return <Skeleton />;
if (error) return <ErrorBoundary error={error} />;
return <UserProfile user={data} />;
// Typed. Tested. Accessible. Your conventions.
```

---

## Быстрый старт

```bash
# Выберите инструмент, скопируйте правило
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**Комбинирование нескольких правил:**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 20 правил — 3 категории

### 🛠️ Фреймворки (7)

| Правило | Цели |
|---------|------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 Сценарии (7)

| Правило | Что покрывает |
|---------|---------------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [Microservices](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [Mobile App](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [Data Science](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API Design](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [Testing](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [Performance](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 Инструменты (6)

| Правило | Инструмент |
|---------|-----------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## Как работают правила

Каждый файл правил следует стандартной структуре:

```
Core Principles    →  3–5 ключевых правил
Code Style         →  Именование, структура файлов, форматирование
Proven Patterns    →  Специфичные для фреймворка паттерны с примерами кода
Common Pitfalls    →  ❌ Антипаттерны → ✅ Правильные паттерны
Dependencies       →  Рекомендуемые библиотеки
Project-Specific   →  Заполняемая секция для вашего проекта
```

Все правила имеют **аннотации версий**, чтобы вы знали, какую версию фреймворка они нацелены.

---

## Сопутствующие инструменты

| Инструмент | Назначение |
|-----------|------------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — оцените готовность вашего проекта к ИИ |

---

## Часто задаваемые вопросы

**Какие инструменты поддерживают пользовательские правила?**
Cursor (`.cursorrules`), Claude Code (`CLAUDE.md`), Kimi Code (`AGENTS.md`), GitHub Copilot (`.github/copilot-instructions.md`), Windsurf (`.windsurfrules`), Cline (`.clinerules`).

**Сколько правил использовать?**
1–3 правила фреймворка + 1–2 правила сценария. Не добавляйте всё подряд — сосредоточьтесь на вашем стеке.

**Эти правила привязаны к версиям фреймворков?**
Да. Каждый файл содержит аннотацию `Last updated: 2026 | Targets: [version]`.

**Могу ли я внести вклад?**
Да! Смотрите [CONTRIBUTING.md](CONTRIBUTING.md). Особенно нужны правила для новых фреймворков и инструментов.

---

## Смотрите также

| Проект | Описание |
|--------|----------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — оцените готовность проекта к ИИ |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — ИИ пишет сообщения коммитов |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | MCP-серверы для Cursor, Claude Code и Kimi Code |

## Участие в проекте

См. [CONTRIBUTING.md](CONTRIBUTING.md).

## Лицензия

[MIT](LICENSE)

---

<div align="center">

**Поставьте звезду, если сэкономил ваше время ⭐**

</div>

---
