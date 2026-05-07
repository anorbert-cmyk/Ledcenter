---
description: Lefuttatja a security-auditor agent-et az utolsó N committra.
argument-hint: [N]
---

Lefuttatom a security-auditor agent-et az utolsó $1 (default: 5) committra.

Lépések:
1. `git log -n $1 --oneline` — committok listája.
2. `git diff HEAD~$1 HEAD` — változások.
3. Hívd be a `security-auditor` sub-agentet a Task tool-on keresztül.
4. Findings → ha bármi P0/P1: jelezd Norbi-nak.
5. Ha minden zöld: report a `docs/security/audit-<date>.md`-be.

A security-auditor a build spec Függelék F (40+ pont) checklist-jét + a projekt-specifikus szabályokat (hard-coded ÁFA, customer impersonation, tilos library-k, currency assertion) ellenőrzi.
