# Interview Preparation — ViT (Dosovitskiy et al., ICLR 2021)

Five question groups, as required by the assignment. Each question has:

1. **Model explanation** — a concise answer you should re-express in your own words.
2. **Evidence** — where it comes from in the paper (`paper/source.md` numbering).
3. **Likely follow-up** — what the interviewer asks next.

Goal: understand well enough to *paraphrase*, not memorize. If you cannot
explain an answer without looking at this file, re-read the corresponding part
of `paper/reading_notes.md` first.

---

## 1. Understanding

### Q1.1 What is the paper's central claim?

**Model explanation.** Convolutions are not necessary for image recognition. A
standard NLP Transformer encoder, applied to a sequence of linearly embedded
image patches, matches or beats state-of-the-art CNNs on image classification
— *when* pre-trained on a large dataset (14M–300M images) and fine-tuned to the
target task — while requiring substantially less pre-training compute.

**Evidence.** Abstract (both halves: sufficiency + efficiency); Sec 1 para 5
(88.55 ImageNet / 90.72 ReaL / 94.55 CIFAR-100 / 77.63 VTAB); Table 2
(accuracy + TPUv3-core-days columns).

**Likely follow-up.** *"Is that claim unconditional?"* No — Sec 4.3 shows ViT
underperforms ResNets when pre-trained on ImageNet-scale data; the condition
"large pre-training data" is part of the claim, not a caveat.

### Q1.2 What problem is it trying to solve?

**Model explanation.** NLP converged on one architecture (Transformer) that
scales predictably; vision had many bespoke attention mechanisms that either
kept CNN scaffolding or used specialized attention patterns that don't run
efficiently on accelerators (Sec 2). The paper asks whether vision needs any
architecture-specific machinery at all, or whether the *generic* NLP recipe
("attention is all you need," pre-train then fine-tune) suffices given scale.

**Evidence.** Sec 1 paras 1–2; Sec 2 (families of prior attention-for-vision
work and their engineering costs; D.6 timing evidence for AxialResNet being
"extremely slow on TPUs").

**Likely follow-up.** *"Why was 2020 the right moment?"* Because NLP had just
demonstrated pre-train/fine-tune scaling (BERT, GPT-3, >100B params, Sec 1)
and vision had large pre-training datasets ready (ImageNet-21k, JFT-300M, Sec
4.1) — the ingredients existed; the paper tests whether the recipe transfers.

### Q1.3 What is genuinely new about the method?

**Model explanation.** Three things, in increasing order of novelty:
(1) patches-as-tokens is *not* new (Cordonnier et al. 2020 used 2×2 patches
with full attention — Sec 2 names them as "most related to ours"); (2) using
practical 16×16 patches so medium-resolution images work, with an
efficient standard implementation; (3) the *demonstration* that this vanilla
architecture, pre-trained at scale, is competitive with or better than SOTA
CNNs — plus the systematic characterization of *how much data* is needed
(Sec 4.3) and the compute-advantage measurement (Sec 4.4).

**Evidence.** Sec 2 (explicit delta vs Cordonnier et al. and iGPT); Sec 3.1
(minimal-modification design statement); Secs 4.3–4.4.

