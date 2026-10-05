# Assessment: `NASA-IMPACT/sddo` as a Source for an SDE Knowledge Graph

**Repo:** https://github.com/NASA-IMPACT/sddo
**Assessed:** 2026-09-02 · commit `d80ee46` ("Move mission and broaden def")
**Question:** Is there useful material here for building a knowledge graph to use with the NASA Science Discovery Engine (SDE)?

---

## 1. Bottom line

**Yes — but not the part the repo advertises.**

The *released* ontology (`sddo.owl`) is thin: 50 native classes, 27 relationship axioms, no domain/range constraints, no instance data. On its own it is not a usable KG schema.

The genuinely valuable material is **buried in `src/scripts/cxlConverter/`** — the working files behind the ontology. These hold three hand-built conceptual models that never made it into the release:

| Artifact | Content | Why it matters for SDE |
|---|---|---|
| SMD Conceptual Information Model | 106 concepts, **90 propositions** | A cross-SMD discovery model, including a **DiscoveryCriterion facet taxonomy** — absent from the released OWL |
| BIO Conceptual Model | ~106 concepts, **99 propositions** | Per-entity discovery facets with concrete field names (Species, Payload, Mission, Strain, ResearchCategory…) |
| LSDA → SDDO crosswalk | **129 propositions** | A complete, worked archive-metadata-to-ontology mapping at the *field* level |

Think of it like a house: the published ontology is the finished facade, but the architect's drawings in the back room are what you actually need — and the drawings are much more detailed than the facade.

**Recommended use:** mine it as a **design reference and vocabulary seed**, not as an importable ontology. Highest-value single item is the DiscoveryCriterion facet model.

---

## 2. What this repo actually is

The Science Data Discovery Ontology (SDDO) is an OBO-style ontology intended to give NASA's Science Mission Directorate a semantic foundation for cross-discipline data discovery (heliophysics, Earth science, planetary, astrophysics, biology, astrobiology, physical science).

