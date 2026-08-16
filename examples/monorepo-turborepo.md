# Example: Monorepo with Turborepo

This example shows how to configure rules for a monorepo project.

## Setup

```bash
# Copy scenario rules
cat rules/scenarios/monorepo.md > AGENTS.md

# Append framework rules based on your stack
cat rules/frameworks/typescript.md >> AGENTS.md
cat rules/frameworks/react.md >> AGENTS.md
```

## Combined Rules Preview

```markdown
# Monorepo AI Rules

## Core Principles
- Use pnpm workspaces for package management
- Use Turborepo for build orchestration
- Follow consistent code style across packages
- Use Changesets for version management

## Project Structure
```
monorepo/
├── apps/
│   ├── web/          # Next.js frontend
│   ├── api/          # Express backend
│   └── admin/        # Admin dashboard
├── packages/
│   ├── ui/           # Shared UI components
│   ├── utils/        # Shared utilities
│   ├── types/        # TypeScript types
│   └── config/       # Shared configs
├── turbo.json
└── pnpm-workspace.yaml
```

## Package Dependencies
- Use workspace:* for internal dependencies
- Keep shared packages independent
- Avoid circular dependencies

## Build Configuration
- turbo.json defines task dependencies
- Enable remote caching for CI
- Use proper output globs

## Version Management
- Use Changesets for versioning
- Create changeset files for each PR
- Automate releases with GitHub Actions
```

## Why This Works

Combined monorepo rules help your AI assistant:
1. Understand **workspace** structure
2. Use proper **dependency** management
3. Follow **Turborepo** conventions
4. Implement **versioning** correctly
5. Maintain **consistency** across packages
