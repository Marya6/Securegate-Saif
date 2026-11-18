# SecureGate: Software-Level Mitigation of Gate-Fingerprint Leakage

**A simulation-based study of gate-identity and gate-order leakage in fault-tolerant quantum circuits, using Pauli-frame-derived detector-record randomization.**

SecureGate investigates gate-fingerprint leakage in fault-tolerant quantum circuits: whether an untrusted classical decoder can use spacetime detector records to infer hidden logical-gate identities or chronological gate order. The project implements a class-independent, software-level randomization channel derived from propagated Pauli frames and evaluates its effect on both attacker inference and logical decoding reliability.

> **Project scope and contribution.** SecureGate brings together two complementary numerical studies on an unrotated distance-5 CSS surface-code patch: logical-gate identity inference (\(H_L\) versus \(S_L\)) and chronological-order inference for matched five-gate circuits. It develops and validates a Pauli-frame-derived detector-mask channel, then quantifies its impact on inference and decoding metrics. The findings provide a simulation-based proof of concept for gate-fingerprint leakage and its software-level mitigation under the circuit, noise, attacker, and decoder settings documented below. Extension to other code distances, broader noise models, and physical hardware is a natural next stage of evaluation.

---

## Research tracks

The repository contains two complementary experimental tracks that examine different forms of gate-fingerprint leakage. They share the same code family and distance, while using study-specific circuits, detector dimensions, inference targets, and decoder paths. Results are therefore interpreted within their respective experimental designs.

### Study I — Logical-gate identity: \(H_L\) vs. \(S_L\)

The principal H/S experiments use an **unrotated distance-5 CSS surface-code layout**:

| Property | Configuration |
|---|---:|
| Data qubits | 41 |
| Ancilla qubits | 40 |
| Physical qubits | **81** |
| Measurements per circuit | 121 |
| Detector features per shot | **100** |
| Logical observables | 1 |
| Encoded operations | \(H_L\), \(S_L\) |

The attacker receives the 100 binary detector features and attempts to distinguish records associated with \(H_L\) and \(S_L\). The evaluated models are:

- Bernoulli Naive Bayes (NB)
- Logistic Regression (LR)
- Histogram-based Gradient Boosting (HGB)

The decoder-utility benchmark is restricted to the **\(S_L\) path**. It uses PyMatching and a frozen CNN that can selectively override a PyMatching prediction when the two models disagree and the CNN confidence meets a validation-selected threshold.

### Study II — Logical gate-order inference

The multi-gate study asks whether detector records reveal the chronological order of logical gates when candidate circuits contain the same gate set and implement the same net logical operation up to global phase.

The three chronological sequences are:

| Class | Sequence |
|---|---|
| A | \(X_L \rightarrow Y_L \rightarrow Z_L \rightarrow H_L \rightarrow S_L\) |
| B | \(H_L \rightarrow S_L \rightarrow X_L \rightarrow Y_L \rightarrow Z_L\) |
| C | \(Y_L \rightarrow Z_L \rightarrow X_L \rightarrow H_L \rightarrow S_L\) |

Each sequence contains one instance of each gate and implements the same net logical operation up to global phase. The classification target is the **ordered gate history**, not the final net logical operation.

Each multi-gate circuit uses the same distance-5 unrotated CSS patch and produces:

| Property | Configuration |
|---|---:|
| Physical qubits | 81 |
| Measurements | 281 |
| Detector features per shot | **260** |
| Logical observables | 1 |
| Sequence classes | A, B, C |
| Physical error parameter | \(p = 10^{-3}\) |

The sequence-inference evaluation uses Bernoulli Naive Bayes and Logistic Regression. The decoder-utility evaluation uses **BP+OSD with a hypergraph-preserving detector error model (DEM)**. This is a separate decoder path from the H/S PyMatching/CNN study.

---

## Mitigation method

SecureGate constructs a library of Pauli-frame-derived detector masks by propagating virtual single-qubit Pauli cases through the implemented circuit model.

For each 41-data-qubit circuit:

\[
41 \text{ data qubits} \times 3 \text{ Pauli types }(X,Y,Z)
= 123 \text{ cases}.
\]

The masks are binary detector-feature vectors. The \(S_L\) path also tracks the corresponding logical-observable frame offsets for trusted-side scoring. These offsets are kept outside the attacker feature set and are used only when evaluating the trusted logical-observable outcome.

