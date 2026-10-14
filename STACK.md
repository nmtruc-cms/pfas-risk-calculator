# PFAS-RISK LAB — STACK.md

**Technical implementation contract · CodeIgniter 4 prototype v0.2 · 14 October 2026**

This is the **developer/Codex implementation specification**, separate from the nontechnical research concept in [README.md](README.md). Read `README.md` first. If this technical plan conflicts with its definitions of outcome, evidence, data rights or mathematical model, **README.md takes precedence**.

## 1. Scope and development constraints

Build an experimental web application supporting:

1. **Chemical Formulation** profiles, versions, compositions/impurities and evidence.
2. **Material** profiles (textile, leather), ordered production steps, chemical applications, components, lots and evidence.
3. **Shared Chemical Knowledge Graph**, starting from reviewed public OECD sources.
4. **Analytical Data** for both sectors, including detection/quantification limits, individual analytes and measured values.
5. **Research Dataset Builder**, expert review, provenance and strictly separated synthetic records.
6. **Statistics/model comparison** in later milestones, with Hierarchical Bayesian inference executed outside PHP.

**Do not implement** qualitative High/Low logic as probability labels, arbitrary expert risk scores as predictions, fabricated posteriors, automatic conformity decisions, web scraping of protected data, or an automatic PFAS determination from a CAS-list non-match.

## 2. Technology choices

| Layer | Preferred implementation |
|---|---|
| UI | CI4 Views, Bootstrap 5 or clean HTML/CSS, vanilla JS |
| API/business rules | CodeIgniter 4, PHP 8.2+ |
| SQL database | MySQL 8 / MariaDB (use migrations) |
| Research statistics | Python 3.11+, FastAPI, PyMC, ArviZ, pandas, NumPy, scikit-learn |
| Chemistry structural parsing | Later: RDKit Python microservice; do **not** infer structure from substance name alone |
| Background work | CI4 CLI job or managed worker for imports/training; model work not inside an HTTP request |
| Knowledge graph MVP | Normalized MySQL nodes, typed edges, assertions, sources; no mandatory Neo4j/Qdrant |
| Deployment | Local development first; Docker Compose optional for PHP/DB/Python components |

Network calls to statistics service must have retries/timeouts, authentication, explicit schema versions and robust failure reporting. Keep API tokens, DB credentials, proprietary dataset URLs and user records out of Git.

## 3. Repository layout (proposed)

```text
pfas-risk-lab/
├─ README.md                 # Research concept, protocol, maths
├─ STACK.md                  # This implementation contract
├─ app/
│  ├─ Controllers/{Formulations,Materials,Knowledge,Analytical,Research,Api}/
│  ├─ Models/
│  ├─ Services/{Evidence,KnowledgeGraph,Dataset,ModelClient}/
│  ├─ Validation/
│  ├─ Database/Migrations/
│  └─ Views/
├─ public/assets/
├─ research-service/
│  ├─ app/{api,models,schemas,validation}/
│  ├─ tests/
│  ├─ requirements.txt
│  └─ README.md
├─ data/
│  ├─ fixtures/             # Clearly labeled synthetic/dev only
│  └─ imports/              # Local ignored staging, not committed
├─ docs/{data-dictionary,methods,provenance}/
└─ tests/
```

## 4. Database schema — minimum logical entities

Use foreign keys, migrations, uniqueness/version constraints and UTC timestamps; display local time in UI. Below are logical groupings rather than a requirement for one table per row.

| Domain | Required tables / entities | Important fields |
|---|---|---|
| Study | `research_projects`, `dataset_versions`, `study_protocols` | study_id, cohort, endpoint, panel_version, matrix, sampling method |
| Formulations | `formulations`, `formulation_versions`, `formulation_batches` | category, subcategory, function, manufacturer pseudonym, valid dates |
| Ingredients | `substances`, `substance_identifiers`, `formulation_components` | CAS/EC/InChIKey, structure provenance, min/max composition, role, disclosure status |
| Material | `materials`, `material_components`, `material_lots` | sector, family (textile/leather), specification, composition, matrix |
| Process | `process_definitions`, `process_steps`, `chemical_applications` | step order, conditions/units, formulation_version, applied mass/dosage |
| Samples | `samples`, `sample_lineage` | sampled object, time, lot, component, selection mechanism |
| Analytical | `analytical_methods`, `analyte_panels`, `panel_members`, `analytical_results` | analyte_id, panel_version, endpoint, result, unit, LOD, LOQ, qualifier, uncertainty |
| Knowledge | `knowledge_sources`, `kg_nodes`, `kg_edges`, `knowledge_assertions` | relation type, source DOI/URL/page, assertion type, evidence level, reviewer |
| Expert | `expert_reviews`, `prior_elicitations`, `review_decisions` | blinded independent reviewers, rationale, distributions, disagreement |
| Modelling | `model_runs`, `model_artifacts`, `predictions`, `validation_reports` | code/data/version hashes, fold, diagnostics, posterior summary, warnings |
| Audit/access | `users`, `roles`, `audit_events`, `file_attachments` | actor, change, before/after, source privacy class, file checksum |

