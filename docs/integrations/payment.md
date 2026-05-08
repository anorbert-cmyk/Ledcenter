# Fizetési módok — Ledcenter.hu

> **Forrás:** Norbi kézzel írt jegyzete, 2026-05-08.
> **Kapcsolódó task-ok:** T3.3 (checkout payment method selection), T4.1 (Barion modul), T4.2 (Számlázz integráció).
> **Kapcsolódó check:** `docs/checks/T0.5.md` 4. pont.

## Áttekintés

Három fizetési mód a checkout-on:

| ID | Mód | Vásárló díja | Implementáció | Effort |
|---|---|---:|---|---|
| `barion` | Barion (kártya) | 0 Ft | Custom Medusa Payment Provider (T4.1) | L |
| `bank-transfer` | Banki előreutalás | 0 Ft | Manual confirmation flow (admin gomb) | M |
| `cash-on-delivery` | Utánvét (készpénz vagy kártya) | **450 Ft** felár | Manual flow + szállítási integráció | M |

## Részletek

### 1. Barion (kártya)

- **Vásárló költsége:** 0 Ft (a Barion tranzakciós díját Norbi cége fizeti — Barion csomag-választás függő, ⚠️ `docs/checks/T0.2.md` 4.).
- **Flow:**
  1. Vásárló választja "Bankkártya" opciót → Medusa cart `payment_session.provider = 'barion'`.
  2. Storefront redirect a Barion Smart Gateway-re.
  3. Vásárló fizet → Barion callback (idempotens, Redis SETNX + DB unique constraint, lásd ADR-008 és G-03).
  4. Order létrejön → Számlázz.hu sztatikus számla → NAV → Resend email.
- **Implementáció:** `apps/medusa/src/modules/payment-barion/` (T4.1).

### 2. Banki előreutalás

- **Vásárló költsége:** 0 Ft.
- **Flow:**
  1. Vásárló választja "Banki előreutalás" → Medusa cart `payment_session.provider = 'bank-transfer'`.
  2. Order státusz: `payment_pending`. Storefront megjeleníti a banki adatokat (Schuszter-Will Kft. számlaszáma + közlemény: rendelési azonosító).
  3. Vásárló átutalja → Norbi a banki kivonaton ellenőrzi → admin-on "Megérkezett" gomb → order `payment_received`.
  4. Számla, NAV, email mint Barion-nál.
- **Implementáció:** `apps/medusa/src/modules/payment-bank-transfer/` — egyszerű provider (no external API), admin-action `confirmPayment(orderId)`.
- **Megjegyzés:** Norbi ránézésre figyeli a banki kivonatot. Automatikus banki integráció (PSD2, Plaid HU) v1-be NEM, M2-re tervezhető.

### 3. Utánvét (készpénz vagy kártya átvételkor)

- **Vásárló költsége:** **+450 Ft felár** a kosárhoz.
- **Hogyan működik:** A vásárló a futártól / csomagautomatából vételekor fizet (készpénz vagy kártya az adott szolgáltató szerint).
- **Flow:**
  1. Vásárló választja "Utánvét" → Medusa cart `payment_session.provider = 'cash-on-delivery'`.
  2. Cart total += 450 Ft (utánvét felár, külön sor a számlán).
  3. Order státusz: `payment_on_delivery`. Csomag feladva, futár begyűjti a pénzt.
  4. Futárcég napi-heti elszámolásban átutalja Norbi cégének → admin-on "Megérkezett" gomb (mint banki előreutalásnál).
  5. **Számla már a feladáskor készül** (utánvétes), NAV-ba elküldve.
- **Implementáció:** `apps/medusa/src/modules/payment-cash-on-delivery/` — provider + felár-logika (Medusa shipping/discount-szerű additional line item).
- **Megjegyzés:** szállító-függő, mely szállítási módnál érhető el. Tipikusan minden Foxpost/GLS/MPL csomagpontnál és házhoz is. ⚠️ ELLENŐRIZD T4.5-ben szolgáltatónként.

## ÁFA-kezelés

| Tétel | ÁFA |
|---|---|
| Termékek | termék `vatRate` szerint (5/18/27%) — lásd ADR-0008 |
| Szállítási költség | 27% (általános magyar szabály, ⚠️ ELLENŐRIZD könyvelővel) |
| Utánvét felár (450 Ft) | 27% (mint a szállítási költség, ⚠️ ELLENŐRIZD könyvelővel) |

## Számla-megjelenítés

A Számlázz.hu számlán külön sorként jelenik meg:
- Termékek (`Item`) — ÁFA termékenként
- Szállítási díj (`Item`) — 27% ÁFA
- Utánvét felár (`Item`) — 27% ÁFA, csak ha utánvét

## ⚠️ Nyitott kérdések (Norbi feloldja)

1. **Szállítási költség ÁFA-kulcsa:** 27%? — `docs/checks/T0.5.md` új 4. b) pont (ELLENŐRIZD könyvelővel).
2. **Utánvét felár ÁFA-kulcsa:** 27%? — ugyanaz.
3. **Banki előreutalás határideje:** 3 munkanap? 7? Mit teszünk, ha nem érkezik időben? (cancel + email, vagy reminder-folyamat.)
4. **Utánvét limit:** van-e maximum kosárérték, ami fölött nem engedett? (Sok kereskedő 100-500 ezer Ft fölött tiltja.)
5. **Részleges utánvét-elutasítás:** ha a vásárló nem veszi át a csomagot — visszáru flow + sztornó számla (G-04 helyesbítő/sztornó+új).

## Tilos lista (CLAUDE.md kapcsolódó)

- **Kártyaadat tárolás:** TILOS. Barion hosted gateway, mi sose látjuk a kártyaszámot. PCI-DSS SAQ A scope.
- **Hard-coded ÁFA a felárra:** TILOS — `vatRate` mezőből számolj.
- **Customer impersonation** "manual payment confirmation" admin-ban: csak admin saját session, NEM customer-impersonation.
