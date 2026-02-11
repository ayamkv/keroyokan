# Keroyokan Multisemesta Web

Official website project for **Keroyokan Multisemesta**.

Current focus: **Phase -1** (foundation and coming-soon landing page).

## Core Stack

- Svelte 5 + SvelteKit
- Tailwind CSS
- shadcn-svelte
- shadcn-svelte-extras registry (via jsrepo)
- GSAP (landing animations)
- beads issue tracking

Planned in next phases: Convex, Better Auth, TipTap, Cloudflare R2 integration.

## Local Development

Install dependencies:

```bash
npm install
```

Run dev server:

```bash
npm run dev
```

Type checks:

```bash
npm run check
```

Build for production:

```bash
npm run build
```

## Workflow Docs

- Project plan: `PROJECT_PLAN.md`
- Commit/push and quality gates: `WORKFLOW.md`
- beads workspace guide: `AGENTS.md`

## Cloudflare Deploy

This repo is configured for Cloudflare via `@sveltejs/adapter-cloudflare` and `wrangler.jsonc`.

- Build command: `npm run build`
- Deploy command (Workers): `npx wrangler deploy`

If using Cloudflare Pages Git integration, set:

- Framework preset: `SvelteKit`
- Build command: `npm run build`
- Build output directory: `.svelte-kit/cloudflare`
