# Paper Craft Workspace — Vision Transformer (ViT)

A reading workspace for completing the **Paper Craft Reading assignment** for
exactly one paper:

> **“An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale”**
> Alexey Dosovitskiy, Lucas Beyer, Alexander Kolesnikov, Dirk Weissenborn,
> Xiaohua Zhai, Thomas Unterthiner, Mostafa Dehghani, Matthias Minderer,
> Georg Heigold, Sylvain Gelly, Jakob Uszkoreit, Neil Houlsby
> (Google Research, Brain Team) — arXiv:2010.11929, ICLR 2021.

This folder lives inside the `tabular-data-agent` project directory but is
self-contained; it touches no project code.

⚠ **The template document's own rule:** the assignment says the template must
capture *your* read of the paper and warns that reviewers can tell when it was
generated rather than worked through. This workspace is built to respect that:
it contains the paper, verified facts, explanations, and **candidate**
observations — never fabricated personal reactions. Your final template answers
must be rewritten in your own words, based on your own reading.

---

## Layout

```
paper-craft/
├── README.md                     ← this file: workflow + status
├── sources.md                    ← source URLs, metadata, honest retrieval log
├── paper/
│   ├── source.md                 ← COMPLETE paper text (verbatim, with provenance header)
│   └── reading_notes.md          ← section-by-section notes (6 prompts per section)
├── template/
│   ├── template_sections.md      ← exact template structure, retrieved from the Google Doc
│   └── working_draft.md          ← working entry for EVERY template section
└── interview/
    ├── questions.md              ← interview prep, 5 groups, model answer + evidence + follow-up each
    └── glossary.md               ← 13+ terms in plain language, tied to this paper
```

## What was completed (status)

| Phase | Status | Notes |
|---|---|---|
| 1. Workspace setup | ✅ | Created; no existing work was overwritten (folder did not exist). |
| 2. Paper retrieval | ✅ **Complete** | Direct PDF fetch failed (`application/pdf` unsupported by tooling); retrieved the same arXiv source via ar5iv HTML. Main text, Eqs. 1–8, references, Appendices A–D, **all 9 tables**, all 14 figure captions are in `paper/source.md`. Figure *images* are not included (captions are); numeric results shown only in plots are available exactly in Appendix Tables 5–6. Declared in `sources.md`. |
| 3. Section-by-section reading | ✅ | `paper/reading_notes.md` — all sections incl. appendices, with paper-vs-interpretation flags and verified numbers. |
| 4. Template retrieval | ✅ **Actual template accessed** | The Google Doc was publicly accessible; every heading/prompt extracted verbatim into `template/template_sections.md`. Nothing invented. |
| 5. Working draft | ✅ | `template/working_draft.md` — every template section has an entry with evidence, explanation, interpretations, uncertainties, and editable draft answers. Subjective sections contain labeled candidates (A)/(B)/(C) for **you** to choose from. |
| 6. Structure analysis | ✅ | In `working_draft.md` §B5 (intro tension, study chain, heavy-lifting sentence, alternative-explanation foreclosure table) plus structural notes throughout the reading notes. |
| 7. Interview prep | ✅ | `interview/questions.md` — Understanding / Technical / Evidence / Critical / Structure, each with model answer, evidence, follow-up. |
| 8. Glossary | ✅ | `interview/glossary.md` — all required terms + 5 adjacent terms. |
| 9. Quality checks | ✅ | See below. |

## Quality checks (performed)

- ✅ Exactly **one** paper used everywhere; no secondary summaries consulted
  (all facts trace to `paper/source.md`).
- ✅ Full paper retrieved and inspected, including appendices and all tables;
  extraction gaps (figure images, pagination) declared in `sources.md` and the
  `source.md` header.
- ✅ All template sections identified from the **actual** document; every
  section has a working entry in the same order.
- ✅ Numbers verified against the paper (Tables 1, 2, 5, 6, 7, 8, 9 quoted
  directly; the Table 2 vs Table 5 ImageNet discrepancy is explained, not
  smoothed over). Citations use section/table/figure numbers since the HTML
  source has no fixed pagination.
- ✅ Paper findings are labeled **[Paper]**; analysis is labeled
  **[Watch for]** / interpretation. Personal-reaction prompts contain candidates
  + a choice prompt, never invented reactions.
- ⚠️ **Open items, deliberately not fabricated:** (1) your genuine reactions
  for all "surprise / criticism / craft" sections; (2) the depth-level checkbox
  (a recommendation with trade-offs is given); (3) venue wording — verify
  "ICLR 2021 (oral)" on the arXiv abstract page if you want to cite it
  precisely.

## How to continue the reading (recommended order)

1. **Read the paper itself, in `paper/source.md`**, top to bottom. Use
   `paper/reading_notes.md` section by section *after* reading each section —
   notes as a check, not a substitute.
2. **Section A of the template first, honestly:** before scrolling past the
   abstract, write your own claim/surprise/test/alternative-explanation
   somewhere scratch. Only then compare with `working_draft.md` §A.
3. Reconcile your scratch answers with the working draft; delete every
   candidate that isn't genuinely yours; rewrite the rest.
4. Work through `working_draft.md` §B and §C the same way — the depth-level
   row and the heavy-lifting-sentence choice are decisions only you can make;
   the file gives you the evidence to defend whichever you pick.
5. Drill with `interview/questions.md` **without** looking at the model answers
   first; then check. Fill the "quick sheet" at the end from memory.
6. Keep `interview/glossary.md` open while doing 3–5; it is the vocabulary
   backbone.

## The one task to do next

**Read Sections 1 and 3 of `paper/source.md` yourself and write, in one
sentence each: (a) the paper's claim, with its condition; (b) what a patch
embedding is, without notation.** Then check both against
`working_draft.md` §A1 and `interview/glossary.md` (“Patch embedding”).
That pair — claim + mechanism — is the interview's spine.

## Filing issues / gaps

If you find anything in these notes that doesn't match `paper/source.md`,
treat the paper as correct and fix the notes — and re-check any interview
answer that depended on it. If you later obtain the published PDF with real
page numbers, you can add page citations, but section/table/figure references
are already precise and preferred.
