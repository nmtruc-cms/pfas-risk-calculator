# PFAS-RISK LAB

**Research concept and methodological protocol · v0.2 · 14 October 2026**

**Purpose of this document:** Explain the research to scientists, chemical professionals, potential collaborators, and reviewers. The software implementation specification is in [STACK.md](STACK.md). This is an **exploratory research proposal**, not a validated PFAS risk model or conformity assessment service.

## 1. Research vision

PFAS-RISK LAB aims to quantify, with stated uncertainty, the probability of analytically defined PFAS occurrence and concentration exceedance in two connected sectors:

- **Sector A — Chemical formulations:** industrial chemical products, including ingredients, by-products, impurities, fluorine sources, and formulation functions.
- **Sector B — Materials:** initially **textiles and leather**, accounting for input chemicals, manufacturing operations, retention, removal, transformation, and external contamination. Other materials (e.g., electronics and automotive) may follow.

Rather than labeling a formulation “high” or “low” risk from a checklist, the project tests whether a **knowledge-informed Hierarchical Bayesian model** can predict carefully defined chemical outcomes from experimental data. Regulatory and industry guidance supplies *definitions or hypotheses* where relevant, but its qualitative risk classifications **are not ground-truth labels or numerical probabilities**.

**A successful model must answer:** *Probability of what, in which population, for which matrix/analytes/method, with how much uncertainty and on what evidence?*

## 2. Research questions

1. Given category, subcategory, use/function, composition, fluorine sources and evidence, how well can we estimate PFAS outcomes **in a chemical formulation**?
2. Given fiber/leather type, processes and finishes, how well can we estimate PFAS outcomes **in a material** without full supply-chain data?
3. How much does connecting **formulation → application → process → material** improve prediction over a material-only model?
4. Does **Hierarchical Bayesian partial pooling** improve out-of-sample calibration and uncertainty estimation compared with simpler baselines?
5. Do curated knowledge-graph features improve performance, and which additional evidence would reduce uncertainty most?

All claimed improvements are **hypotheses to test**, not assumed advantages of Bayesian methods or AI.

## 3. What is being predicted?

A predicted percentage is meaningless unless its *estimand* is specified.

| Research endpoint | Operational definition | Caution |
|---|---|---|
| **Panel positivity** (MVP primary) | Probability a sample is positive under a **named, versioned targeted-PFAS panel and reporting rule** | A limited panel cannot establish absence of every OECD-defined PFAS |
| **Specific-analyte exceedance** | `P(C_analyte > c | information, data)` for a stated matrix, mass basis, units and threshold `c` | Not automatically a regulatory nonconformity decision |
| **Named PFAS sum exceedance** | Probability a precisely specified sum exceeds `c`, with documented treatment of non-detects | Do not create an undefined “total PFAS concentration” |
| **Broad OECD-defined PFAS occurrence** | Probability of at least one OECD-2021 PFAS, only if the measurement/ascertainment model supports such an inference | Often **unresolved** with ordinary targeted testing |
| **Undeclared presence** (later) | Probability of presence inconsistent with a specified, verifiable declaration | Declaration, intentional use and contamination are different concepts |

**Total fluorine (TF), extractable/adsorbable organic fluorine (EOF/AOF), targeted PFAS and TOP assay are different analytical endpoints.** TF can arise from non-PFAS fluorine; targeted non-detects do not show that all PFAS are absent. Test results below LOD/LOQ are not zeros.

Neither PFAS occurrence nor concentration exceedance is by itself a quantitative human-health or ecological risk assessment. Exposure, dose and hazard would require separate models.

## 4. Two connected but statistically distinct sectors

### A — Formulation model

Key information: product version/batch; chemical category → subcategory → function; ingredient identities (CAS, EC, structures where meaningful); concentration ranges; intentionally added components, impurities, by-products and unknown composition; fluorinated and non-PFAS fluorine sources; provenance of each disclosure; analytical outcomes.

