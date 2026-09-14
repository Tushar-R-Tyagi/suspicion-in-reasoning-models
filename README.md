# A Suspicion Direction That Doesn't Refuse

A small mechanistic-interpretability project on **Qwen3.5-9B**, asking whether a
reasoning model carries a real, isolable internal "I don't trust this user" state
after the user confesses malicious intent, and whether that state actually drives
its behaviour.

**Short version of what I found:** there is a real, linear "suspicion" direction in
the residual stream. It generalizes to scenarios it was never built from, it is
distinct from the refusal direction, and injecting it causally changes behaviour, but
it is only loosely coupled to what the model visibly does. The model performs caution
more consistently than the internal signal predicts.

---

## What's in this repo

- **The notebook** — the full working notebook (Colab), with all cells and outputs.
  Fair warning: this is a **real research notebook, not a cleaned library**. It
  contains exploratory scaffolding, dead ends, and cells that aren't guaranteed to run
  top-to-bottom in order. The write-up is the clean account of what was actually done;
  the notebook is the raw workings behind it.
- **The write-up** — executive summary + full write-up, including randomly-selected
  raw transcripts. This is the best starting point if you want to understand the
  project without reading code.
- **outputs/** — generated figures, result CSVs, and exported transcripts.

---

## The findings, briefly

- **Behavioural (n=5/condition):** a malicious admission collapses answer-completion
  rate (100% cold → 40% admission → 87% after retraction). Caveat: "completion" is a
  token-budget-confounded proxy for deliberation length, not a clean refusal.
- **A real suspicion direction (n=10, one scenario):** built by difference-of-means at
  layer 11; stable across 4 train/test splits (z 35.7–127), 20–300× above a
  random-direction noise floor.
- **Generalizes:** built from one scenario, it separates baseline from confession on
  two unseen scenarios, 10/10 samples positive.
- **Not refusal:** cosine 0.02 with a *validated* refusal direction, against a 0.70
  within-direction self-consistency anchor.
- **Causal:** injected into a clean prompt, it suppressed working exploit code in 4/4
  seeds; a matched random direction did so in 0/4.
- **Decoupled from behaviour:** the single most overt refusal in the project scored
  ~0 on the direction (one data point, illustrative not conclusive).

---

## Method / concepts

Difference-of-means direction extraction (following Arditi et al.), residual-stream
activation reads, forward-hook injection and ablation, projection scores. Validation
via split-half scoring, a random-direction noise floor, a role-matched control, and an
out-of-distribution generalization test. Distinctness from refusal tested by cosine
similarity against a separately-built, held-out-validated refusal direction.

## Tooling

Built directly on **HuggingFace Transformers + PyTorch forward hooks**, not
TransformerLens: Qwen3.5-9B is a newer hybrid architecture with uncertain TL support,
and I only needed residual-stream reads and forward-hook ablation, so going direct
avoided a dependency I couldn't fully verify against this checkpoint.

## Limitations

Small samples (n=5 behavioural, n=10 mechanistic, one scenario for the direction),
after a Colab VM crash cost a larger run. Layer selected and validated on the same
pool. Retraction was only tested behaviourally, never read mechanistically. The
direction is shown to be distinct from refusal, but not shown to be specifically about
user intent versus general topic-sensitivity. See the write-up's Limitations section
for the full list.

---

*Full write-up and figures are in this repo. This was a self-directed learning
project; feedback welcome.*
