# Agent Development Guidelines

Development guidelines for AI coding agents working on Cloud Moltbot.

## Project Overview

Cloud Moltbot is a TypeScript project running on Cloudflare Workers + Containers. It uses Durable Objects with `@cloudflare/containers` to manage containerized AI assistant workloads.

**Tech Stack:** Cloudflare Workers, TypeScript (ES2024), pnpm (v10.28.2)

## Commands

```bash
pnpm install          # Install dependencies
pnpm dev              # Start local development server
pnpm deploy           # Deploy to Cloudflare
pnpm lint             # Run formatter (oxfmt) and linter (oxlint)
pnpm cf-typegen       # Regenerate Cloudflare type definitions
```

**Individual tools:**

```bash
npx oxfmt             # Format code only
npx oxlint            # Lint code only
```

**No test suite configured.**

## Project Structure

```
src/
├── index.ts          # Workers entry point (ExportedHandler)
└── container.ts      # AgentContainer class (extends Container)

wrangler.jsonc        # Wrangler configuration
tsconfig.json         # TypeScript configuration
worker-configuration.d.ts  # Auto-generated bindings (DO NOT EDIT)
```

## Code Style

### Formatting (oxfmt - configured in `.oxfmtrc.json`)

- **Single quotes**: `'string'` not `"string"`
- **No semicolons**: Omit trailing semicolons
- **Spaces for indentation**: Use spaces, not tabs
- **Line endings**: LF (Unix style)
- **Final newline**: Always insert

### TypeScript

- **Target**: ES2024, Module: ES2022, Bundler resolution
- **Strict mode**: Enabled
- **No emit**: TypeScript for type checking only

### Imports

```typescript
// Use named imports from cloudflare:workers
import { env } from 'cloudflare:workers'
import { Container } from '@cloudflare/containers'

// Avoid default exports except for main handler
export { AgentContainer } from './container'
```

### Naming Conventions

| Element    | Convention       | Example                    |
| ---------- | ---------------- | -------------------------- |
| Files      | kebab-case       | `my-file.ts`               |
| Classes    | PascalCase       | `AgentContainer`           |
| Functions  | camelCase        | `handleFetch`              |
| Env vars   | UPPER_SNAKE_CASE | `SERVER_PASSWORD`          |
| Interfaces | PascalCase       | `ContainerConfig` (no `I`) |

### Type Patterns

```typescript
const value = env.MY_VAR  // Cloudflare.Env type
export default { fetch } satisfies ExportedHandler<Cloudflare.Env>
async function handleFetch(request: Request): Promise<Response> { ... }
```

### Error Handling

```typescript
return new Response('Unauthorized', { status: 401 }) // HTTP errors
console.error('Critical failure:', error) // Errors
console.warn('WebSocket closed') // Warnings
console.info('Gateway connected') // Info

// Early returns for guard clauses
const authError = verifyBasicAuth(request)
if (authError) return authError
```

### Cloudflare Patterns

```typescript
// Durable Object Container
export class AgentContainer extends Container {
  sleepAfter = '10m'
  defaultPort = 6658
  override async onStart(): Promise<void> { /* init */ }
}

// Singleton pattern
const objectId = env.AGENT_CONTAINER.idFromName('singleton-id')
const container = env.AGENT_CONTAINER.get(objectId, { locationHint: 'wnam' })

// WebSocket handling
const res = await this.containerFetch('http://container/', {
  headers: { Upgrade: 'websocket' },
})
ws.accept()
ws.addEventListener('message', (msg) => { ... })
```

## Environment Variables

| Variable                                                               | Purpose                                |
| ---------------------------------------------------------------------- | -------------------------------------- |
| `SERVER_USERNAME`                                                      | Basic auth username                    |
| `SERVER_PASSWORD`                                                      | Basic auth password (empty = disabled) |
| `MOLTBOT_GATEWAY_TOKEN`                                                | Gateway access token                   |
| `S3_ENDPOINT`, `S3_BUCKET`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY` | S3 storage                             |

## Language Requirements

**All code content MUST be in English:**

- Git commit messages
- Code comments
- Console log messages
- Variable/function names

## Best Practices

1. **Keep handlers thin** - Delegate to focused functions
2. **Use early returns** - For auth and validation checks
3. **Avoid over-engineering** - Simple, readable code
4. **Comments explain why** - Not what the code does
5. **Regenerate types** - Run `pnpm cf-typegen` after binding changes
6. **Test locally** - Use `pnpm dev` before deploying

## Common Tasks

**Adding env variable:** Add to `wrangler.jsonc` → Run `pnpm cf-typegen` → Access via `env.NEW_VAR`

**Container behavior:** Edit `src/container.ts` - key methods: `onStart()`, `sleepAfter`

**Worker handler:** Edit `src/index.ts`:

```typescript
export default { fetch: handleFetch } satisfies ExportedHandler<Cloudflare.Env>
```
