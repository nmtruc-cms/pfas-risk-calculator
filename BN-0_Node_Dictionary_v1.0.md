# BN-0 — BN-Chemical Node Dictionary v1.0 (Consolidated Draft)

**Baseline:** 72 agreed BN-0 decisions. **Status:** Draft for structural validation, not numerical CPT estimation. This document lists **canonical conceptual variables**, not automatically one independent DAG node per metadata field. Each entry includes ID, type, states/domain, causal relations, scientific rationale and uncertainty. `Unknown` is *unobserved* unless a field expressly says `not reported` as a reporting status.

## Node types

- **L** latent/true chemical or operational state; **O** observed evidence; **D** deterministic function/classification; **C** causal/context variable whose state can differ by batch, stage or supplier; **M** metadata or model governance, not a DAG variable by default.
- **Uncertainty codes:** **R** structural rule or definition is determined (given premise); **H** plausible causal hypothesis requiring case-specific scientific support; **E** observation/error mechanism to characterize in a later phase. These are *not* numerical confidence levels.

## Canonical entries

| Node_ID | Module / type | Definition and possible states | Direct parent → child relationship (conceptual) | Rationale / evidence scope | Status |
|---|---|---|---|---|---|
| `CHEM_ID` | Identity / M | Substance record: names, CAS, structure, polymer/UVCB type; fields, not PFAS outcome | metadata for `PFAS_CLASS`, `INTENT` | OECD 2021 structure-based definition; CAS not classification rule | R |
| `PFAS_CLASS` | Identity / D | Whether a sufficiently specified structure satisfies OECD 2021: Yes/No; if insufficient structure, unobserved | `CHEM_ID` structural evidence → `INTENT`, interpretation of identified analytes | Deterministic for known adequately specified structure | R |
| `PRODUCT` | Formulation / M | Commercial product ID and broad category | hierarchy `PRODUCT`→`FORM_VERSION` | Product category not automatically causal | R |
| `FORM_VERSION` | Formulation / C | Actual formulation version and its component configuration | `PRODUCT` context → `INTENT`, `TRANSFORM` | Distinguish revisions from batch variations | H |
| `COMPONENT_INPUT` | Formulation / C | Presence of functional constituent/input in formulation; Yes/No per ingredient | `FORM_VERSION`, ingredient source → `INTENT` | A substance can be PFAS without necessarily being present | R/H |
| `CHEM_TECH` | Formulation / C | Chemical technology family, categorical as justified | application/function, upstream choices → component/source routes | Technology affects choices/pathways, not a magic risk score | H |
| `INTENT` | Formulation / L | PFAS intentionally present as input: Yes/No | `PFAS_CLASS`, `COMPONENT_INPUT` → `PROD_STATE` | Source mechanism 1; classified chemistry plus actual use | R/H |
| `RAW_LOT` | Raw Material / C | Raw material identity, supplier and production lot configuration | identity/supplier/history → `IMPURITY` | Same CAS can have different impurity profiles; IDs are metadata | H |
| `RECYCLED_HISTORY` | Raw Material / C | Origin and prior-use/recovery exposure relevant to legacy PFAS | → `IMPURITY` | Recycled content alone not proof of PFAS | H |
| `SYNTHESIS_PATH` | Raw Material / C | Starting chemistry, synthesis and purification relevant to residual PFAS | → `IMPURITY`, `TRANSFORM` | Route-specific chemistry, not region stereotypes | H |
| `IMPURITY` | Raw Material / L | PFAS impurity introduced via any incoming source/lot: Yes/No | `RAW_LOT`, `RECYCLED_HISTORY`, `SYNTHESIS_PATH`, relevant controls → `PROD_STATE` | Source mechanism 2; legacy residues **not** counted again independently | H |
| `TRANSFORM_COND` | Formulation/Lifecycle / C | Reaction/processing/storage/use conditions enabling relevant reaction | → `TRANSFORM` or stage transition | Transformation possibility ≠ actual formation | H |
| `TRANSFORM` | Formulation / L | PFAS present as transformation product by completion of production: Yes/No | precursors, `SYNTHESIS_PATH`, `TRANSFORM_COND` → `PROD_STATE` | Source mechanism 3; PFAS precursor itself counts when structurally PFAS | H |
| `SITE_LINE` | Manufacturing / C | Actual plant/line used, equipment and line-history conditions | → `CROSS_CONTAM` | Supplier legal entity is distinct from actual manufacturing site | H |
| `CTRL_INGREDIENT` | Manufacturing / C | Actual effectiveness of upstream incoming-material controls, not policy existence | → `IMPURITY` | Controls only act on mechanisms within their scope | H |
| `CTRL_SYNTHESIS` | Manufacturing / C | Actual synthesis/purification control effectiveness | → `IMPURITY`, where physically applicable | Do not assume one overall site control score | H |
| `CTRL_CROSS` | Manufacturing / C | Equipment/cleaning/segregation effectiveness | → `CROSS_CONTAM` | Includes cleaning effectiveness; no duplicate cleaning-source node | H |
| `CROSS_CONTAM` | Manufacturing / L | Manufacturing-line PFAS cross-contamination: Yes/No | `SITE_LINE`, `CTRL_CROSS` → `PROD_STATE` | Source mechanism 4 | H |
| `PROD_STATE` | Lifecycle / D | PFAS present in real production batch at completion: Yes/No | **OR** (`INTENT`, `IMPURITY`, `TRANSFORM`, `CROSS_CONTAM`) → `SAMPLE_TIME_STATE` | Bước 63B; parents potentially dependent, OR deterministic conditional on true states | R |
| `POST_HANDLING` | Lifecycle / C | Contamination-relevant packaging, transfer, transport, storage and controls | → `POST_CONTAM` | Stage after production, prior to collection | H |
| `POST_CONTAM` | Lifecycle / L | PFAS physically introduced after manufacture prior to sampling: Yes/No | `POST_HANDLING` → `SAMPLE_TIME_STATE` | Separate from production contamination and field sampling artifact | H |
| `SAMPLE_TIME_STATE` | Lifecycle / L/D | True PFAS presence in formulation at sampling instant: Yes/No | `PROD_STATE`, `POST_CONTAM`, any supported removal/transformation → `USE_STATE`, `SAMPLE_STATE` | Time-indexed state; exact transition rules deferred | H |
| `USE_STAGE` | Lifecycle / C | Mixing, reaction, removal and conditions occurring before/in use | → `USE_STATE` | Dilution alone is not PFAS removal | H |
| `USE_STATE` | Lifecycle / L | True PFAS presence at defined use stage: Yes/No | `SAMPLE_TIME_STATE`, `USE_STAGE` → downstream material model later | Stage may differ due to demonstrable chemistry, not mere detectability | H |
| `PREANALYTIC` | Measurement / L | Sample contamination introduced by collection, container, handling or preparation: Yes/No | collection/handling practices → `SAMPLE_STATE`, QC observations outside BN | Artifact is not necessarily product contamination | H |
| `SAMPLE_STATE` | Measurement / L | True PFAS state of physically collected/handled sample: Yes/No | `SAMPLE_TIME_STATE`, `PREANALYTIC`, batch heterogeneity → analytical measurements | Sample vs batch distinction (22B, 55B) | H |
| `SAMPLE_SELECTION` | Measurement / C | Inclusion/target-selection pathway for sampling/testing | selection context → whether observations exist / representativeness | Targeted surveys not representative prevalence by default | H |
| `METHOD_SCOPE` | Measurement / M/C | Test method, targeted analyte list, matrix, analyte limits, per-analyte coverage and performance | governs `TARGET_OBS`, `SCREEN_OBS` | No panel can cover all OECD PFAS; LOD/LOQ retained | E |
| `TARGET_OBS` | Measurement / O | Confirmed targeted analyte measurement: Detected/Not_Detected; concentration, units, LOD/LOQ as associated fields | `SAMPLE_STATE` + true analyte-specific concentration + `METHOD_SCOPE` → observed result | Only confirmed chemical identity accepted; <LOD/LOQ simplified ND (59A) | E |
| `SCREEN_OBS` | Measurement / O | TF/EOF/AOF/TOP assay result and limits; method-specific data | underlying fluorine content/precursor reactivity + `METHOD_SCOPE` → observation | Nonspecific fluorine measurements ≠ PFAS structural confirmation | E |
| `DISCLOSURE_PROCESS` | Evidence / C | True information availability and disclosure behavior | actual composition and organization factors → `DECLARED_OBS` / missingness | Disclosure is an observation mechanism not a PFAS generator | H |
| `DECLARED_OBS` | Evidence / O | SDS/full-disclosure/claim/certification and declared scope/version | true formulation + `DISCLOSURE_PROCESS` → report | PFAS-free claim conditional on scope; absence on SDS ≠ absence | E |
| `EVIDENCE_SCOPE` | Evidence / M | Original source, version, batch, analyte, period, methods, publication trace | conditions interpretation of all observations | Repeated citation is not new independent observation | R |
| `QC_ACCEPT` | Evidence / M | Accept/reject; published results presumed accepted unless invalidated | controls ingestion of analytical evidence | Decisions 56A + 57C; not laboratory QC subnetwork | R |
| `MODEL_SCOPE` | Evidence/governance / M | Applicability domain, dataset shift, structural hypotheses and known gaps | determines reporting limitations, not product PFAS | Decisions 45B–48B | H |

