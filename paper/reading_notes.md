# Reading Notes — "An Image is Worth 16x16 Words" (Vision Transformer, ViT)

Paper: Dosovitskiy et al., arXiv:2010.11929 / ICLR 2021.
All notes refer to `source.md` in this folder. **Citations use section, table,
and figure numbers** (the HTML source has no fixed pagination).

**How to read these notes:** for each section of the paper you get the same six
prompts:

1. What question is this section addressing?
2. What does the paper claim?
3. What evidence or method supports the claim?
4. Why is this section included in the paper?
5. What assumptions or weaknesses should I notice?
6. What should I understand before moving to the next section?

Statements are either **[Paper]** (what the paper actually says) or
**[Watch for]** (assumptions / weaknesses / my analysis — flagged so you can
keep paper findings and interpretation apart). Numbers were checked against
`source.md`; do not quote numbers from memory without re-checking them there.

---

## Abstract

**1. Question.** Can the Transformer — the dominant architecture in NLP — be
applied *directly* to image classification, or does vision genuinely need
convolutional structure?

**2. Claim [Paper].** Two-part claim, both parts matter:
- "We show that this reliance on CNNs is not necessary and a pure transformer
  applied directly to sequences of image patches can perform very well on image
  classification tasks."
- *Conditional:* "When pre-trained on large amounts of data and transferred to
  multiple mid-sized or small image recognition benchmarks (ImageNet, CIFAR-100,
  VTAB, etc.), Vision Transformer (ViT) attains excellent results compared to
  state-of-the-art convolutional networks **while requiring substantially fewer
  computational resources to train**."

**3. Evidence [Paper].** The benchmark results of Section 4 (headline numbers
appear in the Introduction): 88.55% ImageNet, 90.72% ImageNet-ReaL, 94.55%
CIFAR-100, 77.63% VTAB-19 for the best model; compute comparison in Table 2.

**4. Why included.** The abstract states the thesis in one breath: (a) a *pure*
transformer works on images, (b) it needs *scale*, (c) it is *compute-efficient*
relative to CNN state of the art.

**5. Assumptions / weaknesses [Watch for].**
- The abstract's optimism hides the paper's most interesting finding: ViT
  *fails* below a certain data scale (Section 4.3). "Large amounts of data" is
  doing heavy lifting in that sentence.
- "Substantially fewer computational resources" means **pre-training** compute
  against two specific baselines (BiT-L, Noisy Student — Table 2), not "cheaper
  than every CNN in every setting."

**6. Before moving on.** Be able to restate the claim with its three
qualifiers intact: *pure* transformer (no convolutions except optionally in the
hybrid variant), *large-scale pretraining*, *transfer* to smaller benchmarks.

---

## 1 Introduction

**1. Question.** Why should anyone try this now, and what happens when you try
it honestly?

**2. Claims [Paper].**
- NLP: Transformers are "the model of choice"; the dominant recipe is
  pre-train on a large corpus, then fine-tune (BERT, GPT line); efficiency and
  scalability enabled >100B-parameter models with "no sign of saturating
  performance."
- Vision: CNNs remain dominant. Prior attention-based attempts either keep the
  CNN structure (attention as an add-on) or replace convolutions with
  *specialized* attention patterns that "have not yet been scaled effectively
  on modern hardware accelerators." So ResNet-like architectures are still SOTA
  at large scale.
- The idea: "we split an image into patches and provide the sequence of linear
  embeddings of these patches as an input to a Transformer. Image patches are
  treated the same way as tokens (words) in an NLP application."
- The honest failure: on mid-sized datasets (e.g. ImageNet alone, without
  strong regularization), ViT yields "modest accuracies of a few percentage
  points below ResNets of comparable size" — expected, because "Transformers
  lack some of the inductive biases inherent to CNNs, such as translation
  equivariance and locality, and therefore do not generalize well when trained
  on insufficient amounts of data."
- The resolution: "We find that large scale training trumps inductive bias."
  With 14M–300M pre-training images, ViT "approaches or beats state of the art
  on multiple image recognition benchmarks" (88.55 ImageNet / 90.72 ReaL /
  94.55 CIFAR-100 / 77.63 VTAB for the best model).

**3. Evidence [Paper].** The full Section 4 program: SOTA comparison (4.2),
data-requirements study (4.3), compute-controlled scaling study (4.4).

