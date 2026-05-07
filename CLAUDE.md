# Ledcenter — projekt-instrukciók (CLAUDE.md)

> Ez a fájl Andrej Karpathy CLAUDE.md mintáját
> ([forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills))
> követi, magyarra fordítva, a Ledcenter projektre szabva.

## Projekt kontextus

A `ledcenter.hu` magyar LED-világítástechnikai webáruház átépítése egy modern,
headless e-commerce stack-re. Backend: Medusa.js 2.x. Storefront: Next.js 15
App Router. Adatbázis: PostgreSQL 16. Fizetés: Barion. Számlázás: Számlázz.hu
(NAV Online Számla 3.0). Hosting: Railway. ~1280 termék migráció ShopRenter-ről.

**A részletes terv a `docs/build-spec.md`-ben van** (4400+ sor, 76 task fázisokba
szervezve). Ez a CLAUDE.md csak az **operatív viselkedést** szabályozza — a *mit*
épít az agent, az a build spec szerint történik.

## A 4 alapelv (Karpathy)

### 1. Gondolj mielőtt kódolnál
- Ne tippelj. Ne titkold a bizonytalanságot. Hozd felszínre a tradeoff-okat.
- Ha egy követelmény homályos: **kérdezz**, ne találj ki magyarázatot.
- Ha 2 implementációs út lehetséges: dokumentáld mind a kettőt egy gyors
  összevetésben, és ajánlj egyet (ADR-rel).

### 2. Egyszerűség elsőbbsége
- Minimum kód, ami a problémát megoldja. Semmi spekulatív.
- Semmilyen feature azon felül, amit kértek.
- Új lib hozzáadása előtt: lehet ezt 20 sor saját kóddal? Ha nem, okold meg ADR-ben.

### 3. Sebészi módosítások
- Csak azt érintsd, amit muszáj. Csak a saját rendetlenséget takarítsd el.
- Refactor csak külön ADR-rel.

### 4. Cél-vezérelt végrehajtás
- Definiáld a sikert (acceptance criteria a build spec-ből).
- Loopolj, amíg verifikálva nincs.
- Ne lépj tovább, amíg az aktuális task minden acceptance criteria ✓.

## A két alapszabály — Kötelező munkamód (build spec 0.5)

### 1. Research-then-code

Minden task **első outputja** `docs/understanding/T<x.y>.md` — NEM kód, **külön
commit**, magyarul. Hossz a komplexitás szerint:

- **S effort** (≤½ nap): 3-5 sor (cél, érintett fájl, library, ADR-worthy?).
- **M effort** (1-2 nap): 8-15 sor.
- **L/XL effort** (3+ nap, kritikus integráció — Barion / Számlázz / NAV /
  migráció): 15-30 sor + alternatívák pro/con + edge case-ek + decision points.

Sablon (M minimum):

```markdown
# T<x.y> Understanding — <cím>

**Cél:** <1 mondat>
**Érintett:** <fájl-paths, külső API-k>
**Library:** <NPM/repo + verzió + miért ezt>
**Acceptance saját szavakkal:** <build spec AC-k összegzése>
**Edge case-ek:** <2-3 pont>
**ADR-worthy?** igen/nem (ha igen → `/challenge` előbb)
**Kérdések Norbi-nak:** <ha van>
```


### 2. Cross-agent decision challenge

Minden ADR-worthy döntést `challenger` sub-agent (külön Task tool spawn, friss
kontextus, NEM ugyanaz a session) megkérdőjelez **MIELŐTT** kód kerülne a repóba.
**Min. 3 ELLEN-érv kötelező** (kockázat, alternatíva, long-term concern), konkrét
forgatókönyvvel — NEM általánosság. Rubber-stamp kizárva. A javaslat csak akkor
megy tovább kódolásra, ha a challenger érveire **érdemi** (NEM formális) válasz
született, és ADR rögzíti a döntést.

ADR-worthy: architektúra-választás, library-választás, security trade-off,
adatmodell-döntés, vendor-választás. NEM ADR-worthy: változó-elnevezés, CSS class.

