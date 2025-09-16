# AGENTS.md — Implementation Agent

This document specifies how an automated agent discovers design mockups, generates production-ready UI pages/components, binds them to backend APIs, and ships code safely without touching the mockup source files.
It serves as an executable process description for automated development in a monorepo.

---

## 0. Goals & Principles

**Goals**

1. Detect new mockups in `designmockup/` and turn them into functional frontend pages/components.
2. Wire the generated UI to defined APIs and authentication.
3. Respect project requirements (declared in `agents.config.json`).
4. Never delete or modify files in `designmockup/`. Do not overwrite completed pages unless explicitly allowed.
5. Deliver accessible (WCAG 2.1 AA), responsive code aligned with the project’s design tokens or style guide.

**Core Principles**

* **Non-destructive:** Read mockups; do not delete or alter them.
* **Idempotent:** Multiple runs must not create duplicates or change finished pages.
* **Configuration over convention:** Paths, names, and mappings are overridable via `agents.config.json`.
* **Security first:** Follow auth flows and access control. Do not leak secrets to the client.
* **Observability:** Log each step (discovery → generation → binding → test).

---

## 1. Assumptions & Tech Stack (example)

The agent is stack-agnostic. A common baseline is:

* **Frontend:** React + TypeScript (or Vue/Svelte/Angular), router, utility-first CSS (e.g., Tailwind), Axios/fetch.
* **Backend:** Node/Express (or equivalent), REST or GraphQL.
* **CI/CD:** GitHub Actions (or similar) with environment-specific deploy.
* **Storage:** File-based or database-backed; document details in `STORAGE.md`.

Projects can tailor any of the above through `agents.config.json`.

---

## 2. Inputs & Outputs

**Inputs**

* `designmockup/`: per-page/component mockups (`.png`, `.jpg`, `.svg`, `.html`, `.css`, `.json`).
* `agents.config.json` (optional): rules, skip lists, routes, and bindings.

**Outputs**

* Generated pages in `frontend/src/pages/*`.
* Components in `frontend/src/components/*`.
* Routes in `frontend/src/router/*` (or `src/router.tsx`).
* API bindings in `frontend/src/api/*`.
* Run log: `agent-run-YYYYMMDD-HHMM.json`.

---

## 3. Discovery & Decision Logic

1. Recursively scan `designmockup/` and index files by page slug.
2. Check whether a corresponding page exists in `src/pages`.
3. If it exists → skip (unless `--allow-update` is set).
4. If it doesn’t exist → generate.
5. Never delete or modify files in `designmockup/`.

---

## 4. Generation Flow

### 4.1 Standard flow

1. **Parse mockup** (HTML/CSS or image).
2. **Build component tree:** atoms → molecules → organisms; place in `components/ui` or feature folders.
3. **Create page container** with state, data fetching, error and loading states.
4. **Add routes** (lazy import, `Suspense`/skeletons, and route guards as needed).
5. **Accessibility:** ARIA roles/labels, focus handling, keyboard navigation.
6. **Tests:** rendering, interaction, and a11y checks.
7. **Telemetry & logging:** page views, key CTAs, network/console errors.
8. **Commit:** semantic commit messages.

### 4.2 Page-specific rules

Define per-page rules in `agents.config.json`, for example:

```json
{
  "pages": {
    "documents": { "standalone": true },
    "projects": { "externalOnly": true }
  }
}
```

---

## 5. Frontend Structure (example)

```
frontend/
  src/
    api/
    components/
      ui/
      patterns/
      feature/
      globals/
    hooks/
    pages/
    router/
    styles/
```

**Rules**

* Anything used on 2+ pages → move to `components/ui` or `components/patterns`.
* `feature/` should primarily compose from `ui` + `patterns` (avoid restyling duplicates).

---

## 6. Parsing & Styling

* Translate HTML/CSS mockups to utility classes where possible.
* When utilities are insufficient, add minimal `@layer components` rules in `styles/components.css`; do **not** embed page-local styles in page files.
* Define tokens (colors, spacing, radii, shadows, typography) in `styles/tokens.css` and surface them through the chosen CSS system.

---

## 7. API Binding & State

* Use an HTTP client with auth/error interceptors.
* Support token refresh flows where applicable.
* Surface errors via toasts and screen-reader announcements.
* Realtime (if applicable): WebSockets/SSE with authenticated connections and scoped rooms.

---

## 8. Quality & Testing

* **Automation:** strict TypeScript, ESLint/Prettier, unit/integration tests (e.g., React Testing Library), a11y checks (axe), Lighthouse budgets in CI.
* **Manual:** responsive behavior (mobile → desktop), full keyboard traversal, contrast compliance, functional smoke tests on key flows.

---

## 9. CI/CD (example)

* NPM scripts: `dev`, `build`, `test`, `lint`, `format`, `preview`.
* Pipeline: install → lint → test → build → artifact → deploy.
* Post-deploy: health checks and process restarts where relevant.

