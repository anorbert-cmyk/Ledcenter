# LEDcenter.hu — Migrációs és technológiai stratégia

**Készült:** 2026. április 27., Norbi részére
**Cél:** ShopRenter → modern, agentic-fejlesztésre optimalizált stack, Railway deploy, Claude Code workflow

---

## Vezetői összefoglaló (TL;DR)

**A három legfontosabb döntés:**

1. **Stack: Medusa 2.0 backend + Next.js 15 storefront + Postgres + Redis, mind Railway-en, monorepo formában.** Hivatalos Railway template (`Deploy medusajs ecommerce Nextjs, Postgres, Redis`). Medusa 2 érett, modulárisan bővíthető, TypeScript-natív. Saleor (Python/Django) és Vendure (NestJS) kiesett.

2. **Magyar integrációs csomag:** **Barion** (0.5–1.5%, gyors szerződés) + **Számlázz.hu** (NAV Online Számla automatikus, `ewngs/szamlazz.js` SDK) + **FoxPost + MPL** (csomagautomata + házhoz). Stripe tartalék.

3. **Migráció lépcsős, párhuzamos futással.** ShopRenter REST API (3 req/s/app, 200/batch) + CSV/XLSX export. **Kritikus SEO:** ~1335 termék-URL átirányítása 301-essel.

**Kritikus kockázatok:**
- **NAV Online Számla 3.0** (kötelező 2025.05.15. óta) — Számlázz.hu-ra hagyatkozni.
- **Domain átállítás SEO:** régi URL-struktúra megőrzése 301-essel.
- **Solo-dev időkockázat:** reális MVP 4–6 hét, full feature parity 8–12 hét.

**Becsült havi futási költség:** ~$45–75 USD (~16 000–27 000 HUF) induló forgalmon.

---

## 1. Jelenlegi állapot — ledcenter.hu

### 1.1 Publikus adatok

- **Backend:** PHP (ShopRenter, `Set-Cookie: PHPSESSID=...`).
- **Cache layer:** Varnish-szerű reverse proxy.
- **Sitemap:** `https://www.ledcenter.hu/sitemap.xml` — **1337 URL**. 1 főoldal + ~5–10 statikus + ~30–50 kategória + **~1280–1300 termék**.
- **URL-struktúra (termékek):** lapos, pl. `/lc-3-fix-lampatest-matt-ezust-659`. Slug + numerikus ID.
- **URL-struktúra (kategóriák):** `/kategoriak/{slug}`, egyszintű.
- **Termékkör:** LED-fényforrások (E27, GU10, G9, MR16), LED-szalagok és tápegységek, lámpatestek (mennyezeti, falra, kerti, fürdőszobai), okos LED-ek (WiFi-s GU10), LED-panelek. Brand-ek: Philips, Spectrum, Eglo, Optonica, ProLED, Algine, Fiale, Norton, Solis, Flamma, Snow, LC-saját.
- **Üzemeltető:** Schuszter-Will Kereskedelmi és Szolgáltató Kft., Szarvas. Saját raktár + SpectrumLED központi.
- **Szállítás:** MPL és FoxPost csomagpont, 1090 Ft-tól.
- **Aggregátorokon:** Árukereső, Depo (XML feed-et újra konfigurálni).

### 1.2 Várható Core Web Vitals

ShopRenter sablonok: kezdő LCP 1.5–2.5s asztali, 3–4s mobil. **Migráció után reset.** Cél Next.js 15 App Router-rel: LCP <1.8s, INP <200ms, CLS <0.1.

### 1.3 Mire NEM számíts

- **Nincs nyilvánosan strukturált adat** (Schema.org Product). → Lehetőség.
- **Nincs llms.txt / AI-friendly végpont** → versenyhátrány, pótolható.

---

## 2. ShopRenter export és migrációs felmérés

### 2.1 ShopRenter publikus API

