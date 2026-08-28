# Reference panels and profiles

This directory contains machine-readable inputs for the plasma quality-control measures described in
[`HACKATHON.md`](../HACKATHON.md). Panel membership and score definition are separate. A panel lists
candidate proteins; it does not prescribe how their quantitative values must be combined.

## Geyer et al. 2019

[`geyer2019_panels.csv`](geyer2019_panels.csv) contains the erythrocyte/haemolysis, platelet, and
directional coagulation panels extracted from Supplementary Table EV2 of
[Geyer et al. 2019](https://doi.org/10.15252/emmm.201910427). See the repository
[`README`](../README.md#3-the-quality-marker-panels) for the extraction checks and known discrepancy
between the paper and its reference implementation.

## Korff et al. 2025

The following files were derived from the supplementary data of
[Korff et al. 2025](https://doi.org/10.1038/s44321-025-00309-0):

| File | Rows | Source | Intended use |
| --- | ---: | --- | --- |
| [`korff2025_panels_workflow_independent.csv`](korff2025_panels_workflow_independent.csv) | 90 | EV3 | 30 markers each for platelet, erythrocyte, and PBMC contamination |
| [`korff2025_panels_workflow_specific.csv`](korff2025_panels_workflow_specific.csv) | 450 | EV4 | 30 markers for each cell type and each of five plasma workflows |
| [`korff2025_plasma_reference_profile.csv`](korff2025_plasma_reference_profile.csv) | 10,092 | EV1 | Pure-plasma abundance profiles for the five workflows |
| [`korff2025_top30_plasma.csv`](korff2025_top30_plasma.csv) | 150 | EV1 | The 30 most abundant pure-plasma protein groups per workflow |

The source article and its associated data are available under CC BY 4.0 and CC0 respectively, as
stated in the article. These CSVs canonicalise column names and omit explanatory trailing rows that
do not contain a protein accession. The biological values are otherwise retained from the
supplementary tables.

The workflow-independent and workflow-specific panels were selected from measured pure-cell and
pure-plasma material. Platelet and erythrocyte candidates required a cell/plasma fold change above
1,000; PBMC candidates required a fold change above 100. The publication also applied cell-type
specific intensity and CV thresholds, required at least two precursors, ordered candidates by
cell-type abundance, and retained 30 proteins. Consult the publication before changing these rules.

A panel derived under one preparation workflow is an analogy for another, not an identity. Panel
membership can be evaluated across workflows and platforms, but the absolute intensities and
derived score thresholds are not transferable without validation. Always retain the observed
member count and fraction beside a panel-based score.

## Integrity

SHA-256 checksums of the committed CSVs:

```text
bbf4833113305d67db06b34da930551301b3e10859c93f8dece17269e0d13bcc  korff2025_panels_workflow_independent.csv
5a06e2baafec373fd3128248a535206fef3c2c2a90485206bfd56c24b769d80a  korff2025_panels_workflow_specific.csv
503f786cf152b130d3e3a11b099040cf16f4513eed5d7610077627369d7237a5  korff2025_plasma_reference_profile.csv
d6c8f24bea8aa057376296c46722f1517d362db212c2cc5b4f868b0520cce4f0  korff2025_top30_plasma.csv
```
