# Security Incident Response Runbook

> **Kapcsolódó:** build spec T7.10.

## Severity osztályok

- **P0:** outage, adatszivárgás, fizetési flow blokkolt — 30 perc reagálás.
- **P1:** kritikus funkció romlott (search, image, email) — 4 óra reagálás.
- **P2:** kisebb hiba, normál idő.

## Kommunikációs kör

1. **Sentry / Better Stack alert** → Norbi SMS + email.
2. **Norbi** → backup admin (egy barát), ha szabadságon.
3. **GDPR breach (P0):** 72 órán belül NAIH ([naih.hu](https://naih.hu)).

## Forgatókönyvek

### 1. Adatszivárgás gyanúja

1. STOP minden ad-hoc DB lekérdezést.
2. Audit log review (utolsó 24h admin műveletek).
3. **GDPR cikk 33: 72h NAIH bejelentés** — ha valós breach.
4. Érintett vásárlók értesítése (cikk 34).
5. Forensic: mely adatok érintettek?

### 2. Sikeres XSS / SQLi gyanú

1. Sentry alert: gyanús payload (`<script>`, `' OR 1=1`).
2. Cloudflare WAF block aktiválás (manuális rule).
3. Code review: Zod input validation hiányzott valahol?
4. Hot-fix branch + emergency deploy.

### 3. DDoS

1. Cloudflare WAF auto-mitigation.
2. Norbi belép Cloudflare dashboard-ra → Security → Events.
3. Ha Cloudflare nem kezeli: "Under Attack" mode bekapcsolás.
4. Source IP block manuális rule-lal.

### 4. Compromised admin credential

1. Norbi-jelzés: gyanús admin login (Sentry).
2. **Azonnal:** password reset + 2FA reset.
3. Audit log review (mit csinált a támadó?).
4. Sessions revoke (összes admin session).
5. Backup admin értesítve.

### 5. GDPR breach notification (72h)

1. Forensic: mi szivárgott (cikk 33 (3) követelmény szerint dokumentálva).
2. NAIH bejelentés ([naih.hu](https://naih.hu) → adatvédelmi incidens).
3. Érintettek értesítése (ha cikk 34 alapján kell).
4. Public disclosure (ha jelentős ügy).

## Hot-fix protokoll

- **Branch:** `hotfix/<issue>` from `production`.
- **Skip:** code-reviewer ha P0 (later post-mortem).
- **NEM skip:** security-auditor (P0 esetén kötelező quick scan).
- **Deploy:** manual approval gate (Norbi explicit go).
- **Post-fix:** PR `main`-be merge.

## Rollback playbook

1. Cloudflare DNS revert (TTL 60s előrelátás).
2. Railway: previous deployment promotion.
3. DB rollback (PITR) ha schema-change volt.
4. Smoke test.
5. Post-mortem doc: `docs/runbooks/post-mortem-<date>.md`.

## Frissítések

- 2026-05-07: első kiadás.
