# REQ-006 – Eigene Bibliothek und In-App-Reader

**Status:** 🟢 DRAFT · **Priorität:** MUST · **Modul:** UI

## 1. Ziel / User Story

Als Elternteil will ich meine fertiggestellten Bücher an einem Ort wiederfinden und sie in der App mit meinem Kind gemeinsam lesen — Seite für Seite, groß, ohne Ablenkung, mobil und am Desktop. Erfolg = `/library` zeigt meine Bücher als Cover-Grid; ein Klick öffnet den Reader mit einer Seite pro Bildschirm, navigierbar per Button, Tastatur und Touch-Swipe.

## 2. Akzeptanzkriterien

- [ ] Route `/library` ist nur angemeldet erreichbar.
- [ ] Bibliothek zeigt alle Bücher des eingeloggten Nutzers als Grid `data-testid="library-grid"`, sortiert nach `createdAt` descending.
- [ ] Jede Buchkachel `data-testid="library-card"` enthält:
  - Cover-Bild (`imageUrl` von Seite 1, sonst Placeholder-Cover mit Stil-Preset-Farbe)
  - Titel (`data-testid="library-card-title"`)
  - Sprache-Badge (DE/EN)
  - Altersstufe-Badge
  - Status-Badge, falls ≠ `ready` (z. B. „In Arbeit" für `generating`, „Fehler" für `error`)
- [ ] Leere Bibliothek: `<EmptyState>` aus REQ-001a mit CTA „Erstes Buch erstellen" → `/books/new`.
- [ ] Klick auf eine Kachel öffnet `/library/{bookId}` (Reader).
- [ ] Reader (`/library/{bookId}`):
  - Zeigt eine Seite pro Bildschirm, Seitenanzeiger `data-testid="reader-page-indicator"` („3 / 8").
  - Seitentext `data-testid="reader-page-text"`, Bild (falls vorhanden) `data-testid="reader-page-image"`.
  - Next-Button `data-testid="reader-next"`, Prev-Button `data-testid="reader-prev"`.
  - Tastaturnavigation: Rechts/Space = next, Links = prev.
  - Touch-Swipe auf Mobile: horizontale Swipes wechseln die Seite.
  - Zurück-Button `data-testid="reader-back"` führt auf `/library`.
  - Der Reader respektiert die `language` des Buchs: Alt-Texte, Seitenindikator („Seite 3 von 8" / „Page 3 of 8").
- [ ] Tief-Links funktionieren: `/library/{bookId}?page=3` öffnet direkt Seite 3.
- [ ] Fremde Nutzer bekommen 404, wenn sie `/library/{fremdeBookId}` öffnen.
- [ ] Reader hat klaren Focus-Ring auf Next/Prev für Tastatur-Nutzer; auf Mobile volle Höhe, keine Sticky-Header.

## 3. Nicht-Ziele

- Kein Editieren von Seiten (→ REQ-008).
- Keine Exportbuttons (→ REQ-009).
- Kein Share-Button (→ REQ-011).
- Keine Ordner/Filter (→ REQ-012).
- Kein Vorlesen (→ REQ-013).

## 4. Datenmodell / API

Keine neuen Models. Reine Read-Operationen:
- `GET /api/books` → Liste der Bücher des Nutzers, je mit Metadaten + Cover-URL
- `GET /api/books/{id}` → Buch inkl. Pages, 403/404 bei fremder id

Sortierung kann serverseitig oder clientseitig stattfinden; serverseitig empfohlen.

## 5. UI / UX

- Bibliothek: Grid 2 Spalten Mobile, 3 Desktop, 4 XL
- Kachel-Aspekt 3:4, Cover oben, Titel + Badges unten
- Reader: Split oben Bild (max 60 % Viewport), unten Text mit großer Schrift (≥ 20 px), großzügiger Zeilenabstand
- Bei `imageDensity=none` zeigt der Reader keine Bildfläche, sondern Text-Only in einem zentrierten Layout

## 6. Tests

- Playwright E2E (`e2e/library-and-reader.spec.ts`):
  - Nach Generierung eines Buchs (Mock-Mode) erscheint die Kachel in `/library`
  - Klick öffnet Reader auf Seite 1
  - Next → Seite 2; Prev → Seite 1; Tastatur Rechts → Seite 2; Swipe (Playwright `page.touchscreen`) → Seite wechselt
  - Tief-Link `?page=3` öffnet Seite 3 direkt
  - Fremder Nutzer kommt nicht in ein Buch
- Unit: Hilfskomponente `<ReaderPage>` rendert Text und Bild korrekt je `imageDensity`.

## 7. Offene Fragen

- Soll es eine Buchdetail-Seite zusätzlich zum Reader geben (mit Metadaten + Delete)? Vorschlag: Ja, als Info-Button in Reader-Kopfzeile → Slide-In Panel „Details" mit Meta + Delete. In dieser REQ als optionales Nice-to-have belassen; wenn der Impl-Agent das nicht liefert, kommt es in einer Folge-REQ.
- Soll der Reader die gelesene Seite merken (Fortsetzen)? Vorschlag: Ja, LocalStorage `bg.reader.lastPage.{bookId}`, wird beim Öffnen angewandt, solange kein `?page=`-Param gesetzt ist.
