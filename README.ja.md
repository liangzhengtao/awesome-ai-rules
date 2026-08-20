[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)
n<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# 🧠 Awesome AI Rules

<div align="center">

### AI に同じことを繰り返し言うのはもうやめよう。ルールは一度書けばいい。

**Cursor、Claude Code、Kimi Code、Copilot などに対応した本番環境向け設定テンプレート 20 選。**

ルールファイルをコピペするだけで → AI アシスタントがすぐに良いコードを書くようになります。

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rules](https://img.shields.io/badge/rules-20-blueviolet)](rules/)

</div>

---

## Before / After

**ルールなし** — AI は汎用的で一貫性のないコードを生成します：
```tsx
const user = await fetch('/api/user');
const data = await user.json();
return <div>{data.name}</div>;
// 型なし。エラー処理なし。ローディング状態なし。テストなし。
```

**`react.md` ルール適用後** — AI は本番環境対応のコードを生成します：
```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['user'],
  queryFn: () => api.getUser(),
});
if (isLoading) return <Skeleton />;
if (error) return <ErrorBoundary error={error} />;
return <UserProfile user={data} />;
// 型付き。テスト済み。アクセシビリティ対応。あなたの規約に準拠。
```

---

## クイックスタート

```bash
# ツールを選んでルールをコピー
cp rules/frameworks/react.md .cursorrules        # Cursor
cp rules/frameworks/react.md CLAUDE.md            # Claude Code
cp rules/frameworks/react.md AGENTS.md             # Kimi Code
```

**ルールの重ね合わせ：**
```bash
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules
cat rules/scenarios/testing.md >> .cursorrules
```

---

## 20 のルール — 3 つのカテゴリ

### 🛠️ フレームワーク（7 件）

| ルール | 対象 |
|--------|------|
| [React](rules/frameworks/react.md) | React 19+ / Server Components / React Compiler |
| [Next.js](rules/frameworks/nextjs.md) | Next.js 15+ / App Router / Async APIs |
| [Vue](rules/frameworks/vue.md) | Vue 3.4+ / Composition API / Pinia |
| [TypeScript](rules/frameworks/typescript.md) | TypeScript 5.x / strict mode |
| [Python FastAPI](rules/frameworks/python-fastapi.md) | Python 3.12+ / FastAPI 0.110+ / Pydantic V2 |
| [Rust](rules/frameworks/rust.md) | Rust 1.75+ / Axum 0.7+ / Tokio |
| [Go](rules/frameworks/go.md) | Go 1.22+ / Chi 5.x |

### 🎭 シナリオ（7 件）

| ルール | 対象範囲 |
|--------|---------|
| [Monorepo](rules/scenarios/monorepo.md) | Turborepo / pnpm workspaces / Changesets |
| [Microservices](rules/scenarios/microservices.md) | gRPC / Kafka / Kubernetes / Saga |
| [Mobile App](rules/scenarios/mobile-app.md) | React Native 0.74+ / Flutter 3.22+ |
| [Data Science](rules/scenarios/data-science.md) | pandas 2.x / scikit-learn / MLflow |
| [API Design](rules/scenarios/api-design.md) | REST / OpenAPI 3.1 / Pagination / Webhooks |
| [Testing](rules/scenarios/testing.md) | Vitest / Playwright / React Testing Library |
| [Performance](rules/scenarios/performance.md) | Core Web Vitals / Vite 5+ / Caching |

### 🔧 ツール（6 件）

| ルール | ツール |
|--------|--------|
| [Cursor](rules/tools/cursor.md) | Cursor 0.40+ |
| [Claude Code](rules/tools/claude-code.md) | Anthropic Claude Code |
| [Kimi Code](rules/tools/kimi-code.md) | Moonshot Kimi Code |
| [GitHub Copilot](rules/tools/copilot.md) | GitHub Copilot (multi-model) |
| [Windsurf](rules/tools/windsurf.md) | Codeium Windsurf |
| [Cline](rules/tools/cline.md) | Cline (VS Code) |

---

## ルールの仕組み

各ルールファイルは標準的な構造に従います：

```
コア原則       →  3〜5 つの譲れないルール
コードスタイル  →  命名、ファイル構造、フォーマット
実績あるパターン →  フレームワーク固有のパターンとコード例
よくある落とし穴 →  ❌ アンチパターン → ✅ 正しいパターン
依存関係       →  推奨ライブラリ
プロジェクト固有  →  プロジェクト用の入力欄
```

すべてのルールには**バージョン注記**があり、対象のフレームワークバージョンを確認できます。

---

## 関連ツール

| ツール | 用途 |
|--------|------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — プロジェクトの AI 対応度をスコアリング |

---

## よくある質問

**どのツールがカスタムルールに対応していますか？**
Cursor (`.cursorrules`)、Claude Code (`CLAUDE.md`)、Kimi Code (`AGENTS.md`)、GitHub Copilot (`.github/copilot-instructions.md`)、Windsurf (`.windsurfrules`)、Cline (`.clinerules`)。

**ルールはいくつ使うべきですか？**
フレームワークルール 1〜3 件 + シナリオルール 1〜2 件。すべて盛り込むのではなく、現在の技術スタックに集中してください。

**これらのルールは特定のフレームワークバージョン向けですか？**
はい。各ファイルに `Last updated: 2026 | Targets: [バージョン]` の注記があります。

**コントリビュートできますか？**
はい！[CONTRIBUTING.md](CONTRIBUTING.md) をご覧ください。特に新しいフレームワークやツールのルールが必要です。

---

## 関連プロジェクト

| プロジェクト | 説明 |
|-------------|------|
| [**vibe-check**](https://github.com/liangzhengtao/vibe-check) | `npx vibe-check` — プロジェクトの AI 対応度をスコアリング |
| [**ai-commit**](https://github.com/liangzhengtao/ai-commit) | `npx ai-commit` — AI が commit メッセージを書く |
| [**awesome-mcp-servers**](https://github.com/liangzhengtao/awesome-mcp-servers) | Cursor、Claude Code、Kimi Code 向け MCP サーバー |

## コントリビューション

[CONTRIBUTING.md](CONTRIBUTING.md) をご覧ください。

## ライセンス

[MIT](LICENSE)

---

<div align="center">

**時間の節約になったら Star をお願いします ⭐**

</div>

---
