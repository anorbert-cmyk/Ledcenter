---
description: A 0.5 Kötelező munkamód végpont-parancsa. Egy parancsban végigviszi: research (understanding.md) → ha ADR-worthy: challenger sub-agent → orchestrator-nek érdemi válasz → ADR → kódolás (Karpathy loop) → code-reviewer → security-auditor.
argument-hint: <T<x.y>>
---

Végrehajtom a `$1` taskot a build spec 0.5 fejezet szabályai szerint. **Az ADR-worthy elágazásnál megállok és visszaadom az orchestrator-nek/Norbi-nak.**

## 1. fázis — Research (mindig)

1. Olvasd a build spec `$1` szakaszát teljesen (Description, Inputok, Outputok, Acceptance criteria, Library, Security checklist, Test, Decision tree, Effort, Dependencies).
2. Olvasd az érintett kódbázist (Read + Grep). Ha külső API: WebFetch a docs-ra.
3. Generálj `docs/understanding/$1.md`-t magyarul a 0.5.1 sablon szerint (S: 3-5 sor, M: 8-15, L/XL: 15-30). KÖTELEZŐ jelölni: **ADR-worthy?** igen/nem.
4. Commit külön: `$1: understanding`. NEM érintesz kódfájlt.

## 2. fázis — Challenge (csak ha ADR-worthy)

5. Ha az understanding.md jelez ADR-worthy döntést, spawnolj challenger sub-agentet:
   ```
   Agent({
     description: "Challenge $1 decision",
     subagent_type: "challenger",
     prompt: "Olvasd a `docs/understanding/$1.md`-t és a build spec $1 szakaszát. Generálj `docs/challenges/$1.md`-t a challenger.md szabályai szerint. Min. 3 ELLEN-érv, mindegyik konkrét kockázattal/alternatívával/long-term concern-nel. NE rubber-stamp."
   })
   ```
6. A challenger visszaad `docs/challenges/$1.md`-t. Ellenőrizd: min. 3 ELLEN-érv konkrét kifejtéssel, min. 1 alternatíva, min. 1 long-term concern, min. 3 konkrét kérdés. Ha bármi nem ✓: ÉRVÉNYTELEN, spawnold újra friss sub-agentet.
7. **STOP** — adj vissza az orchestrator-nek/Norbi-nak: a challenge fájl tartalmával + a kérdéssel: „Írj érdemi választ minden érvre, vagy módosítsd a javaslatot." A `/task` ezen a ponton megszakad.

## 3. fázis — ADR (csak ha challenge-en átment és van érdemi válasz)

8. Az érdemi válaszok rögzítése után számold ki a következő ADR-számot.
9. Generálj `docs/adr/NNNN-<slug>.md`-t a doc-writer.md sablonja szerint, beleértve egy „A challenger érveire adott válaszok" szekciót, ahol minden érvre pontról-pontra reagálsz.
10. Commit: `docs: ADR-NNNN <slug>`.

## 4. fázis — Kód (mindig)

11. Karpathy loop a code-writer.md szerint: PROPOSE → RUN → MEASURE → ITERATE max 5×.
12. Code-reviewer agent → diff review.
13. Security-auditor (ha van checklist) + integration-tester (Fázis 4, 8 kritikus út).
14. Commit: `$1: <cím>`. TaskUpdate completed.

## Megszakítási pontok (mikor áll meg a `/task`?)

- **Challenge után** (5-7. lépés között): orchestrator-nek/Norbi-nak kell érdemi választ írnia.
- **ITERATE max 5× elérve és AC mégsem zöld**: ESCALATE — ADR + Norbi-jóváhagyás kérése.
- **security-auditor P0/P1 finding**: STOP, fix kötelező.
- **ELLENŐRIZD-pont nem feloldott**: STOP, kérdés Norbi-nak.

A `/propose`, `/challenge`, `/decide` belső lépések — manuálisan is hívhatóak, ha valaki finomszemcsésen akar haladni; alapértelmezésben a `/task` umbrella futtatja őket.
