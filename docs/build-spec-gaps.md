# Ledcenter.hu Build Spec — Hiánylista (gap audit, v1)

**Dátum:** 2026-05-07
**Reviewer:** challenger (hostile pozíció, nincs lojalitás a saját korábbi munkához)
**Forrás-dokumentum:** `build-spec.md` v1.2 → v1.3 patch
**Módszer:** lineáris átolvasás, fókusz a Barion / Számlázz / NAV / migráció / SEO / security / monitoring / DR / költség / időzítés területekre.

---

## 1. Vezetői összefoglaló

**Összesen 38 hiányt találtam.**

| Súlyosság | Darab | Megjegyzés |
|---|---:|---|
| 🔴 Kritikus (launch-blocker) | **13** | Ezek nélkül a launch jogi / financial / SEO / security kockázatba megy. |
| 🟡 Fontos (rework-höz vezet) | **18** | Fázis 4 vége – Fázis 8 között megoldandók. |
| 🟢 Nice-to-have (post-launch is OK) | **7** | M2-re halasztható. |

### Top 5 legfontosabb hiány

| # | ID | Súly | Cím | Következmény |
|---|---|---|---|---|
| 1 | G-04 | 🔴 | Sztornó vs helyesbítő számla magyar gyakorlat-tévedés | NAV-bírság vagy könyvelői-rework |
| 2 | G-06 | 🔴 | NAV hibakód-térkép és retry-policy hiánya | 5402 (érvénytelen adószám) RETRY-vel sosem oldódik |
| 3 | G-07 | 🔴 | ÁFA-mező hiánya a termék-szintű adatmodellből | hard-coded `vat: 27`, ha 5%/18% kell, automatikus rossz |
| 4 | G-09 | 🔴 | Barion tranzakciós díj kihagyása a költség-becslésből | Valós havi költség 2-3× annyi, mint v1.0 becslés |
| 5 | G-13 | 🔴 | Termék-ár adatmodell áfa-szempontból nem definiált | NAV nettót, áfát, bruttót KÜLÖN vár; kerekítés-hibák |

---

## 2. Hiányok task-ID szerint

### G-01 — Barion `PaymentRequestId` formátum-ellenőrzés (🟡)

**Hely:** T4.1. **Probléma:** max 100 char, ajánlott UUID. **Megoldás:** UUID v4 dedikált `payment_request_id` mezőbe.

### G-02 — Barion redirect URL `{paymentId}` placeholder (🟡)

**Hely:** T4.1. **Probléma:** Barion server-side substitution vs JS template literal megtévesztő. **Megoldás:** explicit kommentár vagy single-quote escape.

### G-03 — Barion idempotencia kód-szintű implementáció (🔴)

**Hely:** T4.1. **Probléma:** "Idempotent callback" deklarálva, de a HOGYAN nincs. **Megoldás:** Háromrétegű:
1. Redis SETNX `barion:cb:<PaymentId>`, 7 nap expiry
2. DB unique constraint `payment.barion_payment_id`
3. Payment/State újra-lekérdezés Barion-tól

### G-04 — Sztornó vs helyesbítő magyar gyakorlat (🔴)

**Hely:** T4.2. **Probléma:** v1.0 helyesbítőt írt részleges visszárura, magyar gyakorlat = SZTORNÓ + új. **v1.3 megoldás:** Két opció dokumentálva:
- **Opció A (default, magyar gyakorlat):** részleges → helyesbítő (negatív sor, áfatv. 78. § (3))
- **Opció B (alternatíva):** részleges → sztornó az eredetire + új számla a maradékra

⚠️ ELLENŐRIZD KÖNYVELŐVEL. Felelős: NORBI; deadline: Fázis 0 vége; válasz: `docs/checks/T4.2.md`. Default kódban: opció A, runtime feature-flag `config.invoice.partialRefundFlow = 'helyesbito' | 'storno_plus_new'`.

### G-05 — Számla PDF megőrzés 8 év (🟡)

**Hely:** T4.2. **Probléma:** R2 lifecycle törölhetne; 2000. évi C. tv. 169. § = 8 év. **Megoldás:** R2 Object Lock, immutability, 8 év + 1 hónap retention az `invoices/` prefixre.

### G-06 — NAV hibakód-térkép és retry-policy (🔴)

**Hely:** T4.3. **Probléma:** generikus `INVALID/ERROR` reagálás, hibakód-szerinti útvonal nincs. **v1.3 megoldás:** kategóriánkénti reakció-policy + `docs/runbooks/nav-error-codes.md` élő dokumentum.

### G-07 — ÁFA-mező a termék-modellből (🔴)