Required controlled enums (explicit values must be documented):

- `observation_status`: `NOT_TESTED`, `QUANTIFIED`, `DETECTED_BELOW_LOQ`, `BELOW_LOD`, `INVALID`, `MISSING`.
- `assertion_status`: `PROPOSED`, `REVIEWED`, `CONFIRMED`, `DISPUTED`, `REFUTED`.
- `composition_role`: `INTENTIONAL`, `IMPURITY`, `BYPRODUCT`, `CONTAMINATION`, `UNKNOWN`.
- `source_class`: `PUBLIC`, `PARTNER_RESTRICTED`, `CONFIDENTIAL`.
- `data_origin`: `REAL`, `SYNTHETIC_DEV`.

**Never** encode a missing measurement as `0`, or absence of a name in an imported OECD list as `PFAS=false`.

## 5. Chemistry identity and OECD import module

Develop an import pipeline with **preview → validate → normalize → deduplicate → review → publish version**:

1. Allow authorized admins to upload downloaded OECD/NORMAN tabular files (CSV/XLSX). Require file name, URL, publication/version, collection date, license note, checksum and original schema snapshot.
2. Display mapping of actual columns to identifiers and synonyms. **Do not hard-code column names before examining the downloaded dataset.**
3. Normalize CAS/EC where present, InChIKey, structures, synonyms and source identifiers. Preserve raw values and separate identity ambiguity records.
4. Record historical OECD list membership as **source-specific evidence**; do not treat the old list as a complete 2021-definition classifier.
5. For reviewed relations from OECD reports, capture citation/page, relation type, context and reviewer. LLM-extracted relations remain `PROPOSED` until approved.
6. Protect duplicate CAS assignments, salt/mixture ambiguity, polymer/UVCB classification and conflicting structures with `NEEDS_REVIEW` flags.

Seed only with **verified downloaded** source data; do not invent entries, probabilities or reference values. Package sample *synthetic test fixture* separately for UI tests.

## 6. Sector applications and interfaces

### Formulations

- Browse/add/edit versioned formulation; category > subcategory > function controls; ingredient list with range and role; fluorine provenance; upload evidence.
- Show knowledge relations with `confirmed/plausible/unresolved` visible and sources attached.
- Show sample history and analytical records; do not imply a conformity verdict.

### Materials

- Textile/leather taxonomy, material components and separate lots; process timeline with nested subprocesses.
- Chemical application step links to formulation version/batch where known; quantity, concentration, pickup/liquor ratio, expected retention with units.
- Support both *linked* and *unlinked* material records; never require a formulation ID.
- Map analytical results to the exact sampled component and lot.

### Knowledge and evidence

- Substance search across CAS, InChIKey and name; browsable relation graph (text/table graph acceptable for MVP).
- Evidence panel with citation, version, license, review status and source access class.
- Imports require human review workflow and audit trail.

### Laboratory data

- Panel definition and version; named analytes; method/matrix; unit and result qualifier.
- Numeric values and LOD/LOQ must be validated to compatible units; record blanks/non-detects correctly.
- CSV import with dry-run error report and explicit sample matching; no silent overwrites.

### Research workspace

- Define explicit endpoint (`target_panel_positive`, named analyte/sum exceedance), population, sampling design and censoring rules.
- Assemble and freeze eligible dataset; show missingness, class prevalence and origin counts.
- Later: launch model runs, view calibration, uncertainty intervals, diagnostics, subgroup performance and warnings.

## 7. Research service contract (later-phase, reserve interfaces now)

Versioned API contract examples, not guaranteed implemented in Sprint 0–3:

```text
GET  /api/v1/health
POST /api/v1/datasets/validate
POST /api/v1/models/baseline/train
POST /api/v1/models/bayesian/train
GET  /api/v1/models/{run_id}/status
GET  /api/v1/models/{run_id}/report
POST /api/v1/models/{run_id}/predict
```

