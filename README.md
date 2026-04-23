# Book-Generator

Startgerüst für den Book-Generator. Dieses Repo enthält zunächst nur die wiederverwendbaren Arbeitsweisen (CI-Pipeline, Agenten-Prompts, Git-Workflow, Doku-Struktur) und **noch keinen produktspezifischen Code**. Framework-Wahl (Vite/React/…), `package.json` und erste Specs folgen in eigenen Schritten.

## Was hier drin ist

| Bereich | Datei(en) | Zweck |
|---------|-----------|-------|
| Git- & PR-Workflow | [`CLAUDE.md`](./CLAUDE.md) | Regeln für Claude-Sessions: immer PR, nie direkt auf `main`, CI-Polling-Rezept |
| CI-Pipeline | [`.github/workflows/e2e.yml`](./.github/workflows/e2e.yml) | Playwright-Matrix (5 Browser-Projekte) auf jedem PR |
| Parallel-Generation | [`.github/workflows/parallel-generation.yml`](./.github/workflows/parallel-generation.yml) | Triggert bei `docs/requirements/REQ-*.md`-Push zwei isolierte Claude-Agents (Impl + Test) parallel |
| Agenten-Prompts | [`.github/prompts/impl-agent.md`](./.github/prompts/impl-agent.md), [`.github/prompts/test-agent.md`](./.github/prompts/test-agent.md) | Black-Box-Isolation: Impl kennt Tests nicht, Test kennt Code nicht |
| Coding-Vorgaben | [`docs/guides/CONTRIBUTING.md`](./docs/guides/CONTRIBUTING.md) | Branch-Strategie, Commit-Konventionen, Code-Stil |
| Doku-Struktur | [`docs/requirements/README.md`](./docs/requirements/README.md), [`docs/modules/README.md`](./docs/modules/README.md) | Vorlagen für REQ-Specs (MoSCoW) und Architektur-Übersicht |
| Dependency-Updates | [`renovate.json`](./renovate.json) | Wöchentliche Renovate-PRs, Gruppierung nach Ökosystem |
| Node-Version | [`.nvmrc`](./.nvmrc) | Einheitliche Node-Version für alle Umgebungen |

## Was hier bewusst **noch nicht** drin ist

- Kein `src/`, kein `package.json`, keine Build-/Runtime-Configs — kommen, sobald der Tech-Stack gewählt ist.
- Keine Requirements/Specs — nur die Vorlagen dafür.
- Keine Assets (Logos, Icons, Manifeste).

## Nächste Schritte

1. Framework-Scaffolding initialisieren (`npm init`, Vite/Framework-Template, `package.json`, tsconfig).
2. Playwright + Vitest installieren, damit die CI-Workflows grün laufen (Pipeline erwartet `npm ci`, `npx playwright test`, `npm test`).
3. Secret `CLAUDE_CODE_OAUTH_TOKEN` im Repo setzen, falls der Parallel-Generation-Workflow genutzt werden soll.
4. Erste Spec unter `docs/requirements/REQ-001-<slug>.md` ablegen und pushen — der Parallel-Generation-Workflow startet dann automatisch.
5. `CLAUDE.md`, `impl-agent.md`, `test-agent.md` bei Bedarf an den konkreten Tech-Stack anpassen (Framework-Namen, Testbefehle, Stil-Hinweise). Die **Isolations-Regeln** (Agent sieht Gegenseite nicht) sollten bleiben.

## Arbeitsablauf (TL;DR)

```
Spec schreiben  →  push  →  Parallel-Generation-Workflow
                              ├─ Impl-Agent (nur Code)
                              └─ Test-Agent (nur Tests, aus Spec)
                           →  gemeinsamer Commit auf Feature-Branch
                           →  Auto-PR gegen main
                           →  E2E-Matrix (5 Browser) entscheidet grün/rot
```
