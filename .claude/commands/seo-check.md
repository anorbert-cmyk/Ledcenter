---
description: SEO-checker agent: Lighthouse + schema.org validation + sitemap + robots + llms.txt + broken link check.
---

Lefuttatom a seo-checker agent-et a jelenlegi staging vagy production deployment ellen.

Lépések:
1. Lighthouse CI (Mobile Performance ≥85, SEO=100, Accessibility ≥90).
2. Schema validator (`validator.schema.org` + `search.google.com/test/rich-results`) PDP, PLP, főoldal, kategória mintára.
3. `curl https://<host>/sitemap.xml` — XML validation, URL-szám.
4. `curl https://<host>/robots.txt` — Disallow `/admin`, `/checkout`, `/account`; AI crawlers Allow.
5. `curl https://<host>/llms.txt` — méret, formátum.
6. Broken link check (`linkinator` vagy Screaming Frog).
7. Generálj `docs/seo/audit-<date>.md` riportot.

Findings → ha P0/P1: STOP, fix kötelező a deploy előtt.
