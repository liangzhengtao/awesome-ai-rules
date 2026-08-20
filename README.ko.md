[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)

<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# 🧠 Awesome AI Rules

<div align="center">

### AI에게 같은 말을 반복하지 마세요. 규칙을 한 번만 작성하세요.

**Cursor, Claude Code, Kimi Code, Copilot 등용 프로덕션 설정 템플릿 20개.**

규칙 파일을 복사-붙여넣기 하면 → AI 어시스턴트가 즉시 더 나은 코드를 작성합니다.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## Before / After

**규칙 없이** — AI가 범용적이고 일관성 없는 코드를 생성:
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// No types. No error handling. No loading state. No tests.
```

**`react.md` 규칙 사용 시** — AI가 프로덕션 코드를 생성:
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

## 빠른 시작

```bash
# 도구를 선택하고, 규칙을 복사
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**여러 규칙을 결합:**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 규칙 20개 — 3개 카테고리

### 🛠️ 프레임워크 (7)

| 규칙 | 대상 |
|------|------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 시나리오 (7)

| 규칙 | 포함 내용 |
|------|----------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [Microservices](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [Mobile App](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [Data Science](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API Design](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [Testing](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [Performance](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 도구 (6)

| 규칙 | 도구 |
|------|------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## 규칙의 작동 방식

각 규칙 파일은 표준 구조를 따릅니다:

```
Core Principles    →  3~5개의 핵심 규칙
Code Style         →  네이밍, 파일 구조, 포맷팅
Proven Patterns    →  프레임워크별 패턴과 코드 예제
Common Pitfalls    →  ❌ 안티패턴 → ✅ 올바른 패턴
Dependencies       →  추천 라이브러리
Project-Specific   →  프로젝트용 작성 섹션
```

모든 규칙에는 **버전 주석**이 있어 어떤 프레임워크 버전을 대상으로 하는지 알 수 있습니다.

---

## 동반 도구

| 도구 | 용도 |
|------|------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — 프로젝트의 AI 준비도 평가 |

---

## FAQ

**어떤 도구가 커스텀 규칙을 지원하나요?**
Cursor (`.cursorrules`), Claude Code (`CLAUDE.md`), Kimi Code (`AGENTS.md`), GitHub Copilot (`.github/copilot-instructions.md`), Windsurf (`.windsurfrules`), Cline (`.clinerules`).

**규칙을 몇 개 사용해야 하나요?**
프레임워크 규칙 1~3개 + 시나리오 규칙 1~2개. 모든 것을 넣지 마세요 — 스택에 집중하세요.

**이 규칙들은 프레임워크 버전에 특화되어 있나요?**
네. 각 파일에는 `Last updated: 2026 | Targets: [version]` 주석이 있습니다.

**기여할 수 있나요?**
네! [CONTRIBUTING.md](CONTRIBUTING.md)를 참조하세요. 특히 새로운 프레임워크와 도구에 대한 규칙이 필요합니다.

---

## 관련 프로젝트

| 프로젝트 | 설명 |
|---------|------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — 프로젝트의 AI 준비도 평가 |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — AI가 커밋 메시지를 작성 |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | Cursor, Claude Code, Kimi Code용 MCP 서버 |

## 기여하기

[CONTRIBUTING.md](CONTRIBUTING.md)를 참조하세요.

## 라이선스

[MIT](LICENSE)

---

<div align="center">

**시간을 절약해 주었다면 스타를 눌러주세요 ⭐**

</div>

---
