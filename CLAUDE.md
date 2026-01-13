---
description: Use Bun instead of Node.js, npm, pnpm, or vite.
globs: "*.ts, *.tsx, *.html, *.css, *.js, *.jsx, package.json"
alwaysApply: false
---

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Bun automatically loads .env, so don't use dotenv.

## APIs

- `Bun.serve()` supports WebSockets, HTTPS, and routes. Don't use `express`.
- `bun:sqlite` for SQLite. Don't use `better-sqlite3`.
- `Bun.redis` for Redis. Don't use `ioredis`.
- `Bun.sql` for Postgres. Don't use `pg` or `postgres.js`.
- `WebSocket` is built-in. Don't use `ws`.
- Prefer `Bun.file` over `node:fs`'s readFile/writeFile
- Bun.$`ls` instead of execa.

## Testing

Use `bun test` to run tests.

```ts#index.test.ts
import { test, expect } from "bun:test";

test("hello world", () => {
  expect(1).toBe(1);
});
```

## Frontend

Use HTML imports with `Bun.serve()`. Don't use `vite`. HTML imports fully support React, CSS, Tailwind.

Server:

```ts#index.ts
import index from "./index.html"

Bun.serve({
  routes: {
    "/": index,
    "/api/users/:id": {
      GET: (req) => {
        return new Response(JSON.stringify({ id: req.params.id }));
      },
    },
  },
  // optional websocket support
  websocket: {
    open: (ws) => {
      ws.send("Hello, world!");
    },
    message: (ws, message) => {
      ws.send(message);
    },
    close: (ws) => {
      // handle close
    }
  },
  development: {
    hmr: true,
    console: true,
  }
})
```

HTML files can import .tsx, .jsx or .js files directly and Bun's bundler will transpile & bundle automatically. `<link>` tags can point to stylesheets and Bun's CSS bundler will bundle.

```html#index.html
<html>
  <body>
    <h1>Hello, world!</h1>
    <script type="module" src="./frontend.tsx"></script>
  </body>
</html>
```

With the following `frontend.tsx`:

```tsx#frontend.tsx
import React from "react";

// import .css files directly and it works
import './index.css';

import { createRoot } from "react-dom/client";

const root = createRoot(document.body);

export default function Frontend() {
  return <h1>Hello, world!</h1>;
}

root.render(<Frontend />);
```

Then, run index.ts

```sh
bun --hot ./index.ts
```

For more information, read the Bun API docs in `node_modules/bun-types/docs/**.md`.

## Fork Management

This is a fork of `letta-ai/letta-code` with a declarative branch composition system.

### Branch Structure

| Branch | Purpose |
|--------|---------|
| `main` | Mirror of upstream `letta-ai/letta-code:main` |
| `fork` | Composed working branch (auto-built from feature branches) |
| `feature/*` | Fork-only feature branches |
| `bugfix/*` | Fork-only bugfix branches |
| `pr/*` | Branches for upstream PR contributions |

### Remotes

- `origin` - mweichert/letta-code (this fork)
- `upstream` - letta-ai/letta-code (upstream repo)

### Configuration

The fork composition is defined in `fork.yaml`:

```yaml
upstream:
  remote: upstream
  branch: main

base: main

branches:
  - name: feature/example
    base: main
    description: Example feature branch
    docs: branches/feature/example.md
```

### Rebuilding the Fork

To sync with upstream, rebase all branches, and rebuild fork:

```bash
uv run scripts/build-fork.py           # Full rebuild
uv run scripts/build-fork.py --dry-run # Preview changes
```

This script:
1. Fetches upstream and resets `main`
2. Rebases each branch onto its base (topologically sorted)
3. Merges all branches into `fork`
4. Pushes everything to origin

### Adding a New Branch

1. Create from main: `git checkout -b feature/my-feature main`
2. Make changes, commit, push
3. Create branch documentation in `branches/feature/my-feature.md`
4. Add entry to `fork.yaml`
5. Run `uv run scripts/build-fork.py`

### Contributing Upstream

For changes intended for upstream:
1. Create a `pr/*` branch from `main`
2. Make changes and push
3. Create PR via `gh pr create`
4. Do NOT add to `fork.yaml` (these should go upstream, not stay in fork)
