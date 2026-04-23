# REQ-005 – Buch-Schema und One-Shot-Generierungsformular

**Status:** 🟢 DRAFT · **Priorität:** MUST · **Modul:** Features

## 1. Ziel / User Story

Als Elternteil will ich ein neues Buch in einem einzigen Formular konfigurieren (welche Charakter-Variante, welche Altersstufe, welcher Illustrationsstil, worum geht's, wie lang), damit ich die Generierung mit einem Klick anstoßen kann, ohne durch viele Dialog-Schritte zu klicken. Erfolg = ich öffne `/books/new`, fülle das Formular aus, sende ab, und die Anfrage landet an einem definierten Endpoint, der sie an die Generierungs-Pipeline (REQ-005b) weiterreicht.

## 2. Akzeptanzkriterien

- [ ] Route `/books/new` ist nur angemeldet erreichbar (REQ-002).
- [ ] Formular-Felder:
  - Charakter-Variante: Select `data-testid="book-variant"`, gruppiert nach Charakter; nur „bookable"-Charaktere des Nutzers (REQ-004).
  - Altersstufe: Radio-Gruppe `data-testid="book-age-band"` mit festen Werten `3-4`, `5-6`, `7-8`.
  - Stilpreset: Select `data-testid="book-style"` mit festen Slugs: `aquarell`, `pixar-3d`, `scherenschnitt`, `strichzeichnung`, `anime`.
  - Thema/Prompt: Textarea `data-testid="book-topic"` (max. 500 Zeichen, Restzähler `data-testid="book-topic-remaining"`).
  - Seitenzahl: Slider oder Number-Input `data-testid="book-page-count"` (Range 5–15, Default 8).
  - Bilderdichte: Radio `data-testid="book-image-density"` mit Werten `none`, `title-only`, `every`, `every-other` (Default `every`).
  - Sprache: Radio `data-testid="book-language"` mit Werten `de`, `en` (Default = aktuelle UI-Locale).
  - Titel (optional): Textfeld `data-testid="book-title"` — wenn leer, generiert die Pipeline einen.
- [ ] Submit-Button `data-testid="book-generate"` sendet `POST /api/books` mit dem validierten Payload.
- [ ] Validierung clientseitig via Zod; jedes Feld mit Fehler zeigt Fehlermeldung über `<TextField>`-Fehlerslot (REQ-001a).
- [ ] Erfolgreicher `POST /api/books` (Status 201) leitet auf `/books/{id}/generating` um (Generierungs-Fortschrittsscreen aus REQ-005b).
- [ ] Wenn keine Text- oder Bild-API-Keys im LocalStorage sind (REQ-003), wird über dem Submit-Button ein Warnbanner `data-testid="missing-keys-warning"` angezeigt und der Button ist deaktiviert.
- [ ] Wenn der Nutzer keinen einzigen „bookable"-Charakter hat, zeigt die Seite `<EmptyState>` mit CTA zu `/characters`.
- [ ] `POST /api/books` erstellt einen Book-Eintrag mit Status `pending` und returned `{ id }`.

## 3. Nicht-Ziele

- Keine Generierungslogik selbst (→ REQ-005b).
- Keine Proxy-Routen zu den AI-Providern (→ REQ-005a).
- Kein Dialog-Wizard (→ REQ-007, hinter `DIALOG_MODE`-Flag).
- Keine Bibliotheks-/Übersichts-UI (→ REQ-006).
- Keine Cover-Generierung in dieser REQ.

## 4. Datenmodell / API

Prisma-Erweiterung:

```prisma
model Book {
  id           String    @id @default(cuid())
  userId       String
  variantId    String
  title        String
  language     String    // "de" | "en"
  ageBand      String    // "3-4" | "5-6" | "7-8"
  stylePreset  String
  imageDensity String    // "none" | "title-only" | "every" | "every-other"
  topic        String    @db.VarChar(500)
  status       String    @default("pending") // "pending" | "generating" | "ready" | "error"
  createdAt    DateTime  @default(now())
  user         User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  variant      CharacterVariant @relation(fields: [variantId], references: [id])
  pages        Page[]
}

model Page {
  id       String  @id @default(cuid())
  bookId   String
  index    Int     // 1..n
  text     String  @db.Text
  imageUrl String?
  book     Book    @relation(fields: [bookId], references: [id], onDelete: Cascade)
  @@unique([bookId, index])
}
```

`POST /api/books` Payload:

```ts
{
  variantId: string
  ageBand: "3-4" | "5-6" | "7-8"
  stylePreset: string
  topic: string
  pageCount: number   // 5..15
  imageDensity: "none" | "title-only" | "every" | "every-other"
  language: "de" | "en"
  title?: string
}
```

Antwort: `201 { id: string }`.

Server validiert mit Zod, verifiziert `variantId` gehört dem Nutzer.

## 5. UI / UX

- Ein einziger durchscrollbarer Formular-Screen
- Varianten-Select zeigt Label + Alter („Emil mit 4 (4 J.)")
- Stil-Select zeigt neben dem Slug eine Mini-Vorschau (statisches Beispielbild je Preset, als statische Asset-Datei im Repo)
- Bilderdichte: 4 Radiobuttons mit erklärendem Text je Option
- Submit-Button bleibt fixiert am unteren Rand auf Mobile
- Alle Felder sind Pflicht außer „Titel"

## 6. Tests

- Playwright E2E (`e2e/book-form.spec.ts`):
  - Anmelden, mindestens eine Variante anlegen (aus REQ-004 bekannt), API-Keys setzen (REQ-003)
  - `/books/new` besuchen, Formular ausfüllen, absenden
  - Redirect auf `/books/{id}/generating` (URL-Regex prüfen)
  - Leerer API-Key-Zustand: Warnbanner, Button disabled
  - Keine „bookable"-Charaktere: Empty State
- Vitest: Zod-Schema für Book-Create akzeptiert/lehnt alle Randwerte ab.

## 7. Offene Fragen

- Wie sollen die Stil-Vorschauen aussehen? Vorschlag: Impl-Agent legt 5 einfache SVG-Platzhalter unter `public/styles/` ab, mit passendem Alt-Text.
- Soll die Sprache aus der UI-Locale vor-ausgewählt werden? Vorschlag: Ja, aber umschaltbar.
- Soll ein „Entwurf speichern"-Modus möglich sein? Nein im MVP — Formular ist zustandslos bis zum Submit.