**4. Why included [Paper + Watch for].** The introduction is structured as
tension and release: NLP's scaling trajectory vs vision's CNN stagnation → the
simplest possible idea → an *announced* failure at small scale → success at
large scale. **[Watch for]** Announcing the weakness up front converts the
paper's biggest vulnerability ("it only works because you have 300M private
images") into part of the thesis. This is a framing choice worth noticing for
the craft sections of the template.

**5. Assumptions / weaknesses [Watch for].**
- "Large scale training trumps inductive bias" is demonstrated for the image
  *classification* benchmarks tested — not for vision generally.
- "No sign of saturating" describes the 2020 NLP situation; it is context, not
  a result of this paper.

**6. Before moving on.** The two hinges of the whole paper: (1) **patch = token**
(the only vision-specific machinery), and (2) **inductive bias as a
data-dependent crutch** — useful when data is scarce, dispensable when data is
abundant. Everything in Section 4 tests one of these two hinges.

---

## 2 Related Work

**1. Question.** What has been tried before, and what exactly is new here?

**2. Claims [Paper].** The section sorts prior work into families and positions
ViT against each:
- *Local attention* (Parmar et al. 2018; Hu et al. 2019; Ramachandran et al.
  2019): attention restricted to local neighborhoods; can fully replace
  convolution.
- *Scalable approximations to global attention* (Sparse Transformers; axial
  attention; blocks of varying sizes): promising, but "require complex
  engineering to be implemented efficiently on hardware accelerators."
- *Closest prior work* — Cordonnier et al. 2020: extracts 2×2 patches and applies
  full self-attention on top. "This model is very similar to ViT, but our work
  goes further to demonstrate that large scale pre-training makes vanilla
  transformers competitive with (or even better than) state-of-the-art CNNs."
  Also: 2×2 patches only work at low resolution; ViT handles medium-resolution.
- *CNN+attention hybrids* and multimodal models — complementary line of work.
- *iGPT* (Chen et al. 2020a): transformers on (downscaled) pixels, generative
  self-supervised pre-training; 72% ImageNet accuracy.
- *Large-scale supervised pre-training* (Mahajan, Touvron, Xie; Sun et al. 2017
  on data scaling; BiT and Djolonga et al. on transfer from ImageNet-21k/JFT):
  "We focus on these two latter datasets as well, but train Transformers
  instead of ResNet-based models."

**3. Evidence [Paper].** Citations only — this section is positioning, not
experiment. Its factual assertions (e.g., specialized attention being slow on
accelerators) are tested later in Appendix D.6 and Figure 12.

**4. Why included.** It narrows the novelty claim to exactly what it is:
**not** "attention for vision" (many tried), **not** "patches + attention"
(Cordonnier et al. did it), but *"a vanilla, minimally-modified NLP Transformer
trained at scale is competitive with or better than SOTA CNNs — demonstrated
with an efficient, hardware-friendly implementation."*

**5. Assumptions / weaknesses [Watch for].**
- "Specialized attention is hard to implement efficiently" is an engineering
  judgment supported mainly by the authors' own experience (Appendix D.6:
  AxialResNet "is very slow on TPUs"). It justifies the design choice but is
  not a universal result.
- The section cannot cite evidence for its own key novelty claim — that is what
  Section 4 must deliver. Notice how modestly it is worded: "goes further to
  demonstrate," not "is the first."

**6. Before moving on.** Know the one-sentence delta vs Cordonnier et al.
(large-scale pretraining + practical 16×16 patches) and vs iGPT (supervised
classification rather than generative pre-training; higher accuracy). This is
your answer to "what is *genuinely* new?" in the interview.

---

## 3 Method

**1. Question.** How do you feed an image to a Transformer with the *fewest
possible modifications* to the NLP recipe?

**2. Claims / design [Paper].** "In model design we follow the original
Transformer as closely as possible. An advantage of this intentionally simple
setup is that scalable NLP Transformer architectures — and their efficient
implementations — can be used almost out of the box."

### 3.1 Vision Transformer (ViT) — the pipeline (Figure 1, Eqs. 1–4)

In plain language, the five steps:

