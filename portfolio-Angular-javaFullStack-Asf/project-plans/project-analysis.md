# Project Analysis — `portfolio-Angular-javaFullStack-Asf`

**Date:** 2026-10-27  
**Analyzed by:** OpenCode agent  
**Project type:** Angular 22 standalone-component single-page portfolio (frontend only)  
**Status:** Builds and tests pass; backend/DevOps phases not yet implemented.

---

## 1. Executive Summary

This repository is a polished, dark-themed personal portfolio for **Ashfakul Alam** ("Full-Stack Software Developer"). It is a **frontend-only Angular 22 application** built with standalone components, Bootstrap 5, Bootstrap Icons, and a large custom SCSS design system (neon gradients, glassmorphism, keyframe animations). All content is hardcoded in a central `PortfolioService`. There is **no backend, no persistence, no real contact submission, and no routing** — the app is a single scrolling page.

The codebase is clean and compiles cleanly (`npm run build` succeeds, `npm test` reports 2/2 passing). However, the bundled `project-plans/portfolio-improvement-roadmap.md` describes an ambitious full-stack transformation (Spring Boot 3 + PostgreSQL + Docker) that is **not yet implemented**. Several README/roadmap claims (e.g. "Signals-ready", "centralized data store", "broken Vitest test") do not match the current code (plain object state, no signals, tests actually pass).

---

## 2. Tech Stack & Tooling

| Layer | Technology | Version |
|---|---|---|
| Framework | Angular (standalone) | 22.2.3 |
| Language | TypeScript | ~6.0.2 |
| Build | `@angular/build` (esbuild) | 22.0.7 |
| Test | Vitest + jsdom | 4.0.8 / 28.0.0 |
| UI | Bootstrap 5, Bootstrap Icons | 5.3.8 / 1.13.2 |
| Styling | SCSS (global + per-component) | — |
| Typography | Outfit, Fira Code (Google Fonts) | — |
| Runtime | Node.js | 22.23.1 |
| Package manager | npm | 12.2.0 |

Configuration files: `angular.json`, `tsconfig.json` (strict mode, `noImplicitReturns`, `noFallthroughCasesInSwitch`, `strictInjectionParameters`, `strictInputAccessModifiers`), `.prettierrc`, `.editorconfig`, `.vscode/*`.

---

## 3. Project Structure

```
portfolio-Angular-javaFullStack-Asf/
├── angular.json
├── package.json
├── project-plans/
│   └── portfolio-improvement-roadmap.md   # Future-state plan (unimplemented)
├── public/
│   ├── favicon.ico
│   └── images/                            # ashfakul-profile.jpg, 68431.jpg (~193 KB each)
├── src/
│   ├── app/
│   │   ├── app.config.ts                  # only provideBrowserGlobalErrorListeners()
│   │   ├── app.ts / app.html / app.scss   # root orchestrator, no logic
│   │   ├── app.spec.ts                    # 2 tests (create + h1 contains title)
│   │   ├── core/
│   │   │   ├── models/portfolio.model.ts  # Skill, SkillCategory, Project, SocialLink, ContactFormData
│   │   │   └── services/portfolio.service.ts  # hardcoded data + getters
│   │   └── components/
│   │       ├── navbar/   hero/   about/   skills/   projects/   contact/   footer/
│   │       └── (each: .ts, .html, .scss)
│   ├── index.html                          # SEO meta, fonts, theme-color
│   ├── main.ts
│   └── styles.scss                         # global design system (~320 KB compiled)
└── dist/portfolio/                         # production build output
```

Total source: ~7 components + root, 1 service, 1 model file. No routing module, no interceptors, no guards.

---

## 4. Architecture & Data Flow

