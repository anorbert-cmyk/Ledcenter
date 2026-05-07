# Ledcenter.hu — Agent-Team Build Specification (v1)

**Verzió:** 1.3
**Dátum:** 2026-05-07
**Nyelv:** magyar
**Cél:** teljes körű, agent-team-ready specifikáció a ledcenter.hu modern e-commerce stack-re történő átépítéséhez (ShopRenter → Medusa.js 2.x + Next.js 15)
**Kódbázis:** GitHub (privát)
**Hosting:** Railway
**Frontend dizájn:** Google Stitch (Stitch MCP)
**Becsült termékkatalógus:** ~1280 SKU
**Termékkör:** LED világítástechnikai termékek (B2C + B2B)

---

## 0. Bevezetés és célközönség

> **🔴 KIEMELT: Kötelező munkamód — olvasd el ELŐSZÖR.**
>
> **1. Research-then-code.** Minden task első outputja `docs/understanding/T<x.y>.md` (3-30 sor a komplexitás függvényében) — NEM kód, külön commit. Indoklás: „Csak egy lehetőségünk van ezt jól elkészíteni."
>
> **2. Cross-agent decision challenge.** Minden ADR-worthy döntést `challenger` sub-agent (külön Task tool spawn, friss kontextus) megkérdőjelez kód előtt. Min. 3 ELLEN-érv. NEM a code-reviewer (az diffet néz). Rubber-stamp kizárva — a kódolás csak érdemi (NEM formális) válasz után indul.
>
> Részletek: 0.5 alfejezet. Operatív végrehajtás: `/task <T<x.y>>` (lásd 3.5).

### 0.1 Mire való ez a dokumentum?

Ez a dokumentum **egy autonóm agent-csapat** (Claude Code orchestrator + sub-agentek) számára készült. A dokumentum kétértelműség nélkül leírja:

- Mit építünk (terjedelem és funkcionális határok).
- Milyen technológiai stack-ben építjük (kötött döntések — nem nyitottak újragondolásra a fázis 1 előtt).
- Milyen sorrendben építjük (fázis 0 → fázis 9).
- Minden taskhoz: pontos input/output, acceptance criteria, használandó library/SDK, security ellenőrzési lista, teszt követelmény, döntési fa kétértelmű helyzetekre, becsült effort.

### 0.2 Hogyan használja az agent ezt a dokumentumot?

**Kötelező olvasási sorrend (egyszer, lineárisan, tárolja az eredményt CLAUDE.md-ben):**

1. **0., 1., 2., 3., 4. fejezet** — projekt kontextus + architektúra + agent-csapat + repo setup.
2. **5. fejezet** — fázisok teljes átolvasása, hogy lássa a teljes ívet, mielőtt elkezd egy egyedi taskot.
3. **9. fejezet függelékei** — sablonok, csekklisták, teszt-protokollok.

**Munkamenet (per task):**

```
1. TaskList → válaszd ki a következő pending, owner-rel nem foglalt és nem-blokkolt taskot.
2. Olvasd el a task teljes definícióját (Description, Inputok, Outputok, Acceptance criteria, Security checklist, Decision tree).
3. Készíts plan-t (TodoWrite vagy Task SDK), 3-7 lépésre bontva.
4. Implementálj — egyszerre EGY task, EGY commit-blokk.
5. Futtasd a tesztet (acceptance criteria szerint).
6. Security review checklist — ha bármi piros, NE merge-elj.
7. TaskUpdate → completed; commitold a változást "T<x.y>: <cím>" formátumban.
8. Tisztítsd a TodoListát, és vissza 1-re.
```

**Tilos:**

- Új feature-t hozzáadni, ami nem a taskban szerepel ("scope creep").
- Saját kódot írni olyasmire, amire iparági standard library létezik.
- Skip-elni a security checklistet.
- "Megtippelni" egy bizonytalan döntést — KÖTELEZŐ a Decision tree szerinti ellenőrzés, vagy ha az sem dönt, akkor tegyen fel egy ADR-t (Architecture Decision Record) és kérjen Norbi-jóváhagyást.

### 0.3 A dokumentum frissítési protokollja

A dokumentum **élő** — ahogy a projekt halad, az agent-csapat:

- Új tanulságokat ír be a `Decision logs` (6. fejezet) részbe ADR sablonnal.
- Hibás vagy elavult részeket a `## CHANGELOG` szekcióban a dokumentum végén jelez.
- A főtartalom módosítása **csak Norbi explicit jóváhagyásával** engedélyezett (commit subject: `spec: <change>`).

### 0.4 Karpathy-elvek — a kódolás filozófiája

