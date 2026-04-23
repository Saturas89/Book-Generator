# REQ-001a – Design-System, Fehler- und Leerzustände, a11y-Basis

**Status:** 🟢 DRAFT · **Priorität:** MUST · **Modul:** UI

## 1. Ziel / User Story

Als Nutzerin will ich in der gesamten App konsistente Fehler-, Lade- und Leer-Anzeigen sehen, damit mir Probleme (fehlende Daten, Netzwerkausfall, ungültiger Input) sofort verständlich sind. Als Impl- und Test-Agent späterer REQs will ich feste, bereits getestete Primitives haben, damit ich nicht in jeder Feature-Spec neu „Error-Banner" erfinden muss. Erfolg = drei wiederverwendbare Komponenten + ein Basis-Style-Token-Set existieren, werden von späteren REQs konsumiert und sind via feste `data-testid`-Werte prüfbar.

## 2. Akzeptanzkriterien

- [ ] Komponente `<ErrorBanner>` rendert einen Fehler-Container mit `data-testid="error-banner"`, `role="alert"`, zeigt eine i18n-übersetzte Nachricht (Prop `messageKey`) und optional einen „Erneut versuchen"-Button `data-testid="error-retry"` (Prop `onRetry`).
- [ ] Komponente `<EmptyState>` rendert einen Leerzustand mit `data-testid="empty-state"`, einer Illustration/Icon, einer Hauptnachricht und optional einem Call-to-Action-Button `data-testid="empty-state-cta"`.
- [ ] Komponente `<LoadingSpinner>` rendert mit `data-testid="loading-spinner"` und `aria-busy="true"`, optional begleitet von einem Label-Text via Prop `labelKey`.
- [ ] Komponente `<PrimaryButton>` rendert einen Button mit durchgereichtem `data-testid`-Prop, Keyboard-Focus ist sichtbar (Outline oder Ring), bei `disabled` ist `aria-disabled="true"` gesetzt.
- [ ] Komponente `<TextField>` rendert `<label>` + `<input>` korrekt assoziiert (`htmlFor`/`id`), zeigt unter dem Feld Fehlermeldungen mit `data-testid="${props.testId}-error"` wenn `error`-Prop gesetzt.
- [ ] Alle i18n-Keys für generische Fehler existieren in `de.json` und `en.json`: `error.generic`, `error.network`, `error.unauthorized`, `error.rate_limit`, `error.invalid_key`, `error.provider_down`, `error.content_blocked`, `empty.default.title`, `empty.default.description`.
- [ ] Basis-Farben, Abstände, Schriftgrößen sind als CSS-Variablen/Tokens zentral definiert (Datei `src/styles/tokens.css` oder Tailwind-Config).
- [ ] Jede Komponente ist mit Vitest+Testing-Library getestet: Render, `data-testid` vorhanden, Keyboard-Focus, Label-Assoziation.
- [ ] Mindestkontrast: Fließtext mindestens 4.5:1 gegen Hintergrund (WCAG AA).

## 3. Nicht-Ziele

- Keine vollständige Komponenten-Bibliothek (keine Modals, keine Tabs, kein Datepicker) — nur die in §2 genannten Primitives. Weitere Komponenten entstehen bedarfsgetrieben in nachfolgenden REQs.
- Kein Theme-Switching (Dark-Mode) im MVP.
- Keine Animationen über einfache Transitions hinaus.

## 4. Datenmodell / API

Nicht relevant — rein UI.

## 5. UI / UX

- Fehler-Banner: rot-orange gefärbter Hintergrund mit Kontrast, Icon links, Text, optional Retry-Button rechts.
- Leerzustand: zentriert, großzügig Weißraum, freundliche Illustration (SVG-Placeholder OK).
- Spinner: kreisend, 24 px Default, kann per Prop `size="sm|md|lg"` skalieren.
- Buttons: zwei Varianten primary/secondary, klarer Focus-Ring.

## 6. Tests

- Unit/Component (Vitest + Testing Library) für jede Komponente: Props, Testids, a11y-Attribute, Keyboard-Tab-Reihenfolge.
- Keine E2E-Tests nötig — die Primitives werden in späteren REQs e2e-getestet.

## 7. Offene Fragen

- Soll eine konkrete Icon-Library verwendet werden (Lucide, Heroicons)? Vorschlag: Lucide React.
- Sollen Tokens als CSS-Variablen oder Tailwind-Config gepflegt werden? Wenn REQ-001 Tailwind wählt, in Tailwind-Config; sonst CSS-Variablen.
