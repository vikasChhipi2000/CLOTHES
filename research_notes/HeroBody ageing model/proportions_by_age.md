# Body proportions and all body dimensions by age, birth to old age (targets for HeroBody knobs)

Research date: 2026-10-09. Researcher notes for the report writer.

**Access notes.** In this session, WebFetch failed with DNS errors (`getaddrinfo ENOTFOUND`) for every non-GitHub host I tried: pmc.ncbi.nlm.nih.gov, www.ncbi.nlm.nih.gov, mdpi.com, cdc.gov, math.nist.gov, actaorthop.org. curl through the proxy was refused (403 CONNECT) for wwwn.cdc.gov, cdc.gov, who.int, openlab.psu.edu, humanshape.org, mreed.umtri.umich.edu, deepblue.lib.umich.edu, zenodo, figshare and web.archive.org. Only github.com and raw.githubusercontent.com could be reached. So the primary data below were taken from **GitHub copies of the original data files** and analysed with Python in the scratchpad:

- **Snyder et al. 1977 (CPSC/UM-HSRI) individual records**, ages 2 to 20 y. The NIST "AnthroKids" export has 3,900 subjects, 87 measurements plus centre-of-gravity data, and 123 columns. I took it from a student repository (`solcalloni/prediccion-talle-zapatos/chicos.csv`) that downloaded it from the NIST AnthroKids site. The column names and units match the 1977 study: mm, weight in 0.1 kg, 0 = not measured. Measurement sets 1, 2 and 3 cover different subsets of dimensions. For example, head height was measured on only about 1,266 subjects.
- **ANSUR II public CSVs**: 4,082 men and 1,986 women aged 17 to 58, 93 measurements. Taken from a copy in `keenon/nimblephysics`.
- **Dutch Fourth Growth Study (NL4, 1997; Fredriks et al. 2005)** LMS tables for height, sitting height, leg length, sitting-height ratio, waist, hip, waist-hip ratio, head circumference and weight, ages 0 to 21 y. From the R package `AGD` (mirror github.com/cran/AGD).
- **Berkeley Child Guidance Study** longitudinal data: 66 boys and 70 girls, 0 to 21 y, with stem (sitting) length, biacromial and bi-iliac diameters. From the R package `sitar` (github.com/cran/sitar).
- **childsds** reference LMS/BCCG tables: WHO 2006 (length, head and arm circumference, BMI), CDC 2000, Kromeyer-Hauschild, UK 1990, KiGGS, LIFE-Child circumferences, Valencia neck circumference, Sharma NHANES waist. From github.com/cran/childsds.
- **NHANES 2015-2018** waist, height, weight and age with MEC weights, ages 0 to 80+ (top-coded at 80). From the `mpower` package dataset `nhanes1518` (github.com/cran/mpower).
- **MIMo infant growth fits**: log-curve fits to the Snyder 1977 *infant* means (1 to 33 months), in `trieschlab/MIMo/mimoGrowth/data/params.json`.
- **Anny 0.6 source**: the WHO-calibrated "morphological age" anchors in `naver/anny/src/anny/data/shape_calibration/*.pth`.

Everything else comes from search-result snippets, tagged **[snippet]**. Tags: **[fetched]** = I read the primary file or code myself, including data files read from a GitHub copy. **[computed]** = I computed the number from fetched data with the script logic described. **[snippet]** = search snippet only. **[inference]** = my reasoning. Ratios are **median of individual ratios to stature** unless stated. "H" means stature (standing height; recumbent length under 2 y).

---

## 1. Head height / stature, sitting height ratio, leg length / stature, arm span / stature, by age and sex

### Takeaway
- **Body grows bottom-up.** The sitting height ratio (SH/H) falls from **0.69 at birth to 0.60 at 2 y, 0.54 at 6 y and 0.51 at about 12 to 14 y** (the minimum, just before or at the growth spurt). It then rises slightly to **0.515 in men and 0.524 in women at 18 y**. Subischial leg length (H minus SH) is the mirror image: 0.31 of H at birth, 0.40 at 2 y, 0.46 at 6 y, 0.48 to 0.49 at puberty.
- **Head height (vertex to chin) shrinks relative to H.** Measured values are 0.189 H at 2 y (5.3 heads), 0.16 at 6 y (6.2 heads), 0.143 at 10 y (7.0 heads), 0.13 at 14 y (7.6 heads) and 0.125 at 18 y (8.0 heads). Adults are 0.125 to 0.13 (7.7 to 8 heads). Newborns are about 0.24 to 0.25 (about 4 heads); that figure is the textbook value, not measured here. In old age the head stays the same size while stature falls, so heads-tall drops back to about 7.5 to 7.7 at 80 y.
- **Arm span is shorter than stature in infancy and childhood**: about 0.96 to 0.98 H from 0 to 3 y, and about 0.97 to 0.99 H from 4 to 10 y. It crosses 1.0 at about 12 y. Adults are **1.031 H (men) and 1.018 H (women)**. In old age span stays fixed while height falls, so span/H rises to about 1.05 to 1.07 by 80 y.
- **Arm length (acromion to wrist) / H** is nearly flat: 0.32 at 2 to 4 y, 0.33 at 8 y, 0.34 at 12 y and 0.343 in adults. Children's arms are only slightly shorter relative to height.

### Cited Findings
**Sitting height ratio and leg length, NL4 (Dutch 1997) LMS medians [computed from fetched NL4 tables]**

| age y | H M cm | SH/H M | LL/H M | H F cm | SH/H F | LL/H F |
|---|---|---|---|---|---|---|
| 0 | 51.3 | 0.692 | 0.308 | 50.9 | 0.693 | 0.307 |
| 0.25 | 61.2 | 0.682 | 0.318 | 59.6 | 0.683 | 0.317 |
| 0.5 | 68.0 | 0.671 | 0.329 | 66.4 | 0.672 | 0.328 |
| 1 | 76.5 | 0.648 | 0.352 | 75.1 | 0.648 | 0.352 |
| 2 | 88.8 | 0.604 | 0.396 | 87.5 | 0.601 | 0.399 |
| 3 | 98.1 | 0.572 | 0.428 | 96.7 | 0.568 | 0.432 |
| 4 | 105.8 | 0.555 | 0.445 | 104.5 | 0.553 | 0.447 |
| 6 | 120.1 | 0.541 | 0.459 | 118.7 | 0.541 | 0.459 |
| 8 | 132.8 | 0.531 | 0.469 | 131.5 | 0.531 | 0.469 |
| 10 | 143.2 | 0.520 | 0.480 | 143.3 | 0.522 | 0.478 |
| 12 | 154.0 | 0.513 | 0.487 | 155.3 | 0.516 | 0.484 |
| 14 | 168.2 | 0.508 | 0.492 | 164.7 | 0.518 | 0.482 |
| 16 | 178.7 | 0.512 | 0.488 | 168.6 | 0.524 | 0.476 |
| 18 | 182.6 | 0.515 | 0.485 | 169.8 | 0.524 | 0.476 |
| 21 | 184.0 | 0.513 | 0.487 | 170.6 | 0.526 | 0.474 |