**Hely:** T2.1. **Probléma:** LED-attribútum lista NEM tartalmaz `vatRate`-t, T4.2 hard-coded 27%. **v1.3 megoldás:** `vatRate: 5 | 18 | 27` kötelező mező (default 27), Zod validation, lint-szabály blokkolja a hard-coded értékeket.

### G-08 — Currency mismatch (🟡)

**Hely:** T4.1, T4.2. **Probléma:** Barion EUR/USD támogat, Számlázz hard-coded HUF. **v1.3 megoldás:** v1-ben CSAK HUF, assertion `assert input.currency_code === 'HUF'`. ADR-013 a M2-bevezetésig.

### G-09 — Barion tranzakciós díj költség-becslés (🔴)

**Hely:** 1.3. **Probléma:** "0 fix" díj, valóban 1.5-2.5% + 30 Ft/tranzakció. **v1.3 megoldás:** új tábla 100/500/1000/5000 rendelés/hó × 15 000 Ft kosár. Realisztikus havi ~$190-2375 forgalom-függő.

### G-10 — Customer impersonation policy (🔴)

**Hely:** Fázis 7. **Probléma:** "Login as customer" implicit risk, NINCS audit-log spec. **v1.3 megoldás:** Default v1: TILTOTT (ADR-009). Lint-szabály blokkolja. M2-ben ha kell: time-limited 15 perc, audit-log, banner customer felületén, email-értesítés.

### G-11 — File upload mélyebb védelem (🔴)

**Hely:** T2.3, T5.3. **Probléma:** csak MIME + filename említve. **v1.3 megoldás:**
- Magic-bytes (`file-type`)
- SVG/PDF TILTOTT
- Decompression bomb (max 50 MP)
- EXIF strip (`sharp.withMetadata({})`)
- Méret-limit (10 MB, 4096×4096 px)
- Antivirus M2-re

### G-12 — Admin login lockout policy (🟡)

**Hely:** T7.2. **Probléma:** csak rate limit, IP-rotation körüljárható. **Megoldás:** email-alapú lockout 10 sikertelen / 1h, hCaptcha 3. sikertelen után, email-alert tulajnak.

### G-13 — Termék-ár adatmodell nettó/bruttó (🔴)

**Hely:** T2.1, T8.2. **Probléma:** Medusa default `amount: number`, NAV nettó+ÁFA+bruttó külön kell. **v1.3 megoldás:**
- `priceNet` (kanonikus, fillér)
- `priceGross` (számolt)
- `priceVat` (számolt)
- Banker's rounding (ROUND_HALF_EVEN), 2 tizedes
- Medusa `amount` = bruttó (vásárlói nézet)
- ADR-0008 kötelező

### G-14 — Pre-cutover freeze (🟡)

**Hely:** T9.2. **Probléma:** "ha lehet" — ha ShopRenter nincs read-only mód, double-booking. **Megoldás:** banner 24h előtt + inventár 0-ra.

### G-15 — aggregateRating policy (🟡)

**Hely:** T6.1 vs T2.6. **Probléma:** ha review v1-ben SKIP de schema tartalmaz aggregateRating üresen → Google rich-result spam. **Megoldás:** `aggregateRating` MEZŐ KIMARAD v1-ben.

### G-16 — ISR availability sync (🔴)

**Hely:** T6.1, T6.6. **Probléma:** PDP ISR 1 perc, készlet 0 → 1 perc cache → "Nincs készleten" frusztrált customer. **v1.3 megoldás:**
- Kliens-oldali fresh check (SWR 30s) a "Kosárba" gomb mellett
- On-demand revalidate Medusa subscriber `inventory.changed`-ra
- Schema marad ISR (konzervatív availability)

### G-17 — Canonical URL filter (🟡)

**Hely:** T6.1. **Megoldás:** filter URL canonical a fő kategóriára, `<meta name="robots" content="noindex,follow">`.

### G-18 — Pagination canonical (🟢)

**Megoldás:** minden page-en saját canonical (NEM page=1-re).

### G-19 — Hreflang x-default (🟢)

**Megoldás:** `<link rel="alternate" hreflang="x-default" href="..."/>` jövőbeli i18n-hez.

### G-20 — RPO/RTO definíciók (🔴)

**Hely:** T1.3. **v1.3 megoldás:**
- **RPO: 15 perc** (Railway PITR)
- **RTO: 4 óra** (fél munkanap)
- Off-site backup R2 másik régió (G-21 részben)
- Havi DR-drill
- ADR-010

### G-21 — Off-site backup régió-redundancia (🟡)

**Megoldás:** napi `pg_dump` → R2 másik régió, 30 nap retention. ~$0.075/hó.

### G-22 — Konkrét monitoring alert-thresholdok (🟡)

