# ADR-0001: Monorepo (pnpm + turborepo) vs multi-repo

**Dátum:** 2026-05-07
**Státusz:** accepted
**Kapcsolódó task:** T1.1
**Deadline:** Fázis 0 vége

## Kontextus

A projekt két fő alkalmazásból áll: a Medusa.js 2.x backend (`apps/backend`) és a Next.js 15 storefront (`apps/storefront`). Plusz közös típusok (`packages/shared-types`), ESLint config, ShopRenter migrációs eszköz (`tools/shoprenter-migrator`). Egy repó vagy több külön?

## Megfontolt opciók

1. **Monorepo (pnpm workspaces + turborepo)** — egy GitHub repo, közös `package.json` workspace, atomic commit a backend+frontend változáshoz.
2. **Multi-repo** — `ledcenter-backend`, `ledcenter-storefront`, `ledcenter-shared-types` külön GitHub repó-k, npm registry vagy git submodule.
3. **Monorepo nx-szel** — gazdagabb pipelines, de bonyolultabb beállítás.

## Döntés

**Választott opció: Monorepo (pnpm workspaces + turborepo).**

## Indoklás

| Szempont | Monorepo (pnpm + turbo) | Multi-repo |
|---|---|---|
| Közös TS típusok sync | egyszerű (`packages/shared-types`) | npm publish + version-update minden change-nél |
| Atomic commit (backend + frontend változás együtt) | ✓ | ✗ — több PR, race condition |
| CI / CD egyszerűség | egy GitHub Actions workflow | többszörösen replikált |
| Cache (turborepo) | ✓ csak változott package builds | minden repo külön |
| Magyar piaci kontextus | **Norbi solo dev**, NEM csapat — egyszerűbb, ha minden egy helyen | több repo = több kontextus-váltás |
| Vendor-lock-in | nincs (pnpm + turbo MIT) | nincs |

A Karpathy #2 (egyszerűség) és #3 (sebészi módosítások) alapelv mind a monorepo mellett szól.

**nx-et elvetettük**, mert turborepo elég 2 alkalmazáshoz, és nx beállítása több időt vesz, mint amit később megnyer.

## A challenger érveire adott válaszok

(Ennek az ADR-nek a meghozása előtt a challenger sub-agent még nem fut le konkrétan ehhez — a build spec 4.1 szakaszában már döntött Norbi monorepo mellett. Ha utólag challenger érveket hoz, ezt a szekciót frissítjük.)

## Következmények

- Repo struktúra: `apps/`, `packages/`, `tools/`, `docs/`, `.github/` — lásd `docs/build-spec.md` 4.2.
- `pnpm install` a gyökérben — minden függőség egy `pnpm-lock.yaml`-ban.
- `turbo run build/test/lint` — incremental, cached.
- CI: egy workflow, ami a változott package-eket buildeli.
- Migráció multi-repó-ra a jövőben drága lenne, de jelenleg semmi sem mond ennek ellent.

## Frissítések

- 2026-05-07: első kiadás, accepted.
