# Working Draft — Paper Craft Reading Template (ViT)

This is a **study aid**, not your submission. For each template section (exact
structure in `template_sections.md`) you get:

- **The question** — as written in the template.
- **Evidence from the paper** — with section / table / figure references.
- **Plain-language explanation.**
- **Possible interpretations.**
- **Uncertainties to resolve** (things to check or decide while reading).
- **Draft answer(s)** — written so you can edit, replace, or reject them.

⚠ **Personal-reaction sections** (surprise, framing moves, craft lessons, depth
level) contain **candidate observations grounded in the paper**, each labeled
(A)/(B)/(C)… Your job: pick the one that *genuinely* matches your reaction,
rewrite it in your own words, and delete the rest. If none match, write your
own — the candidates exist to jog you, not to answer for you. The template
document itself warns that evaluators can tell when a template was filled
blind, so do not submit any sentence you cannot defend from the paper.

Cross-references: `../paper/reading_notes.md` (section-by-section notes),
`../paper/source.md` (the paper itself).

---

## Paper (header fields)

| Field | Content |
|---|---|
| **Title** | An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale |
| **Authors** | Alexey Dosovitskiy, Lucas Beyer, Alexander Kolesnikov, Dirk Weissenborn, Xiaohua Zhai, Thomas Unterthiner, Mostafa Dehghani, Matthias Minderer, Georg Heigold, Sylvain Gelly, Jakob Uszkoreit, Neil Houlsby (Google Research, Brain Team) |
| **Venue / Year** | ICLR 2021 (published conference paper; oral). arXiv preprint 2010.11929, October 2020. |

*Verify the venue wording yourself on the arXiv abstract page before submitting
(see `../sources.md`).*

---

## A. BEFORE READING PAST THE ABSTRACT

*Discipline check: these four answers should be derivable from the abstract
alone. The candidates below were written after full reading — when you do your
final pass, re-derive them from the abstract and keep only what survives.*

### A1. What's the claim?

**The question.** What does the paper claim?

**Evidence (from the abstract itself).**
- "We show that this reliance on CNNs is not necessary and a pure transformer
  applied directly to sequences of image patches can perform very well on image
  classification tasks."
- "When pre-trained on large amounts of data and transferred to multiple
  mid-sized or small image recognition benchmarks (ImageNet, CIFAR-100, VTAB,
  etc.), Vision Transformer (ViT) attains excellent results compared to
  state-of-the-art convolutional networks while requiring substantially fewer
  computational resources to train."

**Plain language.** You don't need convolutions for vision. Chop the image into
16×16 patches, feed the sequence to an ordinary NLP Transformer, pre-train on a
huge dataset, fine-tune to smaller tasks — and you match or beat the best CNNs
while spending less compute on pre-training.

**Possible interpretations / traps.**
- The claim is *conditional*. Drop "pre-trained on large amounts of data" and
  the claim becomes false by the paper's own experiments (Sec 4.3). A strong
  answer keeps the condition attached.
- Two claims are bundled: (1) *sufficiency* — pure transformers can do vision;
  (2) *efficiency* — at scale, they are cheaper to pre-train than SOTA CNNs.
  They are supported by different experiments (4.2/4.4 vs Table 2's
  core-days column).

**Uncertainties to resolve.** Decide how you will phrase the scope:
"image classification" (safe, correct) vs "computer vision" (too broad — the
paper itself lists detection/segmentation as open, Sec 5).

**Draft answer (edit me).**
> The paper claims that convolutions are not necessary for image recognition:
> a standard Transformer encoder, applied to a sequence of linearly embedded
> image patches, matches or surpasses state-of-the-art CNNs on image
> classification benchmarks — provided it is pre-trained on a large dataset
> (14M–300M images) and then fine-tuned — while requiring substantially less
> pre-training compute.

### A2. What's the surprise?

**The question.** What's the surprise? *(This is your reaction — candidates
below.)*