The primary mitigation experiment is a **classical transformation of detector records sampled from the original noisy circuits**:

\[
d' = d \oplus m,
\]

where \(d\) is the sampled detector record and \(m\) is the validated detector mask selected for a shot. The randomization selection is independent of the hidden gate or sequence label. **The primary mitigation path does not insert physical Pauli gates into the sampled Stim circuits.**

Direct Pauli injection is used as an independent validation reference for the propagated detector masks and the applicable logical-observable offsets; it is not the primary mitigation mechanism.

### Study-specific settings

- **H/S rate sweep:** \(\alpha \in \{0, 0.1, 0.2, 0.3, 0.4, 0.5\}\).
- **H/S integrated comparison:** \(\alpha = 0.10\), with paired baseline/randomized conditions over 20 repetitions.
- **HGB attacker challenge:** a separate validation-locked 20-repetition confirmation.
- **Multi-gate randomization:** \(\alpha = 0.10\), using a common class-independent mask channel across sequences.

These experiments have different designs and should not be pooled into one estimate.

---

## Main results

The following summaries are taken from the archived experiment tables and reports. All values are specific to the stated simulation configurations.

### H/S attacker evaluation

#### Integrated 20-repetition NB/LR comparison

The integrated study used paired baseline and randomized conditions at \(\alpha = 0.10\), with 20 repetitions and 25,000 shots per gate per repetition. The mean held-out test ROC-AUC values were:

| Attacker | Baseline ROC-AUC | Randomized ROC-AUC | Mean paired change | 95% CI for change | Holm-adjusted \(p\) |
|---|---:|---:|---:|---:|---:|
| Bernoulli Naive Bayes | 56.1742% | 55.7494% | −0.4248 pp | [−0.5540, −0.2956] pp | 0.000019 |
| Logistic Regression | 57.2287% | 56.7489% | −0.4798 pp | [−0.6059, −0.3537] pp | 0.000004 |

Both models showed a statistically supported reduction in ROC-AUC after randomization. All 40 run-condition AUC values (20 repetitions × 2 conditions) remained above 50%, so raw and direction-corrected effective AUC coincide in this study. The result demonstrates that the detector-mask channel can reduce measured gate distinguishability across both linear and probabilistic attackers, while preserving a measurable residual signal.

#### Independent validation-locked HGB challenge

A separate 20-repetition challenge, also using 25,000 shots per gate per repetition, evaluated a nonlinear Histogram-based Gradient Boosting (HGB) attacker, with model settings and decision thresholds locked before the confirmation runs:

| Metric | Baseline | Randomized | Mean paired change | 95% CI for change | Holm-adjusted \(p\) |
|---|---:|---:|---:|---:|---:|
| Test ROC-AUC (primary) | 71.3055% | 70.7833% | −0.5223 pp | [−0.6875, −0.3570] pp | 0.000013 |
| Balanced accuracy (secondary) | 63.5400% | 63.1415% | −0.3985 pp | [−0.5575, −0.2395] pp | 0.000181 |

The independent HGB challenge adds a stronger nonlinear attacker to the NB/LR evaluations. Its baseline test AUC was 71.31%, and randomization reduced that value consistently across the confirmation runs. The randomized AUC remained well above the 50% binary-class chance level (70.78%), showing both a measurable mitigation effect and residual gate-identity information.

#### H/S randomization-rate sweep

A separate 20-repetition rate sweep examined \(\alpha \in \{0, 0.1, 0.2, 0.3, 0.4, 0.5\}\). The summary below shows the endpoints and the \(\alpha=0.10\) operating point:

| \(\alpha\) | NB effective ROC-AUC | LR effective ROC-AUC | Mean LER |
|---:|---:|---:|---:|
| 0.00 | 55.5972% | 56.7324% | 0.0965% |
| 0.10 | 55.1670% | 56.1625% | 0.1195% |
| 0.50 | 54.3019% | 54.7538% | 0.2015% |

Across the six tested rates, increasing \(\alpha\) produced a consistent downward trend in both models' effective ROC-AUC and an upward trend in masked-record LER. The Holm-adjusted trend-test values were 0.000339 (NB AUC), 0.000270 (LR AUC), and 0.000241 (LER). This sweep characterizes the dose-response relationship between detector-record randomization, inference performance, and decoding cost. It is a separate experiment from the integrated 20-repetition comparison above; values from the two studies are not pooled.