1. **Cut into patches.** Image **x** ∈ ℝ^(H×W×C) is reshaped into N = HW/P²
   flattened patches **x_p** ∈ ℝ^(N×(P²·C)). With the standard 224×224 input,
   P=16 gives N=196 patches — the "16×16 words" of the title. N is the
   Transformer's sequence length.
2. **Linear projection ("patch embedding").** Each flattened patch is mapped to
   a constant width D by a trainable linear projection **E** ∈ ℝ^((P²·C)×D)
   (Eq. 1). This replaces the word-embedding layer of NLP.
3. **Prepend the [class] token and add position embeddings.** A learnable
   vector **x_class** is prepended (borrowed from BERT); its output state is
   the image representation. Learnable 1D position embeddings **E_pos** ∈
   ℝ^((N+1)×D) are added so the model knows which patch is where (Eq. 1). The
   paper uses plain 1D embeddings "since we have not observed significant
   performance gains from using more advanced 2D-aware position embeddings
   (Appendix D.4)."
4. **Transformer encoder.** L blocks of alternating multi-head self-attention
   (MSA) and MLP (Eqs. 2–3), LayerNorm *before* each block, residual
   connections after; MLP = two layers with GELU. Constant latent size D
   throughout. Appendix A gives the standard qkv attention equations (5–8);
   with k heads, per-head width D_h = D/k keeps compute constant.
5. **Readout / classification head.** Final LayerNorm on the [class] output
   (Eq. 4); head = MLP with one hidden layer during pre-training, a single
   linear layer during fine-tuning.

**Hybrid variant [Paper].** Instead of raw pixels, the input sequence can come
from a CNN feature map (patches of size 1×1 on the feature map) — tested in 4.4.

### Inductive bias [Paper, Sec 3.1 box]

"Vision Transformer has much less image-specific inductive bias than CNNs."
In CNNs, locality, 2D neighborhood structure, and translation equivariance are
"baked into each layer throughout the whole model." In ViT, only the MLP
layers are local and translation-equivariant; self-attention is global; 2D
structure enters only (i) when cutting patches and (ii) at fine-tuning time
when interpolating position embeddings. "All spatial relations between the
patches have to be learned from scratch."

### 3.2 Fine-tuning and higher resolution

**3. Claims [Paper].**
- Transfer = remove the pre-trained head, attach a zero-initialized D×K linear
  layer for the target's K classes (Appendix B.1.1: they replace the *whole*
  two-layer head, "a little more robust than simply re-initializing the very
  last layer").
- Fine-tuning often benefits from higher resolution than pre-training. Patch
  size is kept fixed → longer sequence → pre-trained position embeddings no
  longer match → **2D-interpolate the position embeddings** according to their
  original grid location.
- "Note that this resolution adjustment and patch extraction are the only
  points at which an inductive bias about the 2D structure of the images is
  manually injected into the Vision Transformer."

**4. Why included.** The Method section is deliberately minimal — it establishes
that any performance difference vs CNNs cannot come from clever vision-specific
architecture engineering, because there almost isn't any. This is what makes
Section 4's comparisons meaningful.

**5. Assumptions / weaknesses [Watch for].**
- Each patch is encoded *independently* by the linear projection before any
  mixing — local correlations *within* a patch beyond one linear map must be
  handled later or not at all. (Bigger patches = coarser input; see 4.1 note
  that sequence length scales with 1/P².)
