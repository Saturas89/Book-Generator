# REQ-012 – Ordner / Regale für die Bibliothek

**Status:** 🟢 DRAFT · **Priorität:** COULD · **Modul:** UI

## 1. Ziel / User Story

Als Elternteil mit mehreren Kindern möchte ich meine Bücher in Ordner/Regale gruppieren (z. B. „für Emil", „für Mia", „Gute-Nacht"), damit die Bibliothek bei vielen Büchern übersichtlich bleibt.

## 2. Akzeptanzkriterien

- [ ] Neue Model-Entität `Shelf { id, userId, name, createdAt, order }`.
- [ ] Verknüpfungstabelle `BookShelf { bookId, shelfId }` (ein Buch darf in mehreren Regalen liegen).
- [ ] Bibliothek (REQ-006) zeigt oben eine horizontale Regal-Leiste `data-testid="shelf-strip"`; jedes Regal `data-testid="shelf-{id}"` öffnet gefilterte Ansicht.
- [ ] „Alle Bücher" als erstes virtuelles Regal (nicht löschbar).
- [ ] CRUD für Regale in `/settings/shelves` mit `data-testid="shelf-create"`, `shelf-rename`, `shelf-delete`.
- [ ] Auf Buch-Karte Drag-Drop oder „Zu Regal hinzufügen"-Menü `data-testid="book-add-to-shelf"`.

## 3. Nicht-Ziele

- Keine geteilten Regale zwischen Nutzern.
- Keine Auto-Zuweisung (z. B. nach Charakter-Name).

## 4. Datenmodell / API

Siehe §2. Routes: `GET/POST /api/shelves`, `PATCH/DELETE /api/shelves/{id}`, `POST /api/books/{id}/shelves`, `DELETE /api/books/{id}/shelves/{shelfId}`.

## 5. UI / UX

Horizontale Chip-Leiste; aktives Regal farblich hervorgehoben.

## 6. Tests

Playwright E2E: Regal anlegen, Buch zuweisen, Filter greift, Regal löschen räumt Zuweisungen auf.

## 7. Offene Fragen

- Maximale Regal-Anzahl? Vorschlag: 20 pro Nutzer, soft limit mit Hinweis.
- Reihenfolge der Regale manuell sortierbar? Vorschlag: ja, Drag-Drop im Settings-UI.