**Candidate observations (pick what genuinely fits).**
- **(A) The failure is part of the thesis.** Papers usually bury negative
  results; this one announces in the intro that ViT *underperforms* ResNets on
  mid-sized data ("modest accuracies of a few percentage points below"), then
  makes the recovery — "large scale training trumps inductive bias" — the
  headline. The surprise is rhetorical *and* empirical: the weakness of the
  method becomes its most interesting finding (Sec 1; Sec 4.3).
- **(B) How little vision-specific machinery remains.** After a linear patch
  projection, a [class] token, and position embeddings, there is nothing
  image-specific left — and that suffices. Even two things you'd predict to
  matter turn out not to: 2D-aware position embeddings (Appendix D.4, Table 8:
  all methods ≈ 0.64 vs 0.61 with none) and the [class] token itself (D.3:
  average pooling works with a different learning rate).
- **(C) Convolutions stop paying at scale.** The hybrid models help only at
  small compute budgets and the gap "vanishes for larger models" (Sec 4.4) —
  the authors themselves call this "somewhat surprising," since local
  convolutional processing seems like it should always help.
- **(D) The compute margin is large and measured.** 2–4× less pre-training
  compute at equal performance (Sec 4.4), and 2.5k vs 9.9k/12.3k TPUv3-core-days
  vs the SOTA CNN baselines (Table 2) — for an architecture assumed to be
  *more* expensive.

**Uncertainties to resolve.** In your final pass, choose ONE surprise and state
why it surprised *you* (what you believed before reading). The interview will
probe this: "you said X surprised you — why?"

**Draft skeleton (fill with your genuine reaction).**
> Before reading, I expected ___. What actually happened was ___ (cite
> Sec/Tab/Fig). That surprised me because ___.

### A3. What would I do to test this?

**The question.** What would I do to test this?

**Evidence — what the paper actually did (Sec 4), so you can compare your
instincts to theirs.**
1. Compare the largest ViTs against published SOTA CNNs on many benchmarks,
   reporting pre-training compute (Sec 4.2, Table 2).
2. Vary pre-training dataset size (ImageNet → ImageNet-21k → JFT-300M) and
   track where ViT passes CNNs and where bigger models stop hurting (Sec 4.3,
   Figs 3–4, Table 5).
3. Train on random subsets (9M/30M/90M/300M) with *identical* hyper-parameters
   to measure "intrinsic model properties," not regularization effects
   (Sec 4.3, Fig 4).
4. Compute-matched architecture comparison: ResNets, ViTs, hybrids on the same
   data and recipe, plotting accuracy vs pre-training FLOPs (Sec 4.4, Fig 5,
   Table 6).
5. Probe the mechanism: filter PCA, position-embedding similarity, attention
   distance, attention maps (Sec 4.5, Figs 6–7; App D.7–D.8).
6. Try the NLP-style self-supervised objective (masked patch prediction) and
   quantify the remaining gap (Sec 4.6, App B.1.2).

**Possible additional tests you might propose (grounded, not invented results).**
- Fine-tune/test on dense tasks (detection, segmentation) — the authors name
  this as future work (Sec 5), so proposing it is legitimate and shows scope
  awareness.
- More thorough few-shot study — the paper itself says "Further analysis of
  few-shot properties of ViT is an exciting direction of future work"
  (Sec 4.3).
- Equal hyperparameter-search budget for the ResNet baselines (see B2 below —
  this is a real control gap).

**Draft answer (edit me).**
> I would first replicate the paper's core isolation test: same data, same
> recipe, accuracy plotted against compute for CNNs vs ViTs (their Sec 4.4).
> Then I would attack the claim's weak joint: repeat the data-scaling crossover
> (Sec 4.3) on a *public* dataset family so the result doesn't depend on JFT,
> and extend to a task beyond classification (their stated future work, Sec 5).

### A4. Killer alternative explanation

**The question.** What is the killer alternative explanation — the
non-architecture explanation that would deflate the claim if true?