- Self-attention is O(N²) in sequence length — the reason patch size is the
  compute knob (stated in 4.1: "the Transformer's sequence length is inversely
  proportional to the square of the patch size").
- The [class] token is inherited from BERT, not argued for in the main text.
  Appendix D.3 later shows global average pooling works equally well *with a
  different learning rate* — so the token is convention, not necessity.

**6. Before moving on.** Be able to redraw Figure 1 from memory and explain
Eqs. 1–4 in words without notation. The interview "how does the architecture
work" question is answered by steps 1–5 above.

---

## 4 Experiments

**1. Question.** Does it work, how much data does it need, and is the win
really about architecture rather than data or training recipe?

**2–3. Setup, claims, evidence by subsection:**

### 4.1 Setup

- **Datasets [Paper]:** ImageNet (1k classes, 1.3M images), ImageNet-21k (21k
  classes, 14M), JFT (18k classes, 303M, **in-house**). Pre-training sets are
  **de-duplicated w.r.t. downstream test sets** (following BiT). Transfer
  targets: ImageNet (original + cleaned ReaL labels), CIFAR-10/100, Oxford-IIIT
  Pets, Oxford Flowers-102, and the 19-task **VTAB** suite (1,000 training
  examples per task; groups: Natural, Specialized, Structured).
- **Models [Paper, Table 1]:** ViT-Base (12 layers, D=768, 86M params),
  ViT-Large (24 layers, D=1024, 307M), ViT-Huge (32 layers, D=1280, 632M);
  notation ViT-{size}/{patch}, e.g. ViT-L/16. Baselines: "ResNet (BiT)" =
  ResNet with BatchNorm replaced by GroupNorm + standardized convolutions
  (better transfer); hybrids = R50 feature map into ViT with patch size 1.
- **Training [Paper]:** Adam (β1=0.9, β2=0.999), batch 4096, weight decay 0.1,
  linear LR warmup+decay (Appendix B.1, Table 3 gives per-model settings;
  ImageNet-from-scratch gets strong regularization: wd 0.3, dropout 0.1, cosine
  decay, 300 epochs). Fine-tuning: SGD with momentum, batch 512, LR grid search
  (Table 4), default resolution 384. The Table-2 ImageNet numbers additionally
  use 512 (ViT-L/16) / 518 (ViT-H/14) resolution and Polyak averaging (0.9999).
- **Metrics [Paper]:** fine-tuning accuracy; "few-shot" = regularized
  least-squares on *frozen* features (closed-form; a linear probe).

**Why this setup matters [Watch for].** Three pre-training datasets spaced ~10×
apart make the *data-size* question answerable; one shared recipe across
architectures makes the comparison fairer. Two caveats: JFT is private
(reproducibility limit — mitigated by also reporting ImageNet-21k results);
and although ResNets get the same optimizer, D.1 exists precisely because
Adam-for-ResNet is "quite unconventional" — the recipe was authored in a ViT
lab, so baseline undertuning is a legitimate worry the authors partially close.

### 4.2 Comparison to State of the Art (Table 2, Figure 2)

**Claim [Paper].** ViT-L/16 pre-trained on JFT-300M beats BiT-L (same
pre-training data) on **all** reported tasks with much less compute; ViT-H/14
improves further, "especially on the more challenging datasets."

**[Watch for]** The table is slightly kinder than the prose: on ImageNet-ReaL
ViT-L/16 and BiT-L are exactly tied (90.54 vs 90.54), and on VTAB the table
shows 76.28±0.46 vs 76.29±1.70 — a statistical tie. "Outperforms on all tasks"
survives only as "matches or beats." If you use this claim in the interview,
quote the table.

**Verified numbers [Paper, Table 2]** (mean ± std over 3 fine-tuning runs):

| | ViT-H/14 (JFT) | ViT-L/16 (JFT) | ViT-L/16 (IN-21k) | BiT-L (ResNet152x4) | Noisy Student |
|---|---|---|---|---|---|
| ImageNet | **88.55±0.04** | 87.76±0.03 | 85.30±0.02 | 87.54±0.02 | 88.4/88.5* |
| ImageNet ReaL | **90.72±0.05** | 90.54±0.03 | 88.62±0.05 | 90.54 | 90.55 |
| CIFAR-10 | **99.50±0.06** | 99.42±0.03 | 99.15±0.03 | 99.37±0.06 | – |
| CIFAR-100 | **94.55±0.04** | 93.90±0.05 | 93.25±0.05 | 93.51±0.08 | – |
| Oxford-IIIT Pets | **97.56±0.03** | 97.32±0.11 | 94.67±0.15 | 96.62±0.23 | – |
| Oxford Flowers-102 | 99.68±0.02 | **99.74±0.00** | 99.61±0.02 | 99.63±0.03 | – |
| VTAB (19 tasks) | **77.63±0.23** | 76.28±0.46 | 72.72±0.21 | 76.29±1.70 | – |
| TPUv3-core-days | 2.5k | 0.68k | 0.23k | 9.9k | 12.3k |

(*\* = slightly improved 88.5% reported by Touvron et al. 2020.*) The public
ImageNet-21k ViT-L/16 "could be trained using a standard cloud TPUv3 with 8
cores in approximately 30 days" (Sec 4.2 text).

**Honesty notes [Paper].** The authors immediately add that pre-training
efficiency "may be affected not only by the architecture choice, but also other
parameters, such as training schedule, optimizer, weight decay" — which is
their stated reason for the controlled study in 4.4. **[Watch for]** The
Table-2 ImageNet number for ViT-L/16 (I21k) is 85.30 while Table 5 shows 85.15:
Table 5's caption explains the difference (Table 2 adds Polyak averaging and
higher-resolution fine-tuning). Don't quote the two as contradictory.

