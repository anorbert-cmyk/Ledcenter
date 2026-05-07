# Disaster Recovery (DR) Runbook

> **Kapcsolódó:** ADR-0010 RPO/RTO, build spec T1.3.

## Célok

- **RPO (Recovery Point Objective): 15 perc** — max 15 perces adatvesztés tűrhető.
- **RTO (Recovery Time Objective): 4 óra** — fél munkanap a helyreállásra.

## Backup stratégia

### 1. Railway Postgres PITR (Point-in-Time Recovery)

- **Plan:** Pro plan ($20/hó) — Hobby plan NEM ad PITR-t.
- **Granularitás:** 15 perc.
- **Retention:** 7 nap (Hobby) / 30 nap (Pro).
- **Helyreállítás:** Railway dashboard → Database → Backups → Restore to point-in-time.

### 2. Off-site backup (Cloudflare R2, másik régió)

- **Cron:** napi 02:00 (GitHub Actions).
- **Retention:** 30 nap.
- **Költség:** ~$0.075-0.15/hó (5-10 GB).
- **Script:** `tools/db/backup-offsite.sh`:
  ```bash
  pg_dump $DATABASE_URL | gzip | aws s3 cp - s3://ledcenter-db-backup/$(date +%Y-%m-%d).sql.gz \
    --endpoint-url=$R2_ENDPOINT
  ```

## Helyreállási scenariók

### Scenario A: DB corruption (utolsó 15 percben)

1. Better Stack alert → Norbi mobiltelefon SMS.
2. Norbi belép Railway dashboard-ra.
3. Database → Backups → Select PITR (15 perccel előbbre).
4. Restore → várj ~5-15 percet.
5. Smoke test: storefront elérhető? Login működik? Order létezik?
6. Sentry alert: monitoring újraindul.

**Várt RTO:** ~30 perc.

### Scenario B: Railway egész régió outage

1. Better Stack alert: storefront unavailable >5 perc.
2. Norbi belép Cloudflare R2-be, leszedi a legújabb `pg_dump`-ot.
3. Új Postgres instance Plan B környezetben (pl. Vercel + Neon, vagy másik Railway régió).
4. `psql $NEW_DATABASE_URL < backup.sql`.
5. Cloudflare DNS átállítás az új backend-re (TTL 60s előrelátás).
6. Smoke test.

**Várt RTO:** ~3-4 óra (Plan B kézi process).

### Scenario C: Adatszivárgás / GDPR breach

1. Sentry / PostHog gyanús lekérdezés alert.
2. Norbi STOP-pol minden ad-hoc DB hozzáférést.
3. Audit log review (az utolsó 24 óra admin-műveletek).
4. **GDPR cikk 33: 72 órán belül NAIH bejelentés.**
5. Érintett vásárlók értesítése (cikk 34, ha "magas kockázat").
6. Forensic analysis: mely adatok szivárogtak?

## Havi DR-drill

**Cél:** RPO/RTO mérése egy szándékos staging-incidenssel.

**Lépések:**
1. Staging Postgres-en `INSERT` egy dummy rekordot.
2. Várj 5-10 percet.
3. PITR restore 5 perccel előbbi állapotra.
4. Verify: a dummy rekord nincs ott.
5. Mérve: adatvesztés <15 perc ✓, helyreállás <4 óra ✓.
6. Eredmény dokumentálva: `docs/runbooks/dr-drill-2026-MM.md`.

## On-call szabály

- **Norbi:** elsődleges on-call.
- **Backup admin (egy barát):** elérhetőség Norbi-szabadság alatt.
- **Better Stack alert:** SMS Norbi + email backup admin.

## Frissítések

- 2026-05-07: első kiadás (ADR-0010 alapján).
- TODO: első DR-drill végrehajtása staging-en.
- TODO: Plan B Postgres provider választás (Neon, Supabase) dokumentálva.
