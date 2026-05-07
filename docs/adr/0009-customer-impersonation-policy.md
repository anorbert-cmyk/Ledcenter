# ADR-0009: Customer impersonation policy

**Dátum:** 2026-05-07
**Státusz:** accepted
**Kapcsolódó task:** T7.9b
**Deadline:** Fázis 0 vége
**Kapcsolódó gap:** G-10 (P0)

## Kontextus

A "Login as customer" admin-feature gyakori e-commerce eszköz support célra: az admin betekint a vásárló nézetébe, hogy reprodukáljon egy hibát vagy ellenőrizzen egy panaszt. **De ez egy security-risk:**

- Kompromittált admin = korlátlan customer-impersonation
- GDPR cikk 4 (1) "pseudonymisation" érintettség — adatszivárgás potenciál
- Audit-log nélkül nem rekonstruálható, ki, mikor, mit látott

A build spec NEM említette explicit ezt a feature-t, de implicit kockázat: ha valaki "kényelmi okból" hozzáadja, NEM lesz audit-log, time-limited token, separate session.

## Megfontolt opciók

### Opció A: TILTOTT v1-ben (default)

Az admin UI-ban NINCS „Login as customer" gomb. A support-ügyek customer-screenshot vagy DevTools-Network-trace alapján kezelve.

**Lint-szabály:** keres `impersonate`, `loginAs`, `actAsCustomer` minta-előfordulást, hibát dob ha talál (`eslint.config.mjs` + pre-commit hook).

### Opció B: Engedélyezett, audit-log + biztonsági réteggel

- **Time-limited token:** max 15 perc érvényes session
- **Audit log minden akcióra:** mit látott, mit klikkelt, mit módosított — `audit_log` táblába
- **Banner a customer felületén:** „Admin-impersonation aktív — `<admin neve>`, `<dátum>`"
- **Customer email-értesítés UTÓLAG:** „A támogatási csapat <dátum>-kor megnézte a fiókodat ennél a problémánál: <ticket-id>"
- **NEM session-takeover:** az admin az eredeti admin-session-t megtartja egy szerver-oldali context-tel (NEM cookie-csere)

### Opció C: Engedélyezett, csak read-only

Csak GET request-ek, NEM POST/PUT/DELETE. Még egyszerűbb, de kevésbé támogatja a use case-t.

## Döntés

**Választott opció: A — TILTOTT v1-ben.**

M2-ben a B opció bevezethető, ha a customer-support workflow ezt indokolja.

## Indoklás

| Szempont | A (TILTOTT) | B (audit-log) | C (read-only) |
|---|---|---|---|
| Security risk | minimum | közepes (kompromittált admin) | alacsony |
| GDPR-konform | igen | igen (audit-log) | igen |
| Implementációs költség | 0 (lint-szabály) | M+ (audit-log infra, time-token, banner, email) | S (read-only middleware) |
| Customer support workflow | screenshot / DevTools elég v1-ben | jobb | korlátozott |
| **Karpathy #2 (egyszerűség)** | ✓ | nem (extra infra) | félig |

**v1 launchhoz** a customer support volumen kicsi (Norbi + barátok), screenshot-alapú megközelítés elég. M2-ben (T+60 nap után, lásd post-launch roadmap), ha a support-volumen indokolja, B opció bevezetése külön task-ként.

## A challenger érveire adott válaszok

**ELLEN-érv 1 — "A barátok már most kérni fogják, mert kényelmesebb":**
**Válasz:** screenshot + DevTools-trace 90% support-esetre elég. Ha kényelmesebb opció kell, a `B`-vel készülünk M2-re. v1-ben a security trumps convenience.

**ELLEN-érv 2 — "Lint-szabály körülmegy lehet (változó-elnevezés, dynamic import)":**
**Válasz:** a lint csak az első védelmi vonal. A code-reviewer agent is keresi explicit. Plusz a security-auditor (T7.1) audit-ja minden new feature-nél lefut.

**ELLEN-érv 3 — "Mi van, ha egy ügyfél elveszít egy számlát és segítsünk megtalálni?":**
**Válasz:** Norbi az admin-on saját jogosultsággal letöltheti a számlát és emailben elküldheti. A vásárlói nézetbe való belépés NEM szükséges ehhez. M2-ig van Loom-videós support workflow.

## Következmények

- v1 kódbázisban NINCS impersonation-related route vagy gomb.
- **Lint-szabály** (`eslint.config.mjs` + pre-commit hook):
  ```javascript
  'no-restricted-syntax': ['error', {
    selector: "Identifier[name=/^(impersonate|loginAs|actAsCustomer)$/]",
    message: 'Customer impersonation tiltva v1-ben (ADR-0009).'
  }]
  ```
- T7.9b task lezárva (impersonation-feature NEM épül).
- M2-re tervezett: B opció (time-limited token + audit-log + banner + email).

## Frissítések

- 2026-05-07: első kiadás, accepted.
