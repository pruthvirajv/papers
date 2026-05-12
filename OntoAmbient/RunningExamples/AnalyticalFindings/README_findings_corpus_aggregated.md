# findings_corpus_aggregated.json



## Purpose

This file contains the **10 aggregated analytical findings** that replace individual claim-level outliers as the input to the AmbientOn hypothesis generation pipeline. Each finding represents a population-level anomaly observation of the kind a business analyst would encounter on a BI dashboard — not a single suspicious claim, but a pattern spanning hundreds of providers or thousands of claims.

**Why aggregated findings instead of individual outliers?**  
Record-level outliers (e.g., "Claim CLM123456 has reimbursement $15,000 above the IQR threshold") can be explained by a simple SQL query. Aggregated findings (e.g., "State 49 providers show 2.1× the national fraud rate across 107 providers") require multi-hop ontology traversal across Provider → GeographicLocation → Claim → Beneficiary entity types. Only aggregated findings benefit from AmbientOn's multi-source, multi-hop reasoning.

---

## File Format

JSON array. Each element is one finding object.

```json
[
  { /* Finding F01 */ },
  { /* Finding F02 */ },
  ...
  { /* Finding F10 */ }
]
```

**Total records:** 10 findings (F01–F10)

---

## Finding Object Schema

| Field | Type | Description |
|-------|------|-------------|
| `FindingID` | string | Unique identifier: `F01`–`F10` |
| `FindingType` | string | Camel-case category name (see Finding Types below) |
| `AggregationLevel` | string | Granularity at which the anomaly was computed: `State`, `Quarter`, `Provider`, `PhysicianCount`, `AdmissionDuration`, `ReimbursementPercentile`, `DiagnosisCode`, `ClaimDuration` |
| `EntityType` | string | Primary ontology node type: `GeographicLocation`, `TimeWindow`, `Physician`, `Claim`, `Provider` |
| `FindingDescription` | string | Full plain-English description of the observed anomaly, with specific numbers. This is the text passed to the Orchestrator and GoT Generator. |
| `ObservedValue` | float | The computed metric value for the anomalous entity/group |
| `BaselineValue` | float | The population baseline this value is compared against |
| `DeviationRatio` | float | `ObservedValue / BaselineValue` — how many times above baseline the anomaly is |
| `AffectedEntities` | list[string] | Specific entity identifiers (e.g., `["State_49"]`, `["PRV51003"]`) |
| `AffectedCount` | int | Number of entities involved in the anomaly (providers, claims, physicians, etc.) |
| `AnalyticalMetric` | string | The metric being measured (e.g., `FraudProviderRate`, `AvgReimbursementPerClaim`) |
| `ComparisonBaseline` | string | Human-readable description of what the baseline represents |
| `GroundTruthLabel` | int | `1` = genuine anomaly (supported by PotentialFraud oracle), `0` = not supported |
| `VerificationMethod` | string | `Statistical`, `Multi-hop`, or `Deterministic` — how this finding can be verified |
| `ConfidenceScore` | float | Initial confidence in this finding (0–1), computed from deviation ratio; updated by Phase 2b |
| `DataSources` | list[string] | Simulated data sources needed to investigate this finding (used by Planner agent) |
| `OntologyNodesInvolved` | list[string] | Ontology entity classes traversed during this finding's investigation |
| `RelationPaths` | list[string] | Specific ontology relation paths activated by this finding |

---

## Finding Types (All 10)

| FindingID | FindingType | EntityType | What It Describes |
|-----------|-------------|------------|-------------------|
| F01 | `GeographicFraudConcentration` | GeographicLocation | Which US states have disproportionately high fraud provider rates |
| F02 | `SeasonalFraudSpike` | TimeWindow | Whether any quarter shows elevated claim-level fraud rates |
| F03 | `PhysicianNetworkFraudCorrelation` | Physician | Whether providers with more physicians have higher fraud rates |
| F04 | `ReimbursementBimodality` | Claim | Whether outpatient reimbursements cluster in two extremes |
| F05 | `ChronicConditionMismatch` | Beneficiary | Whether fraud providers bill for patients with fewer chronic conditions |
| F06 | `ShortAdmissionHighReimbursement` | InpatientClaim | Whether 0–1 day hospital stays show inflated reimbursements |
| F07 | `PeerGroupReimbursementDeviation` | Provider | Whether top-5% peer deviants show systematically higher fraud rates |
| F08 | `DiagnosisCodeFraudConcentration` | DiagnosisCode | Whether specific ICD-9 diagnosis codes are concentrated among fraud providers |
| F09 | `ZeroDurationClaimAnomaly` | Claim | Whether zero-duration claims (start = end date) are used for high-value billing |
| F10 | `MultiProviderFraudPatientPattern` | Beneficiary | Whether patients are being used as shared billing vehicles across fraud networks |

---

## Example Record (F07 — PeerGroupReimbursementDeviation)

```json
{
  "FindingID": "F07",
  "FindingType": "PeerGroupReimbursementDeviation",
  "AggregationLevel": "ReimbursementPercentile",
  "EntityType": "Provider",
  "FindingDescription": "Providers in the top 5% of reimbursement vs state peer median bill on average 53.5x their state peers and show a fraud rate of 72.3% — compared to 6.0% for providers billing within peer norms. 271 providers identified. Does extreme peer group deviation systematically predict fraud beyond what absolute reimbursement levels reveal?",
  "ObservedValue": 0.723,
  "BaselineValue": 0.060,
  "DeviationRatio": 12.05,
  "AffectedEntities": ["top_5pct_peer_deviants"],
  "AffectedCount": 271,
  "AnalyticalMetric": "FraudRate_in_top5pct_peer_deviation",
  "ComparisonBaseline": "Fraud rate among providers billing within peer norms",
  "GroundTruthLabel": 1,
  "VerificationMethod": "Statistical",
  "ConfidenceScore": 0.869,
  "DataSources": ["provider_sql", "claims_sql", "analytics_warehouse", "audit_store"],
  "OntologyNodesInvolved": ["Provider", "GeographicLocation", "Claim"],
  "RelationPaths": [
    "(P:Provider)-[operates:operatesIn]-(GL:GeographicLocation)",
    "(P:Provider)-[submits:submitsClaim]-(C:Claim)"
  ]
}
```

---

## DeviationRatio Values (Actual)

| FindingID | FindingType | DeviationRatio | AffectedCount |
|-----------|-------------|----------------|---------------|
| F01 | GeographicFraudConcentration | 2.098 | 107 providers |
| F02 | SeasonalFraudSpike | 1.007 | 144,743 claims |
| F03 | PhysicianNetworkFraudCorrelation | 9.26 | — |
| F05 | ChronicConditionMismatch | — | — |
| F06 | ShortAdmissionHighReimbursement | — | — |
| F07 | PeerGroupReimbursementDeviation | 12.05 | 271 providers |

> **Note:** F02 (SeasonalFraudSpike) has a near-baseline deviation ratio of 1.007 — a strong indicator that this finding will be marked SPURIOUS.

---

## VerificationMethod Values

| Value | Meaning | Findings |
|-------|---------|----------|
| `Statistical` | Verified by statistical test (Chi-square, t-test, Mann-Whitney U, etc.) | F01, F02, F05, F06, F07, F08, F09 |
| `Multi-hop` | Verified by multi-hop ontology traversal across 3+ entity types | F03, F10 |
| `Deterministic` | Verified by exact rule (e.g., claim date after death date) | — (AX1-type findings not in corpus due to data limitation) |

---


