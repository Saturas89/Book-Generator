# REQ-007 – Dialog-Wizard ersetzt das One-Shot-Formular

**Status:** 🟢 DRAFT · **Priorität:** SHOULD · **Modul:** Features

## 1. Ziel / User Story

Als Elternteil will ich beim Buch-Anlegen statt eines langen Formulars durch einen freundlichen Schritt-für-Schritt-Dialog geführt werden, der mich pro Schritt nur nach einer Sache fragt und das nächste Feld je nach vorheriger Antwort dynamisch anpasst. Erfolg = wenn `DIALOG_MODE=1` gesetzt ist, zeigt `/books/new` einen Wizard mit sichtbarem Fortschritt; das Submit-Verhalten ist identisch zu REQ-005.

## 2. Akzeptanzkriterien

- [ ] Wenn `process.env.NEXT_PUBLIC_DIALOG_MODE === "1"` **oder** LocalStorage-Schalter `bg.prefs.dialogMode === "true"` gesetzt ist, rendert `/books/new` den Wizard statt des One-Shot-Formulars.
- [ ] Wizard hat genau diese Schritte, in dieser Reihenfolge; jeder Schritt ist eine eigene Panel-Ansicht mit `data-testid="wizard-step-{n}"` (1-indexiert):
  1. Charakter-Variante (`wizard-step-1`)
  2. Altersstufe (`wizard-step-2`)
  3. Illustrationsstil (`wizard-step-3`)
  4. Thema / Prompt (`wizard-step-4`)
  5. Seitenzahl (`wizard-step-5`)
  6. Bilderdichte (`wizard-step-6`)
  7. Sprache (`wizard-step-7`)
  8. Zusammenfassung (`wizard-step-8`)
- [ ] Progress-Leiste `data-testid="wizard-progress"` mit `aria-valuenow = aktuellerSchritt`, `aria-valuemax = 8`.
- [ ] Navigationsbuttons: `data-testid="wizard-next"`, `data-testid="wizard-prev"`. `wizard-next` ist disabled, solange die Eingabe des aktuellen Schritts nicht valide ist.
- [ ] Schritt 8 ist eine Zusammenfassung aller Eingaben; „Generieren"-Button `data-testid="wizard-generate"` sendet denselben `POST /api/books`-Payload wie REQ-005.
- [ ] Wizard-Zustand bleibt in der aktuellen Sitzung erhalten (im Component-State); es gibt keine Persistenz, wenn der Nutzer die Seite verlässt.
- [ ] `DIALOG_MODE`-Flag kann im Settings-UI (`/settings/preferences`) per Toggle `data-testid="pref-dialog-mode"` pro Nutzer ein-/ausgeschaltet werden; die Auswahl landet im LocalStorage unter `bg.prefs.dialogMode`.
- [ ] Wenn keine „bookable"-Charaktere existieren, zeigt Schritt 1 dasselbe Empty-State-Verhalten wie REQ-005.

## 3. Nicht-Ziele

- Keine Änderungen am Datenmodell.
- Keine Änderungen an den Proxy-Routen.
- Kein serverseitiger Dialog-Agent (keine KI-gestützten Rückfragen in diesem Schritt — der Dialog ist rein UI-gesteuert).
- Kein Cross-Session-Persist des Wizard-Fortschritts.

## 4. Datenmodell / API

Kein neues Modell. Kein Proxy-Change.

LocalStorage-Ergänzung: `bg.prefs.dialogMode` (String `"true"`/`"false"`).

## 5. UI / UX

- Jeder Schritt füllt den Viewport mit einer großen Frage und einem zentrierten Eingabecontrol
- Animierter Wechsel zwischen Schritten (Fade/Slide)
- Sprache-Schritt und Altersstufe werden mit visuellen Cards dargestellt (nicht Radio-Dots)
- Zusammenfassung listet alle Werte untereinander mit einem „Bearbeiten"-Link je Zeile, der zum entsprechenden Schritt zurückspringt

## 6. Tests

- Playwright E2E (`e2e/wizard.spec.ts`) mit `DIALOG_MODE=1`:
  - Jeden Schritt bedienen, Next/Prev funktioniert, Progress ändert sich
  - Validierung: Schritt 4 (Thema) leerer Text → `wizard-next` disabled
  - In Schritt 8 „Bearbeiten" bei Altersstufe → springt zurück zu Schritt 2
  - Generieren → gleiche URL wie REQ-005/b
- Unit: State-Machine des Wizards (Schritt-Wechsel, Validierung, Zusammenfassung).

## 7. Offene Fragen

- Sollen Schritte überspringbar sein (z. B. Titel ist optional)? Vorschlag: Titel ist in einem optionalen Schritt 4b, überspringbar via „Überspringen"-Link.
- Progressive Fragen — z. B. Altersstufe 3–4 → bestimmt Default-Seitenzahl? Vorschlag: Nein im MVP; Defaults sind statisch, nicht adaptiv.
