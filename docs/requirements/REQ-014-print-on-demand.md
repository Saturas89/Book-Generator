# REQ-014 – Print-on-Demand-Integration

**Status:** 🟢 DRAFT · **Priorität:** COULD · **Modul:** Integrations

## 1. Ziel / User Story

Als Elternteil will ich ein fertiges Buch als gedrucktes Hardcover oder Softcover bestellen können, um es zu verschenken oder zu archivieren. Erfolg = aus dem Reader heraus lässt sich ein Bestellvorgang bei einem Print-on-Demand-Anbieter (Lulu, Blurb) starten; die App bereitet das druckfähige PDF (REQ-009) auf und überträgt es an den Anbieter.

## 2. Akzeptanzkriterien

- [ ] Reader zeigt Button `data-testid="book-print"` (nur Besitzer).
- [ ] Klick öffnet einen Dialog mit Anbieter-Auswahl, Format (A5/A4), Cover-Typ (Softcover/Hardcover) und geschätzter Preisberechnung.
- [ ] Bestätigung leitet auf eine vom Anbieter gehostete Checkout-Seite um (API-Integration nach Provider-Doku) **oder** erzeugt eine Download-Datei im exakten Druck-Spec des Anbieters (Bleed, CMYK) zum manuellen Hochladen, je nach Anbieter-Support.
- [ ] Bestellungen werden als `PrintOrder { id, bookId, provider, status, createdAt }` persistiert, Status-Updates via Webhook oder manuellem Refresh.

## 3. Nicht-Ziell

- Keine eigene Bezahlabwicklung.
- Keine Rabatt-/Gutschein-Logik.
- Keine internationale Steuer-/Zollberechnung in der App.

## 4. Datenmodell / API

Siehe `PrintOrder`-Model (§2). Neue Routes: `POST /api/books/{id}/print`, `GET /api/print-orders`, Webhook `POST /api/print/webhook/{provider}`.

## 5. UI / UX

Mehrstufiger Dialog mit Vorschau; Preisauszeichnung in Landeswährung (basierend auf UI-Locale).

## 6. Tests

E2E mit Anbieter-Mock-Mode (`PRINT_MOCK=1`): Druck-Flow geht bis zur Bestätigungsseite, `PrintOrder` ist in DB.

## 7. Offene Fragen

- Welche Anbieter zuerst? Vorschlag: Lulu (öffentliche API, gute Doku).
- Druck-Spec (CMYK, Bleed) beeinflusst Bildgenerierung — sollen generierte Bilder nachträglich umgefärbt werden oder schon beim Generieren in passendem Format entstehen? Vorschlag: nachträgliche serverseitige Konvertierung (kein Eingriff in den Haupt-Generierungsflow).
- Rechtsthemen (Verbraucherrechte, Altersfreigabe) sind explizit **nicht Teil dieser Spec**.