| Tulajdonság | Érték |
|---|---|
| Architektúra | REST + JSON (HTTP Basic Auth) |
| Rate limit | **3 request / másodperc / app / shop** |
| Hibakód túllépésnél | HTTP 429 |
| Batch endpoint | igen, **max 200 request / batch**, **max 32 MB POST** |
| Webhook | igen (order, customer, product) |
| Hivatalos kliens | `github.com/Shoprenter/api-client` (PHP) |
| Hivatalos doksi | `doc.shoprenter.hu` |

**Számolás:** 3 req/s × 200/batch ≈ ~100–150 req/s reálisan. **1280 termék × ~5–10 endpoint ≈ 8000–13000 request → 1–2 perc teljes kinyerés.**

### 2.2 Mi exportálható

| Adat | API endpoint | CSV |
|---|---|---|
| Termékek (alap) | `Product Resource` | igen |
| Termék attribútumok (LED-spec) | `Product Extend Resource` | részben |
| Variánsok | API | igen |
| Kategóriák (hierarchia) | API | igen |
| Képek | API (CDN URL) | URL-listával |
| Vásárlók | API | igen |
| Rendelési előzmény | API | igen |
| SEO meta | API | részben |
| URL slug-ok | API | igen — **301-redirect térképhez** |

### 2.3 Migrációs útvonal

Custom exportert TypeScript-ben (~2–3 nap Claude Code-dal):
- `axios` vagy `undici` HTTP klienssel + `bottleneck` rate limiter (3 req/s)
- Inkrementális JSON dump (resumable)
- `image-downloader` + `sharp` AVIF/WebP konverzió
- Sémamap CSV → Medusa Product seed JSON

---

## 3. Stack ajánlás

### 3.1 Platform-összevetés

| Platform | Stack | Norbi-fit |
|---|---|---|
| **Medusa 2.0** | Node.js + TS, modular monolith | **★★★★★** |
| Saleor | Python + Django + GraphQL | ★★ |
| Vendure | NestJS + GraphQL | ★★★ |
| Next.js Commerce | Next.js + külső backend | ★★★ (storefront) |
| Custom Next.js + Drizzle | tiszta TS | ★★★ (kontroll) |

### 3.2 Medusa 2.0 — érvelés

**Pro:**
- Tisztán Node.js + TypeScript → Claude Code optimális.
- Modular monolith — minden modul cserélhető, plugin-szerűen bővíthető. Egyedi LED-attribútumok saját Product Module Extension-ben.
- Beépített REST API + JS SDK.
- Hivatalos Next.js 15 storefront starter: `github.com/medusajs/nextjs-starter-medusa`.
- Hivatalos Railway template — egy kattintás, Postgres + Redis konfigurált.
- Aktív fejlesztés, érett admin UI.

**Kontra:**
- Magyar fizetési pluginek (Barion, SimplePay) nincsenek hivatalosan — saját Payment Provider Module ~1–2 napos munka.
- 2.0 még friss, néhány third-party plugin csak v1-re.
- Admin UI nem ad bulk-edit/CSV-importot — saját custom dashboard érdemes (T5).

**Verdikt:** Ez a tisztán legjobb választás.

### 3.3 Backup opció (Plan B)

Custom Next.js 15 + Drizzle ORM + Postgres. **Drizzle**: ~12 KB runtime, edge-kompatibilis. **Prisma**: érettebb DX, edge-en csak Accelerate.

---

## 4. SEO, GEO és LLM-optimalizáció

### 4.1 Technikai SEO — Next.js 15

- **Termékoldalak:** SSG + ISR (`revalidate: 60` vagy `3600`). Ár/készlet kliens-side fresh.
- **Kosár:** SSR (server actions Next 15+).
- **JSON-LD:** a `<head>`-be embedelve a HTML-ben (NEM JS-ből).
- **Metadata API** (App Router-natív) — `next-seo` NEM kell.
- **Sitemap:** Next 15 native `app/sitemap.ts`.
- **`next/image`:** automatikus AVIF + WebP, `sizes` attribútum.
- **`next/font`:** Google Fonts vagy önhostolt, `font-display: swap`.

### 4.2 Strukturált adatok — kötelező minimum

