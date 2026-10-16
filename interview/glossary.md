# Glossary — Terms for Discussing the ViT Paper

Each entry: **plain language first**, then **how it appears in this specific
paper** (with section references into `paper/source.md`). Terms are ordered
roughly by how early you need them.

---

### Transformer
A neural network architecture built from stacked blocks of self-attention and
small feed-forward networks, originally designed for text (Vaswani et al.,
2017, "Attention is All You Need"). It processes a *set/sequence* of vectors
("tokens") and has no built-in notion of grid structure, distance, or
neighborhood.

**In this paper:** the entire classifier is "a standard Transformer encoder...
used almost out of the box" (Sec 3). The paper's whole point is that this
text-native architecture, minimally modified, works on images (Sec 3.1).

### Self-attention
A operation where every element of a sequence computes a compatibility score
("query vs key") with every other element, and each element's output is a
weighted average of all elements' values, with those weights. Result: any
token can directly mix information from any other token in one step, regardless
of distance.

**In this paper:** Eqs. 5–7 (Appendix A) give the standard qkv formulation; it
is the reason ViT can integrate information across the *whole image even in the
lowest layers* — verified empirically as "attention distance" (Sec 4.5,
Fig 7 right; D.7), where some layer-0 heads already attend to most of the
image. The cost is quadratic in sequence length — the reason patch size P is
the compute knob (Sec 4.1).

### Patch embedding
Cutting an image into non-overlapping fixed-size patches (here 16×16),
flattening each patch into a vector, and mapping it to width D with a single
trainable linear layer. It plays the role of the word-embedding layer in NLP.

**In this paper:** Sec 3.1, Eq. 1: image x ∈ ℝ^(H×W×C) → N = HW/P² patches →
linear projection **E** ∈ ℝ^((P²·C)×D). A 224² image with P=16 becomes a
sequence of 196 tokens — the title's "16×16 words." The first-layer filters
look like plausible image basis functions (Sec 4.5, Fig 7 left). This step,
plus position-embedding interpolation, is the *only* vision-specific machinery
in the model (Sec 5).

### Positional embedding
An additive vector that tells each token *where* it sits in the input, since
self-attention by itself is order-blind ("bag of tokens"). Can be fixed
(sinusoidal) or learned; absolute (where am I?) or relative (how far am I from
you?).

**In this paper:** learnable 1D embeddings added to patch embeddings (Eq. 1).
The ablation in D.4 (Table 8) shows any positional scheme (~0.640, ImageNet
5-shot linear) beats none (0.61382), while 1D ≈ 2D ≈ relative — and Sec 4.5
(Fig 7 center) explains why: the embeddings *learn* 2D distance and row/column
structure on their own. At fine-tuning on higher-resolution images, the
pre-trained position embeddings are 2D-interpolated to the new grid (Sec 3.2).

### Classification token ([class] token)
One extra learnable vector prepended to the input sequence; its final
state after the encoder is used as the representation of the whole input (as
opposed to pooling all tokens). Borrowed from BERT.

**In this paper:** Sec 3.1, Eq. 1 (prepend) and Eq. 4 (readout). Honest
footnote in D.3: replacing it with global average pooling works *just as well
if the learning rate is adjusted* — so the token is an NLP-fidelity choice,
not a discovered necessity (Fig 9). The pre-training head on top of it is a
1-hidden-layer MLP; fine-tuning replaces the whole head with one
zero-initialized linear layer (Sec 3.1; B.1.1).

### Transformer encoder
The standard stack that forms the Transformer's "reading" half: repeated
blocks of (multi-head self-attention, then a two-layer MLP), each wrapped with
residual connections and LayerNorm applied *before* the block (pre-LN
variant). It outputs one vector per input token.

**In this paper:** Sec 3.1, Eqs. 2–3; L blocks, constant width D; Table 1 gives
the shapes (ViT-B: 12 blocks, D=768, 86M params; ViT-L: 24/1024/307M; ViT-H:
32/1280/632M). Figure 1's encoder illustration deliberately mirrors Vaswani
et al.'s original diagram (Fig 1 caption).

### Pre-training
The first, expensive training phase on a very large (often generic) dataset,
producing weights that already encode useful general features. In modern usage
here it is *supervised* pre-training (labels available), not to be confused
with self-supervised pre-training.

**In this paper:** supervised pre-training on ImageNet (1.3M images),
ImageNet-21k (14M), or JFT-300M (303M, private) (Sec 4.1). The head is
replaced after pre-training. Pre-training cost is reported in TPUv3-core-days
(Table 2) and exaFLOPs (Table 6) — the currency of the paper's efficiency
claim. Data scale during pre-training is the paper's central moderating
variable (Sec 4.3).

### Fine-tuning
Taking a pre-trained model, attaching a fresh task-specific head, and
continuing training (usually all layers, lower learning rate) on the smaller
target dataset.

**In this paper:** Sec 3.2 and B.1.1 — remove the whole pre-trained head,
attach a zero-initialized D×K linear layer, train with SGD+momentum (batch
512, LR grid search, Table 4), typically at *higher resolution* (384; 512/518
for the Table 2 ImageNet headline) than pre-training — made possible by
interpolating the position embeddings (Sec 3.2). VTAB fine-tuning uses 1,000
examples per task (Sec 4.1).

