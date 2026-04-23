# REQ-008 – Seiten-Editor: Text ändern und Bild neu würfeln

**Status:** 🟢 DRAFT · **Priorität:** SHOULD · **Modul:** Features

## 1. Ziel / User Story

Als Elternteil will ich bei einem fertig generierten Buch einzelne Seiten nachbessern können — den Text umformulieren, wenn eine Passage nicht passt, oder das Bild neu würfeln, wenn die Illustration nicht trifft — ohne das ganze Buch neu generieren zu müssen. Erfolg = im Reader ist pro Seite ein Bearbeiten-Modus; ich ändere den Text und speichere, oder klicke „Neu würfeln" und erhalte ein neues Bild zur selben Seite.

## 2. Akzeptanzkriterien

- [ ] Im Reader (REQ-006) existiert pro Seite ein Button `data-testid="page-edit"` (nur für den Besitzer sichtbar).
- [ ] Klick öffnet den Edit-Modus:
  - Textarea `data-testid="page-edit-text"` mit aktuellem Text (vorbelegt, max. 1000 Zeichen).
  - Speichern-Button `data-testid="page-edit-save"` → `PATCH /api/pages/{id}` mit `{ text }`.
  - Abbrechen-Button `data-testid="page-edit-cancel"` → verwirft Änderungen.
- [ ] „Bild neu würfeln"-Button `data-testid="page-reroll-image"`:
  - Nur sichtbar, wenn die Seite ein Bild hat (oder die Bilderdichte eines erlaubt — `every`, `every-other` auf passender Seite, `title-only` auf Seite 1).
  - Ruft `/api/ai/image` erneut auf mit demselben Prompt + Stil-Preset + Referenzfotos und einem zusätzlichen Seed-Parameter (`seed = randomInt`).
  - Während der Request läuft, zeigt `<LoadingSpinner>` (REQ-001a) über dem Bild.
  - Bei Erfolg wird `imageUrl` via `PATCH /api/pages/{id}` aktualisiert.
  - Bei Fehler: `<ErrorBanner>` mit Retry-Button.
- [ ] Nicht-Besitzer (auch Leser mit Share-Link aus REQ-011) sehen keine Edit-Buttons; Server lehnt `PATCH` mit 403 ab.
- [ ] Änderungen sind sofort persistent, ohne Bestätigungsdialog.
- [ ] Undo: nach Speichern eines Text-Edits erscheint 10 s lang ein Toast `data-testid="page-edit-undo"` mit „Rückgängig"-Button; Klick stellt den vorherigen Text wieder her (einstufig, kein History-Stack).

## 3. Nicht-Ziele

- Keine Volltext-Buchbearbeitung (kein Reihenfolge-Ändern, kein Seiten-Einfügen/-Löschen — könnte in Folge-REQ).
- Keine KI-gestützte Textumformulierung („mach es kürzer") — nur manuelles Editieren in diesem Schritt.
- Kein Rich-Text, nur Plain Text.
- Kein Upload eigener Bilder als Seitenbild (nur Reroll via Bild-Provider).
- Keine parallele Bearbeitung durch mehrere Sitzungen (Last-Write-Wins).

## 4. Datenmodell / API

Kein neues Model. Nutzt bestehende:
- `PATCH /api/pages/{id}` → `{ text?: string, imageUrl?: string }`, Besitzer-Check auf Book→User.
- `POST /api/ai/image` (REQ-005a) mit zusätzlichem optionalem Feld `seed: number`; dieser wird an den Provider-Adapter weitergereicht, wenn der Provider Seeds unterstützt; sonst im Prompt als Suffix („seed: 12345") mitgegeben.

## 5. UI / UX

- Edit-Modus ist eine lokale Umschaltung im Reader, keine neue Route.
- Während Text-Edit ist Seitennavigation deaktiviert, bis gespeichert/abgebrochen.
- Reroll-Button sitzt als Floating-Button über dem Bild, rechts oben.
- Toast erscheint unten mittig (mobile) bzw. unten rechts (desktop).

## 6. Tests

- Playwright E2E (`e2e/page-editor.spec.ts`) mit `AI_MOCK=1`:
  - Buch aus dem Generieren-Flow geöffnet → Edit-Mode aktiviert → Text ändern → speichern → Reload, Text ist persistiert
  - Reroll-Button klickt, neues Mock-Bild wird persistiert
  - Undo-Toast wiederherstellt alten Text
  - Zweiter Nutzer (Besitzer-Check): kein Edit-Button sichtbar, `PATCH` direkt → 403
- Unit: Reducer/State für Reader-Edit-Mode (enter, save, cancel).

## 7. Offene Fragen

- Wie lange soll der Undo-Toast sichtbar sein? Vorschlag: 10 s.
- Soll der Reroll-Button einen Zähler anzeigen („Versuch 2")? Vorschlag: nein, aber jeder Reroll-Aufruf loggt serverseitig; keine Begrenzung in dieser REQ (Content-Moderation in REQ-010 stellt Schutz).
