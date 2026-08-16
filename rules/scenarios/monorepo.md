# Monorepo AI 编程规则
> Last updated: 2026 | Targets: Turborepo 2.x / pnpm 9.x / Changesets

## 核心原则

- 使用包管理工具（pnpm workspaces / Turborepo / Nx）
- 遵循统一的代码规范和配置
- 合理拆分包和应用
- 使用 Changesets 管理版本
- 优化构建和缓存

## 代码风格

### 项目结构
```
monorepo/
├── apps/                    # 应用
│   ├── web/                 # 前端应用
│   ├── api/                 # 后端 API
│   └── admin/               # 管理后台
├── packages/                # 共享包
│   ├── ui/                  # UI 组件库
│   ├── utils/               # 工具函数
│   ├── config/              # 共享配置
│   ├── types/               # TypeScript 类型
│   └── database/            # 数据库模型
├── tools/                   # 构建工具
│   └── scripts/
├── .changeset/              # Changeset 配置
├── turbo.json               # Turborepo 配置
├── pnpm-workspace.yaml      # pnpm workspace 配置
└── package.json
```

### 命名规范
- 包名使用 `@org/package-name`：`@myorg/ui`, `@myorg/utils`
- 应用名使用 `app-name`：`web`, `api`, `admin`
- 内部依赖使用 `workspace:*`

## 最佳实践

### Workspace 配置

```yaml
# pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"
```

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"]
    },
    "lint": {
      "outputs": []
    },
    "typecheck": {
      "outputs": []
    }
  }
}
```

### 包配置

```json
// packages/ui/package.json
{
  "name": "@myorg/ui",
  "version": "1.0.0",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "scripts": {
    "build": "tsup",
    "test": "vitest",
    "lint": "eslint src --ext .ts,.tsx"
  },
  "dependencies": {
    "@myorg/utils": "workspace:*",
    "react": "^18.2.0"
  },
  "devDependencies": {
    "@myorg/config": "workspace:*",
    "typescript": "^5.0.0"
  }
}
```

### 共享配置

```typescript
// packages/config/eslint/index.js
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint', 'react'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'prettier',
  ],
  rules: {
    // 共享规则
  },
};
```

```typescript
// packages/config/tsconfig/base.json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}
```

### 依赖管理

```typescript
// ✅ 内部依赖使用 workspace:*
// apps/web/package.json
{
  "dependencies": {
    "@myorg/ui": "workspace:*",
    "@myorg/utils": "workspace:*"
  }
}

// ✅ 使用 pnpm catalog 统一版本
// pnpm-workspace.yaml
catalog:
  react: ^18.2.0
  typescript: ^5.0.0

// packages/ui/package.json
{
  "dependencies": {
    "react": "catalog:"
  }
}
```

### Changesets 版本管理

```bash
# 创建 changeset
pnpm changeset

# 版本升级
pnpm changeset version

# 发布
pnpm changeset publish
```

```markdown
<!-- .changeset/cool-feature.md -->
---
"@myorg/ui": minor
"@myorg/utils": patch
---

添加新的 Button 组件

- 支持多种变体
- 支持 loading 状态
- 修复 utils 中的类型问题
```

### 脚本管理

```json
// package.json
{
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "test": "turbo test",
    "lint": "turbo lint",
    "typecheck": "turbo typecheck",
    "format": "prettier --write \"**/*.{ts,tsx,md}\"",
    "changeset": "changeset",
    "version-packages": "changeset version",
    "ci:version": "turbo run build && changeset version",
    "ci:publish": "turbo run build && changeset publish"
  }
}
```

### CI/CD 配置

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm test
      - run: pnpm build
```

## 包拆分策略

### 按功能拆分
```
packages/
├── auth/              # 认证相关
├── notifications/     # 通知系统
├── analytics/         # 数据分析
└── payments/          # 支付功能
```

### 按层级拆分
```
packages/
├── ui/                # UI 组件
├── core/              # 核心逻辑
├── data/              # 数据层
└── config/            # 配置
```

### 按团队拆分
```
packages/
├── team-a/            # A 团队的共享代码
├── team-b/            # B 团队的共享代码
└── shared/            # 共享代码
```

## 性能优化

### 构建缓存
```json
// turbo.json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"],
      "cache": true
    }
  }
}
```

### 远程缓存
```json
// turbo.json
{
  "remoteCache": {
    "signature": true
  }
}
```

### 增量构建
```bash
# 只构建变更的包
turbo build --filter=...[HEAD^1]

# 构建特定包及其依赖
turbo build --filter=@myorg/ui...
```

## 常见陷阱

### ❌ 避免
```json
// ❌ 直接引用源码路径
{
  "dependencies": {
    "@myorg/ui": "../../packages/ui"  // 不要这样做
  }
}

// ❌ 循环依赖
// packages/a 依赖 packages/b
// packages/b 依赖 packages/a

// ❌ 不一致的版本
// 不同包使用不同版本的 react
```

### ✅ 推荐
```json
// ✅ 使用 workspace 协议
{
  "dependencies": {
    "@myorg/ui": "workspace:*"
  }
}

// ✅ 单向依赖
// packages/ui 不依赖任何业务包
// packages/utils 不依赖 packages/ui

// ✅ 统一版本
// 使用 pnpm catalog 或在根 package.json 中定义
```

## 依赖推荐

- **包管理**: pnpm workspaces
- **构建系统**: Turborepo / Nx
- **版本管理**: Changesets
- **代码规范**: ESLint + Prettier（共享配置）
- **测试**: Vitest（支持 workspace）

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- 包管理器：pnpm
- 构建系统：Turborepo / Nx
- 版本管理：Changesets
- 包范围：@myorg
```
