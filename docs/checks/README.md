# Norbi-feladatok összesítője — Fázis 0 ELLENŐRIZD-pontok

> Ez az **EGYSZERŰSÍTETT to-do lista** Norbi-nak. A teljes részletek a `T0.x.md` fájlokban.

## Sürgős (Fázis 0 vége előtt)

### Cégadatok (T0.1, T0.2, T0.3, T0.5)

- [ ] **Cégnév:** Schuszter-Will Kereskedelmi és Szolgáltató Kft. (?)
- [ ] **Székhely:** Szarvas, ?
- [ ] **Adószám:** ?
- [ ] **Közösségi adószám (HU...):** ?
- [ ] **Cégjegyzékszám:** ?
- [ ] **Statisztikai számjel:** ?
- [ ] **Képviselő:** Norbi neve, anyja neve, lakcíme
- [ ] **Telefon (üzleti):** ?
- [ ] **Email (üzleti, info@ledcenter.hu):** ?
- [ ] **Banki számla (HUF):** ?
- [ ] **Banki számla (EUR, opcionális):** ?

### Domain és DNS (T0.1)

- [ ] **Jelenlegi domain regisztrátor:** ? (`docs/checks/T0.1.md` 1)
- [ ] **Cloudflare account:** új vagy meglévő? (`T0.1.md` 2)
- [ ] **2FA Cloudflare:** bekapcsolva
- [ ] **Backup admin (egy barát) hozzáférés:** ki?

### Fiókok regisztrálása

- [ ] **Barion sandbox** (`T0.2.md`) — KYC adatok benyújtva
- [ ] **Barion live** ApprovalState — Fázis 9 előtt approved
- [ ] **Barion 2FA** + backup admin
- [ ] **⚠️ Barion csomag választás:** Smart Gateway Fix vs IC++? (`T0.2.md` 4)
- [ ] **Számlázz.hu fiók** (DÍJBEKÉRŐ csomag minimum, ~3990 Ft/hó)
- [ ] **Számlázz.hu 2FA**
- [ ] **NAV ügyfélkapu:** Norbi hozzáfér?
- [ ] **NAV technikai user** létrehozva, jogosultság a Számlázz-nak delegálva
- [ ] **Railway account** (Pro plan, $20/hó — ADR-010 miatt)
- [ ] **Railway 2FA**
- [ ] **ShopRenter API hozzáférés** ellenőrzés (OAuth2 elérhető?)

### Számlázz.hu support (írásbeli válaszok kellenek!)

- [ ] **⚠️ Test mode kapcsolása:** hogyan? (külön fiók, paraméter?) — `T0.3.md` 4
- [ ] **⚠️ 24h-túli sikertelen NAV-bejelentés (G-06):** Számlázz kezeli vagy mi? — `T0.3.md` 5

### Könyvelő (G-04 — KRITIKUS)

- [ ] **⚠️ Részleges refund flow:** opció A (helyesbítő) vagy opció B (sztornó+új)? — `T0.3.md` 6

### ÁSZF / adatkezelés (T0.5)

- [ ] **Termékkör pontos leírása**
- [ ] **Szállítási területek** (csak HU vs EU)
- [ ] **Fizetési módok** (Barion + banki + utánvét?)
- [ ] **Garancia / jótállás politika**
- [ ] **⚠️ Ügyvéd-review** (50-150 ezer Ft, Fázis 9 előtt)
- [ ] **DPA-k aláírása** (Resend, PostHog, Sentry, Cloudflare, Bunny.net, Számlázz, Railway) — `T0.5.md` 7

### Brand-elemek (T0.6)

- [ ] **Logó SVG** (transparent BG, dark + light)
- [ ] **Márkaszínek** HEX (primary + secondary + accent)
- [ ] **Font preferencia**
- [ ] **Stílus iránya** (minimalista / barátságos / klasszikus)
- [ ] **Tone of voice** (tegezés / magázás)
- [ ] **Hero-fotók** forrása

### Migráció (T0.7)

- [ ] **ShopRenter admin login**
- [ ] **OAuth2 csomag elérhető?**
- [ ] **ADR-006:** Customer + order migráció YES/NO? (default SKIP, Fázis 8 elején dönt)

---

## Nem sürgős (későbbi fázis-belépőknél)

### T1.3 (Fázis 1) — Backup encryption

- [ ] **⚠️ Railway Postgres encryption-at-rest** ELLENŐRIZD Railway support — `T0.4.md`-ben jegyzett

### T2.3 (Fázis 2) — Antivirus scan

- [ ] **Antivirus** (ClamAV vagy VirusTotal) — szükséges v1-be? (default NEM)

### T2.4 (Fázis 2) — Image CDN

- [ ] **ADR-0002:** Bunny.net vs Cloudflare Images — Norbi végleges választása

### T4.1 (Fázis 4) — Barion details

- [ ] **Barion callback IP allowlist** aktuális lista
- [ ] **Barion test card** aktuális szám

### T4.5 (Fázis 4) — Szállítók

- [ ] **ADR-0012:** FoxPost / MPL / GLS sorrend — melyik elsőként? (default: FoxPost)
- [ ] **Szerződés-kötés** mind a 3 szolgáltatóval

### T7.9 (Fázis 7) — Pen-test

- [ ] **ADR-0007:** Külső pen-test ($1500-5000) vagy OWASP ZAP (ingyenes)?

---

**Status:** ⏳ minden kérdés nyitott — Norbi-felelősség.

**Hogyan használd:** minden sort tölts ki, ahogy halad. A teljes részletek (kontextus, miért fontos) a `T0.x.md` fájlokban. A code-writer agent ezt a fájlt olvassa minden T0.x task elindításakor.
