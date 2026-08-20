[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)

<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# 🧠 Awesome AI Rules

<div align="center">

### Pare de se repetir para a IA. Escreva as regras uma vez.

**20 templates de configuração de produção para Cursor, Claude Code, Kimi Code, Copilot e mais.**

Copie e cole um arquivo de regra → seu assistente de IA imediatamente escreve código melhor.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## Antes / Depois

**Sem regras** — IA gera código genérico e inconsistente:
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// No types. No error handling. No loading state. No tests.
```

**Com regras `react.md`** — IA gera código de produção:
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

## Início Rápido

```bash
# Escolha sua ferramenta, copie a regra
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**Empilhe múltiplas regras:**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 20 Regras — 3 Categorias

### 🛠️ Frameworks (7)

| Regra | Alvos |
|-------|-------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 Cenários (7)

| Regra | O que cobre |
|-------|-------------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [Microservices](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [Mobile App](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [Data Science](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API Design](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [Testing](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [Performance](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 Ferramentas (6)

| Regra | Ferramenta |
|-------|-----------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## Como as Regras Funcionam

Cada arquivo de regra segue uma estrutura padrão:

```
Core Principles    →  3–5 regras inegociáveis
Code Style         →  Nomenclatura, estrutura de arquivos, formatação
Proven Patterns    →  Padrões específicos do framework com exemplos de código
Common Pitfalls    →  ❌ Anti-padrões → ✅ Padrões corretos
Dependencies       →  Bibliotecas recomendadas
Project-Specific   →  Seção preenchível para seu projeto
```

Todas as regras possuem **anotações de versão** para saber qual versão do framework elas visam.

---

## Companheiros

| Ferramenta | Propósito |
|-----------|-----------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — avalie a preparação do seu projeto para IA |

---

## FAQ

**Quais ferramentas suportam regras personalizadas?**
Cursor (`.cursorrules`), Claude Code (`CLAUDE.md`), Kimi Code (`AGENTS.md`), GitHub Copilot (`.github/copilot-instructions.md`), Windsurf (`.windsurfrules`), Cline (`.clinerules`).

**Quantas regras devo usar?**
1–3 regras de framework + 1–2 regras de cenário. Não jogue tudo — foque na sua stack.

**Essas regras são específicas de versão do framework?**
Sim. Cada arquivo tem uma anotação `Last updated: 2026 | Targets: [version]`.

**Posso contribuir?**
Sim! Consulte [CONTRIBUTING.md](CONTRIBUTING.md). Precisamos especialmente de regras para novos frameworks e ferramentas.

---

## Veja Também

| Projeto | Descrição |
|---------|-----------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — Avalie a preparação do seu projeto para IA |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — IA escreve suas mensagens de commit |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | Servidores MCP para Cursor, Claude Code e Kimi Code |

## Contribuição

Consulte [CONTRIBUTING.md](CONTRIBUTING.md).

## Licença

[MIT](LICENSE)

---

<div align="center">

**Dê uma estrela se economizou seu tempo ⭐**

</div>

---