**VTAB breakdown [Paper, Fig 2 + Table 9]:** ViT-H/14 beats BiT-R152x4 and
other methods on Natural and Structized groups; "on the Specialized the
performance of the top two models is similar." Per-task (Table 9), ViT-H/14's
weakest VTAB tasks include sNORB-Azim (33.2) and DMLab (53.1); its strongest
are Flowers102 (99.7) and Pets (97.2).

### 4.3 Pre-training Data Requirements (Figures 3–4, Table 5)

**Question [Paper].** "With fewer inductive biases for vision than ResNets, how
crucial is the dataset size?"

**Experiment 1 [Paper].** Pre-train on ImageNet → ImageNet-21k → JFT-300M
(three models' worth of scale), tuning three basic regularization parameters
(weight decay, dropout, label smoothing) for the smaller datasets; evaluate by
fine-tuning to ImageNet (Fig 3/4; full numbers in Table 5). Findings:
- On ImageNet-scale pre-training, **ViT-Large underperforms ViT-Base** despite
  regularization; with ImageNet-21k they are similar; "Only with JFT-300M, do
  we see the full benefit of larger models."
- "The BiT CNNs outperform ViT on ImageNet, but with the larger datasets, ViT
  overtakes."
- Spot-check from Table 5 (ImageNet top-1 after fine-tuning): ViT-L/16 gets
  76.53 from ImageNet pre-training, 85.15 from ImageNet-21k, 87.12 from JFT —
  an ~11-point swing driven purely by pre-training data.

**Experiment 2 [Paper].** Random subsets of JFT: 9M / 30M / 90M / 300M, **same
hyper-parameters for all sizes and no extra regularization** ("this way, we
assess the intrinsic model properties, and not the effect of regularization"),
early stopping, few-shot linear metric to save compute. Findings:
- "Vision Transformers overfit more than ResNets with comparable computational
  cost on smaller datasets." ViT-B/32 (slightly faster than ResNet50) is much
  worse on the 9M subset but better on 90M+; same crossover for
  ResNet152x2 vs ViT-L/16.
- "This result reinforces the intuition that the convolutional inductive bias
  is useful for smaller datasets, but for larger ones, learning the relevant
  patterns directly from data is sufficient, even beneficial."

**Why this is the paper's key experiment [Watch for].** It converts an
architecture claim into a *data-scaling* claim and locates the crossover. It
also makes the earlier failure (small-data ViT) falsifiable and quantified
instead of hand-waved. Caveats: the crossover location depends on the metric
(linear few-shot probing, which measures something different from fine-tuning)
and on ImageNet as the probe; the subsets come from JFT's distribution, which
is itself large-scale web data.

### 4.4 Scaling Study (Figure 5, Table 6)

**Design [Paper].** With data no longer the bottleneck (everything pre-trained
on JFT-300M), compare **performance vs pre-training compute**: 7 ResNets
(R50x1…R200x3), 6 ViTs (B/32…H/14), 5 hybrids (R50+ViT-B/32…R50+ViT-L/16), at
7 and 14 epochs.

**Findings [Paper], verbatim in substance:**
1. "Vision Transformers dominate ResNets on the performance/compute trade-off.
   ViT uses approximately 2−4× less compute to attain the same performance
   (average over 5 datasets)."
2. "Hybrids slightly outperform ViT at small computational budgets, but the
   difference vanishes for larger models. This result is somewhat surprising,
   since one might expect convolutional local feature processing to assist ViT
   at any size."
3. "Vision Transformers appear not to saturate within the range tried."

**Evidence quality [Watch for].** This is the strongest control in the paper:
same data, same recipe, curves against compute (Table 6 gives every point, in
exaFLOPs — e.g. ViT-L/16@14ep: 87.12 ImageNet at 1567 exaFLOPs vs
ResNet152x2@14ep: 85.56 at 1126, ResNet200x3@14ep: 87.22 at 3306). Remaining
limits: only two training durations (7/14 epochs); "2–4×" is an average over
5 datasets with per-dataset spread; compute is estimated for TPUv3; hybrid
configurations are only R50-based.

### 4.5 Inspecting Vision Transformer (Figures 6–7, Appendix D.7–D.8)

**Question [Paper].** "To begin to understand how the Vision Transformer
processes image data, we analyze its internal representations."

**Findings [Paper].**
- *Patch projection (Fig 7, left):* top principal components of the learned
  filters "resemble plausible basis functions for a low-dimensional
  representation of the fine structure within each patch."
- *Position embeddings (Fig 7, center):* cosine similarity between position
  embeddings tracks spatial distance; row/column structure emerges; sometimes a
  sinusoidal pattern. "That the position embeddings learn to represent 2D image
  topology explains why hand-crafted 2D-aware embedding variants do not yield
  improvements (Appendix D.4)."
- *Attention distance — the receptive-field analog (Fig 7, right; D.7):*
  "some heads attend to most of the image already in the lowest layers...
  Other attention heads have consistently small attention distances in the low
  layers"; attention distance increases with depth; in hybrids the localized
  low-layer attention is less pronounced, "suggesting that it may serve a
  similar function as early convolutional layers in CNNs."
- *Attention maps (Fig 6; method in D.8, Attention Rollout):* the model "attends
  to image regions that are semantically relevant for classification."

**Why included [Watch for].** This section upgrades the paper from "X works"
toward "why X works": it explains a *negative* ablation result (why 2D
embeddings don't help — the model learns 2D structure itself) and shows the
global-mixing capability that attention is supposed to provide is actually
used, even in layer 1. Caveats: all probes are correlational/qualitative;
"attention distance" is a proxy, not a causal intervention; the Fig 6 examples
are "representative," i.e. selected.

### 4.6 Self-supervision (Appendix B.1.2)

**Question [Paper].** NLP's success is partly due to self-supervised
pre-training (BERT/GPT); does that transfer to vision?

**Design [Paper, B.1.2].** Masked patch prediction mimicking BERT: corrupt 50%
of patch embeddings — 80% replaced by a learnable [mask] embedding, 10% by a
random other patch's embedding, 10% kept; predict the **3-bit mean color (512
colors)** of every corrupted patch. 1M steps (~14 epochs), batch 4096, Adam.
They tried three target variants (mean color; 4×4 downsampled patch in
parallel; full-patch L2 regression) — "Surprisingly, we found that all worked
quite well, though L2 was slightly worse."

**Result [Paper].** "With self-supervised pre-training, our smaller ViT-B/16
model achieves 79.9% accuracy on ImageNet, a significant improvement of 2% to
training from scratch, but still 4% behind supervised pre-training." Diminishing
returns after ~100k steps; similar gains pre-training on ImageNet alone.
Contrastive methods explicitly left to future work.

**Why included.** A small, honest "future direction" experiment that mirrors
the NLP parallel without overclaiming — and it quantifies the remaining gap
(2 points gained of 6 possible).

---

## 5 Conclusion

**1. Question.** What was established, and what remains open?

**2. Claims [Paper].**
- "Unlike prior works using self-attention in computer vision, we do not
  introduce image-specific inductive biases into the architecture apart from
  the initial patch extraction step."
- "This simple, yet scalable, strategy works surprisingly well when coupled
  with pre-training on large datasets. Thus, Vision Transformer matches or
  exceeds the state of the art on many image classification datasets, whilst
  being relatively cheap to pre-train."
- Open challenges named: (1) apply ViT to detection and segmentation;
  (2) self-supervised pre-training — "there is still large gap between
  self-supervised and large-scale supervised pre-training"; (3) "further
  scaling of ViT would likely lead to improved performance."

**3–5. Why included / limits [Watch for].** The conclusion stays carefully
inside the evidence: "many image classification datasets," "matches or
exceeds," and an explicit list of what is *not* yet shown (other tasks,
self-supervision parity, more scaling). Not mentioned anywhere: robustness,
distribution shift, efficiency at fine-tuning/inference time, or non-classification
vision. If you criticize the paper in the interview, the fair scope is
"supervised image classification with large pre-training data" — the authors
hold themselves to exactly that scope.

---

## Appendices (high-value contents checklist)

- **A — Multihead self-attention:** standard qkv equations (5–8); k heads run
  in parallel with per-head width D_h = D/k to keep compute and parameters
  constant; outputs concatenated and projected.
- **B.1 / Table 3 — training hyper-parameters:** per-model LRs, 7/14/30/90/300
  epochs; batch 4096; 10k-step warmup; "We found strong regularization to be key
  when training models from scratch on ImageNet" (wd 0.3, dropout 0.1, cosine);
  dropout placement (after every dense layer except qkv projections, and right
  after adding position embeddings).
- **B.1.1 / Table 4 — fine-tuning:** SGD momentum 0.9, batch 512, no weight
  decay, cosine decay, grad clipping 1.0; per-dataset step counts and LR grids;
  dev-split protocol for LR selection; whole-head replacement with a single
  zero-initialized layer; VTAB uses one setting for all 19 tasks (LR 0.01,
  2500 steps) and benefits most from 384×384 resolution on every task.
- **B.1.2:** self-supervision details (see 4.6 above).
- **C / Tables 5–6:** the exact numbers behind Figures 4 and 5 — quoted above;
  use these tables (not plot read-offs) when you need per-model accuracies and
  exaFLOP costs.
- **D.1 / Table 7 — the optimizer control:** Adam beats SGD for *ResNet*
  pre-training on JFT in their setting (average 89.33 vs 88.79 for R50x1;
  94.01 vs 93.72 for R152x2) — this defends the shared-recipe fairness claim.
- **D.2 / Figure 8 — Transformer shape ablation:** depth helps most (visible up
  to 64 layers, diminishing after 16); width helps least; smaller patch size
  (longer sequence) gives "surprisingly robust improvements without introducing
  parameters." Takeaway sentence: "compute might be a better predictor of
  performance than the number of parameters."
- **D.3 / Figure 9 — [class] token vs global average pooling (GAP):** initial
  GAP attempt "performed very poorly," but "the difference in performance is
  fully explained by the requirement for a different learning-rate." So the
  class token is *not* necessary — convention inherited from BERT.
- **D.4 / Table 8 — positional embedding ablation** (ViT-B/16, ImageNet
  5-shot linear): no position embeddings 0.61382; 1D (default) 0.64206; 2D
  0.64001; relative 0.64032; per-layer variants ~0.640–0.643. Big gap vs
  "none," negligible differences among methods — supports the simple default.
- **D.5 / Figure 12 — wall-clock and memory:** ViT speed "comparable to similar
  ResNets"; large ViTs "clearly more memory-efficient"; the feared quadratic
  scaling with image size "only barely starts happening for the largest models
  at the largest resolutions" (on TPUv3).
- **D.6 / Figure 13 — axial attention baselines:** AxialResNet reproduces
  published accuracy but is "extremely slow on TPUs"; Axial-ViT beats plain
  ViT-B at higher compute. Supports the Related Work claim about specialized
  attention patterns.
- **D.9:** ObjectNet: ViT-H/14 gets 61.7% top-1 / 82.1% top-5.
- **D.10 / Table 9:** full VTAB per-task breakdown (quoted above).

---

## One-page synthesis (for your own words, not for submission verbatim)

- **Central claim:** a vanilla NLP Transformer, fed patch embeddings, matches
  or beats SOTA CNNs on image classification — *given enough pre-training
  data* — at 2–4× less pre-training compute than comparable ResNets.
- **Genuinely new:** the demonstration that vision-specific inductive biases
  can be dropped entirely (patch extraction is the only vision-specific step)
  once data is large enough; plus the quantified *crossover* between data scale
  and inductive bias.
- **Strongest evidence:** the compute-controlled scaling study (4.4, Fig 5 /
  Table 6) and the data-requirements study (4.3, Figs 3–4 / Table 5), because
  they isolate architecture from data and recipe.
- **Honest limits:** JFT is private; small-data ViT is worse than CNNs;
  self-supervised ViT still lags supervised by 4 points; everything is
  classification — detection/segmentation untested (their own future work).
