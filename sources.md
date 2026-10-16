# Sources

## Primary paper — the ONE paper selected for this assignment

| Field | Value |
|---|---|
| Title | An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale |
| Authors | Alexey Dosovitskiy, Lucas Beyer, Alexander Kolesnikov, Dirk Weissenborn, Xiaohua Zhai, Thomas Unterthiner, Mostafa Dehghani, Matthias Minderer, Georg Heigold, Sylvain Gelly, Jakob Uszkoreit, Neil Houlsby (Google Research, Brain Team) |
| arXiv ID | 2010.11929 |
| Abstract page | https://arxiv.org/abs/2010.11929 |
| PDF | https://arxiv.org/pdf/2010.11929 |
| Full text actually used | https://ar5iv.labs.arxiv.org/html/2010.11929 (ar5iv rendering of the arXiv source) |
| Venue / Year | Published as a conference paper at **ICLR 2021** (oral); arXiv preprint posted October 2020 |
| Common name | Vision Transformer (ViT) |
| Code (linked by the paper, abstract footnote 1) | https://github.com/google-research/vision_transformer |

No other paper is used anywhere in this workspace.

## Retrieval log

- **2026-10-16** — Attempted direct retrieval of `https://arxiv.org/pdf/2010.11929`.
  The fetching tool rejected the response (`Unsupported fetched file content
  type: application/pdf`); the PDF text layer could not be extracted directly.
- **2026-10-16** — Retrieved the complete paper from
  `https://ar5iv.labs.arxiv.org/html/2010.11929`, a LaTeXML conversion of the
  official arXiv LaTeX source. Saved verbatim as `paper/source.md` (with a
  provenance header prepended; the paper text itself is unmodified).

## What is and is not in `paper/source.md`

**Captured in full:**
- Abstract and Sections 1–5 (Introduction, Related Work, Method, Experiments, Conclusion).
- All method equations (Eqs. 1–8) in text form.
- Appendix A (multihead self-attention), Appendix B (training, fine-tuning,
  self-supervision details; Tables 3–4), Appendix C (Tables 5–6), Appendix D
  (analyses D.1–D.10; Tables 7–9), complete reference list, all 14 figure captions.

**Not captured (declared honestly):**
- Figure **images** (the model diagram, plots, attention maps). Their captions
  are present. Numeric results that live only inside plots (e.g. the curves of
  Figures 3, 4, 5) are described in the prose and, where available, given exactly
  in Appendix Tables 5 and 6 — the notes rely on those numbers, not on read-off
  estimates.
- **Page numbers.** The HTML rendering has no fixed pagination, so all citations
  in the notes use section numbers, table numbers, and figure numbers instead.
  (In the published ICLR version, main content spans roughly pp. 1–12 and
  appendices follow; do not cite page numbers you have not verified yourself.)

## Secondary sources consulted

**None.** Every factual claim in this workspace is taken from the paper text in
`paper/source.md`. Statements that are not from the paper (background context,
hypotheses about author intent, observations) are explicitly flagged as
interpretation in each file.

## Assignment template source

| Field | Value |
|---|---|
| Document | "Easy Task - Paper Craft" — assignment instructions + Paper Craft Reading Template |
| URL | https://docs.google.com/document/d/1wxOj4FGdB5SJ1dycmRn9Bm1OSwXlPtwD/edit |
| Retrieval | 2026-10-16, via the document's public plain-text export (no login required; succeeded on first attempt) |
| Saved as | `template/template_sections.md` — every heading and prompt, in template order |

Note from the template document that constrains this whole workspace: *"This
template is meant to capture your own read of the paper, so please don't use an
LLM to fill it in blind."* Accordingly, `template/working_draft.md` contains
evidence, explanations, and **candidate** observations for you to choose from and
rewrite — it deliberately does not fabricate your personal reactions.