**States convention:** `Yes/No` for physical source/target; unknown data is marginalized. `Valid/Invalid` only a **QC acceptance/reporting** status, not a PFAS state. Categorical technology, process and condition states are to be specified only where scientifically justified (69B). No numerical priors or CPT values are proposed here.

## Observed-field schema (metadata, not automatically additional DAG nodes)

| Evidence type | Retained fields | Interpretation |
|---|---|---|
| Targeted PFAS | Substance ID (confirmed); sample/version/batch; concentration and unit; Detected/ND; LOD/LOQ; method coverage | ND is not physical absence; under-limit initially mapped to ND, original reporting preserved |
| TF, EOF, AOF, TOP | Method name, measurand, numeric value/unit, limits, sample stage and source | Indirect, distinct measurement channels; TF is not PFAS identification |
| Ingredient declaration/SDS | Document version, date, reported ingredient scope, declaring entity | Incomplete declaration is not a negative chemical truth |
| Certification/claim | Scope, scheme, assessed batch/version, time validity, original document | Bounded evidence, not deterministic PFAS-free proof |
| Source provenance | Original publication/study, citation lineage, sampling design, geographical/technology context | Prevent source duplication and nonrepresentative transfer |
| QC acceptance | Published/official, explicitly invalidated/retracted?, decision | Published results presumed valid for ingestion as project assumption, not verified QC compliance |

