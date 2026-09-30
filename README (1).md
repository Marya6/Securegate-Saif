# SecureGate: Mitigating Gate-Fingerprint Leakage in Fault-Tolerant Quantum Computing

SecureGate is a simulation-based proof-of-concept project that investigates
gate-dependent information leakage from spacetime syndrome data in
fault-tolerant quantum computing.

The project studies whether an untrusted decoder can infer a hidden logical
operation from detector records and evaluates randomized circuit-level
virtual Pauli errors as a mitigation strategy.

---

## Project Overview

Fault-tolerant quantum computers use decoders to process syndrome data and
recover the logical state in the presence of noise. However, detector records
can contain gate-dependent statistical fingerprints that may expose information
about the underlying logical computation.

SecureGate develops an experimental workflow to:

1. Construct fault-tolerant logical-gate experiments.
2. Characterize gate-dependent detector fingerprints.
3. Evaluate gate inference from complete detector records.
4. Introduce randomized virtual Pauli errors at the circuit level.
5. Measure the effect on gate inference while monitoring decoder performance.

The current implementation is a controlled simulation-based proof of concept.
It does not claim to provide a complete fault-tolerant privacy protocol or
universal security guarantees.

---

## Key Idea

```text
Hidden Logical Operation
        ↓
Fault-Tolerant Surface-Code Circuit
        ↓
Spacetime Syndrome / Detector Data
        ↓
Untrusted Decoder
        ↓
Gate Inference
        ↓
Circuit-Level Virtual Pauli Randomization
        ↓
Re-evaluation of Gate Inference and Decoder Performance
```

The mitigation stage introduces randomized virtual Pauli errors
`X`, `Y`, or `Z` at the circuit level while tracking the corresponding
Pauli frame.

---

## Experimental Setup

The main gate-fingerprint and mitigation experiments use:

- Distance-5 unrotated surface-code construction
- 81 physical qubits
  - 41 data qubits
  - 40 ancilla qubits
- Logical operations: `H_L` and `S_L`
- 100 detector records per shot
- Physical error probability: `p = 10^-3`
- 10,000 shots per logical gate per run
- 10 independent evaluation runs
- 41 data qubits × 3 Pauli types = 123 circuit-level cases

The `H_L` and `S_L` experiments use the same detector indexing and spacetime
frame, enabling detector-by-detector comparison.

---

## Gate-Fingerprint Inference

Gate-dependent detector structure is evaluated using a Bernoulli Naive Bayes
classifier as a baseline inference model.

The classifier uses the complete detector record as a binary feature vector and
predicts whether the hidden logical operation is `H_L` or `S_L`.

Classification accuracy is used as an empirical indicator of the gate-dependent
statistical information available in the detector records. It is not
interpreted as a percentage of information leakage and does not represent
reliable practical reconstruction of the logical gate.

The fingerprint experiment established a measurable and reproducible, but
modest, gate-dependent signal in the simulated `H_L` / `S_L` detector data.

---

## Circuit-Level Mitigation

SecureGate evaluates randomized virtual Pauli errors at the circuit level.

For the mitigation experiment:

- A data qubit is selected from the 41 data qubits.
- A Pauli operator `X`, `Y`, or `Z` is selected.
- The virtual Pauli error is inserted into the circuit.
- The Pauli frame is propagated through the circuit.
- The modified circuit is resampled to generate detector records.
- Gate inference and decoder performance are evaluated.

This gives:

```text
41 data qubits × 3 Pauli operators = 123 cases
```

The decoder itself is left unchanged during the experiment.

The Pauli-frame component follows the established principle of tracking Pauli
corrections classically through the computation, while SecureGate investigates
its use as an experimental mitigation strategy for decoder-side
gate-fingerprint leakage.

---

## Results

The reported values below are averages over 10 independent runs.

| Metric | Baseline | Mitigation | Change |
|---|---:|---:|---:|
| Gate-inference accuracy | 53.810% | 53.408% | -0.403 pp |
| Logical error rate (LER) | 0.579% | 0.577% | -0.002 pp |
| Decoder utility | 99.421% | 99.423% | +0.002 pp |

### Reproducibility

Gate-inference accuracy across 10 independent runs:

- Baseline: `53.810% ± 0.950 pp`
- Mitigation: `53.408% ± 0.629 pp`

The observed change in gate inference is small. The results are therefore
interpreted as a proof-of-concept evaluation rather than a conclusive
demonstration of strong leakage suppression.

---

The notebooks document the project workflow from backend validation and
state-preparation fingerprint analysis through logical-gate fingerprinting,
gate inference, and circuit-level mitigation.

---

## Notebook Workflow

### Notebook 01 — Backend Validation

Validates the simulation environment and establishes a detector-analysis
baseline using logical-state preparation.

### Notebook 02 — Gate Interface

Defines and validates the logical-gate interface, audits the installed
surface-code backend, and establishes an initial gate-fingerprint baseline.

### Notebook 03 — Gate Fingerprints

Constructs fault-tolerant `H_L` and `S_L` experiments, verifies the common
100-detector spacetime frame, characterizes detector fingerprints, and evaluates
gate inference using Bernoulli Naive Bayes.

### Notebook 04 — Circuit-Level Mitigation

Implements randomized circuit-level virtual Pauli error injection with
Pauli-frame propagation and evaluates the effect on gate inference,
logical error rate, and decoder utility.

---

## Limitations

The current implementation is simulation-based and has the following scope:

- Two logical gates: `H_L` and `S_L`
- A specific `surface-sim` implementation and noise model
- Bernoulli Naive Bayes as the inference baseline
- Simulation-only evaluation
- No validation on real quantum hardware

The present results should not be interpreted as universal security guarantees,
complete elimination of gate-dependent leakage, or reliable practical
single-shot gate identification.

---

## Future Work

Future work will investigate:

- Stronger and more diverse gate fingerprints
- Different physical noise settings
- Additional inference and machine-learning methods
- Larger surface-code distances and circuit sizes
- Larger logical-gate sets
- Validation on real quantum hardware

---

## References

1. Shukla, S., Browne, D. E., & Nishio, S. (2026).
   *Anticipating decoder side-channel attacks in fault-tolerant quantum computers.*
   arXiv:2607.12174.

2. Fowler, A. G., Mariantoni, M., Martinis, J. M., & Cleland, A. N. (2012).
   *Surface codes: Towards practical large-scale quantum computation.*
   Physical Review A, 86, 032324.

3. Reichardt, B. W. (2006).
   *Error-detection-based quantum fault tolerance against discrete Pauli noise.*
   UCB/EECS-2006-157.

---

## Project Resources

The repository contains the project notebooks, source code, figures, and
reproducibility materials. The accompanying technical documentation and
supporting reports are provided through the project's Google Drive archive.

---

## Author

**Marya Alturki**

**SecureGate — SAIF 2026**
