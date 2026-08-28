# plasma-ms

Reference material for **plasma mass-spectrometry proteomics QC** on the
[anndata-omics-bridge](https://github.com/anndata-omics-bridge) spine: datasets, quality-marker
panels, and the QC metric catalogue.

This repository is the supplement to a proposal for the
[EuBIC-MS Hackathon 2027](https://eubic-ms.org/events/hackathon-2027/). It exists so the proposal
itself can stay short, and so that anyone who wants to bring **their own plasma data or their own
metric** has one place to look.

> **We are open to new data and new metrics.** The dataset table below is a starting set, not a
> closed list. If you have a plasma cohort — any vendor, DDA or DIA, public or contributed — or a QC
> metric you think belongs in the catalogue, open an issue. Bringing a dataset that breaks a metric
> is the most useful contribution there is.

---

## 1. Why plasma QC needs a quantitative container

A haemolysed sample, a platelet-contaminated draw, or a coagulation-activated tube produces a plasma
protein table that looks well-formed and is scientifically worthless. Instrument-level QC catches
none of it.

The community QC standard — HUPO-PSI **mzQC** and its controlled vocabulary — has a category for
every kind of QC metric, and the ones that would cover this are **empty**. Counted against
`psi-ms.obo`, `data-version: 4.1.259` (23 June 2026), the QC branch holds **197 terms**:

| `has_metric_category` | Category | Terms |
| --- | --- | ---: |
| `MS:4000009` | ID free metric | 104 |
| `MS:4000012` | single run based metric | 74 |
| `MS:4000022` | MS2 metric | 62 |
| `MS:4000008` | ID based metric | 41 |
| `MS:4000021` | MS1 metric | 40 |
| `MS:4000013` | multiple runs based metric | 7 |
| **`MS:4000010`** | **quantification based metric** | **0** |
| **`MS:4000023`** | **sample preparation metric** | **0** |
| **`MS:4000073`** | **QC sample metric** | **0** |

`MS:4000010` appears exactly once in the 1.17 MB file — as its own definition. Plasma QC is the
intersection of the two empty categories that matter: quantitative measurements about how the
*sample* was handled.

Two further gaps, measured rather than asserted:

- There is **no QC term for the coefficient of variation of a protein across replicates**. The only
  CV term in the ontology, `MS:1003286`, describes a spectral-library peak attribute.
- The only two protein-level QC terms have a vendor in their *names* — `MS:4000090` / `MS:4000091`,
  *"principal component analysis of MaxQuant's protein group raw/lfq intensities"*.

---

## 2. "Contaminant" in plasma means several different things

The word does too much work. At least two distinct classes matter, and conflating them is the error:

| Class | Examples | Origin | Covered by a list today? |
| --- | --- | --- | --- |
| **Handling / process contaminants** | keratin, trypsin autolysis products, BSA, streptavidin and protein A/G from columns, polymers and detergents, column bleed | the lab, not the donor | Yes — the cRAP family and the Hao universal library, and they do it reasonably well |
| **Matrix contamination** | erythrocyte lysis, platelet activation, coagulation cascade, depletion or enrichment breakthrough, extracellular-vesicle content, tissue leakage, anticoagulant and storage effects | the donor and the draw — **real human protein, in the wrong compartment** | Barely. Three panels exist (§3). For most of the rest we found none. |

The second class is what ruins plasma biomarker studies, and **it is structurally invisible to a
contaminant FASTA**. The offending proteins are genuine human proteins that belong in the sample —
they are simply from the wrong compartment, or present because the draw went badly. You cannot
exclude them by sequence, only by **quantitative pattern**. That is why this belongs to a QC layer
over a quantitative container rather than to a search database.

### Getting a matrix panel: measure it

Geyer *et al.* is the worked precedent for the *cellular* subset: spike erythrocytes and platelets
into clean plasma across a counted dilution series and see which proteins rise — nine steps over a
10⁷ range **with cell counting** — selecting *"the 30 most abundant proteins with CVs below 30% and at
least a 10-fold higher expression level in the contaminating cell type than in plasma."* The
coagulation panel came from a plasma-versus-serum contrast instead, because coagulation is a process
rather than a cell.

That is a reproducible protocol rather than a lookup table, which is the useful part: it can be
re-derived for a new matrix, a new enrichment workflow or a new instrument, and the *result* is what
gets versioned and attached to the data.

**Three panels is not coverage.** Depletion and enrichment breakthrough, EV content, tissue leakage,
anticoagulant chemistry and storage-driven proteolysis are all matrix contamination, and we found **no
published machine-readable panel** for any of them — an absent search result, not a proven absence.
Deriving one more by the same protocol, against the bead-enrichment data in `PXD063572`/`PXD063593`,
is a well-shaped piece of work. **If you know of a panel we have missed, please open an issue.**

### The two classes collide

Applying the handling-contaminant list to plasma deletes matrix markers:

- Appending Cambridge CCP cRAP to a plasma search and then filtering contaminants **would delete 9
  of the 99 Geyer plasma QC markers, including 6 of the top haemolysis markers.**
- **All three cRAP-family databases list human albumin as a contaminant** — GPM `>sp|ALBU_HUMAN|`,
  CCP `>sp|cRAP002|P02768|ALBU_HUMAN`, MaxQuant `>P02768-1 … Gene_Symbol=ALB`. In plasma, albumin is
  the dominant component of the analyte.
- In Geyer's own Table EV2, **5 of the 91 marker protein groups contain a MaxQuant `CON__` bovine
  member** (e.g. `PFN1 ← CON__P02584`).
- The Hao-lab universal library is not plasma-safe either: **26 of its 381 entries are bovine
  orthologues of core human plasma proteins**, and **193 of 381 (50.7%) are skin or hair keratin**.

Entry counts, by `grep -c '^>'`:

| List | Entries | Licence | Note |
| --- | ---: | --- | --- |
| GPM cRAP | 116 | GPM terms | HTTP URL dead; only `ftp://ftp.thegpm.org/fasta/cRAP/crap.fasta`. A Zenodo copy is byte-identical, md5 `c1640d4054ec771d05c6f4493307f29f`. |
| Cambridge CCP cRAP | 125 | — | |
| MaxQuant `contaminants.fasta` | 245 | — | Decayed: 152 of 245 (62%) carry no `Gene_Symbol=` and no `Tax_Id=`; 27 have no accession at all. |
| Hao universal contaminant library | 381 | **none declared** | Best content for class 1, **cannot be vendored** — the GitHub API reports `license: None`. |

**Conclusion:** the goal is not a better contaminant FASTA. It is to **keep both classes, keep them
separate, keep them versioned, and attach them to the data as annotations a metric can read.**

## 3. The quality-marker panels

[`data/geyer2019_panels.csv`](data/geyer2019_panels.csv) — 91 rows, columns
`panel, gene_names, protein_ids, protein_names`.

| Panel | Members |
| --- | ---: |
| erythrocyte / haemolysis | 30 |
| platelet | 30 |
| coagulation | 31 |

Extracted from Supplementary Table EV2 of Geyer *et al.* 2019 (file `EMMM-11-e10427-s003.xlsx`,
sheet `Table EV2`), obtained via the Europe PMC supplementary endpoint for `PMC6835559`. The PMC and
publisher paths both return download interstitials or stubs; Europe PMC works.

**Validation of the extraction:** the paper states the panels *"overlap by just two proteins
(actin/ACTB and glyceraldehyde-3-phosphate dehydrogenase/GAPDH)"*. Computed erythrocyte ∩ platelet =
`{ACTB, GAPDH}` — exactly. Also: platelet ∩ coagulation = `{F13A1, PPBP, THBS1}`, erythrocyte ∩
coagulation = `{}`.

### Three things to know before using them

1. **The coagulation panel is directional, not a sum.** Table EV2 carries a plasma-versus-serum
   t-test difference whose sign splits the 31 members: **12 plasma-elevated** (`FGB +2.505`,
   `FGG +2.643`, `FGA +1.593`, `F13A1`, `SERPINC1`, `F2`, `ECM1`, `DSP`, `WDR1`, `SERPINA5`, `F11`,
   `APOC3`) and **19 serum-elevated** (`PPBP −0.715`, `THBS1 −0.669`, `LTF −0.852`, `GP1BA`, `CLU`,
   `ATRN`, `GP5`, `C1RL`, `MAN1A1`, `KNG1`, `BCHE`, `GPLD1`, `MASP1`, `TENM4`, `HSPG2`, `CDH5`,
   `CST3`, `PROC`, `PF4;PF4V1`). A correct coagulation index is a **contrast**.
2. **The haemoglobins are the weakest haemolysis markers, not the strongest.** Enrichment as log10
   erythrocyte/plasma: `CAT` 3.81, `CA2` 3.50, `PRDX2` 3.33, `CA1` 2.98 — but `HBA1` only 1.72 and
   `HBB` 1.59.
3. **The paper and its own reference implementation disagree on one panel.**
   [`MannLabs/Quality-Control-of-the-Plasma-Proteome`](https://github.com/MannLabs/Quality-Control-of-the-Plasma-Proteome)
   (Apache-2.0, last commit 5 April 2022) ships `data/Marker List.xlsx`. Erythrocyte 30/30 identical,
   platelet 30/30 identical, **coagulation shares only 19 members** — 12 paper-only
   (`ATRN CDH5 CST3 DSP GP5 GPLD1 LTF MAN1A1 MASP1 PF4 PROC WDR1`), 11 code-only
   (`ALCAM APOA2 C1S C3 CNTN3 FLNA HYDIN ITIH2 MET TLN1 TNXB`). **Reconciling these is open work.**

**The index definition**, to implement verbatim rather than reinvent: the summed intensities of a
panel's proteins divided by the summed intensities of all quantified plasma proteins.

Also worth noting: Table EV2 lists 30/30, while the Figure 2 legend says *"29 quality markers"* for
each; the preprint records that `NIF3L1` was excluded for inconsistent identification. The CSV
carries the **file** counts.

---

## 4. Datasets

Licence is a **note, not a filter** — PRIDE data is publicly reusable whether the submitter ticked
CC0 or left the default *EBI terms of use*.

| Accession | Study | Design | Software / instrument | Licence |
| --- | --- | --- | --- | --- |
| [`PXD063572`](https://www.ebi.ac.uk/pride/archive/projects/PXD063572) + [`PXD063593`](https://www.ebi.ac.uk/pride/archive/projects/PXD063593) | Geyer *et al.* 2025, *EMBO Mol Med* — pre-analytical drivers of bias in **bead-enriched** plasma proteomics | Pre-analytical variables × bead-based enrichment; PRIDE keywords `Cellular contamination`, `Sample quality` | DIA-NN / Orbitrap Astral | CC0 |
| [`PXD054073`](https://www.ebi.ac.uk/pride/archive/projects/PXD054073) | Multi-centre longitudinal QC of MS proteomics in plasma and serum | 8 centres × 3 timepoints × DDA (544) and DIA (378); PROCAL iRT in every sample; HeLa QC runs; **the paper names the runs it excluded as QC failures** | MaxQuant + DIA-NN | CC0 |
| [`PXD011749`](https://www.ebi.ac.uk/pride/archive/projects/PXD011749) | Geyer *et al.* 2019 — sample-related biases | **9-step dilution series over 10⁷ with cell counting**; reference proteomes of 5 blood fractions from 20 individuals | MaxQuant / Q Exactive | CC0 |
| [`PXD002854`](https://www.ebi.ac.uk/pride/archive/projects/PXD002854) | Geyer *et al.* 2016, *Cell Systems* — assess human health and disease | Plasma, serum, erythrocyte; **three community-curated SDRFs** (84 / 73 / 17 rows) | MaxQuant / Q Exactive | EBI terms of use |
| [`PXD009348`](https://www.ebi.ac.uk/pride/archive/projects/PXD009348) | Geyer *et al.* 2018, *Cell Systems* — bariatric surgery, inflammatory and lipid markers | Longitudinal intervention | MaxQuant / Q Exactive | EBI terms of use |
| [`PXD004242`](https://www.ebi.ac.uk/pride/archive/projects/PXD004242) | Geyer *et al.* 2016, *Mol Syst Biol* — weight loss, apolipoprotein family | Longitudinal intervention | MaxQuant / Q Exactive | EBI terms of use |
| `PXD062274` / `PXD062275` / `PXD062313` | Mann lab — simplified perchloric acid workflow with neutralisation (PCA-N) | `PCA_*` versus `NEAT_*` at biological / technical / analytical replicate level, plus `DIA_methods` and `FAIMS_CV`; a 2.5 GB `Cohort_QC_samples` archive | DIA-NN / Orbitrap Astral | CC0 |
| [`PXD029009`](https://www.ebi.ac.uk/pride/archive/projects/PXD029009) | COVID plasma cohort, Scanning SWATH | **Curated 1189-row SDRF** (868 plasma, 559 individuals). Caveat: `age`, `sex`, `BMI` are `not available` for all rows | DIA-NN | CC0 |
| [`PXD025752`](https://www.ebi.ac.uk/pride/archive/projects/PXD025752) | Time-resolved COVID plasma | **Fragment-level columns in the deposited report** (`Fragment.Quant.Raw`, `Fragment.Correlations`); `batch_info` with `Sample.Type` incl. `SP.QC.Plasma`, `MS.QC` | DIA-NN | CC0 |
| `PXD056598` | ProteoBench PYE (Plasma / Yeast / E. coli) benchmark | Known spike-in ratios | multiple | CC-BY-4.0 |

**Practical note.** Several of these deposit search results alongside very large raw sets — for
`PXD054073` the 252 search archives are **12.25 GB** against 1038 GB of raw. Start from the search
results.

**Not yet resolved:** [Kverneland, Østergaard & Olsen *et al.*, *Benchmarking enrichment and
depletion methods for quantitative plasma proteomics*](https://www.biorxiv.org/content/10.1101/2025.11.03.686186v1.full)
(bioRxiv, Nov 2025) compares Top14 depletion, perchloric acid, SAX-bead and ultracentrifugation EV
enrichment and neat plasma across platelet-rich plasma, platelet-poor plasma and serum — 72 donors,
Astral nDIA, DIA-NN 2.2.0 — and **correlates 10 proteins against a hospital Cobas clinical
analyser**. No PRIDE accession was found in the full text. If you know it, please open an issue.

---

## 5. The QC metric catalogue, by data level

The level a metric needs is what organises the work — and what makes the metrics that need
precursor-ion or fragment evidence impossible to compute from a protein table alone.

| Level | Metrics |
| --- | --- |
| **Sample preparation** (`MS:4000023`, empty) | erythrocyte / haemolysis index; platelet index; coagulation contrast; contaminant abundance fraction; missed-cleavage rate; semi-tryptic fraction |
| **Quantification** (`MS:4000010`, empty) | replicate CV distribution and median CV; data completeness per sample and per feature; dynamic range (P90 − P10, log10); albumin dominance / top-N signal share; biomarker-panel coverage and per-panel CV |
| **Precursor ion** | q-value distribution and decoy separation; retention-time stability and iRT residuals; charge-state distribution; match-between-runs transfer rate |
| **Fragment** | peak-shape similarity; transition-ratio consistency; interference / co-isolation; spectral-library similarity |
| **Run** | TIC, MS1/MS2 counts, injection time, gradient stability |
| **Cohort** | injection-order drift; batch structure versus PCA; RLE plots; pooled-QC trending |

The fragment row is not hypothetical: the published metrics that provably cannot exist above fragment
level — peak-shape similarity, mean profile of relative areas, spectral-library similarity — are the
Avant-garde curation metrics (Vaca Jacome *et al.* 2020,
[10.1038/s41592-020-00986-4](https://doi.org/10.1038/s41592-020-00986-4)).

---

## 6. Related work, honestly

Putting quantitative proteomics into AnnData/MuData is **not novel**. Five active projects do it, and
three ship QC metrics: `alphapepttools` (MannLabs, Apache-2.0), `mulink` (MIT), `msmu` (BSD-3),
`proteopy` (Apache-2.0), `qpx` (Apache-2.0). What distinguishes this work is narrower and testable:
**fragment level** (`alphapepttools` stops at precursors; `msmu` and `proteopy` contain no occurrence
of `fragment`; `qpx` explicitly collapses fragment rows), **plasma panels**, **non-Python access**,
and **column preservation**.

Existing QC tools cover the run and identification levels well and should be reused, not rebuilt:
PTXQC, rawDiag / rawrr, QCloud2, MSstatsQC, Skyline AutoQC + Panorama, pmultiqc. Note that **PTXQC
writes only a "mockup" mzQC file — the metric values are not exported — and its maintainer invites
pull requests.**

`QFeatures` / `SummarizedExperiment` (Bioconductor) already models assay hierarchies with
`AssayLinks`, including hierarchy-aware subsetting. Any container-side claim has to answer that.

---

## 7. References

Verified against Crossref by DOI.

- Geyer PE *et al.* (2019). Plasma Proteome Profiling to detect and avoid sample-related biases in
  biomarker studies. *EMBO Mol Med* 11:e10427. [10.15252/emmm.201910427](https://doi.org/10.15252/emmm.201910427)
- Geyer PE *et al.* (2021). Plasma Proteomes Can Be Reidentifiable and Potentially Contain Personally
  Sensitive and Incidental Findings. *MCP* 20:100035. [10.1074/mcp.ra120.002359](https://doi.org/10.1074/mcp.ra120.002359)
- Bielow C *et al.* (2024). Communicating Mass Spectrometry Quality Information in mzQC. *JASMS*
  35:1875–1882. [10.1021/jasms.4c00174](https://doi.org/10.1021/jasms.4c00174)
- Bielow C, Mastrobuoni G, Kempa S (2015). Proteomics Quality Control. *JPR* 15:777–787.
  [10.1021/acs.jproteome.5b00780](https://doi.org/10.1021/acs.jproteome.5b00780)
- Frankenfield AM *et al.* (2022). Protein Contaminants Matter. *JPR* 21:2104–2113.
  [10.1021/acs.jproteome.2c00145](https://doi.org/10.1021/acs.jproteome.2c00145)
- Vaca Jacome AS *et al.* (2020). Avant-garde. *Nat Methods* 17:1237–1244.
  [10.1038/s41592-020-00986-4](https://doi.org/10.1038/s41592-020-00986-4)
- Dai C *et al.* (2021). A proteomics sample metadata representation. *Nat Commun* 12:5854.
  [10.1038/s41467-021-26111-3](https://doi.org/10.1038/s41467-021-26111-3)
- Anderson NL (2010). The Clinical Plasma Proteome. *Clin Chem* 56:177–185.
  [10.1373/clinchem.2009.126706](https://doi.org/10.1373/clinchem.2009.126706)
- Devreese R, Jachmann C, … Wolski WE, Webel H, *et al.* (2025). ProteoBench: the community-curated
  platform for comparing proteomics data analysis workflows. Preprint.
  [10.64898/2025.12.09.692895](https://doi.org/10.64898/2025.12.09.692895)

PSI-MS controlled vocabulary: `psi-ms.obo`, `data-version: 4.1.259`, 23 June 2026 —
<https://github.com/HUPO-PSI/psi-ms-CV>

---

## Contributing

Open an issue. The most valuable contributions, in order:

1. **A plasma dataset the current metrics handle badly** — especially a vendor or enrichment workflow
   not represented above.
2. **A metric definition** with its data level, required inputs, and what artefact it detects.
3. **A reconciliation** of the coagulation panel discrepancy in §3.
4. **An implementation** in any language that reads AnnData/MuData.

## Licence

**CC BY 4.0** for everything in this repository — the panels, the metric catalogue, and the prose.
This is reference data, not software, and a software licence on a dataset creates exactly the reuse
ambiguity the panels exist to remove. Attribution is the only condition: cite this repository.

**MIT** for any scripts added here later, so code can move freely between this repository and the
rest of the [anndata-omics-bridge](https://github.com/anndata-omics-bridge) spine, which is MIT
throughout.

**Third-party reference material retains its own terms, and CC BY 4.0 here does not override them.**
`data/geyer2019_panels.csv` is derived from the supplementary tables of Geyer et al. 2019 and must
be cited as that paper, not as this repository; see §3 and the references in §7. The licence column
in §4 and the table in §2 record the terms of every other external source.
