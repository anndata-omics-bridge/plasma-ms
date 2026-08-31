# EuBIC-MS Hackathon 2027 — proposal, paste-ready

> Paste each block into the matching field of the **Hackathon Proposals** discussion template at
> <https://github.com/EuBIC/EuBIC2027/discussions/new?category=hackathon-proposals>.
>
> Public project plan, datasets, panels, and references:
> <https://github.com/anndata-omics-bridge/apb-plasma>.

---

## Title

Quality control of biological plasma samples and their MS or Olink measurements with APB

---

## Abstract

[The AnnData Proteomics Bridge (APB)](https://github.com/anndata-omics-bridge) converts
quantitative-proteomics output tables into structured datasets. It preserves tool-specific columns
and links protein-, peptide-, and ion-level evidence; for DIA, it also links fragments. It stores
data as AnnData/MuData, Parquet, or DuckDB, and joins related datasets or data levels in multimodal
containers. APB grew from ProteoBench's need to compare results from several quantification tools.
It also enriches MuData: `apb fasta` adds FASTA annotations, `apb proteobench` adds feature-level
statistics and dataset-level scores.

Participants will extend APB to assess studies containing hundreds of biological plasma samples. The
project examines pre-analytical and biological sample-quality signals, cellular contamination,
expected-protein coverage, marker precision, abundance dynamic range, batch-associated shifts, and
agreement with reference datasets.

Participants will define, implement, and visualise candidate quality measures, starting from
documented panels such as those of [Geyer et al. (2019)](https://doi.org/10.15252/emmm.201910427),
who used counted dilution series to identify erythrocyte and platelet contamination markers. In
collaboration with the [Olink/proteoform project](https://github.com/EuBIC/EuBIC2027/discussions/11),
participants will test protein-level measures on a second modality and separate shared from
platform-specific measures. The prepared APB datasets can be read from Python, R, Julia, or
JavaScript.

## Project Plan

**Before the event**, the organisers will quantify representative label-free DDA and DIA plasma
datasets with FragPipe and DIA-NN, convert the resulting tables into MuData with APB, and prepare a
small fixture, reference MuData datasets, and plasma panels. Together with the discussion 11 team,
they will arrange either an accessible Olink dataset or an executable data hand-off.

**During the four project days**, participants will:

1. define each quality measure's inputs, output, interpretation, data level, and validation dataset;
2. implement panel coverage, marker precision, and reference-dataset comparison;
3. define candidate cellular-contamination scores from documented panels and implement at least one
   for evaluation;
4. use `apb-plasma` to combine a new measurement, reference MuData datasets, and panels;
5. store the resulting annotations, scores, statistics, and relationships in an enriched MuData
   file;
6. evaluate each applicable core protein-level measure on at least two MS datasets and one Olink
   dataset;
7. record whether each measure requires a platform-specific interpretation; and
8. add one agreed quality-control visualisation using Vitessce, Plotly, or another JavaScript-based
   tool.

Participants can contribute plasma datasets, panels, measure definitions, implementations, or
visualisations.

**During and after the hackathon**, participants will document the measure definitions and
validation results, and present the workflow and findings to the EuBIC-MS community. The software,
panels, validation outputs, and enriched MuData examples will be published in the
[`apb-plasma` repository](https://github.com/anndata-omics-bridge/apb-plasma), followed by a
manuscript on the cross-dataset and cross-modality evaluation.

## Technical Details

APB is written in Python and stores results as AnnData/MuData, Parquet, or DuckDB, so participants
can work from Python, R, Julia, or JavaScript. The organisers will quantify the prepared datasets
with FragPipe and DIA-NN before the event.

The Python `apb-plasma` tool to be developed during the hackathon will read three panel types:
plasma biomarkers, cellular contamination markers, and expected plasma proteins. Reference
MuData datasets link protein-, peptide-, and ion-level information; DIA datasets may also link
fragments. Participants will define the measures themselves. No formula is fixed in advance, but
each measure should use only its required matrices and annotations, and should assume no common raw
scale for MS and Olink: a cross-modality measure states either a documented normalisation or a
scale-free comparison such as rank concordance. The enriched MuData will expose `layers`, `var`,
`varm`, `obs`, `obsm`, `obsp`, and `uns`. The organisers will provide an in-browser MuData explorer,
which participants will extend into a plasma-specific view of the file that `apb-plasma` generates,
using Vitessce, Plotly, or other JavaScript libraries.

```mermaid
flowchart LR
    NEW["New biological plasma measurement<br/>MS MuData or mapped Olink table"]
    REF["Reference MuData datasets<br/>protein · peptide · ion<br/>fragment for DIA"]
    PANELS["Plasma panels in Parquet<br/>biomarkers · cellular contaminants · expected proteins"]
    APB["Python apb-plasma tool<br/>annotate · compare · score"]
    OUT["Enriched MuData<br/>annotations · scores · statistics · relationships"]
    VIEW["JavaScript viewer"]

    NEW --> APB
    REF --> APB
    PANELS --> APB
    APB --> OUT --> VIEW
```

Candidate datasets are public deposits. Accessions, designs, validation roles, and availability are
listed in
[`HACKATHON.md` section 6](https://github.com/anndata-omics-bridge/apb-plasma/blob/main/HACKATHON.md#6-candidate-datasets-and-validation-roles),
with the panels, measure definitions, and four-day plan.

## Contact information

Witold E. Wolski — Functional Genomics Center Zurich (FGCZ), ETH Zurich / University of Zurich;
SIB Swiss Institute of Bioinformatics — witold.wolski@fgcz.uzh.ch

Sam van Puyenbroeck — CompOmics, VIB-UGent Center for Medical Biotechnology; Department of
Biomolecular Medicine, Ghent University — sam.vanpuyenbroeck@ugent.be

Tobias Kockmann — Functional Genomics Center Zurich (FGCZ), ETH Zurich / University of Zurich —
tobias.kockmann@fgcz.ethz.ch


