# Skeleton, joints, posture and motion across the lifespan (bone lengths, X-ray maturation, spine posture, ROM, gait for the HeroBody rig)

Research date: 2026-10-09. Researcher notes for the report writer.

**Access notes.** The user asked me to try reaching sites again. I did, at the start of this session. Every non-GitHub host still failed. WebFetch returned `getaddrinfo ENOTFOUND` for pmc.ncbi.nlm.nih.gov, europepmc.org, en.wikipedia.org and archive.cdc.gov. curl through the proxy returned no connection (HTTP 000) for pmc.ncbi.nlm.nih.gov, www.ncbi.nlm.nih.gov, europepmc.org, cdc.gov, who.int, arxiv.org, export.arxiv.org, openaccess.thecvf.com, semanticscholar, openalex, crossref, zenodo, physionet, huggingface.co, mdpi.com, springer, plos, frontiersin, biomedcentral, nature.com, wiley, web.archive.org, doi.org, nagoya.repo.nii.ac.jp and r.jina.ai. `gh search code` is also refused because this session is bound to its own repositories. **What worked:** github.com, raw.githubusercontent.com and `git clone` of any public GitHub repository, plus the GitHub MCP code search. So the primary data below come from **GitHub copies of original data**, analysed with Python in the scratchpad:

- **Maresh (1970) Denver femur diaphyseal lengths** by age and sex, 0.125 to 18 y. Copied from Scheuer & Black's reprint into `AlisJay/BioArchaeology_In_R/Health Index/data/M70.txt`, including the authors' derived −3 SD columns. **[fetched]**
- **Stull (South Africa) subadult long-bone data** `salb_za`: 1,310 children aged 0.09 to 12.99 y, Lodox Statscan whole-body X-rays. Diaphyseal lengths of humerus, radius, ulna, femur, tibia and fibula. From the R package `geanes/kidstats` (GPL-3). **[fetched; computed here]**
- **KidStats: Stature data** (`ElaineYChu/KS-Stature`, MIT): 990 subadults (589 M, 401 F) with **stature and diaphyseal long-bone lengths**, stature 34 to 193 cm. No age column. **[fetched; computed here]**
- **WBDS (Fukuchi et al. 2018) young vs older gait summary** from `BMClab/h2a_age_spt/results/*.csv` (MIT). This is the dataset authors' own re-analysis. **[fetched]**
- **Van Criekinge et al. 2023 (Sci Data)** paper and supplementary tables, as PDF copies in `DaebangStn/gait_dataset`: 138 able-bodied adults aged 21 to 86 with height and leg length. **[fetched; computed here]**
- A GitHub research note (`mfbailey91/Function_Generators_in_Open_Chains`) that copies Soucie 2011 adult 20 to 44 y ROM values. It is a **secondary copy**, tagged **[fetched-secondary]**.

Everything else is from search-result snippets, tagged **[snippet]**. Tags: **[fetched]** = I read the primary file or data. **[computed]** = I computed it from fetched data. **[fetched-secondary]** = I read a third-party copy of the primary numbers. **[snippet]** = search snippet only. **[inference]** = my reasoning or recall, and it needs checking. "H" = stature. "DL" = diaphyseal length (bone shaft without the unfused epiphyses). Ages are in years (y).

---

## 1. Long-bone lengths vs stature and age

### Takeaway
- Limb bones grow faster than the trunk. **Femur/stature rises from about 0.15 in newborns to 0.18 at 1 y, 0.20 at 2 y, 0.225 at 5 y, 0.245 at 8 y, 0.255 at 12 y and about 0.265 to 0.27 in adults.** Tibia/stature follows the same path: 0.13 → 0.15 → 0.185 → 0.22. Humerus/stature goes from 0.13 to 0.185. Radius/stature goes from 0.105 to 0.14.
- **Intra-limb ratios are almost flat after age 2.** Tibia/femur (crural index) is about 0.85 in the first months, then 0.82 to 0.83 from 2 y to adult. Radius/humerus (brachial index) falls from about 0.82 at birth to 0.76 by 5 y, then stays at 0.75 to 0.76. Humerus/femur falls from 0.82 at birth to 0.69 by 10 y. So **legs outgrow arms**, and the distal arm segment (forearm) loses ground in the first 5 years.
- For code, one quadratic per bone in stature fits the 990 KidStats children and adolescents with r ≥ 0.99. **Femur_cm = −4.40 + 0.2045·H + 0.000497·H²**, with H in cm (other bones below). Use it as the prior. Then apply sex and individual offsets.
- For adults, Trotter-Gleser regressions predict **stature from bone**, for example white male S = 2.38·Fem + 61.41 ± 3.27 cm. **Do not invert them to get bone from stature.** The inverse exaggerates deviations from the mean. Use the ratio method instead (Feldesman 1990: femur = 26.74% of stature), or regress bone on stature.
- Anderson-Green-Messner gives the per-plate split: the **distal femoral plate makes about 70% of femoral growth (about 10 mm/yr)** and the **proximal tibial plate about 55 to 60% of tibial growth (about 6 mm/yr)**. Lower-limb growth stops at skeletal age 15 to 17 in boys and 13 to 15 in girls.

### Cited Findings

