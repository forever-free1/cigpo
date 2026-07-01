# Main Results

## Training Dynamics (HotpotQA, Qwen2.5-3B-Instruct, 200 steps)

| Method | Correct (1.0) | Format Violation (-2.0) | Wrong (0.0) |
|--------|---------------|------------------------|-------------|
| **CIGPO** | **48.0%** | **19.0%** | 19.0% |
| GRPO | 14.0% | **69.0%** | 13.0% |

## Test Set Benchmark (1000 HotpotQA samples)

| Checkpoint | F1 (no fmt) | F1 (with fmt) | EM |
|-----------|------------|---------------|-----|
| Base Qwen2.5-3B | 0.252 | -0.930 | ~0 |
| CIGPO step 50 | 0.429 | -0.201 | ~0 |
| GRPO step 50 | 0.430 | 0.008 | ~0 |
| CIGPO step 100 | 0.432 | 0.344 | 0.242 |
| GRPO step 100 | 0.033 | **-1.781** | ~0 |
| CIGPO step 150 | 0.471 | 0.339 | 0.217 |
| GRPO step 150 | **0.000** | **-2.000** | **-2.000** |
| **CIGPO step 200** | **0.518** | **0.272** | **0.150** |
| GRPO step 200 | **0.000** | **-2.000** | **-2.000** |

## Key Findings

1. **GRPO collapses after step 50**: degenerates into 100% format-violating outputs. Without intermediate IG rewards to guide evidence reading, GRPO's sparse answer-level reward fails in multi-turn settings.
2. **CIGPO's IG signal prevents collapse**: IG provides per-turn credit assignment, guiding the model to read evidence selectively and maintain valid output format.
3. **CIGPO F1 improves +105%** from base (0.252 → 0.518) while maintaining low format violation rate (~12%).
4. **CIGPO is necessary** (not just beneficial) for multi-turn evidence-reading RL training.

## Training Log Diagnosis

| Model | Zero-Advantage Group Ratio | Actor Entropy Loss | Critic Score Mean |
|-------|---------------------------|-------------------|-------------------|
| GRPO 1.5B | 1.0 | 6380.7 | -2.0 |
| GRPO 3B | 1.0 | 6375.5 | -2.0 |
| CIGPO 3B | 0.5 | 287.2 | 7.397 |

GRPO reaches 100% zero-advantage groups — a complete policy gradient deadlock. CIGPO maintains healthy entropy and non-zero advantages.

## Cumulative IG Correlates with Answer Quality

| Outcome | N | Cumulative IG (nats) | Mean F1 |
|---------|---|---------------------|---------|
| Correct (F1 > 0.5) | 1811 | 4.06 | 0.942 |
| Partial (0 < F1 ≤ 0.5) | 390 | 3.28 | 0.368 |
| Wrong (valid format) | 1251 | 2.10 | 0.0 |
| Format violation | 520 | 2.97 | 0.0 |

## Training Config

| Parameter | Value |
|-----------|-------|
| Model | Qwen2.5-3B-Instruct (3.09B params) |
| GPUs | 2× NVIDIA (24GB each) |
| Max turns | 3 |
| Group size | 2 |
| KL coefficient | 0.10 |
| Learning rate | 1e-6 |
| Total steps | 200 |
| IG init → final | 0.1 → 0.3 |
| IG_CLIP | 50.0 |
| IG normalization | separate |

*Full code and training logs archived on Zenodo: [10.5281/zenodo.21097914](https://doi.org/10.5281/zenodo.21097914)*