- **Bootstrap:** `main.ts` → `bootstrapApplication(App, appConfig)`.
- **Layout:** `App` template stacks `app-navbar`, `app-hero`, `app-about`, `app-skills`, `app-projects`, `app-contact`, `app-footer` inside `<main>`.
- **Data:** `PortfolioService` (providedIn root) holds `profile`, `socialLinks`, `skillCategories`, `projects` as **plain mutable fields** and exposes getter methods (`getProfile`, `getSocialLinks`, …). Components inject it and read synchronously in constructors (e.g. `profile = this.portfolioService.getProfile()`).
- **Navigation:** Anchor links (`#about`, `#skills`, …) + `scroll-behavior: smooth`. Navbar toggles a `.scrolled` class via a `window:scroll` HostListener. No active-section highlighting (scrollspy) is implemented.
- **Projects modal:** `Projects` component keeps `selectedProject: Project | null` and renders a custom modal backdrop; backdrop click closes, inner click stops propagation.
- **Contact form:** Template-driven (`FormsModule` + `ngModel`) with HTML5 validation; submission simulated with `setTimeout(…, 900)` and resets the form; shows a success panel.

---

## 5. Component Inventory

| Component | Responsibility | Notable |
|---|---|---|
| `Navbar` | Sticky glassmorphic nav, mobile drawer | scroll listener; no scrollspy; no resume CTA |
| `Hero` | Animated gradient hero, avatar, floating badges | h1 = "Full-Stack Software Developer" (drives the passing test) |
| `About` | Bio, strengths, stats grid, availability card | stats hardcoded locally (not from service) |
| `Skills` | 4 grouped skill cards (Frontend/Backend/DB/Mobile) | all badges say "Proficient" regardless of level |
| `Projects` | 4 project cards + details modal | filter/search not implemented; links are `#` |
| `Contact` | Floating-label form + social links | mock submission; no HttpClient |
| `Footer` | Copyright + back-to-top | uses `new Date().getFullYear()` |

---

## 6. Styling & Design System

`src/styles.scss` defines a comprehensive token system: CSS custom properties for colors (`--neon-cyan`, `--neon-magenta`, …), gradients, radii, glows; glassmorphism (`.glass-panel`); vibrant buttons (`.btn-vibrant-primary`/`-outline`); keyframes (`gradientShift`, `floatSmooth`, `pulseNeon`, `rotateBorder`, `slideDown`, `fadeIn`, `scaleUp`); custom scrollbar; reduced-motion media query. Per-component SCSS scopes layout. Bootstrap is imported globally. Compiled CSS bundle is ~321 KB (raw) — large mainly because of Bootstrap + full custom stylesheet.

---

## 7. State Management, Forms, HTTP

- **State:** None reactive. README/roadmap claim "Signals-ready" but no `signal()`/`computed()`/`toSignal()` exists. Components store plain properties; change detection is default.
- **Forms:** Template-driven (`FormsModule`). Required + `email` validators. No `ReactiveFormsModule`, no `FormGroup`.
- **HTTP:** Not configured. `app.config.ts` does not call `provideHttpClient`. No API calls anywhere.
- **Routing:** No `@angular/router` usage despite dependency present.

---

## 8. Testing

- Single spec file `src/app/app.spec.ts` with 2 tests:
  1. Component creation.
  2. `h1` text contains "Full-Stack Software Developer".
- Both pass (Vitest 4.1.12, jsdom). No component-level, service, or integration tests. Coverage is effectively minimal.
- Note: roadmap described this test as "broken" (title mismatch), but the current expectation already matches the rendered `h1`, so it passes.

---

## 9. Build & Bundle

Production build (`npm run build`):
- `main-*.js` ≈ 226 KB raw (≈ 60 KB transfer)
- `styles-*.css` ≈ 321 KB raw (≈ 34 KB transfer)
- Initial total ≈ 547 KB raw (≈ 94 KB transfer)
- Output: `dist/portfolio/browser`. No errors, no warnings.

Budgets configured in `angular.json`: initial 1 MB warn / 2 MB error; any component style 10 KB warn / 20 KB error.

---

## 10. Issues, Risks & Debt

