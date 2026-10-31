# JOE submission checklist

## A. Format (IEEE JOE / IEEEtran)
- [x] `\documentclass[journal]{IEEEtran}` with `\markboth` running heads
- [x] Title/author block with `\thanks` affiliation + corresponding e-mail
- [x] Abstract 199 words (window 150–250), zero citations inside
- [x] `\IEEEkeywords` — 6 index terms
- [x] Sections numbered 1–6 + Conclusions; Acknowledgment unnumbered
- [x] AI-use disclosure paragraph inside Acknowledgment (IEEE policy)
- [x] Numeric citations via IEEEtran.bst; all `\cite` resolve
- [x] Equations (1)–(7) numbered `equation` environments, `\eqref` in prose
- [x] Tables: `booktabs`, wide ones as `table*` + `\resizebox{\textwidth}`
- [x] Figures: vector PDF at `\columnwidth` / `0.49\textwidth`

## B. Content integrity
- [x] audit_numbers.py issues=0 (467 literals vs tables/results.json)
- [x] audit_equations.py broken=0 dupes=0
- [x] audit_sentences.py 455 sentences, 83 over 35 (same profile as review)
- [x] check_manuscript_numbers.py 57/57 claims match
- [x] Wilcoxon pairs re-derived from results.json (F2) — text matches output
- [x] WF-Diff fallback labelled; excluded from ranking claims
- [x] CCF described as internal HSV diagnostic (not Panetta's CCF)

## C. References
- [x] 31/31 entries Crossref-verified (DOIs recorded in bib comments)
- [x] ≥4 IEEE JOE journal papers cited (Panetta 2016/2022, Zhang 2022, Zhou 2025)
- [x] Protocol lineage cites McGlamery 1980 (foundational, venue-adjacent)
- [x] bibtex run: 0 warnings; .bbl shipped

## D. Build artefacts
- [x] `manuscript_joe.pdf` 15 pp., 0 errors, 0 undefined refs
- [x] `supplementary_joe.pdf` 2 pp., Figs. S1–S4, 0 missing
- [x] Overfull hboxes ≤ 0.84 pt (sub-visible)
- [x] Class + bst files copied beside the .tex

## E. Portal paperwork
- [x] Cover letter (`cover_letter.md`) — venue fit, contributions, verification
- [x] Manifest (`submission_manifest.md`) — file-by-file upload list
- [x] Internal review gate passed (`08_revision/reports/joe_review_report.md`)
- [x] Archive of the superseded Ocean Engineering build (`latex_oe_archive/`)

## F. Author actions (cannot be automated)
- [ ] Verify ORCID + author order in the portal matches the .tex
- [ ] Enter subject categories: "Ocean sensing / image processing"
- [ ] Confirm page-count policy at submission (15 pp. as compiled)
- [ ] Upload, then check the portal's built PDF (fonts embedded, no cut floats)
