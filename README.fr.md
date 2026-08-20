[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md)
n<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# 🧠 Awesome AI Rules

<div align="center">

### Arrêtez de vous répéter face à l'IA. Écrivez les règles une fois.

**20 modèles de configuration prêts à l'emploi pour Cursor, Claude Code, Kimi Code, Copilot et plus.**

Copiez-collez un fichier de règles → votre assistant IA écrit immédiatement un meilleur code.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## Avant / Après

**Sans règles** — L'IA génère du code générique et incohérent :
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// Pas de types. Pas de gestion d'erreurs. Pas d'état de chargement. Pas de tests.
```

**Avec les règles `react.md`** — L'IA génère du code de production :
```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['user'],
  queryFn: () => api.getUser(),
});
if (isLoading) return <Skeleton />;
if (error) return <ErrorBoundary error={error} />;
return <UserProfile user={data} />;
// Typé. Testé. Accessible. Vos conventions.
```

---

## Démarrage rapide

```bash
# Choisissez votre outil, copiez la règle
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**Combiner plusieurs règles :**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 20 règles — 3 catégories

### 🛠️ Frameworks (7)

| Règle | Cibles |
|-------|--------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 Scénarios (7)

| Règle | Couverture |
|-------|-----------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [Microservices](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [Mobile App](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [Data Science](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API Design](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [Testing](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [Performance](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 Outils (6)

| Règle | Outil |
|-------|-------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## Comment fonctionnent les règles

Chaque fichier de règles suit une structure standard :

```
Principes fondamentaux  →  3–5 règles non négociables
Style de code           →  Nommage, structure des fichiers, formatage
Patterns éprouvés       →  Patterns spécifiques au framework avec exemples de code
Pièges courants         →  ❌ Anti-patterns → ✅ Patterns corrects
Dépendances             →  Bibliothèques recommandées
Spécifique au projet    →  Section à remplir pour votre projet
```

Toutes les règles comportent des **annotations de version** pour connaître la version du framework ciblée.

---

## Outils compagnons

| Outil | Objectif |
|-------|----------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — notez la préparation IA de votre projet |

---

## FAQ

**Quels outils prennent en charge les règles personnalisées ?**
Cursor (`.cursorrules`), Claude Code (`CLAUDE.md`), Kimi Code (`AGENTS.md`), GitHub Copilot (`.github/copilot-instructions.md`), Windsurf (`.windsurfrules`), Cline (`.clinerules`).

**Combien de règles devrais-je utiliser ?**
1–3 règles de framework + 1–2 règles de scénario. N'en mettez pas trop — concentrez-vous sur votre stack.

**Ces règles sont-elles spécifiques à une version de framework ?**
Oui. Chaque fichier comporte une annotation `Last updated: 2026 | Targets: [version]`.

**Puis-je contribuer ?**
Oui ! Consultez [CONTRIBUTING.md](CONTRIBUTING.md). Nous avons particulièrement besoin de règles pour les nouveaux frameworks et outils.

---

## Voir aussi

| Projet | Description |
|--------|-------------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — Notez la préparation IA de votre projet |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — L'IA écrit vos messages de commit |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | Serveurs MCP pour Cursor, Claude Code et Kimi Code |

## Contribuer

Voir [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

[MIT](LICENSE)

---

<div align="center">

**Donnez une étoile si ce projet vous a fait gagner du temps ⭐**

</div>

---
