# REQ-005a – AI-Proxy-Routes und Mock-Mode

**Status:** 🟢 DRAFT · **Priorität:** MUST · **Modul:** Integrations

## 1. Ziel / User Story

Als Systemarchitektin will ich einen dünnen Server-Proxy vor den AI-Providern, damit (a) die Nutzer-Keys nicht im Client-Netzwerkverkehr an die Provider-URLs sichtbar werden, (b) Fehler einheitlich normalisiert zurückfließen, (c) in CI ohne echte API-Calls deterministische Fixtures geliefert werden können. Erfolg = zwei POST-Endpoints `/api/ai/text` und `/api/ai/image` nehmen Key per Header, delegieren an austauschbare Provider-Adapter und antworten in einem festgelegten Shape; mit `AI_MOCK=1` liefern sie Fixtures ohne echte Calls.

## 2. Akzeptanzkriterien

- [ ] Route `POST /api/ai/text` existiert, ist nur angemeldet erreichbar (REQ-002) und nimmt Payload:
  ```json
  { "provider": "anthropic"|"openai", "model": "string", "language": "de"|"en",
    "ageBand": "3-4"|"5-6"|"7-8", "stylePresetHint": "string", "topic": "string",
    "pageCount": 5..15, "title": "string?" }
  ```
  plus Header `x-api-key: <nutzer-key>`.
- [ ] Antwort bei Erfolg: `200 { "title": "string", "pages": [{ "n": 1, "text": "..." }, ...] }`.
- [ ] Route `POST /api/ai/image` existiert, nimmt Payload:
  ```json
  { "provider": "fal"|"openai", "model": "string",
    "prompt": "string", "stylePreset": "string",
    "referenceUrls": ["https://..."]? }
  ```
  plus Header `x-api-key`.
- [ ] Antwort bei Erfolg: `200 { "url": "https://..." }` — URL zeigt auf ein persistiertes Bild (Vercel Blob).
- [ ] Fehler-Taxonomie (alle Endpoints): `400 { error: "invalid_input" }`, `401 { error: "invalid_key" }`, `403 { error: "content_blocked" }`, `429 { error: "rate_limit" }`, `503 { error: "provider_down" }`, `500 { error: "generic" }`. Kein Stacktrace im Response-Body.
- [ ] **Mock-Mode**: Wenn `process.env.AI_MOCK === "1"`:
  - `/api/ai/text` liefert ohne externen Call: `{ "title": "MOCK_TITLE", "pages": [{ n: 1, text: "MOCK_PAGE_1" }, ..., { n: pageCount, text: "MOCK_PAGE_${pageCount}" }] }`.
  - `/api/ai/image` liefert: `{ "url": "/mock/page-${hash}.png" }`, wobei `hash` = deterministischer Hash aus `prompt`, gemappt auf eines der in `public/mock/` eingecheckten Platzhalterbilder (`page-1.png` … `page-5.png`).
  - Key-Header wird im Mock-Mode **ignoriert**, aber Payload-Validierung läuft trotzdem.
- [ ] Provider-Adapter sind als Interface implementiert, sodass ein weiterer Provider später ohne Änderung der Route-Handler ergänzt werden kann.
- [ ] Keyless-Request ohne `AI_MOCK=1` → `401 invalid_key`.
- [ ] In Logs erscheint der Key nie; geloggt werden Provider, Model, Status, Latenz.

## 3. Nicht-Ziele

- Keine Orchestrierung mehrerer Seitenanfragen in einer einzigen HTTP-Anfrage (→ REQ-005b).
- Kein Streaming (SSE/WebSocket) — Antworten sind JSON-Batch. Fortschritt wird in REQ-005b durch Seiten-weise separate Calls + Polling ausgedrückt.
- Keine Content-Moderation (→ REQ-010) — der Proxy reicht den Content durch.
- Keine Persistenz der Antworten selbst — das erledigt REQ-005b.

## 4. Datenmodell / API

Siehe §2. Zusätzlich:

- Provider-Adapter-Interface in `src/lib/ai/adapters/*`:
  ```ts
  interface TextAdapter { generate(input: TextInput, key: string): Promise<TextOutput> }
  interface ImageAdapter { generate(input: ImageInput, key: string): Promise<ImageOutput> }
  ```
- Mock-Contract für CI wird hier verbindlich festgelegt und ist die **einzige Quelle**, aus der Impl- und Test-Agent später ableiten.
- Eingecheckte Fixtures: `public/mock/page-1.png` … `public/mock/page-5.png` (einfache farbige Placeholder-Bilder, jeweils 512×512, <30 KB).
- LocalStorage-Keys werden hier nicht gelesen — der Client schickt den Key als Header. Dies wird in REQ-005b über den Client-Wrapper `fetchAI(kind, payload)` (`src/lib/ai/client.ts`) umgesetzt.

## 5. UI / UX

Nicht relevant — reine API-Ebene.

## 6. Tests

- Vitest/Integration-Tests (`src/app/api/ai/__tests__/*.test.ts`):
  - Mit `AI_MOCK=1`: Text-Endpoint liefert korrekte `pages`-Länge und `MOCK_PAGE_n` für alle n.
  - Mit `AI_MOCK=1`: Image-Endpoint liefert eine URL, die unter `public/mock/` existiert.
  - Ohne `AI_MOCK=1`, ohne Header → 401.
  - Ungültiger Payload (pageCount=0) → 400.
- Playwright E2E wird nicht direkt gegen diese Routes geschrieben — indirekt via REQ-005b.

## 7. Offene Fragen

- Welches konkrete fal-Modell für den initialen Bild-Adapter? Vorschlag: `fal-ai/flux-pro/kontext` (unterstützt Referenzbild-Input).
- Welches Anthropic-Modell als Default? Vorschlag: `claude-sonnet-4-5`.
- Wie soll der Rate-Limit-Fehler vom Provider gemappt werden, wenn der Provider stattdessen 503 liefert? Vorschlag: explizite Mappings im Adapter; unbekannte 5xx → `provider_down`.
