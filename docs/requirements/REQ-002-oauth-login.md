# REQ-002 – OAuth-Login und Sessions

**Status:** 🟢 DRAFT · **Priorität:** MUST · **Modul:** Integrations

## 1. Ziel / User Story

Als Nutzer will ich mich über mein Google- oder GitHub-Konto anmelden, um meine Charaktere und Bücher privat zu behalten. Erfolg = ich klicke „Mit Google anmelden", werde zum Provider umgeleitet, komme angemeldet zurück in die App und sehe die geschützte Bibliothek.

## 2. Akzeptanzkriterien

- [ ] Route `/login` rendert eine Seite mit zwei Buttons: `data-testid="login-google"` und `data-testid="login-github"`.
- [ ] Klick auf einen der beiden startet den Auth.js-OAuth-Flow für den jeweiligen Provider.
- [ ] Erfolgreiche Anmeldung leitet auf `/library` um.
- [ ] Route `/library` ist geschützt: nicht angemeldete Besuche werden auf `/login` umgeleitet.
- [ ] In der Kopfzeile einer angemeldeten Sitzung ist `data-testid="user-menu"` sichtbar und zeigt Name oder E-Mail.
- [ ] Button `data-testid="logout"` beendet die Session und leitet auf `/` um.
- [ ] Prisma-Adapter: `User`, `Account`, `Session`, `VerificationToken` sind als Models im `schema.prisma` ergänzt (Auth.js-Standard).
- [ ] **E2E-Shim**: Wenn `E2E=1` gesetzt ist, ist zusätzlich ein Credentials-Provider aktiv mit den festen Anmeldedaten `email=e2e@test.local`, `password=e2e-test-password`, der einen persistenten Testnutzer einlöst. Dieser Shim ist in Production (`E2E` nicht gesetzt) **inaktiv**.
- [ ] Im E2E-Modus existiert auf `/login` ein zusätzliches Formular mit `data-testid="e2e-login-email"`, `data-testid="e2e-login-password"`, `data-testid="e2e-login-submit"`.
- [ ] Session-Cookie ist `httpOnly` und `secure` in Production.

## 3. Nicht-Ziele

- Kein eigener E-Mail-Passwort-Registrierungsflow für Endnutzer.
- Kein Magic-Link-Login.
- Keine Profil-Editierung über den Namen hinaus (später).
- Keine Mehrfaktor-Authentifizierung.

## 4. Datenmodell / API

```prisma
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  createdAt     DateTime  @default(now())
  accounts      Account[]
  sessions      Session[]
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime
  @@unique([identifier, token])
}
```

Auth.js-Konfiguration in `src/auth.ts` (oder `src/lib/auth.ts`); Middleware schützt unter `src/middleware.ts` alle Routen unter `/library`, `/characters`, `/books`, `/settings`.

## 5. UI / UX

`/login`:
- Titel („Anmelden bei Fancy Fairy")
- Zwei Provider-Buttons vertikal untereinander, beide mit Provider-Icon und Text
- Unter den Providern im E2E-Modus das Testformular (nicht in Production sichtbar)
- Fußnote mit Datenschutz-Hinweis (Platzhalter-Link zur späteren Datenschutzseite)

Kopfzeile (angemeldet): Logo links, Locale-Switch (aus REQ-001) rechts, daneben `user-menu` (Dropdown mit Name + Logout-Button).

## 6. Tests

- Playwright E2E (`e2e/auth.spec.ts`):
  - Unangemeldeter Besuch von `/library` leitet auf `/login`.
  - Mit E2E-Shim-Formular einloggen → Redirect auf `/library`, `user-menu` sichtbar.
  - `logout` klicken → zurück auf `/`.
- Unit (Vitest): Middleware-Redirect-Logik mit gemockter Session.

## 7. Offene Fragen

- Session-Lifetime? Vorschlag: 30 Tage rolling.
- Was soll im `user-menu` stehen, wenn kein Name gesetzt ist? Vorschlag: E-Mail.
- Was passiert, wenn OAuth-Callback fehlschlägt? Vorschlag: Redirect auf `/login?error=oauth_callback_failed`, `ErrorBanner` aus REQ-001a anzeigen.