---

## 10. Security & Privacy

* Enforce HTTPS.
* No secrets in frontend bundles.
* Store short-lived access tokens in memory; keep refresh tokens in httpOnly cookies (subject to project policy).
* Rate-limit sensitive flows (login, code verification).
* Apply applicable privacy/data-protection rules (e.g., GDPR) per project.

---

## 11. Agent CLI

```
# Dry run: list mockups and planned outputs
npm run agent:plan

# Generate new pages from mockups (no overwrites)
npm run agent:generate

# Allow updates to existing pages (show diffs, require confirm)
npm run agent:generate -- --allow-update

# Limit to specific slugs
npm run agent:generate -- --only documents,projects
```

**Behavior**

* Log all findings to `agent-run-*.json` (time, inputs, output paths).
* Protect existing pages; with `--allow-update`, prompt per file.
* Never `unlink` anything in `designmockup/`.

---

## 12. Mockup → Code Mapping (examples)

* `designmockup/news-list.html` → `components/feature/news/NewsCard.tsx`, `NewsFilters.tsx`, and `pages/News/NewsPage.tsx` wired to `GET /api/news?...`.
* `designmockup/documents.*` → `pages/Documents/DocumentsPage.tsx` (optionally standalone without globals), plus `DocumentTable`/`DocumentCard`, filters, pagination.

---

## 13. Failure Handling & Rollback

* On generation failure, write human-readable diagnostics and diffs to `agent-run-*.json`.
* Roll back partial changes using git (stash/restore) on marked paths.
* Never touch `designmockup/` regardless of errors.

---

## 14. Maintenance & Extensions

* Support variants: `*-mobile`, `*-dark` mockups. Prioritize base variants; gate others behind flags/props.
* Optional Storybook snapshots for core UI.
* E2E (Playwright/Cypress) for critical journeys as the project defines them.

---

## 15. Quick Run Checklist

* [ ] New mockups discovered in `designmockup/`.
* [ ] Existing pages only updated with `--allow-update`.
* [ ] Components and pages generated in correct folders.
* [ ] Routes and guards added.
* [ ] API wiring tested for loading/error states.
* [ ] A11y checks passed.
* [ ] Build succeeded; artifact and run-log written.
* [ ] No files in `designmockup/` were modified or deleted.

---

## 16. Storage & Persistence

* The project may use file-based or database storage.
* Record concrete data rules and formats in `STORAGE.md` (entity schemas, indexes, media rules, atomic writes, locking strategy).

---

## 17. Documentation & Versioning

* Maintain `CHANGELOG.md` using SemVer (`x.y.z`) with **Added / Changed / Fixed / Removed** sections, each dated `YYYY-MM-DD`.
* Keep these docs in sync with code on every relevant change:

  * `README.md` — project overview, how to run/build.
  * `AGENTS.md` — agent behavior and generation rules.
  * `ARCHITECTURE.md` — system architecture, APIs, component overview.
  * `STORAGE.md` — storage model and data formats.
* Create additional docs under `/documents/` as needed (`SECURITY.md`, `API_REFERENCE.md`, `STYLEGUIDE.md`, `EDITOR_GUIDE.md`).

---

## Appendix A — Minimal `agents.config.json` (boilerplate)

```json
{
  "$schema": "./agents.config.schema.json",
  "mockupDir": "designmockup",
  "routeBase": "/",
  "pages": {
    "documents": { "standalone": true },
    "projects": { "externalOnly": false }
  },
  "skip": ["globals"],
  "output": {
    "pagesDir": "frontend/src/pages",
    "componentsDir": "frontend/src/components",
    "apiDir": "frontend/src/api",
    "routerFile": "frontend/src/router.tsx",
    "stylesDir": "frontend/src/styles"
  },
  "style": {
    "useUtilityCss": true,
    "tokensFile": "frontend/src/styles/tokens.css",
    "componentsCss": "frontend/src/styles/components.css"
  },
  "api": {
    "baseUrlEnv": "VITE_API_BASE",
    "client": "axios",
    "auth": {
      "useInterceptors": true,
      "refreshEndpoint": "/api/refresh",
      "redirectOnAuthError": "/login"
    }
  },
  "accessibility": {
    "wcagTarget": "2.1-AA",
    "enableAxeInTests": true
  },
  "testing": {
    "unit": true,
    "a11y": true,
    "lighthouseInCi": true
  },
  "generation": {
    "allowOverwrite": false,
    "allowUpdateFlag": "--allow-update",
    "onlyFlag": "--only"
  },
  "telemetry": {
    "pageViewEvent": "page_view",
    "errorReporting": "sentry"
  }
}
```

**Notes**

* Adjust paths to match your repo layout.
* Omit or expand sections as needed (e.g., replace Axios with fetch, or alter router location).
* If you prefer framework-specific structures (Next.js, Nuxt, etc.), set `output.*` accordingly and the agent should honor it.