Minden terméklapon:
- `Product` (név, kép, leírás, sku, brand)
- `Offer` (priceCurrency, price, availability, **shippingDetails**, **hasMerchantReturnPolicy** — KÖTELEZŐ 2025-2026)
- `additionalProperty` (Wattage, Lumen, Color temp)
- `BreadcrumbList` (kategóriák)
- `FAQPage` (kategória-oldalon)

Layoutban:
- `Organization`
- `WebSite` `SearchAction`

**Validáció:** `search.google.com/test/rich-results`.

### 4.3 GEO (Generative Engine Optimization)

**`llms.txt`** — alacsony költség, korai bevezetők előnyben.

```
# LEDcenter.hu

> Magyar LED-fényforrás és lámpatest webshop, 1200+ termékkel.

## Kategóriák
- [LED izzók](...): MR16, GU10, E27, E14, G9 foglalatok
- [LED panelek](...)
...

## Szállítás és fizetés
[Részletek](...)
```

**`/llms-full.txt`** — Markdown-formátumú teljes terméklista.

**AI-crawler robots.txt:** GPTBot, ClaudeBot, PerplexityBot, OAI-SearchBot, Google-Extended **Allow**.

### 4.4 Termékleírás-generálás Claude Sonnet 4.6

1280 termékhez magyar SEO-leírás:
- Manuálisan: ~50–100 óra
- Anthropic API + Sonnet 4.6 + prompt cache: ~3–6 óra futási idő, ~50–150 USD

**Duplikált tartalom kockázat:** minden leíráshoz 3–5 termék-specifikus paramétert (saját tesztelt érték).

**Alt text:** Sonnet vision-vel ~$0.003/kép → 6000 kép = ~$18 USD.

### 4.5 Kulcsszó-kutatás magyar piacra

| Eszköz | Költség |
|---|---|
| Google Keyword Planner | ingyenes (Ads konto) |
| Senuto (HU erős) | ~5000 Ft/hó |
| Ahrefs / SEMrush | $129/hó |

**Indulásra:** Google Keyword Planner + Senuto.

### 4.6 Top 10 azonnali SEO-akció

1. 301 redirect térkép a régi URL-ekről
2. Schema.org Product + Offer + AggregateRating + shippingDetails + hasMerchantReturnPolicy
3. Dinamikus sitemap.xml cron-revalidációval
4. `next/image` AVIF + lazy load
5. Generált magyar termékleírások (Sonnet 4.6 batch)
6. `/llms.txt` és `/llms-full.txt`
7. Google Search Console + Bing Webmaster Tools
8. IndexNow protokoll
9. Core Web Vitals dashboard (PostHog Web Vitals + Sentry)
10. Internal linking (related products + breadcrumb)

---

## 5. Magyar piaci integrációk

### 5.1 Számlázás (NAV Online Számla 3.0)

**Háttér:** 2025.05.15-től kötelező B2C tranzakcióhoz NAV Online Számla 3.0 XML schema. **Ne saját kézzel** — szolgáltatás végezze.

| Szolgáltató | Node SDK | Erősség | Ár |
|---|---|---|---|
| **Számlázz.hu** | `ewngs/szamlazz.js` (érett, prepayment + storno) | Piaci de-facto, NAV-továbbítás auto | ~3990 Ft/hó |
| Billingo | `@codingsans/billingo-client` (TS, OpenAPI) | Tisztább API V3 | ~3990 Ft/hó |

**Ajánlás:** **Számlázz.hu az `ewngs/szamlazz.js`-szel.** Magyar piac legmegszokottabb.

**Workflow Medusa-integrációban:**
1. Order completed event → Medusa worker queue
2. Worker meghívja `szamlazz.js`-t, generál PDF + XML, küld NAV-nak
3. PDF visszamentődik az order-hez (R2 bucket-ben, 8 év)
4. E-mail a vevőnek Resend-en

### 5.2 Fizetés (HUF)

| Szolgáltató | Tranzakciós díj | DX |
|---|---|---|
| **Barion** | 0.5–1.5% + 0.1% kivét | jó, `aron123/node-barion` |
| SimplePay (OTP) | 1–2% + csatlakozás | közepes |
| Stripe | 2.9% + 30 Ft (HUF) | kiváló |
| K&H/CIB/MKB | szerződéses | rossz |