**Maresh 1970 femur diaphyseal length (mm), Denver longitudinal sample, as reprinted by Scheuer & Black** [fetched, GitHub copy of the reprint](https://raw.githubusercontent.com/AlisJay/BioArchaeology_In_R/master/Health%20Index/data/M70.txt). The −3 SD columns were estimated by the repository author from the 10th and 90th percentiles.

| age y | boys mean | girls mean | boys −3SD | girls −3SD | femur/H boys* | femur/H girls* |
|---|---|---|---|---|---|---|
| 0.125 | 86.0 | 87.2 | 70.4 | 74.9 | — | — |
| 0.25 | 100.7 | 100.8 | 85.1 | 88.9 | — | — |
| 0.5 | 112.2 | 111.1 | 96.2 | 97.8 | 0.166 | 0.169 |
| 1 | 136.6 | 134.6 | 120.7 | 121.0 | 0.180 | 0.182 |
| 1.5 | 155.4 | 153.9 | 135.9 | 132.4 | | |
| 2 | 172.4 | 170.8 | 152.7 | 148.7 | 0.198 | 0.199 |
| 3 | 200.3 | 198.4 | 176.1 | 170.4 | | |
| 4 | 224.1 | 223.2 | 196.7 | 189.6 | | |
| 5 | 247.5 | 247.0 | 215.4 | 214.0 | 0.225 | 0.225 |
| 6 | 269.7 | 268.9 | 232.6 | 227.1 | | |
| 7 | 291.1 | 288.8 | 252.0 | 247.8 | | |
| 8 | 312.1 | 309.8 | 268.2 | 261.2 | 0.245 | 0.245 |
| 9 | 330.4 | 328.7 | 287.8 | 274.1 | | |
| 10 | 349.3 | 347.9 | 301.3 | 290.1 | 0.253 | 0.251 |
| 11 | 367.0 | 367.0 | 319.1 | 292.8 | | |
| 12 | 386.1 | 387.6 | 332.6 | 320.9 | 0.259 | 0.256 |
| 18 | 511.7 | 462.9 | 438.5 | 378.0 | — | — |

\*femur/H divides by the WHO median height at that age, taken from the sibling note `growth_stature_trajectories.md` [computed]. The 18 y row is not a diaphysis-only value. Maresh switched to total length including epiphyses in adolescence, and his series is radiographic, so it includes some magnification. 511.7 mm is about 0.29 of a 176.5 cm man, which is too long for a living-body ratio. **Use the 18 y row only as a ceiling.** [inference]

**South African subadults (Stull, salb_za, n = 1,310), median diaphyseal lengths (mm) and ratios by age bin** [computed from fetched data](https://github.com/geanes/kidstats)

| age bin (median age) | n | HDL | RDL | UDL | FDL | TDL | FBDL | T/F | R/H | U/H | H/F | (H+R)/(F+T) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 0-0.25 (0.17) | 21 | 68.5 | 55.3 | 63.5 | 83.0 | 72.4 | 66.2 | 0.856 | 0.819 | 0.930 | 0.825 | 0.788 |
| 0.25-0.75 (0.56) | 30 | 87.6 | 70.7 | 79.7 | 108.6 | 91.7 | 85.6 | 0.843 | 0.809 | 0.915 | 0.804 | 0.784 |
| 0.75-1.5 (1.18) | 48 | 111.6 | 88.2 | 99.8 | 143.0 | 117.7 | 117.7 | 0.826 | 0.789 | 0.895 | 0.767 | 0.755 |
| 1.5-2.5 (2.0) | 88 | 129.9 | 101.6 | 114.5 | 172.4 | 143.0 | 140.8 | 0.820 | 0.785 | 0.877 | 0.751 | 0.734 |
| 2.5-3.5 (3.0) | 141 | 141.7 | 110.7 | 124.3 | 194.9 | 159.1 | 159.8 | 0.818 | 0.779 | 0.868 | 0.729 | 0.714 |
| 3.5-4.5 (4.0) | 119 | 155.8 | 121.9 | 134.6 | 216.9 | 179.1 | 178.7 | 0.827 | 0.777 | 0.864 | 0.720 | 0.705 |
| 4.5-5.5 (5.0) | 106 | 172.7 | 132.7 | 147.1 | 242.3 | 197.5 | 196.3 | 0.810 | 0.765 | 0.849 | 0.714 | 0.697 |
| 5.5-6.5 (6.1) | 103 | 186.9 | 141.1 | 157.1 | 262.5 | 212.2 | 211.8 | 0.815 | 0.767 | 0.849 | 0.703 | 0.685 |
| 6.5-7.5 (7.0) | 104 | 199.2 | 151.7 | 166.5 | 280.3 | 231.0 | 231.3 | 0.820 | 0.762 | 0.836 | 0.704 | 0.683 |
| 7.5-8.5 (8.0) | 120 | 209.3 | 159.6 | 175.2 | 300.0 | 246.5 | 244.1 | 0.822 | 0.761 | 0.833 | 0.704 | 0.677 |
| 8.5-9.5 (9.0) | 106 | 220.9 | 169.0 | 185.0 | 319.4 | 261.9 | 259.7 | 0.827 | 0.761 | 0.828 | 0.703 | 0.676 |
| 9.5-10.5 (10.0) | 98 | 230.4 | 175.9 | 190.8 | 332.8 | 275.8 | 273.1 | 0.825 | 0.765 | 0.832 | 0.694 | 0.670 |
| 10.5-11.5 (11.0) | 108 | 237.6 | 183.9 | 196.4 | 348.9 | 288.8 | 286.9 | 0.828 | 0.762 | 0.832 | 0.691 | 0.670 |
| 11.5-12.5 (12.0) | 93 | 246.7 | 189.8 | 201.9 | 358.0 | 297.3 | 295.0 | 0.832 | 0.765 | 0.830 | 0.692 | 0.670 |

- The South African femur DL at 0.17 y (83 mm) matches Maresh at 0.125 y (86 mm). At 10 to 12 y it runs 4 to 7% below Maresh (Denver). The documentation notes this sample comes from a lower-SES population. [computed]
- By sex, SA boys and girls differ by less than about 3% at every age to 12 y. Girls are slightly longer at 10 to 12 y because their growth spurt comes earlier. Examples: FDL at 11 y is 344.8 (M) vs 356.0 (F) mm. [computed]

**KidStats: Stature data (n = 990), median bone DL/stature by stature bin** [computed from fetched data](https://github.com/ElaineYChu/KS-Stature)

| stature cm | n | FDL/H | TDL/H | HDL/H | RDL/H | UDL/H | (F+T)/H | T/F | R/H |
|---|---|---|---|---|---|---|---|---|---|
| 30-50 | 31 | 0.148 | 0.130 | 0.130 | 0.106 | 0.119 | 0.278 | 0.862 | 0.810 |
| 50-60 | 110 | 0.155 | 0.133 | 0.131 | 0.104 | 0.119 | 0.288 | 0.853 | 0.799 |
| 60-70 | 85 | 0.165 | 0.138 | 0.134 | 0.104 | 0.117 | 0.302 | 0.843 | 0.781 |
| 70-80 | 67 | 0.181 | 0.150 | 0.144 | 0.109 | 0.123 | 0.331 | 0.825 | 0.771 |
| 80-90 | 66 | 0.192 | 0.159 | 0.146 | 0.112 | 0.126 | 0.350 | 0.827 | 0.764 |
| 90-100 | 48 | 0.203 | 0.166 | 0.152 | 0.114 | 0.127 | 0.369 | 0.827 | 0.754 |
| 100-110 | 49 | 0.215 | 0.179 | 0.154 | 0.118 | 0.132 | 0.394 | 0.825 | 0.761 |
| 110-120 | 41 | 0.225 | 0.185 | 0.161 | 0.120 | 0.133 | 0.409 | 0.818 | 0.755 |
| 120-130 | 24 | 0.229 | 0.189 | 0.164 | 0.122 | 0.135 | 0.418 | 0.820 | 0.751 |
| 130-140 | 25 | 0.232 | 0.197 | 0.170 | 0.125 | 0.139 | 0.431 | 0.838 | 0.765 |
| 140-150 | 39 | 0.244 | 0.204 | 0.178 | 0.131 | 0.146 | 0.447 | 0.836 | 0.740 |
| 150-160 | 64 | 0.259 | 0.216 | 0.185 | 0.138 | 0.149 | 0.477 | 0.831 | 0.750 |
| 160-170 | 140 | 0.265 | 0.220 | 0.185 | 0.142 | 0.151 | 0.486 | 0.829 | 0.759 |
| 170-180 | 142 | 0.267 | 0.223 | 0.188 | 0.143 | 0.154 | 0.491 | 0.832 | 0.758 |
| 180-200 | 59 | 0.264 | 0.222 | 0.185 | 0.140 | 0.152 | 0.487 | 0.838 | 0.758 |

**Regressions fitted to KidStats (sexes pooled; H in cm; bone in cm)** [computed] (verification note: the fits use only rows with that bone present, not all 990: femur n = 843, tibia 855, humerus 783, radius 798. Coefficients, r and SEE for the femur reproduce exactly.)

| bone | stature from bone (linear) | r | SEE cm | bone from stature, quadratic (use this for rigs) |
|---|---|---|---|---|
| femur | H = 32.29 + 3.087·F | 0.994 | 5.22 | F = −4.3986 + 0.204522·H + 0.000497·H² |
| tibia | H = 32.21 + 3.718·T | 0.993 | 5.58 | T = −2.7240 + 0.151839·H + 0.000488·H² |
| humerus | H = 25.42 + 4.602·Hu | 0.993 | 5.50 | Hu = −0.9859 + 0.127202·H + 0.000376·H² |
| radius | H = 24.44 + 6.111·R | 0.991 | 6.13 | R = 0.4019 + 0.076564·H + 0.000364·H² |

Check: at H = 50 cm the femur quadratic gives 7.1 cm (0.142 H). At 176.5 cm it gives 47.2 cm (0.267 H), which equals Feldesman's 26.74%. [computed]

**Adult regressions (stature from bone, cm)**
- Trotter (1970) white male: S = 2.38·Fem + 61.41 (±3.27); S = 2.52·Tib + 78.62 (±3.37); S = 3.08·Hum + 70.45 (±4.05) [snippet](https://intarch.ac.uk/journal/issue11/4/28.html).
- Trotter & Gleser 1952 white female: S = 2.47·Fem + 54.10 (±3.72); S = 2.90·Tib + 61.53 (±3.66) [snippet](https://pressbooks.gvsu.edu/introhumanosteology/?p=298).
- Trotter & Gleser 1958 male multi-bone (white): S = 67.049 + 0.913·F + 0.600·T + 1.225·H − 0.187·R [snippet](https://en.wikipedia.org/wiki/Estimation_of_stature). The 1958 tibia lengths were later found to be mismeasured (1995 re-analysis) [snippet](https://en.wikipedia.org/wiki/Estimation_of_stature).
- Modern white women have relatively longer tibiae than the Terry Collection women, so revised female formulas exist (1992, J Forensic Sci) [snippet](https://ojp.gov/index%2Ephp/node/790896).
- Feldesman et al. 1990 ratio: femur = 26.74% of stature, sexes pooled. The 1992 juvenile femur/stature paper has age-specific ratios (AJPA 87:447-459) [snippet](https://biblio.naturalsciences.be/associated_publications/anthropologica-prehistorica/anthropologie-et-prehistoire/ap-105/ap-105_29-32.pdf). On children from poor environments, the ratio method underestimated stature by 2.9 to 19.3 cm [snippet](https://www.ojp.gov/ncjrs/virtual-library/abstracts/test-three-methods-estimating-stature-immature-skeletal-remains).

**Anderson-Green-Messner and growth plates**
- 1963 "Growth and predictions of growth in the lower extremities" (JBJS) and 1964 "Distribution of lengths of the normal femur and tibia in children from one to eighteen years of age" (JBJS 46-A, doi 10.2106/00004623-196446060-00004). The data are longitudinal annual radiographs of 67 boys and 67 girls, 1 to 18 y [snippet](https://core.tdar.org/document/125576/growth-and-predictions-of-growth-in-the-lower-extremities).
- Distal femoral physis: about 10 mm/yr, 70% of femoral growth. Proximal tibial physis: about 6 mm/yr, 60% of tibial growth. Lower-limb growth stops at skeletal age 15 to 17 in boys and 13 to 15 in girls [snippet](https://www.wheelessonline.com/bones/methods-to-estimate-growth-potenital/).
- A 1997 Dutch study of 182 children found mean femur and tibia lengths had increased compared with Anderson's data (secular trend) [snippet](https://actaorthop.org/actao/article/download/20784/24616/68727).

### Inferences
- **Diaphysis vs rig bone.** Rig bones run joint-centre to joint-centre. They include the cartilage epiphyses that do not show on X-ray. In infants the epiphyses are large cartilage caps. So a joint-centre femur is longer than the DL. I estimate about +12 to 15% at birth, falling to +5 to 8% by 10 y and to about 0 in adults, where max length is about the joint-centre length. **Use the DL tables for the X-ray look and ratios. Use the stature regressions × a cartilage factor for the rig.** I have no measured cartilage factor; it is a gap. [inference]
- **Inverse regression.** If S = a + b·B with correlation r, then the best predictor of B from S has slope r²/b, not 1/b. For adult femur r ≈ 0.8 to 0.9, so inverting Trotter-Gleser overstates bone-length deviations by 1/r² ≈ 1.2 to 1.6 times. For a 200 cm character the error is about +1.5 to 3 cm in femur. Use ratio × H or the quadratics above. [inference]
- The quadratic is fitted on pooled sexes and cross-sectional data. Adult females have T/F about 0.83 and slightly shorter forearms relative to height. Add a sex term of no more than ±1.5% per bone. [inference]
- From 70 to 90 y, bone lengths stay fixed while stature falls 3 to 9 cm (see `growth_stature_trajectories.md`). In the Van Criekinge adults, leg length/height rises from 0.527 to 0.535 at 20 to 59 y to 0.536 to 0.559 at 70 to 89 y. [computed from fetched supplementary table; n = 7 to 14 per cell, cross-sectional] So **old-age bone ratios = adult bone lengths ÷ shrunken H**. Never shrink the limb bones.

### Gaps
- No primary table for tibia, humerus or radius from Maresh, or for Anderson-Green-Messner ages 13 to 18. The SA data stop at 13 y, and KidStats has no age column.
- No measured cartilage-to-joint-centre correction factor by age.
- Feldesman 1992 age-specific femur/stature ratios were not retrieved.

---

## 2. Skeletal maturation for X-ray depictions

### Takeaway
- A newborn has about **270 to 300 separate bony pieces**; an adult has **206**. Many "bones" at birth are several ossification centres joined by cartilage, which shows on X-ray as empty gaps. Pieces fuse through the mid-20s.
- **What shows at birth on an X-ray:** long-bone shafts. The distal femoral and proximal tibial epiphyses (knee) are present. The calcaneus and talus are present (they ossify in the 6th to 8th fetal month). The cuboid appears at or soon after birth. **No carpal bones** show at birth. The femoral head appears at 4 to 6 months.
- **Hand bone age:** capitate (1 to 3 months) → hamate (2 to 4 months) → triquetrum (2 to 3 y) → lunate (2 to 4 y) → scaphoid, trapezium, trapezoid (4 to 6 y) → pisiform (8 to 12 y). Roughly one new carpal per year from 1 to 7 y.
- **Elbow (CRITOE):** capitellum at 1 y, radial head at 4 (F) / 5 (M) y, medial epicondyle at 5 / 7 y, trochlea at 8 / 9 y, olecranon at 8 / 10 y, lateral epicondyle at 11 / 12 y (F / M). The elbow fuses first, at **11 to 15 y**, then shoulder and wrist, then the knee and hip around 14 to 18 y.
- **Last to fuse:** iliac crest (17 to 24 y) and **medial clavicle (25 to 27 y, sometimes 30+)**. Females run **1 to 2 y ahead** of males at every site.
- **Fontanelles:** posterior closes at about 2 to 3 months. Anterior closes with a median of about **14 to 16 months**: 16% closed at 10 months, 53% at 16 months, 88% at 20 months. Range 9 to 24+ months.

### Cited Findings

**Bone count**
- Newborn about 270 to 300 bones; adult 206; separate pieces merge through the mid-20s [snippet](https://healthline.com/health/how-many-bones-does-a-baby-have), [snippet](https://case.edu/news/anatomys-darin-croft-explains-why-adults-have-nearly-100-fewer-bones-when-they-are-born).

**Appearance of ossification centres (age at first visibility on X-ray)**

| site | age of appearance | source / tag |
|---|---|---|
| distal femur epiphysis | 23 to 40 weeks gestation; present in 94.5% at 32 weeks; always visible after 38 weeks | [snippet](https://epos.myesr.org/poster/esr/ecr2019/C-3356/Findings%20and%20procedure%20details) |
| proximal tibia epiphysis | at birth | [snippet](https://wheelessonline.com/?p=16846) |
| calcaneus, talus | 6th to 8th fetal month | [snippet](https://wheelessonline.com/?p=16846) |
| cuboid | at or soon after birth | [snippet](https://wheelessonline.com/?p=16846) |
| femoral head | 4 to 6 months | [snippet](https://radiopaedia.org/articles/24706) |
| greater trochanter | 2 to 4 y | [snippet](https://radiopaedia.org/articles/24706) |
| lesser trochanter | puberty | [snippet](https://radiopaedia.org/articles/24706) |
| capitate, hamate | 1 to 3 months, 2 to 4 months (may be within the first 3 months on ultrasound) | [snippet](https://prod-images-static.radiopaedia.org/page_images/2363/R23_299_Pediatric_Bone_Maturation_Of_the_Upper_Extremity.pdf) |
| triquetrum | 2 to 3 y (girls start of 2nd y; boys after 2.5 y) | [snippet](https://flexikon.doccheck.com/de/Corpus:Carpal_bones) |
| lunate | 2 to 4 y | [snippet](https://prod-images-static.radiopaedia.org/page_images/2363/R23_299_Pediatric_Bone_Maturation_Of_the_Upper_Extremity.pdf) |
| scaphoid, trapezium, trapezoid | 4 to 6 y | same |
| pisiform | 8 to 12 y | same |
| capitellum | 1 y (both sexes) | [snippet](https://pmc.ncbi.nlm.nih.gov/articles/PMC5337779) |
| radial head | 4 y F, 5 y M | same |
| medial epicondyle | 5 y F, 7 y M | same |
| trochlea | 8 y F, 9 y M | same |
| olecranon | 8 y F, 10 y M | same |
| lateral epicondyle | 11 y F, 12 y M | same |
| iliac crest apophysis (Risser 1) | about 13.8 y girls, about 15.2 y boys | [snippet](https://www0.sun.ac.za/ortho/webct-ortho/age/risser.html) |

- Brazilian radiograph series, elbow appearance ranges: capitellum 0 to 1 y, radial head 2 to 6, medial epicondyle 2 to 8, trochlea 5 to 11, olecranon 6 to 11, lateral epicondyle 8 to 13 y. Fusion ranges: 10 to 15, 12 to 16, 13 to 17, 10 to 18, 13 to 16, 12 to 16 y [snippet](https://www.redalyc.org/pdf/657/65752469009.pdf).

**Fusion (epiphysis to shaft; "complete union")**

| site | age complete (F / M where given) | source / tag |
|---|---|---|
| elbow epiphyses | first to fuse, about 11 to 15 y | Cardoso 2008 [snippet](https://baes.uc.pt/handle/10316/8067) |
| capitellum to humerus | by about 14 y | [snippet](https://pmc.ncbi.nlm.nih.gov/articles/PMC5337779) |
| femur (head, trochanters, distal) | 14 to 18 y; greater trochanter 14 to 16 y | [snippet](https://radiopaedia.org/articles/24706) |
| distal femur | starts 13 to 14 y; complete 16 to 17 y F, 17 to 18 y M | Indian radiographs [snippet](https://nicpd.ac.in/ojs-/index.php/njirm/article/view/831) |
| distal radius | complete 17 to 18 y | Bangladesh radiographs [snippet](https://banglajol.info/index.php/BJA/article/view/75532/49929) |
| lateral clavicle | unfused→fusing at 16.5 F / 17.5 M; fused at 21 F / 20 M | US 2015 [snippet](https://www.springermedicine.com/the-lateral-clavicular-epiphysis-fusion-timing-and-age-estimatio/21018050) |
| spheno-occipital synchondrosis | complete 17 M / 19 F (Turkish CT); all fused over 20 y (S. Africa) | [snippet](https://acikerisim.erbakan.edu.tr/items/3aa906b1-467b-40e3-8810-cc791a38012a) |
| iliac crest | 18 to 24 F, 17 to 24 M (Bass); mean 19.9 F / 20.7 M (Lahore); Risser 5 first at about 16 F / 18 M | [snippet](https://www.aafs.org/sites/default/files/media/documents/AAFS-2016-A84.pdf), [snippet](https://www0.sun.ac.za/ortho/webct-ortho/age/risser.html) |
| medial clavicle | last, about 25 to 27 y; not before 22 to 23 y in some series; regularly only over 30 in S. Africa | [snippet](https://baes.uc.pt/handle/10316/8067), [snippet](https://he01.tci-thaijo.org/index.php/CMMJ-MedCMJ/article/view/247344) |

- Females lead by 1 to 2 y in the lower limb and about 2 y in the upper limb (Lisbon collection: 57 F, 49 M aged 9 to 25; 65 F, 56 M aged 9 to 29) [snippet](https://estudogeral.uc.pt/handle/10316/8067).
- A Bosnian sample was at least 2 years ahead of an American sample at all elements, so population matters [snippet](https://www.academia.edu/6356665/Comparison_of_Ages_of_Epiphyseal_Union_in_North_American_and_Bosnian_Skeletal_Material).

**Fontanelles**
- Pindrik 2014 (CT, full-term): anterior fontanelle closed in 3 to 5% at 5 to 6 months, 16% at 10 months, 53% at 16 months, 88% at 20 months [snippet](https://pubmed.ncbi.nlm.nih.gov/24920348).
- Portugal, 684 children: P50 = 14 months; mean 14.3 ± 4.9 months [snippet](https://revistas.rcaap.pt/bgmj/article/view/29291).
- Posterior fontanelle: closed at about 2 to 3 months (6 to 8 weeks in some texts) [snippet](https://taylorandfrancis.com/knowledge/Medicine_and_healthcare/Anatomy/Posterior_fontanelle).

### Inferences
- **How a child's X-ray differs (render rules):**
  1. Joints look widely spaced, because unossified epiphyses and cartilage are radiolucent. In a 1-year-old the knee gap is wide, with small round epiphyseal nuclei. A newborn hand shows metacarpal and phalanx shafts but no carpals.
  2. Every long bone has a thin dark transverse line, the growth plate (physis), between metaphysis and epiphysis until it fuses.
  3. Ossification nuclei start as small round dots and grow toward the adult epiphysis shape.
  4. The pelvis is three bones (ilium, ischium, pubis) meeting at a Y-shaped triradiate cartilage in the acetabulum (fuses about 14 to 16 y [inference from standard texts]).
  5. The skull has open sutures and fontanelles, a large cranium relative to the face, and no erupted permanent teeth (permanent tooth crowns sit in the jaw).
  6. Vertebrae show three ossification centres (body plus two neural arches) in infants. [inference]
- For HeroBody the BodyParts3D adult skeleton (244 bones) cannot show this. You need a per-bone "maturity state" flag: absent / nucleus (scale s) / separate epiphysis with gap / fused. Drive it from the tables above, indexed by skeletal age = chronological age − 1 y for girls (or + 0.5 y for boys) relative to the pooled midpoint. [inference]

### Gaps
- No primary Greulich-Pyle or TW3 table was fetched. The bone-age atlases are copyrighted. The RSNA Pediatric Bone Age dataset is the open data route, but I did not check its license here.
- The appearance age of the humeral head, patella and vertebral ring apophysis was not retrieved. From standard texts: humeral head 0 to 6 months, patella 3 to 6 y. [inference, needs check]
- No count of ossification centres at birth. The "about 800 centres" figure is unverified.

---

## 3. Spine and posture by age

### Takeaway
- **Infant:** the spine is one C-shaped kyphosis. **Cervical lordosis** appears with head control at about **3 to 4 months**. **Lumbar lordosis** appears with sitting, then walking, at about **12 to 18 months**. Thoracic kyphosis on CT is **24 ± 5.6° in infants vs 36.7 ± 6.5° in adults**, and most of the change happens in the first 5 years.
- **Child 3 to 10 y:** TK 42.0 ± 10.6°, LL 53.8 ± 12.0°, PI 43.7 ± 9.0°, PT 5.5 ± 7.6°, SS 38.2 ± 7.7° (646 children). Through growth, **PI rises from 40° to 46° and PT from 4° to 9°**, while SS stays constant (1,059 children).
- **Adult (Japanese 20s to 70s, 626 volunteers):** CL 4.1 ± 11.7°, **TK 36.0 ± 10.1°, LL 49.7 ± 11.2°, PI 53.7 ± 10.9°, PT 14.5 ± 8.4°, SS 39.4 ± 8.0°, SVA 3.1 ± 12.6 mm**. With age, CL, PT and SVA rise and LL and SS fall. SVA jumps between the 60s and 70s.
- **Elderly (community, over 60, mean 74 y):** **SVA 51 mm, PT 19°, PI−LL 8.9°**. Kyphosis rises faster in women: median **27° at 20 to 29 y → 40° at 50 to 95 y for women, 27° → 35° for men** (attributed to Fon 1980, unverified). Neck: C2-C7 SVA 25 → 35 mm (men) from the 50s to the 80s. T1 slope 32° → 36° (men) and 28° → 37° (women). Standing craniovertebral angle in the elderly is **48 ± 6.5°**. Below 50 to 51° counts as forward head posture.
- **Knee flexion in stance increases with age** (r ≈ 0.43 in both sexes, 317 adults aged 20 to 84). This is the compensation for loss of lumbar lordosis, together with pelvic retroversion (PT up).
- **Rib cage:** infant ribs are near-horizontal, with the diaphragm and clavicle heads higher. The adult pattern is mostly reached **by about 2 y**. Through adolescence the ribs rotate further downward (inferiorly) relative to the spine. In old age they rotate back up as kyphosis grows.
- **Pelvis:** the pubis becomes sexually dimorphic at **about 13 y** (differences visible from about 9 y). Adult **subpubic angle 82.6 ± 7.7° in women (range 64 to 100°) vs 65.9 ± 7.2° in men (48 to 81°)**. In pubertal girls the inlet widens and the subpubic angle opens.

### Cited Findings

**Infant curves and rib cage**
- C-shaped spine at birth. Cervical curve with head holding (about 3 to 4 months). Lumbar curve with sitting and walking (about 12 to 18 months) [snippet](https://open-exam-prep.com/study-guides/pa-cat/anatomy-back-abdomen/vertebral-column-curvatures), [snippet](https://experts.arizona.edu/en/publications/development-and-functional-anatomy-of-the-spine/), [snippet](https://healthlibrary.childrenshospital.org/node/146320).
- Thoracic CT from birth to 29 y: thoracic kyphosis 24 ± 5.6° (infants) to 36.7 ± 6.5° (adults), mostly in the first 5 y [snippet](https://cris.biu.ac.il/en/publications/changes-in-thoracic-cage-morphology-from-birth-into-adulthood/).
- Openshaw 1984 (38 radiographs, 1 month to 31 y; 28 CT, 3 months to 18 y): infant ribs more horizontal; sternoclavicular heads and diaphragm domes higher; near-adult pattern by 2 y [snippet](https://thorax.bmj.com/content/39/8/624).
- Weaver et al. 2014 (339 CTs, 0 to 100 y): birth to adolescence, ribs rotate inferiorly relative to the spine and TK decreases. Adulthood to old age, TK increases and ribs rotate superiorly [snippet](https://onlinelibrary.wiley.com/doi/10.1111/joa.12203).

**Children**
- Mac-Thiong review, 3 to 10 y: TK 42.0 ± 10.6°, LL 53.8 ± 12.0°, PI 43.7 ± 9.0°, PT 5.5 ± 7.6°, SS 38.2 ± 7.7°; TK and LL similar between sexes [snippet](https://pmc.ncbi.nlm.nih.gov/articles/PMC3175924).
- Mac-Thiong 2005, 341 subjects 3 to 18 y (mean 12.1): PI 49.1 ± 11.0°, PT 7.7 ± 8.0°, SS 41.4 ± 8.2°, TK 44.0 ± 10.9°, LL 48.0 ± 11.7° [snippet](https://pmc.ncbi.nlm.nih.gov/articles/PMC2200687).
- 1,059 children: PI 40° → 46°, PT 4° → 9°, SS constant [snippet](https://pmc.ncbi.nlm.nih.gov/articles/PMC3175924).
- Pediatric kyphosis rises linearly from 25° at 7 y to 38° at 19 y [snippet](https://pmc.ncbi.nlm.nih.gov/articles/PMC4045366/).

**Adults by decade**
- Fon, Pitt & Thies 1980: 316 subjects (159 M, 157 F), 2 to 77 y, linear fits by sex; kyphosis increases with age, faster in women [snippet](https://pubmed.ncbi.nlm.nih.gov/6768276/). Medians attributed to Fon: F 27° (20 to 29 y) → 40° (50 to 95 y); M 27° → 35° [snippet, attribution uncertain](https://publish.kne-publishing.com/index.php/JMR/article/download/16417/15321/).
- Systematic review (30 studies, about 6,793 adults): kyphosis rises with age; the 40° "normal" cut-off misclassifies many healthy people; differences between age bands are smaller than the SEM [snippet](https://josr-online.biomedcentral.com/articles/10.1186/s13018-021-02592-2).
- Korean 885 subjects: TK 17 to 33° (M) and 17 to 34° (F), rising with age, not sex [snippet](https://synapse.koreamed.org/articles/1122721).
- Yukawa et al. (626 Japanese, at least 50 per sex per decade, 20s to 70s): means as in the Takeaway. Age raises CL, PT and SVA and lowers LL and SS. LL and TK drop and PT rises "from the 7th to 8th decade". Large SVA increase between the 60s and 70s. Sex differences in CL, TK, LL, PI, PT and SVA [snippet](https://link.springer.com/article/10.1007/s00586-016-4807-7).
- Brazil 130 volunteers (mean 48 y): LL 56.8°, PT 12.4°, PI 49.4°, SVA −0.54 cm. Those 60 and over have higher SVA and TPA [snippet](https://www.scielo.br/j/clin/a/fpk7ncdJt7JMxvph8Qbc35h/abstract/?lang=en).
- Japanese elderly over 60 (mean 74 y): SVA 51 mm, PI−LL 8.9°, PT 19° [snippet](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4340200/).
- Oe 2015, volunteers over 50 (by decade, 50s to 80s): T1 slope M 32/31/33/36°, F 28/29/32/37°; C2-C7 SVA M 25/28/34/35 mm, F 20/21/22/28 mm [snippet](https://pubmed.ncbi.nlm.nih.gov/26208229/).
- Hasegawa et al. (EOS + force plate, 136 subjects, 20 to 69 y): TK apex T7, 5.0 cm behind the gravity line; lordosis starts at L2; ear canal over the gravity line (0.0 cm) [snippet](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5382592/). Multicentre 317 adults (20 to 84 y): knee flexion and SVA increase with age (KF r = .427 M, .429 F); KF correlates with global, not local, alignment [snippet](https://nagoya.repo.nii.ac.jp/records/2006442).
- China 584 adults (20 to 89 y): knee and ankle angles differ by sex from the 20s to the 60s, not after 60. Older people recruit pelvis and lower-limb compensation [snippet](https://www.hkmj.org/../system/files/hkmj2510sp7p41.pdf).
- Craniovertebral angle, 70 elderly: standing 48.1 ± 6.5° (FHP group 43.7 ± 6.5°, normal 56.9 ± 4.2°); sitting 52 ± 8.3°; FHP cut-off < 51° (or < 50°) [snippet](https://jmr.tums.ac.ir/index.php/jmr/article/view/160).

**Pelvis**
- Adult subpubic angle F 82.6 ± 7.7° (64 to 100°), M 65.9 ± 7.2° (48 to 81°); no correlation with age; a 74° cut-off gives 88% sensitivity and 95% specificity for female [snippet](https://serval.unil.ch/notice/serval:BIB_85340B215DEB).
- CT, 188 children aged 1 to 18 y: pubic shape dimorphic at 13 y, visible from 9 y [snippet](https://www.mdpi.com/2079-7737/14/6/628).
- Pubertal girls aged 10 to 17: inlet widening and subpubic angle expansion [snippet](https://pcs.khmnu.edu.ua/index.php/pcs/en/article/view/511).

**Discs**
- Lumbar disc height changes are not a simple decline in cross-sectional MRI. Height peaks in middle age (6th decade M, 5th F in one study), and anterior L3-S1 height falls with age [snippet](https://smj.org.sa/content/22/11/1013.full.pdf). Stature loss with age is in the sibling note `growth_stature_trajectories.md`.

### Inferences
**Rest-pose posture targets per stage (degrees, standing).** Values are means from the sources above. Cells marked † are my interpolations. Use the stage slider to interpolate linearly between rows.

| stage / age y | CL (C2-C7 lordosis) | TK (T1/T4-T12 Cobb) | LL (T12/L1-S1) | PT | SS | SVA mm | knee flexion | head forward (CVA) |
|---|---|---|---|---|---|---|---|---|
| newborn 0 (lying) | ~0† (C-curve) | 24 | ~0 (no lordosis)† | — | — | — | hips/knees flexed 30 to 60† | — |
| baby 0.5 to 1 (sitting) | ~10† | 25† | ~10 to 20† | — | — | — | — | — |
| toddler 2 | ~10† | 30† | 40† | 4 | 36† | — | 5 to 10† (knee flex at contact) | — |
| child 3 to 10 | ~10† | 42 | 54 | 5.5 | 38 | ~0† | 0 to 5† (knee hyperext. 2 to 5) | — |
| teen 11 to 18 | ~8† | 44 | 48 to 50 | 8 | 41 | ~0† | 0 | — |
| adult 20 to 39 | 4 | 33 to 36 | 50 to 53 | 12 to 14 | 40 | 0 ± 13 | 0 to 2† | ~55 |
| adult 40 to 59 | 6† | 36 to 40 | 48 | 15 | 38 | 10† | 2 to 4† | ~52† |
| old 60 to 69 | 8† | 40 (F) / 37 (M)† | 45† | 17† | 36† | 25 to 30† | 4 to 6† | ~50 |
| old 70 to 79 | 10† | 42 (F) / 38 (M)† | 40 to 42† | 19 | 34† | 51 | 6 to 8† | 48 |
| old 80+ | 12† | 45† | 35† | 22† | 31† | 70 to 90† | 8 to 12† | 44† (FHP) |

- The knee-flexion and SVA values for 80+ are my extrapolation. The sources give only the trend (KF r ≈ 0.43 with age, SVA jump after 60). [inference]
- **Geometry rule.** Keep PI fixed per individual (PI = PT + SS). In old age, loss of LL is compensated first by PT (pelvis tilts back), then by knee flexion, then by ankle dorsiflexion. Order: LL loss → PT up → KF up. Keep the ear canal over the ankles (gravity line) by solving SVA with the remaining hip and knee angles. [inference from the Hasegawa / Yukawa descriptions]
- **Infant hips.** A newborn's resting pose is a flexed "frog" pose. Infants have physiological hip and knee flexion contractures of about 20 to 30° that resolve within months. This is from standard pediatric texts and was not fetched. [inference]
- **Rib angle.** No degree values were retrieved. As a placeholder, set the rib descent angle in the sagittal plane to about 0 to 10° below horizontal at 0 y, about 25 to 35° by 2 y, about 35 to 40° in adults, and about 30° in old age, with ribs rising back up as TK increases. These numbers are placeholders. [inference]
- **Sex divergence of the pelvis** begins about 9 to 13 y. Blend the female pelvis morph by Tanner stage, not by age. [inference]

### Gaps
- Per-decade tables (Yukawa, Fon, the kyphosis systematic review Table 4) were not readable. All per-decade mean ± SD values except those quoted are missing.
- No numeric knee-flexion-by-decade table, and no numeric rib-angle-by-age table.
- No infant lumbar lordosis angle values (0 to 2 y).

---

## 4. Joint range of motion by age and sex

### Takeaway
- **Children 2 to 8 y are more mobile than adults:** knee flexion 152.6° (F) / 147.8° (M) vs 141.9 / 137.7 at 20 to 44 y (+7%). Shoulder flexion 178.6 / 177.8 vs 172.0 / 168.8 (+4%). **Knee hyperextension 5.4° (F) / 1.6° (M) at 2 to 8 y.** Hip flexion 140.8° (F) / 131.1° (M) at 2 to 8 y.
- **Females are more mobile at every age**, most clearly in ankle plantarflexion and forearm pronation/supination: adult pronation 82.0 vs 76.9°, supination 90.6 vs 85.0°, elbow flexion 150.0 vs 144.6°.
- **Ageing:** average ROM falls with age for all joints (Soucie). Most hip and knee arcs lose only **3 to 5° from 25 to 39 y to 60 to 74 y**, but **hip extension loses more than 20%** (Roach & Miles, NHANES I). Ankle dorsiflexion: 13.8 → 11.6° (F) and 12.7 → 11.9° (M) from 20 to 44 y to 45 to 69 y. Hip extension at 45 to 69 y: 16.7° (F) / 13.5° (M). Shoulder: external rotation about −10° and flexion about −15° from under 30 y to over 60 y. Neck ROM falls in all six directions with age, clearly after 50.
- **Hypermobility** (Beighton) is common in children and declines with age. Using the stricter childhood cut-off of 6 or more, prevalence is about 6% in boys and 13% in girls.

### Cited Findings
- Soucie et al. 2011: 674 healthy subjects aged 2 to 69 (53.6% F), passive goniometry by 9 therapists; groups 2-8, 9-19, 20-44, 45-69 y. Females greater in nearly all joints. ROM falls with age in both sexes [snippet](https://pubmed.ncbi.nlm.nih.gov/21070485/).
- Soucie 2 to 8 y (from a trial SAP quoting the CDC table): hip flexion F 140.8 (139.2 to 142.4), M 131.1 (129.4 to 132.8). Knee extension F 5.4 (3.9 to 6.9), M 1.6 (0.9 to 2.3). Ankle dorsiflexion F 24.8 (22.5 to 27.1), M 22.8 (21.3 to 24.3). Knee flexion F 152.6, M 147.8. Shoulder flexion F 178.6, M 177.8 [snippet](https://cdn.clinicaltrials.gov/large-docs/85/NCT03442985/SAP_001.pdf).
- Soucie 20 to 44 y: knee flexion F 141.9 / M 137.7; shoulder flexion 172.0 / 168.8; elbow flexion 150.0 / 144.6; pronation 82.0 / 76.9; supination 90.6 / 85.0 [fetched-secondary](https://raw.githubusercontent.com/mfbailey91/Function_Generators_in_Open_Chains/main/docs/research/literature/BIOLOGICAL_JOINT_RANGE_REFERENCE_TRACE.md).
- Soucie 45 to 69 y: hip extension F 16.7, M 13.5 [snippet](https://archive.cdc.gov/www_cdc_gov/ncbddd/jointrom/index_1715172647.html). Ankle dorsiflexion F 13.8 → 11.6, M 12.7 → 11.9 (20-44 → 45-69) [snippet](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9517286/). Knee flexion at 9 to 19 y: mean 142° [snippet](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12298373/).
- Public-use CDC dataset (674 subjects) exists at CDC Stacks [snippet](https://stacks.cdc.gov/view/cdc/153156/cdc_153156_DS1.pdf).
- Roach & Miles 1991 (NHANES I, 1,313 subjects aged 25 to 74): young vs old mean differences 3 to 5°, except hip extension down more than 20% [snippet](https://musculoskeletalkey.com/measurement-of-range-of-motion-and-muscle-length-clinical-relevance/).
- Zwerus 2019 (352 adults): active elbow flexion 146°, extension −2°, pronation 80°, supination 87°; passive values 3 to 5° larger [fetched-secondary](https://raw.githubusercontent.com/mfbailey91/Function_Generators_in_Open_Chains/main/docs/research/literature/BIOLOGICAL_JOINT_RANGE_REFERENCE_TRACE.md).
- Functional knee needs (elderly): gait under 90°, stairs and chairs 90 to 120°, bath about 135° (Rowe 2000) [fetched-secondary](https://raw.githubusercontent.com/mfbailey91/Function_Generators_in_Open_Chains/main/docs/research/literature/BIOLOGICAL_JOINT_RANGE_REFERENCE_TRACE.md).
- Shoulder, 6,635 subjects (markerless): ER about −10° (under 30 y → over 60 y), flexion about −15° (under 20 y → over 60 y). Over-60 cohort: flexion and abduction fall across 60-69, 70-79 and 80+, rotations do not [snippet](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9937824/).
- Neck: Youdas 1992 (337 subjects, 11 to 97 y) shows all six cervical AROMs decreasing with age; females greater except flexion. Swinkels: drop after 50 y. Typical adult: flex/ext about 60°, rotation 90°, lateral bend 45°. About 60% of rotation is at C1/2 [snippet](https://experts.umn.edu/en/publications/normal-range-of-motion-of-the-cervical-spine-an-initial-goniometr/), [snippet](https://eorthopod.com/news/is-my-neck-getting-more-stiff-every-year-an-analysis-of-cervical-spine-range-of-motion-changes-over-forty-years/).
- Beighton: 750 Korean females, GJH 58.9% in girls vs 36.5% in women. Nigeria: girls 34%, boys 20%, declining with age. Review (≥ 6 cut-off): 6% boys, 13% girls [snippet](https://search.bvsalud.org/gim/resource/en/wpr-65230), [snippet](https://oars.uos.ac.uk/5075/).

### Inferences
**Per-age joint-limit multipliers for the rig** (multiply the adult 20 to 44 y limit for that sex). Built from the numbers above; † = interpolated or extrapolated.

| joint motion | adult F / M limit (deg) | 0 to 1 y | 2 to 8 y | 9 to 19 y | 45 to 69 y | 70 to 79 y | 80+ y |
|---|---|---|---|---|---|---|---|
| knee flexion | 142 / 138 | 1.10† | 1.07 | 1.00 | 0.98† | 0.96† | 0.93† |
| knee hyperextension | 2 / 1† | 5 to 10° abs.† | 5.4 / 1.6° abs. | 2 / 1† | 0 | 0 | −5 to −10° (flexion contracture)† |
| hip flexion | 125 / 120† | 1.15† | 1.12 (140.8 / 131.1) | 1.03† | 0.97† | 0.95† | 0.92† |
| hip extension | 20 / 17† | 0.0 (flex contracture)† | 1.1† | 1.0 | 0.82 (16.7 / 13.5) | 0.78† | 0.70† |
| shoulder flexion | 172 / 169 | 1.05† | 1.04 | 1.01† | 0.95† | 0.92† | 0.90† |
| shoulder ext. rotation | 90 / 85† | 1.10† | 1.05† | 1.0 | 0.93† | 0.89† | 0.87† |
| elbow flexion | 150 / 145 | 1.03† | 1.02† | 1.0 | 0.99† | 0.98† | 0.97† |
| forearm pron + sup | 173 / 162 | 1.05† | 1.04† | 1.0 | 0.97† | 0.95† | 0.93† |
| ankle dorsiflexion | 14 / 13 | 1.8 to 2.5† | 1.8 (24.8 / 22.8) | 1.2† | 0.84 / 0.94 | 0.80† | 0.75† |
| neck rotation (each side) | 80† | 1.1† | 1.08† | 1.0 | 0.90† | 0.80† | 0.70† |
| neck flex/ext total | 120† | 1.1† | 1.05† | 1.0 | 0.90† | 0.80† | 0.70† |

- Treat these as **hard rig limits = passive ROM**. Gait and everyday animation should use about 70 to 90% of them. [inference]
- Disability and mobility settings in the character profile should override these multipliers per joint, for example a contracture of X° or a fused joint. [inference]

### Gaps
- Only the 2 to 8 y and 20 to 44 y columns of Soucie are partly filled. The 9 to 19 and 45 to 69 y full tables, wrist, ankle plantarflexion and hip ROM for adults are missing. They are in the CDC public-use dataset, which was blocked.
- No infant (0 to 2 y) ROM norms were fetched. No over-70 hip and knee norms by sex were fetched (Roach & Miles stops at 74).

---

## 5. Gait and movement by age for animation

### Takeaway
- **Toddler (first independent steps at about 11 to 14.5 months, delayed if not walking by 16 months):** arms in **high guard** (abducted, externally rotated, elbows flexed, hands near shoulder height). **Wide base** that narrows by about 22 months (11 months after walking onset). Asymmetric foot rotation, high foot lift in swing, flat-foot contact. **Long double support** (novices 42.5% of stance vs improvers 33.9%). Single-limb stance is about **32% of the cycle at 1 y vs 38% (mature) by 4 y**. Reciprocal arm swing first appears at about **1.5 y** and is consistent by **3.5 y**.
- **Child:** the mature pattern is mostly there by **3 y** (sagittal joint angles adult-like from 2 y). Time-distance values keep changing to **7 to 8 y**. Step length / leg length rises until about 4 y, then stays constant. At 6 to 12 y, cadence falls from **132.6 to 114.7 steps/min** and speed rises from **0.99 to 1.20 m/s**.
- **Adult comfortable speed (meta-analysis, 23,111 subjects):** 1.34 to 1.43 m/s from 20 to 59 y, then **1.34 / 1.24 (M / F) at 60 to 69, 1.26 / 1.13 at 70 to 79, 0.97 / 0.94 at 80 to 99**.
- **Older adults at the same speed** take **shorter steps and a higher cadence**. WBDS, 64 vs 28 y, both at about 1.22 m/s: step 0.606 vs 0.637 m (−4.9%), cadence 121.0 vs 116.5 steps/min (+3.9%). They also show **less peak hip extension** (about −30% in one study), less ankle push-off power, wider steps and longer double support. **Cadence at 70 to 85+ y** (Mayo): 102 to 106 steps/min (men), 108 to 114 (women).
- **Body-size scaling (Hof):** v' = v/√(g·l), s' = s/l, f' = f·√(l/g). For mature walkers these are nearly constant: **v' ≈ 0.42, s' ≈ 0.72, f' ≈ 0.58 steps per (√(l/g))** (WBDS young and older). So one dimensionless gait spec plus leg length l gives speed, step and cadence for any character, including anime proportions.
- **Datasets:** adults across the lifespan are well covered under CC BY. **Children are thin.** Lencioni 2019 (6 to 72 y), HumanAttr (5 to 88 y, for motion generation) and a few clinical sets exist. No public SMPL-format child mocap set was found. Toddler mocap is effectively missing.

### Cited Findings

**Toddler and child**
- High guard; wide base narrowing to shoulder width; high foot lift; asymmetric foot rotation [snippet](https://embryology.med.unsw.edu.au/embryology/index.php/Neural_Exam_-_12_month_Motor_9), [snippet](https://wiki.ubc.ca/Course:KIN366/ConceptLibrary/Gait). Arm swing first at 1.5 y, systematic by 3.5 y; walking onset 11 to 14.5 months (delayed after 16 months) [snippet](https://research.vu.nl/ws/files/1725705/134915.pdf).
- Wide base narrows 11 months after walking onset or by 22 months; less experienced walkers have wider steps [snippet](https://pubmed.ncbi.nlm.nih.gov/23271309/).
- Home video, 10 novice and 10 improver infants: double support 42.5% vs 33.9% of stance; novices have lower cadence and more falls [snippet](https://pmc.ncbi.nlm.nih.gov/articles/PMC6586442/).
- Sutherland 1980 (186 children, 1 to 7 y): cadence falls, speed and step length rise; mature by 3 y; sagittal rotations adult-like from 2 y [snippet](https://orthobullets.com/evidence/7364807). Single-limb stance 32% at 1 y → 38% at 4 y; step factor constant after 4 y [snippet](https://www.ouhsc.edu/bserdac/dthompso/web/gait/matgait/matgait.htm).
- Fels study: gait velocity mature by 7 to 8 y; immature walkers take more frequent, relatively longer steps [snippet](https://corescholar.libraries.wright.edu/kinesiology_health/59).
- French children aged 6 to 12: cadence 132.6 → 114.7 steps/min, velocity 99.2 → 120.0 cm/s [snippet](https://www.sciencedirect.com/science/article/pii/S187706571500041X).
- Dusing & Thorpe 2007 (438 children, 1 to 10 y, GAITRite): normalized velocity and step length rise from 1 to 4 y and stabilize from 5 to 10 y [snippet](https://www.sciencedirect.com/science/article/abs/pii/S0966636206001275).
- Rygelová 2023 (64 children, 2 to 6.9 y): speed effect sizes 0.72 (2 vs 3 y), 1.77 (2 vs 6 y); normalized stride width falls strongly (ES 3.17 for 2 vs 6 y); variability falls with age [snippet](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10174554/).
- Froude/Hof normalization removes most size differences in children [snippet](https://clinicalgaitanalysis.com/faq/normalisation.html), [snippet](https://research.dial.uclouvain.be/handle/2078.5/240216). Hof's step relation: s' = 1.1·√v' ± 0.2 [snippet](https://clinicalgaitanalysis.com/faq/speed.html).

**Adults and elderly**
- Bohannon & Andrews 2011 comfortable gait speed (m/s): men 20s 1.36, 30s 1.43, 40s 1.43, 50s 1.43, 60s 1.34, 70s 1.26, 80 to 99 0.97; women 20s 1.34, 30s 1.34, 40s 1.39, 50s 1.31, 60s 1.24, 70s 1.13, 80 to 99 0.94 [snippet](https://www.physicaltherapy.utoronto.ca/sites/default/files/assets/files/11walking-speed-age-and-sex-specific-normative-values-final.pdf).
- WBDS (Fukuchi 2018) re-analysis, young (27.8 ± 4.4 y, n = 22) vs older (64.0 ± 4.9 y, n = 21), trials near comfortable speed. Speed 1.234 vs 1.218 m/s. Step length 0.637 vs 0.606 m (p = 0.012). Cadence 116.5 vs 121.0 steps/min (p = 0.039). Height 1.720 vs 1.643 m. Leg length 0.882 vs 0.857 m. Hof dimensionless: s' 0.724 vs 0.710, f' 0.581 vs 0.595, v' 0.420 vs 0.421 [fetched](https://github.com/BMClab/h2a_age_spt/tree/main/results).
- Hollman 2011 (Mayo, 294 adults aged 70+), cadence steps/min: men 102 ± 8 (70 to 74), 106 ± 10 (75 to 79), 103 ± 8 (80 to 84), 102 ± 11 (85+); women 113 ± 20, 114 ± 13, 110 ± 9, 108 ± 10 [snippet](https://pmc.ncbi.nlm.nih.gov/articles/PMC3104090/).
- Older adults show smaller peak hip extension (Kerrigan 1998, 2001), less plantarflexor power with more hip power (Winter 1990; Judge 1996; DeVita 2000). One small study: −30% hip extension angle, +28% hip flexion angle [snippet](https://pmc.ncbi.nlm.nih.gov/articles/PMC2562040). Review: shorter, slower steps, wider steps and longer double support as a stability strategy [snippet](https://pubmed.ncbi.nlm.nih.gov/26210370/). Double support 26% (middle age) vs 35% (elderly) in a Chinese sample; flashcard-level sources say 18 to 20% young vs 26% elderly [snippet](https://ojs.ub.uni-konstanz.de/cpa/article/view/753/676).

**Datasets with age-diverse motion (license as stated)**

| dataset | ages | content | license | tag |
|---|---|---|---|---|
| Van Criekinge et al. 2023, Sci Data | 138 adults, 21 to 86 y (+50 stroke) | full-body PiG kinematics, kinetics, EMG; C3D + MAT; barefoot preferred speed; per-subject age, sex, height, leg length | CC BY 4.0 (article); data on figshare c.6503791 | [fetched](https://github.com/DaebangStn/gait_dataset) |
| Fukuchi et al. 2018 WBDS, PeerJ | 24 young (21 to 37) + 18 older (57 to 84) | lower-limb + pelvis, overground and treadmill, multiple speeds; c3d/ASCII | CC BY (article); figshare 10.6084/m9.figshare.5722711 | [snippet](https://peerj.com/articles/4640) |
| Lencioni et al. 2019, Sci Data | 50 healthy, **6 to 72 y** | walking at several speeds, toe/heel walking, stairs; markers, joint angles, GRF, EMG | likely CC BY 4.0, not confirmed | [snippet](https://springernature.figshare.com/collections/Human_kinematic_kinetic_and_EMG_data_during_level_walking_toe_heel-walking_stairs_ascending_descending/4494755/1) |
| HumanAttr (with AttrMoGen, AAAI 2026) | 640 subjects, **5 to 88 y**, 18.2k motions, ~2,135 min | text + motion with age and gender labels (74% also height/weight); built from 9 sub-datasets | not checked (depends on the sub-datasets). Verification: a secondary paper note lists sub-sets including BMLmovi, ETRI-Activity3D, KIT and Nymeria; BMLmovi and ETRI-Activity3D are research/non-commercial and Nymeria is CC BY-NC [inference from recall], so assume **non-commercial only** until checked | [snippet](https://arxiv.org/html/2506.21912) |
| MID inertial dataset (2025) | 10 children 5 to 10 y + 10 adults | 17-IMU, BVH, outdoor playground | release/license unknown | [snippet](https://arxiv.org/pdf/2510.17101) |
| Zenodo pediatric IMU set | 25 typically developing + CP + toe-walkers | Xsens IMU | CC BY 4.0 | [snippet](https://zenodo.org/records/17501242) |
| AMASS | adults only, 346 subjects | SMPL mocap | research-only license (no commercial) | [snippet](https://ar5iv.labs.arxiv.org/html/1904.03278) |

**Age-conditioned motion generation**
- AttrMoGen (Wang et al., AAAI 2026, 40(12):10216-10224) disentangles action semantics from attributes. A VQ-VAE gives attribute-free tokens, a text transformer predicts them, and age and gender condition the decoder. Trained on HumanAttr [snippet](https://ojs.aaai.org/index.php/AAAI/article/view/37990).
- A LoRA-MDM age project fits SMPL to Van Criekinge C3D files, exports HumanML3D 263-D features and fine-tunes MDM with age conditioning via LoRA [fetched README](https://github.com/catr1xLiu/LoRA-MDM-Age-Dataset). It is a student pipeline with no license file, so treat it as a pattern only.

### Inferences
**Hof-scaled gait generator** (comfortable speed). l = hip-joint height ≈ 1.09 × subischial leg length; LL/H from the sibling note `proportions_by_age.md`. v', s' targets below 4 y and over 70 y are my calibration to the snippets above. [computed with inference]

| age y | H m | l m | v' | s' | speed m/s | step m | cadence steps/min | check against literature |
|---|---|---|---|---|---|---|---|---|
| 1 | 0.75 | 0.29 | 0.30 | 0.62 | 0.50 | 0.18 | 170 | high cadence, short step (Sutherland trend) |
| 2 | 0.87 | 0.38 | 0.36 | 0.68 | 0.69 | 0.26 | 162 | |
| 3 | 0.96 | 0.45 | 0.39 | 0.72 | 0.82 | 0.32 | 152 | |
| 4 | 1.03 | 0.50 | 0.41 | 0.72 | 0.91 | 0.36 | 151 | step factor mature at 4 |
| 6 | 1.16 | 0.58 | 0.40 | 0.72 | 0.95 | 0.42 | 137 | French 6 y: 0.99 m/s, 132.6/min |
| 10 | 1.38 | 0.71 | 0.41 | 0.72 | 1.08 | 0.51 | 127 | French 12 y: 1.20, 114.7 |
| 14 | 1.62 | 0.85 | 0.42 | 0.72 | 1.21 | 0.61 | 119 | |
| 25 | 1.70 | 0.88 | 0.42 | 0.724 | 1.23 | 0.64 | 116 | WBDS young 1.23 / 0.637 / 116.5 |
| 75 | 1.66 | 0.87 | 0.39 | 0.69 | 1.14 | 0.60 | 114 | Bohannon 70s 1.13 to 1.26; Mayo 102 to 114 |
| 85 | 1.62 | 0.86 | 0.33 | 0.64 | 0.96 | 0.55 | 105 | Bohannon 80-99 0.94 to 0.97; Mayo 102 to 108 |

**Style parameters (beyond speed):**

| parameter | toddler 1 to 2 y | child 3 to 7 y | adult | old 70+ | old 80+ / frail |
|---|---|---|---|---|---|
| step width / l | 0.45 to 0.6† (very wide) | 0.25 → 0.15† | 0.10 to 0.12† | 0.14† | 0.16 to 0.20† |
| double support % cycle | 35 to 45 (novice) | 25 → 20† | ~20 | ~26 | 30 to 35 |
| arm pose | high guard; no reciprocal swing until 1.5 y | swing from 1.5 y, consistent by 3.5 y | normal swing | smaller amplitude† | reduced, often holding aids |
| heel strike | flat-foot contact† | heel strike by ~2 to 3 y† | heel strike | reduced push-off | shuffling, low foot clearance† |
| hip extension peak | low† | adult-like | ~10 to 15°† | −30% | −40%† |
| trunk | lean forward, belly out (lordosis)† | upright | upright | forward lean = rest TK + SVA | stooped (SVA 70 to 90 mm)† |

### Gaps
- No primary numeric table for 1 to 5 y cadence, speed and step length (Sutherland 1980/1988, Dusing & Thorpe 2007 tables not readable). The 1 to 4 y rows above are calibrated, not measured.
- HumanAttr's 9 sub-datasets and their licenses are unknown. arXiv was blocked.
- No toddler or infant mocap dataset with an open license was found. Crawling, cruising and first-steps motion would need video-to-3D capture or hand animation.
- Running, stair, sit-to-stand and fall-recovery changes by age were not researched.

---

## 6. Deliver: per-age bone lengths, rest-pose offsets, joint limits and gait for the 53-bone rig

### Takeaway
- The 53-bone Unreal rig matches the UE mannequin core: root, pelvis, spine_01..03, neck_01, head, and per side clavicle, upperarm, lowerarm, hand, 15 finger bones, thigh, calf, foot, ball. 7 + 2 × (4 + 15 + 4) = 53. [inference]
- Run one function per character and age: `skeleton(age, sex, profile) → {bone_lengths, rest_offsets, joint_limits, gait}`. **Bone lengths come from the solved body, not from tables.** The tables are priors and checks for the knob solver, and the fallback when no drawing exists.

### Cited Findings
- All numeric inputs come from sections 1 to 5 above (their tags apply).

### Inferences

**Step A: target lengths (cm) per bone.** H(age) comes from the growth module (`growth_stature_trajectories.md`).
```python
def long_bones_cm(H, sex, age):
    # KidStats quadratics (pooled, children->adult); H in cm.
    # For age >= ~25 the caller must pass H_peak (adult-peak stature), never the
    # shrunken old-age height: limb bones do not shorten with age.
    F = -4.3986 + 0.204522*H + 0.000497*H*H
    T = -2.7240 + 0.151839*H + 0.000488*H*H
    Hu = -0.9859 + 0.127202*H + 0.000376*H*H
    R = 0.4019 + 0.076564*H + 0.000364*H*H
    if sex == "F" and age >= 13:       # small adult sex terms (inference, unverified)
        R *= 0.985; Hu *= 0.99
    # cartilage / joint-centre factor (inference, no data): 1.13 at 0 y -> 1.0 at 16 y
    k = 1.0 + 0.13*max(0.0, 1 - age/16.0)**1.5
    return dict(thigh=F*k, calf=T*k, upperarm=Hu*k, lowerarm=R*k)
```
- Spine_01..03 + neck_01: trunk length = sitting height − head height (sibling notes give SH/H and head/H by age). Split it over the bones by the Anny joint positions.
- In old age, keep limb bones at adult-peak values. Absorb the stature loss in spine bone lengths (disc and vertebral height, about 60%) plus posture (TK, knee flexion, about 40%). [inference]

**Step B: rest-pose offsets** from the section 3 posture table (interpolate by stage slider).
```python
def rest_offsets(age, sex):
    P = posture_table(age, sex)        # CL, TK, LL, PT, KF, CVA from section 3
    rot = {}
    # distribute Cobb angles over the bones that span them (sagittal X-rotation, +flexion)
    rot["spine_03"] = +P.TK*0.55       # upper/mid thoracic share
    rot["spine_02"] = +P.TK*0.45 - P.LL*0.25
    rot["spine_01"] = -P.LL*0.75
    rot["pelvis"]   = +(P.PT - P.PT_adult_ref)   # posterior tilt = retroversion
    rot["neck_01"]  = -P.CL + (55 - P.CVA)*0.6   # forward head when CVA < 55
    rot["head"]     = -(55 - P.CVA)*0.6          # keep gaze horizontal
    rot["thigh_l"] = rot["thigh_r"] = -P.KF*0.5  # hips flex with knees
    rot["calf_l"]  = rot["calf_r"]  = +P.KF
    rot["foot_l"]  = rot["foot_r"]  = -P.KF*0.5  # ankle dorsiflexion keeps foot flat
    return rot  # then solve SVA: shift pelvis so ear canal is over ankle (gravity line)
```
- Anny's rest pose is adult-neutral. These offsets go into a **per-age pose-space "posture corrective"**, which is the same machinery as the open build item "pose correctives". The rest pose for skinning stays neutral, so the skin weights stay valid. [inference]

**Step C: joint limits** = adult sex limit × the age multiplier from the section 4 table, then the profile overrides (contractures, fused joints, prosthesis). Export to the Unreal Physics Asset constraints (swing1/swing2/twist) and to the IK solver limits.

**Step D: gait parameters** = Hof spec (v', s', f', step width/l, double support %, arm mode, heel strike, hip-extension scale, trunk lean) by stage from the section 5 tables, × leg length l of the solved character. In Unreal, feed speed and cadence into the locomotion blend space (stride warping). Pick the clip family by stage: toddler, child, adult, elderly.

**Step E: X-ray mode.** A per-bone ossification state from the section 2 tables, using skeletal age (F = chronological + 1 y relative to M). It hides BodyParts3D epiphysis sub-meshes, or scales them as nuclei, and draws physis gaps.

### Gaps
- The cartilage factor k, the rib angles and the 80+ posture values are unverified placeholders.
- The anime proportions (Koby at 3.6 heads) need their own check. Hof scaling keeps motion physically plausible for any leg length, but the posture tables are human-realistic, and a manga override should be allowed per stage.

---

## How this plugs into HeroBody

- **Knob solver (build item "knob solver, height last"):** add priors and soft targets for the bone ratios. Femur/H, tibia/H, humerus/H, radius/H come from the KidStats quadratics (section 1). T/F ≈ 0.82 to 0.85 and R/Hu ≈ 0.76 to 0.82 by age come from the salb_za table. Add these to the measure2d landmark set (knee and elbow heights from the blueprint) so the "every ratio within 3%" check includes limb segments. Source files: `hb/measure2d.py` (add knee/ankle/elbow/wrist landmarks) and the solver target table per stage.
- **The four new blendshapes:** "chibi short legs" should move the femur and tibia ratios together (keep T/F about 0.83) so the rig stays plausible. A **baby/toddler limb blendshape** is needed because Anny's age knob breaks at the baby end (Phase 1 atlas). Target femur/H 0.15 to 0.20 and humerus/femur 0.80 at 0 to 1 y.
- **Pose correctives (open build item):** add a **"posture by age" corrective set** driven by the section 3 table: TK, LL, PT, CL, knee flexion, forward head. The 90° bend correctives and the posture correctives share the same corrective solver.
- **Game rig (53 bones, Unreal names):** run `skeleton(age, sex, profile)` at character build time. Write joint limits to the Physics Asset. Write gait parameters to the AnimBP (stage-specific locomotion sets). Rest-pose offsets live as an additive "posture" pose, not as rebinding.
- **Anatomy (BodyParts3D, rig binding):** for X-ray gags and cutaways, add an **ossification state per bone** (section 2), plus a child pelvis (3 parts + triradiate cartilage) and an open-fontanelle skull for 0 to 2 y. The current adult BodyParts3D skeleton cannot produce a child X-ray. This is a new asset need. The per-body clamp still applies after scaling.
- **Character profile (human-in-the-loop):** add fields for skeletal age offset (±2 y, early/late maturer), posture style (upright/average/stooped, as a z-score on TK and SVA), mobility (hypermobile/average/stiff, as a multiplier on ROM), and disability overrides (contracture angles, fused joints, aids such as a cane or walker that change gait style). The manga override can replace any table value per stage.
- **Stage/slider mapping:** each table is indexed in years. The stage slider maps to years (as in the first-pass note), and every function above takes `age_years`.

## Sources
- https://raw.githubusercontent.com/AlisJay/BioArchaeology_In_R/master/Health%20Index/data/M70.txt
- https://raw.githubusercontent.com/AlisJay/BioArchaeology_In_R/master/Health%20Index/HealthIndex.Rmd
- https://github.com/geanes/kidstats
- https://github.com/ElaineYChu/KS-Stature
- https://github.com/BMClab/h2a_age_spt
- https://github.com/DaebangStn/gait_dataset
- https://github.com/catr1xLiu/LoRA-MDM-Age-Dataset
- https://raw.githubusercontent.com/mfbailey91/Function_Generators_in_Open_Chains/main/docs/research/literature/BIOLOGICAL_JOINT_RANGE_REFERENCE_TRACE.md
- https://intarch.ac.uk/journal/issue11/4/28.html
- https://pressbooks.gvsu.edu/introhumanosteology/?p=298
- https://en.wikipedia.org/wiki/Estimation_of_stature
- https://ojp.gov/index%2Ephp/node/790896
- https://biblio.naturalsciences.be/associated_publications/anthropologica-prehistorica/anthropologie-et-prehistoire/ap-105/ap-105_29-32.pdf
- https://www.ojp.gov/ncjrs/virtual-library/abstracts/test-three-methods-estimating-stature-immature-skeletal-remains
- https://core.tdar.org/document/125576/growth-and-predictions-of-growth-in-the-lower-extremities
- https://www.wheelessonline.com/bones/methods-to-estimate-growth-potenital/
- https://actaorthop.org/actao/article/download/20784/24616/68727
- https://healthline.com/health/how-many-bones-does-a-baby-have
- https://case.edu/news/anatomys-darin-croft-explains-why-adults-have-nearly-100-fewer-bones-when-they-are-born
- https://epos.myesr.org/poster/esr/ecr2019/C-3356/Findings%20and%20procedure%20details
- https://wheelessonline.com/?p=16846
- https://radiopaedia.org/articles/24706
- https://prod-images-static.radiopaedia.org/page_images/2363/R23_299_Pediatric_Bone_Maturation_Of_the_Upper_Extremity.pdf
- https://flexikon.doccheck.com/de/Corpus:Carpal_bones
- https://pmc.ncbi.nlm.nih.gov/articles/PMC5337779
- https://www.redalyc.org/pdf/657/65752469009.pdf
- https://www0.sun.ac.za/ortho/webct-ortho/age/risser.html
- https://baes.uc.pt/handle/10316/8067
- https://estudogeral.uc.pt/handle/10316/8067
- https://nicpd.ac.in/ojs-/index.php/njirm/article/view/831
- https://banglajol.info/index.php/BJA/article/view/75532/49929
- https://www.springermedicine.com/the-lateral-clavicular-epiphysis-fusion-timing-and-age-estimatio/21018050
- https://acikerisim.erbakan.edu.tr/items/3aa906b1-467b-40e3-8810-cc791a38012a
- https://www.aafs.org/sites/default/files/media/documents/AAFS-2016-A84.pdf
- https://he01.tci-thaijo.org/index.php/CMMJ-MedCMJ/article/view/247344
- https://www.academia.edu/6356665/Comparison_of_Ages_of_Epiphyseal_Union_in_North_American_and_Bosnian_Skeletal_Material
- https://pubmed.ncbi.nlm.nih.gov/24920348
- https://revistas.rcaap.pt/bgmj/article/view/29291
- https://taylorandfrancis.com/knowledge/Medicine_and_healthcare/Anatomy/Posterior_fontanelle
- https://open-exam-prep.com/study-guides/pa-cat/anatomy-back-abdomen/vertebral-column-curvatures
- https://experts.arizona.edu/en/publications/development-and-functional-anatomy-of-the-spine/
- https://healthlibrary.childrenshospital.org/node/146320
- https://cris.biu.ac.il/en/publications/changes-in-thoracic-cage-morphology-from-birth-into-adulthood/
- https://thorax.bmj.com/content/39/8/624
- https://onlinelibrary.wiley.com/doi/10.1111/joa.12203
- https://pmc.ncbi.nlm.nih.gov/articles/PMC3175924
- https://pmc.ncbi.nlm.nih.gov/articles/PMC2200687
- https://pmc.ncbi.nlm.nih.gov/articles/PMC4045366/
- https://pubmed.ncbi.nlm.nih.gov/6768276/
- https://publish.kne-publishing.com/index.php/JMR/article/download/16417/15321/
- https://josr-online.biomedcentral.com/articles/10.1186/s13018-021-02592-2
- https://synapse.koreamed.org/articles/1122721
- https://link.springer.com/article/10.1007/s00586-016-4807-7
- https://www.scielo.br/j/clin/a/fpk7ncdJt7JMxvph8Qbc35h/abstract/?lang=en
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4340200/
- https://pubmed.ncbi.nlm.nih.gov/26208229/
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5382592/
- https://nagoya.repo.nii.ac.jp/records/2006442
- https://www.hkmj.org/../system/files/hkmj2510sp7p41.pdf
- https://jmr.tums.ac.ir/index.php/jmr/article/view/160
- https://serval.unil.ch/notice/serval:BIB_85340B215DEB
- https://www.mdpi.com/2079-7737/14/6/628
- https://pcs.khmnu.edu.ua/index.php/pcs/en/article/view/511
- https://smj.org.sa/content/22/11/1013.full.pdf
- https://pubmed.ncbi.nlm.nih.gov/21070485/
- https://cdn.clinicaltrials.gov/large-docs/85/NCT03442985/SAP_001.pdf
- https://archive.cdc.gov/www_cdc_gov/ncbddd/jointrom/index_1715172647.html
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9517286/
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12298373/
- https://stacks.cdc.gov/view/cdc/153156/cdc_153156_DS1.pdf
- https://musculoskeletalkey.com/measurement-of-range-of-motion-and-muscle-length-clinical-relevance/
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9937824/
- https://experts.umn.edu/en/publications/normal-range-of-motion-of-the-cervical-spine-an-initial-goniometr/
- https://eorthopod.com/news/is-my-neck-getting-more-stiff-every-year-an-analysis-of-cervical-spine-range-of-motion-changes-over-forty-years/
- https://search.bvsalud.org/gim/resource/en/wpr-65230
- https://oars.uos.ac.uk/5075/
- https://embryology.med.unsw.edu.au/embryology/index.php/Neural_Exam_-_12_month_Motor_9
- https://wiki.ubc.ca/Course:KIN366/ConceptLibrary/Gait
- https://research.vu.nl/ws/files/1725705/134915.pdf
- https://pubmed.ncbi.nlm.nih.gov/23271309/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC6586442/
- https://orthobullets.com/evidence/7364807
- https://www.ouhsc.edu/bserdac/dthompso/web/gait/matgait/matgait.htm
- https://corescholar.libraries.wright.edu/kinesiology_health/59
- https://www.sciencedirect.com/science/article/pii/S187706571500041X
- https://www.sciencedirect.com/science/article/abs/pii/S0966636206001275
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10174554/
- https://clinicalgaitanalysis.com/faq/normalisation.html
- https://clinicalgaitanalysis.com/faq/speed.html
- https://research.dial.uclouvain.be/handle/2078.5/240216
- https://www.physicaltherapy.utoronto.ca/sites/default/files/assets/files/11walking-speed-age-and-sex-specific-normative-values-final.pdf
- https://pmc.ncbi.nlm.nih.gov/articles/PMC3104090/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC2562040
- https://pubmed.ncbi.nlm.nih.gov/26210370/
- https://ojs.ub.uni-konstanz.de/cpa/article/view/753/676
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10692332/
- https://peerj.com/articles/4640
- https://springernature.figshare.com/collections/Human_kinematic_kinetic_and_EMG_data_during_level_walking_toe_heel-walking_stairs_ascending_descending/4494755/1
- https://arxiv.org/html/2506.21912
- https://ojs.aaai.org/index.php/AAAI/article/view/37990
- https://arxiv.org/pdf/2510.17101
- https://zenodo.org/records/17501242
- https://ar5iv.labs.arxiv.org/html/1904.03278

---

## Verification

Adversarial check, 2026-10-09. Access during verification: every non-GitHub host failed again (curl HTTP 000 / WebFetch ENOTFOUND for pmc.ncbi.nlm.nih.gov, pubmed, springer, ojs.aaai.org, wheelessonline, pressbooks.gvsu.edu, utoronto, clinicaltrials.gov). The shared WebSearch budget for this run was already used up, so no new search snippets could be pulled. Data claims were re-computed from fresh clones of the GitHub repositories (salb_za.rda and KS-Stature data.rds read with pyreadr). Other claims were checked against independent GitHub copies where found. Tags: [fetched] = primary file read; [fetched-secondary] = third-party copy read; [inference] = reasoning or recall.

| claim | verdict | note | source |
|---|---|---|---|
| Maresh 1970 femur DL (mm) M/F: 136.6/134.6 (1 y), 172.4/170.8 (2 y), 247.5/247.0 (5 y), 349.3/347.9 (10 y), 386.1/387.6 (12 y) | confirmed | All values and the rest of the table (0.125 to 18 y, -3SD columns) match the file exactly. The file is still a GitHub transcription of the Scheuer & Black reprint, not Maresh's paper. [fetched] | https://raw.githubusercontent.com/AlisJay/BioArchaeology_In_R/master/Health%20Index/data/M70.txt |
| salb_za (n = 1,310, 0.09 to 12.99 y): T/F 0.856 at 0-3 mo to 0.82-0.83 from 2 y; R/H 0.819 to 0.765 by 5 y; H/F 0.825 to 0.69 by 10-12 y | confirmed | Reproduced as medians of per-child ratios with bins (a, b]: 0.856/0.819/0.825 (n = 21), 2 y 0.820, 5 y 0.810/0.765/0.714, 12 y 0.832/0.765/0.692. Ratios of median lengths differ slightly (for example T/F 0.873 in the first bin), so the method matters for the youngest bin. GPL-3 confirmed in DESCRIPTION. [fetched; computed] | https://github.com/geanes/kidstats |
| KidStats: femur DL/H 0.148 (30-50 cm) to 0.265-0.267 (160-180 cm); Femur_cm = -4.3986 + 0.204522·H + 0.000497·H²; H = 32.29 + 3.087·F, r = 0.994, SEE 5.22 | confirmed (with n correction) | Coefficients, r = 0.9938 and SEE = 5.22 reproduce exactly. They are fitted on the 843 rows that have a femur (499 M, 344 F), not all 990, and the bin n values in the table count all rows (30-50 cm has 29 femurs). MIT license confirmed. [fetched; computed] | https://github.com/ElaineYChu/KS-Stature |
| Trotter white male S = 2.38·Fem + 61.41 (±3.27); white female S = 2.47·Fem + 54.10 (±3.72), S = 2.90·Tib + 61.53 (±3.66) | unverifiable (primary) | Coefficients match two independent GitHub implementations (PyExPhys stature.h; ThreeMojo docs), both of which label them the 1952 American White formulae. The male femur line is Trotter & Gleser 1952, reprinted in Trotter 1970. The ± SE values could not be checked. Note: ThreeMojo inverts these regressions, which this note correctly warns against. [fetched-secondary] | https://github.com/dpfens/PyExPhys ; https://github.com/SethKitchen/ThreeMojo |
| Distal femoral physis about 10 mm/yr (70%), proximal tibia about 6 mm/yr (about 60%); growth stops at skeletal age 15-17 boys and 13-15 girls | unverifiable | The source could not be reached. The figures agree with the standard Anderson-Green-Messner / Menelaus teaching (about 3/8 in/yr distal femur, 1/4 in/yr proximal tibia; growth ends about 14 girls, 16 boys) [inference]. The note's Takeaway says 55-60% for tibia while the snippet says 60%; both are within the usual range. | https://www.wheelessonline.com/bones/methods-to-estimate-growth-potenital/ |
| CRITOE sex-specific appearance: capitellum 1, radial head 4/5, medial epicondyle 5/7, trochlea 8/9, olecranon 8/10, lateral epicondyle 11/12 y (F/M) | unverifiable | PMC blocked; no snippet search possible. The sequence and the girls-earlier pattern are standard. The Brazilian ranges in the same section (trochlea 5-11, lateral epicondyle 8-13) show wide spread, so treat these as midpoints only. [inference] | https://pmc.ncbi.nlm.nih.gov/articles/PMC5337779 |
| Lisbon: elbow fuses first (about 11-15 y), medial clavicle last (about 25-27 y), females 1-2 y ahead | unverifiable | Repository not reachable. Consistent with Scheuer & Black general sequence. [inference] | https://baes.uc.pt/handle/10316/8067 |
| Anterior fontanelle (CT): 16% closed at 10 mo, 53% at 16 mo, 88% at 20 mo | unverifiable | PubMed blocked, no GitHub copy found. Plausible against the Portuguese median of 14 months in the same note. [inference] | https://pubmed.ncbi.nlm.nih.gov/24920348 |
| TK 24 ± 5.6° infants vs 36.7 ± 6.5° adults, mostly in first 5 y | unverifiable | Not reachable. Note the tension with Weaver 2014 (TK decreases birth to adolescence) and Mac-Thiong (TK 42-44° in 3-18 y children, above the 36.7° adult value); measurement levels (CT supine vs standing X-ray) differ, so do not mix these in one curve without care. [inference] | https://cris.biu.ac.il/en/publications/changes-in-thoracic-cage-morphology-from-birth-into-adulthood/ |
| Yukawa 626 Japanese: TK 36.0 ± 10.1, LL 49.7 ± 11.2, PI 53.7 ± 10.9, PT 14.5 ± 8.4, SVA 3.1 ± 12.6 mm | unverifiable | Springer blocked; GitHub code search found no copy. Values look internally consistent (PI - PT = 39.2 vs SS 39.4). [inference] | https://link.springer.com/article/10.1007/s00586-016-4807-7 |
| Elderly over 60 (mean 74 y): SVA 51 mm, PI-LL 8.9°, PT 19° | unverifiable | Not reachable. [inference: plausible for community elderly] | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4340200/ |
| Soucie 2011 ROM: 2-8 y hip flexion 140.8/131.1, knee ext 5.4/1.6, DF 24.8/22.8, knee flexion 152.6/147.8; 20-44 y knee flexion 141.9/137.7, shoulder flexion 172.0/168.8, elbow flexion 150.0/144.6 (F/M) | confirmed (20-44 y, secondary) / unverifiable (2-8 y) | The 20-44 y values (plus pronation 82.0/76.9, supination 90.6/85.0) match the secondary note exactly, which cites Haemophilia 17(3):500-507, PMID 21070485. The 2-8 y values could not be re-checked (clinicaltrials.gov blocked). [fetched-secondary] | https://raw.githubusercontent.com/mfbailey91/Function_Generators_in_Open_Chains/main/docs/research/literature/BIOLOGICAL_JOINT_RANGE_REFERENCE_TRACE.md |
| Bohannon & Andrews 2011 comfortable gait speed (m/s): men 1.36/1.43/1.43/1.43/1.34/1.26/0.97, women 1.34/1.34/1.39/1.31/1.24/1.13/0.94 (20s to 80-99) | confirmed (secondary) | All 14 values match an independent copy citing Physiotherapy 97(3):182-189, PMID 21820535 (41 studies, 23,111 subjects). One other GitHub app lists men 50-59 as 1.39; the majority copy and recall give 1.43. A 2022 update (Andrews et al., 51,248 adults) exists. [fetched-secondary] | https://github.com/Bill68Wong/musttoukou (docs/步行速度五档-文献依据-20260918.md) |
| WBDS re-analysis: step 0.606 vs 0.637 m, cadence 121.0 vs 116.5 steps/min, speed 1.218 vs 1.234 m/s, v' 0.421 vs 0.420 | confirmed | All values match descriptive.csv and normalized_descriptive.csv (young n = 22, 27.8 y; older n = 21, 64.0 y; p = 0.012 and 0.039). MIT license. These are selected trials, not the full 24 + 18 WBDS cohort. [fetched] | https://github.com/BMClab/h2a_age_spt/tree/main/results |
| HumanAttr: 18.2k motions, 640 subjects, 5-88 y, age/gender labels; AttrMoGen AAAI 2026 | confirmed (secondary); license flagged | A paper note gives total 640 subjects, 18,199 motions, 2,135.4 min, ages [5, 88], and 4 age groups / 2 genders as discrete labels. Sub-sets shown include BMLmovi, ETRI-Activity3D, KIT and Nymeria; these carry non-commercial or research-only terms [inference from recall], so HumanAttr should be assumed **non-commercial** for HeroBody. Ages 5-18 and 60-88 are sparse. [fetched-secondary] | https://github.com/zhaoyang97/Paper-Notes-en (docs/AAAI2026/human_understanding/generating_attribute-aware_human_motions_from_textual_prompt.md) |

**Net effect on code:** the bone-length quadratics, salb_za ratios, Maresh table, WBDS Hof constants and Bohannon speeds are safe to use. The ossification, fontanelle and spinal-alignment numbers remain snippet-level and should be re-checked when PMC/Springer are reachable. HumanAttr must not be used in a commercial build without a license check.
