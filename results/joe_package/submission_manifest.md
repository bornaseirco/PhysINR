# JOE submission package — manifest

Target journal: **IEEE Journal of Oceanic Engineering** (regular paper)
Manuscript: PhysINR-Zero / HydroBench — build `manuscript_joe.pdf` (15 pp.)
Package assembled: 2026-10, source tree `paper/`.

## Files to upload (ScholarOne/IEEE Author Portal)

| # | Item | File | Notes |
|---|---|---|---|
| 1 | Main manuscript (LaTeX) | `latex/manuscript_joe.tex` | IEEEtran journal mode |
| 2 | Bibliography | `latex/references.bib` | 31 Crossref-verified entries |
| 3 | Compiled .bbl | `latex/manuscript_joe.bbl` | include for safety |
| 4 | Class file | `latex/IEEEtran.cls` | copied from TeX Live |
| 5 | Bib style | `latex/IEEEtran.bst` | copied from TeX Live |
| 6 | Figures (vector PDF) | `latex/figures/*.pdf` | 5 manuscript + 4 supplementary |
| 7 | Manuscript PDF (proof) | `latex/manuscript_joe.pdf` | 15 pp., 0 errors |
| 8 | Supplementary PDF | `latex/supplementary_joe.pdf` | 2 pp., Figs. S1–S4 |
| 9 | Supplementary LaTeX | `latex/supplementary_joe.tex` | standalone article class |
| 10 | Cover letter | `cover_letter.md` | convert to PDF/portal text |
| 11 | Suggested keywords | in manuscript (`\IEEEkeywords`) | 6 terms |
| 12 | AI disclosure | in manuscript Acknowledgment | also `../ai_disclosure.md` |

## Checks passed before packaging (see `../08_revision/reports/joe_review_report.md`)

- pdflatex/bibtex: 0 errors, 0 warnings, 0 undefined refs, 0 duplicate labels
- audit_numbers.py: **issues=0** (467 prose literals, 8 tables)
- audit_equations.py: broken=0, dupes=0, eqrefs=0 (7 numbered equations)
- audit_sentences.py: 455 sentences; 83 over 35 words; longest 81 (reference
  entries dominate the tail); median 20
- Abstract 199 words (JOE window 150–250), no citations in abstract
- 4 IEEE JOE journal references + McGlamery (1980) lineage framing
- Wilcoxon/power/bootstrap scripts re-run (F2): all claims reproduce

## Source-of-truth chain

`paper/02_manuscript/*.md` → `experiments/build_latex_joe.py` →
`paper/06_submission/latex/manuscript_joe.{tex,pdf}`.
Any text change must be made in the Markdown sources and rebuilt — the .tex
is generated. Audit scripts take the .tex path as an argument.