**Ajánlás: Barion elsőként, Stripe másodikként.**
- Barion olcsóbb, gyors szerződés, magyar UI
- Stripe mint Apple/Google Pay backup
- **Utánvét + banki átutalás** kezdeti hónapokban (nulla integráció)

### 5.3 Szállítás

| Szolgáltató | API | Ajánlás |
|---|---|---|
| **FoxPost** | WebAPI v2 | **kötelező** — magyar piac legnépszerűbb csomagautomata |
| **Magyar Posta MPL** | API + címkenyomtatás | **kell** — vidéki PostaPont |
| **GLS Hungary** | hivatalos GLS API | **erős opció** prémium |
| Packeta | api-docs.packeta.dev | **opcionális** |
| DPD | API | nem prioritás |

**Ajánlás indulásra:** **FoxPost + MPL + GLS.**

### 5.4 ÁFA, EKAER, OSS

- **ÁFA:** 27% standard, 18%/5% kedvezményes (LED-re nem releváns).
- **B2B vs B2C:** Számlázz.hu kezeli automatikusan.
- **EKAER:** nem releváns SMB e-commerce-re.
- **EU OSS:** ha cross-border EU >10 000 EUR/év, regisztráció + külföldi áfa.

### 5.5 GDPR és cookie consent

- **`klaro!`** (open-source, self-hosted, magyar locale) — **ajánlott**
- Cookiebot (~11 EUR/hó), CookieYes (olcsóbb)
- **Adatkezelési tájékoztató:** NAIH-minta + GDPR-jogász review (~50–100 ezer Ft)
- **Adat-törlési flow:** 30 napos retention → soft-delete → hard-delete + audit log

### 5.6 Lokalizáció

- **i18n:** **`next-intl`** (Next.js App Router-natív)
- Magyar formázás: `Intl.NumberFormat('hu-HU')` — "2 490 Ft"

---

## 6. GitHub repo-k és Claude Code workflow

### 6.1 Repo-k

| Repo | Cél |
|---|---|
| `medusajs/nextjs-starter-medusa` | storefront alapja |
| `medusajs/medusa` | core, csak ha forkolni kell |
| `vercel/commerce` | referencia |
| `shadcn/ui` | UI-komponensek |
| `Shoprenter/api-client` | ShopRenter REST kliens (referencia) |

### 6.2 Admin UI

| Megoldás | Mikor |
|---|---|
| **Tisztán Medusa Admin** | MVP-hez elég |
| **shadcn/ui + TanStack Table custom** | 2. iteráció (LED-attribútum bulk edit) |
| Refine.dev | közbenső út |
| AdminJS | gyengébb DX |

→ **Indulj Medusa Admin-nal**, fokozatos saját admin oldalak.

### 6.3 Termékkezelő komponensek

| Funkció | Csomag |
|---|---|
| CSV import | `papaparse` |
| Multi image upload | `uppy` vagy `react-dropzone` |
| Drag-drop reorder | `@dnd-kit/core` |
| Kategória-fa | `react-arborist` |
| Rich text | `tiptap` |
| Tábla | `@tanstack/react-table` |

### 6.4 MCP szerverek

| MCP | Cél |
|---|---|
| GitHub MCP | PR-ek, issue-k |
| Postgres MCP | Medusa DB |
| Sentry MCP | hibakezelés |
| shadcn/ui MCP | komponens-keresés |
| Stitch MCP | Norbi-é már |
| Cloudflare MCP | DNS, R2 |
| Railway MCP | deploy |
| PostHog MCP | analytics (Norbi-é már) |

**Day-1:** GitHub, Postgres, Sentry, shadcn — ez a négy minimum.

### 6.5 Subagent-ek

| Subagent | Trigger |
|---|---|
| `code-reviewer` | `/review` slash, PR előtt |
| `migration-runner` | `/migrate` slash, ShopRenter ETL |
| `seo-auditor` | release előtt |
| `db-migrator` | Medusa modul-bővítés |
| `test-writer` | refactor után |

---

## 7. Deploy és infrastruktúra

### 7.1 Railway