## Sub-agent szerepkörök

Mindegyik agent egy `.claude/agents/<name>.md` fájl. Egy agent egyszerre **egy**
task-on dolgozik (TaskUpdate `owner`-rel jelzi).

| Szerepkör | Mit csinál |
|---|---|
| **orchestrator** | Main session — plan, task selection, koordináció. |
| **code-writer** | Kód implementálása acceptance criteria szerint (Karpathy loop). |
| **challenger** | Design / library / architektúra döntést megkérdőjelez kód előtt. Min. 3 ELLEN-érv. |
| **code-reviewer** | Diff review (stílus, hibák, tesztek) — NEM design. |
| **security-auditor** | OWASP / GDPR / NAV-jog ellenőrzés. |
| **integration-tester** | E2E teszt kritikus utakra (checkout, számla, NAV, refund). |
| **seo-checker** | Lighthouse, schema.org, broken link, sitemap, llms.txt. |
| **doc-writer** | ADR, runbook, decision log frissítése. |
| **migration-planner** | ShopRenter → új stack adatmigráció (Fázis 8) tervezése és felügyelete. |

## Slash command-ok

| Parancs | Cél |
|---|---|
| `/task <T<x.y>>` | **Umbrella-parancs** a 0.5 munkamód-flow-jához: research → ha ADR-worthy: challenger → ADR → kód → review → security audit. Ott áll meg, ahol emberi beavatkozás kell. |
| `/security-audit [N]` | security-auditor sub-agent az utolsó N committra. |
| `/seo-check` | seo-checker: Lighthouse + schema validáció. |

## Tech-stack (kötött, ADR nélkül NEM változtatható)

- Backend: **Medusa.js 2.x**
- Storefront: **Next.js 15 App Router** (Server Components, ISR)
- DB: **PostgreSQL 16**
- Cache / queue: **Redis 7** (BullMQ)
- Payment: **Barion Smart Gateway** (sandbox: `test.barion.com`)
- Számlázás: **Számlázz.hu** + `ewngs/szamlazz.js` SDK
- NAV: Számlázz.hu kezeli a beküldést, mi a státuszt monitorozzuk
- Email: **Resend** + **React Email**
- Search: **Meilisearch** (self-hosted Railway-en vagy Cloud)
- Image storage: **Cloudflare R2**
- Image transformation: **Bunny.net Optimizer**
- Hosting: **Railway**
- Monitoring: **Sentry** + **PostHog** + **Better Stack**
- DNS / WAF: **Cloudflare** (proxied)

## Kötelező library-k

- `zod` — minden user input + API I/O validáció
- `@medusajs/js-sdk` — storefront ↔ backend
- `react-hook-form` — form state
- `@anthropic-ai/sdk` — admin AI tools (T5.4, T5.5)
- `sharp` — image processing (file upload validáció)
- `file-type` — magic-bytes detection
- `ioredis` — Redis client
- `szamlazz.js` — Számlázz.hu API

## Tilos library-k

- `lodash` — túl nagy, ES2024 native
- `moment.js` — `date-fns` vagy native `Intl.DateTimeFormat`
- `axios` — natív `fetch`

## Magyar specifikumok

- **Pénznem:** `Ft` szuffix UI-on (NEM `HUF` kód)
- **Dátum:** `YYYY. MM. DD.` (Magyar locale)
- **Szám:** `1 234,56` (space ezres, vessző tizedes)
- **Áfa:** kötelező nettó / áfa / bruttó megjelenítés a checkout-on
- **ÁFA-mező:** termékenként `vatRate: 5 | 18 | 27` (default 27)
- **Kerekítés:** banker's rounding (`ROUND_HALF_EVEN`), 2 tizedes — fillér

## Tilos lista (NEM tárgyalási alap — pre-commit hook + lint blokkolja)

1. **Kódot írni `understanding.md` nélkül.** Pre-commit hook ellenőrzi, hogy a
   `docs/understanding/T<x.y>.md` létezik a kód-commit ELŐTT (külön commit).