Ez a projekt Andrej Karpathy ([github.com/karpathy](https://github.com/karpathy)) coding-agent best practice-ire épít, ahogyan azt Forrest Chang összegezte a [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) repóban (~109k★ 2026 májusában). A négy alapelv minden taskra érvényes:

1. **Think Before Coding** — Ne tippelj. Ne titkold el a bizonytalanságot. Hozd felszínre a tradeoff-okat. Ha egy követelmény homályos, az agent **megáll és kérdez**, nem fabrikál.
2. **Simplicity First** — A minimum kódot írd, ami a problémát megoldja. Semmit ami spekulatív. Semmilyen feature-t azon felül, amit kértek.
3. **Surgical Changes** — Csak azt érintsd meg, amit muszáj. Csak a saját rendetlenséget takarítsd el.
4. **Goal-Driven Execution** — Definiáld a sikert (acceptance criteria). Loopolj, amíg verifikálva nincs.

Ezt a négy elvet a `CLAUDE.md` (lásd Függelék H) gyökérben tárolja a repó, és minden agent munkamenete elején betöltődik.

### 0.5 Kötelező munkamód: Research-then-Code + Cross-Agent Challenge

A projekt két alapszabálya — Norbi explicit kérése, mert „csak egy lehetőségünk van ezt elkészíteni jól".

#### 0.5.1 Research-then-Code

**Szabály:** Minden task **első outputja** `docs/understanding/T<x.y>.md` — NEM kód. Külön commit (`T<x.y>: understanding`), hogy a research a git history-ban olvasható maradjon.

**Hossza taszk-komplexitás szerint:**
- **S effort (≤ ½ nap):** **3-5 sor** — cél, érintett fájl(ok), kulcs library, ADR-worthy? igen/nem.
- **M effort (1-2 nap):** **8-15 sor** — fenti + 2-3 edge case + acceptance criteria saját szavakkal.
- **L / XL (3+ nap, kritikus integráció — Barion / Számlázz / NAV / migráció):** **15-30 sor** — fenti + alternatívák pro/con + min. 3 edge case + decision points + kérdések Norbi-nak.

**Sablon (M-méretű minimum, S-nél ennek a fele elég):**

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

**Tilos:** skip-elni „elég triviális" indoklással (ha tényleg triviális, 3 sor — az is OK); másolni-illeszteni a build spec szövegéből (saját szavak); a kód-commit-tal egy commitba tenni.

#### 0.5.2 Cross-Agent Decision Challenge

**Szabály:** Minden ADR-worthy döntést **kötelezően** megkérdőjelez egy `challenger` sub-agent (lásd Függelék H, `.claude/agents/challenger.md`), **MIELŐTT** kód kerülne a repóba.

**Mi az ADR-worthy?**
- Architektúra-szintű választás (monorepo vs multi-repo, fájlszerkezet, szolgáltatás-szétválasztás).
- Library / framework / SDK választás (pl. Barion: saját HTTP wrapper vs `node-barion` SDK).
- Security trade-off (pl. „ezt a rate limit-et az edge-en vagy a backenden tegyük?").
- Adatmodell-választás (pl. LED-attribútumokat custom modulban vagy metadata-mezőben tároljuk?).
- Külső szolgáltató választás, ha alternatíva létezik (pl. Bunny vs Cloudflare Images).
- Bármi, ami 6 hónap múlva fáj, ha rosszul döntünk.

**Mi NEM ADR-worthy** (ezekre nem kell challenger, elég a code-reviewer később):
- Egy adott függvény elnevezése.
- Egy adott CSS class név.
- Egy hibaüzenet szövege.
- Trivilális kódstruktúra (if-else vs early return).

**A challenger NEM a code-reviewer.** Két különböző szerepkör:

| Szerepkör | Mikor? | Mit néz? |
|---|---|---|
| **challenger** | KÓD ELŐTT (proposal phase) | Design, javaslat, library-választás, architektúra. „Ez egyáltalán a jó megközelítés?" |
| **code-reviewer** | KÓD UTÁN (PR phase) | Diff: stílus, hibák, tesztek, tisztaság. „Ez a megvalósítás helyes?" |

**Mit kérdez a challenger? (Min. 3 érv kötelező — rubber-stamp kizárva.):**

- „Mi a kockázata ennek a választásnak?"
- „Milyen alternatívákat mérlegeltél, és miért nem azokat választottad?"
- „Mit néznél át először, ha 6 hónap múlva újra ránéznél?"
- „Milyen edge case esetén dől össze ez a megoldás?"
- „Mit mondana ennek egy magyar e-commerce specialista, aki látott már 10 NAV-bukást?"
- „Ha 10× több termék lenne, ez még mindig működne?"
- „Ha Norbi 1 év múlva eladja a boltot, ez a választás vendor-lock-in?"

**Workflow:**

1. Agent ír `understanding.md`-t (0.5.1).
2. Agent felismeri, hogy ADR-worthy döntés van → `/challenge` slash command futtatása.
3. `challenger` sub-agent (külön Task tool spawn) elolvassa az `understanding.md`-t és a build spec érintett T<x.y> szakaszát.
4. Challenger generál egy `challenge-T<x.y>.md`-t (`/docs/challenges/T<x.y>.md`):
   ```markdown
   # T<x.y> Challenge — <döntés tárgya>

   **Dátum:** YYYY-MM-DD
   **Challenger agent:** <name>

   ## Az eredeti javaslat (1-2 mondat)
   ...

   ## Érvek a javaslat ELLEN (min. 3)
   1. **<kockázat 1>** — <részletes kifejtés>
   2. **<alternatíva, amit nem mérlegelt>** — <részletes kifejtés>
   3. **<long-term concern>** — <részletes kifejtés>
   (4-5. érv, ha vannak.)

   ## Mit kell az agentnek megválaszolnia, mielőtt zöld?
   - <kérdés 1>
   - <kérdés 2>
   ```
5. Eredeti agent (vagy orchestrator) **érdemi** választ ír — minden érvre. NEM elég „köszönöm, megfontoltam"; konkrét válasz, hogy miért tartja magát a javaslathoz vagy miért módosít.
6. Ha a challenger érveire ÉRDEMI válasz született → `/decide` command: ADR megírása (`/docs/adr/<NNNN>-<slug>.md`), és csak ekkor indul a kódolás.
7. Ha a challenger érveire NINCS érdemi válasz → vissza a tervezőasztalhoz: vagy módosítani kell a javaslatot, vagy a Norbi-szintű eszkaláció szükséges.

**Tilos:**
- Rubber-stamp challenge („Minden rendben, mehet."). Min. 3 ELLEN-érv kötelező.
- Az eredeti agentnek az érvek minden pontjára pontról-pontra **érdemi** kell válaszolni; egy összefoglaló mondat nem elég.
- A challenger ugyanaz a Claude session, mint az eredeti agent? **Nem!** Külön sub-agent invocation a Task tool-on keresztül, hogy friss kontextusban, „elfogultság nélkül" lássa a javaslatot.
- ADR-worthy döntés ADR nélkül commitolva — pre-commit hook blokkolja, ha nincs `docs/adr/` fájl az utolsó 24 órában a TaskUpdate-elt task-ra.

**Acceptance:** minden ADR-worthy döntéshez tartozik:
- `/docs/understanding/T<x.y>.md`
- `/docs/challenges/T<x.y>.md`
- `/docs/adr/NNNN-<slug>.md`
- És csak ezután a kód-commit.

#### 0.5.3 Globális acceptance criteria minden taskra

**Egy mondatban:** minden task `understanding.md`-vel indul, minden ADR-worthy döntés `challenger`-en megy át (külön sub-agent, min. 3 ELLEN-érv) és ADR-rel zárul, és a kód-commitnak `code-reviewer` ✓ + (ha security checklist van) `security-auditor` ✓ a feltétele.

CI ellenőrzés a pre-commit hookban (lásd Függelék H) — ha bármi hiányzik az aktuális T<x.y>-hoz, a kód-commit blokkolva van.

#### 0.5.4 „ELLENŐRIZD" jelölések eljárása (CC-1, P0)

A build spec-ben minden ⚠️ ELLENŐRIZD vagy ELLENŐRIZD jelölés egy nyitott kérdés, amit valaki, valamikor, valamilyen módon meg kell oldjon. **Tilos a feloldás nélkül a kapcsolódó task-ot completed-re állítani.**

**Minden ELLENŐRIZD jelölés mellé kötelezően társul:**

1. **Felelős** — `NORBI` (üzleti / jogi / könyvelői döntés) vagy `agent:<role>` (technikai research).
2. **Időablak** — `Fázis 0 vége előtt`, `task indulása előtt`, `task lezárása előtt`, vagy konkrét dátum.
3. **Megoldási hely** — `docs/checks/<task-id>.md` fájl, amibe a választ írjuk a kérdésre.

**CI ellenőrzés:** `tools/ci/check-ellenorizd-resolved.sh` — minden taszk lezárása előtt megnézi, hogy az érintett `docs/checks/<task-id>.md` létezik-e és minden ELLENŐRIZD-pontra van válasz. A pre-deploy-production hook is futtatja.

---

## 1. Vezetői összefoglaló

### 1.1 Mit építünk

Egy **modern, headless e-commerce platformot** a ledcenter.hu LED-világítástechnikai webáruház számára, amely:

- Lecseréli a meglévő ShopRenter alapú boltot.
- Minden meglévő terméket (~1280) átmigrál.
- 100%-ban megfelel a magyar e-commerce jogszabályoknak (NAV Online Számla 3.0, 45/2014 Korm. rendelet, GDPR, fogyasztói tájékoztatás).
- SEO/GEO/LLM optimalizált — Core Web Vitals 90+, schema.org Product/Offer/Breadcrumb, llms.txt, sitemap.
- Magyar piacra szabott fizetés (Barion) és számlázás (Számlázz.hu → NAV).
- Admin felület testreszabva LED-es termékadatokra (lumen, watt, foglalat, IP, élettartam, szín-hőmérséklet).
- Production-ready Railway-en, GitHub-os CI/CD-vel.

### 1.2 Stack (kötött döntések)

| Réteg | Választás | Indoklás |
|---|---|---|
| Backend + admin | **Medusa.js 2.x** | Headless, modular, MIT, JS-ekosystem, kiterjeszthető plugin/module-okkal. |
| Storefront | **Next.js 15 App Router** (Server Components, ISR) | Best-in-class SEO, kiváló DX, Vercel/Railway-kompatibilis. |
| Adatbázis | **PostgreSQL 16** (Railway) | Medusa követelmény, transaction-safe. |
| Cache / queue | **Redis 7** (Railway) | Medusa workflow, BullMQ. |
| Payment | **Barion Smart Gateway** | Magyar piac vezető PSP, alacsony díj, jó docs. |
| Számlázás | **Számlázz.hu** (`ewngs/szamlazz.js` SDK) | NAV Online Számla 3.0 kompatibilis, automatikus. |
| Email | **Resend + React Email** | DX, deliverability, tranzakcionális. |
| Search | **Meilisearch self-hosted** vagy **Meilisearch Cloud** | Typo-tolerant, gyors, faceting, magyar Latin alphabet. |
| Image storage | **Cloudflare R2** | $0 egress, $0.015/GB. |
| Image transformation | **Cloudflare Images** vagy **next/image + Bunny CDN** | Költségfüggő (lásd T2.4 decision tree). |
| Monitoring | **Sentry** (errors) + **PostHog** (product analytics) + **Better Stack** (uptime) | Open standard, magyar GDPR-friendly opciók. |
| Hosting | **Railway** | Norbi választása, jó Postgres+Redis integráció. |
| CI/CD | **GitHub Actions** | Standard. |
| DNS / WAF | **Cloudflare** (proxied) | Ingyen DDoS / WAF / SSL. |
| Frontend dizájn | **Google Stitch** (Stitch MCP) | Norbi választása. |

### 1.3 Idő- és költségbecslés

**Idő:** ~14-16 hét agent-team-mel (40 óra/hét intenzitásnál).

**Fix infrastruktúra (forgalom-független): ~$100-200 / hó.**

**Variable költségek (Barion fee, Anthropic API):**

| Forgatókönyv | Fix | Variable | **Összesen** |
|---|---:|---:|---:|
| Indulás (100 rendelés/hó) | ~$130 | ~$60 | **~$190** |
| Mid (500 rendelés/hó) | ~$140 | ~$235 | **~$375** |
| Komfort (1 000 rendelés/hó) | ~$160 | ~$455 | **~$615** |
| Skála (5 000 rendelés/hó) | ~$200 | ~$2 175 | **~$2 375** |

### 1.4 A 3 legnagyobb kockázat

1. **NAV Online Számla integráció hibája.** Mitigáció: Számlázz.hu-ra hagyjuk a NAV-protokoll bonyolódást; teljes E2E teszt sandbox környezetben minden számlatípusra; retry queue minden NAV-hibára.
2. **SEO-veszteség az átállás során.** Mitigáció: kötelező 1-1 mappolás minden régi URL-ről (T8.5); Search Console pre-launch verification; staging-en SEO audit.
3. **Barion fizetési flow edge-case-ek.** Mitigáció: idempotent callback handler; reconciliation cron 6 óránként; sandbox QA minden lehetséges hibaúton.

---

## 2. Architektúra-áttekintés

### 2.1 Komponens-diagram

```
                                ┌──────────────────────────────┐
                                │     Cloudflare (DNS + WAF)   │
                                │     ledcenter.hu, www, api.* │
                                └───────────┬──────────────────┘
                                            │
                ┌───────────────────────────┼───────────────────────────┐
                │                           │                           │
                ▼                           ▼                           ▼
   ┌─────────────────────┐   ┌──────────────────────────┐   ┌────────────────────┐
   │  Next.js 15         │   │  Medusa.js 2.x Server    │   │  Medusa.js 2.x     │
   │  Storefront         │◄──┤  (admin + Store API)     │   │  Worker            │
   │  ledcenter.hu       │   │  api.ledcenter.hu        │   │  bg jobs           │
   └─────────┬───────────┘   └────────┬─────────────────┘   └─────────┬──────────┘
             │                        │                               │
             │      ┌─────────────────┼───────────────────────────────┤
             │      │                 │                               │
             ▼      ▼                 ▼                               ▼
   ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────────────┐
   │  Meilisearch     │   │  PostgreSQL 16   │   │  Redis 7                 │
   │  (search index)  │   │  (Medusa data)   │   │  (cache + queue + lock)  │
   └──────────────────┘   └──────────────────┘   └──────────────────────────┘
```

### 2.2 Adatfolyam — vásárlás → fizetés → számla → NAV → email

```
[1] Vásárló a Next.js storefronton "Megrendelés" gombra kattint
[2] Storefront → Medusa Store API: cart complete + payment session create
[3] Medusa Barion payment provider: POST /v2/Payment/Start
[4] Storefront redirect a Barion GatewayUrl-re (3DS2 flow)
[5] Vásárló sikeresen fizet → Barion redirect ledcenter.hu/checkout/success
[6] Barion → POST callback https://api.ledcenter.hu/barion/callback (PARALLEL)
[7] Medusa callback handler: IP allowlist + Payment/State + idempotency
[8] Medusa workflow: order create
[9] Medusa subscriber on order.placed: Számlázz.hu createInvoice + NAV
[10] Medusa subscriber on invoice.created: Resend email vásárlónak + adminnak
[11] Shipping label create (FoxPost/MPL/GLS) + tracking URL
```

### 2.3 Környezetek

| Környezet | Domain | DB | Barion | Számlázz | NAV |
|---|---|---|---|---|---|
| **dev** | localhost | Lokális Postgres | sandbox | sandbox | NAV teszt |
| **staging** | staging.ledcenter.hu | Railway staging | sandbox | sandbox | NAV teszt |
| **production** | ledcenter.hu | Railway prod (PITR) | live | live | NAV éles |

---

## 3. Agent-csapat felépítése

### 3.0 Workflow (kötelező sorrend)

`TaskList → /task <T<x.y>> → understanding.md (külön commit) → ha ADR-worthy: challenger → érdemi válasz → ADR (külön commit) → kód (Karpathy loop) → code-reviewer → security-auditor (ha checklist) → integration-tester (Fázis 4, 8) → TaskUpdate completed`.

### 3.1 Karpathy-minták

A felhasználó által említett "Kárpáti programozó" valószínűleg **Andrej Karpathy**. A releváns minták:

- **karpathy/autoresearch** — agent loop: read code → propose change → run test → measure → keep/discard → repeat. A **Goal-Driven Execution** elv inspirálja.
- **karpathy/agenthub** — több agent koordinációja. Átültetjük a `.claude/agents/` szerepkör-definíciókba.
- **forrestchang/andrej-karpathy-skills** — `CLAUDE.md` 4 alapelv. Teljes egészében átemeljük a `CLAUDE.md`-be.

### 3.2 Agent-szerepkörök

| Szerepkör | Felelősség | Mikor? |
|---|---|---|
| **orchestrator** | Plan, task selection, koordináció. | Mindig — fő session. |
| **code-writer** | Kód implementálás. | Új feature task (ELŐFELTÉTEL: understanding+ADR kész). |
| **challenger** | Design / library / architektúra megkérdőjelezése KÓD ELŐTT. Min. 3 ELLEN-érv. | Minden ADR-worthy döntés. |
| **code-reviewer** | Diff review (stílus, helyesség). | Minden commit után. |
| **security-auditor** | OWASP / GDPR / NAV-jog ellenőrzés. | Fázis 4, 7, 8 + új external integration. |
| **integration-tester** | E2E kritikus utak. | Fázis 4 lezárása + cutover előtt. |
| **seo-checker** | Lighthouse, schema.org, sitemap. | Fázis 6 + minden production deploy előtt. |
| **doc-writer** | ADR, runbook, decision log. | Minden fázis lezárása + új ADR. |
| **migration-planner** | ShopRenter → új stack (Fázis 8). | Fázis 8 tervezése + felügyelet. |

### 3.3 Verification loop (Karpathy `autoresearch`)

```
1. PLAN — TodoWrite 3-7 lépésre.
2. PROPOSE — első iteráció.
3. RUN — tesztek + build.
4. MEASURE — acceptance criteria.
5. ITERATE — sebészi módosítás, max 5×.
6. REVIEW — code-reviewer agent.
7. SECURITY — security-auditor (ha checklist).
8. COMMIT — `T<x.y>: <cím>`.
9. UPDATE — TaskUpdate completed.
```

### 3.5 Slash command-ok

| Parancs | Cél |
|---|---|
| `/task <T<x.y>>` | **Umbrella-parancs** — research → ha ADR-worthy: challenger → ADR → kód → review → security audit. |
| `/security-audit [N]` | security-auditor az utolsó N committra. |
| `/seo-check` | seo-checker: Lighthouse + schema. |

### 3.6 Hooks

- **`pre-commit`** — lint + typecheck + test:unit + research-fájl ellenőrzés.
- **`pre-deploy-production`** — security-auditor + seo-checker + integration-tester.

---

## 4. Repo struktúra és kezdeti setup

### 4.1 Repo döntés: monorepo

**Választás:** `pnpm` workspaces + `turborepo` monorepo. ADR-0001.

### 4.2 Fájlszerkezet

```
ledcenter/
├── .claude/
│   ├── agents/         # 9 sub-agent definicio
│   ├── commands/       # /task, /security-audit, /seo-check
│   └── hooks/          # pre-commit, pre-deploy-production
├── CLAUDE.md
├── README.md
├── package.json
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── apps/
│   ├── storefront/     # Next.js 15
│   └── backend/        # Medusa.js 2.x
├── packages/
│   ├── shared-types/
│   ├── eslint-config/
│   └── tsconfig/
├── tools/
│   ├── shoprenter-migrator/  # Fazis 8
│   └── seo-audit/
├── docs/
│   ├── adr/
│   ├── understanding/
│   ├── challenges/
│   ├── checks/
│   ├── runbooks/
│   ├── legal/
│   ├── brand/
│   └── build-spec.md
└── .github/workflows/
```

### 4.3 Branch + commit konvenció

- Branchek: `main` (staging deploy), `production` (tag-elt). Feature: `feat/T<x.y>-<rovid-cim>`.
- Commit: `T<x.y>: <imperative title>` vagy `chore:`, `fix:`, `docs:`. Conventional Commits.
- PR: squash merge `main`-be. Min. 1 review (code-reviewer agent).

### 4.6 ESLint + Prettier + TS strict

- ESLint flat config: tilos library-k (lodash, moment, axios) + tilos hard-coded ÁFA (`* 1.27`, `vat: 27`).
- Prettier default + `printWidth: 100`, `singleQuote: true`, `trailingComma: 'all'`.
- TypeScript: `strict: true`, `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`.

---

## 5. Fázisok és sprintek

> **Olvasási útmutató:** minden fázis kezdete `### Fázis X — Cím` szekcióval, **célt**, **belépési feltételt**, **kilépési feltételt** + a taskok listáját. Minden task egy `#### T<x.y> Cím` szekció. Az agent egyszerre EGY taskon dolgozik.

### Task sablon

- **Leírás** — 1-3 mondat.
- **Inputok** — mire épül.
- **Outputok** — mit hoz létre.
- **Acceptance criteria** — 3-5 ellenőrizhető pont.
- **Library / repo** — konkrét NPM csomag, GitHub URL, verzió.
- **Security checklist** — 3-5 pont.
- **Test követelmény** — unit / integration / E2E.
- **Decision tree** — kétértelmű helyzetekre.
- **Effort** — S (≤½ nap) / M (1-2 nap) / L (3-5 nap) / XL (1-2 hét).
- **Dependencies** — mely T<x.y>-k kell előbb kész legyenek.

---

### Fázis 0 — Pre-flight (1 hét)

**Cél:** üzleti döntések, fiókok, jogi alap megléte. **Nem kódolás.**
**Belépési feltétel:** Norbi-jóváhagyás erre a build specre + minden „Fázis 0 vége" deadline-os ADR megírva (ADR-0001, 0008, 0009, 0010 kötelező).
**Kilépési feltétel:** minden T0.x task completed; minden szükséges külső fiók aktív, sandbox kulcsok megvannak.

#### T0.1 Domain és DNS előkészítés

**Leírás:** A `ledcenter.hu` domain és DNS-szolgáltató felmérése; Cloudflare-be migrálás tervezett (ingyen WAF, DDoS, SSL, gyors DNS).

**Outputok:**
- Cloudflare account aktív, `ledcenter.hu` domain rajta van pending státuszban (NEM aktiválunk DNS váltást Fázis 9-ig).
- DNS rekordok dokumentálva (`docs/dns/baseline-2026-MM-DD.csv`).
- TTL csökkentve 300-ra (5 perc) min. 24 órával a cutover előtt.

**Acceptance criteria:**
1. Cloudflare dashboard-on a domain pending validation státuszban.
2. Jelenlegi DNS export PDF / CSV elmentve.
3. SPF, DKIM, DMARC rekordok dokumentálva.

**Security checklist:**
- [ ] Cloudflare 2FA bekapcsolva.
- [ ] API token-ek scope-olva (Zone:Edit, Workers:NA).
- [ ] DNSSEC bekapcsolva, ha a regisztrátor támogatja.

**Effort:** S
**Dependencies:** —

#### T0.2 Barion fiók nyitása, sandbox kulcsok

**Outputok:**
- Sandbox `POSKey` (UUID) → `BARION_POSKEY_SANDBOX`.
- Sandbox felhasználó username/password.
- Live `POSKey` regisztrált (de nem használt Fázis 9-ig).
- Webhook IP-allowlist címek dokumentálva.

**Acceptance criteria:**
1. Sandbox környezetben sikeres test fizetés.
2. POSKey és Pixel ID dokumentálva `docs/integrations/barion.md`-ben.
3. Live fiók státusza "Approved" vagy "In review".

**Security checklist:**
- [ ] POSKey csak Railway env-ben.
- [ ] Sandbox és live POSKey külön env változó.
- [ ] Barion fiók 2FA.

**Effort:** S
**Dependencies:** —

#### T0.3 Számlázz.hu fiók, API kulcs, NAV regisztráció

**Outputok:**
- Számlázz.hu auth token → `SZAMLAZZ_AGENT_KEY`.
- NAV Online Számla regisztráció a Számlázz.hu felé delegálva, technikai user megvan.
- Test fiók aktív.

**Acceptance criteria:**
1. Számlázz.hu felületen kézi test számla kiállítva, NAV teszt rendszerbe bement.
2. API auth token működik.
3. NAV Online Számla portálon a teszt számla látszik.

**Security checklist:**
- [ ] Auth token csak env változóban.
- [ ] Számlázz.hu fiók 2FA.
- [ ] NAV technikai user jelszava jelszókezelőben.

**Effort:** M (NAV adminisztráció időigényes)
**Dependencies:** —

#### T0.4 Railway projekt, GitHub repo

**Outputok:**
- GitHub repo `ledcenter` (privát).
- Railway projekt `ledcenter` 3 environment-tel: `dev`, `staging`, `production`.
- Railway services: `backend-server`, `backend-worker`, `storefront`, Postgres add-on, Redis add-on.

**Security checklist:**
- [ ] Repo privát.
- [ ] Railway 2FA.
- [ ] GitHub Actions secrets-be Railway service token.
- [ ] Branch protection a `main`-en.

**Effort:** S
**Dependencies:** —

#### T0.5 ÁSZF / adatkezelési tájékoztató sablon

**Outputok:**
- `docs/legal/aszf.md` (ÁSZF — 45/2014 11. § (1) szerint).
- `docs/legal/adatkezelesi-tajekoztato.md` (privacy policy GDPR cikk 13-14).
- `docs/legal/elallasi-nyilatkozat-minta.md` (45/2014 függelék 2.).
- `docs/legal/cookie-tajekoztato.md`.

**Acceptance criteria:**
1. ÁSZF tartalmazza a kötelező 45/2014 pontokat.
2. Adatkezelési tájékoztató minden processor-t felsorol.
3. Elállási mintanyilatkozat szövege a kormányrendelet 2. melléklettel SZÓRÓL-SZÓRA.
4. **Norbi vagy ügyvédje aláírja: "OK / módosítandó".**

**Effort:** M (ügyvéd review-tól függ)
**Dependencies:** —

#### T0.6 Logó, márkaszínek, fontok (Stitch input)

**Outputok:**
- `docs/brand/logo.svg` (és .png 2x, 3x, fav, apple-touch).
- `docs/brand/colors.md` (HEX, RGB, OKLCH).
- `docs/brand/fonts.md` (Google Fonts vagy Fontshare).
- `docs/brand/stitch-prompt.md` — kész prompt a Stitch-be.

**Acceptance criteria:**
1. Logó vector SVG (transparent bg, dark + light).
2. Min. 3 brand szín + 2 neutral.
3. WCAG AA contrast ratios átmennek.
4. Font choice szabad licencű.

**Effort:** S
**Dependencies:** —

#### T0.7 ShopRenter API hozzáférés

**Outputok:**
- ShopRenter `client_id`, `client_secret`, `shop_id` env-be.
- Hozzáférés tesztelve egy `GET /products?limit=1` hívással.

**Acceptance criteria:**
1. OAuth2 token sikeresen szerezhető.
2. `/products?limit=1` 200 OK.
3. Rate limit dokumentálva.

**Security checklist:**
- [ ] Client secret env-ben.
- [ ] OAuth2 scopes minimumra (read-only).

**Effort:** S
**Dependencies:** —

---

### Fázis 1 — Foundation (1-2 hét)

**Cél:** működő, üres Medusa + Next.js stack staging-en. Auth, monitoring, CI alapok.

#### T1.1 Monorepo init, package manager választás

pnpm workspace + turborepo struktúra a 4.2 fejezet szerint.

#### T1.2 Medusa 2.x backend setup

`apps/backend` Medusa 2.x bázis: `src/api`, `src/modules`, `src/workflows`, `src/subscribers`, `src/jobs`, `src/admin`, `medusa-config.ts`.

#### T1.3 Postgres + Redis Railway-en

**G-20 fix — RPO/RTO célok (lásd ADR-010):**
- **RPO: 15 perc** — max ennyi adatvesztés tűrhető. Railway PITR ezt biztosítja.
- **RTO: 4 óra** — egy fél munkanap.

**Off-site backup:** napi `pg_dump` → Cloudflare R2 (másik régió) cron, 30 nap retention.

**DR runbook:** `docs/runbooks/disaster-recovery.md` — havi DR-drill.

**⚠️ ELLENŐRIZD:** Railway Postgres encryption-at-rest (G-35). Felelős: NORBI; határidő: Fázis 0 vége előtt; válasz: `docs/checks/T1.3.md`.

#### T1.4 Next.js 15 storefront setup

**Security headers:**
- CSP (strict default-self + Sentry/PostHog allowlist nonce-szal).
- X-Frame-Options: DENY (kivéve checkout sandbox iframe).
- Referrer-Policy: strict-origin-when-cross-origin.
- Permissions-Policy: geolocation=(), microphone=(), camera=().

#### T1.5 Storefront ↔ Backend connection (Medusa SDK)

`@medusajs/js-sdk` configured Medusa instance.

#### T1.6 Auth (admin + customer)

Medusa beépített customer auth + admin auth.

#### T1.7 Base CI/CD GitHub Actions

`ci.yml`: lint + typecheck + test:unit + build minden PR-ra.

#### T1.8 Sentry, PostHog setup

`@sentry/nextjs` + `posthog-js` + EU region GDPR-konform.

#### T1.9 CLAUDE.md és sub-agentek bekonfigurálása

A `.claude/` mappa teljes felépítése (lásd Függelék H).

---

### Fázis 2 — Catalog (1-2 hét)

#### T2.1 Termék schema (LED-specifikus + ÁFA + ár-modell)

**G-07 fix — ÁFA-mező:** `vatRate: 5 | 18 | 27` (default 27%, kötelező mező). Hard-coded 27% TILOS.

**G-13 fix — Nettó / ÁFA / Bruttó:**
- `priceNet: number` (fillér, kanonikus)
- `priceGross: number` (számolt)
- `priceVat: number` (számolt)
- Banker's rounding (`ROUND_HALF_EVEN`), 2 tizedes pontosság (fillér).

**Zod schema:**
```typescript
export const VatRate = z.union([z.literal(5), z.literal(18), z.literal(27)])
export const Price = z.object({
  priceNet: z.number().int().nonnegative(),
  priceGross: z.number().int().nonnegative(),
  priceVat: z.number().int().nonnegative(),
}).refine(p => p.priceVat === p.priceGross - p.priceNet)
```

**LED attribútumok:** lumen, wattage, socket, ipRating, lifetime, colorTemp, cri, dimmable, energyClass + `vatRate`.

#### T2.2 Kategória fa

Medusa product_category, max 3 szint. URL slug magyar ékezetek nélkül, kötőjellel.

#### T2.3 Image storage (Cloudflare R2)

**G-11 fix — file upload mélyebb védelem:**
- Magic-bytes ellenőrzés (`file-type` package).
- SVG és PDF upload TILTVA.
- Decompression bomb védelem (max 50 MP).
- Méret-korlát (max 10 MB / kép, max 4096×4096 px).
- EXIF metadata strip (`sharp.withMetadata({})`).
- Filename sanitization.

#### T2.4 Image transformation (decision)

**Ajánlás:** Bunny.net Optimizer ($9.50/mo flat).

#### T2.5 Product list page (PLP)

Szűrők, sort, lapozás, ISR (`revalidate: 300`).

#### T2.6 Product detail page (PDP)

**G-16 fix — kliens-oldali fresh inventory check:** SWR 30s, on-demand revalidate Medusa subscriber-ből.

JSON-LD `Product` + `Offer` + `BreadcrumbList`.

#### T2.7 Search (Meilisearch)

Termék-search, typo-tolerant, magyar Latin alphabet, faceting.

#### T2.8 Készlet kezelés

Medusa Inventory module.

---

### Fázis 3 — Cart, Checkout (1-2 hét)

#### T3.1 Cart state

Medusa cart, vendég cookie, login fold-in.

#### T3.2 Checkout step 1 — addresses, shipping

react-hook-form + zod, magyar IRSZ-város mapping (opcionális).

#### T3.3 Checkout step 2 — payment method selection

Barion / utánvét / banki átutalás.

#### T3.4 Order summary, ÁSZF / adatkezelés checkbox kötelező

KÖTELEZŐ checkbox: "Elolvastam és elfogadom az [ÁSZF-et](/aszf) és az [Adatkezelési tájékoztatót](/adatkezeles)."

**G-32 fix — e-számla GDPR:** explicit tájékoztatás vagy checkbox.

#### T3.5 14 napos elállás link, fogyasztói tájékoztatás

Footer + ODR link.

---

### Fázis 4 — Hungarian Integrations (KRITIKUS — 2-3 hét)

> **Minden T4.x lezárása ELŐTT security-auditor + integration-tester agent KÖTELEZŐ.**

#### T4.1 Barion payment provider modul

**G-03 fix — idempotens callback handler háromrétegű:**
1. Redis SETNX lock (`barion:cb:<PaymentId>`, 7 nap expiry).
2. DB unique constraint a `payment.barion_payment_id`-n.
3. Payment/State újra-lekérdezés Barion-tól.

**G-01 fix:** `PaymentRequestId` UUID v4 (max 100 char).

**Acceptance criteria:**
1. Sandbox sikeres fizetés E2E.
2. Sandbox failed fizetés.
3. Sandbox refund (full + partial).
4. Callback IP allowlist.
5. Idempotens (5× ugyanaz a callback → 1× order).
6. Timeout retry × 3 exponential backoff.

#### T4.2 Számlázz.hu integráció

**G-13 fix:** `vat`, `netUnitPrice` a termék `vatRate` és `priceNet` mezőiből (NEM hard-coded).

**G-04 fix — Sztornó vs helyesbítő (KÖNYVELŐI DÖNTÉS, ⚠️ ELLENŐRIZD):**

| Esemény | Opció A (default, magyar gyakorlat) | Opció B (alternatíva) |
|---|---|---|
| Teljes refund | SZTORNÓ | SZTORNÓ |
| Részleges refund | HELYESBÍTŐ (negatív sor) | SZTORNÓ + új számla a maradékra |
| Adat-javítás | HELYESBÍTŐ | HELYESBÍTŐ |

Feature-flag: `config.invoice.partialRefundFlow = 'helyesbito' | 'storno_plus_new'`. Default: `helyesbito`.

⚠️ ELLENŐRIZD KÖNYVELŐVEL. `docs/checks/T4.2.md`.

**G-05 fix — Számla PDF megőrzés:** R2 Object Lock, 8 év + 1 hónap retention. 2000. évi C. tv. 169. §.

**G-08 fix — Currency assertion:** v1 csak HUF.

#### T4.3 NAV Online Számla validáció + hibakód-alapú retry-policy

**G-06 fix — Hibakód-térkép (`docs/runbooks/nav-error-codes.md`):**

| Kategória | Példa kód | Default reakció | Retry? |
|---|---|---|---|
| INFO | bármi | log only | n/a |
| WARN | bármi | log + dashboard | nem |
| Tartalmi ERROR (érvénytelen vevő-adat) | 5402 | NE retry — admin manual fix | NEM |
| Tartalmi ERROR (érvénytelen áfa) | 5403 / 591 | NE retry — javítás + helyesbítő | NEM |
| Duplikált bizonylat | 5404 | ignore | NEM |
| Tartalmi egyéb | 5xxx | admin manual review | NEM |
| Technikai NAV szerver | 5xx HTTP | retry exponential (1m, 5m, 15m, 30m, 60m) | IGEN |
| Auth | 403 | admin alert (Norbi újrahitelesítés) | NEM |

**24h-túli sikertelen beküldés:** ⚠️ ELLENŐRIZD Számlázz.hu support-tól (a/b kérdés). `docs/checks/T0.3.md` + `docs/runbooks/nav-error-codes.md`.

#### T4.4 Adószám validálás (NAV publikus API)

B2B vásárlás: NAV adószám-ellenőrző API.

#### T4.5 Szállítási integráció — FoxPost, MPL, GLS

3 magyar szolgáltató + csomagpont-választó widget.

#### T4.6 Email tranzakcionális (Resend)

React Email template-ek: order-placed, shipped, delivered, cancelled, password-reset, welcome, admin-new-order, nav-error-alert.

#### T4.7 Edge case-ek dokumentálása

Reconciliation cron 6 óránként. 10+ failure mode dokumentálva.

---

### Fázis 5 — Admin UX (1-2 hét)

T5.1 Medusa admin testreszabás (LED-attribútum widget, NAV-státusz widget, shipping label widget).
T5.2 Bulk product import (CSV wizard).
T5.3 Bulk image upload.
T5.4 Termékleírás generátor (Claude API).
T5.5 Image alt text generátor (Claude vision).
T5.6 Készletkezelő quick-edit.
T5.7 Rendeléskezelő dashboard.

---

### Fázis 6 — SEO / GEO / LLM (1-2 hét)

T6.1 schema.org markup (Product + Offer + BreadcrumbList + Organization + WebSite + FAQPage).
T6.2 Dinamikus sitemap.xml.
T6.3 robots.txt (`/admin`, `/checkout`, `/account` Disallow).
T6.4 llms.txt + llms-full.txt.
T6.5 Open Graph, Twitter Card.
T6.6 ISR konfiguráció + on-demand revalidation.
T6.7 Core Web Vitals audit (LCP <2.5s, INP <200ms, CLS <0.1).
T6.8 Hungarian-language SEO (meta description 120-155 char, H1 unique).
T6.9 Belső linkelés.
T6.10 Külső SEO eszköz integrációs pont (Search Console, GA4/PostHog).

---

### Fázis 7 — Security Hardening (1-1.5 hét)

T7.1 OWASP Top 10 audit.
T7.2 Rate limiting (Upstash Ratelimit).
T7.3 DDoS / WAF (Cloudflare proxy + Cloudflare Access az admin route-ra).
T7.4 Secrets management (gitleaks).
T7.5 2FA admin (TOTP).
T7.6 PCI-DSS minimum (SAQ A scope).
T7.7 GDPR-flow (delete + export user data).
T7.8 Cookie consent.
T7.9 Penetration test (külső, opcionális).

#### T7.9b Customer impersonation policy (G-10 fix, P0)

**Default v1: TILTOTT.** ADR-009. Lint-szabály blokkolja a `impersonate`, `loginAs`, `actAsCustomer` minta-előfordulásokat.

T7.10 Security incident response runbook.

---

### Fázis 8 — Migration ShopRenter → Új stack (1-2 hét)

T8.1 ShopRenter exporter (egyszer használatos TS script `tools/shoprenter-migrator/`).
T8.2 Schema mapping (Zod validation, LED-attribútumok átkonvertálása).
T8.3 Image migration (~1280 termék × ~5 kép → ~6400 kép R2-ban).
T8.4 Customer + order migráció (opcionális, default SKIP).
T8.5 **301 redirect térkép — KRITIKUS SEO.** Cloudflare Page Rules vagy Next.js middleware.
T8.6 Staging deploy + UAT.
T8.7 Pre-cutover audit.

---

### Fázis 9 — Go-Live (1 hét)

T9.1 DNS átállítás (TTL 300, cutover szombat éjszaka 02:00-04:00).
T9.2 Cutover ütemterv (rollback plan: DNS revert + TTL 60s).
T9.3 Search Console, Bing Webmaster Tools.
T9.4 Monitoring dashboard (Sentry, PostHog, Better Stack, alerts).
T9.5 Post-launch hot fix protokoll.
T9.6 Hand-over Norbi-nak és barátainak (admin manual + Loom video session-ek).

---

## 6. Decision logs (ADR-ek)

**Kötelező ADR-ek a v1 indulásra (deadline-okkal):**

| # | Cím | Érintett task | Deadline |
|---|---|---|---|
| ADR-0001 | Monorepo (pnpm + turborepo) vs multi-repo | T1.1 | Fázis 0 vége |
| ADR-0002 | Image CDN (Bunny vs Cloudflare Images) | T2.4 | Fázis 1 vége |
| ADR-0003 | Search engine (Meilisearch self-hosted vs Cloud) | T2.7 | Fázis 1 vége |
| ADR-0004 | Customer auth (Medusa beépített vs NextAuth) | T1.6 | Fázis 1 elején |
| ADR-0005 | Payment provider modul (saját HTTP wrapper vs node-barion) | T4.1 | Fázis 4 elején |
| ADR-0006 | Customer + order migráció YES/NO | T8.4 | Fázis 8 elején |
| ADR-0007 | Penetration test (külső pen-test vs OWASP ZAP) | T7.9 | Fázis 7 közepén |
| **ADR-0008** | **Ár-tárolás: nettó alapú vs bruttó alapú** (G-13) | T2.1 | **Fázis 0 vége — KRITIKUS** |
| **ADR-0009** | **Customer impersonation policy** (G-10) | T7.9b | **Fázis 0 vége — egyszeri policy** |
| **ADR-0010** | **RPO/RTO célok** (G-20) | T1.3 | **Fázis 0 vége — Norbi go-ahead** |
| ADR-0011 | E2E mock vs sandbox (G-28) | Test stratégia | Fázis 1 vége |
| ADR-0012 | Szállítók sorrendje (FoxPost vs MPL vs GLS) | T4.5 | Fázis 4 elején |
| ADR-0013 | Multi-currency v1-ben? (G-08) | T4.1, T4.2 | Fázis 4 elején |

**Kötelező:** Fázis 0 belépési feltétele — minden „Fázis 0 vége" deadline-os ADR megírva (ADR-0001, 0008, 0009, 0010 kötelező).

---

## 7. Risk register

| # | Kockázat | Valószínűség | Hatás | Mitigation |
|---|---|---|---|---|
| 1 | NAV Online Számla bejelentés hibázik | M | H | Számlázz.hu delegálva; retry queue; admin alert; cron polling |
| 2 | SEO veszteség az átállás miatt | M | H | 301 redirect map (T8.5); Search Console; staging audit |
| 3 | Barion edge case-ek | M | H | Idempotent callback; reconciliation cron; sandbox tesztelés |
| 4 | Image migration kép-tört | M | M | Hash check; failed-list 1%-ra retry |
| 5 | Hosting (Railway) outage | L | H | Better Stack monitor; rollback DNS; Vercel backup |
| 6 | Magyar áfa hibás termékenként | L | M | Zod schema; Norbi-jóváhagyás új áfa-osztályra |
| 7 | DDoS attack | L | M | Cloudflare WAF + DDoS protection |
| 8 | GDPR vs számla 8 év konfliktus | M | L | Adatkezelési tájékoztató: számlák 8 év, profil-adat azonnali |
| 9 | Admin jelszó kompromittálás | L | H | 2FA kötelező; Cloudflare Access |
| 10 | Új termék-attribútumok jövőbeli igénye | H | L | Metadata flexible; Zod bővíthető |
| 11 | **Skipping research phase** | M | H | Pre-commit hook ellenőrzi understanding.md-t |
| 12 | **Rubber-stamp challenge** | M | H | challenger.md kötelezi min. 3 ELLEN-érvet, friss sub-agent |

---

## 8. Glossary

| Magyar | Angol | Magyarázat |
|---|---|---|
| Áru | Product | Medusa: `Product`. |
| Termékvariáns | Product variant | Medusa: `ProductVariant`. |
| Kosár | Cart | Medusa: `Cart`. |
| Megrendelés / Rendelés | Order | Medusa: `Order`. |
| Készlet | Inventory / Stock | Medusa: `InventoryItem`. |
| Áfa / ÁFA | VAT | Magyar áfa (5/18/27%). |
| Számla | Invoice | NAV Online Számla 3.0. |
| Sztornó | Cancelled invoice / Storno | Teljes érvénytelenítés. |
| Helyesbítő számla | Correction invoice | Részleges javítás. |
| Fogyasztói tájékoztatás | Consumer notification | 45/2014 11. § (1). |
| Elállási jog | Right of withdrawal | 14 napos visszavonás. |
| Csomagpont | Pickup point | FoxPost / MPL automaták / üzletek. |
| Lumen | Lumen (lm) | LED fényerő. |
| Wattage | Wattage (W) | Elektromos teljesítmény. |
| Foglalat | Socket | E14, E27, GU10, ... |
| IP védettség | IP rating | IP20-IP68. |
| Élettartam | Lifetime | Üzemórák száma. |
| Szín-hőmérséklet | Color temperature (K) | 2700K - 6500K. |
| CRI | CRI | Színvisszaadási index (0-100). |
| Headless commerce | Headless commerce | Frontend és backend különálló. |
| ISR | Incremental Static Regeneration | Next.js static + on-demand. |
| PSP | Payment Service Provider | Barion. |
| IPN | Instant Payment Notification | Webhook callback fizetésről. |

---

## 9. Függelékek

### Függelék B: Barion sandbox teszt-protokoll

| # | Forgatókönyv | Várható eredmény |
|---|---|---|
| 1 | Sikeres fizetés | Order created, invoice issued, email sent |
| 2 | Sikertelen fizetés (kártya elutasítva) | Cart `payment_failed`, NO invoice |
| 3 | 3DS challenge fail | Cart `payment_failed` |
| 4 | Vásárló bezárja a Barion gateway-t | reconciliation cron 30 perc → cancelled |
| 5 | Callback delayed | Storefront polling 30s után state-check |
| 6 | Duplikált callback | Idempotent: 1× order |
| 7 | Refund full | Sztornó számla |
| 8 | Refund partial | Helyesbítő (Opció A) vagy sztornó+új (Opció B) |
| 9 | Reservation flow | `payment_authorized`, capture by admin |
| 10 | Barion API down | Retry × 3, admin alert |

### Függelék C: Számlázz.hu test-protokoll

| # | Forgatókönyv | Várt invoice |
|---|---|---|
| 1 | B2C magán Magyar | Sima 27% áfa |
| 2 | B2B magyar adószámmal | Sima, adószám-mező |
| 3 | EU B2B (közösségi) | Fordított adózás label |
| 4 | EU B2C (más tagállam) | Magyar áfa, OSS-ig |
| 5 | EU-n kívüli | Áfa-mentes export |
| 6 | Kuponos kedvezmény | Külön sor |
| 7 | Ingyenes szállítás | Szállítás 0 Ft sor |
| 8 | Részleges refund | Helyesbítő (A) vagy sztornó+új (B) |
| 9 | Teljes refund | Sztornó |
| 10 | Több termék vegyes áfával | Helyesen szétosztva |

### Függelék E: SEO checklist (60+ pont — kivonat)

**Technical:** HTTPS+HSTS, WWW canonicalization, mobile-friendly, viewport, lang="hu".
**Indexability:** robots.txt, sitemap.xml, canonical, hreflang.
**Performance:** LCP<2.5s, INP<200ms, CLS<0.1, TTFB<800ms, lazy load, font-display:swap.
**On-page:** title 50-60 char, meta desc 120-155, H1 unique, alt text, no broken links.
**Schema.org:** Organization, WebSite, BreadcrumbList, Product, Offer, FAQPage.
**OG/Social:** og:*, Twitter Card, Facebook Debugger.
**LLM/GEO:** llms.txt, llms-full.txt, AI crawler allowed.
**E-commerce:** zoom képgaléria, készletszám, ár+áfa, szállítási idő, visszaküldés link.
**Pre-launch:** 301 redirect map, Search Console verified, sitemap submit.

### Függelék F: Security checklist (40+ pont — kivonat)

**Auth:** strong password, bcrypt, 2FA admin, session expiry, reset token 1h, rate limit login.
**Input:** Zod validation, SQL injection (ORM), XSS (React), CSRF (same-site), path traversal.
**API:** rate limit, CORS strict, CSP strict, no sensitive data in URL.
**Secrets:** env-only, gitleaks CI, rotation 6 hav.
**Network:** HTTPS, HSTS, DB+Redis private, Cloudflare WAF, DDoS.
**Payment:** Barion IP allowlist, no card data stored/logged, PCI-DSS SAQ A.
**GDPR:** adatkezelési tájékoztató, cookie consent, data export, data delete, DPA-k, breach 72h.
**Monitoring:** Sentry, failed login alert, audit log, backup napi, restore-test havi.

### Függelék G: Pre-launch checklist (50+ pont — kivonat)

Üzleti, technical, payment, invoice, email, shipping, search, performance, SEO, security, monitoring, documentation, legal — minden kategóriában tételes ellenőrzés.

### Függelék H: Agentic team `.claude/` fájlok

A `.claude/` mappában: 9 sub-agent (`agents/*.md`), 3 slash command (`commands/*.md`), 2 hook (`hooks/pre-commit`, `hooks/pre-deploy-production`).

Részletek a tényleges fájlokban — `.claude/agents/orchestrator.md`, `code-writer.md`, `challenger.md`, `code-reviewer.md`, `security-auditor.md`, `integration-tester.md`, `seo-checker.md`, `doc-writer.md`, `migration-planner.md`.

---

## CHANGELOG

| Dátum | Verzió | Változtatás |
|---|---|---|
| 2026-05-07 | 1.0 | Első kiadás. |
| 2026-05-07 | 1.0-QA | Háromszoros review átment: 76 task acceptance criteria-val, 82 forrás-URL, magyar piaci kötelezettségek (NAV 3.0, 45/2014, GDPR), 20 ELLENŐRIZD-instrukció. |
| 2026-05-07 | 1.1 | Research-then-code + Cross-agent challenge (Karpathy 0.5 fejezet). 9. sub-agent: challenger. /propose, /challenge, /decide. CLAUDE.md sablon. R11, R12 kockázatok. |
| 2026-05-07 | 1.2 | Egyszerűsítés: 11 → 9 slash command (`/task` umbrella), workflow szöveges, understanding sablon komplexitás-skála, challenger.md tömörítve. |
| 2026-05-07 | 1.3 | **P0 patch — 12 launch-blocker fix.** G-03 (Barion idempotencia háromrétegű), G-04 (sztornó vs helyesbítő — KÖNYVELŐI ELLENŐRIZD, opció A=helyesbítő default), G-06 (NAV hibakód-térkép kategóriánkénti reakció), G-07 (vatRate mező), G-09 (Barion fee költség), G-10 (impersonation TILTOTT, ADR-009), G-11 (file upload mélyebb), G-13 (priceNet/Vat/Gross, banker's rounding, ADR-008), G-16 (kliens fresh inventory check), G-20 (RPO 15p, RTO 4h, ADR-010), CC-1 (ELLENŐRIZD-feloldás eljárás), CC-6 (ADR-0008-013 deadline-okkal). |

---

**A dokumentum vége.**

> **Megjegyzés Norbi-nak:**
> Bármelyik T<x.y> task elindítása előtt javasolt:
> 1. Decision tree-k review-ja — Cloudflare Images vs Bunny.net (T2.4), customer migration (T8.4).
> 2. Fázis 0 jogi review (T0.5) — ügyvéd MIELŐTT élesedünk.
> 3. Karpathy CLAUDE.md skills és autoresearch — magyar nyelvű "Kárpáti programozó" forrás, ha találtál, jelezd.