1. **Roadmap vs. reality gap.** The roadmap (Phase 1–5) is almost entirely unimplemented: no signals, no scrollspy, no project filter/search, no toast service, no CV download, no Spring Boot backend, no HttpClient, no Docker, no SEO beyond basic meta. README markets some of these as present.
2. **Placeholder links.** `socialLinks` point to generic `https://github.com`, `https://linkedin.com`, etc.; project `demoUrl`/`githubUrl` are `#`. Not production-ready.
3. **Mock contact form.** `onSubmit` only `setTimeout`s; no validation beyond required fields, no honeypot, no rate limiting, no persistence/notification.
4. **Unused `CommonModule` imports.** Every component imports `CommonModule` but templates use the new control flow (`@if`/`@for`) and no pipes/directives from it. Likely dead weight (not a compile error).
5. **No accessibility beyond basics.** Project modal lacks focus trap/Escape handling and `role="dialog"`/`aria-modal`; mobile drawer doesn't manage focus; no skip-to-content link.
6. **Remote images.** Project cards load 4 Unsplash images; hero avatar is a 193 KB JPEG at 472×591. No responsive `srcset`/modern formats.
7. **Stats duplicated.** `About` keeps its own `stats` array instead of deriving from `profile` (experienceYears, completedProjects, satisfactionRate), risking drift.
8. **Skills badge hardcoded.** All skills render "Proficient" regardless of `level` (which holds descriptive text like "Signals & Standalone Architecture").
9. **No error handling / offline fallback.** Service is synchronous and in-memory; no loading states.
10. **No lint/test scripts beyond `ng test`.** No ESLint, no coverage thresholds.
11. **Git:** repository has no commits; everything is untracked. Risk of losing work; no history.
12. **packageManager field** declares `npm@12.2.0` (unusual major), though installed npm is 12.2.0 — consistent here but may confuse CI.

---

## 11. Roadmap Alignment Matrix

| Roadmap item | Status |
|---|---|
| 1.1 Fix broken unit test | Already passing (no action needed) |
| 1.2 Signals migration (`signal`/`computed`) | Not started |
| 2.1 Active scrollspy navbar | Not started |
| 2.2 Project filter + search + empty state | Not started |
| 2.3 Skills proficiency indicators / tabs | Partial (level text only) |
| 2.4 Resume/CV download CTA | Not started |
| 2.5 Toast notification service | Not started |
| 3.x Spring Boot 3 backend | Not started (no `backend/` dir) |
| 4.1 Angular HttpClient layer | Not started |
| 4.2 Reactive contact form | Not started |
| 5.1 Docker orchestration | Not started |
| 5.2 SEO / OpenGraph / JSON-LD | Partial (basic meta only) |

---

## 12. Recommended Next Steps (prioritized)

1. **Quick wins (frontend):** Remove unused `CommonModule` imports; derive `About.stats` from `profile`; make skills badge reflect `level`; replace placeholder social/project URLs; add Escape-to-close + focus management to modal and mobile drawer.
2. **Phase 1 (signals):** Convert `PortfolioService` fields to `signal()` and expose `computed()` (featured projects, filtered lists, counts); switch components to `OnPush` where beneficial.
3. **Phase 2 (UX):** Implement navbar scrollspy (IntersectionObserver), project category pills + debounced search + empty state, CV download button/modal, toast service.
4. **Phase 3–4 (full-stack):** Scaffold `backend/` Spring Boot 3 app (Java 21, PostgreSQL/H2, JPA) with `/api/v1/{profile,skills,projects,contact}`; add honeypot + Bucket4j rate limiting; wire Angular `provideHttpClient(withFetch())` with offline fallback; migrate contact form to `ReactiveFormsModule`.
5. **Phase 5 (DevOps/SEO):** Add Dockerfiles + `docker-compose.yml`, OpenGraph/JSON-LD tags, responsive images, and component-level tests to reach meaningful coverage.
6. **Housekeeping:** Create an initial git commit, add ESLint + coverage thresholds, optimize the 193 KB avatar.

---

## 13. Verification Performed

- `npm test -- --watch=false` → **2/2 tests passed** (1.03 s env + 1.03 s tests).
- `npm run build` → **success**, initial bundle 547 KB raw / 94 KB transfer, no warnings.
- `ng version` → Angular 22.2.3, Node 22.23.1, npm 12.2.0.
- Source scanned for `TODO/FIXME/XXX/HACK` → none found.
- No `console.*` in components except `main.ts` bootstrap error handler.