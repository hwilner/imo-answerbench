# imo-answerbench

Calibrated benchmark construction for competitive mathematical derivation, with dual-boundary solver difficulty calibration and answer-verifiable evaluation items.

## Introduction

Evaluating multi-step mathematical reasoning is bottlenecked by two failure modes: items whose difficulty is misjudged relative to actual solver populations, and evaluation protocols that conflate lucky final answers with valid derivations. **imo-answerbench** addresses both by constructing competitive-mathematics benchmark items with *calibrated difficulty boundaries* and *machine-verifiable final answers*, enabling trustworthy measurement of long-horizon reasoning in generative sequence models.

The benchmark is designed as the evaluation endpoint of the [Controlled Plastic Linear Models](https://github.com/hwilner/controlled-plastic-linear-models) research programme: test-time-adaptive and stability-constrained architectures are only as convincing as the rigor of the reasoning benchmark they are evaluated on.

## Extended Introduction

Modern mathematical evaluation environments estimate item difficulty from solver behavior, but a single pass-rate statistic is fragile: a solver population shift (new model family, new decoding budget) invalidates the calibration. imo-answerbench instead applies a **dual-boundary calibration** methodology: each item carries two empirically estimated thresholds — a *lower boundary* (fraction of a weak solver ensemble that solves it, detecting items too easy to discriminate) and an *upper boundary* (fraction of a strong solver ensemble that fails it, detecting items too hard to attribute signal). Items are retained in difficulty strata only when both boundaries fall inside preregistered bands, and the disagreement $\mathcal{H}$ between the two solver ensembles serves as a continuous item-quality score.

This same disagreement signal is consumed downstream by [discrete-diffusion-adaptation](https://github.com/hwilner/discrete-diffusion-adaptation) as an entropy-regulated test-time guidance penalty, and as a curriculum-trajectory synthesizer for training discrete world models: items are sequenced so that each curriculum step stays within the calibrated band where guidance signal is maximally informative.

## Methods

### Item schema

Each benchmark item comprises:

| Field | Description |
|---|---|
| `problem` | Competition-style problem statement (LaTeX) |
| `final_answer` | Machine-verifiable answer (integer, fraction, or closed form) |
| `derivation_rubric` | Checkpoints a valid derivation must pass through |
| `difficulty_band` | Dual-boundary stratum assignment (B1–B5) |
| `solver_disagreement` | $\mathcal{H}$ between weak/strong solver ensembles |

### Dual-boundary calibration protocol

1. **Solver ensembles:** a weak ensemble (small open-weight models, greedy decoding) and a strong ensemble (frontier models, high decoding budget) attempt each candidate item under matched, preregistered prompting.
2. **Boundary estimation:** per-item solve rates $p_{\text{weak}}$, $p_{\text{strong}}$ are estimated over $n \geq 8$ independent samples per solver with Wilson score confidence intervals.
3. **Band assignment:** an item enters band $B_k$ only if $p_{\text{weak}}$ and $p_{\text{strong}}$ lie within the preregistered interval for $B_k$; otherwise it is revised or discarded.
4. **Answer verification:** final answers are checked symbolically; derivation validity is scored against the rubric checkpoints.

### Curriculum synthesis

For training/evaluation of discrete generative world models, calibrated items are ordered into trajectories that maximize expected information gain per step, subject to remaining inside the band where verifier disagreement is reliable.

## Results

| Stratum | Items | $p_{\text{weak}}$ (target) | $p_{\text{strong}}$ (target) | Status |
|---|---|---|---|---|
| B1 (entry) | TBD | 0.5–0.9 | ≥ 0.95 | calibration pending |
| B3 (discriminative) | TBD | 0.05–0.3 | 0.5–0.9 | calibration pending |
| B5 (frontier) | TBD | < 0.05 | 0.05–0.4 | calibration pending |

*Results tables will be populated with captioned calibration statistics as the solver-ensemble runs complete; see the issue tracker for atomic subtasks.*

## Repository structure

```
imo-answerbench/
├── data/            # Calibrated items (JSONL) — pending first calibration run
├── calibration/     # Dual-boundary estimation pipeline
├── verification/    # Symbolic answer checking and rubric scoring
└── curriculum/      # Curriculum trajectory synthesis
```

## License

MIT. See [LICENSE](LICENSE).
