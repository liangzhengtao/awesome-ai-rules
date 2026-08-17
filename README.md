<div align="center">

[English](#english) | [中文](#中文)

</div>

---

<a name="english"></a>
# 🧠 Awesome AI Rules

<div align="center">

### Stop repeating yourself to AI. Write the rules once.

**20 production-ready configuration templates for Cursor, Claude Code, Kimi Code, Copilot, and more.**

Copy-paste a rule file → your AI assistant instantly writes better code.

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## Before / After

**Without rules** — AI generates generic, inconsistent code:
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// No types. No error handling. No loading state. No tests.
```

**With `react.md` rules** — AI generates production-ready code:
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

## Quick Start

```bash
# Pick your tool, copy the rule
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**Stack multiple rules:**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 20 Rules — 3 Categories

### 🛠️ Frameworks (7)

| Rule | Targets |
|------|---------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 Scenarios (7)

| Rule | What It Covers |
|------|----------------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [Microservices](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [Mobile App](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [Data Science](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API Design](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [Testing](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [Performance](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 Tools (6)

| Rule | Tool |
|------|------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## How Rules Work

Each rule file follows a standard structure:

```
Core Principles    →  3–5 non-negotiable rules
Code Style         →  Naming, file structure, formatting
Best Practices     →  Framework-specific patterns with code examples
Common Pitfalls    →  ❌ Anti-patterns → ✅ Correct patterns
Dependencies       →  Recommended libraries
Project-Specific   →  Fill-in section for your project
```

All rules have **version annotations** so you know which framework version they target.

---

## Companions

| Tool | Purpose |
|------|---------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — score your project's AI-readiness |

---

## FAQ

**Which tools support custom rules?**
Cursor (`.cursorrules`), Claude Code (`CLAUDE.md`), Kimi Code (`AGENTS.md`), GitHub Copilot (`.github/copilot-instructions.md`), Windsurf (`.windsurfrules`), Cline (`.clinerules`).

**How many rules should I use?**
1–3 framework rules + 1–2 scenario rules. Don't dump everything — keep it focused on your stack.

**Are these rules framework-version-specific?**
Yes. Each file has a `Last updated: 2026 | Targets: [version]` annotation.

**Can I contribute?**
Yes! See [CONTRIBUTING.md](CONTRIBUTING.md). We especially need rules for new frameworks and tools.

---

## See Also

| Project | Description |
|---------|-------------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — Score your project's AI-readiness |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — AI writes your commit messages |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | MCP servers for Cursor, Claude Code, and Kimi Code |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](LICENSE)

---

<div align="center">

**Star this repo if it saved you time ⭐**

</div>

---

<a name="中文"></a>
# 🧠 Awesome AI Rules — 中文版

<div align="center">

### 别再对 AI 重复解释了。把规则写一次就好。

**20 个开箱即用的配置模板，覆盖 Cursor、Claude Code、Kimi Code、Copilot 等主流 AI 编程工具。**

复制粘贴一个规则文件 → 你的 AI 助手立刻写出更好的代码。

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## 没有规则 vs 有规则

**没有规则** — AI 生成的代码泛用、不一致：
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// 没有类型。没有错误处理。没有加载状态。没有测试。
```

**使用 `react.md` 规则** — AI 生成生产级代码：
```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['user'],
  queryFn: () => api.getUser(),
});
if (isLoading) return <Skeleton />;
if (error) return <ErrorBoundary error={error} />;
return <UserProfile user={data} />;
// 有类型。有测试。可访问。符合你的规范。
```

---

## 快速开始

```bash
# 选择你的工具，复制规则文件
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**叠加多条规则：**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 20 条规则 — 3 大分类

### 🛠️ 框架规则（7 条）

| 规则 | 适用版本 |
|------|---------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 场景规则（7 条）

| 规则 | 覆盖内容 |
|------|----------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [微服务](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [移动应用](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [数据科学](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API 设计](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [测试](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [性能优化](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 工具规则（6 条）

| 规则 | 工具 |
|------|------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## 规则的工作原理

每条规则文件遵循统一的结构：

```
核心原则       →  3–5 条不可妥协的规则
代码风格       →  命名、文件结构、格式化
最佳实践       →  框架相关的代码模式和示例
常见陷阱       →  ❌ 反模式 → ✅ 正确写法
依赖推荐       →  推荐的第三方库
项目定制       →  可填写的项目专属部分
```

所有规则都带有**版本标注**，方便你确认适用的框架版本。

---

## 配套工具

| 工具 | 用途 |
|------|------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — 为你的项目 AI 友好度打分 |

---

## 常见问题

**哪些工具支持自定义规则？**
Cursor (`.cursorrules`)、Claude Code (`CLAUDE.md`)、Kimi Code (`AGENTS.md`)、GitHub Copilot (`.github/copilot-instructions.md`)、Windsurf (`.windsurfrules`)、Cline (`.clinerules`)。

**应该用多少条规则？**
1–3 条框架规则 + 1–2 条场景规则。不要全部堆上去，聚焦你当前的技术栈即可。

**这些规则是否针对特定框架版本？**
是的。每个文件都有 `Last updated: 2026 | Targets: [版本]` 的标注。

**可以参与贡献吗？**
当然！详见 [CONTRIBUTING.md](CONTRIBUTING.md)。我们尤其需要新框架和新工具的规则。

---

## 相关项目

| 项目 | 说明 |
|------|------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — 为你的项目 AI 友好度打分 |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — AI 帮你写 commit 信息 |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | 适用于 Cursor、Claude Code 和 Kimi Code 的 MCP 服务器 |

## 参与贡献

详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 许可证

[MIT](LICENSE)

---

<div align="center">

**如果这个项目帮到了你，请点个 Star ⭐**

</div>