**Sampling unit:** formulation version × batch × specimen. Distinct versions and production batches must not be silently pooled.

### B — Material model

Initially textiles and leather: composition; dyeing, tanning, finishing, coating, printing, washing, drying and other ordered process steps; wet-chemistry conditions; recycled source; component/lot identity; associated analytical results.

Three evidence modes are required:

- **Material only:** no reliable formulation-level information, uncertainty remains explicit.
- **Connected pathway:** identified formulations and doses in particular processing steps.
- **Material plus observations:** analytical data update estimates while respecting each method's scope.

**Sampling unit:** material specification × lot × component/specimen. An entire shoe, fabric layer, coating and leather upper are not interchangeable matrices.

The two sectors **share chemical ontology and evidence concepts**, but do not automatically share occurrence rates, concentration distributions or model coefficients.

## 5. Chemical Knowledge Foundation: OECD and other public sources

We propose an evidence-provenance-aware **Chemical Knowledge Graph**, not a vector search system masquerading as a probability model.

### Candidate public sources and proposed research uses

| Source | Proposed use | What it cannot provide alone |
|---|---|---|
| [OECD PFAS topic hub](https://www.oecd.org/en/topics/sub-issues/risk-management-risk-reduction-and-sustainable-chemistry/per-and-poly-fluorinated-chemicals.html) | Navigate publications on PFAS identities, use categories, and sector-specific applications | Representative incidence or prevalence in formulations/materials |
| [OECD 2021 PFAS terminology](https://www.oecd.org/en/publications/reconciling-terminology-of-the-universe-of-per-and-polyfluoroalkyl-substances_6e5a1e2f-en.html) | Define structural/classification concepts, requiring careful handling of polymers, mixtures and unknown structures | An exhaustive static CAS list that automatically decides all structures |
| OECD global PFAS database (2018) and [NORMAN S25 / Zenodo record](https://zenodo.org/records/6348990) | Candidate substance identifiers, matching, deduplication, historical group membership; audit original source/version/license | Complete coverage of OECD 2021 definition or observed failure rates |
| [OECD PFAS fact cards](https://www.oecd.org/en/publications/fact-cards-of-major-groups-of-per-and-polyfluoroalkyl-substances-pfass_59e7ffc6-en.html) | Group → structure → function, fate/transformations and supporting references | Measured probabilities for an individual formulation |
| [OECD side-chain fluorinated polymer report](https://www.oecd.org/en/publications/synthesis-report-on-understanding-side-chain-fluorinated-polymers-and-their-life-cycle_e13559f7-en.html) | Precursor, polymer and possible transformation/fate relationships | Proof of a specific product's composition |
| [OECD fluoropolymer life-cycle report](https://www.oecd.org/en/publications/synthesis-report-on-understanding-fluoropolymers-and-their-life-cycle_35b035df-en.html) | Polymer-specific knowledge and sector expansion | One-to-one mapping from polymer name to target-panel result |
| OECD publications on coatings, lubricants and PFPEs | Non-repellent functions and sector-specific hypotheses | Prevalence estimates without a sampling denominator |
| EU Safety Gate/RAPEX, CASP, ECHA reports | Alert cases, pathways, confirmed concerns, targeted surveillance patterns | An unbiased population prevalence dataset or a negative control set |
| Participating laboratories and authorized research partners | **Measured pass/fail and quantitative results**, including non-detects and sampling design | Automatically representative data without careful sampling/validation |

**Data sourcing status:** These are *candidate* inputs. Each import must verify actual fields, update date, provenance, copyright/license, and the suitability of the data for the intended use. Public substance lists and reports are **knowledge or hypothesis data**, not statistical observations of PFAS occurrence.

### Knowledge-graph principles

Entities can include `SUBSTANCE`, `PFAS_GROUP`, `FORMULATION`, `INGREDIENT`, `IMPURITY`, `FUNCTION`, `PROCESS`, `SUBPROCESS`, `MATERIAL`, `MEASUREMENT`, `SOURCE`, `EXPERT_ASSERTION`.

Typed relationships include `HAS_COMPONENT`, `MAY_CONTAIN_IMPURITY`, `USED_IN`, `MAY_TRANSFORM_TO`, `MAY_TRANSFER_TO`, `MAY_BE_REMOVED_BY`, `SUPPORTED_BY`, `CONTRADICTED_BY`.

Every scientific assertion carries a **source, version, evidence strength, review status and context**. “May occur” is never converted into “confirmed present.” LLM-assisted extraction creates *proposals for expert verification*, not ground truth.

## 6. Core mathematical model: Hierarchical Bayesian inference

### 6.1 Probability of a defined observation

Let `y_i=1` mean the explicitly specified targeted panel returned a positive outcome for sample `i`. A basic hierarchical model is:

```text
y_i ~ Bernoulli(pi_i)
logit(pi_i) = alpha_sector[s(i)]
            + alpha_category[k(i)]
            + beta' x_i
            + u_group[g(i)]
alpha_category[k] ~ Normal(mu_sector[s(k)], sigma_category)
u_group[g]      ~ Normal(0, sigma_group)
```

`x_i` contains verified or explicitly missing formulation/material/process descriptors and relevant knowledge-derived features. Hierarchical **partial pooling** allows smaller categories to borrow statistical strength from related ones without asserting equal risk. Structure, group effects and prior distributions must be justified and tested for identifiability and sensitivity.

**Unobserved test outcome ≠ negative result.** A limited panel detects a defined set of analytes; if measurement sensitivity/specificity is relevant and estimable, represent the latent chemical state and measurement process separately.

### 6.2 Concentration distributions and censored data

For an identified analyte and sample matrix, a candidate extension is:

```text
z_i ~ Bernoulli(pi_i)
log(C_i) | z_i=1 ~ Normal(mu_i, sigma_i^2)
```

A structural-zero plus positive-concentration (hurdle/mixture) formulation is only a **candidate**. Alternatives must be tested. `<LOD`, detected `<LOQ`, quantified values and invalid/not-tested results use appropriate distinct likelihood/observation treatment. Do not substitute 0 or LOQ/2 by default.

Posterior results may report `P(C > c | evidence, study data)` and 95% credible intervals, **only for a defined analyte, sum or analytical endpoint**. A credible interval does not guarantee that an unvalidated model is well-calibrated.

### 6.3 Linked formulation → material fate/transfer

For one named PFAS analyte `a`, a first mass-balance hypothesis is:

```text
C_material,a ≈ [ sum_j (mass_used_j × C_formulation,j,a × retention_j,a) ]
               / material_mass + background_a
```

The retention term summarizes loss/removal for simple cases; formation, precursor transformation, partitioning and additional sources need mechanistic extensions. Inputs should be distributions when uncertain. All concentration units and mass bases must be reconciled. Material observations serve to **validate**, not merely restate, these assumptions.

### 6.4 What Bayesian inference contributes

- Express uncertainty in parameters and predictions rather than inventing one fixed point score.
- Combine independently documented expert **priors** with laboratory likelihoods; priors are not measured prevalence.
- Model variation across categories, materials and processes; calculate posterior predictive checks.
- Test whether the model is calibrated and useful **out of sample**, including unseen suppliers/categories.

Knowledge-graph nodes or embeddings can become predictors or structural priors, but **graph-link similarity alone is not a calibrated occurrence probability**. LLM and vector retrieval are tools for sourcing evidence, not substitutes for the Bayesian likelihood.

## 7. Expert collaboration and dataset development

Invite expertise in analytical chemistry/PFAS, formulation chemistry, textile wet processing, leather processing, statistical modelling, and data governance. The proposed protocol:

1. Agree on target population, chemical/analytical definitions, pilot sectors and specific panel version.
2. Independently review causal/pathway hypotheses and sources; capture disagreements and provenance.
3. Elicit priors as distributions with explicit reference populations, intervals and rationales, **not** arbitrary High/Low labels.
4. Compile authorized laboratory samples including positive, negative, censored and missing outcomes and the reason each sample was selected.
5. Deliberately include random/representative or independently sampled subsets to evaluate selection bias from targeted testing.
6. Freeze datasets and protocol versions before model comparison and external validation.

**Confidentiality:** Internal bluesign, customer and supplier data can only be used with explicit permissions, appropriate contracts/consents and access controls. Research-public and restricted datasets remain separate. Avoid publication or export of identifiable proprietary compositions.

## 8. Scientific validation requirements

Competing models: empirical frequency, penalized logistic regression, **Hierarchical Bayesian**, calibrated gradient-boosted trees, and optionally graph-enhanced/GNN challengers when warranted by dataset scale.

Evaluate: log loss, Brier score, calibration/reliability plots, AUROC/AUPRC where appropriate, recall at defined operational constraints, subgroup performance, posterior predictive checks, coverage of uncertainty intervals, and **external validation** by time, supplier/product family, laboratory or site.

Prevent leakage across the same product/formulation/lot under different IDs. Compare on the **same held-out test set**; perform ablation experiments for ontology and cross-sector features. Report sampling bias, missingness, scope of analytical panels and concept drift. Treat predictions outside supported data distributions as uncertain or out of domain.

Risk bands, if later desired for communication, must be transparently derived from validated probabilities and separately justified decision costs—not imported from qualitative (High/Medium/Low) classifications.

## 9. Research phases and deliverables

| Stage | Research output | Exit requirement |
|---|---|---|
| **R0 — Protocol** | Research questions, target definition, study population, ontology and measurement schema | Expert-approved protocol and data dictionary |
| **R1 — Knowledge foundation** | OECD candidate substance registry, cited assertions, review workflow | Provenance audit and sample manual review |
| **R2 — Observational dataset** | Independent Formulation and Material datasets plus linked pathways where authorized | QC of units, LOD/LOQ, sampling and labels |
| **R3 — Baseline models** | Logistic/frequency baselines and validation report | Leakage-resistant splits and calibration report |
| **R4 — Bayesian model** | Versioned hierarchical posterior model and model card | Convergence, posterior predictive checks and external evaluation |
| **R5 — Transfer/graph experiments** | Connected-pathway and graph-feature ablation results | Evidence of incremental utility or clear negative finding |

Research claims and manuscript results require a properly defined and validated dataset. Synthetic data may exercise implementation only, and must never be mixed with real data in any scientific result.

## 10. Scientific outputs and limitations

Minimum research output per assessment: **what** was predicted, **for which** population/matrix/panel/threshold, posterior distribution and interval, model/data versions, data completeness, evidence sources, uncertainty warnings, and limitations. The system can legitimately output **“insufficient basis to estimate”**.

The project is not a regulatory certification tool, a toxicological exposure model, or a claim that an OECD name/CAS list captures every PFAS. Findings depend on representativeness, measurement validity, model assumptions, and permissible access to datasets.

## 11. Reading and companion implementation document

- [OECD PFAS topic hub](https://www.oecd.org/en/topics/sub-issues/risk-management-risk-reduction-and-sustainable-chemistry/per-and-poly-fluorinated-chemicals.html)
- [OECD 2021 terminology report](https://www.oecd.org/en/publications/reconciling-terminology-of-the-universe-of-per-and-polyfluoroalkyl-substances_6e5a1e2f-en.html)
- [NORMAN / Zenodo OECDPFAS S25](https://zenodo.org/records/6348990)
- [OECD PFAS fact cards](https://www.oecd.org/en/publications/fact-cards-of-major-groups-of-per-and-polyfluoroalkyl-substances-pfass_59e7ffc6-en.html)
- [EU Safety Gate](https://ec.europa.eu/safety-gate-alerts/screen/search)

For application architecture, database, endpoints, validation and developer tasks, see **[STACK.md](STACK.md)**. Technology choices must implement—not redefine—the scientific contract above.