Each prediction request includes: sector, object version/lot, endpoint definition, matrix, analytes/panel, evidence snapshot hash and model version. Response includes model/data version, probability **only if estimable**, interval, feature coverage, `out_of_domain` and diagnostic warnings. Failure/insufficient data returns a structured status—not invented fallback percentages.

Example **response shape only** (no actual numerical prediction):

```json
{
  "status": "INSUFFICIENT_EVIDENCE",
  "sector": "FORMULATION",
  "endpoint": "target_panel_positive",
  "panel_version": "REQUIRED",
  "posterior": null,
  "model_version": null,
  "limitations": ["No validated model for this panel/matrix"]
}
```

## 8. Statistics implementation contract

**Baseline first:** frequency estimator and penalized logistic regression with held-out splits. XGBoost/boosting with probability calibration is an optional comparator.

**Primary Bayesian model:** implement and document the hierarchical occurrence specification in `README.md` using PyMC; use explicit priors, centered/non-centered alternatives where indicated, diagnostics (R-hat, effective sample size, divergences), prior/posterior predictive checks and sensitivity tests. Store complete experiment metadata and reproducible seeds.

**Concentration model:** implement only when real analyte/matrix data allow fitting a zero-inflated/hurdle and censored likelihood. Respect LOD vs LOQ distinctions; prohibit arbitrary non-detect replacement. **Formulation→material** transfer/fate is an optional mechanistic extension with mass-balance units tested explicitly.

Split datasets by grouped formulation/product/lot IDs and future time where feasible. Keep validation folds frozen, log selection mechanism, and never train on hold-out labels. Do not present high/medium/low categories as posterior probabilities.

## 9. Privacy, access and integrity

- Research-public and restricted datasets must be separate by provenance and access policy; prohibit accidental export of bluesign/customer formulas or supplier identity.
- Use role-based access, server-side validation, CSRF protection, safe upload handling and MIME checks.
- Parameterize SQL, record import batches and checksums, maintain audit events and immutable model-run manifests.
- Do not fetch internal company data automatically without explicit authorization and appropriate rights.
- Keep raw observations immutable; corrections are versioned with actor, timestamp and reason.
- Model outputs require a banner **Experimental research — not a certification or compliance decision**.

## 10. Sprint plan and acceptance criteria

| Sprint | Focus | Required acceptance evidence |
|---|---|---|
| **0** | Scaffold CI4, roles, migrations, data dictionary, schema conventions | Fresh migration works; test suite bootstraps; source version metadata exists |
| **1** | Formulation profiles, ingredients, evidence, OECD/NORMAN import preview | Version/batch integrity; one reviewed source import; ambiguous identities flagged |
| **2** | Textile/leather, process sequence, chemical application links | Material-only and linked material cases both work; quantities/units checked |
| **3** | Samples, analyte panels, analytical results, dataset builder | Below-LOD/LOQ and missing records distinguished; imports auditable; CSV export with provenance |
| **4** | Baseline statistics, data leakage safeguards and evaluation UI | Held-out validation; calibration and class-balance report; reproducible model run |
| **5** | Hierarchical Bayesian occurrence service | Documented priors, diagnostics, posterior predictive checks, model card |
| **6** | Fate/transfer, expert elicitation, graph ablations | Reproducible comparison and uncertainty/sensitivity reports |

**Initial implementation directive for Codex:** Implement **only Sprint 0–3** unless the task explicitly expands the scope. Before coding, produce a short implementation plan and database ERD. Make small verified commits or logical increments. Supply database migrations, seeds for controlled enums (not fabricated research observations), routes, controllers, views, validation, tests and setup instructions. Mark functionality not yet implemented explicitly rather than showing simulated probability values.

## 11. Minimal prototype acceptance scenarios

1. Create formulation with a disclosed ingredient and separate **possible impurity** assertion, each with its own source and disclosure status.
2. Import a reviewed public PFAS source file and keep its original IDs and reference/version.
3. Record a formulation laboratory result reported `<LOQ` without converting it to zero.
4. Create a textile material linked to a specific formulation version through a coating/finishing step.
5. Create a leather material with *unknown formulation input* and still record an independent material sample.
6. Assemble a dataset version for a **specific targeted panel**, excluding untested records from negative labels.
7. Attempt assessment without a validated model: system responds **INSUFFICIENT_EVIDENCE**, not a numerical risk score.
8. Restrict exporting confidential formulation details for users without authorization.

## 12. Developer handoff

Start with the scientific contract in [README.md](README.md), then implement this specification. If a requested feature is impossible without verified measured data, deliver its import, storage, validation and review workflow first; **do not emulate scientific probabilities using manually assigned risk weights**.