### Inductive bias
The set of assumptions a model architecture builds in that steer it toward
certain solutions independently of data. CNNs assume locality (nearby pixels
matter together), 2D neighborhood structure, and translation equivariance
(the same feature detector applies everywhere). Fewer biases mean more
flexibility but a higher data requirement to *learn* what others assume.

**In this paper:** the paper's central conceptual axis (Sec 3.1 "Inductive
bias" box; Sec 1). ViT has almost none of CNNs' vision biases — self-attention
is global, only MLPs are local, 2D structure enters only via patch cutting and
position-embedding interpolation. The empirical finding: this bias is a
*data-dependent asset* — helpful below roughly 10–90M pre-training images,
dispensable above (Sec 4.3): "large scale training trumps inductive bias."
Local attention heads in low layers may re-learn a bias resembling early
convolutions (Sec 4.5) — an observation, not a proof.

### Convolution
A layer that slides small learned filters over a 2D grid, each filter computing
a weighted sum over a small local neighborhood; this single operation bakes in
locality and translation equivariance. CNNs (LeCun → AlexNet → ResNet) were
vision's dominant architecture for a decade.

**In this paper:** the incumbent being challenged. Baselines are ResNets — but
modified "ResNet (BiT)": GroupNorm instead of BatchNorm, standardized
convolutions (Sec 4.1). Findings vs CNNs: CNNs win on small pre-training data;
ViTs win the compute-performance trade-off at scale (~2–4× less compute, Sec
4.4, Fig 5); hybrids help only at small budgets (Sec 4.4); large ViTs are more
memory-efficient (D.5, Fig 12).

### Ablation study
An experiment that removes, replaces, or varies one component of a system
while holding everything else fixed, to measure that component's actual
contribution (or the sensitivity of results to a choice).

**In this paper:** the Appendix D toolkit — position-embedding variants
including a no-position floor (D.4, Table 8: 0.614 vs ~0.640), class token vs
average pooling (D.3, Fig 9), model-shape axes: depth/width/patch size (D.2,
Fig 8: depth helps most, width least, smaller patches "surprisingly robust...
without introducing parameters"), optimizer for the *baselines* (D.1, Table 7:
Adam ≥ SGD for ResNet pre-training in their setup). The ablations do
argumentative work: they defend every unconventional choice against the
obvious reviewer objection.

### Benchmark
A fixed dataset + evaluation protocol that lets papers compare on equal
ground. Weaknesses: saturation (near-ceiling accuracy), leaderboard overfitting,
and hidden distribution specifics.

**In this paper:** six transfer benchmarks — ImageNet (+ cleaned ReaL labels),
CIFAR-10/100, Oxford-IIIT Pets, Oxford Flowers-102 — plus the 19-task **VTAB**
suite (1,000 training examples per task, grouped Natural / Specialized /
Structured; Sec 4.1) and ObjectNet (D.9: 61.7 top-1 / 82.1 top-5 for ViT-H/14).
The VTAB per-task breakdown (Table 9) shows ViT-H/14's weakest tasks
(sNORB-Azim 33.2, DMLab 53.1) — use it when asked about weaknesses rather than
the 77.6 mean.

### Transfer learning
Using a model pre-trained on one (usually large) dataset as the starting point
for another task, instead of training from scratch. The pre-trained weights are
the transferable asset; fine-tuning adapts them.

**In this paper:** the entire evaluation protocol: pre-train on ImageNet /
ImageNet-21k / JFT, then transfer to the benchmarks above (Sec 4.1–4.2). The
paper is a *representation-learning* study (Sec 4's opening sentence): what
matters is the quality of transferred features, which is why Table 2's
cross-benchmark column structure and VTAB's low-data regime (1k examples/task)
are the right tests. The data-requirements study (4.3) is effectively a map of
*when transfer from ViT is safe*.

---

## A few adjacent terms you may hear in the interview (quick forms)

- **Top-1 accuracy:** fraction of images where the single highest-scoring class
  is correct — the headline metric in Tables 2/5/6. (Top-5 is more forgiving.)
- **Linear probe / few-shot linear evaluation:** freeze the model, train only a
  simple linear classifier on its features (here: closed-form regularized least
  squares, Sec 4.1). Measures representation quality without full fine-tuning —
  used for the Fig 4 crossover curves.
- **Polyak averaging / weight averaging:** averaging successive weight
  snapshots during training for a small, free accuracy bump — used only for
  the Table 2 ImageNet numbers (factor 0.9999, Sec 4.1).
- **Scaling (up):** enlarging model/data/compute and observing how performance
  moves; the paper's Fig 8 (model shape) and Fig 5 (compute) are scaling
  studies, and "compute might be a better predictor of performance than the
  number of parameters" (D.2) is a scaling-law-flavored claim.
- **ExaFLOPs / TPUv3-core-days:** concrete compute currencies the paper reports
  (Table 2, Table 6) so "efficient" is checkable.
