# Keroyokan Multisemesta - Web Platform Plan

Last updated: 2026-02-11

## 1) Project Goal

Build the official web platform for **Keroyokan Multisemesta** with these major surfaces:

- Public landing page
- Public wiki
- Admin dashboard
- Future card compendium and game systems pages

The card attribute model (HP, element, semesta, island) is still evolving, so architecture should stay flexible and migration-friendly.

---

## 2) Core Stack (Locked)

- App framework: **Svelte 5 + SvelteKit**
- UI system: **Tailwind CSS + shadcn-svelte + shadcn-svelte-extra**
- Animation: **GSAP**
- Database and backend functions: **Convex**
- Auth: **Better Auth**
- Rich text editor: **TipTap**
- Image storage: **Cloudflare R2**
- Issue tracking: **beads**

Notes:
- Use a Convex-compatible Better Auth setup from day one.
- Use R2 as the canonical storage for uploaded media and image assets (wiki images, card art, banners).

---

## 3) Product Surfaces

### Public

- `/` landing page
- `/wiki` wiki index
- `/wiki/[slug]` wiki detail page
- `/cards` card index (can start as placeholder)

### Admin

- `/dashboard` overview
- `/dashboard/wiki` wiki management
- `/dashboard/cards` card management
- `/dashboard/media` media management (R2-backed)

### System

- Auth routes (based on Better Auth integration)
- Protected route middleware/hooks

---

## 4) Phase Roadmap

## Phase -1: Kickstart Foundation + Coming Soon Landing

### Objective

Get a clean foundation running quickly and publish a branded coming soon page.

### Scope

1. Initialize SvelteKit (Svelte 5, TypeScript)
2. Add and configure:
   - Tailwind CSS
   - shadcn-svelte
   - shadcn-svelte-extra
3. Initialize beads in the project workspace
4. Build landing page only (no wiki/admin yet)
5. Landing content:
   - project logo
   - "Coming Soon" headline
   - optional short subtitle and social/community links
6. Add interactive hero element:
   - 3D tilt card on hover
   - uses `card_backcover.png` as the card face/background image

### Acceptance Criteria

- Local dev server runs cleanly
- Landing is responsive on desktop and mobile
- Landing includes logo + "Coming Soon"
- Card tilt interaction works on pointer devices and degrades gracefully on touch devices
- Basic accessibility checks pass (semantic structure, sufficient contrast, keyboard-safe page)

---

## Phase 0: Platform Baseline

### Objective

Prepare production-ready baseline for auth, backend, and deployment conventions.

### Scope

- Convex project wiring
- Better Auth integration and role scaffolding
- Cloudflare R2 upload/access strategy
- Environment variable standards and secrets handling
- Route protection strategy for dashboard area

### Acceptance Criteria

- Auth session flow is functioning
- Protected routes enforce access control
- Media upload proof-of-concept to R2 succeeds

---

## Phase 1: Design System + App Shell

### Scope

- Global layout and navigation
- Design tokens and reusable base components
- Motion guidelines with GSAP usage boundaries
- Shared loading/error/empty states

---

## Phase 2: Wiki (Read Experience)

### Scope

- Wiki list and detail pages
- Search/filter basics
- SEO metadata and share previews
- Related content blocks

---

## Phase 3: Wiki Authoring + Admin Workflows

### Scope

- TipTap editor integration
- Draft and publish workflow
- Revision history and rollback baseline
- Media library powered by R2

---

## Phase 4: Card Data Integration (When Schema Finalizes)

### Scope

- Implement final card schema
- Add migration path from placeholder fields
- Public card browsing and admin CRUD
- Connect taxonomy dimensions (element, semesta, island)

---

## Phase 5: Hardening + Release Prep

### Scope

- Quality gates (typecheck, lint, tests)
- Performance and accessibility pass
- Security and permission validation
- Production readiness checklist

---

## 5) Data Strategy While Card Schema Is Pending

Use a versioned structure to avoid rework:

- Add `schemaVersion` on card records
- Keep stable core fields (`name`, `slug`, `rarity`, `status`, `description`, `artwork`)
- Keep evolving game mechanics under a structured temporary field (`attributesRaw`)
- Plan explicit migration scripts when the final schema lands

This allows the website and admin workflow to move now without blocking on game mechanics details.

---

## 6) Media Strategy (Cloudflare R2)

- Store original and optimized image variants in R2
- Persist metadata in Convex (key, URL, dimensions, owner, tags, timestamps)
- Use signed upload flow for admin uploads
- Define folder conventions early:
  - `landing/`
  - `wiki/`
  - `cards/`
  - `branding/`
- Enforce MIME and size validation on upload pipeline

---

## 7) beads Setup Plan

Initial epics:

- EPIC-PHASE-M1: Kickstart Coming Soon
- EPIC-PHASE-0: Platform Baseline
- EPIC-PHASE-1: Design System and Shell
- EPIC-PHASE-2: Wiki Read
- EPIC-PHASE-3: Wiki Authoring and Admin
- EPIC-PHASE-4: Card Integration
- EPIC-PHASE-5: Hardening and Release

Suggested labels:

- `frontend`, `backend`, `infra`, `auth`, `media`, `wiki`, `admin`, `a11y`, `seo`

---

## 8) Immediate Next Actions

1. Execute **Phase -1** only
2. Keep scope strict: one landing page with logo + coming soon + tilt card
3. Defer wiki/admin implementation until Phase 0+ kickoff
4. Finalize role model and card schema proposal in parallel documentation
5. Follow commit/push quality gates in `WORKFLOW.md`