**Likely follow-up.** *"So the idea isn't the architecture, it's the scaling
study?"* Largely yes — and saying so precisely is a strength, not a weakness:
the architecture is deliberately boring ("follow the original Transformer as
closely as possible," Sec 3), which is what makes the scaling conclusions
clean.

---

## 2. Technical explanation

### Q2.1 How does the architecture work?

**Model explanation.** Five steps (Sec 3.1, Figure 1, Eqs. 1–4):
1. Split the H×W image into N = HW/P² non-overlapping P×P patches (standard:
   224² image, P=16 → 196 patches — the "16×16 words").
2. Flatten each patch and map it to width D with a trainable linear projection
   (the "patch embedding," Eq. 1).
3. Prepend a learnable [class] token (from BERT) and add learnable 1D position
   embeddings so patch order is known (Eq. 1).
4. Run L standard Transformer encoder blocks: pre-LayerNorm, multi-head
   self-attention (global across all patches) + MLP, residuals (Eqs. 2–3;
   attention math in App A, Eqs. 5–8).
5. Take the [class] token's final state (after final LayerNorm, Eq. 4) as the
   image representation; MLP head during pre-training, single linear layer
   during fine-tuning.

**Evidence.** Sec 3.1; Figure 1 caption; Table 1 (ViT-B: 12 layers, D=768,
86M; ViT-L: 24, 1024, 307M; ViT-H: 32, 1280, 632M).

**Likely follow-up.** *"Where is the 2D structure in all this?"* Almost
nowhere — only in patch extraction and, at fine-tuning, in 2D-interpolating
position embeddings for new resolutions (Sec 3.2). This near-absence of
inductive bias is the point.

### Q2.2 What are the main components, and which are load-bearing?

**Model explanation.** Patch projection, position embeddings, [class] token,
encoder blocks, classification head. The ablations say which matter:
position embeddings matter a lot *as a category* (Table 8: 0.614 with none vs
~0.640 with any scheme) but *which scheme* barely matters (1D ≈ 2D ≈ relative)
— because the model learns 2D topology itself (Sec 4.5, Fig 7 center). The
[class] token is not load-bearing either: global average pooling matches it
once the learning rate is tuned (D.3, Fig 9). Patch size is the compute knob:
sequence length scales with 1/P² (Sec 4.1), and smaller patches at equal
compute are one of the most robust wins in the shape ablation (D.2, Fig 8).

**Evidence.** App D.2–D.4; Table 8; Fig 8; Fig 9; Sec 4.5.

**Likely follow-up.** *"If the class token doesn't matter, why keep it?"*
Fidelity to the original Transformer/BERT design — the paper's stated strategy
is minimal deviation from NLP (Sec 3), and it simplifies comparisons.

### Q2.3 What are the important training and evaluation choices?

**Model explanation.**
- *Pre-training:* Adam (β1 0.9, β2 0.999), batch 4096, weight decay 0.1, linear
  warmup (10k steps) + decay; 7 epochs on JFT for most models, 14 for the
  largest; 90 on ImageNet-21k; 300 on ImageNet with strong regularization
  (Table 3).
- *Fine-tuning:* SGD + momentum 0.9, batch 512, no weight decay, cosine decay,
  LR grid search per dataset (Table 4); default resolution 384; Table-2 ImageNet
  numbers additionally use 512/518 resolution + Polyak averaging (Sec 4.1).
- *Transfer protocol:* discard the whole pre-trained head; attach a
  zero-initialized linear layer (B.1.1).
- *Metrics:* fine-tuning accuracy, plus few-shot *linear* evaluation (regularized
  least squares on frozen features) for cheap large-scale probing (Sec 4.1).
- *Data hygiene:* pre-training sets de-duplicated against downstream test sets
  (Sec 4.1).

**Evidence.** Sec 4.1; Tables 3–4; B.1.1.

**Likely follow-up.** *"Why does fine-tuning use SGD when pre-training uses
Adam?"* The paper doesn't theorize; it documents. SGD+momentum with a small LR
grid is the transfer recipe inherited from the BiT line of work (B.1.1 follows
Kolesnikov et al. 2020's protocol); note D.1 shows the Adam-vs-SGD question
they *did* test was for ResNet *pre-training*, where Adam won.

---

## 3. Evidence and findings

### Q3.1 Which experiments provide the strongest evidence?

**Model explanation.** In order:
1. **The compute-controlled scaling study (Sec 4.4, Fig 5, Table 6)** — same
   data, same recipe, accuracy vs pre-training compute for 18 models. This is
   the paper's cleanest causal comparison: ViT needs 2–4× less compute at equal
   performance (5-dataset average).
2. **The pre-training data-requirements study (Sec 4.3, Figs 3–4, Table 5)** —
   three dataset scales plus random JFT subsets at fixed hyper-parameters;
   establishes the crossover: ResNets win small-data, ViT wins large-data, and
   bigger ViT models only pay off with enough data (ViT-L < ViT-B on
   ImageNet-scale pre-training).
3. **The SOTA comparison (Sec 4.2, Table 2)** — headline credibility: ViT-L/16
   (JFT) matches or beats BiT-L (same data) across every listed benchmark at
   ~14× less pre-training compute (0.68k vs 9.9k TPUv3-core-days). Note: the
   paper's prose says "outperforms BiT-L... on all tasks," but the table shows
   two non-wins — ReaL is an exact tie (90.54 vs 90.54) and VTAB is 76.28 vs
   76.29, i.e. statistical ties given the ± values. Quote the table, not the
   prose, if challenged.

**Evidence.** As cited; note Table 2 reports mean±std over 3 fine-tuning runs.

**Likely follow-up.** *"Why is the SOTA table not the strongest evidence?"*
Because it compares systems that differ in many factors at once (architecture,
recipe, epochs); its job is headline competitiveness, while 4.3/4.4 do the
isolating.

### Q3.2 What results are surprising?

**Model explanation.** Pick two and justify:
- Hybrids (CNN front-end + ViT) help only at small compute and the gap
  "vanishes for larger models" — the authors themselves call this "somewhat
  surprising" (Sec 4.4), because convolutional locality seems like it should
  always help.
- The model learns what it needs: position embeddings spontaneously encode 2D
  distance and row/column structure (Fig 7 center), explaining why hand-crafted
  2D position schemes add nothing (Table 8) — the "dumb" default is fine.
- Also defensible: crude self-supervision (predict 3-bit mean color) recovers
  most of the from-scratch-to-supervised gap's first half (+2%, still −4%;
  Sec 4.6); or the memory efficiency of large ViTs (D.5).

**Evidence.** Sec 4.4; Sec 4.5 + App D.4; Sec 4.6 + B.1.2; App D.5.

**Likely follow-up.** *"Which of these did YOU find most surprising, and what
did you believe before?"* — have a first-person answer ready; this is the
template's "surprise" section coming back. Use your genuine reaction from
`template/working_draft.md` A2.

### Q3.3 What alternative explanations remain possible?

**Model explanation.** The paper closes the big ones: data leakage
(de-duplication, Sec 4.1), data-scale confound (same-JFT controlled study,
4.4), optimizer unfairness (D.1: Adam > SGD for ResNet pre-training in this
setup, Table 7). What remains genuinely open:
- **Hyperparameter-search parity** — the recipe was authored around ViT; equal
  *search budgets* for baselines are not documented.
- **Private data dependence** — the strongest headline numbers use JFT-300M;
  the public ImageNet-21k ViT-L/16 is competitive but does *not* dominate
  BiT-L (Table 2: 85.30 vs 87.54 ImageNet; VTAB 72.72 vs 76.29).
- **Metric dependence** — the small-data crossover is measured with linear
  few-shot probing (Fig 4); fine-tuning curves could place the crossover
  elsewhere.
- **Single hardware platform** — wall-clock claims are TPUv3-specific (D.5).

**Evidence.** Sec 4.1; Sec 4.3; Sec 4.4; Table 2; App D.1, D.5.

**Likely follow-up.** *"Is 'it's the data' dead, then?"* At large scale, with
data held constant, yes — that's the whole point of Sec 4.4. But the *public*
data result shows scale matters for dominance, so the honest statement is
"architecture wins at equal data on compute efficiency; data scale determines
whether you beat SOTA outright."

---

## 4. Critical reading

### Q4.1 What are the paper's limitations?

**Model explanation.** The paper's own list plus grounded observations:
1. Supervised classification only — detection/segmentation explicitly open
   (Sec 5).
2. Self-supervised ViT still 4% behind supervised pre-training (Sec 4.6).
3. Small-data regime: ViT overfits more than ResNets (Sec 4.3) — practical
   limitation for most real-world users with <10M images.
4. JFT-300M is private → the flagship results aren't reproducible outside
   Google (mitigated, not eliminated, by the ImageNet-21k row, Table 2).
5. Efficiency claims concern *pre-training*; fine-tuning/inference costs and
   the quadratic sequence-length scaling with resolution are documented but not
   the headline (Sec 3.2, App D.5).
6. No robustness or distribution-shift evaluation in this paper (absence —
   be careful to present as absence, not as a demonstrated flaw).

**Evidence.** As cited; Sec 5 for the authors' own open problems.

**Likely follow-up.** *"Which limitation would you stress in one sentence?"*
Suggested: "It's a classification paper: everything it proves, it proves for
supervised image classification with large pre-training data — the authors say
so themselves."

### Q4.2 Which claims are well-supported vs too strong?

**Model explanation.**
- *Well-supported:* competitive accuracy at scale (Table 2, 3 runs, ±std);
  compute advantage vs ResNets at equal data (Fig 5/Table 6); the data-size
  dependence (Figs 3–4); position-embedding and head-type ablations (Tables 8,
  Fig 9); memory/speed measurements (Fig 12).
- *Supported but scope-limited:* "requires substantially fewer computational
  resources to train" — true vs BiT-L/Noisy Student and vs ResNets in Fig 5;
  "2–4×" is a 5-dataset average with visible per-dataset spread (Table 6).
- *Too strong if extended:* "large scale training trumps inductive bias" as a
  general principle — established here only for these classification
  benchmarks; "ViT approaches or beats state of the art" — true on this table
  in Oct 2020, but the SOTA column is a moving target (Table 2's starred 88.5
  ImageNet entry is a contemporaneous improved result reported in Touvron et
  al. 2020, grafted onto the Noisy Student column by footnote).

**Likely follow-up.** *"Would you defend the 'trumps inductive bias' sentence
in an oral?"* Recommended posture: defend it *within its evidence* (Sec 4.3's
crossover) and volunteer the scope limit before being told.