**Hivatalos Medusa template** (`railway.com/deploy/QvfPwp`):
- Medusa backend (Node)
- Next.js storefront
- Postgres 16 (managed, auto backup)
- Redis (managed)
- Healthcheck-ek

**Árazás:**
- Hobby: $5/hó base + use-based, ~$5–15/hó
- Pro: $20/hó base + use-based
- Postgres: ~$5–10/hó

### 7.2 CDN és képtárolás

| Szolgáltató | Tárolás | Egress | EU |
|---|---|---|---|
| **Bunny.net** | $0.01–0.03/GB/hó | $0.005–0.02/GB | EU (Slovenia) |
| **Cloudflare R2** | $0.015/GB | **$0** | US, EU opció |
| AWS S3 EU + CloudFront | drágább | drágább | EU |

**Ajánlás:** **Bunny.net.**
- EU-natív, GDPR-defaultban tiszta
- Bunny Optimizer Pro on-the-fly kép-transzformáció
- ~$0.5/hó kis forgalomnál

### 7.3 Email — tranzakciós

| Szolgáltató | Ingyen | Pro |
|---|---|---|
| **Resend** | 3000/hó | $20/hó |
| Postmark | nincs | $15/hó |
| Brevo | 300/nap | ~$25/hó |
| SendGrid | nincs (2024 óta) | $20/hó |

**Ajánlás: Resend** — `react-email` integráció.

### 7.4 Monitoring

| Réteg | Eszköz | Költség |
|---|---|---|
| **Errors** | Sentry | ingyen 5K event/hó |
| **Analytics + Replay + Flags** | PostHog | ingyen 1M event/hó |
| **Uptime + log** | Better Stack | ingyen 50/0.5GB |
| **Performance** | PostHog Web Vitals + Sentry | a fentiek része |

### 7.5 CI/CD

GitHub Actions (ingyen 2000 perc/hó private):
1. Lint (Biome / ESLint + Prettier)
2. Typecheck (`tsc --noEmit`)
3. Unit test (Vitest)
4. Build (`next build` + `medusa build`)
5. E2E (Playwright)
6. Deploy (Railway PR preview, main → prod)

### 7.6 Auth

| Megoldás | Mikor |
|---|---|
| **Better Auth** | self-hosted, TS-native, 2FA, RBAC, **★★★★★** admin |
| Auth.js (NextAuth v5) | Better Auth átvette 2025.09. |
| Clerk | hosztolt, vendor-lock-in |
| Medusa beépített customer auth | **vásárlókhoz használjuk** |

**Ajánlás:** **Better Auth admin + Medusa customer auth.**

### 7.7 Backup + DR

- **Postgres napi backup** Railway: 7 nap Hobby, 30 nap Pro
- **Off-Railway:** GitHub Actions cron `pg_dump | gzip | s3cmd put` Bunny Storage-ra, heti, 12 héten át
- **Képek backup:** Bunny Geo-replicated tier (~$0.03/GB)

---

## 8. Biztonság

### 8.1 OWASP Top 10

| Veszély | Mitigation |
|---|---|
| SQL injection | Drizzle/Medusa ORM (parametrizált) |
| XSS | React escape, ESLint `dangerouslySetInnerHTML` szabály |
| CSRF | Next.js Server Actions auto-mitigation |
| Broken auth | Better Auth + 2FA + brute-force védelem |
| Sensitive data | NEVER log password/kártya, PII anonymizálás |
| Security misconfig | CSP, HSTS, secure cookies |
| Vulnerable deps | Dependabot weekly + `pnpm audit` |
| Auth bypass | Server-side check minden admin route-ra |
| Insecure deserialization | csak Zod-validált JSON |
| Insufficient logging | Sentry + Better Stack log retention |

### 8.2 Rate limiting

**Upstash Ratelimit** middleware:
- `/api/auth/*` → 5/15 perc/IP
- `/api/checkout/*` → 10/perc/IP
- `/api/webhooks/payment` → 100/perc (IP whitelist)
- `/admin/*` → 30/perc/sessionToken

**Cloudflare Free** + Turnstile CAPTCHA regisztrációhoz.

