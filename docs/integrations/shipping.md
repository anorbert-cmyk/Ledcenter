# Szállítási módok — Ledcenter.hu

> **Forrás:** Norbi kézzel írt jegyzete, 2026-05-08.
> **Kapcsolódó task-ok:** T3.2 (checkout shipping), T4.5 (FoxPost/MPL/GLS integráció), T5.1 (admin termék-konfiguráció), T2.1 (termék hosszúság attribútum).
> **Kapcsolódó check:** `docs/checks/T0.5.md` 3. pont.

## Áttekintés

Két szint:

1. **Standard szállítás** — minden termékre elérhető (kis méret, csomagba fér).
2. **Speciális hosszú-termék szállítás** — fénycsöves/armatúrás/LED-alu-profil termékekre (60-200 cm hosszúság), CSAK GLS házhozszállítással.

## Standard szállítási módok

| ID | Szolgáltató | Mód | Bruttó ár | Implementáció |
|---|---|---|---:|---|
| `foxpost-locker` | Foxpost | Csomagautomata | **1 490 Ft** | T4.5 — Foxpost API |
| `gls-locker` | GLS | Csomagautomata (CsomagPont) | **1 590 Ft** | T4.5 — GLS API |
| `gls-home` | GLS | Házhozszállítás | **2 590 Ft** | T4.5 — GLS API |
| `mpl-locker` | Magyar Posta | Csomagautomata | **1 190 Ft** | T4.5 — MPL API |
| `mpl-postapont` | Magyar Posta | Postapont | **1 190 Ft** | T4.5 — MPL API |
| `mpl-postan-marad` | Magyar Posta | Postán maradó | **1 190 Ft** | T4.5 — MPL API |
| `mpl-home` | Magyar Posta | Házhozszállítás | **2 290 Ft** | T4.5 — MPL API |

## Speciális — hosszú termékek

**Termékkategóriák:** fénycső, armatúra, LED-alu-profil.
**Hosszúság-kategóriák:** 60 cm, 100 cm, 120 cm, 150 cm, 200 cm.

**Egyedüli elérhető szállítás:** `gls-home` (GLS Házhozszállítás, 2 590 Ft).

### Implementáció — termék-szintű konfiguráció

A termékadatlapon egy **"Fizetés és szállítás" fülön** termékenként állítható, hogy mely szállítási módok elérhetőek. Adatmodell:

```typescript
// packages/shared/src/schemas/product-shipping.ts (T2.1 + T5.1)
import { z } from 'zod'

export const ShippingMethodId = z.enum([
  'foxpost-locker',
  'gls-locker',
  'gls-home',
  'mpl-locker',
  'mpl-postapont',
  'mpl-postan-marad',
  'mpl-home',
])

export const ProductShippingConfig = z.object({
  // Default: minden szallitasi mod elerheto.
  // Hosszu termekekre: csak ['gls-home'].
  allowedMethods: z.array(ShippingMethodId).min(1),
  // Csak hosszu termekeknel kotelezo:
  lengthCm: z.union([z.literal(60), z.literal(100), z.literal(120), z.literal(150), z.literal(200)]).optional(),
})
```

### Checkout flow (T3.2)

1. Vásárló kosárhoz a termékeket adja.
2. Storefront a `cart.items[].product.allowedMethods` metszetét veszi minden termékre.
3. A **metszet** a kosár-szintű `availableShippingMethods` lista.
4. Ha pl. a kosárban van egy 200 cm-es LED-alu-profil + egy E27 izzó:
   - LED-alu-profil: `['gls-home']`
   - E27 izzó: minden 7 mód
   - **Metszet: csak `['gls-home']`** → a vásárló csak GLS házhoz választható.
5. A storefront magyarázza el: "A 200 cm-es LED-alu-profil miatt csak GLS házhozszállítás választható."

## Csomagpont-választó UI

A Foxpost/GLS/MPL **csomagpont-csomagautomata** módoknál a vásárló kiválasztja a konkrét pontot (pl. "Foxpost - Tesco Szarvas").

- **Foxpost:** beépített Foxpost iframe widget VAGY saját implementáció a Foxpost API helymegnevezés-listájával.
- **GLS:** GLS pont API, saját React komponens.
- **MPL:** Postapont API, saját React komponens.

⚠️ ELLENŐRIZD T4.5 előtt: melyik szolgáltatónak van iframe widget vs API-only.

## Tracking és értesítés

Minden szállítási módnál:
- Címke generálás admin trigger-rel (T5.1 widget).
- Tracking URL automatikusan a rendeléshez.
- Resend email vásárlónak: "Csomag feladva, tracking: ..." (T4.6).

## ÁFA

Szállítási költség **27% ÁFA-tartalommal** (általános magyar szabály) — ⚠️ ELLENŐRIZD könyvelővel (`docs/checks/T0.5.md` új pont).

## Sorrend / prioritás (ADR-0012 később)

A storefront UI-on a sorrend (felülről lefelé) — ⚠️ Norbi-jóváhagyás T4.5 előtt:

1. Foxpost Csomagautomata — 1 490 Ft (legolcsóbb, népszerű)
2. Magyar Posta Csomagautomata — 1 190 Ft (még olcsóbb, vidéki)
3. Magyar Posta Postapont — 1 190 Ft
4. Magyar Posta Postán maradó — 1 190 Ft
5. GLS Csomagautomata — 1 590 Ft
6. GLS Házhozszállítás — 2 590 Ft
7. Magyar Posta Házhozszállítás — 2 290 Ft

## ⚠️ Nyitott kérdések (Norbi/code-writer feloldja)

1. **Sorrend a checkout-on:** ár szerint, népszerűség szerint, vagy szolgáltató-csoportosítva? — ADR-0012.
2. **Szállítás ÁFA-kulcsa:** 27%? — `docs/checks/T0.5.md` ELLENŐRIZD könyvelővel.
3. **Csomagpont-widget:** iframe vs saját React? — T4.5 indulása előtt.
4. **Méret/súly limit szállítóanként** — pl. MPL csomagpontba max X kg? Foxpost max Y dimenzió?
5. **Ingyenes szállítás határ:** van? Pl. 25 000 Ft fölött ingyen GLS? Norbi marketing-döntés.
6. **Hosszú termék kategorizáció:** mely SKU-knak `lengthCm` érték? T8.2 ShopRenter migráció során a régi adatból kihúzva, vagy admin manual jelölés?

## Megjegyzés ShopRenter migrációhoz (T8.2)

A jelenlegi ledcenter.hu (ShopRenter) szállítási árai részben különböznek (`/szallitas_6` szerint MPL és FoxPost csomagpont 1090 Ft-tól). **Az új árlista átveszi** a régit — Norbi explicit döntése a magasabb árakról (1190+ Ft).

A T8.2 schema mapping során a "hosszúság" attribútumot ki kell nyerni a ShopRenter termékadat-extend mezőkből, és a Medusa `metadata.lengthCm`-be tenni.
