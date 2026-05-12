
## Purpose

This file is `findings_corpus_aggregated.json` **enriched with statistical ground truth validation**. Each of the 10 findings from Phase 1b has been subjected to an appropriate statistical test, and each record now carries a `GroundTruthVerdict` — the key gating field that determines whether a finding enters hypothesis generation.

**Why statistical validation before hypothesis generation?**  
With 558,211 claim rows in DS1, even trivially small effects achieve `p < 0.001` due to large sample sizes. A purely p-value based filter would include findings with negligible practical importance (e.g., a 0.003 Cohen's d difference). The dual threshold — both `p < 0.05` AND meaningful effect size — prevents LLM API resources from being spent explaining statistical noise.

---

## File Format

JSON array of 10 objects. Each object is the original Phase 1b finding **plus** 8 additional fields from statistical validation.

```json
[
  { /* All original Phase 1b fields */ + /* 8 new GT fields */ },
  ...
]
```

---

## New Fields Added by Phase 2b

These fields are appended to every finding record:

| Field | Type | Description |
|-------|------|-------------|
| `GroundTruthVerdict` | string | **`CONFIRMED`**, **`WEAK`**, or **`SPURIOUS`** — the gating verdict for Phase 4 |
| `StatisticalTest` | string | Name of the test applied (see table below) |
| `PValue` | float | Probability the pattern occurred by chance. `< 0.05` = statistically significant |
| `EffectSize` | float | Magnitude of the effect (Cohen's d, rank-biserial r, Cramér's V, Cohen's h, or log odds ratio) |
| `EffectSizeName` | string | Name of the effect size metric used (e.g., `"Cohen's d"`, `"rank-biserial r"`) |
| `EffectLabel` | string | Interpretation: `negligible`, `small`, `medium`, or `large` |
| `StatisticallySignificant` | bool | `true` if `p < 0.05`, else `false` |
| `GTInterpretation` | string | Plain-English interpretation of the statistical result |
| `GroundTruth` | dict | Full raw test result object (see sub-schema below) |

The `ConfidenceScore` field from Phase 1b is also **updated** based on the verdict:
- `CONFIRMED` → `ConfidenceScore = max(original, 0.75)`
- `WEAK` → `ConfidenceScore = min(original, 0.65)`
- `SPURIOUS` → `ConfidenceScore = min(original, 0.40)` and `GroundTruthLabel = 0`

---

## `GroundTruth` Sub-Object Schema

```json
"GroundTruth": {
  "test_name":      "Name of statistical test run",
  "p_value":        0.0000,
  "effect_size":    -0.663,
  "effect_size_name": "rank-biserial r",
  "effect_label":   "large",
  "significant":    true,
  "verdict":        "CONFIRMED",
  "interpretation": "Plain-English summary of what the test found"
}
```

---

## Verdict Assignment Rules

| Verdict | Criteria | Action |
|---------|----------|--------|
| **CONFIRMED** | `p < 0.05` AND `\|effect\| ≥ 0.50` (medium or large) | Include in Phase 4; hypothesis generation proceeds |
| **WEAK** | `p < 0.05` AND `0.10 ≤ \|effect\| < 0.50` (small) | Include in Phase 4; hypotheses may be REFINED rather than VALIDATED |
| **SPURIOUS** | `p ≥ 0.05` OR negligible effect | **Exclude from Phase 4**; reported as null result in paper |

> **Note:** F09 (ZeroDurationClaimAnomaly) and F05 (ChronicConditionMismatch) have small but significant effects with **negative sign** — the effect direction is opposite to the domain-intuitive expectation. These are intentionally retained as WEAK findings because CoVe will REFUTE the intuitive hypotheses, demonstrating the system's self-correction capability.

---

## Statistical Tests Per Finding

| FindingID | FindingType | Test Applied | Rationale |
|-----------|-------------|-------------|-----------|
| F01 | GeographicFraudConcentration | Chi-square + Cohen's h | Comparing two proportions (state fraud rate vs national rate) |
| F02 | SeasonalFraudSpike | One-way ANOVA + eta-squared | Comparing means across 4 independent groups (quarters) |
| F03 | PhysicianNetworkFraudCorrelation | Welch t-test + Cohen's d | Two-group comparison, unequal variance assumed |
| F04 | ReimbursementBimodality | Welch t-test + Cohen's d | Two-cluster fraud rate comparison |
| F05 | ChronicConditionMismatch | Welch t-test + Cohen's d | Continuous variable (ChronicConditionCount) across fraud/non-fraud groups |
| F06 | ShortAdmissionHighReimbursement | Welch t-test + Cohen's d | Fraud rates in short-stay vs longer-stay claim groups |
| F07 | PeerGroupReimbursementDeviation | Mann-Whitney U + rank-biserial r | Non-parametric: binary outcome (FraudLabel), skewed reimbursement distribution |
| F08 | DiagnosisCodeFraudConcentration | Chi-square + Cramér's V | Categorical fraud concentration across diagnosis code groups |
| F09 | ZeroDurationClaimAnomaly | Welch t-test + Cohen's d | Fraud rates in zero-duration vs non-zero-duration claims |
| F10 | MultiProviderFraudPatientPattern | Fisher's exact + log odds ratio | Small cell counts in 2×2 contingency table |

---

## Complete Validation Results (Actual Values)

| ID | Type | Test | p-value | Effect | EffectLabel | Verdict |
|----|------|------|---------|--------|-------------|---------|
| F01 | GeographicFraudConcentration | Chi-sq + Cohen's h | 0.0247 | h = 0.303 | small | **WEAK** |
| F02 | SeasonalFraudSpike | One-way ANOVA | 0.5290 | η² = 0.000 | negligible | **SPURIOUS** |
| F03 | PhysicianNetworkFraudCorrelation | Welch t-test | < 0.001 | d = 0.496 | small | **WEAK** |
| F04 | ReimbursementBimodality | Welch t-test | 0.5771 | d = −0.003 | negligible | **SPURIOUS** |
| F05 | ChronicConditionMismatch | Welch t-test | < 0.001 | d = −0.030 | negligible | **WEAK** |
| F06 | ShortAdmissionHighReimbursement | Welch t-test | 0.0281 | d = 0.033 | negligible | **WEAK** |
| **F07** | **PeerGroupReimbursementDeviation** | **Mann-Whitney U** | **< 0.001** | **r = −0.663** | **large** | **CONFIRMED** |
| F08 | DiagnosisCodeFraudConcentration | Chi-square | < 0.001 | V = 0.015 | negligible | **WEAK** |
| F09 | ZeroDurationClaimAnomaly | Welch t-test | < 0.001 | d = −0.171 | negligible | **WEAK** |
| F10 | MultiProviderFraudPatientPattern | Fisher's exact | 1.0000 | OR = 0.492 | medium | **SPURIOUS** |

**Summary:** 1 CONFIRMED, 6 WEAK, 3 SPURIOUS → **7 usable findings** proceed to Phase 4.

---

## Noteworthy Results

**F07 is the only CONFIRMED finding.** With Mann-Whitney rank-biserial r = −0.663 (large effect), the peer group deviation finding is the strongest statistical signal in DS1. The negative sign indicates that higher peer deviation is strongly associated with the fraud label (one-sided test: greater deviation → greater fraud probability). This finding generates the highest-quality hypotheses in Phase 4.

**F05 and F09 have reversed effect direction.** Domain intuition suggests fraud providers bill for patients with fewer chronic conditions (F05) and use zero-duration claims for phantom billing (F09). The actual data shows the opposite: fraud providers' patients have slightly *more* chronic conditions (d = −0.030), and zero-duration claims have a *lower* fraud rate (d = −0.171). These null results are valuable: AmbientOn generates domain-intuitive hypotheses in Phase 4, and CoVe correctly REFUTES them in Phase 5, demonstrating the system's ability to override expert intuition with evidence.

**F10 has an odds ratio < 1.** The multi-provider patient pattern shows the opposite of the expected direction (OR = 0.492), meaning patients appearing across multiple providers are actually *less* associated with fraud providers. Fisher's exact p = 1.000. Classic SPURIOUS result.

---


