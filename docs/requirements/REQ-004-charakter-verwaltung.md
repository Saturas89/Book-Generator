# REQ-004 – Charakter-Verwaltung mit Varianten

**Status:** 🟢 DRAFT · **Priorität:** MUST · **Modul:** Domain

## 1. Ziel / User Story

Als Elternteil will ich wiederverwendbare Charaktere anlegen (z. B. „Emil", „Mia", „Tante Ava") und pro Charakter mehrere Varianten pflegen (z. B. „Emil mit 4", „Emil mit 6"), damit ich beim Anlegen eines Buchs nur die passende Altersvariante auswähle, statt jedes Mal von Null zu beschreiben. Jede Variante trägt optionale Referenzfotos, die später das Bildmodell als visuellen Anker nutzt. Erfolg = ich kann Charaktere und Varianten anlegen, bearbeiten, löschen; Fotos werden zu Vercel Blob hochgeladen; „bookable"-Flag erscheint, sobald ein Charakter mindestens eine Variante hat.

## 2. Akzeptanzkriterien

- [ ] Route `/characters` ist nur angemeldet erreichbar (REQ-002) und zeigt eine Liste der Charaktere des eingeloggten Nutzers.
- [ ] Leere Liste zeigt `<EmptyState>` (REQ-001a) mit CTA `data-testid="create-character-cta"`.
- [ ] Button `data-testid="create-character"` öffnet ein Formular mit Feld `data-testid="character-name"` und Speichern-Button `data-testid="character-save"`. Speichern legt einen `Character` an und leitet auf `/characters/{id}`.
- [ ] `/characters/{id}` zeigt Name (editierbar) und eine Liste der Varianten mit je `data-testid="variant-card"`.
- [ ] Button `data-testid="add-variant"` öffnet ein Varianten-Formular mit Feldern:
  - `data-testid="variant-label"` (Text, z. B. „Emil mit 4")
  - `data-testid="variant-age"` (Zahl, 0–18)
  - `data-testid="variant-description"` (Textarea, max. 2000 Zeichen, Restzähler `data-testid="variant-description-remaining"`)
  - `data-testid="variant-photos"` (File-Input, multiple, akzeptiert nur `image/*`, max. 4 Bilder, max. 5 MB pro Bild)
  - Speichern-Button `data-testid="variant-save"`
- [ ] Beim Speichern werden Fotos clientseitig validiert, zum Next.js-Endpoint `POST /api/blob/upload` gesendet, dort via Vercel Blob signiert hochgeladen; nur die resultierenden `BlobRef`-Objekte werden auf der Variante persistiert.
- [ ] Auf der Varianten-Karte werden Thumbnails der Referenzfotos sichtbar (`data-testid="variant-photo"` pro Bild).
- [ ] Jede Variante hat einen Löschen-Button `data-testid="variant-delete"` mit Bestätigungsdialog `data-testid="confirm-delete"`.
- [ ] Jeder Charakter hat einen Löschen-Button `data-testid="character-delete"`; löst ein Cascade-Delete aller Varianten aus und entfernt hochgeladene Fotos aus Vercel Blob.
- [ ] Charakter zeigt `data-testid="bookable-badge"` mit Text „Bereit für Bücher" / „Ready for books", sobald mindestens eine Variante existiert. Attribut `data-variant-count="{n}"` ist auf diesem Badge gesetzt.
- [ ] Ein Nutzer kann keine Charaktere anderer Nutzer sehen oder bearbeiten (Server prüft `userId` an jeder Route).

## 3. Nicht-Ziele

- Keine KI-generierten Portraits als Referenzfoto im MVP — Referenzfotos sind ausschließlich vom Nutzer hochgeladen.
- Keine Charakter-Vorlagen/Teilen zwischen Nutzern.
- Keine automatische Foto-Bearbeitung (Crop, Rotate) — nur Upload, clientseitige Validierung.
- Keine Versionierung der Beschreibung (keine Historie).

## 4. Datenmodell / API

Prisma-Erweiterung:

```prisma
model Character {
  id        String             @id @default(cuid())
  userId    String
  name      String
  createdAt DateTime           @default(now())
  user      User               @relation(fields: [userId], references: [id], onDelete: Cascade)
  variants  CharacterVariant[]
}

model CharacterVariant {
  id          String    @id @default(cuid())
  characterId String
  label       String
  ageYears    Int
  description String    @db.VarChar(2000)
  photos      Json      // BlobRef[] — siehe unten
  createdAt   DateTime  @default(now())
  character   Character @relation(fields: [characterId], references: [id], onDelete: Cascade)
}
```

`BlobRef`:
```ts
type BlobRef = { url: string; width: number; height: number; mimeType: string }
```

API-Routen (alle unter `/api/characters` + `/api/variants` + `/api/blob`):
- `GET /api/characters` → Liste des eingeloggten Nutzers
- `POST /api/characters` → `{ name }` → neuer Character
- `PATCH /api/characters/:id` → `{ name }`
- `DELETE /api/characters/:id`
- `POST /api/characters/:id/variants` → `{ label, ageYears, description, photos: BlobRef[] }`
- `PATCH /api/variants/:id`
- `DELETE /api/variants/:id`
- `POST /api/blob/upload` → Multipart → `BlobRef[]`; serverseitig Validierung von Mime + Größe

Alle Routen: 401 wenn nicht eingeloggt, 403 wenn Besitzerprüfung fehlschlägt, 400 bei Validierungsfehler (Zod-Schema).

## 5. UI / UX

- Charakter-Liste: Karten-Grid (2 Spalten Mobile, 3–4 Desktop), je Karte Name + Varianten-Anzahl + Bookable-Badge
- Charakter-Detail: Kopfzeile mit Name + Bookable-Badge, darunter Grid der Varianten-Karten, CTA „Variante hinzufügen" unten
- Varianten-Formular als modaler Dialog oder Inline-Form (Impl-Agent wählt)
- Foto-Upload zeigt Lade-Overlay (`<LoadingSpinner>` aus REQ-001a) während Upload
- Fehler werden via `<ErrorBanner>` (REQ-001a) dargestellt

## 6. Tests

- Playwright E2E (`e2e/characters.spec.ts`):
  - Anmelden (E2E-Shim), `/characters` → leer → CTA klickt → Charakter anlegen → Redirect
  - Variante anlegen inkl. Foto-Upload (Fixture-Bild aus `e2e/fixtures/mock-photo.png`)
  - Prüfen: `bookable-badge` sichtbar nach erster Variante, `data-variant-count` = 1
  - Zweite Variante anlegen → `data-variant-count` = 2
  - Variante löschen → Badge bleibt, aber Count = 1
  - Alle Varianten löschen → Badge verschwindet
  - Fremder Nutzer kann diese Charaktere nicht sehen (zweiter Account anmelden)
- Vitest: Zod-Schemas für Variant-Input (ageYears-Range, Description-Länge, Photos-Count).

## 7. Offene Fragen

- Wie viele Bilder maximal pro Variante? Vorschlag: 4 (bei mehr wird die Prompt-Qualität am Bildmodell unzuverlässig).
- Was passiert mit hochgeladenen Bildern beim Charakter-Delete? Vorschlag: synchron Blob-Delete; wenn Blob-Delete fehlschlägt, wird der DB-Eintrag trotzdem entfernt und ein Warn-Log geschrieben.
- Soll es eine Vorschau/Zoom für hochgeladene Fotos geben? Vorschlag: Klick öffnet Foto in Lightbox (out of scope für diese REQ — in REQ-008 Editor folgen).