**Candidate alternative explanations (pick the one you find most dangerous and
check what the paper does about it).**
- **(A) "It's the data, not the architecture."** JFT-300M is ~230× larger than
  ImageNet and private. *Paper's response:* the same JFT data trains the
  ResNet baselines in the controlled study (Sec 4.4, Fig 5) — at equal data,
  ViT still wins on compute (2–4×). *Careful with the public-data row:* the
  ImageNet-21k ViT-L/16 does **not** beat BiT-L on any Table 2 benchmark
  (ImageNet 85.30 vs 87.54; CIFAR-100 93.25 vs 93.51; VTAB 72.72 vs 76.29) —
  the paper only claims it "performs well on most datasets too, while taking
  fewer resources to pre-train" (0.23k vs 9.9k TPUv3-core-days). **Resolution:
  the architecture win is established at equal (private) data on
  compute-efficiency; outright SOTA dominance does depend on data scale. The
  obvious-sounding story and the table's actual columns are not identical —
  verify every number before quoting it.**
- **(B) "It's the recipe, tuned in a ViT lab."** Adam + weight decay 0.1 +
  batch 4096 for everyone, but authored around ViT. *Paper's response:*
  Appendix D.1 shows Adam beats SGD for ResNet pre-training in this setup
  (Table 7: avg 89.33 vs 88.79 R50x1; 94.01 vs 93.72 R152x2); fine-tuning runs
  an LR grid for all models and an extra BiT-style setup for ResNets
  (B.1.1). *Residual wiggle:* no published evidence of equal *search budget*.
- **(C) "It's the metric."** Headline wins are fine-tuning accuracy at boosted
  resolution (512/518) with Polyak averaging (Sec 4.1); the scaling study uses
  linear few-shot probing (a different property). *Paper's response:* Table 2
  reports the same protocol across all compared systems; Table 5's caption
  discloses the extra tricks. *Residual wiggle:* crossover location in Fig 4
  is metric-dependent.
- **(D) "Data leakage."** De-duplicated pre-training sets w.r.t. downstream
  test sets (Sec 4.1) is the explicit counter — a control the paper remembered
  to run and state.

**Draft answer (edit me).**
> The most dangerous alternative is that the win comes from JFT-300M rather
> than from the architecture. The paper partly forecloses this by pre-training
> *every* architecture on the same JFT data in the controlled scaling study
> (Sec 4.4): at equal data and recipe, ViT reaches equal accuracy with 2–4×
> less compute. What remains open is hyperparameter-search parity (their D.1
> control softens but does not eliminate this) and the fact that the strongest
> headline numbers rely on private data.

---

## B. AFTER READING THE FULL PAPER

### B1. What did they do that I didn't predict?

**The question.** After reading everything: which moves would you not have
predicted from the abstract? *(Candidates — pick what genuinely surprised you
on the full read.)*

- **(A) 2D-interpolated position embeddings at fine-tuning (Sec 3.2).** The
  mechanism that lets a model trained at 224² run at 384²/512² — keep patch
  size, get a longer sequence, bilinearly interpolate the learned position
  embeddings on the original grid. Elegant, and easy to miss: it is one of the
  "only two points" where 2D structure is injected.
- **(B) The baseline-optimizer control in Appendix D.1.** Using Adam for
  ResNet pre-training is "quite unconventional" — and rather than ignoring the
  suspicion of baseline undertuning, they ran the SGD-vs-Adam experiment and
  published it.
- **(C) The [class] token turns out to be optional (D.3).** The badly
  performing global-average-pooling baseline "performed very poorly" — but
  the deficit vanished with a different learning rate. The paper keeps using
  the token anyway, for NLP-fidelity. A predicted design element with an
  unpredicted *explanation*.
- **(D) Self-supervision target: 3-bit mean color (B.1.2).** Predicting one of
  512 mean colors per patch — a much cruder target than BERT's next-word
  prediction — still gave most of the benefit (+2 of the 6 available points
  vs scratch).
- **(E) ViT's memory efficiency (D.5, Fig 12).** Large ViTs "have a clear
  advantage in terms of memory-efficiency over ResNet models" — relevant to
  training at all, never mentioned in the abstract.

### B2. Controls I missed

**The question.** Which controls (present or absent) did you initially miss?

**Reading aid: controls the paper HAS (easy to overlook on a first read).**
- Same data for all architectures in the scaling study (Sec 4.4) — kills
  "different data" explanations.