2. **ADR-worthy döntést `challenger` nélkül commitolni.** A pre-commit hook
   ellenőrzi, hogy a `docs/challenges/T<x.y>.md` és a megfelelő `docs/adr/`
   fájl létezik.
3. **Hard-coded ÁFA-érték** (`vat: 27`, `* 1.27`). Lint-szabály blokkolja.
   A termék `vatRate` mezőjéből számolj.
4. **Nettó / bruttó / áfa keverése.** A `Price` schema invariáns: `priceVat =
   priceGross - priceNet`. Egységesen fillérben tárolunk.
5. **Customer impersonation** (login as customer, actAsCustomer, impersonate)
   v1-ben TILTOTT. ADR-009. Lint-szabály blokkolja.
6. **HUF-on kívüli currency** v1-ben TILTOTT (Barion + Számlázz konzisztencia).
   ADR-013 a M2-bevezetésig.
7. **Új feature pluszban**, ami nincs a build spec-ben. Scope creep tilos.
8. **Saját kód olyan helyettesítésére, ami iparági standard.** ZOD, react-hook-form,
   sharp, stb. mindig.
9. **Skip security checklist.** Fázis 4, 7, 8 minden taszk lezárása előtt
   security-auditor agent kötelező.
10. **Tippelni, ahol Decision tree „ELLENŐRIZD"-mond.** Minden ELLENŐRIZD-hez
    felelős + deadline + `docs/checks/<task-id>.md` (lásd build spec 0.5.4).
11. **Rubber-stamp challenge** — min. 3 ELLEN-érv, konkrét kifejtéssel,
    különben a challenge-fájl érvénytelen, friss sub-agent spawn újra.
12. **Skip-elni az `understanding.md`-t** „elég triviális" indoklással. Ha
    tényleg triviális, 3 sor — az is OK, de muszáj.

## Tesztelés

- **Vitest** unit teszt minden funkcióra. Cél coverage: kritikus modulok ≥80%
  (Barion, Számlázz, mapper, NAV, ár-kalkuláció).
- **Playwright** E2E a kritikus flow-kra (checkout success, fail, refund,
  login, search, admin product create).
- **Lighthouse CI** minden PR-ra (Mobile Performance ≥85, SEO=100,
  Accessibility ≥90).
- **Schema.org validation** script CI-ben.
- **Banker's rounding** unit-tesztelve (`roundHalfEven(2.5, 0) === 2`).

## Workflow (egy mondatban)

`TaskList → /task <T<x.y>> → understanding.md (külön commit) → ha ADR-worthy:
challenger (külön sub-agent, min. 3 ELLEN-érv) → érdemi válasz → ADR (külön
commit) → kód (Karpathy loop) → code-reviewer → security-auditor (ha checklist) →
integration-tester (Fázis 4, 8) → TaskUpdate completed`.

A pre-commit hook (`.claude/hooks/pre-commit`) blokkolja a kód-commitot, ha a
research-fájlok (understanding.md / challenges / ADR) hiányoznak.

## Kötelező hivatkozások

- **`docs/build-spec.md`** — 4400+ soros master spec (76 task, 9 fázis, függelékek).
- **`docs/build-spec-gaps.md`** — gap audit (38 hiány, 13 P0 fix v1.3-ban).
- **`docs/strategy.md`** — eredeti stratégiai dokumentum (üzleti kontextus).

A build spec **élő dokumentum** — minden architecturális döntéshez ADR
(`docs/adr/NNNN-<slug>.md`), minden runbook a `docs/runbooks/`-ba, minden
ELLENŐRIZD-feloldás a `docs/checks/<task-id>.md`-be.

---

**Indulás:** olvasd a `docs/build-spec.md` 0., 1., 2., 3., 4. fejezetét, majd
fuss `/task T0.1`. Az első task „Domain és DNS előkészítés" — mind a 76 task
ezután a build spec sorrendje szerint követi.
