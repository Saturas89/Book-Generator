# REQ-010 – Content-Moderation

**Status:** 🟢 DRAFT · **Priorität:** SHOULD · **Modul:** Integrations

## 1. Ziel / User Story

Als Elternteil muss ich mich darauf verlassen können, dass keine angsteinflößenden, gewalttätigen oder explizit unpassenden Inhalte generiert werden — weder auf Prompt-Seite (mein Thema-Input) noch in der KI-Ausgabe. Erfolg = Input wird vor dem Senden an den AI-Provider gegen eine deterministische Policy geprüft; generierte Texte werden nach dem Empfang erneut geprüft; Verletzungen werden durch klare Fehlermeldungen sichtbar, die Pipeline liefert **kein** problematisches Ergebnis aus.

## 2. Akzeptanzkriterien

- [ ] Eine Policy-Datei `src/content-moderation/policy.json` existiert mit:
  - `blockedKeywords`: Array aus Strings (deutsch + englisch), case-insensitive Match auf Worte/Phrasen
  - `ageBandKeywords`: Objekt `{ "3-4": string[], "5-6": string[], "7-8": string[] }` — Worte, die für diese Altersstufe **zu komplex/beängstigend** sind
  - `maxPromptLength`: Int (Default 500, konsistent mit REQ-005)
- [ ] Pre-Moderation (im `POST /api/ai/text`-Handler aus REQ-005a, vor dem Provider-Call):
  - Prüft `topic` gegen `blockedKeywords` → bei Treffer `403 { error: "content_blocked", matched: "keyword" }`.
  - Prüft Länge → bei Überschreitung `400 { error: "invalid_input" }`.
- [ ] Post-Moderation (im `POST /api/ai/text`-Handler, nach Provider-Response):
  - Jede generierte Seite wird gegen `blockedKeywords` + `ageBandKeywords[ageBand]` geprüft.
  - Bei Treffer: **gesamte Antwort verwerfen**, `403 { error: "content_blocked", matched: "keyword", page: n }`.
- [ ] Pre-Moderation für Bild-Prompts (`POST /api/ai/image`): analog mit `blockedKeywords`, aber ohne Altersstufen-Check (Bilder nutzen keinen ageBand im Payload).
- [ ] In REQ-005b wird ein Content-Block-Fehler als prominenter `<ErrorBanner>` angezeigt mit i18n-Text `error.content_blocked` und Hinweis „Bitte Thema anpassen".
- [ ] Die Policy ist in CI via Fixtures getestet:
  - Eingaben aus `e2e/fixtures/moderation-blocked.txt` → `content_blocked`
  - Eingaben aus `e2e/fixtures/moderation-ok.txt` → durchgelassen
- [ ] Keine Abhängigkeit von externen Moderation-APIs im MVP (deterministisch, CI-tauglich).
- [ ] Option für Impl-Agent, einen Provider-Moderation-Adapter zusätzlich zu aktivieren (z. B. OpenAI `omni-moderation-latest`) **nur wenn ein Moderation-Key in LocalStorage unter `bg.apiKeys.moderation` vorhanden ist**. Wenn nicht, läuft ausschließlich die lokale Policy.

## 3. Nicht-Ziele

- Keine subjektive „ist-das-kindgerecht"-Bewertung — rein deterministisch über Keyword-Listen + optionale Provider-API.
- Keine Lern-/Adaptions-Logik.
- Keine Admin-UI zum Editieren der Policy — die Datei wird über PRs gepflegt.
- Keine Bild-Output-Moderation (nur Bild-**Prompt**).

## 4. Datenmodell / API

Kein neues Persistenz-Model. Policy-Datei ist reiner Code-Asset.

Fehler-Shape wie in REQ-005a:
```json
{ "error": "content_blocked", "matched": "<keyword>", "page": 3 }
```

Optionaler LocalStorage-Key: `bg.apiKeys.moderation` → `{ provider: "openai", model: "omni-moderation-latest", key: "..." }`. In REQ-003 als **optionaler** dritter Block ergänzen, falls Impl-Agent diesen Zusatz nimmt — sonst nur in REQ-010 dokumentiert.

## 5. UI / UX

- Content-Block-Fehler im Generating-Screen (REQ-005b): rot getönter `<ErrorBanner>` mit festem i18n-Text
- Im Settings-UI (`/settings/api-keys`) optional ein dritter Block „Moderation (optional)" — aber nur wenn Provider-Adapter aktiv

## 6. Tests

- Vitest (`src/content-moderation/__tests__/policy.test.ts`):
  - Jeder Eintrag aus `moderation-blocked.txt` wird geblockt
  - Jeder Eintrag aus `moderation-ok.txt` geht durch
  - Altersstufen-Check: gleicher Text für `3-4` blockt, für `7-8` geht durch (wenn das Keyword in `ageBandKeywords["3-4"]` aber nicht `["7-8"]` steht)
- Integration-Test am Proxy (`src/app/api/ai/text/__tests__`):
  - Mock-Mode + geblockter Topic → 403 `content_blocked` **vor** Mock-Response
- Playwright E2E (`e2e/moderation.spec.ts`):
  - Formular mit geblocktem Topic absenden → Generating-Screen zeigt Error-Banner, Buch-Status = `error`

## 7. Offene Fragen

- Mit welchen Keyword-Seeds starten? Vorschlag: kurze kuratierte Liste mit offensichtlichen Fällen (Gewalt, explizite Sexualität, Drogen, bekannte Hass-Symbole), bewusst konservativ, wird über Nutzung iterativ ergänzt.
- Sollen Treffer für Nutzer transparent sein („Wort X geblockt") oder nur generisch? Vorschlag: generisch in der UI („Thema nicht erlaubt"), aber matched-Keyword im Log + Response-Body (für Debugging).
