# ADR-0010: RPO és RTO célok

**Dátum:** 2026-05-07
**Státusz:** accepted
**Kapcsolódó task:** T1.3, T7.x (DR-drill)
**Deadline:** Fázis 0 vége — Norbi go-ahead
**Kapcsolódó gap:** G-20 (P0)

## Kontextus

A build spec v1.0-ban a backup-stratégia "napi snapshot, 7 nap retention + point-in-time recovery" — DE **RPO** (Recovery Point Objective: mennyi adatvesztés tűrhető) és **RTO** (Recovery Time Objective: mennyi idő alatt kell visszajönni) NEM definiált. Ha jön egy DB-corruption: nincs döntés, hogy 24 órás adatvesztés OK-e Norbi-nak vagy 1 nap downtime OK-e. Incidens során improvizálunk → kaotikus.

Plusz: Railway Postgres egy-régiós. Régió-outage esetén nincs hova fordulni.

## Megfontolt opciók

### Opció A: Nagyvállalati szint — RPO 1 perc, RTO 30 perc

- Multi-region active-active Postgres replikáció
- Streaming backup minden tranzakcióra
- 24/7 on-call csapat

**Költség:** ~$500-1000/hó. Nem reális Norbi solo dev + ~1280 termék webshop méretre.

### Opció B: SMB szint — RPO 15 perc, RTO 4 óra

- Railway Postgres PITR (Point-in-Time Recovery) — 15 perc granularitás Pro plan-en
- Off-site backup: napi `pg_dump` → R2 másik régió, 30 nap retention
- Havi DR-drill: szándékos staging restore-teszt
- Norbi as on-call (egyedüli admin)

**Költség:** ~$5-10/hó (R2 backup) + Railway Pro plan ($20/hó).

### Opció C: Minimum — RPO 24 óra, RTO 24 óra

- Csak napi backup, semmi PITR
- Off-site backup nincs
- DR-drill nincs

**Költség:** ~$0/hó plus Hobby Railway plan.

**Probléma:** 24 óra adatvesztés egy aktív rendelési napban = elveszett rendelések, vásárlói panaszok, NAV-számla-disztribúció-zavar. Nem elfogadható egy webshop-ra.

## Döntés

**Választott opció: B — RPO 15 perc, RTO 4 óra.**

## Indoklás

| Szempont | A (1p/30m) | B (15p/4h) | C (24h/24h) |
|---|---|---|---|
| Adatvesztés worst-case | <1 perc | <15 perc | <24 óra |
| Helyreállás idő | 30 perc | 4 óra | 24 óra |
| Költség / hó | ~$500-1000 | ~$25-30 | ~$0-5 |
| Komplexitás | nagy (csapat kell) | közepes | minimális |
| **Norbi solo dev kompatibilis** | **NEM** | **igen** | igen |
| Webshop UX kompatibilis | ✓ | ✓ — fél munkanap downtime tűrhető | ✗ — 1 nap kiesés vásárlói panasz |

**B opció megfelelő**, mert:
- 15 perc RPO = max 15 perces adatvesztés (Railway PITR ezt biztosítja Pro plan-en)
- 4 óra RTO = fél munkanap, Norbi on-call elérhető és helyreállíthat
- Off-site backup R2-ban védi a régió-outage ellen

## A challenger érveire adott válaszok

**ELLEN-érv 1 — "Mi van, ha Norbi szabadságon van és RTO 4 óra alatt sem oldható meg?":**
**Válasz:** post-launch roadmap (T+60) — 1 backup admin (egy barát) képzése DR-drill-en. Loom-videó a helyreállítási folyamatról. Better Stack alert SMS-en Norbi mobiltelefonjára.

**ELLEN-érv 2 — "Railway egy-régiós, mi van ha az egész régió kiesik (precedens: AWS us-east-1)?":**
**Válasz:** off-site backup R2 másik régióban (Cloudflare globális, multi-region tárolás). DR runbookban dokumentált helyreállítás idegen környezetbe (pl. Vercel + Neon Postgres backup) ~8-24 órás process M2-re — ez a "Plan B" forgatókönyv.

**ELLEN-érv 3 — "Havi DR-drill drága time-investment Norbi-nak":**
**Válasz:** havi 30 perc — staging restore-test, dokumentált eredmény `docs/runbooks/disaster-recovery.md`-be. Olcsóbb, mint egy valódi incidens kapkodás. Plus: az első drill után automatizálható (`tools/db/dr-drill.sh`).

## Következmények

- **Railway Pro plan kötelező** ($20/hó), Hobby plan nem ad PITR-t.
- **Off-site backup:** GitHub Actions cron `pg_dump | gzip | aws s3 cp` Cloudflare R2-re, napi, 30 nap retention. ~$5/hó.
- **DR runbook (`docs/runbooks/disaster-recovery.md`):**
  - Helyreállítási lépések PITR-ből
  - Helyreállítási lépések R2 backup-ból
  - Régió-outage Plan B (idegen környezet)
- **Havi DR-drill:** új T7.x task ("Backup restore-test havonta"), eredmény dokumentálva.
- **Better Stack alert:** Postgres unavailable >2 perc → SMS Norbi-nak.
- **Encryption at rest:** ⚠️ ELLENŐRIZD Railway docs / support (G-35 a `docs/checks/T1.3.md`-ben). GDPR cikk 32 miatt kritikus.
- **Norbi go-live előtt:** szándékos staging-incidens — adatvesztés < 15 perc, helyreállás < 4 óra mérve.

## Frissítések

- 2026-05-07: első kiadás, accepted.
