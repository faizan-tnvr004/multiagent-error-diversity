# Pre-Registered Analysis Plan

**Study:** When does multi-agent LLM collaboration help, and when does it hurt? Error diversity across models as a predictor of multi-agent benefit.

This document is the pre-registered analysis plan. It is committed before any experiments are run; the commit date is the pre-registration timestamp. Sections 2–5 are fixed in advance. Section 6 and any other observation are exploratory and labelled as such.

## 1. Summary

This is an empirical study testing whether the *diversity of errors that different language models make on a task* predicts whether having those models collaborate via debate improves or worsens accuracy on that task. The study uses three small open models on approximately 90 programmatically generated, automatically graded tasks.

## 2. Hypothesis

**Intuition:** When multiple models attempt the same task, their collaboration is only helpful if models fail in different ways i.e. a model that is wrong is corrected by another one that is right, or wrong in a different manner. When models happen to be confidently wrong in the *same* way, collaboration has no basis to correct against and instead reinforces the shared error.

**H1.** Cross-model error diversity D_across(t) is positively correlated with multi-agent benefit Δ(t): tasks on which the models produce more varied errors show greater accuracy gains from collaboration than tasks on which the models make correlated errors.

**H2.** This relationship holds *after controlling for task difficulty* d(t) i.e. D_across(t) predicts Δ(t) beyond what difficulty alone explains. A relationship that disappears once difficulty is controlled does **not** support the hypothesis.

## 3. Metric Definitions

**Attempts and the A/B split**

For each task, each of the three models is sampled <u>10 times</u> at nonzero temperature (independent single-agent attempts). These samples are split <u>disjointly per task</u> into **set A (6 samples/model)** and **set B (4 samples/model).**

- Set A is used to compute D_across(t) and d(t) (the predictor and the control).
- Set B is used only for the self-consistency baseline inside Δ(t) (the outcome).

The disjoint split exists to sever the predictor from the outcome: the quantity that predicts benefit and the baseline that defines benefit are computed from different samples, so any correlation is not an artifact of shared data.

**"Same wrong answer" (per-family fingerprint)**

Two wrong attempts count as the *same* error if their fingerprints match:

- **Math word problems / Data questions:** identical final number.
- **Code debugging:** identical failing-test set.
- **Constraint puzzles:** identical violated constraint set.

The code-debugging definition is the loosest (two attempts can fail the same tests with different broken code); this is acknowledged as a limitation (§8).

**D_across(t) — cross-model error diversity**

With three models there are three model-pairs: (A,B), (A,C), (B,C). Over the cross-sample pairings in set A where <u>both models in the pair are incorrect</u>, D_across(t) is the **fraction of those both-wrong pairings that produced *different* wrong answers** (by fingerprint).

- High D_across = models fail in different ways = diverse errors (debate should help).
- Low D_across = models share the same wrong answer = correlated errors (debate should hurt).

Disagreement is measured only among <u>both-wrong</u> pairings; pairings where one model is correct are not informative about error diversity and are excluded.

**d(t) — difficulty**

d(t) = pooled single-agent error rate across all three models' set-A samples.

**Δ(t) — multi-agent benefit**

Δ(t) = heterogeneous-debate accuracy on the task − self-consistency baseline accuracy. The self-consistency baseline (single model, majority vote over set-B samples) uses a sample count **matched to the total model-call budget of the debate condition**, so that Δ measures the benefit of *collaboration*, not merely of additional compute. Heterogeneous debate is the headline condition.

**Inclusion filter (pre-registered)**

A task enters the <u>primary analysis</u> only if it has **≥10 both-wrong pair-observations** in set A. Tasks below this threshold (typically very easy tasks, where models are rarely wrong) are excluded. This rule is fixed in advance and applied automatically. The number of tasks passing the filter is reported.

## 4. Experimental Conditions

**Models:** Three small open models (e.g. a Gemini Flash model via Google AI Studio, and two open models such as Llama-3-8B and Mistral-7B via Groq). Exact model IDs/versions are recorded in the repository; the strongest small open models available at coding time are used. Three is the minimum for D_across to be meaningful.

**Tasks:** Approximately 90 programmatically generated tasks across four families: Math word problems (~25), code debugging (~25), constraint puzzles (~20), data questions (~20). All graded automatically with no human judgment in the scoring loop. Task difficulty is calibrated before the full run so that most tasks fall in the 20–70% single-agent-error band.

**Conditions:** Single-agent (baseline sampling); homogeneous debate (A+A); heterogeneous debate (A+B, headline); pipeline (Plan→Execute→Review, heterogeneous); self-consistency (single model, majority vote, matched sample count) as the maximum-error-correlation control.

## 5. Primary Analysis

The single pre-committed test:

**Partial Spearman rank correlation between D_across(t) and Δ(t), controlling for d(t),** over all tasks passing the inclusion filter.

Predicted direction: **positive**. The hypothesis lives or dies on this test (H2). A plain (uncontrolled) Spearman correlation between D_across(t) and Δ(t) is reported alongside it as a reference (H1).

## 6. Secondary Analyses (exploratory)

The following are exploratory and clearly labelled as such; they are not confirmatory:

- Linear/rank regression of Δ(t) on D_across(t) and d(t) jointly.
- Binned analysis: tasks grouped into low/medium/high D_across buckets, mean Δ per bucket.
- **Within-model diversity vs. self-consistency benefit:** whether within-model resampling diversity predicts self-consistency gain (the companion question to the cross-model headline). Treated as a secondary, not primary, result.
- Per-family breakdown of the D_across–Δ relationship.
- Whether the relationship holds within individual model pairs, not only pooled.
- Qualitative inspection of 5–10 transcripts where debate amplified a shared error.

## 7. Pre-Committed Outcomes

- **Hypothesis supported:** the partial Spearman correlation (D_across vs Δ, controlling for d) is positive and statistically meaningful.
- **Null result:** no correlation, or a correlation that vanishes once difficulty is controlled. A within/no-control correlation that disappears under the difficulty control is reported as <u>not supporting</u> the hypothesis.
- **Commitment:** a null or mixed result will be reported honestly and in full. The paper's contribution is the method and the analysis, not a guaranteed positive finding. Inclusion thresholds and the primary test are fixed here, before data collection, and will not be changed after seeing results.

## 8. Limitations (stated upfront)

- **Synthetic tasks:** all tasks are programmatically generated and may not represent the difficulty structure of real-world problems.
- **Three small open models only:** D_across measures error independence among these specific three models, not a universal task property. Results may not generalize to larger models or other model sets.
- **Small sample:** ~50 tasks after filtering is enough to detect a strong-to-moderate effect but underpowered for a weak one; a non-significant result may be a false negative.
- **Code-family fingerprint:** "same failing-test set" is a coarse proxy for "same error."
- **Single language/domain scope.**

## 9. Fixed vs. Exploratory

Sections 2–5 (hypothesis, metric definitions, conditions, primary analysis) and the inclusion filter in §3 are **fixed before any experiments are run**. Section 6 and any post-hoc observation are exploratory and labelled as such throughout the paper. This separation is the safeguard against selecting analyses to fit the data after seeing it.
