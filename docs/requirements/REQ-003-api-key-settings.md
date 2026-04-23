# REQ-003 – API-Key-Settings-UI

**Status:** 🟢 DRAFT · **Priorität:** MUST · **Modul:** Features

## 1. Ziel / User Story

Als angemeldete Nutzerin will ich meine eigenen AI-Provider-Keys einmal pro Gerät im Browser hinterlegen, damit die App in meinem Namen Text- und Bild-KI-Calls machen kann, ohne dass der Betreiber meine Keys jemals persistent sieht. Erfolg = ich öffne `/settings/api-keys`, trage Keys für Text- und Bild-Provider ein, speichere, lade die Seite neu — die Keys sind noch da, und ein Warnbanner macht klar, dass sie nur in diesem Browser leben.

## 2. Akzeptanzkriterien

- [ ] Route `/settings/api-keys` ist nur angemeldet erreichbar (REQ-002).
- [ ] Oben auf der Seite ein Warnbanner (`data-testid="api-keys-warning"`) mit i18n-Text: Keys werden ausschließlich lokal im Browser gespeichert, verlassen den Server nie dauerhaft, müssen pro Gerät neu eingetragen werden.
- [ ] Zwei Formularblöcke „Text-Provider" und „Bild-Provider":
  - Provider-Select (`data-testid="apikey-text-provider"` / `"apikey-image-provider"`) mit festen Optionen: Text = `anthropic` | `openai`, Bild = `fal` | `openai`.
  - Model-Eingabefeld (`data-testid="apikey-text-model"` / `"apikey-image-model"`), Freitext.
  - Key-Eingabefeld (`data-testid="apikey-text-key"` / `"apikey-image-key"`), Typ `password`.
  - Speichern-Button pro Block (`data-testid="apikey-text-save"` / `"apikey-image-save"`).
- [ ] Beim Klick auf Speichern wird ein JSON-Objekt `{ provider, model, key }` unter `localStorage.bg.apiKeys.text` bzw. `localStorage.bg.apiKeys.image` abgelegt.
- [ ] Nach Speichern sichtbare Bestätigung `data-testid="apikey-saved"` (verschwindet nach 3 s).
- [ ] Beim Neuladen der Seite werden Provider+Model aus LocalStorage eingelesen und in den Feldern angezeigt; das Key-Feld bleibt leer, zeigt aber einen Hinweis `data-testid="apikey-text-has-saved"` bzw. `"apikey-image-has-saved"` („Key gespeichert"), solange im LocalStorage ein Key existiert.
- [ ] „Löschen"-Button pro Block (`data-testid="apikey-text-clear"` / `"apikey-image-clear"`) entfernt den LocalStorage-Eintrag.
- [ ] Ungültige Eingaben (leerer Key, leeres Model) werden per `<TextField>`-Fehlermeldung (REQ-001a) angezeigt; kein Speichern erfolgt.

## 3. Nicht-Ziele

- Keine Verschlüsselung im LocalStorage (Reiner Schutz vor anderen Nutzern desselben Rechners liegt im Verantwortungsbereich des OS-Users).
- Keine Server-seitige Persistenz der Keys.
- Kein Import/Export von Key-Sets.
- Kein automatisches Testen der Keys gegen die Provider-APIs („Ping").
- Keine Provider-spezifische Ratenbegrenzung hier — wird in REQ-005a auf Proxy-Ebene behandelt.

## 4. Datenmodell / API

LocalStorage-Schlüssel (feste Namen, dürfen später nur additiv erweitert werden):

```
bg.apiKeys.text  → { "provider": "anthropic", "model": "claude-sonnet-4-5", "key": "sk-..." }
bg.apiKeys.image → { "provider": "fal", "model": "flux/dev", "key": "..." }
```

Keine Server-Routes in dieser REQ.

## 5. UI / UX

- Titel „API-Schlüssel" / „API Keys"
- Warnbanner (Gelb/Amber, Info-Icon)
- Zwei separate Karten untereinander
- Responsive: auf Mobile volle Breite, auf Desktop max. 600 px
- Fokus-Reihenfolge: Provider → Model → Key → Speichern

## 6. Tests

- Playwright E2E (`e2e/api-keys.spec.ts`):
  - Key eintragen → speichern → Bestätigung sichtbar
  - Seite neu laden → Provider+Model vorausgefüllt, „Key gespeichert"-Hinweis sichtbar
  - LocalStorage prüfen (via `page.evaluate`) — JSON-Shape wie in §4
  - Löschen → LocalStorage-Eintrag entfernt
- Vitest: Hilfsfunktionen `readApiKey(kind)`, `writeApiKey(kind, value)`, `clearApiKey(kind)` werden isoliert getestet (Happy + Invalid-JSON).

## 7. Offene Fragen

- Welche Text-Provider/-Modelle sollen standardmäßig vorausgewählt sein? Vorschlag: `anthropic` + `claude-sonnet-4-5`.
- Soll beim Eintragen eines neuen Keys der alte Key sofort überschrieben werden oder Bestätigungsdialog? Vorschlag: überschreiben ohne Dialog, ein Hinweis „Ersetzt vorhandenen Key" reicht.
