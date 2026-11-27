# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Cloudflare Worker built with [Hono](https://hono.dev), scaffolded from the `create-hono` Cloudflare Workers template. The entire app is currently `src/index.ts`, which exports a Hono app as the Worker's default export (Wrangler's entry point is set by `main` in `wrangler.jsonc`).

## Commands

- `npm install`: install dependencies. `package-lock.json` is present, but `pnpm-workspace.yaml` also exists (it allows the `esbuild`/`workerd` build scripts), so either package manager works.
- `npm run dev`: run locally with `wrangler dev` (Miniflare). Local state for KV/D1/R2 and similar goes in `.wrangler/`, which is gitignored.
- `npm run deploy`: deploy to Cloudflare with `wrangler deploy --minify`.
- `npm run cf-typegen`: generate the `CloudflareBindings` type from `wrangler.jsonc`. Rerun it after you add or change bindings.

No test runner, linter, or build step is configured. Wrangler bundles TypeScript directly. To type-check, run `npx tsc --noEmit`.

## Architecture notes

- **Bindings**: you declare Worker bindings (vars, KV, R2, D1, AI, observability) in `wrangler.jsonc`, where commented-out examples are already present. After adding one, run `cf-typegen`, then type the app as `new Hono<{ Bindings: CloudflareBindings }>()` so you can reach the bindings through `c.env`. The app is not typed with bindings yet.
- **Runtime**: this is the Workers runtime, not Node. Node APIs need `"compatibility_flags": ["nodejs_compat"]` in `wrangler.jsonc`. `tsconfig.json` uses `lib: ["ESNext"]` only, with no DOM or Node types.
- **JSX**: `tsconfig.json` sets up `hono/jsx` as the JSX runtime, so `.tsx` files can render HTML with Hono's JSX and no React.
- **Local secrets**: put them in `.dev.vars` (gitignored), not in `wrangler.jsonc` `vars`.
