# Internal organs and anatomy at every age: masses, volumes, positions, phantoms, scaling rules for HeroBody

Research date: 2026-10-09. Researcher notes for the report writer.

**Access notes.** The network in this session was badly limited. WebFetch failed with DNS errors (`getaddrinfo ENOTFOUND`) for every non-GitHub host I tried: pmc.ncbi.nlm.nih.gov, www.icrp.org, radon-and-life.narod.ru. curl through the proxy got `403 CONNECT` for pmc.ncbi.nlm.nih.gov, www.ncbi.nlm.nih.gov, europepmc.org, www.ebi.ac.uk, www.icrp.org, psec.uchicago.edu (a full ICRP 89 PDF copy is there), radon-and-life.narod.ru (another ICRP 89 copy), cern.ch (Geant4's ICRP 110 data), zenodo, figshare, doi.org, huggingface, cdn.humanatlas.io, itis.swiss, lifesciencedb.jp, dbarchive.biosciencedbc.jp and web.archive.org. The GitHub REST API is limited to this session's own repo. Only **raw.githubusercontent.com**, the GitHub code-search tool, pypi.org and gitlab/bitbucket front pages could be reached. The shared web-search budget ran out after 3 searches; 2 of them returned useful snippets. So the primary numbers come from **machine-readable GitHub copies**:

- **PK-Sim / Open Systems Pharmacology database** (`Open-Systems-Pharmacology/PK-Sim`, `src/Db/PKSimDB.sqlite`, 30.7 MB, GPLv2 software). Its population `European_ICRP_2002` encodes ICRP Publication 89. It holds body mass, height and organ volumes for newborn, 1, 5, 10, 15 y and adult, plus PK-Sim's own extension to 40, 50, 60, 70, 80, 90 and 100 y. I read it with Python `sqlite3` (read-only).
- **US EPA httk R package** (`cran/httk`, `R/tissue_mass_functions.R`, `R/tissue_masses_flows.R`, `R/tissue_scale.R`). This is the httk-pop organ mass model (Ring et al. 2017). It gives allometric equations for organ mass vs age, height, weight and BSA.
- **ICRP 89 adult male/female organ mass CSV** (`RayzeBio/Dosimetry-calculator`, two CSV files citing Ann. ICRP 32(3-4) and OLINDA's ICRP 89 phantoms).
- **NCI PHANTOM user manual** (`ncidose/ncidose.github.io`, `_manuals/PHANTOM-User-Manual.md`, release of 2026-09-30). This is the primary document for the UF/NCI phantom library.
- **HuBMAP HRA 3D reference library page data** (`hubmapconsortium/hra-ui`, `.../3d-reference-library-page/data.yaml`).
- **Geant4 example READMEs** for ICRP 110 and ICRP 145 (`Geant4/geant4`).
- **Third-party surveys that quote primary sources**: `Opening-Science/open-twin-xr` docs `GEOMETRY_SOURCES_SURVEY.md` and `research/ORGAN_SHAPE_MODELS.md`, which give licence checks for ICRP, XCAT, ViP, UF/NCI, BodyParts3D, Z-Anatomy and OpenAnatomy, and the Segars 2014 / iPhantom numbers. Also a skin-dose project (`ben-m-shields/shields_skin_dose_calculator`) and a GATE study README (`fatimatuzzahr0h/Age-dependence-of-Basal-Layer`) that list the ICRP 156 paediatric mesh phantoms.

**Tags.** **[fetched]** = I read the primary file or code myself, including data files read from a GitHub copy. **[computed]** = I computed it from fetched data; the method is stated. **[fetched-secondary]** = I read a third-party document that quotes the primary source. **[snippet]** = search-result snippet only. **[inference]** = my reasoning or recall from training. It was **not** checked in this session and must be checked before it is used as ground truth. Ages are in years (y). Masses are in g unless stated.

**How the ICRP 89 child values were verified (important).** I could not open ICRP 89 itself. I wrote down the Table 2.8 values from memory, then tested them against the PK-Sim ICRP database. PK-Sim stores organ **volumes** that include an organ's blood and a body-wide normalisation. I found that, for 11 organs, `V_PKSim / m_ICRP = k_organ × f(age, sex)`. Here f is one common age/sex factor, taken from the skeleton. k_organ is a constant: 1.16384 for liver, 1.12326 heart, 1.25455 kidneys, 1.44179 spleen, 1.20749 pancreas, 0.998 stomach, 0.990 small and large intestine, 1.0126 skin, 0.991 muscle and 1.022 gonads. It is the same for both sexes to 5 digits. Back-computing gives **exactly** the rounded ICRP numbers at all 12 age/sex points: 9.5 g spleen at birth, 2430 g skeleton at 5 y, 7180 g skeleton in a 15 y female, and so on. Brain is stored as `m × 1.0406` (male) and `m × 1.0438` (female), so brain masses read straight out. These rows are tagged **[computed]**. The test **corrected two of my recalled values**: brain at 10 y male is 1400, not 1430, and ovaries at 15 y are 6 g, not 11. Rows that PK-Sim does not hold are still memory only and tagged [inference]: lungs, thymus, thyroid, bladder, adrenals and adipose. Their adult values are confirmed by the RayzeBio CSV.

---

## 1. ICRP Publication 89 (2002) reference organ masses: newborn, 1, 5, 10, 15 y, adult male and female

### Takeaway
- ICRP 89 defines 6 reference ages (newborn, 1, 5, 10, 15 y, adult). Sexes are the same up to 10 y for most organs. Brain and gonads split earlier. Body mass is **3.5, 10, 19, 32, 56 (M) / 53 (F), 73 (M) / 60 (F) kg**. Height is **51, 76, 109, 138, 167 (M) / 161 (F), 176 (M) / 163 (F) cm**.
- The big age effects: the **brain is 10.9% of body mass at birth and 2.0% in the adult male**. The liver is 3.7% vs 2.5%. The thymus is 0.37% vs 0.03%. The kidneys are 0.71% vs 0.42%. Muscle is only 23% of body mass at birth vs 40% in the adult male. These ratios are the "infant look" inside the body.
- The brain reaches 65% of its adult mass by 1 y (950 of 1450 g) and 90% by 5 y. The thymus peaks in absolute mass around 10 y (40 g) and then falls to 25/20 g in adults. Most other organs follow body mass with exponents of 0.82 to 1.03.

### Cited Findings
**Reference body size (ICRP 89 as encoded in PK-Sim `European_ICRP_2002`)** [fetched] ([PK-Sim DB](https://github.com/Open-Systems-Pharmacology/PK-Sim/blob/develop/src/Db/PKSimDB.sqlite))

| | 0 y | 1 y | 5 y | 10 y | 15 y M | 15 y F | adult M | adult F |
|---|---|---|---|---|---|---|---|---|
| body mass kg | 3.5 | 10 | 19 | 32 | 56 | 53 | 73 | 60 |
| height cm | 51 | 76 | 109 | 138 | 167 | 161 | 176 | 163 |

**Organ and tissue masses, g (ICRP 89 Table 2.8 values).** Evidence per row is in the last column.

| organ / tissue | 0 y | 1 y | 5 y | 10 y | 15 y M | 15 y F | adult M | adult F | evidence |
|---|---|---|---|---|---|---|---|---|---|
| brain | 380 | 950 | 1310 M / 1180 F | 1400 M / 1220 F | 1420 | 1300 | 1450 | 1300 | [computed] from PK-Sim; 0/1/5 y also [snippet] ([ICRP draft](https://www.icrp.org/docs/Specific%20Absorbed%20Fractions%20for%20Reference%20Paediatric%20Individuals.pdf)) |
| liver | 130 | 330 | 570 | 830 | 1300 | 1300 | 1800 | 1400 | [computed]; row also [snippet] ([ICRP 89 copy](https://psec.uchicago.edu/Simulation/icrpp89.pdf)) |
| heart (tissue only, no blood) | 20 | 50 | 85 | 140 | 230 | 220 | 330 | 250 | [computed] |
| kidneys (both) | 25 | 70 | 110 | 180 | 250 | 240 | 310 | 275 | [computed] |
| spleen | 9.5 | 29 | 50 | 80 | 130 | 130 | 150 | 130 | [computed] |
| pancreas | 6 | 20 | 35 | 60 | 110 | 100 | 140 | 120 | [computed] |
| stomach wall | 7 | 20 | 50 | 85 | 120 | 120 | 150 | 140 | [computed] |
| small intestine wall | 30 | 85 | 220 | 370 | 520 | 520 | 650 | 600 | [computed] |
| large intestine wall (R + L colon + rectosigmoid) | 17 | 50 | 120 | 210 | 300 | 300 | 370 | 360 | [computed] |
| skin | 175 | 350 | 570 | 820 | 2000 | 1700 | 3300 | 2300 | [computed] |
| skeleton (bone + marrow + cartilage etc.) | 370 | 1170 | 2430 | 4500 | 7950 | 7180 | 10500 | 7800 | [computed] |
| skeletal muscle | 800 | 1900 | 5600 | 11000 | 24000 | 17000 | 29000 | 17500 | [computed] |
| testes | 0.85 | 1.5 | 1.7 | 2.0 | 16 | – | 35 | – | [computed] (PK-Sim "Gonads") |
| ovaries | 0.3 | 0.8 | 2.0 | 3.5 | – | 6 | – | 11 | [computed] |
| lungs (with blood) | 60 | 150 | 300 | 500 | 900 | 750 | 1200 | 950 | adult [fetched-secondary] ([RayzeBio CSV](https://github.com/RayzeBio/Dosimetry-calculator/blob/main/tissue-masses-human-ICRP89-male-female.xlsb.csv)); children [inference] |
| thymus | 13 | 30 | 30 | 40 | 35 | 30 | 25 | 20 | adult [fetched-secondary]; children [inference] |
| thyroid | 1.3 | 1.8 | 3.4 | 7.9 | 12 | 12 | 20 | 17 | adult [fetched-secondary]; children [inference] |
| urinary bladder wall | 4 | 9 | 16 | 25 | 40 | 35 | 50 | 40 | adult [fetched-secondary]; children [inference] |
| adrenals (both) | 6 | 4 | 5 | 7 | 10 | 9 | 14 | 13 | adult [fetched-secondary]; children [inference] |
| adipose, total (excl. yellow marrow) | ~930 | ~3800 | ~5500 | ~8600 | ~12000 | ~18700 | 18200 | 22500 | [inference] all ages; see the companion note `body_composition_fat_muscle_by_age.md` |
| blood volume, mL | 270 | 500 | 1400 | 2400 | 4500 | 3300 | 5300 | 3900 | adult [fetched-secondary]; children [inference] |
| gall bladder wall | – | – | – | – | – | – | 10 | 8 | [fetched-secondary] |
| eyes (both) | – | – | – | – | – | – | 15 | 15 | [fetched-secondary] |
| breast | – | – | – | – | – | – | 25 | 500 | [fetched-secondary] |
| salivary glands | – | – | – | – | – | – | 85 | 70 | [fetched-secondary] |
| oesophagus wall | – | – | – | – | – | – | 40 | 35 | [fetched-secondary] |
| prostate / uterus | 0.8 / 4.0 | 1.0 / 1.5 | 1.2 / 3 | 1.6 / 4 | 4.3 | 30 | 17 | 80 | adult [fetched-secondary]; children [inference] |
| GI contents (stomach / SI / colon), adult | – | – | – | – | – | – | 250 / 350 / 300 | 205 / 288 / 247 | [fetched-secondary] |

**Organ mass as % of body mass (male reference)** [computed from the table above]

| organ | 0 y | 1 y | 5 y | 10 y | 15 y | adult |
|---|---|---|---|---|---|---|
| brain | 10.86 | 9.50 | 6.89 | 4.38 | 2.54 | 1.99 |
| liver | 3.71 | 3.30 | 3.00 | 2.59 | 2.32 | 2.47 |
| heart | 0.57 | 0.50 | 0.45 | 0.44 | 0.41 | 0.45 |
| lungs | 1.71 | 1.50 | 1.58 | 1.56 | 1.61 | 1.64 |
| kidneys | 0.71 | 0.70 | 0.58 | 0.56 | 0.45 | 0.42 |
| spleen | 0.27 | 0.29 | 0.26 | 0.25 | 0.23 | 0.21 |
| thymus | 0.37 | 0.30 | 0.16 | 0.13 | 0.06 | 0.03 |
| skeleton | 10.57 | 11.70 | 12.79 | 14.06 | 14.20 | 14.38 |
| muscle | 22.86 | 19.00 | 29.47 | 34.38 | 42.86 | 39.73 |
| skin | 5.00 | 3.50 | 3.00 | 2.56 | 3.57 | 4.52 |

- ICRP 89 covers Western European and North American reference individuals at 6 ages, both sexes [snippet] ([ICRP 89 page](https://www.icrp.org/publication.asp?id=icrp+publication+89)).
- The paediatric ICRP phantoms assign the **5 y and 10 y brain target as the mean of the male and female ICRP 89 values**. For example, 5 y = 1245 g (1180 F / 1310 M) [snippet] ([ICRP draft SAF report](https://www.icrp.org/docs/Specific%20Absorbed%20Fractions%20for%20Reference%20Paediatric%20Individuals.pdf)).
- PK-Sim stores the same newborn and 1 y organ values for both sexes. It starts splitting sexes at 5 y for brain and gonads and at 15 y for the rest [computed].

### Inferences
- HeroBody should treat **ICRP 89 as the age backbone** for organ masses. Anchor at 0, 1, 5, 10, 15 and adult, then interpolate in log-mass vs age, or better vs body mass (Section 3).
- Sex differences in organs before 15 y are small and come from body size. Apply them through the body (height and weight), not as separate organ tables. Brain and gonads are the exceptions.

### Gaps
- Lungs, thymus, thyroid, bladder wall, adrenals, adipose and blood volume for children are from memory, not fetched. They need checking against ICRP 89 Table 2.8. The psec.uchicago.edu PDF copy, the ICRP 143 report tables or the NCI `phantom-mastertable.xlsx` "ICRP Arms" sheet would settle them.
- ICRP 89 has no values beyond "adult" (it assumes about 20 to 30 y). Old age needs other sources (Section 3).

---

## 2. Computational phantoms across ages: coverage, format, licence

### Takeaway
- **No openly licensed whole-body organ set exists for children or old people.** Every phantom with age-specific organs (ICRP 143/156, UF/NCI, XCAT, ViP) is behind a permission request, a software transfer agreement or a fee. Openly licensed organ meshes are BodyParts3D (CC BY 4.0, one adult male) and HuBMAP HRA (CC BY 4.0, adult Visible Human male 38 y and female 59 y, no paediatric organs). So HeroBody must **scale adult meshes to age targets** (Sections 5 and 6). The phantoms are useful as **published numbers**: masses, positions and papers, not as assets.
- For a personal, non-commercial project, the ICRP 145/156 meshes can be downloaded and studied. ICRP's terms are "permission required for reuse", so they should be used as **reference only**: to measure organ positions and sizes per age. Do not ship them inside HeroBody assets.

### Cited Findings
| phantom | ages | format | licence / access | evidence |
|---|---|---|---|---|
| **ICRP 110** adult reference voxel phantoms (2009) | adult M (176 cm, 73 kg), adult F (163 cm, 60 kg) | voxel; AM 254×127×222 voxels, 2.137 mm in-plane × 8.0 mm slices; AF 299×137×348, 1.775 × 4.84 mm | data on CD-ROM with the publication and as "Supplementary Data" on icrp.org; used in Geant4 "with the kind permission of the ICRP" | [fetched] ([Geant4 ICRP110 README](https://github.com/Geant4/geant4/blob/master/examples/advanced/ICRP110_HumanPhantoms/README)); licence [fetched-secondary] ([open-twin survey](https://github.com/Opening-Science/open-twin-xr/blob/main/docs/research/ORGAN_SHAPE_MODELS.md)) |
| **ICRP 143** paediatric reference computational phantoms (2020) | newborn, 1, 5, 10, 15 y, M and F (10 phantoms) | voxel | ICRP permission regime; also redistributed as NIfTI inside the NCI PHANTOM library ("icrp-reference", 12 phantoms incl. adults) under NCI's agreement | [fetched] ([NCI manual](https://github.com/ncidose/ncidose.github.io/blob/master/_manuals/PHANTOM-User-Manual.md)); [fetched-secondary] ([open-twin survey](https://github.com/Opening-Science/open-twin-xr/blob/main/docs/research/ORGAN_SHAPE_MODELS.md)) |
| **ICRP 145** adult mesh-type reference computational phantoms (MRCP, 2020) | adult M, F | polygon mesh (OBJ+MTL) + tetrahedral mesh; about 187 organs, 53 materials, about 8.5 M tetrahedra; download about 1.75 GiB, no registration | "ICRP retains copyrights on all its publications; even free to access publications require permission for reuse". Data free to download, all rights reserved | [fetched] ([Geant4 ICRP145 README](https://github.com/Geant4/geant4/blob/master/examples/advanced/ICRP145_HumanPhantoms/README)); licence and size [fetched-secondary] ([open-twin geometry survey](https://github.com/Opening-Science/open-twin-xr/blob/main/docs/GEOMETRY_SOURCES_SURVEY.md)) |
| **ICRP 156** paediatric mesh-type reference computational phantoms (2024, Ann. ICRP 53(1-2)) | newborn, 1, 5, 10, 15 y, M and F (10 phantoms) at ICRP 89 heights and weights | polygon + tetrahedral mesh | obtained "under licence from the ICRP", **not redistributed** by users | [fetched-secondary] ([shields skin dose project](https://github.com/ben-m-shields/shields_skin_dose_calculator/blob/main/quarto_presentation/presentation.qmd), [GATE study README](https://github.com/fatimatuzzahr0h/Age-dependence-of-Basal-Layer/blob/main/README.md)) |
| ICRP pregnant-female mesh phantoms (recent) | pregnant adult | mesh | ICRP | [fetched-secondary] (GitHub repo names of BH-Shin dose-coefficient datasets) |
| **UF/NCI reference-size phantoms** (Lee 2010) | 12 subjects: newborn, 1, 5, 10, 15 y, adult, M and F | hybrid NURBS/polygon in origin; released as voxel NIfTI (`.nii.gz`), high and low resolution, with and without arms, plus `organ-metadata.csv` (tag, material, density, voxel count, volume, mass) and `phantom-mastertable.xlsx` | NCI "secure User Portal"; access "subject to the applicable STA or license agreement". The survey quotes "non-profit only, destroy on completion, raw mesh format not provided" | [fetched] ([NCI manual](https://github.com/ncidose/ncidose.github.io/blob/master/_manuals/PHANTOM-User-Manual.md), [NCI portal script](https://github.com/ncidose/ncidose.github.io/blob/master/scripts/portal/subscriptions.js)); [fetched-secondary] terms |
| **UF/NCI body-size-dependent phantoms** (Geyer 2014) | **362 subjects**, 0 y to adult, across height and weight percentiles; IDs encode age, sex, height, weight, e.g. `00f050005` = 0 y female, 50 cm, 5 kg; adult code 35 | NIfTI voxel, 1,448 files | as above | [fetched] ([NCI manual](https://github.com/ncidose/ncidose.github.io/blob/master/_manuals/PHANTOM-User-Manual.md)) |
| UF/NCI pregnant phantoms (Maynard 2014) | 8 gestational ages | NIfTI + MCNP | as above | [fetched] (same) |
| **XCAT / 4D XCAT** (Segars, Duke) | adult reference + anatomically variable population (58 adult + 69 paediatric CT used in Segars 2014); 4D beating heart and breathing | NURBS / subdivision surfaces | no published terms; "Request Resource" form; licence fee (only published price: XCAT Brain $400 first year, $200/yr after). XCAT 3.0 moved to segmenting patient CT | [fetched-secondary] ([open-twin survey](https://github.com/Opening-Science/open-twin-xr/blob/main/docs/research/ORGAN_SHAPE_MODELS.md)) |
| **IT'IS Virtual Population (ViP)** | adults (e.g. Duke 34 y M, Ella 26 y F, Glenn 84 y M), children (e.g. Thelonious 6 y M, Roberta 5 y F, Billie 11 y F, Louis 14 y M), pregnant, posable | CAD surfaces + voxel | paid licence; clause 2.3.2 bars distribution of data or derivatives; clause 2.3.4 forbids reading model data from memory | licence [fetched-secondary] (same); model names and ages [inference] |
| **BodyParts3D** (DBCLS) | one adult male | OBJ/STL, FMA IDs, about 3,000 structures | **CC BY 4.0** since 2025-02-27 (older mirrors still say CC BY-SA 2.1 JP) | [fetched-secondary] ([open-twin geometry survey](https://github.com/Opening-Science/open-twin-xr/blob/main/docs/GEOMETRY_SOURCES_SURVEY.md)) |
| **HuBMAP HRA 3D reference organs** | adult only: Visible Human male (38 y, 180.3 cm, 199 lb) and female (59 y, 171.2 cm, obese) | GLB with crosswalk CSV; male, female, L/R variants; also Allen brain, NIH lymph node, SBU large intestine models | CC BY 4.0. **The library page lists only "Female" and "Male" variants; no paediatric organs** | [fetched] ([HRA 3D library page data](https://github.com/hubmapconsortium/hra-ui/blob/main/apps/humanatlas.io/public/assets/content/3d-reference-library-page/data.yaml)) |
| **Z-Anatomy** | BodyParts3D adult male, retopologised, TA2 names, over 7,000 structures | .blend, OBJ | CC BY-SA 4.0 (share-alike); one UW white-matter part has no licence | [fetched-secondary] ([open-twin geometry survey](https://github.com/Opening-Science/open-twin-xr/blob/main/docs/GEOMETRY_SOURCES_SURVEY.md)) |
| **Open Anatomy / SPL atlases** | per atlas; abdomen atlas is one 42 y male, 94 `.vtk` meshes | Slicer scenes, VTK, glTF | 3D Slicer Licence §B (BSD-like, commercial allowed) | [fetched-secondary] (same) |
| **PIPER child model** | about 1.5 to 6 y, scalable to 12 y | FE mesh | GPL v3 (model), CC BY 4.0 (data) | [fetched-secondary] (same) |
| **VIVA+** | average adult M and F | FE mesh, simplified viscera | LGPL v3 | [fetched-secondary] (same) |
| **SPARC organ scaffolds** | adult, per organ (no liver, kidney, spleen or pancreas) | cubic-Hermite FE | Apache-2.0 code, CC BY 4.0 data | [fetched-secondary] ([open-twin survey](https://github.com/Opening-Science/open-twin-xr/blob/main/docs/research/ORGAN_SHAPE_MODELS.md)) |

- The NCI master table has sheets "ICRP Arms" and "ICRP Armless" with **ICRP reference organ masses by phantom**. `organ-metadata.csv` gives per-organ volume and mass for every UF/NCI phantom, including the 362 size-dependent ones [fetched] ([NCI manual](https://github.com/ncidose/ncidose.github.io/blob/master/_manuals/PHANTOM-User-Manual.md)). This is the best machine-readable source for organ mass vs (age, height, weight) if access is granted.

### Inferences
- **For HeroBody (non-commercial, solo): use phantoms as measurement references, not as shipped meshes.** The cleanest legal path is BodyParts3D + HRA meshes (CC BY 4.0, already in the project) scaled to ICRP targets.
- If the user can get the NCI STA, the 362 size-dependent phantoms are the ideal **training data** for a "body shape → organ size and position" regressor at every age. Under STA terms only the learned numbers could be kept, not the geometry.

### Gaps
- ICRP 156 organ count, mesh format details and exact terms were not read from ICRP itself.
- Whether HRA has any paediatric organ planned was not confirmed. The 3D library page as of this release lists adult male and female only.
- ViP model ages are from memory.

---

## 3. Organ allometry, growth curves and old-age change

### Takeaway
- **Brain follows age, not body size.** A saturating curve fits it well: `m_brain(age) = B0 · (3.68 − 2.68·exp(−age/0.89)) · exp(−age/629)` kg, with B0 = 0.405 kg (male) and 0.373 kg (female). It reaches 90% of peak by about 2.5 y. The ICRP data plateau later (1400 at 10 y) than this curve does.
- **Most trunk organs follow body size.** Fitted to the ICRP 89 male anchors, `m = a · BW^b` with **b ≈ 0.82 to 0.91** for liver, kidneys, heart and spleen, and **b ≈ 1.0** for lungs, pancreas and GI walls. RMS log error is 0.02 to 0.07, so within ±7%. Children's organs are relatively larger because b < 1.
- **Children: Ogiu 1997 equations** use height × √weight. **Adults: scale the reference by (H/H_ref)^0.75** (httk).
- **Old age, ratio to the 30 y value, at 70 / 80 / 90 y (PK-Sim ICRP population)**: liver 0.73 / 0.60 / 0.56 (M), 0.79 / 0.75 / 0.70 (F); brain 0.95 / 0.93 / 0.91 (M), 0.95 / 0.92 / 0.86 (F); kidneys 1.01 / 0.90 / 0.84 (M), 0.90 / 0.81 / 0.77 (F); spleen 0.77 / 0.66 / 0.55 (M); pancreas 0.87 / 0.79 / 0.76; muscle 0.88 / 0.82 / 0.73 (M), 0.78 / 0.66 / 0.61 (F); **heart grows**: 1.11 / 1.06 / 1.05 (M), 1.22 / 1.26 / 1.31 (F); fat 1.31 / 1.34 / 1.39 (M).

### Cited Findings
**httk-pop organ equations (US EPA httk, Ring et al. 2017)** [fetched] ([httk tissue_mass_functions.R](https://github.com/cran/httk/blob/master/R/tissue_mass_functions.R), [tissue_masses_flows.R](https://github.com/cran/httk/blob/master/R/tissue_masses_flows.R), [tissue_scale.R](https://github.com/cran/httk/blob/master/R/tissue_scale.R)). Units: H in cm, W in kg, mass in g unless stated.

```text
Brain (all ages, kg):  B0*((3.68 - 2.68*exp(-age/0.89)) * exp(-age/629)),  B0 = 0.405 M, 0.373 F
Default adult organs:  m = m_ref * (H/H_ref)^(3/4)          # tissue_scale()
Children <= 18 y (Ogiu et al. 1997):
  Liver   M: 576.9*H/100 + 8.9*W - 159.7          F: 674.3*H/100 + 6.5*W - 214.4
  Kidneys M: (10.24*H/100*sqrt(W) + 7.85) + (9.88*H/100*sqrt(W) + 7.2)      # L + R
          F: (10.65*H/100*sqrt(W) + 6.11) + (9.88*H/100*sqrt(W) + 6.55)
  Lungs   M: (29.08*H/100*sqrt(W) + 11.06) + (35.47*H/100*sqrt(W) + 5.53)
          F: (31.46*H/100*sqrt(W) + 1.43) + (35.30*H/100*sqrt(W) + 1.53)
  Pancreas M: 7.46*H/100*sqrt(W) - 0.79           F: 7.92*H/100*sqrt(W) - 2.09
  Spleen   M: 8.75*H/100*sqrt(W) + 11.06          F: 9.36*H/100*sqrt(W) + 7.98
BSA (m^2): age < 18: Haycock 0.024265 * W^0.5378 * H^0.3964 ; age >= 18: Mosteller sqrt(W*H/3600)
Skin (kg): exp(1.64*BSA - 1.93)                     # Bosgra 2012
Blood (L): M 3.33*BSA - 0.81 ; F 2.66*BSA - 0.46    # fallback by age: 83.3 mL/kg <3 mo, 87 (3-6 mo), 80 (7 mo-6 y), 75 (6-10 y), 71 (10-15 y), 71 M / 70 F adult
Skeletal muscle <= 18 y (kg, Webber & Barr 2012): C1/(1+C2*exp(-C3*age)) + C4/(1+exp(-C5*(age-C6)))
       M: C = 12.4, 11.0, 0.45, 19.7, 0.85, 13.7 ; F: C = 7.0, 6.5, 0.55, 13.0, 0.75, 11.5
Skeletal muscle > 18 y: m_ref_scaled - 0.001*age^2 (kg)          # Janssen 2000
Bone mineral (kg), > 1 y: M 0.89983 + (2.99019-0.89983)/(1+exp((14.17081-age)/1.58179))
                          F 0.74042 + (2.14976-0.74042)/(1+exp((12.35466-age)/1.35750))
   <= 1 y: (77.24 + 24.94*W + 0.21*age_days - 1.889*H)/1000 ; >= 50 y: minus 0.0056*age (F), 0.0019*age (M)
   bone mass = BMC/0.65 ; skeleton = bone/0.5
Adipose = W - sum(other tissues) - 0.047*W  (GI contents 1.4% + rest of body 3.3%)
Cardiac output: scaled ref * (1 - 0.005*(age-25)) for age > 25
```

**The httk equations evaluated at ICRP 89 heights and weights, compared with ICRP 89, g** [computed]

| sex, age | brain httk / ICRP | liver | kidneys | lungs | pancreas | spleen | skin |
|---|---|---|---|---|---|---|---|
| M 0 y | 405 / 380 | 166 / 130 | 34 / 25 | 78 / 60 | 6 / 6 | 19 / 9.5 | 210 / 175 |
| M 1 y | 1136 / 950 | 368 / 330 | 63 / 70 | 172 / 150 | 17 / 20 | 32 / 29 | 312 / 350 |
| M 5 y | 1475 / 1310 | 638 / 570 | 111 / 110 | 323 / 300 | 35 / 35 | 53 / 50 | 504 / 570 |
| M 10 y | 1467 / 1400 | 921 / 830 | 172 / 180 | 520 / 500 | 57 / 60 | 79 / 80 | 886 / 820 |
| M 15 y | 1455 / 1420 | 1302 / 1300 | 266 / 250 | 823 / 900 | 92 / 110 | 120 / 130 | 2028 / 2000 |
| F 15 y | 1340 / 1300 | 1216 / 1300 | 253 / 240 | 785 / 750 | 91 / 100 | 118 / 130 | 1809 / 1700 |

Ogiu matches ICRP within about 10% from 5 to 15 y. It is 20 to 30% high at birth for liver and kidneys and 2× high for the newborn spleen. **Use ICRP anchors for 0 to 1 y and Ogiu for individual variation above 1 y.** [inference]

**httk brain curve, g** [computed]: male 405 (0 y), 871 (0.5 y), 1136 (1 y), 1371 (2 y), 1446 (3 y), 1475 (5 y), 1467 (10 y), 1444 (20 y), 1399 (40 y), 1355 (60 y), 1312 (80 y). Female 373, 802, 1046, 1263, 1332, 1358, 1351, 1330, 1288, 1248, 1209. The curve peaks at 5 y and falls slowly after that. ICRP (380, 950, 1310/1180, 1400/1220, 1420/1300, 1450/1300) rises to adulthood. **Use ICRP for 0 to 25 y and the slow decline only for age > 40 y.** [inference]

**Allometric fits to the ICRP 89 male anchors (0, 1, 5, 10, 15 y, adult)** [computed]. `m(g) = a · BW(kg)^b` and `m = a' · BSA(m²)^b'`.

| organ | a | b | RMS log err | a' (BSA) | b' (BSA) |
|---|---|---|---|---|---|
| liver | 46.02 | 0.844 | 0.037 | 781.6 | 1.194 |
| heart | 6.20 | 0.907 | 0.047 | 130.2 | 1.283 |
| lungs | 16.35 | 0.993 | 0.041 | 458.1 | 1.405 |
| kidneys | 9.72 | 0.819 | 0.066 | 152.0 | 1.160 |
| spleen | 3.29 | 0.910 | 0.062 | 69.9 | 1.289 |
| pancreas | 1.72 | 1.029 | 0.040 | 54.5 | 1.456 |
| stomach wall | 2.04 | 1.030 | 0.119 | 64.7 | 1.461 |
| small int. wall | 8.67 | 1.035 | 0.127 | 280.1 | 1.469 |
| large int. wall | 4.93 | 1.035 | 0.111 | 159.3 | 1.468 |
| bladder wall | 1.36 | 0.838 | 0.021 | 22.7 | 1.187 |
| thyroid | 0.30 | 0.927 | 0.243 | 6.6 | 1.311 |
| skeleton | 92.69 | 1.108 | 0.021 | 3819.5 | 1.569 |
| brain (poor fit, use age) | 295.9 | 0.417 | 0.203 | – | – |

**Old age: PK-Sim ICRP population organ volume ratios to age 30 y (40 y for female brain)** [computed] ([PK-Sim DB](https://github.com/Open-Systems-Pharmacology/PK-Sim/blob/develop/src/Db/PKSimDB.sqlite)). PK-Sim's own data source for > 30 y was not read; volumes include organ blood.

| organ | sex | 40 | 50 | 60 | 70 | 80 | 90 |
|---|---|---|---|---|---|---|---|
| brain | M | 1.00 | 1.00 | 0.97 | 0.95 | 0.93 | 0.91 |
| brain | F | 1.00 | 1.00 | 0.97 | 0.95 | 0.92 | 0.86 |
| liver | M | 0.99 | 0.94 | 0.87 | 0.73 | 0.60 | 0.56 |
| liver | F | 0.99 | 0.98 | 0.88 | 0.79 | 0.75 | 0.70 |
| kidneys | M | 1.09 | 1.07 | 1.04 | 1.01 | 0.90 | 0.84 |
| kidneys | F | 1.00 | 0.99 | 0.95 | 0.90 | 0.81 | 0.77 |
| heart | M | 1.04 | 1.05 | 1.09 | 1.11 | 1.06 | 1.05 |
| heart | F | 1.04 | 1.08 | 1.15 | 1.22 | 1.26 | 1.31 |
| spleen | M | 0.91 | 0.86 | 0.81 | 0.77 | 0.66 | 0.55 |
| spleen | F | 0.90 | 0.87 | 0.83 | 0.75 | 0.68 | 0.45 |
| pancreas | M/F | 1.00 | 0.97 | 0.94 | 0.87 | 0.79 | 0.76 |
| lungs | M | 1.03 | 1.03 | 1.04 | 0.98 | 0.88 | 0.78 |
| lungs | F | 1.01 | 1.03 | 1.01 | 1.00 | 0.84 | 0.80 |
| muscle | M | 1.00 | 0.96 | 0.92 | 0.88 | 0.82 | 0.73 |
| muscle | F | 1.14 | 1.05 | 0.94 | 0.78 | 0.66 | 0.61 |
| skin | M | 1.01 | 1.00 | 0.98 | 0.97 | 0.94 | 0.91 |
| bone | M | 1.00 | 0.97 | 0.94 | 0.88 | 0.82 | 0.76 |
| bone | F | 0.97 | 0.95 | 0.92 | 0.86 | 0.77 | 0.73 |
| fat | M | 1.10 | 1.12 | 1.21 | 1.31 | 1.34 | 1.39 |
| fat | F | 1.03 | 1.23 | 1.40 | 1.44 | 1.33 | 1.29 |

PK-Sim's ICRP population also lists mean body mass and height for older ages: male 74.4 kg at 40 y, 71.1 kg at 70 y, 68.1 kg at 80 y; height 175.1, 167.4, 165.5 cm. Female 63.3 kg at 40 y, 62.2 kg at 70 y, 56.1 kg at 80 y; height 160.9, 155.4, 151.7 cm [fetched].

**Other growth equations from memory (not checked here)** [inference]:
- **Liver volume vs BSA.** Urata 1995 (children and adults, Japanese): `SLV(mL) = 706.2 × BSA + 2.4`. Vauthey 2002 (Western adults): `TLV(mL) = −794.41 + 1267.28 × BSA`. Check: Urata at the ICRP adult male BSA of 1.89 m² gives 1337 mL ≈ 1400 g.
- **Kidney length.** Rosenbaum 1984 ultrasound means: about 4.5 cm at birth, 6.2 cm at 1 y, 7.9 cm at 5 y, 9.2 cm at 10 y, 10.3 cm at 15 y, adult 10 to 12 cm. Linear fits: `L(cm) ≈ 4.98 + 0.155 × age_months` below 1 y and `L ≈ 6.79 + 0.22 × age_years` from 1 to 18 y. In old age length falls about 0.5 cm per decade after 50 y.
- **Thymus.** Largest relative to body at birth. Absolute peak around puberty (ICRP 40 g at 10 y). Then involution with fat replacement. The organ envelope keeps a similar size (about 15 to 25 g with fat) into old age, but by 70 y the tissue is mostly fat.
- **Brain atrophy.** Autopsy brain mass peaks about 19 to 20 y: Dekaban 1978 male about 1450 g, female about 1290 g. It falls about 10% by 85+ y. MRI shows whole-brain loss of about 0.2%/y at 40 y, rising to about 0.5%/y after 70 y. Ventricles and sulci enlarge, and CSF space fills the difference, so the **skull and head size do not change**.
- **Heart.** Heart mass rises slightly with age, mainly the left-ventricle wall (more so in women). This matches PK-Sim.

### Inferences
- **The rule set for HeroBody**: brain by age (ICRP table, slow decline over 40 y); trunk organs by body size with the ICRP-fitted exponents, adjusted by an age multiplier after 50 y; thymus by age only.
- The old-age liver decline in PK-Sim (−27% at 70 y M) is steeper than many reviews state (about −20 to −40% between 20 and 90 y). Use PK-Sim ratios as the default, behind a "realism" knob.

### Gaps
- No primary source for PK-Sim's > 30 y organ data was read.
- Kidney length, liver volume formulas, Dekaban numbers and MRI atrophy rates are from memory.

---

## 4. Organ position and orientation by age

### Takeaway
- **Infant body plan**: a big head, a big liver filling the upper abdomen, a near-round chest with horizontal ribs, a horizontal heart (larger cardiothoracic ratio), a bladder in the abdomen, a large thymus on top of the heart, and a high larynx. These differences shrink by about 6 to 10 y. From then on positions are adult-like and scale with the skeleton.
- **External body size predicts organ position well in children (r² ≈ 0.79 to 0.89) but poorly in adults (r² ≤ 0.44)** (Segars 2014). For a stylised anime pipeline this means: drive organ positions from the **skeleton** (vertebral levels, rib cage, pelvis), not from the skin.

### Cited Findings
- Segars et al. 2014 (58 adult + 69 paediatric CT): external body dimensions predict organ axial position with adult r² ≤ 0.439 and organ volume with r² ≤ 0.410. Children: r² ≈ 0.79 to 0.89 for organ position vs height/age. Spleen volume stays about 40% uncertain whatever external measure is used (Whalen 2008). An affine-transformed size-matched template reaches organ Dice of only 0.2 to 0.6. Diffeomorphic registration reaches 0.8 to 0.9 (iPhantom, Fu 2021) [fetched-secondary] ([open-twin survey](https://github.com/Opening-Science/open-twin-xr/blob/main/docs/research/ORGAN_SHAPE_MODELS.md)).

**Positions by age: textbook values from memory** [inference]. Vertebral levels are for a supine body in quiet breathing.

| feature | newborn / infant | child (5 to 10 y) | adult | old age |
|---|---|---|---|---|
| liver size | 3.7% of BW; lower edge 1 to 3 cm below right costal margin (normal up to about 2 to 3 cm); left lobe crosses midline to the left upper abdomen | edge at or 1 cm below costal margin | 2.3 to 2.5% of BW; edge at the costal margin (about L1 to L2 midclavicular) | mass −20 to −40%; edge may drop with flatter diaphragm |
| liver span (percussion, MCL) | about 4.5 to 5 cm (1 wk) | about 6 to 7 cm (5 to 10 y) | 10 to 12 cm M, 8 to 10 cm F | – |
| diaphragm dome | flatter, inserts more horizontally; right dome about T8 to T9 | about T9 to T10 | right dome T9 to T10, left about one level lower (expiration) | lower, flatter (hyperinflation, kyphosis) |
| heart | apex 4th intercostal space, lateral to MCL; axis more horizontal; CTR up to about 0.6 | apex moves to 5th ICS; CTR ≤ 0.55 | apex 5th ICS at MCL, axis about 45°; CTR < 0.5 | more horizontal (raised diaphragm in obesity, kyphosis); unfolded aorta |
| thymus | large, front of mediastinum, can reach the diaphragm on X-ray ("sail sign") | still visible to about 5 to 8 y | small, fatty | fat |
| kidneys | relatively large (0.7% BW), lobulated; lower pole near the iliac crest | – | right T12 to L3, left T11 to L2/L3; hilum about L1; right 1 to 2 cm lower | length −0.5 cm per decade after 50 y |
| bladder | **abdominal organ**: empty bladder sits above the pubic symphysis | descends into the pelvis about 6 y | true pelvis behind the symphysis | pelvic floor descent |
| stomach | more horizontal; capacity about 20 to 30 mL at birth | – | J-shaped; 1 to 1.5 L | – |
| larynx | high, about C3 to C4 | – | C6 | slight descent |
| spinal cord end (conus) | L2 to L3 | – | L1 to L2 | – |
| chest shape | AP depth ≈ transverse width (round); ribs horizontal | ribs slope down | AP/transverse ≈ 0.7 | barrel chest in some (ratio rises) |
| head | brain 11% of BW; head about 1/4 of length | – | brain 2% of BW | skull fixed, brain shrinks, CSF space grows |

### Inferences
- Encode positions as **landmark targets in skeleton coordinates**: vertebral levels for liver edge, kidney poles, diaphragm dome and bladder; rib/ICS for the heart apex; pelvis frame for the bladder. Then interpolate between age anchors. That reuses the rig and the per-age skeleton from the sibling note `skeleton_posture_motion_by_age.md`.
- Heart axis: about 60 to 70° from vertical at birth, 45° adult [inference]. A rotation knob per stage is enough.

### Gaps
- None of the position numbers were fetched this session. The ICRP 143/156 phantoms, or the UF/NCI size-dependent phantoms if access is granted, would let us measure these levels directly per age.

---

## 5. Methods to scale/deform adult organ meshes to a child or elderly body

### Takeaway
- The phantom builders work in **two layers**:
  1. Shape the **body contour and skeleton** to the target (height, sitting height, circumferences).
  2. Fit **each organ to a target mass/volume** (ICRP 89 or a size regression), then fix overlaps.
- They tolerate small mass errors (UF/NCI and ICRP mesh phantoms aim for organ masses within about 1% of reference) [inference].
- Do **not** deform organs by the skin transform alone. Adult organs land wrong by 10 to 20 mm (Dice 0.2 to 0.6). Children are better.
- Recommended HeroBody method:
  1. Place organs with a skeleton-driven warp: rib cage, spine and pelvis landmarks, thin-plate or harmonic.
  2. Scale each organ about its own centroid to its target volume, `s = (V_target / V_mesh)^(1/3)`, with optional anisotropy per age (infant liver wider, chest rounder).
  3. Resolve containment and collisions with a signed-distance push inside a cavity hull.
  4. Verify volume error ≤ 2%.

### Cited Findings
- UF/NCI body-size-dependent phantoms come in 362 height/weight combinations from newborn to adult. This shows the "reference phantom deformed to body size" approach at scale [fetched] ([NCI manual](https://github.com/ncidose/ncidose.github.io/blob/master/_manuals/PHANTOM-User-Manual.md)).
- Phantom personalisation tiers (dosimetry field): reference → patient-dependent → **patient-sculpted** ("outer body contour reshaped to the individual, internal anatomy not individualized") → patient-specific [fetched-secondary] ([open-twin survey](https://github.com/Opening-Science/open-twin-xr/blob/main/docs/research/ORGAN_SHAPE_MODELS.md)).
- Matching a phantom by height and weight cuts CT organ-dose RMS error from 39.1% (reference phantom) to 20.3%. The best water-equivalent diameter gives 17.6%. About half the error cannot be removed by external matching (Stepusin 2017) [fetched-secondary] (same).
- Segars 2013 built 58 variable XCAT phantoms from **segmented patient CT**. It scaled only head, arms and legs from the reference [fetched-secondary] (same).
- Anatomy Transfer (Dicko et al., SIGGRAPH Asia 2013): transfer the template skeleton to the target, then map internal anatomy by **harmonic deformation driven by the target skin eroded by fat thickness** [fetched-secondary] (same). This is the closest graphics precedent to HeroBody.
- The project already has a per-body clamp: about 1% of anatomy pokes out of the skin on the neutral body and 1 to 5% on extreme bodies (project context).

### Inferences
**Recommended deformation stack for one character at one age (CPU-only, no ML needed):**
1. **Targets.** Compute the organ targets m_i(age, sex, H, W) by Section 6 rules. Volume target `V_i = m_i / ρ_i`.
2. **Skeleton warp (positions).** Build landmarks on the adult template: T1, T4, T8, T10, T12, L1, L3 and L5 vertebral bodies, sternum top/bottom, rib-6 lateral points, iliac crests, pubic symphysis, acetabula. Get the same landmarks from the age-scaled rig. Warp organ **centroids and axes** with thin-plate splines (TPS) or harmonic coordinates. Then add the age offsets from Section 4: bladder up, liver edge down, heart axis rotation, kidney poles.
3. **Organ scale (size).** For each organ, scale about the warped centroid: `s_i = (V_i / V_mesh_i)^(1/3)`. Optional anisotropy per stage, for example infant liver ×1.08 laterally and ×0.93 vertically, keeping volume [inference, not measured].
4. **Cavity hull.** Build the inner wall of the thorax (rib cage inner surface + diaphragm), abdomen (abdominal wall inner surface = skin minus fat-layer thickness minus muscle) and pelvis. Precompute a signed distance field (SDF) per cavity on a coarse grid (64³ is enough on CPU).
5. **Containment and collisions.** Iterate: for each organ vertex outside the cavity SDF, push inward along the gradient. For organ pairs, push vertices out along the other organ's SDF gradient. Smooth with Laplacian steps. Rescale each organ back to V_i after each pass. Stop when overlap < 0.5% of volume and volume error < 2%. This is the same idea as the existing per-body clamp, with volume restoration added.
6. **Hollow organs.** Stomach, bladder, intestines and lungs scale by **wall mass** (ICRP) and **fill state** (content or air) separately. Lungs: tissue + blood mass with an inflated density of about 0.38 g/cm³ (ICRP 110 lung medium [inference]) gives the parenchyma volume.
7. **Verification.** Report per organ: mass error, centroid offset from the vertebral-level target in mm, fraction outside the cavity. This fits the "zero manual cleanup, human verifies the profile" rule.

### Gaps
- No anisotropic per-age organ shape data was fetched. The ICRP 156 paediatric meshes (reference-only study) or UF/NCI size phantoms are the source to measure aspect ratios per age.
- The exact ICRP 145/156 conversion method (mass-matching tolerances, overlap removal) was not read.

---

## 6. Recommended per-age organ pipeline for HeroBody

### Takeaway
- **Data.** ICRP 89 anchors (Section 1) + httk/Ogiu individual equations + PK-Sim old-age multipliers. Meshes are BodyParts3D (male) + HRA (female, 37 organs). There are no new assets to licence.
- **Rule per organ** (table below). The brain is tied to the head module's interior scale. Bones follow the skeleton note: growth plates and ossification centres as **visibility/material states** on the adult bone meshes, not separate geometry.
- **Interpolation.** Monotone cubic (PCHIP) in `log(mass)` vs age between the anchors 0, 1, 5, 10, 15, 20, 30, 40, 50, 60, 70, 80, 90 y. For individual size, multiply by `(BW/BW_ref(age))^b` with the fitted b, or use Ogiu for 1 to 18 y.

### Cited Findings
**Organ mass targets per stage, g, reference body.** 0 to 15 y and adult columns are ICRP 89 (evidence as in Section 1). 70 y and 85 y = adult × PK-Sim ratio (85 = mean of the 80 and 90 y ratios) [computed]. "adult*" means hold the adult value (no data). Densities are typical ICRP soft-tissue media [inference].

| organ | sex | 0 y | 1 y | 5 y | 10 y | 15 y | adult (20 to 30 y) | 70 y | 85 y | density g/cm³ | adult volume cm³ |
|---|---|---|---|---|---|---|---|---|---|---|---|
| brain | M | 380 | 950 | 1310 | 1400 | 1420 | 1450 | 1378 | 1332 | 1.04 | 1394 |
| brain | F | 380 | 950 | 1180 | 1220 | 1300 | 1300 | 1238 | 1159 | 1.04 | 1250 |
| liver | M | 130 | 330 | 570 | 830 | 1300 | 1800 | 1309 | 1046 | 1.05 | 1714 |
| liver | F | 130 | 330 | 570 | 830 | 1300 | 1400 | 1106 | 1011 | 1.05 | 1333 |
| heart (tissue) | M | 20 | 50 | 85 | 140 | 230 | 330 | 368 | 348 | 1.05 | 314 |
| heart (tissue) | F | 20 | 50 | 85 | 140 | 220 | 250 | 304 | 322 | 1.05 | 238 |
| lungs (tissue + blood) | M | 60 | 150 | 300 | 500 | 900 | 1200 | 1179 | 995 | 0.38 inflated | 3158 |
| lungs (tissue + blood) | F | 60 | 150 | 300 | 500 | 750 | 950 | 946 | 778 | 0.38 inflated | 2500 |
| kidneys (both) | M | 25 | 70 | 110 | 180 | 250 | 310 | 313 | 270 | 1.05 | 295 |
| kidneys (both) | F | 25 | 70 | 110 | 180 | 240 | 275 | 248 | 216 | 1.05 | 262 |
| spleen | M | 9.5 | 29 | 50 | 80 | 130 | 150 | 115 | 91 | 1.04 | 144 |
| spleen | F | 9.5 | 29 | 50 | 80 | 130 | 130 | 98 | 74 | 1.04 | 125 |
| pancreas | M | 6 | 20 | 35 | 60 | 110 | 140 | 122 | 108 | 1.04 | 135 |
| pancreas | F | 6 | 20 | 35 | 60 | 100 | 120 | 105 | 92 | 1.04 | 115 |
| stomach wall | M | 7 | 20 | 50 | 85 | 120 | 150 | adult* | adult* | 1.04 | 144 |
| stomach wall | F | 7 | 20 | 50 | 85 | 120 | 140 | adult* | adult* | 1.04 | 135 |
| small intestine wall | M | 30 | 85 | 220 | 370 | 520 | 650 | adult* | adult* | 1.04 | 625 |
| small intestine wall | F | 30 | 85 | 220 | 370 | 520 | 600 | adult* | adult* | 1.04 | 577 |
| large intestine wall | M | 17 | 50 | 120 | 210 | 300 | 370 | adult* | adult* | 1.04 | 356 |
| large intestine wall | F | 17 | 50 | 120 | 210 | 300 | 360 | adult* | adult* | 1.04 | 346 |
| thymus | M | 13 | 30 | 30 | 40 | 35 | 25 | adult* (fatty) | adult* (fatty) | 1.03 | 24 |
| thymus | F | 13 | 30 | 30 | 40 | 30 | 20 | adult* (fatty) | adult* (fatty) | 1.03 | 19 |
| thyroid | M | 1.3 | 1.8 | 3.4 | 7.9 | 12 | 20 | adult* | adult* | 1.05 | 19 |
| thyroid | F | 1.3 | 1.8 | 3.4 | 7.9 | 12 | 17 | adult* | adult* | 1.05 | 16 |
| bladder wall | M | 4 | 9 | 16 | 25 | 40 | 50 | adult* | adult* | 1.04 | 48 |
| bladder wall | F | 4 | 9 | 16 | 25 | 35 | 40 | adult* | adult* | 1.04 | 38 |
| adrenals (both) | M | 6 | 4 | 5 | 7 | 10 | 14 | adult* | adult* | 1.03 | 14 |
| adrenals (both) | F | 6 | 4 | 5 | 7 | 9 | 13 | adult* | adult* | 1.03 | 13 |
| skin | M | 175 | 350 | 570 | 820 | 2000 | 3300 | 3190 | 3061 | 1.09 | 3028 |
| skin | F | 175 | 350 | 570 | 820 | 1700 | 2300 | 2302 | 2134 | 1.09 | 2110 |
| skeleton (whole) | M | 370 | 1170 | 2430 | 4500 | 7950 | 10500 | 9200 | 8266 | ~1.3 mean | 8077 |
| skeleton (whole) | F | 370 | 1170 | 2430 | 4500 | 7180 | 7800 | 6680 | 5815 | ~1.3 mean | 6000 |
| skeletal muscle | M | 800 | 1900 | 5600 | 11000 | 24000 | 29000 | 25613 | 22500 | 1.05 | 27619 |
| skeletal muscle | F | 800 | 1900 | 5600 | 11000 | 17000 | 17500 | 13590 | 11117 | 1.05 | 16667 |

**Stage mapping** (the project's 5 stages with in-stage sliders): baby = 0 to 2 y (anchors 0 and 1 y, slider to 2 y), child = 2 to 10 y (anchors 5, 10), teen = 10 to 18 y (anchors 10, 15, 18 ≈ adult values × (H/H_adult)^0.75), adult = 18 to 60 y (anchors 25, 40, 50, 60 via PK-Sim ratios), old = 60 to 100 y (anchors 70, 80, 90) [inference].

**Scaling rule per organ** [inference, built on the fetched equations above]

| organ group | target rule | position rule | notes |
|---|---|---|---|
| brain | ICRP by age and sex (slow decline > 40 y). **Do not scale by body size.** | fills cranial cavity; tie to the anime head module's "interior scaled by head size" | anime heads > 2× realistic: scale brain to the cranial volume of the drawn skull for cutaways, and keep the ICRP mass for "realistic" mode |
| liver, kidneys, spleen, pancreas, lungs | 1 to 18 y: Ogiu equations (H, W); 0 to 1 y: ICRP interpolation; adult: ICRP × (H/H_ref)^0.75; > 30 y: × PK-Sim age ratio | vertebral-level targets per age (Section 4) | spleen has ±40% natural spread: allow a per-character knob |
| heart | ICRP by age × (BW/BW_ref)^0.91; > 30 y × PK-Sim ratio | apex ICS + axis angle by age | horizontal in infants |
| GI walls, stomach, bladder | ICRP × (BW/BW_ref)^1.03 (bladder ^0.84) | bladder: abdominal < 6 y, pelvic after | contents/fill as a separate gag knob |
| thymus | ICRP by age only | anterior mediastinum over the heart | > 40 y: render as fat-coloured |
| thyroid, adrenals | ICRP by age only | fixed to larynx (C5 to T1) / kidney upper poles | adrenals are large at birth (6 g) and shrink in year 1 |
| skin | Bosgra `exp(1.64·BSA − 1.93)` kg, or ICRP | the outer surface itself | thickness map from the companion fat/skin note |
| muscle, fat | from the fat-layer / muscle notes | – | THE FAT LAYER build item |
| skeleton | ICRP mass as a sanity check; geometry from the rig and the skeleton note | bones ride the rig | see bones below |

**Bones with growth plates** [inference; details in `skeleton_posture_motion_by_age.md`]
- Keep **one adult bone mesh per bone** (BodyParts3D, 244 bones). Scale per age by the rig: bone length from the skeleton note's diaphysis-to-stature ratios, width by the age cross-section rule.
- Add a per-bone **epiphysis mask** (vertex group) at each end. For each age, set it to: *cartilage* (not ossified: invisible in X-ray mode, soft material), *ossifying* (small sphere inside the cartilage cap, growing), *open plate* (dark gap band 1 to 3 mm in X-ray mode) or *fused*. Drive the state from the ossification and fusion age tables in the skeleton note (e.g. no carpal bones at birth; femoral head appears at 4 to 6 months; elbow fuses at 11 to 15 y; clavicle last, early 20s).
- Skull: open fontanelles as a mask (posterior closes 2 to 3 months, anterior about 14 to 16 months median). Newborns have about 270 to 300 bony pieces; show them by splitting masks, not separate meshes.
- Old age: lower bone density in X-ray shader (PK-Sim bone volume −12% at 70 y, −24% at 90 y in men, −14% / −27% in women). Kyphosis comes from the posture note.

### Inferences
- Total organ volume should be checked against trunk volume per stage. Infants have a relatively larger abdominal organ volume (liver 3.7% BW), which gives the round belly. This can feed the "round torso" blendshape target in the anime range test.
- For anime styles that change body proportions (chibi), keep organs **realistic in mass ratios to body mass** but fit them into the stylised cavities by the SDF containment step. Report when the cavity is too small (volume ratio < 0.9) so the artist can accept a gag or adjust.

### Gaps
- Child values for lungs, thymus, thyroid, bladder, adrenals and adipose need verifying against ICRP 89.
- No per-age organ shape (aspect) data; no fetched position data.

---

## How this plugs into HeroBody

- **New module `hb/organs_age.py`** (Phase 2, alongside the knob solver):
  - `icrp89_table` with the Section 1 values; `[computed]` rows can be trusted, `[inference]` rows flagged.
  - `organ_targets(age, sex, H_cm, W_kg, realism=1.0)` returns mass and volume per organ, using the Section 3/6 rules. httk/Ogiu formulas go in as plain Python.
  - Old-age multipliers from the PK-Sim table.
- **Inputs from the character profile**: age (stage + slider → years), sex (fact, never fitted), solved height and weight from the knob solver (height applied last, so organ targets must be computed **after** the final uniform scale), body conditions (e.g. an organ-specific override for disability or illness gags).
- **Build items touched**:
  - (1) **Rig binding for female organs**: the HRA female organs get the same `organ_targets` and the per-age vertebral-level landmarks.
  - (2) **The per-body clamp**: becomes the containment step of Section 5, with volume restoration and a report.
  - (3) **THE FAT LAYER**: the abdominal cavity hull is skin − fat − muscle, so the organ step runs after the fat layer.
  - (4) **New blendshapes**: "round torso" and "anime waist" should keep cavity volume ≥ organ volume, or the report flags it.
- **Head module**: brain volume tied to the cranial interior. In "realistic" mode, check brain mass vs ICRP by age; in anime mode, scale to the drawn skull.
- **X-ray / cutaway shader**: thymus fatty after 40 y; growth-plate masks per bone per age; bone density by age.
- **Verification report** (zero manual cleanup): per organ mass error %, centroid offset (mm) from the vertebral-level target, % outside cavity, collisions.

## Sources

- PK-Sim database (Open Systems Pharmacology), `European_ICRP_2002` population: https://github.com/Open-Systems-Pharmacology/PK-Sim/blob/develop/src/Db/PKSimDB.sqlite (raw: https://raw.githubusercontent.com/Open-Systems-Pharmacology/PK-Sim/develop/src/Db/PKSimDB.sqlite)
- US EPA httk organ mass functions: https://github.com/cran/httk/blob/master/R/tissue_mass_functions.R ; https://github.com/cran/httk/blob/master/R/tissue_masses_flows.R ; https://github.com/cran/httk/blob/master/R/tissue_scale.R
- ICRP 89 adult masses (CSV, citing Ann. ICRP 32(3-4) / OLINDA): https://github.com/RayzeBio/Dosimetry-calculator/blob/main/tissue-masses-human-ICRP89-male-female.xlsb.csv ; https://github.com/RayzeBio/Dosimetry-calculator/blob/main/tissue-masses-human-ICRP89.csv
- ICRP Publication 89 page: https://www.icrp.org/publication.asp?id=icrp+publication+89 (blocked; snippet)
- ICRP 89 full-text copies (blocked; snippet only): https://psec.uchicago.edu/Simulation/icrpp89.pdf ; https://radon-and-life.narod.ru/pub/ICRP_89.pdf
- ICRP draft SAF report for paediatric individuals (snippet): https://www.icrp.org/docs/Specific%20Absorbed%20Fractions%20for%20Reference%20Paediatric%20Individuals.pdf
- ICRP paediatric phantom pages (snippet): https://www.icrp.org/page.asp?id=392 ; https://www.icrp.org/page.asp?id=393
- NCI PHANTOM user manual: https://github.com/ncidose/ncidose.github.io/blob/master/_manuals/PHANTOM-User-Manual.md ; NCI portal script: https://github.com/ncidose/ncidose.github.io/blob/master/scripts/portal/subscriptions.js
- Geant4 ICRP 110 example README: https://github.com/Geant4/geant4/blob/master/examples/advanced/ICRP110_HumanPhantoms/README
- Geant4 ICRP 145 example README: https://github.com/Geant4/geant4/blob/master/examples/advanced/ICRP145_HumanPhantoms/README
- ICRP 156 phantom list (skin dose project): https://github.com/ben-m-shields/shields_skin_dose_calculator/blob/main/quarto_presentation/presentation.qmd
- ICRP 156 licence statement (GATE study): https://github.com/fatimatuzzahr0h/Age-dependence-of-Basal-Layer/blob/main/README.md
- HuBMAP HRA 3D reference library page data: https://github.com/hubmapconsortium/hra-ui/blob/main/apps/humanatlas.io/public/assets/content/3d-reference-library-page/data.yaml
- HRA releases README: https://github.com/hubmapconsortium/ccf-releases
- open-twin-xr geometry sources survey (licences for BodyParts3D, Z-Anatomy, OpenAnatomy, ICRP, XCAT, ViP, UF/NCI, VIVA+, PIPER): https://github.com/Opening-Science/open-twin-xr/blob/main/docs/GEOMETRY_SOURCES_SURVEY.md
- open-twin-xr organ shape models (Segars 2014, Stepusin 2017, iPhantom, Anatomy Transfer, phantom licences): https://github.com/Opening-Science/open-twin-xr/blob/main/docs/research/ORGAN_SHAPE_MODELS.md
- phantomicrp (ICRP 110/143 voxel reader): https://github.com/kutsen/phantomicrp
- Companion notes in this folder: `skeleton_posture_motion_by_age.md`, `body_composition_fat_muscle_by_age.md`, `growth_stature_trajectories.md`, `proportions_by_age.md`
