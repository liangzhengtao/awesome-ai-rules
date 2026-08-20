[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md)
n<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# 🧠 Awesome AI Rules

<div align="center">

### Deja de repetirle lo mismo a la IA. Escribe las reglas una sola vez.

**20 plantillas de configuración listas para producción para Cursor, Claude Code, Kimi Code, Copilot y más.**

Copia y pega un archivo de reglas → tu asistente de IA escribe mejor código al instante.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## Antes / Después

**Sin reglas** — La IA genera código genérico e inconsistente:
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// Sin tipos. Sin manejo de errores. Sin estado de carga. Sin tests.
```

**Con reglas `react.md`** — La IA genera código de producción:
```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['user'],
  queryFn: () => api.getUser(),
});
if (isLoading) return <Skeleton />;
if (error) return <ErrorBoundary error={error} />;
return <UserProfile user={data} />;
// Tipado. Testeado. Accesible. Tus convenciones.
```

---

## Inicio rápido

```bash
# Elige tu herramienta, copia la regla
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**Apilar múltiples reglas:**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 20 reglas — 3 categorías

### 🛠️ Frameworks (7)

| Regla | Objetivos |
|-------|----------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 Escenarios (7)

| Regla | Cobertura |
|-------|----------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [Microservices](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [Mobile App](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [Data Science](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API Design](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [Testing](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [Performance](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 Herramientas (6)

| Regla | Herramienta |
|-------|------------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## Cómo funcionan las reglas

Cada archivo de reglas sigue una estructura estándar:

```
Principios fundamentales  →  3–5 reglas innegociables
Estilo de código          →  Nomenclatura, estructura de archivos, formato
Patrones probados         →  Patrones específicos del framework con ejemplos de código
Errores comunes           →  ❌ Anti-patrones → ✅ Patrones correctos
Dependencias              →  Bibliotecas recomendadas
Específico del proyecto   →  Sección para completar con tu proyecto
```

Todas las reglas tienen **anotaciones de versión** para saber qué versión de framework apuntan.

---

## Herramientas complementarias

| Herramienta | Propósito |
|-------------|----------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — evalúa la preparación de IA de tu proyecto |

---

## FAQ

**¿Qué herramientas soportan reglas personalizadas?**
Cursor (`.cursorrules`), Claude Code (`CLAUDE.md`), Kimi Code (`AGENTS.md`), GitHub Copilot (`.github/copilot-instructions.md`), Windsurf (`.windsurfrules`), Cline (`.clinerules`).

**¿Cuántas reglas debería usar?**
1–3 reglas de framework + 1–2 reglas de escenario. No lo pongas todo — enfócate en tu stack actual.

**¿Estas reglas son específicas de una versión de framework?**
Sí. Cada archivo tiene una anotación `Last updated: 2026 | Targets: [versión]`.

**¿Puedo contribuir?**
¡Sí! Consulta [CONTRIBUTING.md](CONTRIBUTING.md). Especialmente necesitamos reglas para nuevos frameworks y herramientas.

---

## Ver también

| Proyecto | Descripción |
|----------|-------------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — Evalúa la preparación de IA de tu proyecto |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — La IA escribe tus mensajes de commit |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | Servidores MCP para Cursor, Claude Code y Kimi Code |

## Contribuir

Ver [CONTRIBUTING.md](CONTRIBUTING.md).

## Licencia

[MIT](LICENSE)

---

<div align="center">

**Dale una estrella si te ahorró tiempo ⭐**

</div>

---
