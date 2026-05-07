# NAV Online Számla 3.0 — Hibakód-térkép és reakció-policy

> **Élő dokumentum.** Frissíteni kell, ahogy a Számlázz.hu / NAV új információt ad.
> **Kapcsolódó:** build spec T4.3, gap audit G-06.

## Reakció kategóriák

A NAV Online Számla 3.0 specifikációja három üzenet-kategóriát ad:
- **INFO** — csak információ
- **WARN** — figyelmeztetés, NEM blokkolja a beküldést
- **ERROR** — blokkolja a beküldést

A v1.3 build spec **NEM rögzít teljes hibakód-listát** (gyorsan elavulna), hanem **kategóriánkénti reakció-policy-t** ad:

| Kategória | NAV-üzenet | Default reakció | Retry-nek értelme? |
|---|---|---|---|
| **INFO** | bármi `INFO` típusú | log only | nem releváns |
| **WARN** | bármi `WARN` típusú | log + admin dashboard widget | nem (a számla bement) |
| **Tartalmi ERROR** — érvénytelen vevő-adat (adószám, név, cím) | pl. 5402 | **NE retry** — admin manual fix → helyesbítő számla | NEM |
| **Tartalmi ERROR** — érvénytelen áfa | pl. 5403 / 591 / 593 / 596 | **NE retry** — termék `vatRate` javítása + helyesbítő | NEM |
| **Tartalmi ERROR** — duplikált bizonylat | pl. 5404 | ignore (mert már bent van) — admin értesítés | NEM (de nem is hiba) |
| **Tartalmi ERROR** — érvénytelen dátum | pl. 330 | NE retry — adat-javítás + helyesbítő | NEM |
| **Tartalmi ERROR** — egyéb 5xxx | sokféle | admin manual review | NEM (default) |
| **Technikai ERROR** — NAV szerver hiba | 5xx HTTP / connection timeout | **retry** exponenciális backoff (1m, 5m, 15m, 30m, 60m) | IGEN |
| **Technikai ERROR** — auth probléma | 403 / authentication | admin alert (Norbi újrahitelesítés NAV ügyfélkapun) | NEM (auth-fix után automatikus retry) |

## 24 órán túli sikertelen beküldés

⚠️ **ELLENŐRIZD Számlázz.hu support-tól írásban** (`docs/checks/T0.3.md` 5. pont):
- **(a)** automatikusan a Számlázz.hu végzi (és nálunk csak admin-monitoring), VAGY
- **(b)** nálunk kell vezetni a kézi NAV ügyfélkapus pótbeküldés folyamatát?

**Norbi-tól várt válasz** Fázis 0 vége előtt.

## Konkrét hibakód-lista

> **TODO**: Ezt a listát Norbi és/vagy a code-writer agent bővíti, ahogy a NAV publikus docs-ot és a Számlázz.hu support email-eket olvassa.
> Forrás: [onlineszamla.nav.gov.hu/dokumentaciok](https://onlineszamla.nav.gov.hu/dokumentaciok), [github.com/nav-gov-hu/Online-Invoice](https://github.com/nav-gov-hu/Online-Invoice).

| Kód | Magyar üzenet | Kategória | Reakció |
|---|---|---|---|
| 5402 | Érvénytelen adószám | tartalmi ERROR | NE retry, admin fix |
| 5403 | Érvénytelen ÁFA | tartalmi ERROR | NE retry, vatRate javít |
| 5404 | Duplikált bizonylat | tartalmi ERROR (ignore) | log only |
| 330 | Érvénytelen dátum | tartalmi ERROR | NE retry, helyesbítő |
| 591 | (egészítsd ki) | tartalmi ERROR | NE retry |
| 593 | (egészítsd ki) | tartalmi ERROR | NE retry |
| 596 | (egészítsd ki) | tartalmi ERROR | NE retry |
| ... | ... | ... | ... |

## Implementáció (T4.3)

```typescript
// apps/backend/src/jobs/nav-status-poll.ts
async function dispatchNavError(invoiceId: string, errorCode: string) {
  const reaction = getNavErrorReaction(errorCode)
  switch (reaction.kind) {
    case 'log-only': /* INFO/WARN */ return
    case 'ignore-duplicate': await markInvoiceAsBent(invoiceId); return
    case 'retry-exponential':
      await scheduleRetry(invoiceId, reaction.delays) // [1, 5, 15, 30, 60] perc
      return
    case 'admin-manual-fix':
      await sentryEvent('NAV tartalmi ERROR', { invoiceId, errorCode })
      await sendAdminEmail(invoiceId, errorCode)
      return
  }
}
```

## Frissítések

- 2026-05-07: első kiadás (T4.3 build spec alapján).
- TODO: konkrét hibakód-lista bővítése a NAV docs-ból.
- TODO: 24h-túli pótbeküldés Számlázz.hu support válaszával.