### H/S decoding reliability — \(S_L\) path only

| Decoder path | LER |
|---|---:|
| Original PyMatching | 0.0972% |
| Randomized PyMatching | 0.1192% |
| Randomized PyMatching + CNN selective override | 0.1056% |

In the integrated study, randomization increased LER by 0.0220 percentage points relative to original PyMatching (Holm-adjusted paired \(p=0.000435\)). The frozen CNN selective override recovered approximately 61.8% of that added penalty, reducing LER from 0.1192% to 0.1056% (Holm-adjusted paired \(p=0.002044\) versus randomized PyMatching). The hybrid result remained 0.0084 percentage points above the original PyMatching baseline. This evaluates the CNN as a selective correction layer, not as a replacement decoder.

A separate fresh confirmation used 500,000 new \(S_L\) shots and validation-calibrated operating thresholds:

| Decoder path | LER |
|---|---:|
| Randomized PyMatching | 0.1168% |
| CNN alone | 0.6466% |
| Randomized PyMatching + CNN override (2% validation-calibrated target) | 0.1024% |

At the 2% validation-calibrated setting, 126 overrides corrected 99 PyMatching errors and introduced 27, for 72 net errors saved. The paired test was significant after Holm correction (Holm-adjusted \(p=1.12\times10^{-10}\)). The realized override rate was 3.59% of confirmation disagreements; therefore, 2% refers to the validation-calibrated target setting, not an exact realized test quota. These fresh-confirmation results are distinct from the integrated 20-repetition decoder study and are reported separately.

### Multi-gate order inference

With 260 detector features and three sequence classes, the recorded cross-dataset transfer accuracies were:

| Transfer direction | Bernoulli NB | Logistic Regression |
|---|---:|---:|
| Pilot → Confirmatory | 37.5280% | 38.9130% |
| Confirmatory → Pilot | 37.3593% | 38.9863% |
| Three-class chance | 33.3333% | 33.3333% |

Both transfer directions exceeded the 33.33% three-class chance level for both classifiers. The Holm-adjusted permutation value for the full-feature transfer tests was 0.006. The cross-dataset consistency demonstrates a statistically reproducible, above-chance sequence signature under the tested simulation conditions. The reported accuracies provide a baseline for further development of gate-order inference models.

At \(\alpha = 0.10\), the multi-gate randomization comparison found:

- No statistically significant change in balanced accuracy after Holm correction (Holm-adjusted \(p \approx 0.401\)).
- A small statistically significant decrease in macro one-vs-rest ROC-AUC for both models (NB: Holm-adjusted \(p \approx 0.023\); LR: Holm-adjusted \(p \approx 0.019\)).

### Multi-gate decoding reliability — BP+OSD

| Condition | LER |
|---|---:|
| Baseline | 0.3960% |
| Randomized (\(\alpha = 0.10\)) | 0.4257% |

The increase was approximately +0.0297 percentage points, with Holm-adjusted \(p = 0.000175\). The study used 20 paired repetitions and 300,000 decoded shots per condition. The BP+OSD assessment uses the sequence-specific decoder models defined for this experiment. Its LER result belongs to the multi-gate decoder path and is not directly comparable to the H/S \(S_L\) PyMatching/CNN LER values.

---

## Additional validation and inference analyses

### Pauli-frame propagation and mask validation

The propagation workflow derives detector masks from virtual \(X\), \(Y\), and \(Z\) cases on data qubits. The mask library is checked against direct Pauli-injection reference circuits under noiseless and noisy conditions. For the multi-gate study, the saved Phase 5.3 validation table records 123 cases per sequence across all three sequences, with no detector-mask or applicable observable-offset mismatches in the listed checks. This establishes a validated mapping between the propagated Pauli-frame cases and their detector-record transformations for the tested circuits.

### Bounded exact-inference analysis

`SecureGate_Bounded_Exact_ML_Feasibility.ipynb` implements XOR-state dynamic programming to calculate exact detector-pattern probability mass functions (PMFs) from the extracted detector error model (DEM) for selected detector windows up to 20 detector bits. The dynamic-programming implementation was checked against independent enumeration and PMF-normalization tests; the reported reproducibility differences were on the order of \(10^{-15}\) to \(4\times10^{-15}\).

For the 20-detector window, the model-based likelihood discriminator produced:

