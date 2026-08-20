[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)

<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# 🧠 Awesome AI Rules

<div align="center">

### Hören Sie auf, sich bei der KI zu wiederholen. Schreiben Sie die Regeln einmal.

**20 Produktions-Konfigurationsvorlagen für Cursor, Claude Code, Kimi Code, Copilot und mehr.**

Regeldatei kopieren → Ihr KI-Assistent schreibt sofort besseren Code.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## Vorher / Nachher

**Ohne Regeln** — KI generiert generischen, inkonsistenten Code:
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// No types. No error handling. No loading state. No tests.
```

**Mit `react.md` Regeln** — KI generiert produktionsreifen Code:
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

## Schnellstart

```bash
# Wählen Sie Ihr Tool, kopieren Sie die Regel
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**Mehrere Regeln kombinieren:**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 20 Regeln — 3 Kategorien

### 🛠️ Frameworks (7)

| Regel | Ziele |
|-------|-------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 Szenarien (7)

| Regel | Abdeckung |
|-------|-----------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [Microservices](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [Mobile App](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [Data Science](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API Design](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [Testing](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [Performance](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 Tools (6)

| Regel | Tool |
|-------|------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## So funktionieren Regeln

Jede Regeldatei folgt einer Standardstruktur:

```
Core Principles    →  3–5 nicht verhandelbare Regeln
Code Style         →  Benennung, Dateistruktur, Formatierung
Proven Patterns    →  Framework-spezifische Muster mit Codebeispielen
Common Pitfalls    →  ❌ Anti-Patterns → ✅ Korrekte Patterns
Dependencies       →  Empfohlene Bibliotheken
Project-Specific   →  Ausfüllbare Sektion für Ihr Projekt
```

Alle Regeln haben **Versionshinweise**, damit Sie wissen, welche Framework-Version sie anvisieren.

---

## Begleiter

| Tool | Zweck |
|------|-------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — Bewerten Sie die KI-Bereitschaft Ihres Projekts |

---

## FAQ

**Welche Tools unterstützen benutzerdefinierte Regeln?**
Cursor (`.cursorrules`), Claude Code (`CLAUDE.md`), Kimi Code (`AGENTS.md`), GitHub Copilot (`.github/copilot-instructions.md`), Windsurf (`.windsurfrules`), Cline (`.clinerules`).

**Wie viele Regeln sollte ich verwenden?**
1–3 Framework-Regeln + 1–2 Szenario-Regeln. Nicht alles auf einmal — konzentrieren Sie sich auf Ihren Stack.

**Sind diese Regeln frameworkversionsspezifisch?**
Ja. Jede Datei hat eine `Last updated: 2026 | Targets: [version]`-Anmerkung.

**Kann ich beitragen?**
Ja! Siehe [CONTRIBUTING.md](CONTRIBUTING.md). Wir brauchen besonders Regeln für neue Frameworks und Tools.

---

## Siehe auch

| Projekt | Beschreibung |
|---------|-------------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — KI-Bereitschaft bewerten |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — KI schreibt Commit-Nachrichten |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | MCP-Server für Cursor, Claude Code und Kimi Code |

## Mitwirken

Siehe [CONTRIBUTING.md](CONTRIBUTING.md).

## Lizenz

[MIT](LICENSE)

---

<div align="center">

**Stern vergeben, wenn es Zeit gespart hat ⭐**

</div>

---