### Q4.3 What experiment would you run next?

**Model explanation.** Ground your answer in gaps the paper itself flags, then
add one of your own:
- Self-supervised pre-training to close the 4% gap (Sec 5 names it; contrastive
  methods explicitly deferred in Sec 4.6).
- Dense prediction tasks — detection/segmentation (Sec 5) — tests whether
  "no inductive bias" survives outside classification.
- The few-shot study the authors call "an exciting direction" (Sec 4.3).
- *Your own addition (defensible):* repeat the crossover study on fully public
  data (e.g., ImageNet-21k subsets) to de-privatize the paper's central
  finding; or a compute-matched baseline with an equal hyperparameter-search
  budget (closes the B2 gap).

**Evidence.** Sec 4.3; Sec 4.6; Sec 5; working-draft B2 for the search-budget gap.

**Likely follow-up.** *"What result would falsify your proposed experiment's
hypothesis?"* For the public-data crossover: if ViT's crossover stayed at ~90M
images on a public dataset family, the finding generalizes beyond JFT; if it
moved to 300M+, the "inductive bias vs data" story would be JFT-distribution
specific.

### Q4.4 (Bonus) What's the weakest part of the evidence chain?

**Model explanation.** The fine-print of Fig 4: the crossover is located using
*linear few-shot* evaluation with early stopping, a metric that privileges
different representations than fine-tuning — the qualitative story (ViT
overfits small data) is robust, but the exact crossover point is
metric-dependent. Second-weakest: baseline recipe provenance (B2 above).

