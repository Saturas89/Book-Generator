# REQ-009 – Export als PDF und ePub

**Status:** 🟢 DRAFT · **Priorität:** SHOULD · **Modul:** Integrations

## 1. Ziel / User Story

Als Elternteil will ich ein fertiges Buch als PDF (zum Ausdrucken und Archivieren) und als ePub (für E-Book-Reader) herunterladen können. Erfolg = im Reader habe ich zwei Export-Buttons; ein Klick löst den Download einer Datei mit Titel, allen Seiten inkl. Bildern und einem hübschen Cover-Layout aus.

## 2. Akzeptanzkriterien

- [ ] Reader zeigt (nur für Besitzer) Buttons `data-testid="export-pdf"` und `data-testid="export-epub"` (z. B. im Details-Panel).
- [ ] Klick auf „Export PDF" startet den Download einer Datei `{titelSlug}.pdf`:
  - Seite 1 = Cover (Titel groß, Stil-Preset-Hintergrund, Autor/Besitzer-Name)
  - Folgende Seiten: pro Buch-Seite eine PDF-Seite mit Bild oben + Text darunter (oder Text-Only bei `imageDensity=none`)
  - Schriftgröße mindestens 14 pt
  - Format A5 Hochformat
- [ ] Klick auf „Export ePub" startet den Download `{titelSlug}.epub`:
  - Valides ePub3 (`epubcheck`-kompatibel, muss in Tests nicht ausgeführt werden, aber die verwendete Library liefert valides ePub)
  - Metadaten: Titel, Sprache, Erstellungsdatum, Autor (Besitzer-Name)
  - Cover-Bild aus Seite 1
  - Jede Buchseite = ein XHTML-Dokument mit Bild + Text
- [ ] Export läuft serverseitig über eine neue Route `GET /api/books/{id}/export/{format}` (format = `pdf` | `epub`), Besitzer-Check; Response ist die Datei mit passendem `Content-Type` + `Content-Disposition: attachment; filename="..."`.
- [ ] Während Export-Request läuft (serverseitige Generierung ist synchron, dauert ≤ 10 s für 15-seitige Bücher): Button zeigt Spinner, ist disabled.
- [ ] Fehler (Buch nicht `ready`, Asset unreachable) → `<ErrorBanner>` mit Key `error.export_failed`.

## 3. Nicht-Ziele

- Keine eigenen PDF-/ePub-Editoren.
- Keine mehrseitigen Bild-Layouts (ein Bild pro Seite fest).
- Keine Schriftarten-Wahl durch den Nutzer.
- Kein Print-on-Demand (→ REQ-014).
- Keine serverseitige Queue für Batch-Exports.

## 4. Datenmodell / API

Kein Model-Change. Neue Route:
- `GET /api/books/{id}/export/{format}` → Binary-Download
- `format` ∈ `{"pdf", "epub"}`, sonst 400
- 401 unangemeldet, 403 fremdes Buch, 409 wenn `status ≠ "ready"`

Libraries (Vorschlag, nicht bindend):
- PDF: `pdf-lib` oder `@react-pdf/renderer`
- ePub: `epub-gen` oder `epub-press-lib`

## 5. UI / UX

- Export-Buttons im Details-Panel oder Reader-Footer
- Auf Mobile kleine Icons, auf Desktop Icons + Text

## 6. Tests

- Playwright E2E (`e2e/export.spec.ts`) mit `AI_MOCK=1`:
  - Buch generieren (3 Seiten, `imageDensity=every`)
  - Klick „Export PDF" → Download-Event einfangen, Dateigröße > 10 KB, MIME `application/pdf`
  - Klick „Export ePub" → Dateigröße > 10 KB, MIME `application/epub+zip`
  - Export eines fremden Buchs → 403
  - Export eines `generating`-Buchs → 409, Error-Banner
- Unit: ePub-Generator-Funktion erzeugt strukturell gültiges Zip (enthält `mimetype`, `META-INF/container.xml`, OPF).

## 7. Offene Fragen

- Welche PDF-Library liefert auf Vercel-Serverless akzeptable Performance (Cold-Start)? Vorschlag: `pdf-lib` (keine nativen Deps, läuft in Edge/Node stabil). Abhängig von realen Messungen evtl. Swap.
- Soll in ePub jede Seite eigener Kapitel-Marker sein? Vorschlag: ja, für bessere Navigation im Reader.
