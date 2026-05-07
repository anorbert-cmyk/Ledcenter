# Norbi onboarding — Fázis 0 indulása

> **Cél:** Te (Norbi) végigfutsz ezen a fájlon **2 óra alatt**, és minden külső fiók aktív + minden ELLENŐRIZD-pont feloldva. Ezután a `/task T0.1` indítható.

## 1. Olvasd el (~30 perc)

1. **[`README.md`](./README.md)** — projekt áttekintés.
2. **[`CLAUDE.md`](./CLAUDE.md)** — agent munkamód: Karpathy 4 alapelv, research-then-code, 12 pontos tilos lista.
3. **[`docs/checks/README.md`](./docs/checks/README.md)** — **a te feladatlistád**.

## 2. Töltsd ki a checks fájlokat (~60-90 perc)

A `docs/checks/T0.1.md` ... `T0.7.md` fájlokban minden kérdéshez van egy `**Válasz (YYYY-MM-DD):** *[Norbi tölti ki]*` placeholder. Edit-eld:

- `docs/checks/T0.1.md` — domain, Cloudflare, cégadatok, banki számla
- `docs/checks/T0.2.md` — Barion (sandbox + KYC) + ⚠️ csomag választás
- `docs/checks/T0.3.md` — Számlázz.hu + NAV + 2× ⚠️ Számlázz support email + 1× ⚠️ könyvelő
- `docs/checks/T0.4.md` — Railway (Pro plan jóváhagyás)
- `docs/checks/T0.5.md` — termékkör, szállítás, fizetés, garancia, **ügyvéd**
- `docs/checks/T0.6.md` — logó SVG, márkaszínek, fontok, stílus, tone of voice
- `docs/checks/T0.7.md` — ShopRenter API hozzáférés

## 3. Olvasd el a 4 ADR-t (~15 perc)

- `docs/adr/0001-monorepo-vs-multirepo.md` — pnpm + turborepo
- `docs/adr/0008-price-model-net-based.md` — nettó-alapú ár-tárolás (G-13)
- `docs/adr/0009-customer-impersonation-policy.md` — TILTOTT v1 (G-10)
- `docs/adr/0010-rpo-rto-targets.md` — RPO 15p, RTO 4h (G-20)

Ha bármelyiket módosítani szeretnéd: jelezd a chatben, és új ADR-rel vagy ADR-update-tel haladunk.

## 4. Külső fiókok regisztrálása (~30-60 perc közben)

A `docs/checks/T0.1-T0.7.md` minden task-ban felsorolja a szükséges fiókokat. Sorrendben:

1. **Cloudflare** ([dash.cloudflare.com](https://dash.cloudflare.com)) — domain pending
2. **Barion** ([www.barion.com](https://www.barion.com)) — KYC kezdés (1-3 nap)
3. **Számlázz.hu** ([szamlazz.hu](https://szamlazz.hu)) — DÍJBEKÉRŐ csomag minimum
4. **Railway** ([railway.com](https://railway.com)) — Pro plan ($20/hó)
5. **ShopRenter admin** — OAuth2 API hozzáférés ellenőrzés

## 5. Sürgős — írásbeli kommunikáció (1-2 munkanap várhatóan)

Ezeket **most küldd el**, hogy a válasz Fázis 0 vége előtt megérkezzen:

- **Számlázz.hu support email**: 2 kérdés (`docs/checks/T0.3.md` 4. és 5. pont)
- **Könyvelő**: 1 kérdés (`docs/checks/T0.3.md` 6. pont)
- **Ügyvéd**: ÁSZF + adatkezelési tájékoztató review-időpont egyeztetés (`docs/checks/T0.5.md` 6.)
- **Barion ügyfélszolgálat**: csomag-választás kérdés (`docs/checks/T0.2.md` 4.)

## 6. Szólj nekem

Amint a `docs/checks/T0.1.md` minden válasza ki van töltve, **írd ide a chatbe**, hogy "T0.1 ready". Akkor elindítjuk a `/task T0.1`-et:

1. Cloudflare zóna-átvétel előkészítés
2. DNS baseline export
3. ADR-rel zárás (vagy a meglévő struktúra elég)
4. Commit + TaskUpdate completed

Aztán T0.2 (Barion), T0.3 (Számlázz), és így tovább a build spec sorrendje szerint, **egyenkénti jóváhagyással** (te választottad ezt a tempót).

---

## Idő-becslés

| Lépés | Idő |
|---|---|
| Olvasás (README + CLAUDE + checks) | ~30 perc |
| 4 ADR átnézése | ~15 perc |
| Cégadatok összegyűjtése | ~20 perc |
| Cloudflare regisztráció | ~10 perc |
| Barion KYC adatok benyújtása | ~30 perc (KYC válaszra ~1-3 nap) |
| Számlázz.hu fiók nyitás | ~15 perc |
| Railway projekt setup | ~15 perc |
| Email-ek (Számlázz, könyvelő, ügyvéd, Barion) | ~30 perc |
| **Összesen aktív munka** | **~3 óra** |
| **Plus várakozás (KYC, könyvelő válasz)** | **~3-7 nap** |

## Ha elakadsz

- **Technikai kérdés:** `docs/build-spec.md` érintett task szakasza.
- **Üzleti kérdés:** írd ide a chatbe, segítek kontextust adni.
- **Jogi kérdés:** Norbi ügyvédje (ne én).
- **Könyvelői kérdés:** Norbi könyvelője (ne én).
