# REQ-005b – Generierungs-Pipeline und Persistenz

**Status:** 🟢 DRAFT · **Priorität:** MUST · **Modul:** Features

## 1. Ziel / User Story

Als Elternteil will ich nach dem Absenden des Buchformulars (REQ-005) auf einem Fortschrittsscreen sehen, wie mein Buch Seite für Seite entsteht, und am Ende ein fertiges, persistentes Buch haben, das ich in der Bibliothek wiederfinde. Erfolg = nach dem Submit starte ich direkt in `/books/{id}/generating`, sehe pro Seite einen Fortschritts-Eintrag, und sobald alle Seiten fertig sind, werde ich auf den Erfolgsscreen mit Link ins Buch geleitet.

## 2. Akzeptanzkriterien

- [ ] Route `/books/{id}/generating` ist nur angemeldet erreichbar, nur der Besitzer darf sie öffnen.
- [ ] Beim Öffnen der Seite startet clientseitig die Pipeline:
  1. Ruft `/api/ai/text` genau einmal auf (mit Text-Key aus LocalStorage) und erhält Titel + alle Seitentexte.
  2. Speichert Titel via `PATCH /api/books/{id}` und alle Seiten via `POST /api/books/{id}/pages` (atomar pro Seite).
  3. Iteriert Seite 1..n: falls die Bilderdichte für diese Seite ein Bild vorsieht, ruft `/api/ai/image` mit (a) Seitentext als Prompt-Kontext, (b) Stil-Preset, (c) Referenz-URLs aus der gewählten Charakter-Variante.
  4. Speichert `imageUrl` pro Seite via `PATCH /api/pages/{id}`.
  5. Setzt am Ende `Book.status = "ready"`.
- [ ] Bilderdichte-Regeln:
  - `none` → kein `/api/ai/image`-Call
  - `title-only` → nur für Seite 1 ein Bild
  - `every` → für jede Seite ein Bild
  - `every-other` → für ungerade Seiten (1, 3, 5, …) ein Bild
- [ ] UI auf `/books/{id}/generating`:
  - Titel der Seite: „Buch wird erstellt…"
  - Gesamtbalken `data-testid="generation-progress-bar"` mit `aria-valuenow`
  - Seitenliste `data-testid="generation-page-list"`, pro Seite ein Eintrag `data-testid="generation-page-{n}"` mit Status-Icon (pending|done|error)
  - Abbruch-Button `data-testid="generation-cancel"` — setzt `Book.status="error"` und navigiert zur Bibliothek.
- [ ] Bei Fehler einer Seite:
  - Seiteneintrag zeigt Fehlermeldung (via Error-Taxonomie aus REQ-005a)
  - Button `data-testid="generation-retry-page-{n}"` versucht genau diese Seite neu
  - Pipeline bricht **nicht** automatisch die gesamte Generierung ab — andere Seiten laufen weiter.
- [ ] Nach vollständiger Generierung: Erfolgsscreen mit `data-testid="generation-success"`, einem Link „Buch öffnen" → `/library/{bookId}` (aus REQ-006) und einem Link „Zurück zur Bibliothek" → `/library`.
- [ ] Alle AI-Calls laufen durch den Client-Wrapper `fetchAI(kind, payload)`, der Key aus LocalStorage als `x-api-key`-Header setzt.
- [ ] `Book.status`-Feld reflektiert korrekt `pending` → `generating` → `ready` (oder `error`).

## 3. Nicht-Ziele

- Kein serverseitiger Job-Queue / Worker — die Orchestrierung läuft **im Browser** des Nutzers, weil der Key nie den Server verlässt. Schließt der Nutzer den Tab, bleibt das Buch im Zustand `generating` — Recovery über „Fortsetzen"-Button in der Bibliothek kommt in späterer REQ.
- Kein Echtzeit-Push-Update (SSE/WebSocket) — Fortschritt wird lokal durch die Pipeline-Schritte selbst gesetzt.
- Kein Cover-Bild separat — das Titelbild ergibt sich aus Seite 1 (bei `title-only` oder `every`/`every-other`).
- Keine Seiten-Editierung (→ REQ-008).

## 4. Datenmodell / API

Ergänzende Routes:
- `PATCH /api/books/{id}` → `{ title?, status? }`
- `POST /api/books/{id}/pages` → `{ index, text }` → `201 { id }`
- `PATCH /api/pages/{id}` → `{ text?, imageUrl? }`

Alle Routes: Besitzer-Check; 401/403/400 wie in REQ-005a.

Client-Wrapper `fetchAI`:
```ts
async function fetchAI<K extends "text"|"image">(
  kind: K, payload: InputFor<K>
): Promise<OutputFor<K>> {
  const ls = localStorage.getItem(`bg.apiKeys.${kind}`)
  const { key } = ls ? JSON.parse(ls) : { key: "" }
  const res = await fetch(`/api/ai/${kind}`, {
    method: "POST",
    headers: { "content-type": "application/json", "x-api-key": key },
    body: JSON.stringify(payload)
  })
  if (!res.ok) throw new AIError((await res.json()).error)
  return res.json()
}
```

## 5. UI / UX

- Zentrierter Progress-Bereich, ca. 600 px max. Breite
- Seitenliste vertikal, jeder Eintrag 48–64 px hoch mit Status-Icon links und „Seite n: …"-Titel
- Erfolgsscreen: großzügig, kurze Gratulation, primärer CTA „Buch öffnen"
- Bei Offline/Netzwerkfehler: `<ErrorBanner>` (REQ-001a) mit Retry

## 6. Tests

- Playwright E2E (`e2e/book-generation.spec.ts`) mit `AI_MOCK=1`:
  - Formular ausfüllen (REQ-005) → Generating-Screen
  - Warten, bis `generation-success` sichtbar — max. 30 s
  - Prüfen: Jede Seite in `generation-page-list` hat Status „done"
  - `MOCK_PAGE_n`-Text und Mock-Bild-URLs sind auf dem resultierenden Buch persistiert (Zugriff via `/library/{id}` aus REQ-006)
- Variante pro `imageDensity`:
  - `none` → keine Bilder persistiert, alle `page.imageUrl` null
  - `title-only` → nur Seite 1 hat Bild
  - `every-other` → Seite 1, 3, 5 haben Bilder
- Fehler-Simulation: Test-Modus-Schalter (Header `x-mock-error: rate_limit`) sorgt für Fehler bei einer Seite, Retry-Flow wird getestet.

## 7. Offene Fragen

- Soll die Pipeline parallelisieren (mehrere Bild-Calls gleichzeitig)? Vorschlag: max. 2 parallel, weil Provider-Rate-Limits sonst häufig auslösen.
- Was passiert, wenn der Nutzer den Tab verlässt? Vorschlag: Buch verbleibt in `generating`, ein Banner in der Bibliothek zeigt „Unterbrochen" und bietet „Weitermachen" an (→ Folge-REQ).
- Wie wird der Bild-Prompt pro Seite formuliert? Vorschlag: Der Text-Endpoint liefert zu jeder Seite einen zusätzlichen `imagePrompt`-Vorschlag; wenn das Textmodell das nicht tut, fällt der Client auf „Seitentext + Stil + Charakterbeschreibung" zurück. Das exakte Feld wird in REQ-005a präzisiert, wenn Entscheidung fällt.