- Same recipe across architectures, *plus* the D.1 optimizer justification
  (Table 7) — addresses "different training" explanations.
- De-duplication of pre-training data vs downstream test sets (Sec 4.1) —
  kills leakage.
- The no-extra-regularization subset study (Sec 4.3, experiment 2) — isolates
  "intrinsic model properties" from regularization tuning.
- Position-embedding ablation including a no-position "bag of patches" floor
  (D.4, Table 8: 0.61382 vs ≈0.640) — quantifies how much the component is
  worth.
- Head-type ablation with a learning-rate diagnosis (D.3, Fig 9).
- Wall-clock and memory measurements instead of trusting theoretical FLOPs
  (D.5) — "not always well predicted by theoretical FLOPs."

**Controls still missing (fair criticisms for the interview).**
- Equal hyperparameter-*search* budget for ResNet baselines (grid sizes are
  documented in B.1.1, but the recipe's provenance is ViT-centric).
- A second CNN family trained by the same team (they compare to BiT and
  Noisy Student from literature; EfficientNet is only a literature number in
  Table 2).
- Statistical spread for the scaling study (Table 2 has ±std over 3 runs;
  Table 6 does not).

### B3. Experimental choices that surprised me

**The question.** Which experimental choices stood out? *(Candidates.)*

- **(A) Reporting compute as TPUv3-core-days (Table 2) and exaFLOPs (Table 6).**
  Most architecture papers report only accuracy; here compute is a first-class
  column, which is what makes "cheaper" checkable.
- **(B) Training everyone with Adam, batch 4096, weight decay 0.1 — including
  ResNets (Sec 4.1).** Unconventional for CNNs; defended in D.1.
- **(C) Fine-tuning all 19 VTAB tasks with a single hyper-parameter setting
  (LR 0.01, 2500 steps; B.1.1).** One setting for all tasks sacrifices per-task
  tuning to prevent overfitting the benchmark — a deliberately conservative
  choice.
- **(D) Using simple 1D position embeddings on 2D data by default (Sec 3.1)**
  and testing the fancier alternatives in an appendix — the ablation then
  explains why (the model learns 2D topology itself; Sec 4.5).
- **(E) Keeping the hybrid architecture in the paper at all (Sec 4.4).** They
  built and scaled five hybrid models, found they stop helping — and reported
  the null result rather than quietly dropping it.

### B4. Framing moves I wouldn't have made

**The question.** Framing choices you'd have handled differently. *(Candidates;
each is defensible — the exercise is to articulate your own taste.)*

- **(A) Announcing the small-data failure in the intro (Sec 1).** Alternative
  instinct: lead with strengths. Payoff: it pre-empts the "300M private images"
  attack and sets up the data-scaling thesis. If your taste is "hook first,
  caveats later," say so — and say what you'd lose.
- **(B) The title.** "An Image is Worth 16x16 Words" is memorable branding for
  a technically modest fact (patch size); a conservative reader could call it
  gimmicky relative to the careful prose inside.
- **(C) The very brief self-supervision section (4.6).** One experiment, one
  paragraph of results, gap openly stated; an alternative would foreground it
  as the bridge to the NLP narrative rather than a coda. Notice their own
  conclusion calls the gap "large."
- **(D) burying the strongest efficiency evidence (D.5's wall-clock/memory
  results) in an appendix** while the main text leans on FLOPs/core-days.

### B5. The structural moves

#### B5a. How does the introduction set up tension?

**Evidence (Sec 1).** Paragraph 1: NLP's trajectory — pre-train, fine-tune,
100B parameters, no saturation. Paragraph 2: vision's stagnation — CNNs still
dominant; prior attention attempts either lean on CNN structure or use
specialized patterns that don't scale on accelerators. Paragraph 3: the minimal
idea (patches as tokens). Paragraph 4: the admitted expected failure (lacks
inductive biases → underperforms on mid-sized data). Paragraph 5: the release
— "large scale training trumps inductive bias," with the headline numbers.