**Evidence.** Sec 4.3 (metric choice stated: "To save compute, we report
few-shot linear accuracy instead of full fine-tuning accuracy"); B.1.1 (LR grids).

---

## 5. Paper structure

### Q5.1 Why does the paper introduce concepts in this order?

**Model explanation.** Abstract → two-part claim. Sec 1 → tension (NLP scale vs
vision CNN status quo) → minimal idea → *announced* small-data failure →
large-scale resolution. Sec 2 → novelty scoped against three families of prior
attention-for-vision work and the large-scale-transfer literature. Sec 3 →
the deliberately boring method (minimal deviation from NLP) — which is what
makes Sec 4's comparisons attributable to architecture, not engineering. Sec 4
→ escalating controls: 4.2 can it win? → 4.3 how much data does it need? →
4.4 is the win architectural at fixed data/compute? → 4.5 how does it work? →
4.6 what's next? Sec 5 → restrained conclusion matching the evidence's scope.

**Evidence.** Sec 1 paras 1–5; Sec 2 closing paragraph; Sec 3 opening sentence;
Secs 4.1–4.6 transitions.

**Likely follow-up.** *"Why announce the weakness in the intro instead of
burying it in Sec 4.3?"* Because it converts the obvious attack ("only works
with 300M private images") into the paper's thesis — the failure *is* the
finding (see working_draft B5a). It also pre-conditions the reader to read
Sec 4.3 as confirmation rather than damage control.

### Q5.2 Why are the ablations important?

**Model explanation.** Each appendix ablation defends a choice an interviewer
or reviewer would otherwise attack:
- D.1 (Adam vs SGD for ResNet pre-training) → defends fairness of the shared
  recipe — the baselines aren't strawmen (Table 7).
- D.4 (position embeddings incl. a no-position floor) → shows the component's
  worth (0.614 → 0.640) and that the simple 1D default is as good as any scheme
  (Table 8) — together with Sec 4.5's learned-topology analysis, this turns a
  design shortcut into a *finding*.
- D.3 (class token vs GAP) → reveals the token isn't magic; the deficit was a
  learning-rate artifact (Fig 9) — intellectual honesty about an inherited NLP
  convention.
- D.2 (shape ablation) → justifies the scaling direction (depth > width;
  smaller patches > parameters; "compute might be a better predictor of
  performance than the number of parameters").
- D.5/D.6 → replace FLOP folklore with measurements (wall-clock, memory,
  axial-attention timing) supporting the efficiency claim's engineering side.

**Likely follow-up.** *"Which ablation would you cut if space-constrained, and
what would you lose?"* Defensible: D.2 (loses the scaling-guidance story) or
D.6 (loses the strongest support for a Related-Work claim). Do **not** cut D.1
— it protects the headline comparison's fairness.

### Q5.3 How do the figures and tables support the argument?

**Model explanation.** A map from artifact to argumentative job:
- **Figure 1** (architecture diagram) → makes "minimal deviation from NLP"
  *visible*; the figure literally looks like the original Transformer paper's
  diagram.
- **Table 2** → headline credibility: accuracies *and* TPUv3-core-days side by
  side, so "cheaper" is a column, not an adjective.
- **Figures 3–4 + Table 5** → the thesis-in-a-plot: ViT vs ResNet regions
  crossing as pre-training data grows; Table 5 gives every number so the claim
  is auditable without eyeballing curves.
- **Figure 5 + Table 6** → the controlled comparison; plotting vs *compute*
  (not parameters or layers) is itself an argument (cf. D.2's "compute is the
  better predictor").
- **Figures 6–7** → mechanism and interpretability: attention lands on
  semantic regions; position embeddings encode 2D structure — visual evidence
  that the model behaves like a vision model despite lacking built-in bias.
- **Table 9** (VTAB breakdown) → prevents the mean (77.6) from hiding per-task
  weaknesses (sNORB-Azim 33.2) — the honest version of a benchmark table.

**Likely follow-up.** *"Which single figure would you keep for a 60-second
pitch, and why?"* Figure 5 — it carries the paper's most general claim
(architecture efficiency at fixed data) in one axes pair, and every
alternative-explanation discussion routes back to it.

---

## Interview-day quick sheet (self-made — fill from memory, then check)

- One-sentence claim (with its condition): ___
- The surprise I chose, and what I believed before: ___
- The heaviest sentence, and why: ___
- One limitation I volunteer before being asked: ___
- The experiment I'd run next, and what would falsify it: ___
- Three table numbers I can quote exactly: ___