It was scaffolded with the [INCATools ontology-starter-kit](https://github.com/INCATools/ontology-starter-kit), which means it ships a full ODK toolchain: ROBOT `Makefile`, 24 SPARQL QC queries, CI workflows, import modules, and release products in OWL/OBO/OBO-JSON.

Ontology authoring credit in the concept maps goes to **Ruth Duerr** and **Peter Fox**; commits are largely from **Dan Berrios**.

### Structural facts (measured, not claimed)

| Metric | Value |
|---|---|
| Native SDDO classes in release | **50** |
| Native SDDO object properties | **28** |
| Relationship axioms using SDDO properties | **27** |
| `rdfs:domain` / `rdfs:range` declarations | **0** |
| Total classes in release incl. imports | 268 |
| Imported class provenance | OBI 318, RO 207, IAO 108, UO 62, BFO 56, NCBITaxon 22, UBERON/SWO/GO 6 each |
| Individuals / instance data | **none** |
| Last commit | **2023-01-31** (dormant ~3.5 years) |
| Stars / forks / open issues | 2 / 2 / 2 |
| Tagged releases | none |

### Upper-ontology commitments

SDDO is anchored in **BFO** via **OBI**, with **RO** relations, **IAO** for information entities, **SWO** for software, plus alignment to **SOSA/SSN** (`sosa:Sensor`, `sosa:Platform`, `sosa:Observation`) and **FOAF** (`foaf:Agent`). This is a realist-ontology stance — heavier than a typical property-graph or SKOS-based discovery KG.

---

## 3. Repository health — read this before committing to it

These are the facts that determine whether you *depend on* SDDO or merely *learn from* it:

- **Dormant since 31 Jan 2023.** No commits, no releases, no tags.
- **Not published anywhere.** Verified live:
  - `http://purl.obolibrary.org/obo/sddo.owl` → **404**
  - `https://obofoundry.org/ontology/sddo` → **404**
  - `https://www.ebi.ac.uk/ols4/api/ontologies/sddo` → **404**
  - Ontobee `SDDO` → **500**

  The README, `CONTRIBUTING.md`, and `src/metadata/sddo.md` all point users at OBO Foundry / OLS / Ontobee endpoints that do not exist. `activity_status: active` in the metadata is stale.
- **Stale URLs throughout.** README badge and issue links point to `aeleish/sddo`; metadata references a retired Travis CI job.
- **Internal inconsistency.** `src/ontology/reports/*.tsv` were generated from an older edit state and disagree with the release: e.g. `SDDO_3000114` is `NASA program` in `sddo.obo` but `observableDiscoveryCriterion` in `synonyms.tsv`. **IDs were reassigned across edits** — do not treat SDDO IDs as stable identifiers.
- **Malformed IRIs** in the reports (`.../obo/SDDO_/SDDO_1000003`), indicating a build/template bug.
- **35 open QC violations** in `sddo-edit.owl-obo-report.tsv` — 1 ERROR (missing label on `foaf:Agent`) and ~34 WARNs, nearly all *missing definitions on the object properties*. The relations that matter most for a KG are the least documented.
- **Naming is not normalized:** `laboratoryProcedure`, `experiment Apparatus`, `NASAMission`, `photoGallery`, `data Product` coexist — camelCase, spaced, and inconsistently capitalized.

**Implication:** treat this as an *abandoned but instructive prototype*. Do not `owl:imports` it into a production SDE graph. Do not resolve its PURLs at runtime.

---

## 4. What is genuinely useful (ranked)

### 4.1 ★★★★★ The DiscoveryCriterion facet taxonomy — *unreleased, highest value*

Found only in `input/SMD_ConceptualIMVersion1-20211221_DCB.xml` and the intermediate `sddo-merged.owl`. **It is not in `sddo.owl`.**

```
DiscoveryInterface --leadTo--> AccessInterface
DiscoveryInterface --present--> DiscoveryCriterion
DiscoveryInterface --aggregate--> {Data, Information, Model, Tool, Application}DiscoveryCriterion

DataDiscoveryCriterion subtypes:
  SpaceTimeBounding · Location · Observable · Platform · Instrument
  InstrumentOperatingMode · Mission · Operation · Sample · Experiment · DataProduct
```

This is a **facet model for a discovery UI**, produced by asking exactly the right question — the concept map's own description reads: *"Looking across SMD data portals, how are data characterized?"* That is the SDE question.

**Direct SDE applications:** faceted-search schema, query-intent slot filling for natural-language search, and a normalized target for cross-portal facet harmonization.

### 4.2 ★★★★★ The LSDA → SDDO crosswalk — *best worked example in the repo*

129 propositions in `input/LSDA2SMDmapping_20220311_dcb.xml`, mapping the Life Sciences Data Archive schema to SDDO/OBI/RO at field granularity:

```
lsda:experiment      are            OBI_0000066 / sddo:experiment
lsda:experiment      ro:has participant  {biospecimen, institution, investigator, payload, personnel, subject}
lsda:experiment      sddo:generates      lsda:dataset
lsda:experiment      sddo:uses           lsda:hardware
lsda:experiment      lsda:fundedBy       lsda:program
lsda:experiment      lsda:managedAt      lsda:nasa_center
lsda:researchMission ro:part of          lsda:program
lsda:researchMission sddo:uses           lsda:payload
lsda:hardware        ro:part of          lsda:payload
lsda:dataset         lsda:ownedBy        lsda:personnel
```

Plus an inverse-property registry (`LSDA2SMDmapping_20220303_dcb_Propositions.csv`): `depicts/depictedIn`, `fundedBy/funded`, `implements/implementedBy`, `investigates/investigatedBy`, `managedAt/managed`, `operates/operatedBy`, `ownedBy/owns`.

**This is the reusable methodology**: archive schema → concept map → aligned ontology terms → OWL. If SDE needs to ingest heterogeneous SMD archives (PDS, SPASE, GeneLab, LSDA, CMR), this is a template for the mapping step, and a reason not to reinvent it.

### 4.3 ★★★★☆ The BIO conceptual model — concrete facet field names

99 propositions in `input/BIO-ConceptualModel20220104RED.xml`, giving per-entity discovery fields:

| Entity | Discovery fields |
|---|---|
| Experiment | Center, ExperimentAlias, ExperimentType, Investigator, Keyword, LaunchYear, Mission, Payload, Program, ResearchCategory, Species |
| Mission | Center, Duration, Mission, Payload, Program, ResearchCategory, Spacecraft, Species, Year |
| Subject | Age, Experiment, Gender, Hardware, Mission, Payload, Program, ScientificName, Strain, SubjectSpecies, Weight |
| Biospecimen | BiospecimenName, ExperimentTitle, Hardware, Mission, Payload, Protocol, SpecimenCategory, SpecimenSubcategory |
| Hardware | ComponentTitle, ExperimentTitle, HardwareAlias, HardwareType, Mission, Payload, ResearchCategory, VersionTitle |
| Person | City, Country, Institution, Mission/Study, Payload, Role |

These are **real facets from real NASA portals** — usable directly as a candidate property set for KG node types.

### 4.4 ★★★☆☆ The 27 released relationship axioms — a minimal core schema

```
mission        organizedBy      space agency
mission        deployedOn       hardware
NASA program   funds            NASAMission
experiment     partOf           mission
experiment     componentOf      research investigation
experiment     generates        specimen
experiment     conductedAt      facility
experiment     conductedOn      organism
experiment     uses             experiment Apparatus
instrument     operatesOn       instrument host
campaign       uses             instrument host
campaign       resultsIn        product
observation    ←isResultOf—     data element
assay          createdBy        laboratoryProcedure
assay          capableOfDetecting / capableOfIdentifying   event / feature
specimen       usedIn           assay
person         hasRole          author | organism
organization   hasRole          author | funder | investigator
factor         experimentalContextFor  observation
laboratoryProcedure  conductedAt  facility
```

Small, but it is a defensible **mission → program → experiment → instrument → data product → observation** provenance spine, and the `hasRole` pattern (agents bear roles rather than being subclassed by role) is worth adopting.

### 4.5 ★★★☆☆ Definitions with citations

~40 classes carry `IAO:0000115` definitions with `IAO:0000119` provenance pointing at real authorities — PDS4 Data Dictionary, SPASE DOIs, NASA Thesaurus, NCIT, schema.org, ORCIDs of the definers. Reusable as-is for node-type documentation, with attribution already attached.

### 4.6 ★★★☆☆ Archive-specific data-product subclasses

Explicit modeling of how each archive names its data products — the seed of a cross-archive type alignment:

- `pds_Data_Set_PDS3`, `pds_Collection_with_Type_Data` (PDS4), with an unusually candid comment: *"PDS data is tricky as while PDS3 has the concept of a data set; PDS4 doesn't really."*
- `spase:NumericalData`, `spase:DisplayData`, `spase:Catalog` (heliophysics)
- `study` — "A data product from the GeneLab repository"

### 4.7 ★★☆☆☆ Engineering scaffolding

The ODK setup is reusable if you build any OWL/SKOS artifact for SDE: 24 SPARQL QC queries (`src/sparql/`) covering missing labels, redundant subClassOf, IRI range violations, self-referential definitions, equivalent-class violations; plus a working ROBOT `Makefile` and GitHub Actions CI.

Also `src/scripts/cxlConverter/` — Python that converts CmapTools CXL concept maps to RDF/OWL. Useful if SMEs are more comfortable drawing concept maps than writing OWL, which is a realistic assumption.

---

## 5. What is missing or unusable

| Gap | Detail |
|---|---|
| **No Earth-science depth** | Despite Earth science being SDE's core, there are **no GCMD/KMS keywords, no CMR/UMM alignment, no ISO 19115, no DCAT, no STAC, no CF conventions**. `UMM` appears only in passing text. This is the single biggest gap for SDE. |
| **No science-variable vocabulary** | No measured parameters, no phenomena hierarchy, no units bound to variables. `data element` is a single abstract class. |
| **No instances** | Zero individuals. No mission, instrument, or dataset records. It is a schema sketch, not a graph. |
| **No domain/range** | 28 properties with no constraints and mostly no definitions — you cannot validate or infer with them as-is. |
| **DiscoveryCriterion never released** | The best content is stranded in `src/scripts/`, an intermediate build directory. |
| **Unstable identifiers** | Confirmed ID reassignment between edit states. |
| **Heavy BFO/OBI commitment** | 268 imported classes to pull in 50 native ones. Most SDE-adjacent KG work uses SKOS + schema.org/DCAT, which is a poor fit for BFO realism. Adopting SDDO wholesale imports an ontological stance you may not want. |

### Ecosystem context

SDE-adjacent KG work at NASA IMPACT went a different direction. [NASA Science Mission Directorate Knowledge Graph Discovery](https://arxiv.org/abs/2303.10871) (Timmer, Khoo, Mark, Martins, Berea, Renard, **Bugbee** — March 2023, ACM WWW'23 companion) builds SMD KGs via **NLP over text**, and does not use SDDO. Separately, GCMD keywords are already published as SKOS via KMS. So the practical center of gravity for an SDE KG is *NLP-derived nodes + SKOS vocabularies*, not a BFO-grounded ontology — which also explains why SDDO went quiet the same month that paper appeared.

---

## 6. Recommendation

**Mine it, don't import it.**

Do:

1. **Lift the DiscoveryCriterion taxonomy** as the starting point for SDE's facet/intent schema. Re-derive it from the CXL source, not from `sddo.owl` (where it is absent). It answers the right question and no other artifact in the ecosystem does.
2. **Adopt the LSDA crosswalk as methodology** for onboarding each SMD archive into the KG. Reuse the concept-map → propositions → aligned-terms → OWL pipeline, and the CXL converter if SMEs will author in Cmap.
3. **Reuse the 27-axiom provenance spine** (mission/program/experiment/instrument/product/observation) and the `hasRole` pattern for agents. Re-mint identifiers under an SDE namespace; do not carry SDDO IDs forward.
4. **Harvest the cited definitions** for node-type documentation — the PDS4/SPASE/NASA-Thesaurus provenance is already attached.
5. **Keep the SPARQL QC suite** if any OWL/SKOS artifact ships for SDE.

Do not:

- `owl:imports` SDDO or resolve its PURLs — nothing resolves, and the release lags the working files.
- Treat SDDO IDs as stable.
- Expect Earth-science coverage. Source that from **GCMD/KMS SKOS, CMR/UMM, and CF conventions**, which SDDO does not touch.

Worth a short conversation with **Kaylin Bugbee** (SDE lead, co-author on the SMD KG paper) and **Ruth Duerr / Dan Berrios** on why SDDO stalled — it likely saves rediscovering the same dead end, and the DiscoveryCriterion model may exist in a more developed form outside this repo.

---

## 7. Extracted artifacts

Machine-readable extracts placed in [`extracted/`](extracted/):

| File | Rows | Source |
|---|---|---|
| [smd_conceptual_model_propositions.tsv](extracted/smd_conceptual_model_propositions.tsv) | 90 | `SMD_ConceptualIMVersion1-20211221_DCB.xml` |
| [bio_conceptual_model_propositions.tsv](extracted/bio_conceptual_model_propositions.tsv) | 99 | `BIO-ConceptualModel20220104RED.xml` |
| [lsda_to_sddo_crosswalk_propositions.tsv](extracted/lsda_to_sddo_crosswalk_propositions.tsv) | 129 | `LSDA2SMDmapping_20220311_dcb.xml` |
| [sddo_released_classes.tsv](extracted/sddo_released_classes.tsv) | 50 | `sddo.obo` (id, label, definition, parents) |
| [sddo_released_relation_axioms.tsv](extracted/sddo_released_relation_axioms.tsv) | 27 | `sddo.owl` |

---

## 8. Verification notes

Claims in this assessment were checked as follows:

- **Repo obtained** by `git clone` at commit `d80ee46`; full file inventory enumerated (108 files outside `.git`).
- **Class/property/axiom counts** derived by parsing `sddo.obo` and `sddo.owl` directly, not from the repo's own `reports/` (which were found stale). Counts cross-checked between OBO and OWL serializations.
- **Corrected during analysis:** an initial count of 86 SDDO classes came from a raw IRI grep that also matched object and annotation properties; parsing `[Term]` blocks gives the true figure of **50**. Likewise, `sddo-edit.owl` initially appeared to contain zero axioms — it is in OWL Functional Syntax, not RDF/XML, so the XML-tag grep missed them; the release does contain 27.
- **Publication status** verified with live HTTP requests to the PURL, OBO Foundry, OLS4, and Ontobee (all 404/500 as recorded above).
- **Concept-map propositions** extracted with a purpose-written CXL parser reconstructing `concept → linking-phrase → concept` paths; totals (90/99/129) reproduced in the TSVs.
- **Ecosystem claim** checked against the arXiv abstract for 2303.10871 — confirmed it makes no mention of SDDO.
- **Not verified:** whether a more developed DiscoveryCriterion model exists outside this repo (no public trace found); whether the 2 open issues contain relevant discussion (issue bodies not read).

---

## Sources

- [NASA-IMPACT/sddo](https://github.com/NASA-IMPACT/sddo) — repository under assessment
- [NASA Science Mission Directorate Knowledge Graph Discovery (arXiv:2303.10871)](https://arxiv.org/abs/2303.10871)
- [GCMD Keywords Overview (NTRS 20230015694)](https://ntrs.nasa.gov/api/citations/20230015694/downloads/GCMD%20Keywords.pdf)
- [GCMD Keyword Viewer — NASA Earthdata](http://www.earthdata.nasa.gov/learn/find-data/idn/gcmd-keywords)
- [Science Keyword Taxonomy and SKOS/Semantic Relationships in NASA's GCMD (ESIP)](https://commons.esipfed.org/node/2523)
- [INCATools ontology-starter-kit](https://github.com/INCATools/ontology-starter-kit)