Source: [AGD nl4.shh / nl4.sit / nl4.lgl / nl4.hgt on GitHub](https://github.com/cran/AGD/tree/master/data). The paper is Fredriks et al. 2005, Arch Dis Child 90:807-812 [fetched data; paper not fetched]. Below 2 y, SH is crown-rump length and H is recumbent length.

- **Snyder 1977 (US, 1975-77) cross-check of SH/H**, medians per sex at whole-year ages [computed from fetched individual data]:
  - Boys: 0.594 at 2, 0.584 at 3, 0.575 at 4, 0.553 at 6, 0.538 at 8, 0.526 at 10, 0.516 at 12, 0.510 at 14, 0.518 at 16, 0.520 at 18.
  - Girls: 0.586, 0.580, 0.564, 0.555, 0.539, 0.524, 0.518, 0.523, 0.528, 0.530.
  - Source: [Snyder 1977 AnthroKids export (GitHub copy)](https://github.com/solcalloni/prediccion-talle-zapatos). The original report is [Snyder et al. 1977, UM-HSRI-77-17, Deep Blue](https://deepblue.lib.umich.edu/items/9918421c-4112-4872-a654-b51292fd97cb) [fetched data; report not fetched].
- **Berkeley Child Guidance Study (born 1928-29; Tuddenham & Snyder 1954) SH/H** (corrected in verification: was "Berkeley Growth Study", which is a different cohort; sitar::berkeley is the Child Guidance Study) [computed from fetched data]: boys 0.535 at 8, 0.523 at 10, 0.512 at 12, 0.509 at 14, 0.514 at 16, 0.520 at 18; girls 0.532, 0.520, 0.513, 0.521, 0.523, 0.525. Source: [sitar::berkeley](https://github.com/cran/sitar/blob/master/man/berkeley.Rd) [fetched].
- **ANSUR II adults SH/H** [computed]:
  - Men 0.5236 (SD 0.0135); women 0.5270 (SD 0.0141).
  - By age band, men: 0.524 (17-24), 0.525 (25-34), 0.522 (35-44), 0.520 (45-58).
  - Women: 0.527, 0.528, 0.525, 0.525.
  - Crotch height/H 0.481 M / 0.480 F. Trochanterion height/H 0.512 / 0.519. Knee height (mid-patella)/H 0.278 / 0.275.
  - Source: [ANSUR II male/female public CSV (GitHub copy)](https://github.com/keenon/nimblephysics/tree/master/python/nimblephysics/models/rajagopal_data) [fetched].
- An Argentine reference reports a median SH/H of 0.67 at birth, falling to 0.57 at 4 y, then a plateau at 0.52 (boys) and 0.53 (girls) from 12 to 17 y — [SAP Arch Argent Pediatr 2017](https://sap.org.ar/docs/publicaciones/archivosarg/2017/v115n3a05e.pdf) [snippet].
- In NHANES III US children 2 to 18 y, SH/H falls from prepuberty to early puberty and rises in late puberty. Non-Hispanic Black children have lower SH/H, meaning relatively longer legs, at all ages — [Hawkes et al. 2020 / Data in Brief, PMC7452688](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7452688/) [snippet].
- Bogin and Varela-Silva: legs grow faster than other post-cranial segments from birth to puberty. Relative leg length is a marker of growth environment, and a high SH/H, meaning short legs, signals a poor environment — [Bogin & Varela-Silva 2010, IJERPH 7:1047](https://dspace.lboro.ac.uk/2134/14468) [snippet].

**Head height (vertex to menton) / H, Snyder 1977 [computed from fetched data; n per cell 7 to 55]** (corrected in verification: was "n per cell 14 to 61"; ages are binned to the nearest whole year, i.e. a ± 0.5 y; with floor binning the values shift by up to −0.004, e.g. boys 0.158 at 6 y, 0.139 at 10 y)

| age y | HH/H M | heads M | HH/H F | heads F | face height/H M | tragion-top/H M |
|---|---|---|---|---|---|---|
| 2 | 0.189 | 5.3 | 0.187 | 5.4 | 0.157 | 0.125 |
| 3 | 0.186 | 5.4 | 0.184 | 5.4 | 0.151 | 0.120 |
| 4 | 0.176 | 5.7 | 0.174 | 5.7 | 0.143 | 0.115 |
| 5 | 0.168 | 6.0 | 0.163 | 6.1 | 0.136 | 0.108 |
| 6 | 0.162 | 6.2 | 0.159 | 6.3 | 0.133 | 0.102 |
| 8 | 0.150 | 6.7 | 0.148 | 6.8 | 0.125 | 0.095 |
| 10 | 0.143 | 7.0 | 0.142 | 7.1 | 0.116 | 0.089 |
| 12 | 0.136 | 7.4 | 0.133 | 7.5 | 0.113 | 0.085 |
| 14 | 0.132 | 7.6 | 0.128 | 7.8 | 0.109 | 0.078 |
| 16 | 0.126 | 7.9 | 0.127 | 7.9 | 0.103 | 0.074 |
| 18 | 0.125 | 8.0 | 0.125 | 8.0 | 0.102 | 0.075 |

The robust SD of HH/H is 0.0096 at 2 to 4 y and 0.006 at 16 to 19 y. The 1977 child file starts at 2 y.

- **ANSUR II adult head height estimate** [computed; my construction, see Inferences]: (sitting height − eye height sitting) + menton-sellion length − 12 mm gives a median of 224 mm in men (0.1278 H, 7.8 heads, SD 0.0066) and 210 mm in women (0.129 H, 7.75 heads). The direct ANSUR tragion-to-top-of-head is 131 mm in men, which matches Snyder 18 y (0.0754 × 1769 = 133 mm).
- Textbook proportions: head about 1/4 of length at birth (some sources give 1/4 to 1/3), about 1/5 at 2 y, about 1/6 at 5 to 6 y, and 1/7 to 1/8 in adults — [LibreTexts Child Growth 4.02](https://socialsci.libretexts.org/Bookshelves/Early_Childhood_Education/Child_Growth_and_Development_(Paris_Ricardo_Rymond_and_Johnson)/04%3A_Physical_Development_in_Infancy_and_Toddlerhood/4.02%3A_Proportions_of_the_Body), [Scientific American "Human body ratios"](https://www.scientificamerican.com/article/human-body-ratios) [snippet].
- Farkas (1994, *Anthropometry of the Head and Face*) has v-gn norms by age, but I could not retrieve the table values. About 30 subjects per age group — [epublications.vu.lt summary](https://epublications.vu.lt/object/elaba:2133010/2133010.pdf) [snippet].

**Arm span / H**
- ANSUR II adults: span/H **1.0311 M (SD 0.027), 1.0180 F (SD 0.029)** [computed]. Regression: span = 5.80 + 0.976·H + 0.042·W + 0.022·age (cm, kg, y), R² 0.68 [computed].
- Chinese infants and toddlers (n = 1,101 full-term): arm span / body length 0.962 to 0.976 in boys and 0.957 to 0.973 in girls, rising with age — [Chin J Child Health Care 2025-0696](https://cjchc.xjtu.edu.cn/EN/10.11852/zgetbjzz2025-0696) [snippet].
- Turkish children 3 to 18 y (n = 1,302): span < height at young ages, slightly greater than height from about 12 y — [Turk J Pediatr](https://turkjpediatr.org/article/view/3) [snippet].
- An older study (4 to 16 y): H/span changed linearly from 1.03 to 1.00 in girls and from 1.03 to 0.98 in boys, so span/H ≈ 0.97 to 1.02 — [hrcak.srce.hr 5221](https://hrcak.srce.hr/en/5221) [snippet].
- In older adults, arm span exceeds height in every age group of 60+ (Delhi, n = 711), and span minus height grows with height loss, for example 1.9 to 5.4 cm in Japanese women — [Indian J Public Health 2018 (DOAJ)](https://doaj.org/article/ca5dcbe4047e4a53a38fade966229ed7), [nsg.repo.nii.ac.jp 3620](https://nsg.repo.nii.ac.jp/records/3620) [snippet].

**Arm length (acromion to wrist, radiale-stylion) / H [computed]**
- Snyder: boys 0.322 at 2, 0.320 at 3.5, 0.329 at 6, 0.333 at 8, 0.337 at 10, 0.343 at 12, 0.342 at 14 to 16, 0.345 at 18; girls 0.314, 0.318, 0.316, 0.319, 0.322, 0.327, 0.330, 0.327, 0.325.
- ANSUR adults: 0.343 M, 0.339 F. Acromion to fingertip: 0.453 M, 0.450 F.

**Height loss in old age (needed for all elderly ratios)**
- Baltimore Longitudinal Study of Aging: height loss starts at about 30 y. Cumulative loss from 30 to 70 y is about 3 cm in men and 5 cm in women; by 80 y about 5 cm and 8 cm — [Sorkin, Muller & Andres 1999, Am J Epidemiol 150:969](https://www.proquest.com/docview/224823953) via a [2025 Frontiers review](https://www.frontiersin.org/journals/endocrinology/articles/10.3389/fendo.2025.1542962/pdf) [snippet; the 80 y figures come from the secondary source].
- Cross-sectional height medians also include birth-cohort effects. Kromeyer-Hauschild German reference medians: men 181.0 (20 y), 177.5 (50 y), 174.5 (70 y), 172.0 (90 y); women 167.3, 165.5, 163.5, 161.0 — [childsds kro.ref](https://github.com/cran/childsds/tree/master/data) [computed]. NHANES 2015-18 US weighted medians: men 176.7 (25-34), 176.2 (45-54), 173.4 (65-74), 170.3 (80+); women 163.1, 161.8, 159.8, 156.2 — [mpower::nhanes1518](https://github.com/cran/mpower) [computed].

### Inferences
- **Classical "heads tall" by age**, combining Snyder for 2 to 18 y, the ANSUR estimate for adults, textbook values for 0 y and head-circumference scaling for 1 y: **4.0 to 4.2 (birth), 4.8 (1 y), 5.3 (2 y), 5.5 (3 to 4 y), 6.2 (6 y), 6.7 (8 y), 7.0 (10 y), 7.4 (12 y), 7.6 (14 y), 7.9 to 8.0 (16 to 18 y), 7.8 to 7.9 (adult), 7.7 (65 y), 7.5 to 7.6 (80 y)** [inference].
  - The 1 y value comes from HH ≈ 0.34 × head circumference. Snyder gives HH/HC = 0.349 at 2 y. NL4 HC at 1 y is 47.2 cm, so HH ≈ 16.0 cm over 76.5 cm, which is 0.21.
  - The same scaling at birth gives 0.23 (4.3 heads), a little under the textbook 0.25.
- **Adult head height**: two independent routes agree within 3%: Snyder 18 y direct 0.125 versus ANSUR construction 0.128 to 0.129. Use **0.127 ± 0.006 (1 SD)** for both sexes. The ANSUR construction assumes sellion sits about 12 mm above the eye-height landmark (ectocanthus) [inference].
- **Elderly SH/H**: about 70 to 80% of senile height loss is trunk (disc thinning, vertebral wedging, kyphosis). The rest is posture of the hips and knees and flattening of the foot arch.
  - Applying this to Sorkin's losses gives SH/H ≈ **0.518 (M) / 0.520 (F) at 65 y** and **0.512 / 0.513 at 80 y**, with leg/H rising by the same amount [inference].
  - Cross-sectional data often hide this, because older cohorts had relatively shorter legs (secular trend). For example, the 1960-62 US survey gives an erect sitting height of 34.2 in (86.9 cm) for men aged 75-79 — [NCHS sr11_008](https://www.cdc.gov/nchs/data/series/sr_11/sr11_008.pdf) [snippet].
- **Span is not 2 × arm + biacromial.** In ANSUR, 2 × 0.453 + 0.236 = 1.14 H, but measured span = 1.03 H. The shoulder girdle moves when the arms are abducted, so the solver should use span and arm length as separate targets with their own definitions [computed + inference].

### Gaps
- No measured head height under 2 y. The 1977 infant file (2 weeks to 2 y) and the Farkas tables were not reachable.
- No direct elderly SH/H or head-height data. The NHANES III exam file (sitting height to 90+) was blocked.
- Bodyspace (Pheasant) tables were not accessed.
- Snyder's "head height" landmark definition was not re-read from the report (assumed vertex-menton).

---

## 2. Breadths and circumferences as ratios of stature by age and sex

### Takeaway
- **Infants are wide and round.** At birth, hip (buttock) circumference ≈ 0.60 H, waist ≈ 0.63 H and **waist-hip ratio ≈ 1.11**: the belly is bigger than the hips. Chest ≈ 0.66 H and head circumference 0.69 H.
  - By 2 y: waist 0.53 H, hip 0.55 H, chest 0.55 H, WHR 0.96 to 0.97.
  - All girths fall in proportion to H until about 8 to 12 y, the slimmest stage: waist ≈ 0.41 to 0.42 H, chest 0.48 to 0.49 H, hip 0.48 to 0.50 H, neck 0.20 H.
  - They rise again after puberty and keep rising through adult life (waist/H 0.50 to 0.53 at 20 to 30 y, 0.58 to 0.60 at 45 to 55 y, 0.61 to 0.63 at 65 to 80 y in the US).
- **Shoulder width (biacromial) is a near-constant fraction of H**: about 0.215 to 0.23 from 2 to 16 y in both sexes. At puberty it rises only in boys, to 0.236 in men; women stay at 0.225.
- **Pelvis width (bi-iliac)** is 0.16 H in childhood in both sexes. It rises only in girls, to 0.17 at 14 to 18 y; adult women are 0.167 and men 0.157.
- **The shoulder/hip divergence at puberty** in the Berkeley data, biacromial/bi-iliac: boys **1.35 at 8 y → 1.44 at 18 y**; girls **1.34 at 8 y → 1.27 at 15 to 18 y**. The curves split at 12 to 13 y.
  - Adults (ANSUR): biacromial/bicristal 1.51 M vs 1.34 F. Biacromial/hip breadth 1.20 vs 1.04. Bideltoid/hip breadth 1.48 vs 1.27. WHR 0.92 vs 0.84.
- **Old age**: waist keeps growing. Bi-iliac breadth grows with age, and the apparent effect is magnified as stature falls. Shoulder width is flat to slightly narrower.

### Cited Findings
**Breadths / H, Snyder 1977 (2 to 18 y) [computed from fetched data]**

| age | biac M | biac F | bideltoid M | bideltoid F | hip br. (trochanter) M | hip br. F | chest br. M |
|---|---|---|---|---|---|---|---|
| 2 | 0.229 | 0.238* | 0.262 | 0.271 | 0.191 | 0.197 | 0.180 |
| 3 | 0.232 | 0.229 | 0.259 | 0.261 | 0.188 | 0.188 | 0.173 |
| 4 | 0.228 | 0.228 | 0.253 | 0.255 | 0.183 | 0.185 | 0.164 |
| 6 | 0.224 | 0.224 | 0.247 | 0.241 | 0.174 | 0.176 | 0.156 |
| 8 | 0.218 | 0.218 | 0.241 | 0.242 | 0.171 | 0.176 | 0.149 |
| 10 | 0.221 | 0.217 | 0.242 | 0.237 | 0.170 | 0.172 | 0.150 |
| 12 | 0.216 | 0.218 | 0.236 | 0.235 | 0.174 | 0.181 | 0.149 |
| 14 | 0.215 | 0.218 | 0.241 | 0.239 | 0.182 | 0.190 | 0.153 |
| 16 | 0.218 | 0.218 | 0.246 | 0.244 | 0.185 | 0.197 | 0.158 |
| 18 | 0.225 | 0.222 | 0.257 | 0.244 | 0.184 | 0.199 | 0.165 |

\*n = 8. Biacromial was measured in one measurement set only (n 21 to 53 per cell); other breadths n 60 to 155.

**Berkeley longitudinal: biacromial and bi-iliac / H [computed from fetched sitar::berkeley]**

| age | biac/H M | biil/H M | biac/biil M | biac/H F | biil/H F | biac/biil F |
|---|---|---|---|---|---|---|
| 6 | – | 0.162 | – | – | 0.161 | – |
| 8 | 0.219 | 0.161 | 1.352 | 0.214 | 0.162 | 1.342 |
| 10 | 0.215 | 0.159 | 1.357 | 0.217 | 0.161 | 1.348 |
| 12 | 0.216 | 0.157 | 1.372 | 0.216 | 0.162 | 1.337 |
| 13 | 0.217 | 0.158 | 1.376 | 0.215 | 0.166 | 1.295 |
| 14 | 0.219 | 0.158 | 1.389 | 0.215 | 0.169 | 1.275 |
| 15 | 0.220 | 0.157 | 1.407 | 0.217 | 0.171 | 1.272 |
| 16 | 0.224 | 0.158 | 1.422 | 0.219 | 0.172 | 1.268 |
| 18 | 0.229 | 0.159 | 1.438 | 0.220 | 0.171 | 1.285 |

Absolute values at 18 y: biacromial 40.7 cm M / 36.6 F; bi-iliac 28.6 / 28.3 cm.

**Circumferences / H, Snyder 1977 [computed]**

| age | chest M | chest F | waist M | waist F | hip M | hip F | neck M | neck F | upper arm M | thigh M | thigh F | calf M |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 2 | 0.553 | 0.557 | 0.521 | 0.539 | 0.573 | 0.569 | 0.261 | 0.270 | 0.174 | 0.309 | 0.320 | 0.223 |
| 3 | 0.536 | 0.533 | 0.509 | 0.508 | 0.539 | 0.546 | 0.252 | 0.252 | 0.167 | 0.303 | 0.315 | 0.217 |
| 4 | 0.523 | 0.515 | 0.485 | 0.485 | 0.518 | 0.532 | 0.246 | 0.238 | 0.158 | 0.296 | 0.311 | 0.206 |
| 6 | 0.501 | 0.495 | 0.453 | 0.451 | 0.500 | 0.511 | 0.226 | 0.223 | 0.149 | 0.288 | 0.299 | 0.199 |
| 8 | 0.492 | 0.489 | 0.436 | 0.442 | 0.495 | 0.513 | 0.212 | 0.212 | 0.144 | 0.286 | 0.307 | 0.198 |
| 10 | 0.485 | 0.483 | 0.430 | 0.432 | 0.495 | 0.512 | 0.204 | 0.198 | 0.144 | 0.291 | 0.308 | 0.199 |
| 12 | 0.484 | 0.483 | 0.428 | 0.425 | 0.499 | 0.518 | 0.201 | 0.194 | 0.147 | 0.296 | 0.306 | 0.199 |
| 14 | 0.494 | 0.493 | 0.423 | 0.430 | 0.509 | 0.545 | 0.197 | 0.187 | 0.150 | 0.302 | 0.326 | 0.203 |
| 16 | 0.509 | 0.505 | 0.417 | 0.438 | 0.513 | 0.554 | 0.196 | 0.190 | 0.158 | 0.300 | 0.331 | 0.203 |
| 18 | 0.533 | 0.505 | 0.442 | 0.441 | 0.528 | 0.563 | 0.207 | 0.191 | 0.165 | 0.310 | 0.332 | 0.207 |

- **NL4 Dutch LMS medians (0 to 21 y) [computed]**:
  - Waist/H, boys: 0.631 (0), 0.618 (0.5), 0.579 (1), 0.528 (2), 0.507 (3), 0.484 (4), 0.444 (6), 0.425 (8), 0.418 (10), 0.415 (12), 0.406 (14), 0.404 (16), 0.414 (18), 0.433 (21).
  - Waist/H, girls: 0.630, 0.616, 0.575, 0.530, 0.509, 0.484, 0.442, 0.423, 0.411, 0.402, 0.398, 0.403, 0.410, 0.418.
  - Hip/H, boys: 0.604, 0.608, 0.584, 0.546, 0.524, 0.512, 0.492, 0.483, 0.492, 0.495, 0.498, 0.502, 0.506, 0.511.
  - Hip/H, girls: 0.592, 0.619, 0.590, 0.553, 0.538, 0.525, 0.502, 0.498, 0.503, 0.509, 0.526, 0.537, 0.548, 0.554.
  - **WHR (tabulated LMS median)**, boys: 1.112 (0), 1.013 (0.5), 0.988 (1), 0.968 (2), 0.955 (3.5), 0.905 (6), 0.878 (8), 0.855 (10), 0.838 (12), 0.825 (14), 0.820 (16), 0.824 (18). Girls: 1.112, 0.997, 0.973, 0.959, 0.935, 0.879, 0.849, 0.820, 0.792, 0.768, 0.755, 0.750.
  - Source: [AGD nl4.wst / nl4.hip / nl4.whr](https://github.com/cran/AGD/tree/master/data) [fetched data].
- **LIFE-Child Leipzig, 3 to 18 y (BCCG medians) [computed]**: thigh circumference 27.4 cm (3 y) → 38.8 (10 y) → 52.6 (18 y, M) / 47.9 (18 y, F). Waist/H medians fall from 0.508 at 3 y to 0.415 to 0.42 at 9 to 12 y, then 0.427 (M) / 0.406 (F) at 18 y — [childsds life_circ.ref (Roennecke et al. 2019, Obes Facts)](https://github.com/cran/childsds/tree/master/data) [fetched data].
- **Neck circumference, Spanish children 6 to 11 y**: 25.7 to 28.8 cm (M), 24.6 to 28.3 cm (F) — [childsds valencia_nc.ref](https://github.com/cran/childsds/tree/master/data) [fetched data].
- **WHO mid-upper-arm circumference**: 14.2 cm at 0.5 y, 14.6 at 1 y, 15.2 at 2 y, 16.1 at 4 y, 16.5 at 5 y (boys). Divided by WHO length: 0.21, 0.19, 0.17, 0.16, 0.15 — [childsds who.ref / WHO 2006](https://github.com/cran/childsds/tree/master/data) [computed].
- **ANSUR II adults / H** (median, SD) [computed]:

| dim | men | women |
|---|---|---|
| biacromial | 0.2365 (0.0098) | 0.2245 (0.0096) |
| bideltoid | 0.290 (0.018) | 0.276 (0.017) |
| bicristal (bi-iliac) | 0.157 (0.009) | 0.167 (0.013) |
| hip breadth (standing) | 0.196 (0.013) | 0.217 (0.015) |
| chest breadth | 0.165 | 0.165 |
| chest circ | 0.603 (0.049) | 0.576 (0.050) |
| waist circ | 0.534 (0.063) | 0.524 (0.060) |
| buttock (hip) circ | 0.581 (0.042) | 0.625 (0.044) |
| neck circ | 0.226 (0.016) | 0.202 (0.012) |
| biceps circ (flexed) | 0.204 (0.020) | 0.186 (0.019) |
| forearm circ (flexed) | 0.177 | 0.162 |
| thigh circ | 0.356 (0.033) | 0.377 (0.033) |
| calf circ | 0.223 (0.017) | 0.228 (0.017) |
| ankle circ | 0.130 | 0.132 |
| wrist circ | 0.100 | 0.095 |
| head circ | 0.327 | 0.344 |
| waist depth / waist breadth | 0.134 / 0.186 | 0.128 / 0.183 |
| chest depth | 0.145 | 0.151 |
| buttock depth | 0.140 | 0.142 |

- **ANSUR age trend (17-24 → 25-34 → 35-44 → 45-58)** [computed]:
  - Waist/H: men 0.497 → 0.534 → 0.564 → 0.576; women 0.505 → 0.526 → 0.549 → 0.569.
  - Chest/H: men 0.578 → 0.603 → 0.624 → 0.630; women 0.562 → 0.577 → 0.597 → 0.621.
  - Hip circ/H: men 0.565 → 0.583 → 0.590 → 0.593; women 0.615 → 0.627 → 0.638 → 0.648.
  - Bicristal/H: men 0.155 → 0.158 → 0.158 → 0.159; women 0.165 → 0.168 → 0.172 → 0.176.
  - Biacromial/H is flat: men 0.236 to 0.237, women 0.223 to 0.225.
  - Neck/H: men 0.221 → 0.233; women 0.199 → 0.212.
- **NHANES 2015-2018 (US, MEC-weighted medians) waist/H** [computed from fetched mpower::nhanes1518]:

| age | men WC/H | men WC cm | men BMI | women WC/H | women WC cm | women BMI |
|---|---|---|---|---|---|---|
| 6-8 | 0.463 | 58.3 | 16.4 | 0.465 | 57.4 | 16.5 |
| 9-11 | 0.467 | 66.6 | 18.7 | 0.466 | 67.9 | 18.7 |
| 12-14 | 0.445 | 72.9 | 20.6 | 0.483 | 77.0 | 22.2 |
| 15-17 | 0.456 | 79.1 | 22.5 | 0.489 | 79.0 | 23.2 |
| 18-24 | 0.505 | 88.3 | 25.7 | 0.534 | 86.5 | 25.3 |
| 25-34 | 0.546 | 95.5 | 27.9 | 0.566 | 91.3 | 27.1 |
| 35-44 | 0.575 | 100.7 | 29.0 | 0.594 | 96.2 | 29.0 |
| 45-54 | 0.585 | 103.0 | 29.1 | 0.595 | 97.1 | 28.5 |
| 55-64 | 0.599 | 104.6 | 29.1 | 0.614 | 98.8 | 29.3 |
| 65-74 | 0.610 | 106.0 | 28.5 | 0.631 | 100.6 | 29.0 |
| 75-79 | 0.621 | 106.0 | 28.0 | 0.637 | 99.9 | 28.6 |
| 80+ | 0.610 | 104.2 | 27.1 | 0.627 | 98.5 | 27.5 |

- Bi-iliac breadth increases with age, and this more than offsets the stature decline that starts in the 30s to 40s — [Ruff et al. 2017 body-mass estimation, via search](https://pmc.ncbi.nlm.nih.gov/articles/PMC5198355/table/T3) [snippet]. A Japanese rural sample (n > 3,600, 14+ y) showed an age-related increase in bi-iliac breadth with little sex difference — [PubMed 3228169](https://pubmed.ncbi.nlm.nih.gov/3228169) [snippet]. In Japanese data, older groups were smaller in stature and shoulder width and larger in waist girth [snippet, source unclear in the result].
- NL4 waist/H and hip/H for Dutch teens (0.41 / 0.51 at 18 y M) are leaner than US NHANES teens (0.456 at 15-17 y) [computed]. Population choice matters by ±10% on girths.

### Inferences
- **Puberty in numbers.** Between 10 and 18 y:
  - Male biacromial/H rises by about 0.012 to 0.015 (to 0.23 at 18, 0.236 adult), with no rise in bi-iliac/H.
  - Female bi-iliac/H rises by 0.010 (0.161 → 0.171) and hip circ/H by 0.05 (0.51 → 0.56), with biacromial flat.
  - So the shoulder:pelvis ratio diverges by about 0.17 (1.44 vs 1.27) [computed + inference].
  - For HeroBody, puberty should drive these as two separate sex-specific curves between the teen stage start and adult start. They should not be a uniform scale.
- **Elderly** [inference]. Hold absolute biacromial width, head size, hand and foot length, and span constant from 45 y. Reduce stature per Sorkin. Raise waist/H to the NHANES 65-80 values, raise bi-iliac/H by +0.002 to 0.004 per decade, and lower thigh and calf circumference by about 2 to 5% (sarcopenia). This reproduces the cross-sectional pattern: wider pelvis, thicker waist, thinner limbs, same shoulders, shorter trunk.
- **Infant chest** [inference from snippets]: newborn chest ≈ head circumference − 1.5 to 2 cm, so ≈ 0.65 to 0.66 H. Chest equals head at about 1 to 2 y.

### Gaps
- Biacromial and chest under 2 y were not measured in the data I reached. Only the newborn chest/head relation is from snippets.
- No elderly breadth data (65+) were fetched; NHANES III bitrochanteric/biacromial files were blocked.
- Neck circumference under 2 y: no data.
- ANSUR is a fit military population. Its girths differ from civilians: about +5% chest in men, and less fat in the waist of young adults.

---

## 3. Statistical body-shape models and regression methods (few inputs → full consistent measurement set)

### Takeaway
- The best-documented open methods are the UMTRI regressions and statistical body shape models (SBSM). Predictors are stature, BMI or weight, age and sex, sometimes plus sitting-height ratio. The method is PCA on landmarks or meshes plus linear regression on predictors plus a stochastic residual.
- The UMTRI HumanShape models are **free for non-commercial use only**. The UMTRI 3D-child technical report is CC BY-NC-ND.
- For HeroBody, the practical and license-clean route is to **rebuild the regression yourself from public raw data**:
  - ANSUR II for adults (cleared for unlimited public release);
  - Snyder 1977 individual data for children 2 to 18 y (US government-funded contract data distributed by NIST; no explicit licence seen);
  - NL4/WHO/CDC LMS tables for the medians.
- I ran this regression here (tables below). Lengths are explained by stature (R² 0.45 to 0.97; hand and foot length only 0.45 to 0.55). Girths are explained mainly by weight (R² 0.61 to 0.88; men 0.68 to 0.88, women 0.61 to 0.85); breadths less well (bideltoid and hip breadth 0.69 to 0.79, bicristal 0.48 to 0.50, biacromial 0.36 to 0.43). (Corrected in verification: was "lengths R² 0.6 to 0.97; girths and breadths R² 0.7 to 0.9".) Head size is poorly predicted (R² 0.2 to 0.5).
- The conditional-Gaussian formula then fills any missing measurement from any known subset.

### Cited Findings
- **UMTRI HumanShape**: tools generate 3D shapes of standing and seated children and seated toddlers, plus downloadable models for specific stature/weight combinations — [UMTRI Virtual Child Models](https://www.umtri.umich.edu/?p=3479) [snippet]. The models predict body shape, standard anthropometry and landmarks from stature, BMI, age and sex — [UMTRI Human Computational Modeling](https://www.umtri.umich.edu/human-computational-modeling) [snippet]. Licence: a free non-commercial licence; commercial use and incorporation into software products are not permitted — [U-M available inventions: online body shape models](https://available-inventions.umich.edu/product/online-body-shape-models/print) [snippet].
- **UMTRI 3D child report**: whole-body laser scans of 150 children aged 4 to 12, with models parameterised by stature, body weight and erect sitting height. The report licence is CC BY-NC-ND 4.0 — [Deep Blue "Development of Three Dimensional Anthropometric Models of Seated and Standing Children"](https://deepblue.lib.umich.edu/items/5821a113-13f3-4ab5-b883-e0c45ffcf541), [Deep Blue record 2027.42/174130](https://deepblue.lib.umich.edu/handle/2027.42/174130?show=full) [snippet].
- **YouthShape.US**: a newer standing model for ages 3 to 17 from more than 2,300 children and youth, with demo access planned — [Matt Reed's page](https://mreed.umtri.umich.edu/mreed/), [Human Factors DOI 10.1177/00187208261478468](https://www.citedrive.com/en/discovery/youthshapeus-a-new-anthropometric-resource-for-us-children-and-youth/) [snippet].
- **Older and obese adults**: Park, Jones, Ebert & Reed (2022, *Ergonomics*, doi 10.1080/00140139.2021.1992020) built a parametric seated adult body shape model that includes age. UMTRI morphs GHBMC human models to obese and elderly targets with an SBSM of about 200 scans, using 45 landmarks and an RBF thin-plate spline — [UMTRI profile of B-K. Park](https://www.umtri.umich.edu/people/park-byoung-keon-daniel), [IBRC 2024 poster](https://ibrc.osu.edu/wp-content/uploads/2024/05/2024-IBRC-Poster_Neeluru-et-al.pdf) [snippet].
- **Parkinson & Reed, "Creating virtual user populations by analysis of anthropometric data"** (Int J Ind Ergon, 2010): PCA plus linear regression to combine databases, match a target population, and add a stochastic component so all variance is kept — [OPEN Design Lab](https://www.openlab.psu.edu/2010/01/01/creating-virtual-user-populations-by-analysis-of-anthropometric-data/) [snippet]. Nadadur & Parkinson (SAE 2008-01-1858) use stature and BMI as predictors in one regression and model the residual variance not explained by them — [SAE](https://saemobilus.sae.org/content/2008-01-1858) [snippet].
- **ANSUR II**: cleared for unlimited public release in 2017: 4,082 men and 1,986 women, 93 directly measured dimensions plus 15 demographic variables. The 3D scans are withheld for privacy — [army.mil 2017 release note](https://www.army.mil/article/188601/for_good_measure_natick_releases_raw_data_from_army_wide_anthropometric_survey), [ANSUR II overview memo](https://ph.health.mil/PHC%20Resource%20Library/ANSURIIDatabasesOverview.pdf) [snippet]. Penn State notes it is not representative of US civilians — [openlab.psu.edu/ansur2](https://www.openlab.psu.edu/ansur2/) [snippet]. I read the CSVs: 108 columns including age [fetched].
- **CAESAR**: no public licence. It is available through the WEAR subscription (CAESAR North America data); a separate WEAR 3D set of 13,200 scans costs 1,000 € (500 € for members) — [bodysizeshape.com](https://bodysizeshape.com/) [snippet].
- **SMIL (infant SMPL)**: research-only licence (PS:License 1.0) — [MPI SMIL page](https://ps.is.tuebingen.mpg.de/code/skinned-multi-infant-linear-model-smil) [snippet]. **AGORA kid model**: SMPL-X cannot represent children, so AGORA interpolates between the SMPL-X adult template and the SMIL infant template. The SMPL-X licence is non-commercial — [AGORA project](https://agora.is.tue.mpg.de/), [arXiv 2104.14643](https://arxiv.org/pdf/2104.14643) [snippet].
- **Anny 0.6** (Apache-2.0 code, CC0 MakeHuman assets): blendshape age anchors are `linspace(-1/3, 1, 5)` for newborn, baby, child, young, old — [naver/anny phenotype.py](https://github.com/naver/anny/blob/main/src/anny/models/phenotype.py) [fetched].
  - The "newborn" blendshape is "a scaled down version of the baby blend shapes" with "empirical values" — [full_model.py](https://github.com/naver/anny/blob/main/src/anny/models/full_model.py) [fetched].
  - `SimpleShapeDistribution` uses a handcrafted morphological-age mapping "calibrated to match the height vs age distribution of WHO data". Its anchors, read from `shape_calibration/boys.pth` and `girls.pth` (identical for both), are Anny age 0, 0.05, 0.215, 0.415, 0.67, 0.77, 0.83, 1.0 ↔ **0, 1, 4, 11, 16, 18, 64, 110 years** [fetched].
  - Height and weight are conditional Beta distributions given age, so Anny's age knob is calibrated for *stature only*, not for proportions [fetched + inference].

**Regression, adults (ANSUR II): dim(cm) = b0 + bH·H(cm) + bW·W(kg) + bA·age(y)** [computed]

| dim | men b0 / bH / bW / bA | R² | RMSE cm | R² (H only) | women b0 / bH / bW / bA | R² | RMSE |
|---|---|---|---|---|---|---|---|
| sitting height | 22.90 / 0.388 / 0.018 / −0.025 | 0.61 | 2.22 | 0.61 | 22.50 / 0.390 / 0.004 / −0.022 | 0.59 | 2.12 |
| span | 5.80 / 0.976 / 0.042 / 0.022 | 0.68 | 4.77 | 0.68 | −0.97 / 0.998 / 0.067 / 0.001 | 0.68 | 4.72 |
| biacromial | 21.03 / 0.089 / 0.061 / −0.012 | 0.43 | 1.44 | 0.28 | 13.93 / 0.125 / 0.040 / −0.016 | 0.36 | 1.47 |
| bideltoid | 43.28 / −0.061 / 0.215 / 0.001 | 0.79 | 1.49 | 0.10 | 38.95 / −0.064 / 0.246 / −0.006 | 0.76 | 1.42 |
| hip breadth | 23.62 / −0.008 / 0.147 / −0.007 | 0.72 | 1.28 | 0.15 | 25.90 / −0.030 / 0.209 / 0.008 | 0.69 | 1.49 |
| bicristal | 11.12 / 0.061 / 0.070 / −0.011 | 0.50 | 1.24 | 0.26 | 15.55 / 0.014 / 0.131 / 0.023 | 0.48 | 1.61 |
| chest circ | 98.19 / −0.269 / 0.602 / 0.111 | 0.88 | 3.09 | 0.06 | 93.46 / −0.305 / 0.698 / 0.126 | 0.75 | 4.14 |
| waist circ | 92.76 / −0.396 / 0.759 / 0.198 | 0.87 | 4.07 | 0.04 | 96.34 / −0.450 / 0.882 / 0.114 | 0.77 | 4.76 |
| buttock circ | 83.17 / −0.150 / 0.544 / −0.047 | 0.88 | 2.60 | 0.12 | 84.39 / −0.177 / 0.685 / 0.007 | 0.85 | 2.95 |
| neck circ | 39.36 / −0.079 / 0.157 / 0.029 | 0.68 | 1.46 | 0.04 | 29.69 / −0.043 / 0.143 / 0.020 | 0.61 | 1.21 |
| biceps circ fl. | 38.35 / −0.124 / 0.227 / −0.006 | 0.71 | 1.88 | 0.04 | 33.28 / −0.136 / 0.279 / 0.019 | 0.80 | 1.39 |
| thigh circ | 63.78 / −0.201 / 0.429 / −0.088 | 0.87 | 2.13 | 0.07 | 59.36 / −0.198 / 0.519 / −0.022 | 0.83 | 2.27 |
| calf circ | 36.49 / −0.074 / 0.194 / −0.028 | 0.71 | 1.59 | 0.07 | 32.40 / −0.059 / 0.232 / −0.041 | 0.66 | 1.66 |
| hand length | 2.56 / 0.090 / 0.008 / 0.009 | 0.47 | 0.72 | 0.45 | 2.90 / 0.086 / 0.017 / 0.004 | 0.45 | 0.75 |
| hand breadth | 4.51 / 0.018 / 0.012 / 0.003 | 0.35 | 0.35 | 0.22 | 4.10 / 0.018 / 0.011 / 0.001 | 0.30 | 0.32 |
| foot length | 4.76 / 0.119 / 0.018 / −0.005 | 0.54 | 0.88 | 0.52 | 3.93 / 0.117 / 0.025 / −0.003 | 0.55 | 0.83 |
| head circ | 48.50 / 0.028 / 0.053 / −0.019 | 0.28 | 1.36 | 0.12 | 45.74 / 0.045 / 0.055 / −0.024 | 0.17 | 1.77 |
| crotch height | −20.64 / 0.619 / −0.038 / −0.007 | 0.75 | 2.34 | 0.74 | −19.52 / 0.616 / −0.027 / −0.023 | 0.73 | 2.33 |
| cervicale height | −6.48 / 0.887 / 0.022 / 0.018 | 0.97 | 1.06 | 0.97 | −8.56 / 0.906 / 0.007 / 0.002 | 0.97 | 1.05 |
| knee height | −11.66 / 0.344 / −0.002 / 0.007 | 0.70 | 1.55 | 0.69 | −10.28 / 0.335 / 0.006 / 0.004 | 0.71 | 1.39 |
| shoulder-elbow | −2.33 / 0.220 / −0.003 / 0.012 | 0.67 | 1.04 | 0.67 | −2.19 / 0.216 / 0.005 / 0.004 | 0.67 | 0.98 |

Note that ANSUR weight is stored in hectograms; I divided by 10.

**Residual correlations after removing H, W and age (ANSUR men / women)** [computed]. These are needed to sample individual variation that stays consistent:
- hand length – foot length **0.58 / 0.61**
- hip breadth – buttock circumference **0.76 / 0.83**
- sitting height – foot length **−0.26 / −0.31**
- sitting height – hand length **−0.29 / −0.37** (long trunk goes with short extremities)
- waist – hip breadth 0.32 / 0.09
- waist – biacromial −0.22 / −0.07
- biacromial – hand length 0.15 / 0.23
- chest – waist 0.13 / 0.35

**Regression, children (Snyder 1977), same form, by sex and age band** [computed]. The bH, bW and bA coefficients are in cm per cm, cm per kg and cm per year. Selected examples, boys 7 to 12 y:
- erect sitting height = 20.57 + 0.370 H + 0.094 W − 0.238 age (R² 0.85, RMSE 1.6)
- biacromial = 8.67 + 0.137 H + 0.083 W − 0.046 age (R² 0.76, RMSE 1.0)
- hip breadth = 13.66 + 0.010 H + 0.236 W + 0.128 age (R² 0.89, RMSE 0.8)
- waist = 64.16 − 0.290 H + 1.124 W − 0.095 age (R² 0.87, RMSE 2.7)
- chest = 53.90 − 0.125 H + 0.804 W + 0.417 age (R² 0.86)
- hand length = 0.77 + 0.099 H + 0.021 W (R² 0.76, RMSE 0.6)
- foot length = 0.75 + 0.143 H + 0.038 W (R² 0.82, RMSE 0.7)
- head height = 12.82 + 0.045 H + 0.028 W − 0.031 age (R² 0.33, RMSE 0.8)

Girls 12 to 19 y:
- hip circumference = 51.23 − 0.060 H + 0.770 W + 0.455 age (R² 0.91)
- biacromial = 16.22 + 0.068 H + 0.098 W + 0.184 age (R² 0.56)

All bands: 2-7, 7-12, 12-19 y × sex × 17 dimensions. The full printout is reproducible with the scratchpad script `regkids.py`.

### Inferences
- **Conditional Gaussian fill-in** (the core of a Parkinson/Reed-style generator) [inference]. Stack the predictors and measurements into a vector x, for example log values of H, W, SH, biacromial, waist, and so on. Within an age-sex cell, x ~ N(μ, Σ). Given known components x_k (for example H and W from the profile, or SH/H measured on the blueprint), the unknown components are:
  `x_u | x_k ~ N( μ_u + Σ_uk Σ_kk⁻¹ (x_k − μ_k),  Σ_uu − Σ_uk Σ_kk⁻¹ Σ_ku )`.
  - Take μ from the stage table (Section 6).
  - Estimate Σ from ANSUR (adults) and from Snyder (per age band, 2 to 18 y), on log-ratios to H so it transfers across stature.
  - Use the conditional mean for the "typical" body and add a draw from the conditional covariance for a "random person".
- **Use log(dim/H) as variables.** This makes the H-independence explicit; it is the "height applied last as uniform scale" principle. It leaves weight or BMI z-score as the main second predictor for girths.
- **Licences.** ANSUR II (US government, public release) and the NL4/WHO/CDC LMS tables can be shipped in a non-commercial or commercial project. The Snyder 1977 data are US-government-funded (CPSC contract) and freely distributed by NIST; licence not stated, so verify before commercial use. HumanShape, SMIL, SMPL-X/AGORA and CAESAR are not usable commercially without a separate licence [inference from snippets].

### Gaps
- Did not read the Parkinson & Reed paper's equations, or UMTRI model files and accuracies (sites blocked).
- No open statistical body shape model covers 0 to 90 y with proportions validated by age. Anny covers the range but is calibrated only on height.
- No infant (0 to 2 y) individual data were reached for the covariance; only the MIMo mean fits.

---

## 4. Hands and feet by age (for the broken hand-scale knob)

### Takeaway
- **Babies do not have small hands relative to body length.** Hand length / H is about **0.124 at 1 month and 0.119 at 12 months** (verification: with the sex-pooled WHO length median at exactly 1 month, 54.2 cm, the 1-month values are 0.125 hand / 0.154 foot; 12-month values reproduce exactly), then **0.113 at 2 y**, 0.109 to 0.111 from 6 to 14 y and **0.110 in adults** (men 0.110, women 0.111 in ANSUR; 0.106 in Snyder's 18-year-old girls).
  - Infant hands are relatively **broad**: hand breadth / H is 0.069 at 1 month, 0.060 at 12 months, 0.056 at 2 y and 0.050 in adults. Hand breadth/length is 0.50 at 2 y and 0.46 in adults.
- **Feet** are 0.146 to 0.153 H in infancy, **0.155 to 0.159 H from 2 to 13 y** (relatively largest at 11 to 13 y, before the growth spurt), then 0.152 (men) / 0.145 to 0.151 (women) in adults.
  - Feet finish growing early: 99% of final foot length at about **13 y (girls) and 14 y (boys)**, about 2 years before stature.
- The hand-scale knob should therefore target a nearly **constant** hand/H (0.106 to 0.124) with a small infant excess. Big-hand and small-hand looks in characters are style offsets of about ±5% (1 SD ≈ 0.0045 H).

### Cited Findings
**Hands and feet / H [computed]**

Sources: MIMo log-fits of Snyder 1977 infant means divided by WHO length medians (sexes pooled) for 1 to 24 months; Snyder 1977 individual data for 2 to 18 y; ANSUR II for adults.

| age | hand L M | hand L F | hand B M | hand B F | foot L M | foot L F | foot B M | foot B F |
|---|---|---|---|---|---|---|---|---|
| 1 mo | 0.124 | (pooled) | 0.069 | | 0.153 | | 0.068 | |
| 3 mo | 0.120 | | 0.065 | | 0.146 | | 0.064 | |
| 6 mo | 0.120 | | 0.063 | | 0.146 | | 0.063 | |
| 12 mo | 0.119 | | 0.060 | | 0.149 | | 0.063 | |
| 24 mo (fit) | 0.114 | | 0.056 | | 0.152 | | 0.063 | |
| 2 y (Snyder) | 0.113 | 0.113 | 0.0565 | 0.0546 | 0.157 | 0.157 | 0.068 | 0.065 |
| 3 | 0.114 | 0.112 | 0.055 | 0.053 | 0.159 | 0.156 | 0.066 | 0.064 |
| 4 | 0.113 | 0.113 | 0.054 | 0.053 | 0.159 | 0.158 | 0.065 | 0.064 |
| 6 | 0.111 | 0.109 | 0.052 | 0.0505 | 0.157 | 0.155 | 0.063 | 0.061 |
| 8 | 0.109 | 0.109 | 0.051 | 0.050 | 0.156 | 0.156 | 0.062 | 0.060 |
| 10 | 0.109 | 0.109 | 0.051 | 0.0485 | 0.157 | 0.156 | 0.061 | 0.059 |
| 12 | 0.109 | 0.109 | 0.050 | 0.048 | 0.158 | 0.155 | 0.061 | 0.058 |
| 14 | 0.110 | 0.107 | 0.051 | 0.047 | 0.158 | 0.148 | 0.061 | 0.057 |
| 16 | 0.108 | 0.106 | 0.050 | 0.047 | 0.152 | 0.145 | 0.059 | 0.057 |
| 18 | 0.108 | 0.106 | 0.051 | 0.047 | 0.152 | 0.145 | 0.059 | 0.056 |
| adult (ANSUR) | 0.110 | 0.111 | 0.050 | 0.048 | 0.154 | 0.151 | 0.058 | 0.057 |

Sources: [MIMo params.json and update.py (AnthroKids IDs 566/261 hand length, 586/417 foot length)](https://github.com/trieschlab/MIMo/tree/main/mimoGrowth/data) [fetched code]; Snyder 1977 data [fetched]; ANSUR II [fetched]. The robust SD of hand/H is 0.0042 to 0.0046 at all ages 2 to 18 y; for foot/H it is 0.005 to 0.008 [computed]. ANSUR adult SD: hand/H 0.0042 to 0.0047, foot/H 0.0052 to 0.0053 [computed].

- MIMo's fitted model is `size(age_months) = a·ln(age + b) + c`, fitted on Snyder 1977 infant means at mean ages [1, 3, 7, 10, 13.5, 17.5, 21.5, 33] months [fetched code]. Values at 0 months are extrapolations and I do not use them.
- Beijing children: 99% of final foot length at 14 y (boys) and 13 y (girls), versus 99% of final height at 16 and 15 y. Peak foot growth at 11 y (boys) and 9 y (girls) — [Front Public Health 2024 1322333](https://frontiersin.org/journals/public-health/articles/10.3389/fpubh.2024.1322333/full) [snippet]. Dutch shoe-size peak velocity at 10.4 y (girls) and 11.5 y (boys), 1.3 to 2.5 y before the sitting-height peak — [Scoliosis 2011 6:1 (link.springer.com)](https://link.springer.com/article/10.1186/1748-7161-6-1) [snippet].
- Newborn foot length is used as a proxy for low birth weight in Ethiopia — [PMC5401464](https://pmc.ncbi.nlm.nih.gov/articles/PMC5401464/) [snippet; values not read].

### Inferences
- The broken hand-scale knob in Anny/MakeHuman is better replaced by a direct HeroBody target: **hand_length = r_hand(age, sex) · H**, with r_hand from the table and the hand mesh scaled uniformly about the wrist joint. Then apply a separate breadth/length ratio (0.50 → 0.46) to get "pudgy" baby hands [inference].
- Distal-first growth means feet and hands look relatively large in early puberty (11 to 13 y), before the trunk catches up. This is worth a small positive offset at the 12 to 14 y stage, which the table already contains [inference].

### Gaps
- Infant values are sex-pooled fits, not raw data. No newborn hand/foot measurement was read directly. No elderly hand/foot change data was found; I assume constant absolute size and arch flattening only.

---

## 5. Infant and toddler body-shape specifics

### Takeaway
- **Round, belly-first torso.** At birth, waist circumference ≥ hip circumference (NL4 WHR 1.11). The ratio is about 1.0 at 6 to 12 months and 0.96 at 2 y; it drops below 0.9 only after about 6 y. Waist/H is 0.63 at birth, 0.58 at 1 y and 0.53 at 2 y.
- **Big head, short legs, long trunk**: head circumference/H 0.69 at birth and 0.61 at 1 y; SH/H 0.69 at birth and 0.65 at 1 y; legs 0.31 to 0.35 H.
- **Fat**: BMI peaks at about 0.5 to 0.8 y (WHO median 17.3 in boys at 6 months) and is lowest at about 5 to 6 y ("adiposity rebound"). The chubbiest look is at 6 to 12 months and the leanest at 5 to 8 y.
- **Legs**: bow legs (genu varum) are greatest at about 6 months and neutral at about 18 months. Knock-knee (valgus) peaks at about 8° (normal up to 12°) at 3 to 4 y, then settles to about 5 to 6° adult valgus by 6 to 11 y.
- **Feet**: flexible flat foot in 54% at 3 y and 24% at 6 y. This is linked to a medial plantar fat pad.
- **Neck**: short and partly hidden. The chin-to-sternal-notch distance is only about 0.05 H at 2 to 4 y, versus about 0.055 to 0.065 H in adults, and in infants it is masked by fat folds (estimate).

### Cited Findings
- **NL4 infant girths [computed]**:
  - At 0, 0.25, 0.5 and 1 y (boys): waist 32.4, 39.4, 42.0, 44.3 cm; hip 31.0, 37.3, 41.4, 44.7 cm; head circumference 35.7, 40.9, 44.1, 47.2 cm; length 51.3, 61.2, 68.0, 76.5 cm.
  - WHR LMS medians 1.112 (0 y), 1.041 (0.25 y), 1.013 (0.5 y), 0.988 (1 y), 0.968 (2 y).
  - Source: [AGD nl4.*](https://github.com/cran/AGD/tree/master/data) [fetched data].
- **BMI medians [computed]**:
  - WHO: peak 17.34 at 0.52 y (boys) and 16.91 at 0.53 y (girls).
  - NL4: peak 17.44 / 16.92 at 0.77 y.
  - CDC 2000 minimum ("adiposity rebound"): 15.38 at 5.7 y (boys) and 15.15 at 5.2 y (girls). NL4 minimum 15.51 at 5.5 y / 15.37 at 5.0 y. WHO 2007 minimum 15.26 / 15.24 at 5.25 y.
  - Sources: [childsds who.ref, cdc.ref, who2007.ref; AGD nl4.bmi](https://github.com/cran/childsds/tree/master/data) [fetched data].
- **Newborn girths** (Yemen, n = 1,000 term newborns): crown-heel length 48.91 cm, head circumference 33.78, chest 32.09, mid-upper arm 10.09, abdomen 30.10, calf 10.94 cm. A neonatal reference gives head circumference 33 to 35 cm and chest 30 to 33 cm; head is usually about 2 cm larger than chest, and crown-rump ≈ head circumference (31 to 35 cm) — [IJ Pediatrics newborn anthropometry](https://www.ijpediatrics.com/index.php/ijcp/article/download/523/461/2008), [GLOWM "The Normal Neonate"](https://www.glowm.com/section-view/heading/The%20Normal%20Neonate:%20Assessment%20of%20Early%20Physical%20Findings/item/147) [snippet].
- **Chest/head crossover**: in Indian children, chest overtook head at 20 to 21 months in normally nourished children, and at 28 to 31 months in slum and rural children — [search result, Indian 1995 study](https://www.ijcmph.com/index.php/ijcmph/article/download/10593/6429/41908) [snippet]. In Snyder at 2 y, chest/H 0.553 > head circumference/H 0.542, so chest is already larger at 2 y [computed].
- **Knee angle**: Heath & Staheli 1993 (196 white children, 6 months to 11 y) found maximal bowleg at 6 months and approximately 0° at 18 months. The greatest mean knock-knee was 8° at 4 y, falling to < 6° at 11 y. Normal range 2 to 11 y: valgus up to 12°, intermalleolar distance up to 8 cm. Bowlegs after 2 y are abnormal — [PubMed 8459023](https://pubmed.ncbi.nlm.nih.gov/8459023/) [snippet]. Salenius & Vankka 1975 found pronounced varus before 1 y, a change to valgus between 18 months and 3 y, a maximum at about 3.5 y, then correction to about 5 to 6° valgus by 5 to 6 y, with no sex difference — [Acta Orthop / review table](https://actaorthop.org/actao/article/view/28707) [snippet]. A Korean radiographic study found maximum valgus of 7.8° at 4 y — [Korean study via search](https://synapse.koreamed.org/articles/1020721) [snippet].
- **Flat foot**: 835 children aged 3 to 6 y, 3D scanner. Flexible flat foot 44% overall: 54% at 3 y, 24% at 6 y; boys 52%, girls 36%; associated with overweight — [Pfeiffer et al. 2006, PubMed 16882817](https://pubmed.ncbi.nlm.nih.gov/16882817/) [snippet]. Boys have a thicker medial midfoot plantar fat pad (ultrasound, n = 88) — [search result](https://pmc.ncbi.nlm.nih.gov/articles/PMC12450481/) [snippet].
- **Potbelly and lumbar lordosis**: normal in toddlers. It is described as a weak abdominal wall plus relatively large abdominal organs plus lumbar lordosis, and usually disappears by school age — [UF Health "Potbellies and toddlers"](https://ufhealth.org/conditions-and-treatments/potbellies-and-toddlers), [Mount Sinai](https://www.mountsinai.org/health-library/special-topic/potbellies-and-toddlers) [snippet; consumer-level sources].
- **Snyder 2 to 4 y torso [computed]**:
  - Chest/waist circumference ratio 1.04 to 1.06 at 2 to 3 y (a barrel torso with little waist), versus 1.13 (men) / 1.10 (women) in ANSUR adults.
  - Chest breadth/H 0.18 at 2 y versus 0.165 in adults.
  - Bideltoid/biacromial 1.15 at 2 y versus 1.23 in adult men (less muscle on the shoulder).

### Inferences
- **"Round torso" blendshape target for 0 to 2 y** [inference from the computed tables]: WHR 1.11 → 0.97, chest/waist 1.0 to 1.05, waist/H 0.63 → 0.53. Waist depth/breadth near 1, meaning a circular cross-section, versus 0.72 in adults (ANSUR waist depth/breadth 134/186).
- **"Chibi short legs" in real data**: a real 1 y old has legs ≈ 0.35 H and a 2 y old ≈ 0.40 H, against 0.47 to 0.48 H in adults. HeroBody's planned blendshape should reach at least leg/H = 0.31 (newborn) to cover real babies before going into stylised territory [inference].
- **Knee angle curve for the rig rest pose** (degrees, + = valgus): **−15 (0 to 0.5 y), −8 (1 y), 0 (1.5 y), +6 (2.5 y), +8 to +10 (3 to 4 y), +7 (5 y), +6 (7 y), +5 to +6 (11 y to adult)**. The −15° infant value is my reading of "pronounced varus"; the original papers give means of about 10 to 17° (not confirmed here) [inference].
- **Fat-layer schedule** [inference from BMI medians]: relative subcutaneous fat thickness peaks at 6 to 9 months and is lowest at 5 to 6 y. Then it is sex-specific: girls gain fat from puberty (hips, thighs, breasts), boys gain lean mass. The adult rise continues to 50 to 60 y (NHANES waist), then limbs thin from about 70 y.

### Gaps
- No measured infant waist depth/breadth or neck length. No lumbar lordosis angles by age were fetched. The cheek fat pad (buccal) has no numeric data here. Salenius & Vankka's exact degree table was not read.

---

## 6. Target ratio table per stage and sex (for a knob solver working in ratios of height), plus the individual-variation method

### Takeaway
- The two tables below are recommended **population medians**. I smoothed and blended across sources and rounded to 3 decimals. Sources per column are given in the footnotes.
- H (cm) is listed only for reference; **height is applied last as a uniform scale**, so every other column is a ratio to H.
- Individual variation is added with z-scores and the SDs in the third table, correlated through the residual correlation matrix (Section 3). Weight/BMI is the second predictor for girths and breadths.

### Cited Findings (compiled table; every number traces to the sections above)

**Table 6a. Male (recommended medians, ratios to stature)**

| stage (y) | H cm | heads | HH/H | SH/H | leg/H | arm(acr-wrist)/H | span/H | hand/H | foot/H | biac/H | hip br./H | bi-iliac/H | chest/H | waist/H | hip circ/H | neck/H | head circ/H | thigh/H | calf/H | BMI |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | 49.9 | 4.2 | 0.240 | 0.692 | 0.308 | 0.32 | 0.96 | 0.124 | 0.153 | 0.23 | 0.23 | – | 0.656 | 0.631 | 0.604 | – | 0.695 | 0.35 | 0.25 | 13.4 |
| 1 | 75.7 | 4.8 | 0.210 | 0.648 | 0.352 | 0.32 | 0.97 | 0.119 | 0.149 | 0.23 | 0.22 | – | 0.600 | 0.579 | 0.584 | – | 0.616 | 0.31 | 0.24 | 16.8 |
| 2 | 86.5 | 5.3 | 0.189 | 0.604 | 0.396 | 0.322 | 0.97 | 0.113 | 0.157 | 0.229 | 0.191 | – | 0.553 | 0.528 | 0.546 | 0.261 | 0.555 | 0.309 | 0.223 | 16.6 |
| 3-4 (3.5) | 98.7 | 5.5 | 0.181 | 0.562 | 0.438 | 0.320 | 0.98 | 0.113 | 0.159 | 0.230 | 0.185 | – | 0.530 | 0.496 | 0.518 | 0.249 | 0.499 | 0.299 | 0.212 | 15.9 |
| 6 | 115.4 | 6.2 | 0.162 | 0.541 | 0.459 | 0.329 | 0.98 | 0.111 | 0.157 | 0.224 | 0.174 | 0.162 | 0.501 | 0.444 | 0.492 | 0.226 | 0.431 | 0.288 | 0.199 | 15.4 |
| 8 | 127.9 | 6.7 | 0.150 | 0.531 | 0.469 | 0.333 | 0.99 | 0.109 | 0.156 | 0.218 | 0.171 | 0.161 | 0.492 | 0.425 | 0.483 | 0.212 | 0.395 | 0.286 | 0.198 | 15.7 |
| 10 | 138.6 | 7.0 | 0.143 | 0.520 | 0.480 | 0.337 | 0.99 | 0.109 | 0.157 | 0.218 | 0.170 | 0.159 | 0.485 | 0.418 | 0.492 | 0.204 | 0.372 | 0.291 | 0.199 | 16.6 |
| 12 | 149.1 | 7.4 | 0.136 | 0.513 | 0.487 | 0.343 | 1.00 | 0.109 | 0.158 | 0.216 | 0.174 | 0.157 | 0.484 | 0.415 | 0.495 | 0.201 | 0.351 | 0.296 | 0.199 | 17.6 |
| 14 | 163.8 | 7.6 | 0.132 | 0.508 | 0.492 | 0.342 | 1.01 | 0.110 | 0.158 | 0.217 | 0.182 | 0.158 | 0.494 | 0.406 | 0.498 | 0.197 | 0.329 | 0.302 | 0.203 | 19.1 |
| 16 | 173.5 | 7.9 | 0.126 | 0.512 | 0.488 | 0.342 | 1.02 | 0.108 | 0.152 | 0.221 | 0.185 | 0.158 | 0.509 | 0.404 | 0.502 | 0.196 | 0.316 | 0.300 | 0.203 | 20.5 |
| 18 | 176.2 | 8.0 | 0.125 | 0.515 | 0.485 | 0.345 | 1.025 | 0.108 | 0.152 | 0.227 | 0.184 | 0.159 | 0.533 | 0.414 | 0.506 | 0.207 | 0.313 | 0.310 | 0.207 | 21.9 |
| 25 | 176.7 | 7.9 | 0.127 | 0.524 | 0.476 | 0.343 | 1.031 | 0.110 | 0.154 | 0.236 | 0.194 | 0.157 | 0.590 | 0.520 | 0.570 | 0.222 | 0.314 | 0.352 | 0.222 | 25.0* |
| 45 | 176.2 | 7.8 | 0.128 | 0.522 | 0.478 | 0.343 | 1.033 | 0.110 | 0.154 | 0.237 | 0.200 | 0.159 | 0.628 | 0.578 | 0.592 | 0.232 | 0.315 | 0.360 | 0.224 | 27.5* |
| 65 | 173.4 | 7.7 | 0.130 | 0.518 | 0.482 | 0.348 | 1.045 | 0.112 | 0.156 | 0.237 | 0.204 | 0.163 | 0.635 | 0.610 | 0.600 | 0.236 | 0.318 | 0.350 | 0.215 | 28.5 |
| 80+ | 170.3 | 7.6 | 0.132 | 0.512 | 0.488 | 0.351 | 1.060 | 0.113 | 0.158 | 0.238 | 0.208 | 0.166 | 0.640 | 0.610 | 0.605 | 0.240 | 0.322 | 0.340 | 0.205 | 27.1 |

**Table 6b. Female (recommended medians, ratios to stature)**

| stage (y) | H cm | heads | HH/H | SH/H | leg/H | arm(acr-wrist)/H | span/H | hand/H | foot/H | biac/H | hip br./H | bi-iliac/H | chest/H | waist/H | hip circ/H | neck/H | head circ/H | thigh/H | calf/H | BMI |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | 49.1 | 4.2 | 0.240 | 0.693 | 0.307 | 0.32 | 0.96 | 0.124 | 0.153 | 0.23 | 0.23 | – | 0.656 | 0.630 | 0.592 | – | 0.688 | 0.35 | 0.25 | 13.3 |
| 1 | 74.0 | 4.8 | 0.210 | 0.648 | 0.352 | 0.31 | 0.97 | 0.119 | 0.149 | 0.23 | 0.22 | – | 0.600 | 0.575 | 0.590 | – | 0.610 | 0.31 | 0.24 | 16.4 |
| 2 | 85.0 | 5.3 | 0.187 | 0.601 | 0.399 | 0.314 | 0.97 | 0.113 | 0.157 | 0.233 | 0.197 | – | 0.557 | 0.530 | 0.553 | 0.270 | 0.549 | 0.320 | 0.229 | 16.4 |
| 3-4 (3.5) | 97.4 | 5.6 | 0.179 | 0.558 | 0.442 | 0.318 | 0.98 | 0.113 | 0.157 | 0.229 | 0.186 | – | 0.524 | 0.497 | 0.532 | 0.245 | 0.494 | 0.313 | 0.214 | 15.7 |
| 6 | 114.7 | 6.3 | 0.159 | 0.541 | 0.459 | 0.316 | 0.98 | 0.109 | 0.155 | 0.224 | 0.176 | 0.161 | 0.495 | 0.442 | 0.502 | 0.223 | 0.429 | 0.299 | 0.200 | 15.2 |
| 8 | 127.6 | 6.8 | 0.148 | 0.531 | 0.469 | 0.319 | 0.99 | 0.109 | 0.156 | 0.218 | 0.176 | 0.162 | 0.489 | 0.423 | 0.498 | 0.212 | 0.393 | 0.307 | 0.201 | 15.8 |
| 10 | 138.0 | 7.0 | 0.142 | 0.522 | 0.478 | 0.322 | 0.99 | 0.109 | 0.156 | 0.217 | 0.172 | 0.161 | 0.483 | 0.411 | 0.503 | 0.198 | 0.368 | 0.308 | 0.200 | 16.8 |
| 12 | 151.2 | 7.5 | 0.133 | 0.516 | 0.484 | 0.327 | 1.00 | 0.109 | 0.155 | 0.217 | 0.181 | 0.162 | 0.483 | 0.402 | 0.509 | 0.194 | 0.347 | 0.306 | 0.198 | 18.0 |
| 14 | 160.4 | 7.8 | 0.128 | 0.518 | 0.482 | 0.330 | 1.00 | 0.107 | 0.148 | 0.217 | 0.190 | 0.169 | 0.493 | 0.398 | 0.526 | 0.187 | 0.332 | 0.326 | 0.204 | 19.3 |
| 16 | 162.5 | 7.9 | 0.127 | 0.524 | 0.476 | 0.327 | 1.005 | 0.106 | 0.146 | 0.219 | 0.197 | 0.172 | 0.505 | 0.403 | 0.537 | 0.190 | 0.327 | 0.331 | 0.208 | 20.4 |
| 18 | 163.1 | 8.0 | 0.125 | 0.524 | 0.476 | 0.325 | 1.01 | 0.107 | 0.147 | 0.221 | 0.199 | 0.171 | 0.505 | 0.410 | 0.548 | 0.191 | 0.325 | 0.332 | 0.208 | 21.3 |
| 25 | 163.1 | 7.8 | 0.128 | 0.527 | 0.473 | 0.339 | 1.018 | 0.109 | 0.148 | 0.225 | 0.215 | 0.168 | 0.570 | 0.520 | 0.600 | 0.201 | 0.324 | 0.377 | 0.228 | 24.0* |
| 45 | 161.8 | 7.75 | 0.129 | 0.525 | 0.475 | 0.339 | 1.024 | 0.109 | 0.149 | 0.224 | 0.222 | 0.175 | 0.615 | 0.570 | 0.640 | 0.210 | 0.326 | 0.386 | 0.230 | 27.0* |
| 65 | 159.8 | 7.6 | 0.132 | 0.520 | 0.480 | 0.345 | 1.044 | 0.111 | 0.152 | 0.226 | 0.228 | 0.180 | 0.635 | 0.630 | 0.650 | 0.215 | 0.332 | 0.380 | 0.220 | 29.0 |
| 80+ | 156.2 | 7.4 | 0.135 | 0.515 | 0.485 | 0.352 | 1.071 | 0.113 | 0.155 | 0.229 | 0.232 | 0.184 | 0.640 | 0.627 | 0.650 | 0.220 | 0.341 | 0.370 | 0.210 | 27.5 |

Footnotes (per column) [all computed from fetched data unless marked]:
- **H**: WHO 2006 (0 to 1 y), CDC 2000 (2 to 18 y), NHANES 2015-18 weighted medians (25 = 25-34 y, 45 = 45-54, 65 = 65-74, 80+ = 80+).
- **HH/H**: Snyder 1977 (2 to 18). Adults from the ANSUR construction blended with Snyder 18 y. 0 y = textbook 1/4 lowered to 0.24 by head-circumference scaling [snippet + inference]. 1 y = HC scaling [inference]. Elderly = adult HH ÷ Sorkin-reduced H [inference].
- **SH/H, leg/H**: NL4 (0 to 18), ANSUR (25, 45). 65 and 80+ [inference] from trunk-dominant height loss.
- **Arm (acromion to radiale-stylion)/H**: Snyder (2 to 18), ANSUR (adults). 0 to 1 y from MIMo segment fits scaled by the ANSUR segment-sum factor 0.927 [inference]. Elderly = adult length ÷ reduced H [inference].
- **Span/H**: ANSUR (adult). 0 to 3 y from the Chinese infant range 0.96 to 0.976 [snippet]. 4 to 16 y from H/span 1.03 → 0.98 (boys) / 1.00 (girls) and the Turkish crossover at about 12 y [snippet]. Elderly [inference].
- **Hand, foot**: MIMo/Snyder infant fits (0 to 1; sex-pooled), Snyder (2 to 18), ANSUR (adult). The adult female values are smoothed between Snyder 18 y (0.106 / 0.145) and ANSUR (0.111 / 0.151).
- **Biacromial**: Snyder (2 to 18) averaged with Berkeley (8 to 18), and ANSUR. 0 to 1 y [inference] from Snyder 2 y.
- **Hip breadth**: Snyder bitrochanteric (2 to 18), ANSUR "hip breadth" (adult; a slightly different landmark). 0 to 1 y from MIMo fits (0.24 at 1 month, 0.22 at 12 months).
- **Bi-iliac**: Berkeley (6 to 18), ANSUR bicristal (adult). Elderly trend +0.002 to 0.004 per decade [snippet + inference].
- **Chest**: Snyder (2 to 18), ANSUR (adult; 25 = mean of the 17-24 and 25-34 bands). 0 y = (HC − 1.7 cm)/H [snippet + inference]; 1 y ≈ HC [inference]. Elderly [inference].
- **Waist, hip circumference**: NL4 (0 to 18). At 25 y, the waist is set to 0.52 within the ANSUR 17-24 to 25-34 range (0.497 to 0.534 M; 0.505 to 0.526 F) to match the BMI-25 "healthy-average" default; the US NHANES 25-34 medians are higher, 0.546 M / 0.566 F. At 45 y, ANSUR 45-58 (0.576 / 0.569). At 65 and 80+, NHANES medians. Elderly hip circumference [inference].
- **Neck**: Snyder (2 to 18), ANSUR (adult). Elderly [inference].
- **Head circumference**: NL4 (0 to 21); adults = NL4 21 y ÷ reduced H.
- **Thigh, calf**: Snyder (2 to 18; upper thigh), MIMo (0 to 1; mid-thigh), ANSUR (adult). Elderly −3 to −8% [inference].
- **BMI**: WHO (0 to 1), CDC (2 to 18), NHANES (65, 80+). \*25 and 45 are set as "healthy-average" choices, below the US medians of 27.9 / 29.1 (M) and 27.1 / 28.5 (F).

**Table 6c. Individual variation: 1 SD of the ratio (robust SD = IQR/1.349), Snyder 1977 by age band (sexes pooled) and ANSUR adults [computed]**

| ratio | 2-4 y | 4-7 | 7-10 | 10-13 | 13-16 | 16-19 | adult M | adult F |
|---|---|---|---|---|---|---|---|---|
| HH/H | 0.0096 | 0.0095 | 0.0074 | 0.0075 | 0.0073 | 0.0062 | 0.0066 | 0.0063 |
| SH/H | 0.0162 | 0.0144 | 0.0140 | 0.0124 | 0.0155 | 0.0136 | 0.0135 | 0.0141 |
| biac/H | 0.0091 | 0.0078 | 0.0085 | 0.0089 | 0.0090 | 0.0099 | 0.0098 | 0.0096 |
| hip br./H | 0.0079 | 0.0083 | 0.0100 | 0.0092 | 0.0119 | 0.0119 | 0.0129 | 0.0153 |
| chest/H | 0.026 | 0.022 | 0.028 | 0.029 | 0.030 | 0.033 | 0.049 | 0.050 |
| waist/H | 0.031 | 0.029 | 0.033 | 0.035 | 0.032 | 0.033 | 0.063 | 0.060 |
| hip circ/H | 0.028 | 0.024 | 0.034 | 0.037 | 0.041 | 0.035 | 0.042 | 0.044 |
| hand/H | 0.0045 | 0.0045 | 0.0042 | 0.0043 | 0.0046 | 0.0042 | 0.0042 | 0.0047 |
| foot/H | 0.0057 | 0.0053 | 0.0051 | 0.0059 | 0.0080 | 0.0064 | 0.0052 | 0.0053 |
| neck/H | 0.011 | 0.012 | 0.0105 | 0.011 | 0.0106 | 0.012 | 0.016 | 0.012 |
| thigh/H | 0.023 | 0.021 | 0.030 | 0.030 | 0.031 | 0.028 | 0.033 | 0.033 |
| span/H | – | – | – | – | – | – | 0.027 | 0.029 |

The adult SDs are ordinary SDs of the ratio.

### Inferences: the method to fill in individual variation (code-ready)
1. **Real years.** Map stage plus in-stage slider to real years: `age = stage_start + s·(next_stage_start − stage_start)`. Interpolate every column of 6a/6b in age with monotone cubic (PCHIP). Do not interpolate linearly across 0 to 2 y; the curves are steep there.
2. **Population prior** for sex s and age a: `r̄_j(a, s)` from the table, `σ_j(a)` from 6c (interpolate between bands; use the 2-4 y SDs ×1.1 for 0 to 2 y [inference]).
3. **Character z-vector** `z` (one number per ratio, constant across life):
   - Draw it from N(0, R), where R is the residual correlation matrix (ANSUR adults; Snyder per band for children). Key entries: hand–foot 0.6, hip breadth–hip circumference 0.8, SH–hand/foot −0.3.
   - Or set it from the human-verified profile ("long legs" = SH z −1.5).
   - Keep z fixed when ageing a character. Growth-chart percentiles track over childhood, which is a standard assumption; I did not fetch a tracking coefficient [inference].
4. **Ratios** `r_j = r̄_j + z_j·σ_j`. For girths and breadths, also add the body-mass term. Using the regression slopes (Section 3), shift by `b_W,j·ΔW/H`, where ΔW = W_character − W_median(a, s, H). Equivalently, use a BMI z-score with a per-dimension loading. Lengths get essentially no weight term (bW ≈ 0.00 to 0.02).
5. **Blueprint override (2D-first).** For every ratio m_j measured on the approved blueprint (measure2d.py), combine it with the prior as a Gaussian product:
   `r_j* = (m_j/τ_j² + r_j/σ_j²) / (1/τ_j² + 1/σ_j²)`.
   - Set τ_j from the measurement error. Approval allows a 1% height tolerance, so τ ≈ 0.003 to 0.005 for lengths and is tiny relative to σ. The blueprint therefore wins wherever it is drawn.
   - Measurements not visible in the blueprint, for example circumferences from front and side views only, come from the conditional Gaussian (Section 3) given the measured ones.
6. **Hard checks** before the knob solver:
   - leg/H = 1 − SH/H;
   - chest ≥ waist except at 0 to 2 y and at high BMI;
   - WHR > 1 allowed only at age < 1 y or for an obese character;
   - arm and span consistency is not additive (see Section 1).
7. **Old age**: H_age = H_45 − ΔH(age, sex) (Sorkin: −3 cm M / −5 cm F at 70; −5 / −8 cm at 80). Remove 75% of ΔH from the trunk (SH) and 25% from the legs. Keep absolute head, hand, foot, span and biacromial values. Then recompute the ratios. This is what generates the 65 and 80+ rows.

### Gaps
- The 0 to 1 y biacromial, chest, neck, head height and span values are inferences or snippets. Adult girth defaults depend on the BMI choice; the US medians are about 10% higher than ANSUR young adults.
- The pooled-sex SD for 2 to 13 y is fine (sexes are similar), but from 13 y it should be sex-specific. I computed pooled values for brevity.

---

## How this plugs into HeroBody

- **New data file**: `D:\anime\herobody\hb\data\proportions_by_age.csv`. Long format: `sex, age_y, ratio_name, median, sd, source_tag`, holding Table 6a/6b/6c plus the raw source tables.
  - Generate it from the scratchpad scripts (`nl4.py`, `berk.py`, `ak2.py`, `ansur.py`, `nh1518.py`, `sd.py`, `regress.py`, `regkids.py`). Re-download the inputs from the GitHub mirrors listed in Sources.
  - Keep ANSUR II and NL4/WHO/CDC as redistributable. Mark the Snyder-derived numbers "licence unverified".
- **Knob solver (open build item "knob solver, height last")**:
  - Use this table as the **prior** term. Blueprint ratios from `hb/measure2d.py` are the likelihood term (Section 6, step 5).
  - The 30 measurement knobs map directly: head height/H, SH/H (torso), leg/H, crotch/H (≈ leg/H + 0.005 in adults), arm/H, span/H, hand/H, foot/H, biacromial/H, hip breadth/H, chest, waist, hip, neck, thigh and calf circumferences/H.
  - Solve in log-ratio space. Apply H last as a uniform scale, which is already the decision.
- **Anny age knob**: drive Anny's age parameter from real years using Anny's own WHO-height-calibrated anchors: years [0, 1, 4, 11, 16, 18, 64, 110] → anny_age [0, 0.05, 0.215, 0.415, 0.67, 0.77, 0.83, 1.0].
  - Anny's blend anchors are newborn −1/3, baby 0, child 1/3, young 2/3, old 1. The newborn shape is an empirically scaled baby. This explains the Phase-1 "age knob breaks at the baby end": below anny_age 0 the model extrapolates.
  - **Fix**: clamp anny_age ≥ 0. Let the HeroBody ratio targets (head 0.24 H, SH 0.69, WHR 1.11, legs 0.31 H) carry the newborn look through the new blendshapes.
- **The four new blendshapes (open build item)**, with real-data targets:
  - *Head bigger than 2×*: real range HH/H 0.24 (newborn) → 0.125 (adult). So "2×" relative to adult is already a real newborn. Anime beyond that is pure style offset.
  - *Chibi short legs*: real leg/H 0.31 (birth), 0.35 (1 y), 0.40 (2 y), 0.44 (3.5 y).
  - *Round torso*: WHR 1.11 → 0.96 (0 to 2 y); chest/waist 1.0 to 1.05; waist depth/breadth ≈ 1.
  - *Anime waist*: the adult real waist/H minimum is about 0.40 at 12 to 16 y (girls 0.398 at 14 y); anything lower is stylisation.
- **Hand scale knob (Phase 1: broken)**: replace it with a direct target hand_length = r_hand(age, sex)·H (0.124 → 0.11) and hand_breadth/hand_length = 0.50 (≤ 4 y) → 0.46 (adult). Do the same for the foot (0.15 to 0.159 H). Scale the hand bones uniformly about the wrist joint so the rig follows.
- **Fat layer (open build item)**: drive fat-pad thickness by the BMI-for-age median curve (peak at 0.5 to 0.8 y, minimum at 5 to 6 y), then the sex-specific pubertal split (bi-iliac and hip circumference rise in girls only). Then add an adult waist gain of +0.08 H of waist circumference between 20 and 65 y, and limb thinning after 70 y. Toddler plantar fat pad (flat foot 54% at 3 y → 24% at 6 y) and knee-angle rest pose are per-age rig corrections.
- **Rig and anatomy**: from 0 to 4 y, set the rest-pose knee varus/valgus from the Section 5 curve. Elderly: add thoracic kyphosis to absorb about 75% of stature loss. The BodyParts3D skeleton registered through the rig then gets a shorter trunk without changing bone lengths, which also reduces "anatomy poking out" on the elderly body. The per-body clamp stays.
- **Character profile (human-in-the-loop)**:
  - Store sex, birth date or age, and BMI or weight category.
  - Store the z-vector (per ratio, fixed for life) and the manga override images per stage.
  - The solver recomputes the body from these numbers for every stage. This fits "every character has its own body, calculated from scratch".

## Sources
- Snyder RG et al. 1977, Anthropometry of infants, children and youths to age 18 for product safety design (UM-HSRI-77-17), Deep Blue: https://deepblue.lib.umich.edu/items/9918421c-4112-4872-a654-b51292fd97cb
- AnthroKids 1977 data, GitHub copy used: https://github.com/solcalloni/prediccion-talle-zapatos (file chicos.csv); NIST AnthroKids (blocked): https://math.nist.gov/~SRessler/anthrokids/
- MIMo growth fits (Snyder 1977 infant means): https://github.com/trieschlab/MIMo/tree/main/mimoGrowth/data
- ANSUR II public CSVs (GitHub copy): https://github.com/keenon/nimblephysics/tree/master/python/nimblephysics/models/rajagopal_data
- ANSUR II release: https://www.army.mil/article/188601/for_good_measure_natick_releases_raw_data_from_army_wide_anthropometric_survey ; https://ph.health.mil/PHC%20Resource%20Library/ANSURIIDatabasesOverview.pdf ; https://www.openlab.psu.edu/ansur2/
- AGD R package (NL4 Dutch references, Fredriks et al. 2005): https://github.com/cran/AGD
- childsds R package (WHO, CDC, KiGGS, LIFE-Child, Kromeyer-Hauschild, UK1990, Valencia neck, Sharma waist): https://github.com/cran/childsds
- sitar R package, Berkeley Child Guidance Study data: https://github.com/cran/sitar/blob/master/man/berkeley.Rd
- mpower R package, NHANES 2015-2018 subset: https://github.com/cran/mpower
- Anny (NAVER): https://github.com/naver/anny ; phenotype.py https://github.com/naver/anny/blob/main/src/anny/models/phenotype.py ; full_model.py https://github.com/naver/anny/blob/main/src/anny/models/full_model.py
- Argentine SH/H reference: https://sap.org.ar/docs/publicaciones/archivosarg/2017/v115n3a05e.pdf
- Hawkes et al. 2020 US leg length / sitting height references: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7452688/
- Bogin & Varela-Silva 2010: https://dspace.lboro.ac.uk/2134/14468 ; https://carta.anthropogeny.org/node/13606
- Head-to-body proportions (textbook): https://socialsci.libretexts.org/Bookshelves/Early_Childhood_Education/Child_Growth_and_Development_(Paris_Ricardo_Rymond_and_Johnson)/04%3A_Physical_Development_in_Infancy_and_Toddlerhood/4.02%3A_Proportions_of_the_Body ; https://www.scientificamerican.com/article/human-body-ratios
- Farkas norms context: https://epublications.vu.lt/object/elaba:2133010/2133010.pdf
- Infant arm span (China): https://cjchc.xjtu.edu.cn/EN/10.11852/zgetbjzz2025-0696
- Turkish arm span/height: https://turkjpediatr.org/article/view/3
- Arm span 4-16 y: https://hrcak.srce.hr/en/5221
- Elderly arm span: https://doaj.org/article/ca5dcbe4047e4a53a38fade966229ed7 ; https://nsg.repo.nii.ac.jp/records/3620
- Sorkin et al. 1999 height loss: https://www.proquest.com/docview/224823953 ; review https://www.frontiersin.org/journals/endocrinology/articles/10.3389/fendo.2025.1542962/pdf
- NCHS 1960-62 survey sitting height: https://www.cdc.gov/nchs/data/series/sr_11/sr11_008.pdf
- Fryar et al. 2021 anthropometric reference data (not read): https://stacks.cdc.gov/view/cdc/100478
- Bi-iliac breadth and age: https://pmc.ncbi.nlm.nih.gov/articles/PMC5198355/table/T3 ; https://pubmed.ncbi.nlm.nih.gov/3228169
- UMTRI virtual child models: https://www.umtri.umich.edu/?p=3479 ; Human computational modeling: https://www.umtri.umich.edu/human-computational-modeling ; licence: https://available-inventions.umich.edu/product/online-body-shape-models/print
- UMTRI 3D child report: https://deepblue.lib.umich.edu/items/5821a113-13f3-4ab5-b883-e0c45ffcf541 ; https://deepblue.lib.umich.edu/handle/2027.42/174130?show=full
- Matt Reed / YouthShape.US: https://mreed.umtri.umich.edu/mreed/ ; https://www.citedrive.com/en/discovery/youthshapeus-a-new-anthropometric-resource-for-us-children-and-youth/
- B-K. Park profile: https://www.umtri.umich.edu/people/park-byoung-keon-daniel ; IBRC poster https://ibrc.osu.edu/wp-content/uploads/2024/05/2024-IBRC-Poster_Neeluru-et-al.pdf
- Parkinson & Reed 2010: https://www.openlab.psu.edu/2010/01/01/creating-virtual-user-populations-by-analysis-of-anthropometric-data/ ; Nadadur & Parkinson 2008: https://saemobilus.sae.org/content/2008-01-1858
- CAESAR/WEAR: https://bodysizeshape.com/
- SMIL: https://ps.is.tuebingen.mpg.de/code/skinned-multi-infant-linear-model-smil ; AGORA: https://agora.is.tue.mpg.de/ ; https://arxiv.org/pdf/2104.14643
- Foot growth timing: https://frontiersin.org/journals/public-health/articles/10.3389/fpubh.2024.1322333/full ; https://link.springer.com/article/10.1186/1748-7161-6-1
- Newborn anthropometry: https://www.ijpediatrics.com/index.php/ijcp/article/download/523/461/2008 ; https://www.glowm.com/section-view/heading/The%20Normal%20Neonate:%20Assessment%20of%20Early%20Physical%20Findings/item/147 ; https://pmc.ncbi.nlm.nih.gov/articles/PMC5401464/
- Chest/head crossover (India): https://www.ijcmph.com/index.php/ijcmph/article/download/10593/6429/41908
- Heath & Staheli 1993: https://pubmed.ncbi.nlm.nih.gov/8459023/ ; Salenius & Vankka context: https://actaorthop.org/actao/article/view/28707 ; Korean knee angle: https://synapse.koreamed.org/articles/1020721
- Pfeiffer et al. 2006 flat foot: https://pubmed.ncbi.nlm.nih.gov/16882817/ ; fat pad: https://pmc.ncbi.nlm.nih.gov/articles/PMC12450481/
- Toddler potbelly: https://ufhealth.org/conditions-and-treatments/potbellies-and-toddlers ; https://www.mountsinai.org/health-library/special-topic/potbellies-and-toddlers

## Verification

Adversarial re-check, 2026-10-09. Data were re-downloaded independently from the GitHub mirrors into `scratchpad/dl/verify_props/` and recomputed with fresh Python (rdata for .rda, a torch-free unpickler for Anny .pth). Non-GitHub hosts (pubmed, frontiersin, available-inventions.umich.edu, crossref, europepmc) were refused by the proxy, and the session's web-search budget was used up, so the snippet-level claims could not be re-checked against a primary source. Evidence level per row: [fetched] = primary data or code read and recomputed; [inference] = my reasoning; no row here relies on a snippet alone.

| claim | verdict | note | source |
|---|---|---|---|
| NL4 SH/H medians 0.692 (0), 0.648 (1), 0.604 (2), 0.541 (6), 0.520 (10), 0.508 boys (14), 0.515 M / 0.524 F (18) | confirmed | [fetched] nl4.shh LMS M column: M 0.6920, 0.6481, 0.6036, 0.5406, 0.5204, 0.5077, 0.5149; F 18 y 0.5244. Exact. | https://github.com/cran/AGD (data/nl4.shh.rda) |
| Snyder 1977 HH/H boys 0.189 / 0.162 / 0.143 / 0.132 / 0.125 at 2/6/10/14/18 y; girls almost identical | confirmed (with caveat) | [fetched] Reproduced exactly when age is rounded to the nearest year (a ± 0.5 y). With floor binning (completed years) boys are 0.188 / 0.158 / 0.139 / 0.129 / 0.127. Girls 0.186 / 0.159 / 0.142 / 0.128 / 0.125. Cell n is 7 to 55, not 14 to 61 (corrected inline). Head height measured only in measurement set 3 (n = 1,264). The "vertex-menton" landmark definition was not re-read from the 1977 report [inference: 18 y median 220 mm is consistent with v-gn]. | https://github.com/solcalloni/prediccion-talle-zapatos (chicos.csv) |
| ANSUR II ratios: span 1.031/1.018, biacromial 0.2365/0.2245, bicristal 0.157/0.167, hand 0.110/0.111, foot 0.154/0.151, SH 0.524/0.527, crotch 0.481/0.480 | confirmed | [fetched] Medians of individual ratios: 1.0311/1.0180, 0.2365/0.2245, 0.1570/0.1674, 0.1100/0.1109, 0.1543/0.1512, 0.5236/0.5270, 0.4811/0.4797. n = 4,082 M / 1,986 F, age 17-58. | https://github.com/keenon/nimblephysics/tree/master/python/nimblephysics/models/rajagopal_data |
| Berkeley biacromial/bi-iliac boys 1.35 (8 y) → 1.44 (18 y); girls 1.34 → 1.27 (15-18 y); female bi-iliac/H 0.161 → 0.171, male 0.157-0.159 | confirmed (with caveat) | [fetched] Boys 1.352 → 1.438; girls 1.342 (8), 1.272 (15), 1.268 (16), 1.285 (18) so "1.27-1.29 at 15-18 y". Female bi-iliac/H 0.160-0.161 → 0.171-0.172; male 0.157-0.159 from 10 y (0.162 at 8 y). Dataset is the Berkeley **Child Guidance** Study (Tuddenham & Snyder 1954), not the Berkeley Growth Study (name corrected inline). | https://github.com/cran/sitar (data/berkeley.rda, man/berkeley.Rd) |
| NL4 WHR median 1.112 (0), 1.013 boys (0.5), 0.988 (1), 0.968 (2), 0.905 (6), 0.82 M / 0.75 F at 16-18 y; waist/H 0.63 at birth | confirmed | [fetched] nl4.whr M: 1.1122, 1.0129, 0.9882, 0.9684, 0.9048, 0.8198-0.8239; F 0.7548-0.7505. Waist 32.37 cm / length 51.32 cm = 0.631. | https://github.com/cran/AGD |
| Infant hand/H 0.124 (1 mo), 0.119 (12 mo); foot 0.153, 0.149 (MIMo log-fits ÷ WHO length) | confirmed (minor) | [fetched] MIMo params.json, model a·ln(age+b)+c, fitted on AnthroKids IDs 566/261 and 586/417 at mean ages [1,3,7,10,13.5,17.5,21.5,33] mo. ÷ sex-pooled WHO length: 1 mo 0.125 hand / 0.154 foot; 12 mo 0.119 / 0.149. 1-month value ~1% higher than stated (note added inline). These are fits to group means, not raw data. | https://github.com/trieschlab/MIMo/tree/master/mimoGrowth/data |
| NHANES 2015-18 weighted median waist/H men 0.546/0.585/0.610/0.610, women 0.566/0.595/0.631/0.627 (25-34, 45-54, 65-74, 80+) | confirmed | [fetched] Exact, using WTMEC4YR weighted medians; heights 176.7/176.2/173.4/170.3 and 163.1/161.8/159.8/156.2 also reproduce. 19,225 rows = full 2015-18 demographic sample. | https://github.com/cran/mpower (data/nhanes1518.rda) |
| Anny anchors anny_age [0, .05, .215, .415, .67, .77, .83, 1.0] ↔ [0,1,4,11,16,18,64,110] y; blend anchors linspace(-1/3,1,5) newborn..old; newborn = empirically scaled baby | confirmed | [fetched] boys.pth and girls.pth `morphological_age_mapping` identical; phenotype.py linspace(-1/3, 1, len(age)); model_data.py age = [newborn, baby, child, young, old]; full_model.py scaling [0.922, 0.922, 0.75] "Empirical values"; shape_distribution.py "calibrated to match the height vs age distribution of WHO data". | https://github.com/naver/anny/tree/main/src/anny |
| BMI peak 0.52 y (WHO 17.3 boys) to 0.77 y (NL4); adiposity-rebound minimum 5.2-5.7 y (CDC 2000) | confirmed | [fetched] WHO peak 17.34 at 0.520 y (boys), 16.91 at 0.531 y (girls); NL4 peak 17.44 / 16.92 at 0.767 y; CDC minimum 15.38 at 5.71 y (boys), 15.15 at 5.21 y (girls). Note the WHO 0-5 y table alone puts the minimum at 5.08 y (boys) and 4.33 y (girls), earlier than CDC. | https://github.com/cran/childsds ; https://github.com/cran/AGD |
| ANSUR regressions: girths R² 0.71-0.88; men's waist = 92.76 − 0.396 H + 0.759 W + 0.198 age, RMSE 4.1; hand-foot residual r 0.58-0.61 | corrected | [fetched] Waist equation, R² 0.87 and RMSE 4.07 reproduce exactly; residual hand-foot r 0.585 M / 0.610 F. But the girth R² range is 0.61-0.88 (men 0.68-0.88, women 0.61-0.85), breadths only 0.36-0.79, and lengths 0.45-0.97 (hand/foot 0.45-0.55). Section 3 takeaway fixed inline. | https://github.com/keenon/nimblephysics/tree/master/python/nimblephysics/models/rajagopal_data |
| Snyder hand/H and foot/H 2-18 y (Table 4, used for hand-scale knob) | confirmed | [fetched] Spot check, nearest-year bins: hand boys 0.113/0.111/0.109/0.110/0.108, girls 0.112/0.109/0.109/0.107/0.106; foot boys 0.156/0.157/0.157/0.158/0.152, girls 0.157/0.155/0.156/0.148/0.145 at 2/6/10/14/18 y. Within 0.001 of the table. | https://github.com/solcalloni/prediccion-talle-zapatos |
| Knee angle: max bowleg 6 mo, ~0° at 18 mo, mean valgus peak 8° at 4 y, < 6° by 11 y, normal valgus up to 12° (Heath & Staheli 1993) | unverifiable | PubMed refused by proxy; no search budget left. Consistent with my prior knowledge of the abstract [inference], but not re-read. | https://pubmed.ncbi.nlm.nih.gov/8459023/ |
| Height loss 30→70 y ≈ 3 cm M / 5 cm F; by 80 y ≈ 5 / 8 cm (Sorkin 1999, BLSA) | unverifiable | Frontiers and ProQuest refused. Consistent with my prior knowledge of Sorkin et al. 1999 [inference], not re-read. This drives the 65 and 80+ rows. | https://www.frontiersin.org/journals/endocrinology/articles/10.3389/fendo.2025.1542962/pdf |
| Flexible flat foot 54% at 3 y, 24% at 6 y (n = 835) | unverifiable | PubMed refused. Consistent with my prior knowledge of Pfeiffer et al. 2006 (Pediatrics) [inference]. | https://pubmed.ncbi.nlm.nih.gov/16882817/ |
| Feet reach 99% of final length at 14 y (boys) / 13 y (girls) vs height at 16 / 15 y | unverifiable | frontiersin.org refused; not re-read. | https://frontiersin.org/journals/public-health/articles/10.3389/fpubh.2024.1322333/full |
| UMTRI HumanShape models free for non-commercial use only; commercial use / incorporation in software not permitted | unverifiable | available-inventions.umich.edu refused. Treat HumanShape as not usable until the licence page is read; the HeroBody plan does not depend on it (it rebuilds regressions from ANSUR/Snyder). | https://available-inventions.umich.edu/product/online-body-shape-models/print |