### 8.3 PCI-DSS

**SAQ-A** (Barion / Stripe iframe / hosted):
- Soha ne logold a kártya-payload-ot
- Iframe-ek `<iframe sandbox>` HTTPS-en
- Évi self-assessment

### 8.4 Secrets management

**Railway env vars** alapból, **Doppler** ha skálázódik ($0–18/hó/10 user).

### 8.5 GDPR-folyamatok

- Adatkezelési tájékoztató: NAIH-minta + jogász review
- Cookie consent: `klaro!` magyar locale
- Adathordozhatóság: `/api/me/export` JSON
- Törlés joga: `/api/me/delete` 30 nap soft-delete + audit log
- Adatvédelmi incidens: 72h NAIH bejelentés

---

## 9. Admin és termékfeltöltő felület

### 9.1 LED-attribútumok adatmodellezése

`ProductLEDSpecs` entity:

| Mező | Típus |
|---|---|
| `wattage` | Float |
| `legacy_wattage_equivalent` | Float |
| `lumen` | Integer |
| `color_temp_k` | Integer |
| `cri` | Integer |
| `socket` | Enum (E27/E14/GU10/G9/MR16/...) |
| `bulb_shape` | Enum |
| `dimmable` | Boolean |
| `ip_rating` | String (IP20/IP44/IP65/...) |
| `lifetime_h` | Integer |
| `voltage_v` | Integer |
| `beam_angle_deg` | Integer |
| `smart_protocol` | Enum (none/wifi/zigbee/...) |

→ Faceted navigation (filter sidebar) — **SEO-aranybánya** long-tail keresésekhez.

### 9.2 Bulk-edit és CSV-import

- ShopRenter migráció: egyszeri `pnpm exec migrate-shoprenter`
- Folyamatos: CSV-import wizard (PapaParse + multistep), bulk-edit TanStack Table

### 9.3 AI-segédlet

`Generate description` és `Generate alt text` gombok admin-on:
- Anthropic API server action, Claude Sonnet 4.6, prompt caching
- Felhasználó átnézheti és módosíthatja
- Workflow: `draft → reviewed → published`

### 9.4 Jóváhagyási lépcsők

- Norbi (admin): mindenhez hozzáfér
- Barátok (editor): csak sajátjuk
- `featured` kategória: csak admin
- Audit log minden state-változásra

---

## 10. Migrációs stratégia

### 10.1 Lépcsős terv

| Fázis | Munkamennyiség | Eredmény |
|---|---|---|
| **1. Adat-audit** | 1–2 nap | `data/shoprenter-snapshot/` JSON-ok |
| **2. Séma-mapping** | 1 nap | `mapping.ts` Zod-dal |
| **3. Kép-migráció** | 1 nap (~30–60 perc futás) | ~6000 kép R2 + WebP/AVIF |
| **4. URL-redirect térkép** | — | `next.config.ts` 301 redirect |
| **5. Medusa seed** | 1 nap (~1–2 perc futás) | Bulk Create API |
| **6. Staging futás** | 1 hét | UAT, Norbi + 2-3 barát |
| **7. DNS-átállás** | 1 óra technikai | TTL 60s előre, Cloudflare DNS |
| **8. Monitoring** | 4 hét | Search Console, Sentry, PostHog |

### 10.2 Hibatűrés

- **Inkrementális:** state-mentés (`progress.json`), resumable
- **Idempotens:** Medusa `external_id` = ShopRenter `id`
- **Validation report:** hibás sorok CSV-ben

---

## 11. Idő- és költségbecslés

### 11.1 Realisztikus mérföldkövek (solo + Claude Code)

| Hét | Eredmény |
|---|---|
| 1–2 | Setup: Railway template, GitHub, CLAUDE.md, MCP-k, Stitch, ShopRenter exporter prototípus |
| 3–4 | MVP frontend, Barion sandbox, Számlázz.hu sandbox |
| 5–6 | Faceted filterek, kép-migráció, FoxPost+MPL, Resend |
| 7–8 | Saját admin (bulk-edit), AI-leírás, jóváhagyás, SEO-hardening |
| 9–10 | UAT, perf-tuning, a11y, security-review, 301-redirect, staging full teszt |
| 11–12 | Go-live: DNS, monitoring, hibajavítás, indexálás |

