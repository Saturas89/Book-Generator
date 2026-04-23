# REQ-013 – TTS-Vorlesemodus

**Status:** 🟢 DRAFT · **Priorität:** COULD · **Modul:** Features

## 1. Ziel / User Story

Als Elternteil will ich ein Buch vorlesen lassen können, wenn ich gerade nicht selbst lesen kann. Erfolg = im Reader gibt es einen „Vorlesen"-Button; der aktuelle Seitentext wird via Web-Speech-API oder (wenn Key vorhanden) über einen KI-TTS-Provider ausgegeben.

## 2. Akzeptanzkriterien

- [ ] Reader zeigt Button `data-testid="reader-tts"`; Klick startet/stoppt die Wiedergabe.
- [ ] Standard: Browser-TTS via `window.speechSynthesis` mit der Buch-Sprache (de/en).
- [ ] Wenn in LocalStorage `bg.apiKeys.tts` ein Key für OpenAI- oder ElevenLabs-TTS liegt, nutzt die App stattdessen den Provider via neue Route `POST /api/ai/tts` (Shape analog zu REQ-005a).
- [ ] Während Wiedergabe ist der aktuelle Satz visuell hervorgehoben.
- [ ] Auto-Advance: Option (LocalStorage `bg.reader.autoAdvance`) schaltet nach Seitenende automatisch zur nächsten.

## 3. Nicht-Ziele

- Keine Stimmpersonalisierung („Papa-Stimme aufnehmen") im MVP.
- Kein Audio-Download.

## 4. Datenmodell / API

Neue Route `POST /api/ai/tts` wie in REQ-005a (Mock liefert statische MP3 aus `public/mock/tts.mp3`). Kein DB-Change.

## 5. UI / UX

Play/Pause-Button in Reader-Kopfzeile; Fortschrittsleiste des aktuellen Satzes.

## 6. Tests

E2E: Button klick → `speechSynthesis.speaking === true` (via `page.evaluate`); Provider-Pfad mit Mock-TTS-Endpoint getestet.

## 7. Offene Fragen

- Welche Provider initial unterstützen? Vorschlag: OpenAI TTS als SHOULD, ElevenLabs als COULD.
