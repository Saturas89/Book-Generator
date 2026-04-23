# REQ-001 – Projekt-Bootstrap

**Status:** 🟢 DRAFT · **Priorität:** MUST · **Modul:** Core

## 1. Ziel / User Story

Als Entwicklerin will ich ein lauffähiges Next.js-15-Projekt vorfinden, das Tests, Linting, i18n und Datenbank-Skelett mitbringt, damit alle nachfolgenden REQs darauf aufbauen können, ohne Tooling-Entscheidungen zu wiederholen. Erfolg = `npm run dev` zeigt eine Landing-Seite, `npm test` und `npm run test:e2e` laufen grün durch, und die CI-Matrix in `.github/workflows/e2e.yml` endet erfolgreich.

## 2. Akzeptanzkriterien

- [ ] `package.json` existiert mit Scripts `dev`, `build`, `start`, `lint`, `test` (Vitest), `test:e2e` (Playwright), `typecheck`.
- [ ] Next.js 15 mit App-Router, TypeScript strict (`"strict": true`, `"noUncheckedIndexedAccess": true`) und ESLint sind installiert.
- [ ] Internationalisierung: `/de` und `/en` sind separate Locale-Segmente; Besuche von `/` leiten per Middleware auf die bevorzugte Locale um. Umschalter-UI rendert zwei Buttons mit `data-testid="locale-switch-de"` und `data-testid="locale-switch-en"`.
- [ ] Landing-Seite zeigt in beiden Locales einen sichtbaren Titel `data-testid="landing-title"` und einen Primär-CTA `data-testid="primary-cta"` (in DE „Anmelden", in EN „Sign in"). Der CTA verlinkt auf `/login`, die Route existiert bereits als Platzhalter.
- [ ] `prisma/schema.prisma` existiert mit `provider = "postgresql"` und leerem Model-Block; Prisma-Client ist als Dev-Dependency installiert; `npm run prisma:generate` ist ein Script.
- [ ] `.env.example` listet mindestens `DATABASE_URL`, `BLOB_READ_WRITE_TOKEN`, `NEXTAUTH_SECRET`, `AUTH_GOOGLE_ID`, `AUTH_GOOGLE_SECRET`, `AUTH_GITHUB_ID`, `AUTH_GITHUB_SECRET`, `DIALOG_MODE`, `AI_MOCK`, `E2E` — je mit Kurzkommentar.
- [ ] `playwright.config.ts` setzt `webServer` so, dass Playwright `next dev` mit `AI_MOCK=1 E2E=1` startet.
- [ ] `vitest.config.ts` existiert und `npm test` endet mit Exit 0 (auch wenn noch keine Assertion-Tests existieren).
- [ ] `npm run typecheck` endet mit Exit 0.
- [ ] README.md (Repo-Root) dokumentiert: Install, Dev-Start, Tests, Env-Vars.

## 3. Nicht-Ziele

- Kein Auth-Setup (→ REQ-002), keine eigenen UI-Komponenten über das absolute Minimum hinaus (→ REQ-001a), keine Datenmodelle für Domänenobjekte (→ REQ-004, REQ-005), keine AI-Proxy-Routen (→ REQ-005a).
- Keine Wahl eines spezifischen UI-Frameworks (Tailwind, MUI, …) wird in dieser Spec erzwungen — Impl-Agent darf eines ergänzen, solange die Test-Ableitbarkeit der `data-testid`-Werte erhalten bleibt.

## 4. Datenmodell / API

`prisma/schema.prisma` startet als:

```prisma
datasource db { provider = "postgresql"; url = env("DATABASE_URL") }
generator client { provider = "prisma-client-js" }
```

Keine Models in dieser REQ — folgende REQs erweitern das Schema additiv.

`.env.example` (gekürzt):

```
DATABASE_URL=postgresql://user:pass@localhost:5432/bookgen
BLOB_READ_WRITE_TOKEN=
NEXTAUTH_SECRET=
AUTH_GOOGLE_ID=
AUTH_GOOGLE_SECRET=
AUTH_GITHUB_ID=
AUTH_GITHUB_SECRET=
DIALOG_MODE=0
AI_MOCK=0
E2E=0
```

## 5. UI / UX

Landing-Seite minimal: Titel, kurzer Erklärtext, Primär-CTA „Anmelden/Sign in", Locale-Switcher oben rechts. Mobile-first, Breakpoint bei ~768 px. Keine Bilder außer eventuell ein SVG-Logo.

## 6. Tests

- Playwright-Smoke (`e2e/bootstrap.spec.ts`): Besucht `/de` und `/en`, prüft Sichtbarkeit von `landing-title` und `primary-cta`, klickt Locale-Switcher und verifiziert URL-Wechsel.
- Vitest-Platzhalter (`src/app/__smoke__.test.ts`) der einen trivialen Assert laufen lässt, damit `npm test` keinen „no tests found"-Exit-Code wirft.

## 7. Offene Fragen

- Welche Styling-Lösung soll verbindlich werden? Vorschlag Impl-Agent: Tailwind CSS (geringer Bootstrap-Overhead, gute Lesbarkeit in Tests via Klassen unabhängig). Wenn keine Festlegung, steht es Impl-Agent frei.
- Verwendet `next-intl` v3? Annahme ja, aktuell stabil mit App-Router.
