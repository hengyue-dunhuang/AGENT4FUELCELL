# AGENT4FUELCELL

Agent-derived literature dataset for high-entropy alloy and multi-principal-element alloy electrocatalysts in fuel-cell-relevant reactions.

This repository accompanies the study **"Agent-driven discovery of advanced high-entropy alloy electrocatalysts for next-generation fuel cells"**. It provides the curated catalyst dataset, intermediate literature-mining records, query history, and supplementary material needed to inspect the search and extraction workflow.

## Repository Contents

| File | Description |
| --- | --- |
| `HEA_FUELCELL.csv` | Final curated dataset of 104 HEA/MPEA fuel-cell electrocatalyst records. |
| `processed_papers_with_titles.csv` | Candidate-paper metadata retained during literature mining, including paper identifiers and titles. |
| `all_query_history.csv` | Complete generated-query history from the Agent literature-search stage. |
| `SM.md` | Supplementary material documenting audit results, interpretation, prompts, schema, and extraction protocol. |

## Dataset Scope

The curated records focus on experimental high-entropy alloy, high-entropy nanomaterial, multi-principal-element alloy, or closely related multimetallic catalysts for fuel-cell-relevant electrocatalysis. Covered reaction classes include ORR, HOR, MOR, EOR, FAOR, AOR, fuel oxidation, and fuel-cell/MEA testing.

Each final record captures compact design and performance descriptors, including:

- catalyst composition, element list, element count, and atomic ratio;
- support, particle size, crystal structure, and synthesis method;
- reaction type, test mode, electrolyte, and benchmark catalyst;
- ECSA, half-wave potential, mass activity, specific activity, power density, and durability;
- bibliographic metadata and a short design-performance finding.

## Agent Workflow

The dataset was built with a staged Agent extraction workflow:

1. **Title-abstract prescreen** to identify papers likely related to HEA/MPEA fuel-cell electrocatalysts.
2. **Full-text relevance check** to enforce experimental data, multimetallic catalyst scope, fuel-cell relevance, and quantitative metric requirements.
3. **Structured feature extraction** using a fixed JSON schema with explicit `null` values for missing or non-applicable fields.
4. **Manual audit and correction** for records affecting reaction assignment, composition parsing, or quantitative performance analysis.

The supplementary material (`SM.md`) documents the prompts, fixed schema, parser traceability, and quality-control protocol.

## Quality Control

A random 30-record audit was checked manually against the original articles. The audit reported:

| Metric | Value |
| --- | ---: |
| Precision | 0.9703 |
| Recall | 0.9423 |
| F1 score | 0.9561 |
| Field-level extraction accuracy | 0.9159 |

The main residual error sources were incomplete durability descriptors, ambiguity in activity normalization, and missing details for multi-step synthesis routes. These cases were corrected where they affected the final analysis.

## Suggested Use

The repository is intended for:

- literature-level trend analysis of HEA/MPEA fuel-cell electrocatalysts;
- catalyst composition and reaction-type statistics;
- benchmarking data-driven catalyst-discovery workflows;
- machine-learning feature engineering and exploratory screening.

Please inspect the fields and source metadata before using the dataset for quantitative modeling. Reported electrochemical metrics may differ in reference scale, normalization basis, electrolyte, and test configuration across papers.

## Citation

If you use this dataset, please cite the associated manuscript:

> Agent-driven discovery of advanced high-entropy alloy electrocatalysts for next-generation fuel cells.

## License

No license file is currently included. Please contact the repository owner before redistributing or using the dataset in a commercial setting.
