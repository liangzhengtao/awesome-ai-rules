[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)

<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# 🧠 Awesome AI Rules

<div align="center">

### توقف عن تكرار نفسك للذكاء الاصطناعي. اكتب القواعد مرة واحدة.

**20 قالب إعدادات إنتاجية لـ Cursor و Claude Code و Kimi Code و Copilot والمزيد.**

انسخ ملف القواعد والصقه ← مساعدك الذكي يكتب كوداً أفضل فوراً.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## قبل / بعد

**بدون قواعد** — يُولّد الذكاء الاصطناعي كوداً عامياً وغير متسق:
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// No types. No error handling. No loading state. No tests.
```

**مع قواعد `react.md`** — يُولّد الذكاء الاصطناعي كوداً جاهزاً للإنتاج:
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

## البدء السريع

```bash
# اختر أداتك، انسخ القاعدة
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**تكدس عدة قواعد:**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 20 قاعدة — 3 فئات

### 🛠️ أُطُم العمل (7)

| القاعدة | الأهداف |
|---------|---------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 السيناريوهات (7)

| القاعدة | ما تغطيه |
|---------|----------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [Microservices](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [Mobile App](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [Data Science](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API Design](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [Testing](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [Performance](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 الأدوات (6)

| القاعدة | الأداة |
|---------|--------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## كيف تعمل القواعد

كل ملف قواعد يتبع هيكل قياسي:

```
Core Principles    →  3–5 قواعد لا تُناقَش
Code Style         →  التسمية، هيكل الملفات، التنسيق
Proven Patterns    →  أنماط خاصة بالإطار مع أمثلة الكود
Common Pitfalls    →  ❌ أنماط معاكسة → ✅ أنماط صحيحة
Dependencies       →  مكتبات موصى بها
Project-Specific   →  قسم مخصص لمشروعك
```

جميع القواعد تحتوي **تعليقات إصدار** حتى تعرف أي إصدار إطار تستهدفه.

---

## أدوات مساعدة

| الأداة | الغرض |
|--------|-------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — قيّم جاهزية مشروعك للذكاء الاصطناعي |

---

## أسئلة شائعة

**أي الأدوات تدعم القواعد المخصصة؟**
Cursor (`.cursorrules`)، Claude Code (`CLAUDE.md`)، Kimi Code (`AGENTS.md`)، GitHub Copilot (`.github/copilot-instructions.md`)، Windsurf (`.windsurfrules`)، Cline (`.clinerules`).

**كم عدد القواعد التي يجب استخدامها؟**
1-3 قواعد إطار + 1-2 قواعد سيناريو. لا تدرج كل شيء — ركز على تقنياتك.

**هل هذه القواعد خاصة بإصدارات الأُطُم؟**
نعم. كل ملف يحتوي تعليق `Last updated: 2026 | Targets: [version]`.

**هل يمكنني المساهمة؟**
نعم! انظر [CONTRIBUTING.md](CONTRIBUTING.md). نحتاج بشكل خاص قواعد لأُطُم وأدوات جديدة.

---

## انظر أيضاً

| المشروع | الوصف |
|---------|-------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — قيّم جاهزية مشروعك للذكاء الاصطناعي |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — الذكاء الاصطناعي يكتب رسائل الارتباط |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | خوادم MCP لـ Cursor و Claude Code و Kimi Code |

## المساهمة

انظر [CONTRIBUTING.md](CONTRIBUTING.md).

## الترخيص

[MIT](LICENSE)

---

<div align="center">

**قيّم بنجمة هذا المستودع إذا وفر لك الوقت ⭐**

</div>

---