**Plain language.** The intro is a four-beat story: *they succeeded over there
→ we're stuck over here → the obvious bridge → the bridge collapses at small
scale → it holds at large scale.* Each beat sets up an experiment: beat 2 →
the SOTA comparison (4.2); beat 4 → the data-requirements study (4.3); beat 5
→ the scaling study (4.4).

**Interpretation [not the authors' explicit statement].** The tension is doing
argumentative work: by the time results appear, the reader has already accepted
that data scale is the moderating variable, so Sec 4.3's crossover reads as
confirmation rather than as a concession.

**What would be lost without it.** Without the announced failure, Sec 4.3's
negative results would look like damage control; with it, they look like
findings. The intro converts the paper's biggest vulnerability into its
thesis.

#### B5b. How do the studies build on each other?

**The chain (my analysis, each link sourced).**
1. **4.2 (Can it beat SOTA?)** → Yes, at the top end, with less compute
   (Table 2). Immediately raises: *is that just the private JFT data?*
2. **4.3 (How much data does it need?)** → Quantifies the dependence: ViT-L
   *loses* to ViT-B on ImageNet-scale pre-training; overtakes only at JFT;
   subset study shows the ViT-vs-ResNet crossover (Figs 3–4, Table 5).
   Raises: *with data held constant, is the architecture itself better?*
3. **4.4 (Architecture at fixed data, fixed recipe, vs compute)** → ViT
   dominates the compute axis 2–4×; hybrids stop helping (Fig 5, Table 6).
   Raises: *does it actually work like a vision model internally?*
4. **4.5 (Mechanism)** → Filters, position embeddings, attention distance
   behave sensibly; explains the D.4 null result (Figs 6–7).
5. **4.6 (Future)** → The NLP-parallel self-supervised objective helps +2%
   but lags 4% — a quantitative pointer to the next problem.

**What would be lost without the chain.** Table 2 alone supports only "big
private data + new arch = good numbers." The force of the paper comes from 4.3
and 4.4 *isolating* the variables — data and recipe — one at a time. Remove
4.4 and the "it's the recipe/data" alternative explanations survive; remove
4.3 and the headline "large scale trumps inductive bias" is unearned.

#### B5c. The one heavy lifting sentence

**The question.** Which single sentence carries the paper's weight?
*(Candidates — choose and defend yours.)*

- **(A) "We find that large scale training trumps inductive bias." (Sec 1,
  para 5).** *Case:* it is the sentence that turns a negative result into a
  law-like claim; Sec 4.3 is its evidence; the entire framing (and the paper's
  title-level message) hangs on it.
- **(B) "We show that this reliance on CNNs is not necessary and a pure
  transformer applied directly to sequences of image patches can perform very
  well on image classification tasks." (Abstract).** *Case:* the thesis
  sentence — everything else is support.
- **(C) "Vision Transformers overfit more than ResNets with comparable
  computational cost on smaller datasets." (Sec 4.3).** *Case:* the most
  *informative* sentence — the honest, falsifiable, mechanism-adjacent finding
  that a skeptic would keep from the paper.

**Recommendation:** (A) is the best default answer for the interview because it
identifies the load-bearing *claim* rather than the summary; be ready to justify
against (B).

#### B5d. How do they foreclose alternative explanations?

**The map (source each entry).**

| Alternative explanation | Foreclosed by | Status |
|---|---|---|
| Data leakage into benchmarks | De-duplication vs downstream test sets (Sec 4.1) | Closed |
| It's just more data, not architecture | Same-JFT controlled study: ViT 2–4× more compute-efficient (Sec 4.4, Fig 5) | Closed at large scale |
| Baselines trained worse | Shared recipe; D.1 Adam>SGD control (Table 7); same LR grids + BiT setup for ResNets (B.1.1) | Partially closed — search-budget parity unpublished |
| Position embeddings / class token are secretly doing the vision work | D.4 ablation (Table 8); D.3 head-type study | Closed |
| Speed claims are FLOP-theory only | D.5 wall-clock + memory measurements (Fig 12) | Closed on TPUv3 |
| Gains confined to one benchmark | 6 transfer datasets + VTAB-19 breakdown (Table 2, Table 9) + ObjectNet (D.9) | Mostly closed |

**What is *not* foreclosed (say this in the interview — it shows control).**
Private data as the headline driver; hyperparameter-search parity; other
hardware platforms; tasks beyond classification; robustness/distribution shift.

### B6. Depth level of the paper — tick ONE row

| Level | Description | Case |
|---|---|---|
| 0 | Proposes X, it works | Clearly too low — the paper is far more than a proposal + demo. |
| 1 | X works + strong evidence | **Fits unambiguously:** SOTA comparison + controlled compute study + data-scaling study. |
| 2 | Why X works + reveals problem structure | **Defensible:** Sec 4.5 mechanism probes, D.2 scaling laws ("compute better predictor than parameters"), and the inductive-bias/data crossover reveal a *structure* — inductive bias as a data-efficiency trade — not visible before. |
| 3 | General principle that reframes broader thinking | Tempting via "scale trumps inductive bias," but the paper never argues this beyond image classification; claiming 3 over-reads the evidence. |

**Recommendation (your call):** tick **2**, and be ready to defend it with the
crossover experiment + the mechanism analyses — or tick 1 and argue that the
"Why" evidence is qualitative (4.5 is observational, not causal). Either is
interviewable; 3 is the one that gets you challenged.

---

## C. WHAT I LEARNED ABOUT CRAFT

*All three prompts below ask for YOUR takeaway. Candidates are grounded in
paper mechanics; pick, rewrite, own.*

### C1. The framing lesson

**Candidates.**
- **(A) Turn your method's weakness into the dependent variable.** The paper's
  most quotable result (inductive bias vs data scale) exists because they
  *designed* the experiment their weakness demanded (Sec 4.3) instead of
  hiding the weakness.
- **(B) State the conditionality of your claim precisely, and let every
  experiment honor it.** The abstract's conditional sentence ("when pre-trained
  on large amounts of data...") is matched by an entire section measuring that
  condition — claim and evidence have the same shape.
- **(C) Foreclose the top alternative explanation with a dedicated experiment,
  not a paragraph of prose.** Sec 4.4 exists solely to kill "it's the data,"
  and it is the paper's most-cited defense.

### C2. Fruitfulness

**Candidates.**
- **(A) A good open-problem list is evidence.** The conclusion's three open
  directions (dense tasks, self-supervision gap, more scaling) are each
  *derived from a result in the paper* (Sec 5; the 4-point self-supervision gap
  is quantified in 4.6), not generic wish-list items.
- **(B) The paper's framing immediately generates the next questions it then
  answers itself** (does it scale? Sec 4.4; does it work inside? Sec 4.5) —
  a model of a self-contained research narrative.
- **(C) Publish the negative hybrid result.** Reporting that hybrids stop
  helping (Sec 4.4) made the simple story *more* credible and gave later
  readers a clean decision rule.

### C3. One thing I'll steal

**Candidates.**
- **(A) Report compute as a headline column** (TPUv3-core-days in Table 2;
  exaFLOPs in Table 6) — turns "efficient" from an adjective into a checkable
  number.
- **(B) The "most related work" paragraph recipe (Sec 2).** Name the closest
  prior work, state in two sentences exactly what it did, then exactly what
  you add — Cordonnier et al. 2020 gets two sentences and the novelty claim is
  sharper for it.
- **(C) The ablation appendix as a credibility engine (App D).** Every
  unconventional choice (Adam for ResNets, 1D positions, class token, patch
  size, FLOPs vs wall-clock) has a corresponding measurement waiting in D.1–D.7
  — so any interviewer objection has a table number to answer with.

---

## Fill-in checklist (before you submit)

- [ ] Every bracketed candidate in this file is either rewritten in your words or deleted.
- [ ] Every number you quote appears in `paper/source.md` (check Table 2, Table 5, Table 6, Table 8, Table 9 — the commonly misquoted ones).
- [ ] Section A answers are re-derived from the abstract alone.
- [ ] Depth-level row ticked and defensible out loud.
- [ ] Your "surprise" answer names what you believed *before* reading.
- [ ] Nothing in the final submission is a sentence you cannot defend from the paper.