**Reálisan:** 12–14 hét = 3–3.5 hónap.

### 11.2 Költség minimum tier (Hobby)

| Tétel | USD/hó | HUF |
|---|---:|---:|
| Railway Hobby | 8–12 | 2 880–4 320 |
| Domain `.hu` | ~4 | ~125 |
| Bunny.net | 1–2 | 360–720 |
| Resend (3K free) | 0 | 0 |
| Sentry Hobby | 0 | 0 |
| PostHog Cloud | 0 | 0 |
| Better Stack | 0 | 0 |
| Cloudflare Free | 0 | 0 |
| Számlázz.hu | ~11 | 3 990 |
| **Összesen** | **~20–25** | **~7 500–9 500** |

### 11.3 Comfortable tier (Pro, 100–500 rendelés/hó)

| Tétel | USD/hó |
|---|---:|
| Railway Pro | 25–40 |
| Bunny.net | 2–5 |
| Resend Pro | 20 |
| Sentry Team | 26 |
| PostHog | 0–50 |
| Better Stack | 0–25 |
| Számlázz Pro | 15 |
| **Összesen** | **~90–180** |

### 11.4 Egyszeri költségek

- GDPR-jogász review: 50–150 ezer Ft
- Logo / brand: 50–500 ezer Ft, vagy DIY
- Számlázz.hu csatlakozás: nincs
- Barion regisztráció: nincs

### 11.5 Tranzakciós díjak

100 000 Ft kosár:
- Barion 0.5–1.5%: 1200–1800 Ft + 100 Ft kivét
- Stripe 2.9% + 30 Ft fix: 3200 Ft

→ **Barion mindig olcsóbb HUF-ra.**

---

## 12. Azonnal használható next steps — 48 óra

### 12.1 Norbi 48 órás feladatlistája

**Nap 1 (~4–6 óra):**
1. Domain és DNS előkészítés
2. Új Railway project (Medusa template)
3. Repo setup (pnpm workspaces, monorepo)
4. MCP install (GitHub, Postgres, Sentry, shadcn)
5. Stitch design első iteráció

**Nap 2 (~3–4 óra):**
6. ShopRenter API kulcs
7. Számlázz.hu sandbox
8. Barion sandbox
9. FoxPost partner-fiók
10. Logó és brand-asset audit

### 12.2 Üzleti döntések a barátokkal

| Kérdés | Norbi javasolt válasz |
|---|---|
| Megtartjuk a `ledcenter.hu` domaint? | igen, NEM váltunk |
| Kit illet a domain? | Schuszter-Will Kft. |
| Új vagy régi banki számla? | régi |
| Forgalom (rendelés/hó, kosárérték)? | <100 → Hobby; 100–500 → Pro |
| Adminok / editor-ek? | max 3 admin, 5–10 editor |
| Akciókat ki kezeli? | Norbi indulásra |
| Marketing csatornák? | Google Ads, Facebook Pixel, Árukereső, Depo |

### 12.3 Asset-lista

- Logó SVG (transparent BG, dark+light)
- Brand-színek hex (primary + secondary + accent)
- Hero-fotók (saját termék vagy márka stock)
- ÁSZF-tervezet
- Adatkezelési tájékoztató (jogász review)
- Vásárlási feltételek
- GYIK (kategóriánként 5–10 kérdés)
- "Rólunk" oldal szöveg

---

## Záró megjegyzés

**Két lépés-ajánlat:**

**A. „Speed run":** Még ezen a héten Railway template deploy + ShopRenter exporter prototípus 5 termékkel staging-en. Holnap megnézheted, hogy néz ki egy LED-szpot az új stack-en.

**B. „Plan first":** Részletes munka-lebontási struktúra (Linear / Notion), Stitch dizájn-iteráció, csak utána kódolunk.

A többségnek (Claude Code agentic stílus): **A** javasolt — egy működő MVP gyorsabban tanít.

---

**A dokumentum vége.**
