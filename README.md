# Ledcenter.hu

Magyar LED-világítástechnikai webáruház — modern headless e-commerce stack.

**Status:** Fázis 0 (pre-flight) — ÚSZF-ek, fiókok, jogi alap előkészítése.

## Stack

- **Backend:** Medusa.js 2.x (Node 20, TypeScript)
- **Storefront:** Next.js 15 App Router + Tailwind + shadcn/ui
- **DB:** PostgreSQL 16 (Railway)
- **Cache / queue:** Redis 7 (BullMQ)
- **Payment:** Barion Smart Gateway
- **Számlázás:** Számlázz.hu (`ewngs/szamlazz.js`) → NAV Online Számla 3.0
- **Email:** Resend + React Email
- **Search:** Meilisearch
- **Image:** Cloudflare R2 + Bunny.net Optimizer
- **Hosting:** Railway
- **Monitoring:** Sentry + PostHog + Better Stack
- **DNS / WAF:** Cloudflare

## Kötelező olvasmány — sorrendben

1. **[`CLAUDE.md`](./CLAUDE.md)** — agent-instrukciók (Karpathy 4 alapelv, research-then-code, sub-agent szerepkörök, tilos library-k, magyar specifikumok).
2. **[`docs/build-spec.md`](./docs/build-spec.md)** — 4400+ soros master spec (76 task, 9 fázis, függelékek). **Ez a fő doksi.**
3. **[`docs/build-spec-gaps.md`](./docs/build-spec-gaps.md)** — gap audit (38 hiány, 13 P0 fix v1.3-ban).
4. **[`docs/strategy.md`](./docs/strategy.md)** — eredeti üzleti / stratégiai kontextus.

## Repo struktúra

```
ledcenter/
├── .claude/
│   ├── agents/          # 9 sub-agent definicio
│   ├── commands/        # /task, /security-audit, /seo-check
│   └── hooks/           # pre-commit, pre-deploy-production
├── docs/
│   ├── build-spec.md    # master spec
│   ├── adr/             # Architecture Decision Records
│   ├── understanding/   # T<x.y>.md research-fajlok (Karpathy 0.5.1)
│   ├── challenges/      # challenger sub-agent ELLEN-erv fajlok
│   ├── checks/          # ELLENORIZD-feloldasok (Norbi-feladatok)
│   ├── legal/           # ASZF, adatkezelesi, elallasi, cookie sablonok
│   ├── brand/           # Stitch design-prompt, logo, szinek, fontok
│   ├── runbooks/        # incident response, NAV hibakodok, DR
│   ├── security/        # OWASP audit, pen-test report
│   └── ...
├── apps/                # (T1.1 utan) Medusa backend + Next.js storefront
├── packages/            # (T1.1 utan) shared types, eslint-config
├── tools/               # (T8 utan) shoprenter-migrator, seo-audit
└── .github/workflows/   # CI/CD
```

## Workflow

A 0.5 Kötelező munkamód operatív parancsa: **`/task <T<x.y>>`**.
Ez egy parancsban végigviszi a flow-t: research → ha ADR-worthy: challenger → ADR → kód → review → security audit.

```
TaskList → /task T0.1 → understanding.md (kulon commit)
       → ha ADR-worthy: challenger sub-agent (min. 3 ELLEN-erv)
       → erdemi valasz → ADR (kulon commit)
       → kod (Karpathy loop) → code-reviewer → security-auditor
       → TaskUpdate completed
```

A pre-commit hook (`.claude/hooks/pre-commit`) blokkolja a kód-commitot, ha a research-fájlok hiányoznak.

## Indítás

A jelenlegi task: **`/task T0.1`** — Domain és DNS előkészítés.

Részletek: [`docs/build-spec.md`](./docs/build-spec.md) Fázis 0.

## Üzemeltető

Schuszter-Will Kereskedelmi és Szolgáltató Kft. (Szarvas)
