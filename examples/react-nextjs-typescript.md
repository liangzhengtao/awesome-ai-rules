# Example: React + Next.js + TypeScript

This example shows how to combine multiple rules for a modern React project.

## Setup

Copy the combined rules to your project root:

```bash
# Copy framework rules
cat rules/frameworks/react.md > .cursorrules
cat rules/frameworks/nextjs.md >> .cursorrules
cat rules/frameworks/typescript.md >> .cursorrules

# Append scenario rules
cat rules/scenarios/testing.md >> .cursorrules
cat rules/scenarios/performance.md >> .cursorrules
```

## Combined Rules Preview

```markdown
# React + Next.js + TypeScript AI Rules

## Core Principles
- Use React 18+ features (Concurrent Features, Suspense)
- Prefer functional components and Hooks
- Use Next.js 14 App Router with Server Components
- Enable TypeScript strict mode

## Code Style
- Components: PascalCase (UserProfile, NavigationBar)
- Hooks: use prefix (useAuth, useFetch)
- Files: kebab-case (user-profile.tsx)

## Proven Patterns
- Server Components by default, Client Components only when needed
- Use fetch() in Server Components, React Query for client state
- Implement proper error boundaries
- Use Tailwind CSS for styling

## Testing
- Use Vitest + React Testing Library
- Test behavior, not implementation
- Use AAA pattern (Arrange-Act-Assert)
- Coverage target: 80%+

## Performance
- Use React.memo for expensive components
- Implement proper code splitting
- Optimize images with Next.js Image component
- Use Suspense for loading states
```

## Why This Works

By combining rules, your AI assistant understands:
1. **React** patterns and proven patterns
2. **Next.js** specific features (App Router, Server Components)
3. **TypeScript** type safety requirements
4. **Testing** standards
5. **Performance** optimization techniques

This gives the AI complete context to generate high-quality, consistent code.