## Aliases and deduplication map

| Historical terms in BN-0 discussion | Canonical destination | Why |
|---|---|---|
| `Legacy_PFAS_Residues`, input-lot contaminants | `IMPURITY` with `RECYCLED_HISTORY` as source context | One physical impurity pathway, no double counting |
| `Cleaning_Effectiveness`, cross-contamination controls | `CTRL_CROSS` | An equipment-cleaning submechanism of manufacturing control |
| `Raw_Material_PFAS_State` | `IMPURITY` and/or `INTENT` by actual input role | Raw-material state is a summarizing interface, not an extra 5th source |
| `Manufacturing_Contamination_State` | `CROSS_CONTAM` | Same physical manufacturing source |
| `Formulation_PFAS_State`, `PFAS_State_At_Production` | `PROD_STATE` | Same batch/timepoint state |
| `PFAS_State_At_Sampling` | `SAMPLE_TIME_STATE` | Product at collection time; distinct from handled sample |
| `Sample_PFAS_State` | `SAMPLE_STATE` | Includes pre-analytical artifacts |
| `PFAS_State_At_Use` | `USE_STATE` | Later lifecycle timepoint |
| `Analytical_Non_Detection`, `Targeted_PFAS_Result` | `TARGET_OBS` | Same observed analyte channel |
| `QC_Validity_Status`, method blank/recovery/precision QC | `QC_ACCEPT` metadata only | 56A and 57C; no detailed QC DAG |
| `Unmodelled_Causal_Factors`, structural uncertainty, applicability | `MODEL_SCOPE` governance + scoped hypothesis notes | Not an invented fifth PFAS source |

## Structural validation checklist

- [x] All 72 decisions reflected at the level of scope, canonical states, evidence or governance, without pretending all are physical DAG variables.
- [x] Four PFAS source mechanisms retained; deterministic OR identified.
- [x] Polymer/UVCB kept at substance-level (62A); structurally unresolved does not mean non-PFAS.
- [x] QC acceptance simplified (56A/57C); left-censored reporting simplified (59A); confirmed identity only (60A).
- [x] Seven-module structure and interfaces mapped; aliases consolidated.
- [ ] Review each proposed edge against published chemistry or process evidence and mark `R/H/E` case-specifically.
- [ ] Instantiate formal node state spaces and run cycle/duplicate-path checks on the machine-readable DAG.
- [ ] Validate temporal transitions from production to sampling and use, especially transformation/removal without inference from dilution.
- [ ] Validate the observation model for TF/EOF/AOF/TOP without unintentionally treating screening as confirmed identity.
- [ ] Sign off DAG+dictionary jointly and mark BN-0 as **FROZEN**.

**Current conclusion:** BN-0 scope and structural architecture are consolidated. Structural freeze **criteria have been selected, but the actual evidence-based edge review has not been completed**; thus `v1.0` is a version label for the draft, not a false claim of validation.
