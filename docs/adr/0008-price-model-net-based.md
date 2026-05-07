# ADR-0008: Ár-tárolás — nettó alapú vs bruttó alapú

**Dátum:** 2026-05-07
**Státusz:** accepted
**Kapcsolódó task:** T2.1, T4.2, T8.2
**Deadline:** Fázis 0 vége — KRITIKUS
**Kapcsolódó gap:** G-13 (P0)

## Kontextus

A magyar NAV Online Számla 3.0 elvárás: a számlán **külön sorként** kell szerepelnie a nettó árnak, az ÁFA-tartalomnak, és a bruttó árnak. A Medusa.js 2.x default ár-modellje egyetlen `amount: number` mezőt kínál — ez nem elég.

Két opció kezelhetetlen:
1. Ha **bruttó-tárolás** + ÁFA visszaszámolás → kerekítés-eltérés 1-2 Ft halmozódhat (NAV-jelentésnél hibás végösszeg).
2. Ha **nettó-tárolás** + UI-on bruttó-számolás → minden kerekítés egyetlen helyen történik (kanonikus).

Plusz: vásárló bruttó árat lát (UI), de a NAV nettóból + ÁFA%-ból dolgozik.

## Megfontolt opciók

### Opció A: Nettó-tárolás (kanonikus)

```typescript
interface ProductVariantPrice {
  priceNet: number       // fillér, kanonikus, tárolt
  vatRate: 5 | 18 | 27   // százalék, T2.1 mező
  // priceGross és priceVat számolt:
  // priceVat = roundHalfEven(priceNet * vatRate / 100, 0)
  // priceGross = priceNet + priceVat
}
```

UI: `priceGross` számolva → `formatPrice(priceGross) + ' Ft'`.
NAV-számla: `priceNet` és `vatRate` közvetlenül a Számlázz.hu API-nak.

### Opció B: Bruttó-tárolás

```typescript
interface ProductVariantPrice {
  priceGross: number     // fillér, kanonikus, tárolt
  vatRate: 5 | 18 | 27
  // priceNet = roundHalfEven(priceGross / (1 + vatRate/100), 0)
  // priceVat = priceGross - priceNet
}
```

UI: közvetlenül `priceGross`.
NAV-számla: `priceNet` visszaszámolva.

### Opció C: Mind a hármat tárolni redundánsan

```typescript
interface ProductVariantPrice {
  priceNet: number
  priceVat: number
  priceGross: number
  vatRate: 5 | 18 | 27
}
```

Invariáns: `priceVat === priceGross - priceNet`. Zod schema `.refine`-jal.

## Döntés

**Választott opció: A — Nettó-tárolás (kanonikus), `priceVat` és `priceGross` számolt.**

Implementáció a `Price` schema-ban (Zod-dal validálva, `packages/shared/src/schemas/price.ts`):

```typescript
export const VatRate = z.union([z.literal(5), z.literal(18), z.literal(27)])

export const Price = z.object({
  priceNet: z.number().int().nonnegative(),    // fillér, kanonikus
  priceVat: z.number().int().nonnegative(),    // számolt
  priceGross: z.number().int().nonnegative(),  // számolt
  vatRate: VatRate,
}).refine(p => p.priceGross === p.priceNet + p.priceVat, {
  message: 'Invariant: priceGross === priceNet + priceVat',
})
```

**Kerekítés:** banker's rounding (`ROUND_HALF_EVEN`), 2 tizedes — fillér.

**Medusa default `amount`:** = `priceGross` (mert a vásárló bruttót lát).

## Indoklás

| Szempont | A (nettó) | B (bruttó) | C (mind a 3) |
|---|---|---|---|
| NAV-konzisztens | ✓ — közvetlenül megy a Számlázz-nak | kerekítés-eltérés visszaszámolásnál | ✓ |
| UI gyors | ✓ + roundHalfEven | ✓ közvetlen | ✓ |
| Storage | 1 mező | 1 mező | 3 mező (3× több) |
| Invariáns-veszély | nincs | nincs | **van** — Zod refine kell |
| Refactor-költség | minimális | közepes (visszaszámoláshoz utility kell) | nagy |

**Opció A nyer**, mert a NAV-nettó-konzisztencia kritikus, és a UI-számolás `roundHalfEven`-nel egyszer megtörténik.

## A challenger érveire adott válaszok

**ELLEN-érv 1 — "Banker's rounding bonyolult, nem mindenki ismeri":**
**Válasz:** unit-teszttel (`roundHalfEven(2.5, 0) === 2`, `roundHalfEven(3.5, 0) === 4`) lefedve, `packages/shared/src/format.ts`-ben dokumentálva. Magyar NAV ezt várja el (`ROUND_HALF_EVEN`), nem opció.

**ELLEN-érv 2 — "Mi van, ha a ShopRenter bruttóban tárolja az árakat?":**
**Válasz:** T8.2 (schema mapping) első task: ShopRenter ár formátumát ellenőrizni (`docs/checks/T8.2.md`-be), és a mapper-ben `priceNet`-re konvertálni. Ha ShopRenter bruttó, akkor 1× kerekítünk a migrációnál, és onnantól nettó a kanonikus.

**ELLEN-érv 3 — "Mi van akkor, ha NAV-szabály változik (pl. új áfa-osztály)?":**
**Válasz:** `vatRate` enum bővíthető (`5 | 18 | 27` → új érték hozzáadva). Migration scripttel létező termékek frissítve. A `priceNet` érintetlen marad.

## Következmények

- T2.1 (termék schema) Zod-dal validál — minden termék-variánsnak `priceNet` + `vatRate`.
- T4.2 (Számlázz mapper) közvetlenül `priceNet` és `vatRate` mezőkből dolgozik — **NEM hard-coded `vat: 27`**.
- T8.2 (ShopRenter migráció) első ellenőrzés: bruttó vagy nettó-e a ShopRenter ár.
- Lint-szabály blokkolja a hard-coded `* 1.27`, `vat: 27`, `vatRate: 27` mintákat (lásd `eslint.config.mjs`).
- Banker's rounding unit-tesztelve (`packages/shared/src/__tests__/format.test.ts`).

## Frissítések

- 2026-05-07: első kiadás, accepted.
