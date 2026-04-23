# Anforderungen

Alle funktionalen und nicht-funktionalen Anforderungen des Projekts. Jede Datei unter `REQ-0XX-<slug>.md` beschreibt ein abgeschlossenes Feature oder Querschnittsthema und wird von der Parallel-Generation-Pipeline (`.github/workflows/parallel-generation.yml`) aufgegriffen, sobald sie auf einem Feature-Branch gepusht wird.

---

## 📑 Übersichtstabelle (Vorlage)

Pro Projekt hier eine Tabelle aller REQs pflegen. Leer zu Beginn:

| ID | Titel | Modul | Priorität | Status |
|----|-------|-------|-----------|--------|
| [REQ-001](./REQ-001-projekt-bootstrap.md) | Projekt-Bootstrap (Next.js 15, Tests, i18n) | Core | MUST | 🟢 DRAFT |
| [REQ-001a](./REQ-001a-design-system-und-fehlerzustaende.md) | Design-System, Fehler-/Leerzustände, a11y-Basis | UI | MUST | 🟢 DRAFT |
| [REQ-002](./REQ-002-oauth-login.md) | OAuth-Login (Google/GitHub) + Sessions | Integrations | MUST | 🟢 DRAFT |
| [REQ-003](./REQ-003-api-key-settings.md) | API-Key-Settings-UI (LocalStorage) | Features | MUST | 🟢 DRAFT |
| [REQ-004](./REQ-004-charakter-verwaltung.md) | Charakter-Verwaltung mit Varianten | Domain | MUST | 🟢 DRAFT |
| [REQ-005](./REQ-005-buch-schema-und-formular.md) | Buch-Schema + One-Shot-Generierungsformular | Features | MUST | 🟢 DRAFT |
| [REQ-005a](./REQ-005a-ai-proxy-und-mock-mode.md) | AI-Proxy-Routes + Mock-Mode | Integrations | MUST | 🟢 DRAFT |
| [REQ-005b](./REQ-005b-generierungs-pipeline.md) | Generierungs-Pipeline + Persistenz | Features | MUST | 🟢 DRAFT |
| [REQ-006](./REQ-006-bibliothek-und-reader.md) | Eigene Bibliothek + In-App-Reader | UI | MUST | 🟢 DRAFT |
| [REQ-007](./REQ-007-dialog-wizard.md) | Dialog-Wizard (ersetzt One-Shot, Flag-gesteuert) | Features | SHOULD | 🟢 DRAFT |
| [REQ-008](./REQ-008-seiten-editor.md) | Seiten-Editor: Text + Bild neu würfeln | Features | SHOULD | 🟢 DRAFT |
| [REQ-009](./REQ-009-export-pdf-epub.md) | Export als PDF und ePub | Integrations | SHOULD | 🟢 DRAFT |
| [REQ-010](./REQ-010-content-moderation.md) | Content-Moderation | Integrations | SHOULD | 🟢 DRAFT |
| [REQ-011](./REQ-011-buch-teilen.md) | Buch-Teilen via Read-only-Link | Features | SHOULD | 🟢 DRAFT |
| [REQ-012](./REQ-012-ordner-regale.md) | Ordner / Regale für Bibliothek | UI | COULD | 🟢 DRAFT |
| [REQ-013](./REQ-013-tts-vorlesemodus.md) | TTS-Vorlesemodus | Features | COULD | 🟢 DRAFT |
| [REQ-014](./REQ-014-print-on-demand.md) | Print-on-Demand-Integration | Integrations | COULD | 🟢 DRAFT |

---

## 🎯 MoSCoW-Priorisierung

Neue Anforderungen werden nach MoSCoW priorisiert:

### MUST
Kern-Funktionalität ohne die das Produkt seinen Zweck nicht erfüllt.

### SHOULD
Wichtige, aber nicht blockierende Ergänzungen zum Kern.

### COULD
Nice-to-have — wenn Zeit bleibt.

### WON'T (diese Iteration)
Bewusst ausgeschlossen — begründen, damit die Entscheidung später nachvollziehbar ist.

---

## 🧾 Spec-Vorlage

Neue Specs folgen dieser Gliederung:

```md
# REQ-0XX – <Titel>

**Status:** 🟢 DRAFT · **Priorität:** MUST | SHOULD | COULD · **Modul:** <Modul>

## 1. Ziel / User Story
Worum geht es, wer profitiert, wodurch ist Erfolg messbar?

## 2. Akzeptanzkriterien
- [ ] Jedes Kriterium so formuliert, dass ein Test es direkt prüfen kann.
- [ ] Selektoren/IDs (falls UI) eindeutig aus dem Wording ableitbar.

## 3. Nicht-Ziele
Was explizit **nicht** Teil dieser Spec ist.

## 4. Datenmodell / API (falls relevant)
Felder, Typen, Versionierung, Rückwärtskompatibilität.

## 5. UI / UX (falls relevant)
Skizze der Flows, Texte, erwartetes Verhalten auf Fehlern.

## 6. Tests
Welche Testarten (Unit, Component, E2E), welche Edge-Cases.

## 7. Offene Fragen
Gezielt adressieren, bevor die Parallel-Generation startet — sonst weichen die beiden Agents bei Ambiguität auseinander.
```

---

## 🔒 Globales Prinzip: Rückwärtskompatibilität

Jedes Update persistierter Nutzerdaten muss abwärtskompatibel sein:

- Neue Felder in `localStorage` / IndexedDB / DB-Schema sind immer **optional** und haben Defaults.
- Bestehende Feldnamen und Speicherschlüssel werden **nicht umbenannt oder entfernt**.
- Backup-/Import-Formate erhalten bei strukturellen Änderungen eine neue Versionsnummer; der Import-Handler füllt fehlende Felder mit Defaults.

---

## 📊 Status-Legende

| Symbol | Status | Bedeutung |
|--------|--------|-----------|
| 🟢 | DRAFT | In Planung / Konzept |
| 🟡 | REVIEW | Zur Überprüfung bereit |
| ✅ | APPROVED | Genehmigt, noch nicht implementiert |
| 🔵 | IN PROGRESS | Teilweise implementiert |
| ✔️ | COMPLETED | Vollständig implementiert |
| 🔴 | DEPRECATED | Verworfen |