| Metric | Baseline PMFs | Randomized PMFs (\(\alpha=0.10\)) |
|---|---:|---:|
| Balanced accuracy | 56.6336% | 56.4789% |
| Model-implied ROC-AUC | 61.4378% | 61.0616% |
| Total-variation distance | 0.132672 | 0.129577 |

These are exact likelihood results for the selected detector window under the extracted DEM and its independent-mechanism model. This provides a bounded exact-inference benchmark that complements the fitted NB/LR/HGB attacker evaluations. The full-detector section below separately characterizes the structure and computational cost of the complete H/S detector models; it does not report a 100-detector exact-likelihood result.

### Full 100-detector factor-graph feasibility audit

`SecureGate_Full100_Detector_FactorGraph_Audit..ipynb` analyzes the full H/S detector error models as parity-constrained factor graphs and quantifies the complexity of conventional dense variable elimination for the recorded elimination orderings. The corrected induced widths are 374 for \(H_L\) and 323 for \(S_L\). Under dense float64 storage, the corresponding largest-factor estimates are approximately \(3.078\times10^{113}\) bytes and \(1.367\times10^{98}\) bytes, respectively.

This audit supplies a quantitative structural baseline for the complete detector models and motivates structure-preserving exact-inference approaches that retain the XOR/parity structure. The memory figures are theoretical dense-factor estimates for the audited orderings, not measured RAM use and not a conclusion about every possible exact-inference algorithm.


The repository is research-notebook-led, with saved circuits, datasets, analysis outputs, and supporting implementation modules.

---

## Reproducibility and data

SecureGate was developed through simulation-based experiments with separate evaluation pipelines for gate-identity inference, gate-order inference, and decoding reliability.

This public repository provides the project overview, scientific documentation, and selected aggregate results. The complete research notebooks, raw experimental datasets, model checkpoints, and internal implementation artifacts are not included in this public release.

The reported findings distinguish repeated statistical experiments from illustrative demonstration outputs. Further methodological details are provided in the Public Technical Report.

---

## Findings and study boundary

SecureGate provides a connected set of simulation results and research artifacts:

1. **Gate-identity fingerprinting:** detector records distinguish \(H_L\) from \(S_L\) under NB, LR, and a stronger nonlinear HGB attacker, with held-out and validation-locked evaluations across repeated runs.
2. **Gate-order fingerprinting:** matched five-gate circuits with the same gate multiset and net logical operation produce cross-dataset detector-record classification above the three-class chance level.
3. **A validated software-level mitigation channel:** propagated Pauli-frame cases are converted into binary detector masks and checked against direct-injection reference circuits; the primary mitigation operates by transforming sampled detector records, without inserting physical Pauli gates into the sampled circuits.
4. **A measured leakage–reliability trade-off:** the experiments quantify how randomization changes attacker metrics and logical error rates, while a frozen CNN selective override recovers part of the H/S decoding penalty in the evaluated \(S_L\) path.
5. **Reproducible computational evidence:** the repository includes notebooks, circuit files, detector datasets, mask-validation outputs, model artifacts, paired run-level results, statistical summaries, and recorded environment information.

The findings should be interpreted within each study's design. The H/S inference analysis uses 100 detector features and its \(S_L\) decoder evaluation uses PyMatching with a CNN selective override; the multi-gate analysis uses 260 detector features and BP+OSD. Their LER values therefore describe different decoder configurations and are not direct head-to-head comparisons. For multi-gate order inference, the cross-dataset signal is statistically reproducible but modest in absolute accuracy. For exact inference, the PMFs are exact for the selected detector windows and stated DEM model, while the full-detector factor-graph work is a structural feasibility audit for the dense elimination approach examined.

All reported findings are numerical results from the documented simulation configurations. Generalization to other code distances, circuit families, noise regimes, decoder architectures, and physical hardware remains an important direction for further validation. The project does not claim a formal information-theoretic privacy bound or a cryptographic security guarantee.

## References

1. Shashvat Shukla, Dan E. Browne, and Shin Nishio, “Anticipating Decoder Side-channel Attacks in Fault-tolerant Quantum Computers,” arXiv:2607.12174 (2026).
2. A. G. Fowler, M. Mariantoni, J. M. Martinis, and A. N. Cleland, “Surface codes: Towards practical large-scale quantum computation,” *Physical Review A* 86, 032324 (2012).
3. B. W. Reichardt, “Error-detection-based quantum fault tolerance against discrete Pauli noise,” UCB/EECS-2006-157 (2006).
