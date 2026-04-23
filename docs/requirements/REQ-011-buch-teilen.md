# REQ-011 – Buch-Teilen via Read-only-Link

**Status:** 🟢 DRAFT · **Priorität:** SHOULD · **Modul:** Features

## 1. Ziel / User Story

Als Elternteil will ich ein selbst generiertes Buch mit anderen registrierten Nutzern der App teilen können (z. B. mit Oma, die einen eigenen Account hat), damit sie es in ihrer Bibliothek lesen — aber nicht editieren oder weiter verteilen. Erfolg = ich klicke „Teilen" auf einem Buch, bekomme einen Link; wenn ein anderer angemeldeter Nutzer diesen Link öffnet, erscheint das Buch in dessen Bibliothek unter „Geteilt mit mir" und ist lesbar, aber nicht editierbar.

## 2. Akzeptanzkriterien

- [ ] Reader zeigt (nur für Besitzer) einen Button `data-testid="book-share"`.
- [ ] Klick öffnet einen Dialog:
  - Eingabefeld `data-testid="share-email"` (optional — wenn gesetzt, wird der Link nur für diesen Account gültig)
  - Erzeugen-Button `data-testid="share-create"` → `POST /api/books/{id}/shares`; Response enthält Share-Token und vollständige URL
  - Angezeigte URL `data-testid="share-url"` mit Copy-Button `data-testid="share-copy"`
- [ ] Liste aktueller Shares pro Buch `data-testid="share-list"` mit Revoke-Button je Eintrag `data-testid="share-revoke"`.
- [ ] Share-URL hat Form `/shared/{token}`.
- [ ] `/shared/{token}` (unangemeldet) → Redirect auf `/login?next=/shared/{token}`.
- [ ] `/shared/{token}` (angemeldet, Token gültig):
  - Prüft, ob Share an E-Mail gebunden ist und der eingeloggte Nutzer diese E-Mail hat — wenn ja, Zugang; wenn nein, 403.
  - Legt in der DB einen `BookShareAccess`-Eintrag an (idempotent), der das Buch in der Bibliothek des Lesers unter einem „Geteilt mit mir"-Abschnitt zeigt.
  - Leitet auf `/library/{bookId}` weiter.
- [ ] Bibliothek (REQ-006) zeigt „Geteilt mit mir" als zusätzlichen Grid-Bereich mit eigenem `data-testid="library-shared-grid"`.
- [ ] Geteilte Bücher sind im Reader voll lesbar, aber kein `page-edit`, `page-reroll-image`, `export-*`-Button sichtbar (außer der Besitzer nutzt denselben Account).
- [ ] Revoke entfernt den Share-Eintrag; Leser verliert Zugriff; Buch verschwindet aus dessen Bibliothek.
- [ ] Share-Token ist URL-safe, mindestens 24 Zeichen Entropie; ausgelaufen nach 365 Tagen (hart codiert).

## 3. Nicht-Ziele

- Kein öffentliches Teilen (ohne Account).
- Kein „Fork" / Kopieren eines geteilten Buches in die eigene Bibliothek als Bearbeitbares.
- Keine Benachrichtigungen (E-Mail, Push) beim Teilen.
- Keine Kommentar-/Reaktions-Funktion.
- Kein Granularitätsgrad "nur bestimmte Seiten" — immer das ganze Buch.

## 4. Datenmodell / API

Prisma-Erweiterung:

```prisma
model BookShare {
  id        String   @id @default(cuid())
  bookId    String
  token     String   @unique
  email     String?  // wenn gesetzt, nur dieser Account darf einlösen
  createdAt DateTime @default(now())
  expiresAt DateTime
  book      Book     @relation(fields: [bookId], references: [id], onDelete: Cascade)
  accesses  BookShareAccess[]
}

model BookShareAccess {
  id        String   @id @default(cuid())
  shareId   String
  userId    String
  createdAt DateTime @default(now())
  share     BookShare @relation(fields: [shareId], references: [id], onDelete: Cascade)
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@unique([shareId, userId])
}
```

`User`-Relation `sharedWithMe` via `BookShareAccess` um Bibliothek zu joinen.

Routes:
- `POST /api/books/{id}/shares` → `{ email? }` → `{ token, url }`
- `GET /api/books/{id}/shares` → Liste eigener Shares für dieses Buch
- `DELETE /api/shares/{id}` → revoke
- `GET /shared/{token}` (Page-Route) → Einlöse-Logik

## 5. UI / UX

- Share-Dialog als modaler Overlay im Reader
- Share-URL in monospaced Input-Feld, Copy-Button kopiert in Clipboard
- In Bibliothek: Abschnitts-Header „Eigene Bücher" + „Geteilt mit mir"; Letzterer nur sichtbar, wenn ≥ 1 Share

## 6. Tests

- Playwright E2E (`e2e/share.spec.ts`) mit `AI_MOCK=1`:
  - Besitzer erstellt Share, kopiert URL
  - Zweiter Nutzer (via E2E-Shim mit anderer E-Mail anmelden — Annahme: Shim unterstützt mehrere Fixture-Accounts) öffnet URL → Redirect, Buch in „Geteilt mit mir"
  - Zweiter Nutzer öffnet Reader → keine Edit-Buttons, Export-Buttons fehlen
  - Besitzer revokiert Share → zweiter Nutzer verliert Zugriff (erneutes Öffnen der URL → 410/403)
- Unit: Token-Generator liefert mindestens 24 Zeichen, URL-safe.

## 7. Offene Fragen

- Soll das E2E-Shim mehr als einen festen Account erlauben? Vorschlag: REQ-002 erweitern um zweiten Fixture-Nutzer `e2e-friend@test.local`.
- Sollen abgelaufene Shares automatisch gelöscht werden? Vorschlag: nein, nur invalidiert; Cron-Cleanup später.
- Soll der Teilende per Mail benachrichtigt werden, wenn jemand die Freigabe einlöst? Vorschlag: nicht im MVP.