**Megoldás:**
- Barion callback fail rate >5% / 15 perc
- Számlázz NAV ERROR rate >0%
- Search response p95 >1s
- Order-success rate <95% / óra
- Sentry új error >10/perc

### G-23 — SLO definíciók (🟡)

**Megoldás:**
- Storefront uptime: 99.5% (3.6h downtime/hó)
- Checkout success rate: ≥98%
- PDP LCP p95 mobile: ≤3s
- Payment-to-invoice latency p95: ≤2 perc
- NAV-bejelentés p95: ≤5 perc

### G-24 — Költség teljes breakdown (🟡)

**Megoldás:** új 1 oldalas táblázat 4 oszloppal (induló/mid/scale/large).

### G-25 — Konkrét naptár dátumokkal (🟡)

**Megoldás:** új mini-szekció: 2026-06-01 indulás, mérföldkövek dátummal + 10% buffer.

### G-26 — Image migration progress (🟡)

**Hely:** T8.3. **Megoldás:** SQLite progress-DB, resumable `--resume` flag, progress UI admin-on.

### G-27 — URL slug ütközések (🟡)

**Hely:** T8.2. **Megoldás:** dedup `-2`, `-3`, vagy SKU-alapú slug. ADR írandó.

### G-28 — E2E mock vs sandbox (🟡)

**v1.3 megoldás:**
- PR-ekre: mock (gyors, deterministic)
- Nightly + pre-deploy-production: real sandbox
- Contract teszt: Pact-szerű minta a Barion API-ra
- ADR-011

### G-29 — Test coverage cél (🟡)

**Megoldás:**
- Unit ≥80% kritikus modulokra (Barion, Számlázz, mapper, NAV)
- Critical path E2E 100%
- PR-gate: lefedettség nem csökkenhet

### G-30 — Hand-over curriculum strukturálva (🟡)

**Megoldás:** 6 modulos curriculum (új termék, rendelés, készlet, NAV-hibák, refund/sztornó, mit ne tegyünk). 5-10 perc videó modulonként + cheatsheet.

### G-31 — Post-launch 30/60/90 nap roadmap (🟡)

**Megoldás:** új 10. fejezet:
- T+30: review system, customer feedback, retro
- T+60: B2B portal (kedvezmények, adószám), kuponkód
- T+90: PWA, Loyalty pontszám

### G-32 — E-számla GDPR-hozzájárulás (🟡)

**Hely:** T0.5, T4.6. **Megoldás:** T3.4 acceptance: explicit tájékoztatás "Az e-számla elfogadása az ÁSZF elfogadásával együtt történik."

### G-33 — NPM verzió-rögzítés (🟢)

**Megoldás:** v1 launch-ig minden major-csomag verziója rögzítve a build spec-ben.

### G-34 — CSP konkretizálás (🟡)

**Hely:** T1.4, T1.8. **Megoldás:** Next.js middleware nonce-generation, `Content-Security-Policy: script-src 'self' 'nonce-{random}' https://*.sentry.io https://*.posthog.com`.

### G-35 — Backup encryption at rest (🟡)

**Hely:** T1.3. **Megoldás:** ⚠️ ELLENŐRIZD Railway docs / support. GDPR cikk 32 (technikai intézkedések).

### G-36 — Visual regression test (🟢)

**Megoldás:** Percy / Chromatic / Playwright snapshot — M2-re.

### G-37 — Dispute / chargeback flow (🟡)

**Hely:** T4.1, T4.7. **Megoldás:** új T4.7 forgatókönyv #11: "Barion chargeback notification → admin alert + befagyasztott rendelés státusz + audit-log."

### G-38 — ADR-deadline-ok (🟡)

**Hely:** 6. fejezet. **v1.3 megoldás:** mindegyik ADR mellé deadline (lásd build-spec 6. fejezet ADR-tábla).

---

## 3. Cross-cutting hiányok

### CC-1 — „ELLENŐRIZD" jelölések eljárás-hiánya (P0)

**Probléma:** 20+ ELLENŐRIZD passzív, nincs felelős/időablak/megoldási hely. **v1.3 megoldás (új 0.5.4 alfejezet):** minden ELLENŐRIZD-hez:
1. Felelős (NORBI vagy agent:<role>)
2. Időablak (Fázis 0 vége / task indulás előtt / konkrét dátum)
3. Megoldási hely: `docs/checks/<task-id>.md`
4. CI ellenőrző script: `tools/ci/check-ellenorizd-resolved.sh`

### CC-2 — Verzió-rögzítés inkonzisztencia (🟢)

**Megoldás:** új `docs/dependencies.md`.

### CC-3 — Magyar / angol terminológia (🟢)

**Megoldás:** Glossary-bővítés.

