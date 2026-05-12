# DS1 Healthcare Provider Fraud Detection Ontology

**File:** `ds1_healthcare_fraud_ontology.json`  
**Version:** 2.0 | **Domain:** Healthcare Insurance Fraud | **Dataset:** DS1  
**Source Dataset:** [Kaggle — Healthcare Provider Fraud Detection Analysis](https://www.kaggle.com/datasets/rohitrox/healthcare-provider-fraud-detection-analysis)

---

## Overview

This ontology is the **single source of truth** for the AmbientOn framework's DS1 experiment pipeline. It simultaneously encodes:

- **Entity schema** — what data entities exist and what attributes they carry
- **Relation paths** — how entities connect across multi-hop traversal routes
- **Formal axioms** — domain rules for fraud detection and verification
- **Datasource routing** — which tool function to call for each entity type at runtime

The unified design means the Planner agent reads `datasource.tool_name` directly from each entity class definition. Changing this file changes routing and reasoning behaviour **without any code modification**.

---

## Source Files

| File | Contents |
|------|----------|
| `Train-1542865627584.csv` | Provider fraud labels — `PotentialFraud` ground truth (5,410 providers) |
| `Train_Beneficiarydata-1542865627584.csv` | Patient demographics + 11 chronic condition flags (138,556 patients) |
| `Train_Inpatientdata-1542865627584.csv` | Hospital admission claims (40,474 records) |
| `Train_Outpatientdata-1542865627584.csv` | Outpatient visit claims (517,737 records) |

After merging on `BeneID` and `Provider`, the unified dataset contains **558,211 rows across 68 columns**.

---

## JSON Top-Level Structure

```
ds1_healthcare_fraud_ontology.json
├── ontology_name          — display name
├── domain                 — "Healthcare Insurance Fraud"
├── version                — "2.0"
├── dataset                — "DS1"
├── source                 — Kaggle URL
├── source_files           — list of 4 raw CSV files
├── description            — plain-English summary of coverage
├── nodes                  — 10 entity class definitions (schema + datasource binding)
├── relationtrees          — 12 directed traversal paths + 5 multi-hop examples
├── axioms                 — 5 formal fraud detection rules (AX1–AX5)
├── ground_truth_strategy  — how verification oracles are used
└── hypothesis_examples    — 5 few-shot examples for the GoT generation prompt
```

---

## Entity Classes (`nodes`)

Each entity class contains a **`datasource` block** (routing) followed by **field definitions**.

### Datasource Block Format

```json
"datasource": {
  "tool_name":   "function name called by the Planner agent",
  "source_name": "logical source name (maps to simulated database)",
  "simulates":   "production database technology being simulated",
  "key_fields":  ["primary key columns for this entity"],
  "join_keys":   {"RelatedEntity": "join column name"},
  "description": "what this source contains"
}
```

---

### 1. `Provider`
**Datasource:** `query_provider_db` → `provider_sql` (simulates SQLite)  
**Key:** `Provider`  
**Joins to:** `Claim` on `Provider`

The central entity for fraud detection. `PotentialFraud` is the **primary ground truth oracle** — the binary label used for all validation accuracy calculations.

| Field | Type | Description |
|-------|------|-------------|
| `ProviderID` | Text | Unique provider identifier |
| `PotentialFraud` | Boolean | **Ground truth** — Yes = fraud-flagged by Medicare |
| `TotalClaimsCount` | Numeric (derived) | Total claims submitted; high counts relative to peers = fraud signal |
| `TotalReimbursementAmt` | Numeric (derived) | Total USD reimbursed; strongest single fraud indicator |
| `TotalDeductibleAmt` | Numeric (derived) | Low deductibles + high reimbursements = inflated billing |
| `UniquePatientCount` | Numeric (derived) | Distinct patients treated; abnormally high = fraud indicator |
| `UniquePhysicianCount` | Numeric (derived) | Distinct attending physicians; many physicians = ghost billing network signal |
| `AvgClaimDuration` | Numeric (derived) | Average claim duration in days; short duration + high reimbursement = upcoding |
| `InpatientClaimsCount` | Numeric (derived) | Count of inpatient (hospital) claims |
| `OutpatientClaimsCount` | Numeric (derived) | Count of outpatient (visit) claims |
| `FraudRiskScore` | Numeric (derived) | Composite 0–1 risk score from reimbursement rank + claim count rank |

---

### 2. `Beneficiary`
**Datasource:** `query_beneficiary_db` → `beneficiary_sql` (simulates MySQL)  
**Key:** `BeneID`  
**Joins to:** `Claim` on `BeneID`

Patient demographic and chronic condition data. The 11 chronic condition flags are individually addressable fields.

| Field | Type | Description |
|-------|------|-------------|
| `BeneID` | Text | Primary key; foreign key linking claims to demographics |
| `DOB` | Date | Date of birth; used for age validation |
| `DOD` | Date | Date of death — **NULL if alive**. Claims after DOD = posthumous fraud (AX1) |
| `Gender` | Numeric | 1=Male, 2=Female |
| `Race` | Numeric | 1=White, 2=Black, 3=Other, 5=Hispanic |
| `State` | Numeric | US state code of residence (used in geographic fraud analysis) |
| `County` | Numeric | County code within state |
| `IsDeceased` | Boolean (derived) | 1 if DOD is not null |
| `Age` | Numeric (derived) | Age calculated from DOB; Medicare patients typically 65+ |
| `NoOfMonths_PartACov` | Numeric | Months covered under Medicare Part A (hospital). Range 0–12 |
| `NoOfMonths_PartBCov` | Numeric | Months covered under Medicare Part B (medical). Range 0–12 |
| `IPAnnualReimbursementAmt` | Numeric | Annual inpatient reimbursement total |
| `IPAnnualDeductibleAmt` | Numeric | Annual inpatient deductible paid |
| `OPAnnualReimbursementAmt` | Numeric | Annual outpatient reimbursement total |
| `OPAnnualDeductibleAmt` | Numeric | Annual outpatient deductible paid |
| `RenalDiseaseIndicator` | Boolean | 1 = chronic kidney/renal disease |
| `ChronicCond_Alzheimer` | Boolean | 1 = Alzheimer disease |
| `ChronicCond_Heartfailure` | Boolean | 1 = heart failure |
| `ChronicCond_KidneyDisease` | Boolean | 1 = chronic kidney disease |
| `ChronicCond_Cancer` | Boolean | 1 = cancer |
| `ChronicCond_ObstrPulmonary` | Boolean | 1 = COPD |
| `ChronicCond_Depression` | Boolean | 1 = depression |
| `ChronicCond_Diabetes` | Boolean | 1 = diabetes |
| `ChronicCond_IschemicHeart` | Boolean | 1 = ischemic heart disease |
| `ChronicCond_Osteoporasis` | Boolean | 1 = osteoporosis |
| `ChronicCond_rheumatoidarthritis` | Boolean | 1 = rheumatoid arthritis |
| `ChronicCond_stroke` | Boolean | 1 = stroke history |
| `ChronicConditionCount` | Numeric (derived) | Sum of all 11 condition flags (range 0–11). **Key fraud signal**: mismatch between condition count and procedures billed |

---

### 3. `Claim`
**Datasource:** `query_claims_db` → `claims_sql` (simulates PostgreSQL)  
**Keys:** `ClaimID`, `Provider`, `BeneID`  
**Joins to:** `Provider` on `Provider`, `Beneficiary` on `BeneID`, `Physician` on `AttendingPhysician`

Base claim entity covering both inpatient and outpatient records.

| Field | Type | Description |
|-------|------|-------------|
| `ClaimID` | Text | Primary key |
| `BeneID` | Text | FK → Beneficiary |
| `ProviderID` | Text | FK → Provider |
| `ClaimStartDt` | Date | First date of service |
| `ClaimEndDt` | Date | Last date of service |
| `ClaimDuration` | Numeric (derived) | Days between start and end; very short + high reimbursement = upcoding |
| `InscClaimAmtReimbursed` | Numeric | Total USD reimbursed by Medicare — **primary financial fraud signal** |
| `DeductibleAmtPaid` | Numeric | Patient deductible; low deductibles + high reimbursement = inflated billing |
| `ClaimType` | Text (derived) | "Inpatient" or "Outpatient" |

---

### 4. `InpatientClaim`
**Datasource:** `query_claims_db` → `claims_sql` with filter `claim_type = Inpatient`  
**Subtype of:** `Claim`

Hospital admission-specific fields.

| Field | Type | Description |
|-------|------|-------------|
| `AdmissionDt` | Date | Hospital admission date |
| `DischargeDt` | Date | Hospital discharge date |
| `AdmissionDuration` | Numeric (derived) | Days admitted. **Strongest upcoding signal**: 0–1 day stays with high reimbursement |
| `AttendingPhysician` | Text | Supervising physician ID |
| `OperatingPhysician` | Text | Surgeon ID (NULL if no surgery) |
| `OtherPhysician` | Text | Additional physician (NULL if not applicable) |
| `ClmDiagnosisCode_1` | Text | Primary ICD-9 diagnosis code |
| `ClmProcedureCode_1` | Text | Primary ICD-9 procedure code |
| `InscClaimAmtReimbursed` | Numeric | Inpatient reimbursement (legitimate range: $0–$20,000) |
| `DeductibleAmtPaid` | Numeric | Patient deductible for this stay |
| `DiagnosisGroupCode` | Text | DRG code classifying the inpatient case for reimbursement |

---

### 5. `OutpatientClaim`
**Datasource:** `query_claims_db` → `claims_sql` with filter `claim_type = Outpatient`  
**Subtype of:** `Claim`

Outpatient visit-specific fields. Same physician fields as inpatient.

| Field | Type | Description |
|-------|------|-------------|
| `ClaimStartDt` | Date | Date of outpatient service |
| `ClaimEndDt` | Date | End of service period (often same as start) |
| `AttendingPhysician` | Text | Attending physician ID |
| `ClmDiagnosisCode_1` | Text | Primary ICD-9 diagnosis code |
| `ClmProcedureCode_1` | Text | Primary ICD-9 procedure code |
| `InscClaimAmtReimbursed` | Numeric | Outpatient reimbursement (legitimate max: ~$3,500; above = anomalous) |
| `DeductibleAmtPaid` | Numeric | Patient deductible for this visit |

---

### 6. `Physician`
**Datasource:** `query_physician_store` → `physician_nosql` (simulates MongoDB)  
**Key:** `PhysicianID`  
**Joins to:** `Claim` on `AttendingPhysician`, `Provider` on `PhysicianID`

Network risk features for physicians who appear across multiple claims and providers.

| Field | Type | Description |
|-------|------|-------------|
| `PhysicianID` | Text | Derived from `AttendingPhysician`; same physician can span multiple providers |
| `PhysicianRole` | Text | Attending / Operating / Other |
| `TotalPatientsAttended` | Numeric (derived) | Distinct patients attended; high counts = high-volume fraud involvement |
| `TotalClaimsInvolved` | Numeric (derived) | Total claims across all providers; extreme values = central fraud network node |
| `AvgReimbursementPerClaim` | Numeric (derived) | High values = association with expensive/fraudulent claims |
| `UniqueProvidersWorkedWith` | Numeric (derived) | **Key AX3 field**: physicians working across many providers = organized fraud indicator |
| `FraudAssociationRate` | Numeric (derived) | Proportion of this physician's claims belonging to fraud-flagged providers (range 0–1). **AX3 threshold: > 0.5** |

> **Note on `FraudNetworkRisk`:** This field is computed during Phase 3 and stored in `physician_nosql`. It is set to `"High"` when `UniqueProvidersWorkedWith > 2` AND `FraudAssociationRate > 0.5` (AX3). The threshold was relaxed from `> 5` to `> 2` because the Kaggle dataset's physician network is sparse — the original threshold yielded zero flagged physicians.

---

### 7. `DiagnosisCode`
**Datasource:** `query_claims_db` → `claims_sql`  
**Key:** `ClmDiagnosisCode_1`  
**Joins to:** `Claim` on `ClmDiagnosisCode_1`

ICD-9 diagnosis codes derived from claim records; aggregated fraud statistics per code.

| Field | Type | Description |
|-------|------|-------------|
| `DiagnosisCode` | Text | ICD-9 code string |
| `DiagnosisDescription` | Text | Human-readable description (e.g., 4019 = Unspecified essential hypertension) |
| `FrequencyInFraudClaims` | Numeric (derived) | Times this code appears in fraud-flagged provider claims |
| `AvgReimbursementAmt` | Numeric (derived) | Average reimbursement for claims using this code; high-value codes preferred by fraudsters |
| `FraudRate` | Numeric (derived) | Proportion of claims using this code belonging to fraud providers (range 0–1) |

---

### 8. `ProcedureCode`
**Datasource:** `query_claims_db` → `claims_sql`  
**Key:** `ClmProcedureCode_1`  
**Joins to:** `Claim` on `ClmProcedureCode_1`

ICD-9 procedure codes; aggregated fraud statistics per code.

| Field | Type | Description |
|-------|------|-------------|
| `ProcedureCode` | Text | ICD-9 procedure code string |
| `ProcedureDescription` | Text | Human-readable procedure description |
| `FrequencyInFraudClaims` | Numeric (derived) | Times this procedure appears in fraud-flagged claims |
| `AvgReimbursementAmt` | Numeric (derived) | Average reimbursement; high-reimbursement procedures commonly misused in upcoding |
| `FraudRate` | Numeric (derived) | Proportion of claims using this procedure belonging to fraud providers (range 0–1) |

---

### 9. `GeographicLocation`
**Datasource:** `query_analytics_warehouse` → `analytics_warehouse` (simulates Snowflake)  
**Key:** `ProviderState`  
**Default filter:** `query_type = "geographic"`  
**Joins to:** `Provider` on `ProviderState`, `Beneficiary` on `State`

Pre-aggregated geographic fraud statistics by US state.

| Field | Type | Description |
|-------|------|-------------|
| `State` | Numeric | US state code |
| `County` | Numeric | County code within state |
| `StateName` | Text | Human-readable state name |
| `TotalProviders` | Numeric (derived) | Distinct providers in this state |
| `FraudProviderRate` | Numeric (derived) | Proportion of providers that are fraud-flagged. **High values = geographic fraud hotspots** |
| `AvgReimbursementPerClaim` | Numeric (derived) | Average reimbursement per claim from this state |

---

### 10. `TimeWindow`
**Datasource:** `query_analytics_warehouse` → `analytics_warehouse` (simulates Snowflake)  
**Keys:** `ClaimYear`, `ClaimMonth`, `ClaimQuarter`  
**Default filter:** `query_type = "time_window"`  
**Joins to:** `Claim` on `ClaimStartDt`

Pre-aggregated time-period fraud statistics.

| Field | Type | Description |
|-------|------|-------------|
| `ClaimYear` | Numeric | Calendar year of claim start |
| `ClaimMonth` | Numeric | Calendar month (1–12) |
| `ClaimQuarter` | Numeric | Quarter (1–4) |
| `IsHolidayPeriod` | Boolean (derived) | 1 if Q4 (Oct–Dec); oversight changes may affect fraud rates |
| `AvgClaimsPerDay` | Numeric (derived) | Average daily claim volume; spikes indicate coordinated fraud bursts |
| `FraudRateInPeriod` | Numeric (derived) | Proportion of claims in this period belonging to fraud-flagged providers |

---

## Simulated Data Sources

| Source Name | Simulates | Tool Function | Handles Entity |
|-------------|-----------|---------------|----------------|
| `provider_sql` | SQLite operational DB | `query_provider_db` | Provider |
| `beneficiary_sql` | MySQL patient records DB | `query_beneficiary_db` | Beneficiary |
| `claims_sql` | PostgreSQL claims DB | `query_claims_db` | Claim, InpatientClaim, OutpatientClaim, DiagnosisCode, ProcedureCode |
| `physician_nosql` | MongoDB document store | `query_physician_store` | Physician |
| `analytics_warehouse` | Snowflake analytics warehouse | `query_analytics_warehouse` | GeographicLocation, TimeWindow |
| `audit_store` | External REST API | `query_audit_store` | Ground truth findings corpus |

All sources are backed by CSV/JSON files read via pandas. Tool function signatures are identical to what production database connectors would expose, enabling direct substitution in production deployment.

---

## Relation Paths (`relationtrees`)

12 directed traversal paths. Each path specifies the hop tools and join column used by the Executor agent for automatic key injection between steps.

| # | Path | Hop Tools | Join Column |
|---|------|-----------|-------------|
| 1 | `(P:Provider)→(C:Claim)` | query_provider_db → query_claims_db | `Provider` |
| 2 | `(B:Beneficiary)→(C:Claim)` | query_beneficiary_db → query_claims_db | `BeneID` |
| 3 | `(C:Claim)→(PH:Physician)` | query_claims_db → query_physician_store | `AttendingPhysician` |
| 4 | `(C:Claim)→(D:DiagnosisCode)` | query_claims_db → query_claims_db | `ClmDiagnosisCode_1` |
| 5 | `(C:Claim)→(PC:ProcedureCode)` | query_claims_db → query_claims_db | `ClmProcedureCode_1` |
| 6 | `(C:Claim)→(TW:TimeWindow)` | query_claims_db → query_analytics_warehouse | `ClaimStartDt` |
| 7 | `(B:Beneficiary)→(GL:GeographicLocation)` | query_beneficiary_db → query_analytics_warehouse | `State` |
| 8 | `(P:Provider)→(GL:GeographicLocation)` | query_provider_db → query_analytics_warehouse | `ProviderState` |
| 9 | `(IC:InpatientClaim)→(C:Claim)` | query_claims_db → query_claims_db | `ClaimID` |
| 10 | `(OC:OutpatientClaim)→(C:Claim)` | query_claims_db → query_claims_db | `ClaimID` |
| 11 | `(PH:Physician)→(P:Provider)` | query_physician_store → query_provider_db | `PhysicianID` |
| 12 | `(B:Beneficiary)→(D:DiagnosisCode)` | query_beneficiary_db → query_claims_db | `BeneID` |

### 4-Hop Multi-Hop Example Paths

These illustrate the non-trivial traversal patterns that AmbientOn's GoT reasoning is designed to reason over:

| Path | Causal Question |
|------|----------------|
| Provider → Claim → Beneficiary → ChronicCondition | Are high-reimbursement providers billing patients whose conditions don't justify the procedures? |
| Provider → Claim → Physician → Provider | Are the same physicians appearing across multiple fraud-flagged providers (organized network)? |
| Provider → Claim → DiagnosisCode → ProcedureCode | Are providers billing high-reimbursement procedures for diagnoses that don't typically require them? |
| Beneficiary → Claim → TimeWindow → Provider | Are deceased patients' claims submitted after DOD, and which providers are responsible? |
| Provider → GeographicLocation → Provider → Claim | Are geographically clustered providers showing correlated fraud patterns (regional rings)? |

---

## Formal Axioms

Five domain rules encoding fraud detection logic. Each axiom constrains hypothesis generation in the GoT prompt and drives CoVe verification question formulation.

### AX1 — Posthumous Billing Detection
```
IF Claim.ClaimStartDt > Beneficiary.DOD
   AND Beneficiary.IsDeceased = 1
THEN Claim.FraudType = 'PosthumousBilling'
```
- **Verification type:** Deterministic (exact SQL join)
- **Confidence:** 1.0
- **⚠️ Dataset note:** Yields **zero findings** in the Kaggle release. All deceased patients have `DOD = 2009-12-31` (anonymised). The axiom is domain-correct and retained for completeness.

---

### AX2 — Reimbursement Outlier Detection
```
IF Claim.InscClaimAmtReimbursed
     > Q3(InscClaimAmtReimbursed | same DRG)
       + 1.5 × IQR(InscClaimAmtReimbursed | same DRG)
THEN Claim.IsReimbursementOutlier = TRUE
```
- **Verification type:** Statistical (IQR threshold per DRG group)
- **Confidence:** 0.85
- **Usage:** Basis for upcoding and peer deviation hypotheses

---

### AX3 — Physician Multi-Provider Flag
```
IF Physician.UniqueProvidersWorkedWith > 2
   AND Physician.FraudAssociationRate > 0.5
THEN Physician.FraudNetworkRisk = 'High'
```
- **Verification type:** Multi-hop (graph traversal)
- **Confidence:** 0.78
- **Threshold note:** Relaxed from `> 5` to `> 2` — the Kaggle dataset's physician network is sparse; the original threshold yielded 0 flagged physicians. A physician working across 3+ providers with > 50% fraud association remains a strong organized fraud signal.

---

### AX4 — Procedure-Diagnosis Mismatch
```
IF ProcedureCode.ProcedureCode
     NOT IN expected_procedures_for(DiagnosisCode.DiagnosisCode)
THEN Claim.FraudType = 'ProcedureDiagnosisMismatch'
```
- **Verification type:** Deterministic
- **Confidence:** 0.75
- **Usage:** Drives procedure-diagnosis mismatch fraud hypotheses

---

### AX5 — High-Volume Low-Condition Billing
```
IF Provider.TotalReimbursementAmt > PEER_95TH_PERCENTILE
   AND AVG(Beneficiary.ChronicConditionCount) < PEER_MEDIAN
THEN Provider.FraudRiskScore += 0.2
```
- **Verification type:** Statistical
- **Confidence:** 0.78
- **Usage:** Flags providers billing disproportionately high amounts for patients with few chronic conditions — a sign of unnecessary or phantom procedures

---

## Ground Truth Strategy

| Method | Description |
|--------|-------------|
| **Primary** | `PotentialFraud` column — provider-level binary label (direct verification oracle) |
| **Secondary: Statistical** | IQR outlier detection on `InscClaimAmtReimbursed` per DRG group — flags claim-level anomalies |
| **Secondary: Temporal** | Rolling 4-week z-score on `AvgClaimsPerDay` per TimeWindow — flags burst patterns |
| **Secondary: Deterministic** | `ClaimStartDt > BeneDOD` — posthumous fraud (no statistical test needed) |
| **Evaluation corpus size** | 400 findings |
| **Expert validation** | 10% stratified sample (40 findings), 3 judges, majority vote |

---

## Hypothesis Examples (Few-Shot Prompts)

These 5 examples are injected into the GoT generation prompt as few-shot demonstrations to enforce the correct hypothesis JSON schema:

1. *"Providers billing high reimbursements for patients with no matching chronic conditions are likely committing upcoding fraud."*
2. *"Physicians appearing across more than 5 providers with a fraud association rate above 0.5 are central nodes in an organized Medicare fraud network."*
3. *"Claims submitted after a beneficiary's recorded date of death indicate posthumous billing — the provider is billing for services that were never rendered."*
4. *"Providers in geographic clusters (same state/county) with correlated FraudRiskScores suggest regional fraud rings sharing billing patterns."*
5. *"Outpatient claims with reimbursements above $3,500 using high-fraud-rate diagnosis codes indicate systematic upcoding of outpatient services as inpatient."*

---

## How the Ontology Is Used in the Pipeline

| Phase | Agent | How This Ontology Is Used |
|-------|-------|--------------------------|
| Phase 3 | Data Setup | `datasource.tool_name` bindings used to register tool functions and validate routing |
| Phase 4 | Orchestrator-Analyzer | Reads `relationtrees.paths` and `axioms` to build investigation context for each finding |
| Phase 4 | Hypothesis Generator | Receives `nodes`, `axioms`, and `hypothesis_examples` as part of the GoT prompt |
| Phase 5 | Planner Agent | Reads `datasource.tool_name` per entity class to resolve which tool to call for each evidence step |
| Phase 5 | Executor Agent | Uses `join_keys` from each node's datasource block for automatic key injection between pipeline steps |
| Phase 5 | Consolidator (CoVe) | Receives axiom list to constrain verification questions; uses `ground_truth_strategy` to locate the oracle |
| Phase 6 | Ablation Study | Same ontology used across all 7 reasoning configurations — only generation strategy changes |

---

## Extending or Modifying the Ontology

### Adding a New Entity Class
1. Add a new key under `nodes` with the entity name
2. Include a `datasource` block specifying `tool_name`, `source_name`, `key_fields`, and `join_keys`
3. List all field definitions in the same block
4. Add corresponding traversal paths under `relationtrees.paths`
5. Implement the tool function with the name specified in `tool_name`

### Adding a New Axiom
1. Append to the `axioms` array with a new `axiom_id` (e.g., `AX6`)
2. Specify `rule` (IF/THEN), `confidence` (0–1), and `verification_type` (Deterministic / Statistical / Multi-hop)
3. The axiom will automatically be included in the Orchestrator's context for all subsequent findings

### Modifying Datasource Routing
1. Change `datasource.tool_name` or `datasource.source_name` in the relevant node
2. No code changes are required — the Planner reads routing at runtime from this file

---

## Known Limitations

| Issue | Detail |
|-------|--------|
| AX1 yields zero findings | Kaggle anonymised all deceased patient DODs to `2009-12-31`. Posthumous billing cannot be detected from this dataset. |
| AX3 threshold relaxation | Original `>5` threshold yielded 0 physicians. Relaxed to `>2` based on dataset's sparse network structure. |
| Reversed-direction findings | F05 (ChronicCondMismatch) and F09 (ZeroDurationAnomaly) show effects in the opposite direction to domain expectation. These are valid null results — CoVe correctly REFUTES domain-intuitive hypotheses for these findings. |
| Physician FraudNetworkRisk | This derived field is computed in Phase 3 and stored in `physician_nosql`. It is not present in the raw Kaggle files. |

---


*AmbientOn Research Project — DS1 Healthcare Fraud Detection*  
*Ontology version 2.0 | Last updated: Phase 3 update*