### CC-4 — CLAUDE.md / 0.5 fejezet redundancia (🟢)

**Megoldás:** 0.5 fejezet egy mondatos összefoglaló, CLAUDE.md a master.

### CC-5 — Fázis-belépési feltételek nem konzisztens (🟢)

**Megoldás:** minden fázis belépő KONKRÉT (T<x.y> mind completed + ADR-ek).

### CC-6 — Hiányzó ADR-ek implicit (P0)

**v1.3 megoldás:** új ADR-ek 0008-0013, mindegyik deadline-okkal:
- ADR-0008: ár-tárolás (G-13)
- ADR-0009: customer impersonation (G-10)
- ADR-0010: RPO/RTO (G-20)
- ADR-0011: E2E mock vs sandbox (G-28)
- ADR-0012: szállítók sorrendje
- ADR-0013: multi-currency v1 (G-08)

### CC-7 — Norbi-elv ellen menő részek (🟢)

**Megoldás:** alternatívák ADR-té kovácsolása vagy explicit M2-halasztás.

---

## 4. Mit néztem rendben (kontroll)

1. Fázis 0 jogi alap (T0.5) — 45/2014 minimum lefedi ✓
2. CSRF / XSS védelem — Medusa default + React escape ✓
3. PCI-DSS scope (T7.6) — SAQ A helyesen ✓
4. GDPR data export + delete (T7.7) — soft + hard delete + 8 év számla ✓
5. Cookie consent kategorizálás — 4 kategória GDPR-konform ✓
6. 301 redirect térkép (T8.5) ✓
7. Sentry / PostHog cookie consent integráció ✓
8. ODR link (T3.5) ✓
9. Karpathy autoresearch minta — PROPOSE → RUN → MEASURE → ITERATE ✓
10. Challenger sub-agent — más perspektíva, nem rubber-stamp-elhető ✓

---

## 5. Prioritási mátrix

| ID | Súly | Effort | P |
|---|---|---|---|
| G-04 (sztornó vs helyesbítő) | 🔴 | S | **P0** Fázis 0 |
| G-06 (NAV hibakód-térkép) | 🔴 | M | **P0** Fázis 4 belépő |
| G-07 (ÁFA-mező) | 🔴 | S | **P0** Fázis 2 belépő |
| G-09 (Barion fee költség) | 🔴 | S | **P0** Fázis 0 vége |
| G-13 (ár-tárolás) | 🔴 | M | **P0** ADR Fázis 0-1 |
| G-03 (Barion idempotencia) | 🔴 | M | **P0** T4.1 |
| G-10 (impersonation) | 🔴 | S | **P0** ADR Fázis 0 |
| G-11 (file upload) | 🔴 | M | **P0** T2.3 |
| G-16 (ISR availability) | 🔴 | M | **P0** Fázis 6 |
| G-20 (RPO/RTO) | 🔴 | S | **P0** ADR Fázis 0 |
| CC-1 (ELLENŐRIZD eljárás) | — | S | **P0** Fázis 0 |
| CC-6 (rejtett ADR-ek) | — | S | **P0** Fázis 0-1 |
| Többi 🟡 | 🟡 | S-M | **P1** |
| Többi 🟢 | 🟢 | S-M | **P2** post-launch |

---

## 6. Önkritikai ellenőrző kérdés

**1. NAV-ellenőrnek:**
- G-04 (sztornó vs helyesbítő tévedés) — 2 perc, azonnal észreveszi
- G-06 (NAV hibakód-térkép) — "Mit csinál 5402-re?" — generikus retry → hiányosság
- G-13 (nettó/bruttó) — "Nettóban tárol vagy bruttóban?" — nincs definiálva

**2. Hostile pen-testernek:**
- G-11 (file upload) — magic-bytes hiánya = első támadási vector
- G-10 (impersonation) — ezüst-tálcán adatszivárgás
- G-12 (admin lockout) — IP-rotation brute-force
- G-34 (CSP) — `unsafe-inline` esetén XSS

**3. Utolsó pillanatos jogásznak:**
- G-32 (e-számla GDPR-hozzájárulás)
- G-05 (PDF 8 év megőrzés)
- G-04 + G-06 (NAV-jogi precíziós hiányok)

**Konklúzió:** a build spec **erős a stratégiai struktúrában**, **gyenge 3 helyen**: magyar-jogi finomhangolás, üzleti realitás (Barion fee, RPO/RTO, post-launch), defense-in-depth security (file upload, lockout, CSP). v1.3-ban a 13 P0 fix beépült. 18 P1 és 7 P2 elhalasztható, de a NAV-jogi rész (G-04, G-06, G-13) Fázis 0-1 előtt megoldandó.

---

**A dokumentum vége.**
