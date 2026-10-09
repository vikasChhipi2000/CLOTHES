# Head and face growth and ageing, teeth and hair by age (targets for the HeroBody anime head module)

Research date: 2026-10-09. Researcher notes for the report writer.

**Access notes.** In this session WebFetch failed with DNS errors (`getaddrinfo ENOTFOUND`) for every non-GitHub host tried: pmc.ncbi.nlm.nih.gov, epos.myesr.org, www.cceb.upenn.edu, en.wikipedia.org. curl through the proxy was refused (403 CONNECT) for who.int, cdc.gov, wikipedia, ncbi, europepmc, semanticscholar, frontiersin, plos, mdpi, springer, doi.org, researchgate, figshare, zenodo, osf.io and cran.r-project.org. Only raw.githubusercontent.com was reachable. The shared web-search budget then ran out partway through the work, so some planned searches were never run. Those gaps are marked below. The primary data I **fetched and analysed** with Python in the scratchpad are:

- **Snyder et al. 1977 (CPSC/UM-HSRI) individual records**, ages 2 to 20 y. This is the NIST AnthroKids export: 3,900 subjects, mm, 0 = not measured. I took it from `solcalloni/prediccion-talle-zapatos/chicos.csv` (the same copy used in `proportions_by_age.md`). It includes **head circumference, head breadth, head length, head height, bizygomatic breadth, frontal breadth, "lower face height", "face height", tragion-to-top-of-head, bitragion breadth, mouth breadth and nose length**. That makes it the only fetched source of face dimensions by age. There are 20 to 109 subjects per age-sex cell for the face measures.
- **WHO Child Growth Standards** head circumference and length LMS tables, 0 to 5 y, daily. File `hcanthro.txt` / `lenanthro.txt` from github.com/WorldHealthOrganization/anthro (package licence GPL-3; the standards themselves are WHO).
- **UK 1990 reference (Cole, Freeman, Preece)** LMS for head circumference (0 to 18 y boys, 0 to 17 y girls) and height, from the R package `sitar` data `uk90.rda` (github.com/cran/sitar).

Everything else (Farkas norms, eye and IPD norms, skeletal ageing CT studies, Glocker, teeth, hair) comes from **search-result snippets only** and is tagged [snippet]. Values I recalled from background knowledge but could not check this session are tagged **[inference: background, unverified]**. The report writer should treat those as placeholders until the primary table is read.

Evidence tags: [fetched] = I read or computed from the primary data file. [snippet] = search snippet only. [inference] = my reasoning or arithmetic.

Landmark note for Snyder 1977 [inference]: the adult male "LOWER FACE HEIGHT" median is 121 mm. That matches the Farkas adult male n-gn (nasion to gnathion) of 121.3 mm [snippet]. So I read Snyder's "lower face height" as **morphological face height, sellion/nasion to menton**. Snyder's "FACE HEIGHT" (182 mm in adult men) is longer, so it is probably measured from the hairline (trichion) or forehead to menton. I did not re-read the 1977 landmark definitions, so I use "lower face height" = n-gn and do not rely on "face height".

---

## 1. Head proportions by age: cranial vault vs face, eye height on the head, eyeball, IPD, head circumference to face height

### Takeaway
- The cranium grows early and the face grows late. At birth, neurocranium volume is about 8 to 9 times the facial skeleton volume. It is 5:1 at 2 y, 3:1 at 6 y and 2:1 in adults.
- Head circumference (HC) is already 61.5% of adult size at birth, 83% at 1 y, 90% at 3 y and 95% at 10 y. The face lags well behind.
- On real children, the **eyes are not much bigger relative to the head**. Eyeball diameter divided by head radius is about 0.30 at birth and 0.26 in adults. The palpebral fissure is 0.35 vs 0.32. What changes is the **vertical layout**: the face below the eyes is short in children. The nasion sits 53% of head height down from the vertex at 2 to 3 y, versus 45% (men) to 47% (women) at 18 to 20 y. Eye level moves up the head with age.
- Interpupillary distance (IPD) goes from about 46 mm at 1 to 2 y to about 51 mm at 4 to 5 y, then about 63 mm in adults.

### Cited Findings
- Neurocranium to face **volume** ratio: 8-9:1 at birth, 5:1 at 2 y, 3:1 at 6 y, 2:1 in adults. A 2D version (area on a lateral skull radiograph) gives cranium:face 4-4.5 at birth, 3-3.5 at 2 y, 2.5 at 6 y and 1.5-2 in adults. This is an educational radiology poster, not a primary study — [ESR EPOS C-1449 "Development of the skull"](https://epos.myesr.org/poster/esr/ecr2016/C-1449/background) [snippet].
- **Head circumference, WHO 2006 median (cm)** [fetched; `hcanthro.txt`]:

| age y | boys HC | girls HC | boys length | HC / length (boys) |
|---|---|---|---|---|
| 0 | 34.46 | 33.88 | 49.88 | 0.691 |
| 0.5 | 43.32 | 42.19 | 67.59 | 0.641 |
| 1 | 46.06 | 44.89 | 75.74 | 0.608 |
| 2 | 48.25 | 47.18 | 87.80 | 0.550 |
| 3 | 49.46 | 48.51 | 96.07 | 0.515 |
| 5 | 50.74 | 49.92 | 109.96 | 0.461 |

- **Head circumference, UK 1990 median (cm), with HC as a fraction of the 18 y value** [fetched; `uk90.rda`]:

| age y | boys HC | girls HC | boys HC / adult | boys height | boys HC / height | girls HC / height |
|---|---|---|---|---|---|---|
| 0 | 35.20 | 34.54 | 0.615 | 51.0 | 0.690 | 0.688 |
| 1 | 47.70 | 46.48 | 0.833 | 75.5 | 0.632 | 0.629 |
| 2 | 50.22 | 49.02 | 0.877 | 86.8 | 0.579 | 0.571 |
| 3 | 51.47 | 50.33 | 0.899 | 95.4 | 0.539 | 0.532 |
| 6 | 53.17 | 52.17 | 0.929 | 115.9 | 0.459 | 0.452 |
| 10 | 54.48 | 53.74 | 0.951 | 138.4 | 0.394 | 0.388 |
| 14 | 55.86 | 54.87 | 0.976 | 162.4 | 0.344 | 0.344 |
| 16 | 56.59 | 55.32 | 0.988 | 173.4 | 0.326 | 0.339 |
| 18 (girls 17) | 57.26 | 55.52 | 1.000 | 177.1 | 0.323 | 0.340 |

- **Snyder 1977 head and face dimensions (median mm)**, boys / girls [fetched; computed from `chicos.csv`]:

| age y | head height (v-menton) | "lower face height" (≈ n-gn) | bizygomatic breadth | head breadth | head length | nose length | mouth breadth | tragion to top of head |
|---|---|---|---|---|---|---|---|---|
| 2-3 | 177 / 169 | 82 / 81 | 108 / 106 | 135 / 132 | 180 / 173 | 30 / 29 | 33 / 33 | 114 / 112 |
| 4-5 | 182 / 177 | 87 / 86 | 113 / 111 | 138 / 136 | 182 / 178 | 33 / 32 | 35 / 35 | 116 / 116 |
| 6-7 | 188 / 185 | 91 / 90 | 120 / 117 | 141 / 138 | 183 / 181 | 36 / 35 | 38 / 37 | 120 / 116 |
| 8-9 | 192 / 189 | 96 / 95 | 120 / 119 | 143 / 141 | 187 / 184 | 39 / 39 | 39 / 40 | 120 / 116 |
| 10-11 | 198 / 196 | 100 / 98 | 124 / 122 | 145 / 143 | 187 / 184 | 42 / 41 | 40 / 41 | 123 / 119 |
| 12-13 | 201 / 200 | 103 / 103 | 126 / 126 | 147 / 146 | 190 / 186 | 44 / 44 | 41 / 42 | 125 / 119 |
| 14-15 | 210 / 203 | 111 / 106 | 132 / 129 | 150 / 147 | 193 / 190 | 48 / 45 | 44 / 44 | 126 / 121 |
| 16-17 | 220 / 205 | 114 / 108 | 136 / 132 | 153 / 148 | 198 / 188 | 49 / 45 | 46 / 45 | 129 / 123 |
| 18-20 | 220 / 202 | 121 / 105 | 139 / 131 | 154 / 149 | 201 / 188 | 51 / 46 | 48 / 47 | 131 / 123 |

  There are 20 to 109 subjects per cell. The 18-20 cells are the smallest (20 to 27).
  (Verification note: the age bins are by **rounded** age, so "2-3" = 1.5 to 3.49 y and "18-20" = 17.5 y and over; almost all of the "18-20" subjects are 17.5 to 18.9 y, only 3 are 19+. With strict bins [18, 21) the face-measure cells shrink to 13-14 boys and 9-10 girls.)

- **Vertical layout from Snyder (per-subject ratios, median)** [fetched]:

| age y | nasion depth from vertex / head height, boys | same, girls | n-gn / head height, boys | head height / HC, boys | head breadth / head length, boys |
|---|---|---|---|---|---|
| 2-3 | 0.534 | 0.526 | 0.46 | 0.355 | 0.760 |
| 6-7 | 0.512 | 0.514 | 0.49 | 0.361 | 0.770 |
| 10-11 | 0.495 | 0.498 | 0.51 | 0.371 | 0.778 |
| 14-15 | 0.475 | 0.477 | 0.53 | 0.382 | 0.779 |
| 18-20 | 0.452 | 0.471 | 0.55 | 0.390 | 0.781 |

  Head breadth / head length (cephalic index) stays near 0.76 to 0.78 from 2 to 20 y in this sample.

- **Eyeball axial length**: full-term newborn 16 to 18 mm (one source gives 16.8 mm). Most of the growth happens in the first 3 to 6 months of life. About 3.9 mm is added in the first 2 y and 1.2 mm from 2 to 5 y, reaching 23.6 mm in young adults. The eye reaches its adult emmetropic axial length by about 13 y. Adult range 22 to 25 mm — [JCDR axial length study](https://jcdr.net/ReadXMLFile.aspx?id=3473), [Review of Myopia Management](https://reviewofmm.com/whats-normal-whats-not-emmetropisation-and-normal-ocular-growth-in-caucasian-and-asian-children/), [eophtha normal values](https://www.eophtha.com/posts/normal-values-in-ophthalmology) [snippet].
- **Palpebral fissure length (visible eye width)**: full-term newborns have a mean of 19.4 mm (range 16 to 25 mm). Adults are about 30 mm wide and 8 to 11 mm high. In a Korean sample (1 month to 92 y), over 70% of people aged 11 to 60 measured 27 to 32 mm. In Caucasians, palpebral measures reach adult size between 8 and 16 y depending on sex and measure — [EyeWiki dysmorphology](https://eyewiki.org/Dysmorphology_of_the_Eye_and_Periorbital_Region), [PubMed 1707200](https://pubmed.ncbi.nlm.nih.gov/1707200/), [Astley PFL charts](https://depts.washington.edu/fasdpn/pdfs/pfl2012.pdf) [snippet].
- **IPD in children** (Caucasian, newborn to 6 y; far / near fixation, mm): newborn near 40.5. 12-23 months 46.5 / 43.0. 24-35 months 47.5 / 43.5. 36-47 months 49.5 / 46.0. 48-59 months 51.0 / 46.5. 48-71 months about 51.0 / 46.5 — [Pacific University thesis](https://commons.pacificu.edu/works/publication-dissertation/cmjf8-q9b78) [snippet]. MacLachlan and Howland (2002, *Ophthalmic Physiol Opt* 22:175-182) give means and SDs from 1 month to 19 y, but I could not read the table — [arXiv 2604.15328 citing it](https://arxiv.org/pdf/2604.15328) [snippet]. A 2020s EHR study (1,440 children, 0 to 18 y) gives sex-stratified nomograms for inner canthal, outer canthal and interpupillary distance — [ACMG abstract](https://www.acmgmeeting.net/conference-program/assessment-interpupillary-distance-racially-ethnically-diverse-pediatric-sample-electronic-health-record-data) [snippet].
- Drawing-anatomy sources agree on the direction: infant eyes sit **below** the half-height line of the head, adult eyes sit **near** it, and the upper skull changes little while the nose, mouth and jaw "grow down, out and away" — [Rimmer 1864 plate, Heidelberg](https://digi.ub.uni-heidelberg.de/diglit/rimmer1864/0048), [mybluprint child's face](https://www.mybluprint.com/article/childs-face-mastering-proportions) [snippet].
- **Cardioidal strain** (Pittenger and Shaw 1975; Mark and Todd 1983 in 3D) is a classic one-parameter head growth transform. More strain shrinks the forehead, pushes the chin forward and moves the features up the face. It changes perceived age, and its effects are large in the first 20 years and small after that — [polar growth paper, White Rose](https://eprints.whiterose.ac.uk/185518/1/polargrowth-v7.pdf), [Bond University](https://research.bond.edu.au/en/publications/further-experiments-on-the-perception-of-growth-in-three-dimensio/) [snippet].

### Inferences
- **Head radius unit.** Define r = HC / (2π). With the UK90 medians, r = 5.60 cm at birth, 7.59 cm at 1 y, 8.19 cm at 3 y, 8.46 cm at 6 y, 8.67 cm at 10 y, 8.89 cm at 14 y and 9.11 cm in adult men [inference from fetched data].
- **Eye size in head units** [inference]. Eyeball diameter / r = 1.68 / 5.60 = 0.30 at birth and 2.36 / 9.11 = 0.26 in adults (×1.16 at birth). Palpebral fissure / r = 19.4 / 56.0 = 0.346 at birth and 30 / 91.1 = 0.329 adult (×1.05; corrected in verification: was ×1.07 from rounded ratios). So a realistic baby eye is only about 5 to 16% larger in head units than an adult eye. Big baby eyes in anime are a style choice, not anatomy.
- **IPD in head units** [inference]: 46.5 mm / 75.9 mm = 0.61 at 1 y, 51 / 83 = 0.61 at 4 to 5 y, and 63 / 91 = 0.69 adult. Real toddler eyes are slightly **closer together** in head units. (The adult IPD of about 63 mm is background knowledge, not checked this session.)
- **Eye-line height** [inference]. The eye centre sits about 8 to 10 mm below nasion. Eye-line depth from vertex / head height ≈ nasion depth + 0.04 to 0.05. That gives about **0.58 at 2 to 3 y, 0.56 at 6 y, 0.54 at 10 y, 0.52 at 14 y and 0.50 (men) to 0.51 (women) in adults**. Linear extrapolation of the Snyder trend plus the cranium:face ratios gives about **0.60 to 0.62 at birth**.
- **n-gn / head height** goes from 0.46 (2 to 3 y) to 0.55 (adult men). The extrapolation to birth is about 0.40 to 0.42. That is consistent with the 2D area ratio falling from 4.25 to 1.75.
- **Cardioidal strain equation** [inference: background, unverified]. In polar coordinates (θ = 0 at the top of the head, origin near the head centre): θ' = θ, R' = R·(1 + k·(1 − cos θ)). k > 0 "ages" the profile. Ramanathan and Chellappa used this for young-face age progression. Useful as a cheap 2D silhouette check, not as the main growth model.

### Gaps
- No fetched data under 2 y for face dimensions. The Farkas 1-year norms and Snyder's 1975 infant file were not reachable.
- The MacLachlan and Howland IPD table (1 month to 19 y) was not read.
- The exact eye-centre to nasion offset by age is assumed, not measured.
- The cardioidal strain equation was not verified against Pittenger and Shaw (1975) or Todd et al. (1980).

---

## 2. Craniofacial anthropometric norms by age (Farkas and others)

### Takeaway
- The standard source is **Farkas, Hreczko and Katic (1994)**, *Craniofacial norms in North American Caucasians from birth (one year) to young adulthood*. It is Appendix A (pp. 241-335) of Farkas (ed.), *Anthropometry of the Head and Face*, 2nd ed., Raven Press 1994. It has 132 measurements for ages 1 to 18 y with about 30 subjects per age and sex, plus young adult norms (about 109 men and 200 women, 18 to 25 y). The book is listed on the Internet Archive lending library. I could not read the tables.
- The adult multi-ethnic norms are in **Farkas, Katic and Forrest 2005**, *J Craniofac Surg* 16:615-646 (1,470 subjects aged 18 to 30). Its Table 1 is the North American white reference.
- Age 2 to 20 y face dimensions are available **today** from the Snyder 1977 file (Section 1 table: head height, n-gn, bizygomatic breadth, nose length, mouth breadth). That is enough to drive the head module. Ear dimensions and lip heights are missing.

### Cited Findings
- Farkas 1994 Appendix A: norms for 132 craniofacial measurements, North American Caucasians, 1 to 18 y; young adult samples of about 109 men and 200 women aged 18 to 25 — [Farkas 1996 AJMG](https://onlinelibrary.wiley.com/doi/10.1002/ajmg.1320650102), [Internet Archive listing](https://archive.org/details/anthropometryofh0000unse) [snippet].
- Farkas et al. also measured 600 healthy white North Americans aged 16 to 90 y (adult ageing data) — same AJMG snippet [snippet].
- Farkas 2005 international study: 1,470 healthy subjects aged 18 to 30 (750 men, 720 women). North American whites are the reference group. Measures include zy-zy (face width), al-al (nose width), ch-ch (mouth width) and n-gn (face height) — [PubMed 16077306](https://ncbi.nlm.nih.gov/entrez/query.fcgi?cmd=Retrieve&db=PubMed&dopt=Abstract&list_uids=16077306), [ResearchGate table](https://www.researchgate.net/figure/Normal-Range-of-Measurements-of-North-American-White-Young-Adult-Population_tbl1_7682966) [snippet].
- Farkas n-gn, young adults: 121.3 mm men and 111.8 mm women, reproduced in a secondary arXiv paper — [arXiv 2008.06989](https://arxiv.org/pdf/2008.06989) [snippet]. Snyder 18 to 20 y "lower face height" is 121 mm (men) and 105 mm (women) [fetched], which supports reading that column as n-gn.
- Farkas, Posnick and Hreczko 1992 (*Cleft Palate Craniofac J* 29:308-315; 1,594 subjects aged 1 to 18): at 1 y the mandible has reached **80.2% of adult width but only 66.6% of adult height**. Mandible height and width grow strongly from 1 to 5 y. Face height, upper face height, face width and face depth keep growing gradually after 5 y. In girls, upper face height, mandible height and face width are mature at 12 y. In boys, face height, mandible height, face width and mandible depth are mature at 15 y. The face matures at 12 to 15 y in boys and about 2 years earlier in girls — [PubMed 1643058](https://pubmed.ncbi.nlm.nih.gov/1643058/) [snippet].
- Population norms do not transfer cleanly between ethnic groups — [Romanian pilot study, PMC8156684](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8156684/), [Kenyan norms, PMC6384287](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6384287/) [snippet].

**Young-adult Farkas values recalled from background knowledge** [inference: background, unverified; replace with Appendix A / 2005 Table 1 values]. Men / women, mm:

| measure | landmarks | men | women |
|---|---|---|---|
| face height | n-gn | 121 | 112 |
| face width | zy-zy | ~137 | ~130 |
| mandible width | go-go | ~97 | ~91 |
| nose height | n-sn | ~55 | ~51 |
| nose width | al-al | ~35 | ~31 |
| mouth width | ch-ch | ~54 | ~50 |
| upper lip height | sn-sto | ~22 | ~20 |
| ear length | sa-sba | ~63 | ~58 to 59 |
| ear width | pra-pa | ~35 | ~33 |
| intercanthal | en-en | ~33 | ~32 |
| biocular | ex-ex | ~91 | ~88 |

Only n-gn (men 121.3, women 111.8) has snippet support.

### Inferences
- **Growth fractions (2 to 3 y value / 18 to 20 y value), Snyder** [inference from fetched data]:

| measure | boys | girls |
|---|---|---|
| head breadth | 0.88 | 0.89 |
| head height | 0.80 | 0.84 |
| bizygomatic (face) width | 0.78 | 0.81 |
| n-gn face height | 0.68 | 0.77 |
| mouth breadth | 0.69 | 0.70 |
| nose length | 0.59 | 0.63 |

  Order from fastest to slowest maturing: cranium > face width > face height ≈ mouth > nose. This matches Farkas 1992 (mandible width at 1 y is 80% of adult, height 67%).
- Men's face height keeps growing to 18 to 20 y in Snyder (n-gn 114 mm at 16 to 17 y, 121 mm at 18 to 20 y). Women plateau by about 14 to 15 y (106 mm, then 105 to 108 mm). That fits the Farkas sex timing.
- Ear length: no age data was fetched. Background expectation [inference: background, unverified]: about 40 to 45 mm at 1 y, about 85 to 90% of adult length by 8 to 10 y, adult about 60 to 63 mm, then about +0.22 mm/y lifelong (Section 3).

### Gaps
- The Farkas Appendix A age tables (lip heights, ear length and width, nose width, intercanthal widths) were not read. The best route is the Internet Archive borrow copy of the 1994 book, or a library copy of Farkas 2005.
- Snyder has no ear length, no nose width and no lip heights.
- Mixed-ethnicity norms are not covered. One Piece characters span many ethnic looks, so per-character style offsets will matter more than the population norm.

---

## 3. Puberty growth and adult facial ageing, quantified

### Takeaway
- **Puberty.** The lower face, nose and mandible keep growing after the cranium is nearly done. Boys' faces mature at about 15 y and girls' at about 12 to 13 y. In Snyder, the male nose grows from 42 mm (10 to 11 y) to 51 mm (18 to 20 y), +21%. The female nose grows from 41 to 46 mm, +12%. Male n-gn grows from 100 to 121 mm (+21%) and female from 98 to 105 mm (+7%).
- **Adult bone.** Ageing is not shrinkage everywhere. It is targeted resorption. The orbital aperture gets bigger (inferolateral rim first, by middle age; superomedial in old age). The maxillary angle drops by about 10 to 11°. The glabellar angle drops by about 6°. The pyriform aperture area grows. The mandible gets shorter and lower and its angle opens. Changes occur earlier in women (young to middle age) and later in men (middle to old age).
- **Soft tissue.** The ears lengthen by about 0.22 mm/y. The nose lengthens by about 1.2 to 1.7 mm per 12 y and the tip droops. The cutaneous upper lip lengthens while the vermilion gets thinner, with change speeding up after 40. Facial fat sits in separate compartments; these lose volume and shift unevenly.
- **Wrinkles.** Forehead lines and crow's feet appear in the 20s. Smile lines rise fast from about 23 to 24 y. Frown and glabellar lines appear around 30 to 40+. Perioral lines appear in the 50s. Men wrinkle earlier and more.

### Cited Findings
- Face maturation: boys 12 to 15 y, girls about 2 y earlier (Farkas 1992) — [PubMed 1643058](https://pubmed.ncbi.nlm.nih.gov/1643058/) [snippet].
- Puberty face growth from Snyder (median mm, 10 to 11 y → 18 to 20 y): boys nose 42 → 51, n-gn 100 → 121, bizygomatic 124 → 139, head breadth 145 → 154. Girls nose 41 → 46, n-gn 98 → 105, bizygomatic 122 → 131. Bizygomatic / head breadth rises from 0.80 to 0.90 in boys and 0.82 to 0.89 in girls between 2 and 20 y [fetched].
- Mendelson and Wong 2012 review: the orbital aperture increases in area and width with age. Resorption is uneven: the inferolateral rim changes by middle age, the superomedial rim mainly in old age. The maxillary angle decreases by about 10° between young (< 30 y) and old (> 60 y) people — [PMC3404279](https://pmc.ncbi.nlm.nih.gov/articles/PMC3404279) [snippet].
- Longitudinal 3D CT, 96 Caucasian adults scanned about 11 years apart (Aesthetic Surg J 46(4):366): pyriform angle −5°, maxillary angle −11°, glabellar angle −6.5°, orbital aperture area +91 mm². Most changes were largest at ages 30 to 50. The snippet also says the mandibular angle "grew by about 30 degrees". That is far bigger than other reports, so I treat it as a summarising error until the paper is read — [ASJ 46/4/366](https://academic.oup.com/asj/article/46/4/366/8200963) [snippet].
- Shaw and Kahn 2007 (*Plast Reconstr Surg* 119:675-681; 60 Caucasians): glabellar and maxillary angles decrease with age in both sexes, and the pyriform aperture area increases. One summary says the pyriform angle did not change; another says it did — [Springer summary](https://link.springer.com/article/10.1007/s00266-012-9904-3), [HealthDay](https://www.healthday.com/healthpro-news/cosmetic/bony-elements-in-the-mid-face-change-over-time-602251.html) [snippet].
- Shaw et al. 2010, "Aging of the mandible" (*Plast Reconstr Surg* 125:332-342; 120 Caucasians, 3D CT; young 20-36, middle 41-64, old 65+): the mandibular angle increases, mandible length decreases (young → middle) and mandible height decreases (middle → old). Women change earlier (young → middle) and men later (middle → old) — [ScienceDaily](https://sciencedaily.com/releases/2010/03/100323121836.htm), [DrBicuspid](https://www.drbicuspid.com/clinical/imaging-cad-cam/3d-printing/article/15359264/jaw-bone-plays-key-role-in-appearance-over-time) [snippet].
- Taiwanese (Chinese) CT: orbital aperture area increased with age only in men. Japanese CT: the orbital aperture becomes more elliptical after about 60 y — [NTU thesis](https://tdr.lib.ntu.edu.tw/handle/123456789/15894), [ASJ 43/4/420](https://academic.oup.com/asj/article/43/4/420/6967154) [snippet].
- Rohrich and Pessa 2007 (*Plast Reconstr Surg* 119:2219-2227; 30 hemifacial cadavers): subcutaneous facial fat is split into separate compartments. The nasolabial fat is a discrete unit. "Malar fat" is three compartments (medial, middle, lateral temporal-cheek). There are three superficial forehead compartments. Deep compartments (deep medial cheek, SOOF, submental) were described in 2008. Volume loss in compartments is linked to the hollow look of ageing — [UT Southwestern](https://utsouthwestern.elsevierpure.com/en/publications/the-fat-compartments-of-the-face-anatomy-and-clinical-implication), [McMaster PDF](https://surgery.healthsci.mcmaster.ca/wp-content/uploads/2023/05/2007_facial_fat_pads_anatomy.pdf), [NBC News](https://www.nbcnews.com/health/health-news/cheeky-study-finds-beauty-secret-cadavers-flna1C9462204) [snippet].
- **Ear**: Heathcote (BMJ 1995; 206 patients aged 30+): ear length = 55.9 mm + 0.22 mm × age (95% CI of the slope 0.17 to 0.27), about 1 cm over 50 years. Japanese and Italian studies confirm growth with age in both sexes. A Texas study found ear circumference rises about 0.51 mm/y — [McGill bigears PDF](https://jhanley.biostat.mcgill.ca/bios601/Surveys/BigEars.pdf), [Discover](https://www.discovermagazine.com/health/british-ears) [snippet].
- **Nose**: longitudinal photographs about 12 y apart: profile nasal length +1.7 ± 1.7 mm (men), +1.4 ± 1.9 mm (women). Nasal bridge height +1.2 mm (men). The tip turns downward. Anatolian men by age group (20-40, 40-60, 60+): nasal bridge length 60.30, 63.43 and 64.63 mm. A 2022 CT study found nasal length and nasofrontal angle increase in men, the tip droops, nasal skin and soft tissue thicken and the nasal bone resorbs — [Clin Exp Otorhinolaryngol 2024, PMC10933809](https://pmc.ncbi.nlm.nih.gov/articles/PMC10933809), [Springer Anatolian men](https://www.springermedicine.com/age-related-changes-in-the-external-noses-of-the-anatolian-men/20766398), [PubMed 35994354](https://pubmed.ncbi.nlm.nih.gov/35994354/) [snippet].
- **Lips**: 3D stereophotogrammetry in 169 Chinese women: the cutaneous upper and lower lip heights increase, upper vermilion height decreases, and the vermilion border flattens, with changes speeding up after 40. Korean women 60-79 vs 20-39: shorter lower vermilion and a longer, wider philtrum. A CBCT study found women in their 40s had greater mouth width than women in their 20s — [PMC7984336](https://pmc.ncbi.nlm.nih.gov/articles/PMC7984336/), [PMC7875215](https://pmc.ncbi.nlm.nih.gov/articles/PMC7875215) [snippet].
- **Wrinkles**: 200 people aged 20 to 70: periorbital lines appear first in women and forehead lines first in men. Glabellar lines were not seen before 40 in either sex. Men wrinkle earlier and more severely — [PubMed 24267416](https://pubmed.ncbi.nlm.nih.gov/24267416/) [snippet]. Review: forehead creases and crow's feet appear by the third decade, glabellar lines at variable ages, and perioral lines in the fifth decade — [ehb dermatology topic](https://bibliotheek.ehb.be:2400/topics/en-us/859/diagnosis-approach) [snippet]. AI study (431,321 people; industry conference, not peer reviewed): smile lines (crow's feet, nasolabial) rise fast from 23 to 24 y and level off in the early 50s. Frown lines start around 30 and level off in the late 50s — [MedEsthetics](https://www.medestheticsmag.com/news/news/22864571/evelab-insight-announces-preliminary-findings-of-skin-aging-with-aidriven-research) [snippet].

### Inferences
- **Code-ready adult soft-tissue drift** (apply after 25 to 30 y) [inference from snippets]:

| feature | rule |
|---|---|
| ear length | +0.22 mm/y. At age a: L = L30 + 0.22 × (a − 30). This is +11 mm (+17 to 18%) by 80. |
| nose length | +1.2 mm per decade from 30 (≈ +2.4%/decade); tip rotates down by 2 to 4° per decade (the angle rate is my guess). |
| upper vermilion height | −0% to 40 y, then −10 to 15% per decade (my guess; direction and the after-40 speed-up come from the snippets). |
| cutaneous upper lip (sn to vermilion) | +3 to 5% per decade after 40 (guess). |
| orbit aperture | +91 mm² over about 11 y in the 30-60 range. That is about +5 to 8% area (adult orbit opening area is about 1,200 to 1,400 mm², background). Model it as the inferolateral rim moving down and out. |
| maxilla | maxillary angle −10 to 11° from young to old. Model it as the cheekbone base moving back, so the midface flattens. |
| mandible | length and height −3 to 6% (guess) from young to old; gonial angle +3 to 6° (guess); women start earlier. |
| fat | cheek compartments (medial, middle malar) lose 20 to 40% volume by 70 (guess). Jowl and submental gain. The nasolabial fold deepens. |

- **Puberty for the head module**: n-gn/head height rises from 0.51 to 0.55 in boys between 10 and 20 y and from 0.50 to 0.53 in girls. Nose length/r rises from 0.49 to 0.57 in boys and 0.49 to 0.53 in girls [inference from fetched data]. Brow ridge and frontal sinus growth in boys is the usual textbook claim, but no numbers were found this session.

### Gaps
- No mm-per-decade numbers were read for mandible, maxilla or orbit. The ASJ longitudinal paper and Shaw 2010 tables are the targets.
- No brow-ridge or frontal-sinus growth data by age.
- Lip thinning in mm per decade is not quantified in any source I saw.
- No wrinkle-onset data stratified by ethnicity.

---

## 4. Baby schema (Kindchenschema) and how anime draws age

### Takeaway
- Glocker et al. 2009 measured baby schema with **face width (pixels, with head length fixed) plus five ratios**: forehead length / face length, eye width / face width, nose length / head length, nose width / face width and mouth width / face width. High baby schema = rounder face, higher forehead, bigger eyes, smaller nose and mouth. They edited faces within **±2 SD** of their sample. Cuteness ratings and caretaking motivation rose with baby schema.
- Real infant faces differ from adults mostly in **vertical layout** and **nose/mouth size**, not in eye size relative to the head (Section 1). Anime exaggerates eye size.
- Only one peer-reviewed quantitative cartoon study was found (MDPI *Symmetry* 2019, 100 characters). It measured Japanese animation eye area at **3.4× human** and American cartoon eye area at 2× human. No study measured how anime eye size changes with a character's age. Artist guides say younger characters get larger eyes relative to the face, a rounder outline, a lower eye line and shorter spacing between hairline, eyes, mouth and chin.

### Cited Findings
- Glocker, Langleben, Ruparel, Loughead, Gur and Sachser 2009, *Ethology* 115:257-263: infant faces manipulated to high or low baby schema (e.g. round face and high forehead vs narrow face and low forehead); 122 students rated cuteness and caretaking motivation, both higher for high baby schema — [UPenn repository](https://repository.upenn.edu/cog_neuro_pubs/11) [snippet].
- The Glocker measures: face width in pixels plus five indices, fol/fal, ew/fw, nl/hl, nw/fw, mw/fw. Head length was fixed at 500 px (PNAS) or 600 px (later reuse). Manipulations were guided by the sample mean and SD and limited to ±2 SD — [figshare "Objective measures of baby schema"](https://figshare.com/articles/dataset/_Objective_measures_of_baby_schema_cf_Glocker_et_al_2_/1368417/1), [Frontiers Psychol 2014](https://public-pages-files-2025.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2014.00411/pdf), [PNAS 2009](https://www.pnas.org/doi/10.1073/pnas.0811620106) [snippet].
- Lorenz's baby schema list: large head, high and protruding forehead, large eyes, chubby cheeks, small nose and mouth, short thick limbs, plump body — [PMC4019884](https://pmc.ncbi.nlm.nih.gov/articles/PMC4019884/) [snippet].
- Japanese animation characters have an eye area 3.4× that of human faces; American cartoons 2×. Both exaggerate eyes, nose, ears, forehead and chin — [MDPI Symmetry 11(5):664](https://www.mdpi.com/2073-8994/11/5/664) [snippet].
- Artist guides (not peer reviewed): realistic eye-to-face ratio about 1:10, anime about 1:3 to 1:4, and bigger eyes read younger. Lower eyes read more babyish. Spacing between hairline, eyes, mouth and chin widens with age. Children's irises are proportionally larger — [Clip Studio "Drawing characters of all ages"](https://tips.clip-studio.com/en-us/articles/7059), [Anime Art Magazine](https://animeartmagazine.com/how-to-represent-different-ages-in-anime-women/), [TV Tropes "Animation Anatomy Aging"](https://tvtropes.org/pmwiki/pmwiki.php/Main/AnimationAnatomyAging) [snippet].

### Inferences
- **Real-face Glocker-style indices by age, from Snyder** [inference from fetched data]. Using head height in place of Glocker's head length, and bizygomatic breadth as face width:

| age y | nose length / head height (nl/hl-like), boys | girls | mouth width / face width (mw/fw), boys | girls |
|---|---|---|---|---|
| 2-3 | 0.164 | 0.172 | 0.301 | 0.307 |
| 6-7 | 0.198 | 0.188 | 0.323 | 0.321 |
| 10-11 | 0.213 | 0.212 | 0.323 | 0.336 |
| 14-15 | 0.229 | 0.223 | 0.336 | 0.341 |
| 18-20 | 0.232 | 0.235 | 0.343 | 0.355 |

  The nose index drops by 30% from adult to toddler and the mouth index by 13%. These are the two strongest real "baby" signals after vertical layout.
- **Practical cuteness control** for the head module: one scalar `babySchema` in [−2, +2] SD, mapping to forehead fraction up, eye width up, nose length and width down, mouth width down and face roundness up. That matches the Glocker parameter set and its validated range.
- **Anime stage defaults** (style layer, not realism) [inference: art practice, unvalidated]: eye width / face width about 0.30 to 0.35 for baby and child, 0.25 to 0.30 for teen, 0.18 to 0.25 for adult, and 0.15 to 0.22 for old. This is one style family only; One Piece itself uses smaller adult eyes than shoujo styles. Eye line at 0.58 to 0.62 of head height from the top for babies down to about 0.50 to 0.52 for adults, following the real data.

### Gaps
- The Glocker per-feature SD values and the exact manipulation size were not read.
- No measured dataset of anime or manga characters by age. A small in-house study is cheap: measure One Piece characters with known canon ages (e.g. flashback child vs adult versions of the same character) using `hb/measure2d.py`.

---

## 5. Teeth timeline

### Takeaway
- 20 primary teeth erupt from about 6 months to about 33 months. Mixed dentition runs from about 6 y (first permanent molars and lower central incisors) to about 12 y (last primary molars shed). All permanent teeth except third molars are in by about 13 y. Third molars erupt at 17 to 21 y if they erupt at all.
- In US adults aged 65 and over, 17.6% have no natural teeth (2011-2014): 13.9% at 65 to 74 and 23.0% at 75 and over. Rates have fallen sharply over decades.

### Cited Findings
- About 20 primary teeth, starting to erupt at about 6 months. All 32 permanent teeth usually in by 21 y. Lower central incisors come first, at 6 to 12 months — [ADA MouthHealthy eruption charts](https://www.mouthhealthy.org/all-topics-a-z/eruption-charts) [snippet].
- Edentulism, NHANES 2011-2014: 17.6% of adults aged 65+, 13.9% at 65-74, 23.0% at 75+. Non-Hispanic Black 27.0%, non-Hispanic white 16.2%, Asian 18.0%, Hispanic 16.4% — [CDC MMWR QuickStats 66(3)](https://www.cdc.gov/mmwr/volumes/66/wr/mm6603a12.htm) [snippet]. Among adults 50+, edentulism fell from 17% (1999-2004) to 11% (2009-2014). For ages 65 to 74 it has fallen more than 75% over five decades — [PMC6394416](https://pmc.ncbi.nlm.nih.gov/articles/PMC6394416) [snippet].

**Eruption table** [inference: background, unverified. These are the standard ADA chart values from memory; the ADA page was found but not read]. Ages in months (primary) or years (permanent), upper / lower:

| tooth | primary erupts (months) | primary sheds (y) | permanent erupts (y) |
|---|---|---|---|
| central incisor | 8-12 / 6-10 | 6-7 / 6-7 | 7-8 / 6-7 |
| lateral incisor | 9-13 / 10-16 | 7-8 / 7-8 | 8-9 / 7-8 |
| canine | 16-22 / 17-23 | 10-12 / 9-12 | 11-12 / 9-10 |
| first molar (primary) / first premolar (permanent) | 13-19 / 14-18 | 9-11 / 9-11 | 10-11 / 10-12 |
| second molar (primary) / second premolar (permanent) | 25-33 / 23-31 | 10-12 / 10-12 | 10-12 / 11-12 |
| first molar (permanent) | – | – | 6-7 / 6-7 |
| second molar (permanent) | – | – | 12-13 / 11-13 |
| third molar | – | – | 17-21 / 17-21 |

### Inferences
- **Mouth-interior asset states** (pick by age, slider inside the stage) [inference]:

| state | age | content |
|---|---|---|
| gums | 0 to 0.5 y | no teeth |
| primary partial | 0.5 to 2.5 y | lower then upper incisors, then first molars, canines, second molars |
| primary full | 2.5 to 6 y | 20 teeth |
| mixed | 6 to 12 y | gaps: front-tooth gap gag at 6 to 8 y; large incisors with small primary canines |
| permanent | 12 to 18 y | 28 teeth (+4 third molars from about 18 y, optional) |
| adult | 18 to 60 y | 28 to 32 |
| old | 60+ | per profile: full / partial loss / dentures / edentulous. Default probability of no teeth: 14% at 65-74, 23% at 75+ (US). Add tooth wear, gum recession and yellowing. |

- With no teeth, the lower face height shrinks (the chin and nose come closer; the lips fold in). This is a known clinical effect, but the amount was not sourced; a guess is −10 to 15% of lower face height (sn-gn) without dentures.

### Gaps
- The ADA chart values were not read directly. Check them against the ADA PDF or AAPD tables.
- No data on mean remaining teeth by age, or on partial tooth loss patterns.

---

## 6. Hair by age

### Takeaway
- **The "50-50-50" rule is false.** Panhard et al. 2012 (4,192 people worldwide) found only **6 to 23%** of people aged 50 have at least 50% grey hair, depending on ethnicity and natural hair colour. Between 45 and 65 y, 74% have some grey, with a mean grey intensity of 27%. Men grey more than women. Asian and African groups grey less than European groups at the same age.
- **Male pattern hair loss** rises roughly with age in decades: about 30% at 30 y, 50% at 50 y and 80% at 70 y in Caucasian men. In Gan and Sinclair (Australia), mid-frontal loss affected 73.5% of men aged 80+.
- **Female pattern hair loss** rises from 3 to 6% under 30 y to 29 to 42% at 70+ (UK/US), and to 57% at 80+ (Gan and Sinclair).
- Men's eyebrows whiten more than women's. Brow position in men does not drop with age in those without frank ptosis. Ear and nose hair increase in older men; the mechanism is unclear.

### Cited Findings
- Panhard, Lozano and Loussouarn 2012, *Br J Dermatol*, "Greying of the human hair: a worldwide survey, revisiting the '50' rule of thumb": 4,192 volunteers. 6 to 23% have ≥ 50% grey coverage at 50 y. Between 45 and 65 y, 74% have grey hair with a mean intensity of 27%. Men greyer than women. Asian and African groups less grey than European groups — [Cosmetics Design Europe](https://www.cosmeticsdesign-europe.com/Article/2012/10/02/50-shades-of-grey-Not-quite-as-L-Oreal-study-finds-grey-hair-less-common-than-previously-thought), [Donovan Medical](https://donovanmedical.com/hair-blog/50-50-50-rule-revisited) [snippet].
- Male androgenetic alopecia: about 30% of Caucasians by 30 y, 50% by 50 y and 80% by 70 y — [Nigerian J Dermatology](https://www.nigjdermatology.com/index.php/NJD/article/download/159/123/418) [snippet]. Nigerian men: 50.4% at 18-29 and 76.1% at 50-59, but mostly mild grades; grades III to VII were 20.3% overall — same source [snippet]. Norwood 1975 classified 1,000 balding men by type and age of onset; all types become more common with age — [ISHRS](https://ishrs.org/male-and-female-pattern-hair-loss-are-they-predictable/) [snippet]. A US survey (as cited in a review) gives moderate or severe loss in 53% of men aged 40 to 49 — [Endotext chapter](https://www.endotext.org/wp-content/uploads/word/male-androgenetic-alopecia.docx) [snippet]. A clinic site gives 16% of men aged 18 to 29 with some loss; this is marketing, not a primary source [snippet].
- Gan and Sinclair 2005, *J Investig Dermatol Symp Proc* 10:184-189 (Maryborough, Australia): mid-frontal hair loss rises with age and affects 57% of women and 73.5% of men aged 80+ — [Epworth repository](https://knowledgebank.epworth.org.au/epworthjspui/handle/11434/552?mode=full) [snippet].
- Female pattern hair loss: UK and US data show 3 to 6% of women under 30 rising to 29 to 42% at 70+ — [Primary Care Notebook](https://primarycarenotebook.com/en-AU/pages/dermatology/female-pattern-hair-loss-fphl) [snippet]. Taiwan (26,226 women aged 30+): Ludwig grade > I in 11.8%, rising with age — [TMU](https://hub.tmu.edu.tw/en/publications/factors-associated-with-female-pattern-hair-loss-and-its-prevalen/) [snippet].
- Eyebrows: in 1,545 patients, men showed more eyebrow whitening than women, but most people had none. In men without brow ptosis, brow position relative to the pupil and lateral canthus does not fall with age — [PMC3907512](https://pmc.ncbi.nlm.nih.gov/articles/PMC3907512/), [Science Focus](https://www.sciencefocus.com/the-human-body/why-dont-eyebrows-go-grey-at-the-same-time-as-head-hair/) [snippet]. Ear-canal and nasal hair growth increases in older men; research on the mechanism is scarce — [Science Focus](https://www.sciencefocus.com/the-human-body/prominent-nasal-hair-with-age) [snippet].

**Recalled values** [inference: background, unverified]:
- Norwood 2001 (*Dermatol Surg*, 1,006 women) reported female pattern loss in about 12% by 29 y, 25% by 49 y, 41% by 69 y and more than 50% by 79 y. The user's figures of 12% and 25% match this, not Gan and Sinclair. This could not be confirmed this session.
- Typical greying onset: Caucasians mid-30s, Asians late 30s, Africans mid-40s. Premature greying is usually defined as before 20 y (Caucasian) or before 30 y (African).
- Newborns: lanugo is mostly shed before or shortly after birth. Scalp hair at birth varies a lot, and many infants lose some scalp hair at 2 to 4 months. Pubic hair starts around 10 to 12 y (girls) and 11 to 13 y (boys). Axillary hair follows about 1 to 2 y later, and facial hair in boys about 2 to 3 y after pubic hair. These are Tanner-stage norms.

### Inferences
- **Greying model** (code-ready) [inference]: `grey_fraction(age) = sigmoid((age − onset) / spread)`, scaled so the population median gives about 27% at 55 y for Europeans. Use onset ~ N(34, 6) y for European looks, N(38, 6) for East Asian and N(44, 6) for African, with spread about 8 y. Calibration check: P(grey ≥ 50% at 50 y) should be about 0.06 to 0.23 depending on group (Panhard). Order of greying: temples, then crown, then the rest; beard and eyebrows later than scalp. Men about +3 to 5 y "greyer" than women.
- **Hairline model** [inference]: store a Norwood class (I to VII) or Ludwig class (I to III) as a stage-dependent random variable in the character profile. Defaults for men: P(≥ III) ≈ 0.15 at 25 y, 0.30 at 35 y, 0.45 at 50 y, 0.55 at 65 y, 0.65 at 80 y. These are guesses consistent with "30% at 30, 50% at 50, 80% at 70" for any loss. Women use Ludwig ≥ I at 0.05 to 0.10 at 30, 0.20 to 0.25 at 50 and 0.40 to 0.55 at 75+.
- **Brows and body hair**: male eyebrows lengthen and roughen after about 50 (hair-card length +20 to 50%, guessed). Ear and nose hair become visible from about 50 in men. Eyebrow greying lags scalp greying.

### Gaps
- Panhard's per-group curves (grey % by age) were not read; only the headline numbers.
- No primary decade-by-decade Norwood prevalence table was read (the Rhodes 1998 claim could not be confirmed).
- No infant scalp-hair length or density data.

---

## 7. Which head-module parameters change with age, and by how much

### Takeaway
- Treat every head parameter as `value(age) = identity_adult_value × M_param(age) × style_gain(stage)`. M is a realistic multiplier taken from the data above (1.0 at 20 to 30 y). style_gain is the anime exaggeration per stage and is overridden by artist reference images when the user picks them.
- **Identity stays fixed**: eye shape, iris colour, eyebrow shape, nose shape class, mouth corner shape, ear shape, scars, any designed asymmetry, hair colour (before greying) and hairline shape (before recession). These are stored once in the character profile and scaled, never re-solved.
- **Age changes**: head size relative to height, vertical layout (eye line, nasion, mouth line), face height, face and jaw width relative to the head, nose length and width, mouth width, chin and mandible height, ear size, cheek fat, lip vermilion, wrinkles, teeth state, hair colour, hairline and density.

### Cited Findings
All numbers trace to Sections 1 to 6. The main fetched inputs are the Snyder head-unit ratios and the UK90 head circumference ratios below.

**Snyder 1977 in head-radius units (r = HC / 2π), boys / girls** [fetched; computed]:

| age y | r (mm) | head height / r | n-gn / r | bizygomatic / r | nose length / r | mouth breadth / r | head breadth / r |
|---|---|---|---|---|---|---|---|
| 2-3 | 79.7 / 77.7 | 2.23 / 2.20 | 1.03 / 1.05 | 1.35 / 1.39 | 0.375 / 0.378 | 0.404 / 0.418 | 1.71 / 1.71 |
| 4-5 | 80.9 / 79.4 | 2.24 / 2.22 | 1.07 / 1.08 | 1.39 / 1.41 | 0.412 / 0.405 | 0.434 / 0.441 | 1.71 / 1.71 |
| 6-7 | 82.4 / 80.9 | 2.27 / 2.28 | 1.11 / 1.11 | 1.44 / 1.44 | 0.438 / 0.438 | 0.459 / 0.460 | 1.72 / 1.71 |
| 8-9 | 83.9 / 82.6 | 2.30 / 2.27 | 1.15 / 1.15 | 1.43 / 1.46 | 0.460 / 0.470 | 0.470 / 0.480 | 1.71 / 1.70 |
| 10-11 | 84.7 / 84.1 | 2.33 / 2.33 | 1.18 / 1.17 | 1.45 / 1.46 | 0.494 / 0.486 | 0.475 / 0.484 | 1.72 / 1.71 |
| 12-13 | 85.9 / 85.0 | 2.37 / 2.34 | 1.20 / 1.21 | 1.47 / 1.48 | 0.512 / 0.525 | 0.484 / 0.491 | 1.71 / 1.71 |
| 14-15 | 87.5 / 85.9 | 2.40 / 2.36 | 1.25 / 1.24 | 1.49 / 1.51 | 0.550 / 0.531 | 0.507 / 0.511 | 1.71 / 1.71 |
| 16-17 | 90.2 / 86.6 | 2.43 / 2.36 | 1.27 / 1.25 | 1.52 / 1.51 | 0.552 / 0.521 | 0.512 / 0.521 | 1.71 / 1.71 |
| 18-20 | 91.0 / 86.6 | 2.45 / 2.33 | 1.30 / 1.24 | 1.52 / 1.52 | 0.570 / 0.533 | 0.526 / 0.542 | 1.70 / 1.72 |

Head breadth / r is constant at 1.71, so the cranium keeps its shape. Everything in the face grows faster than r.

### Inferences
**Per-stage multiplier table M(age)** (adult 20 to 30 y = 1.00; sexes pooled unless noted). Values from 2 to 18 y come from the Snyder table above, rounded. Values at 0 and 1 y are extrapolations [inference]. Values from 50 y on come from Section 3 rules [inference]. Head-relative units: horizontal sizes ÷ r, vertical positions ÷ head height measured from the vertex.

| param (head units) | 0 y | 1 y | 3 y | 6 y | 10 y | 14 y | 18 y | 30 y | 50 y | 70 y | 85 y |
|---|---|---|---|---|---|---|---|---|---|---|---|
| headScale (HC / height ÷ adult) | 2.14 | 1.96 | 1.67 | 1.42 | 1.22 | 1.07 | 1.00 | 1.00 | 1.00 | 1.02 | 1.03 |
| eyeSize (eye width / r) | 1.08 | 1.07 | 1.05 | 1.03 | 1.02 | 1.01 | 1.00 | 1.00 | 0.99 | 0.97 | 0.96 |
| eyeOpenHeight (aperture height) | 1.05 | 1.05 | 1.03 | 1.02 | 1.01 | 1.00 | 1.00 | 1.00 | 0.95 | 0.88 | 0.85 |
| eyeSpacing (IPD / r) | 0.90 | 0.89 | 0.89 | 0.92 | 0.95 | 0.98 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| eyeHeight (eye-line depth from vertex / head height; absolute, not a multiplier) | 0.61 | 0.60 | 0.58 | 0.56 | 0.54 | 0.52 | 0.51 | 0.50 to 0.51 | 0.51 | 0.51 | 0.51 |
| faceHeight (n-gn / r) | 0.70 | 0.76 | 0.81 | 0.87 | 0.92 | 0.97 | 0.99 | 1.00 | 1.00 | 0.98 | 0.95 (0.88 with no teeth) |
| mouthHeight (stomion depth from nasion / n-gn) | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.01 | 1.02 | 1.03 |
| faceWidth (bizygomatic / r) | 0.84 | 0.86 | 0.90 | 0.95 | 0.96 | 0.98 | 1.00 | 1.00 | 1.00 | 0.99 | 0.98 |
| jawWidth (bigonial / r) | 0.82 | 0.85 | 0.89 | 0.93 | 0.95 | 0.97 | 1.00 | 1.00 | 1.00 | 0.98 | 0.97 |
| chin (mandible height / r) | 0.62 | 0.68 | 0.76 | 0.84 | 0.90 | 0.96 | 0.99 | 1.00 | 0.99 | 0.96 | 0.93 (0.85 with no teeth) |
| noseLength (nose length / r) | 0.55 | 0.60 | 0.68 | 0.78 | 0.87 | 0.96 | 1.00 | 1.00 | 1.03 | 1.06 | 1.08 |
| noseWidth (al-al / r) | 0.75 | 0.78 | 0.82 | 0.87 | 0.91 | 0.96 | 1.00 | 1.00 | 1.02 | 1.04 | 1.05 |
| mouthWidth (ch-ch / r) | 0.66 | 0.70 | 0.78 | 0.86 | 0.90 | 0.96 | 1.00 | 1.00 | 1.01 | 1.02 | 1.02 |
| upperVermilion (height) | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 0.90 | 0.75 | 0.65 |
| earLength (/ r) | 0.70 | 0.78 | 0.85 | 0.90 | 0.94 | 0.98 | 1.00 | 1.00 | 1.07 | 1.14 | 1.19 |
| cheekFat (malar volume) | 1.40 | 1.40 | 1.25 | 1.10 | 1.05 | 1.00 | 1.00 | 1.00 | 0.90 | 0.75 | 0.65 |
| orbitAperture (area) | – | – | – | – | – | – | – | 1.00 | 1.04 | 1.07 | 1.09 |

How the rows were derived:
- headScale: from UK90 HC / height (0.690, 0.632, 0.539, 0.459, 0.394, 0.344, 0.323 for boys) ÷ 0.323 [fetched]. (Corrected in verification: the row was 2.10, 1.93, 1.65, 1.40, 1.20, 1.05, which is 1 to 2% below the stated division; recomputed as 2.14, 1.96, 1.67, 1.42, 1.22, 1.07.) The old-age rise follows from stature loss with constant head size [inference].
- eyeSize and eyeSpacing: from the eyeball, palpebral-fissure and IPD paragraphs in Section 1. eyeOpenHeight in old age reflects a lower lid aperture from brow and lid ptosis (guessed amount).
- eyeHeight: Section 1 inference (nasion depth + 0.045). It is a **position**, so the module should store it absolutely, not as a multiplier.
- faceHeight, faceWidth, noseLength and mouthWidth: from Snyder head-unit ratios ÷ the 18 to 20 y value, then smoothed. The 0 and 1 y values extend the trend and use Farkas 1992 (mandible height 67% at 1 y vs head at 83% ⇒ chin ≈ 0.68 relative to r at 1 y).
- jawWidth: the two sources disagree. Farkas 1992 (mandible width 80% of adult at 1 y, head circumference 83%) implies about 0.96 in head units at 1 y. Snyder bizygomatic / r is only 0.89 at 2 to 3 y. I used 0.85 at 1 y as a compromise guess. Treat it as uncertain by ±0.08 until the Farkas go-go table is read.
- noseWidth and earLength: no age data fetched; values are interpolated guesses. The adult rise of earLength uses Heathcote: +0.22 mm/y on a ~62 mm ear.
- noseLength after 30: +1.2 to 1.4 mm per 12 y on a ~51 mm nose.

**Rules for the module** [inference]:
1. Stage and slider: stages baby 0 to 2, child 2 to 12, teen 12 to 18, adult 18 to 60, old 60+. Within a stage, interpolate M linearly in age between the table columns (use a piecewise-cubic monotone spline if the slider looks jerky).
2. Identity: solve the character's adult values from the blueprint once (`measure2d.py`). For a younger blueprint, divide by M(age_of_blueprint) to get the adult identity values. This lets a single child blueprint make an adult and vice versa.
3. Override: if the user supplies a manga image for a given age, measure it and store per-parameter overrides `value_override(age)`. Blend: `value = override` at that age, fading back to `identity × M × style_gain` over ±1 stage.
4. Anime style gain per stage (start values; replace with measurements of One Piece references): eyeSize × (1.5 baby, 1.4 child, 1.25 teen, 1.15 adult, 1.1 old); noseLength × (0.5, 0.6, 0.75, 0.9, 1.0); mouthWidth × (0.8, 0.85, 0.9, 1.0, 1.0); eyeHeight absolute + (0.03, 0.02, 0.01, 0, 0). One Piece varies by character; the profile can carry a style gain per character.
5. Sex: apply Snyder per-sex ratios from 12 y. From 14 y, male n-gn, nose length and chin rise 4 to 7% above female in head units. Brow ridge: boys only, from 13 y, with an unsourced default of +2 to 4 mm forward projection by 18 y.
6. Babyschema dial: an optional `babySchema` scalar (−2 to +2 SD, Glocker) on top of everything, for cute or not-cute variants at the same age.

### Gaps
- The 0 to 1 y face values, nose width, ear length and lip heights by age are extrapolations or guesses. Farkas Appendix A would replace them.
- Old-age bone changes are directionally sourced but not numerically.
- The anime style gains are unvalidated.

---

## How this plugs into HeroBody

- **Anime head module (Phase 2)**. Add an `age_curves` table (the Section 7 M table, sex-specific where Snyder gives it) as data, e.g. `hb/data/head_age_curves.json`. The head module reads `profile.age`, `profile.sex`, the identity values and the style gains, and outputs the head knobs in head-radius units: eyeSize, eyeSpacing, eyeHeight, mouthHeight, jawWidth, chin, noseLength, noseWidth, mouthWidth, earLength, cheekFat, upperVermilion. The drawn-feature decals scale with eyeSize, noseLength and mouthWidth. The mouth rings scale with mouthWidth and upperVermilion. The interior (teeth, tongue, eyeballs) scales with head size, and the eyeball uses its own curve (axial 16.8 mm at birth → 23.6 mm adult).
- **Knob solver (height applied last)**. headScale gives the head-size target as HC / height per age (UK90 table). The head-height target is in `proportions_by_age.md` (heads tall). The new "head bigger than 2×" blendshape is needed only for anime babies and chibi. A real newborn has head ratio 2.1× the adult HC/height, which is in range.
- **Blueprint approval checks**. Add per-age plausibility checks to `hb/measure2d.py` output: the eye line between 0.48 and 0.64 of head height, n-gn / head height within ±0.05 of the age value, and nose length / head height within ±30% of the age value (wide, for anime). Flag but do not block, since anime can go outside.
- **Character profile**. New fields: `teeth_state` (enum from Section 5, default sampled from the age), `grey_onset_age`, `grey_spread`, `hairline_class` (Norwood or Ludwig with an age), `brow_ageing` (bool), `wrinkle_onset_age` per region (forehead, crow's feet, glabella, nasolabial, perioral), `babySchema`. The human only verifies these values.
- **Hair system (scalp cap + clumps)**. Add per-strand `melanin` that is reduced by `grey_fraction(age, region)` with the order temples → crown → rest. Recession masks per Norwood or Ludwig class on the scalp cap. Clump density × (1 − loss fraction).
- **Mouth interior asset**. Make 6 tooth states (gums, primary partial, primary full, mixed, permanent, old variants with partial loss and dentures). Choose by age with slider-driven tooth visibility. When there are no teeth, apply the lower-face collapse multipliers (faceHeight 0.88 and chin 0.85 at 85 y).
- **Fat layer build item**. The cheekFat row ties the face to the fat layer: baby buccal fat at 1.4× and old-age malar loss at 0.65×. Use compartment masks following Rohrich and Pessa (nasolabial, medial, middle and lateral malar, forehead, jowl, submental).
- **Anatomy (BodyParts3D skull)**. For old age, deform the skull with a few CT-based morph targets: orbital rim inferolateral out, maxillary angle −10°, mandible length and height −3 to 6%, gonial angle +3 to 6°. Then let the per-body clamp keep the anatomy inside the skin.

## Sources

Fetched (primary data files):
- WHO Child Growth Standards head circumference and length tables: https://raw.githubusercontent.com/WorldHealthOrganization/anthro/master/data-raw/growthstandards/hcanthro.txt and https://raw.githubusercontent.com/WorldHealthOrganization/anthro/master/data-raw/growthstandards/lenanthro.txt (package licence GPL-3; https://github.com/WorldHealthOrganization/anthro)
- UK 1990 growth reference LMS (sitar package data): https://raw.githubusercontent.com/cran/sitar/master/data/uk90.rda
- Snyder et al. 1977 child anthropometry (NIST AnthroKids export copy): https://raw.githubusercontent.com/solcalloni/prediccion-talle-zapatos/main/chicos.csv

Snippet only:
- https://epos.myesr.org/poster/esr/ecr2016/C-1449/background
- https://jcdr.net/ReadXMLFile.aspx?id=3473
- https://reviewofmm.com/whats-normal-whats-not-emmetropisation-and-normal-ocular-growth-in-caucasian-and-asian-children/
- https://www.eophtha.com/posts/normal-values-in-ophthalmology
- https://eyewiki.org/Dysmorphology_of_the_Eye_and_Periorbital_Region
- https://pubmed.ncbi.nlm.nih.gov/1707200/
- https://depts.washington.edu/fasdpn/pdfs/pfl2012.pdf
- https://commons.pacificu.edu/works/publication-dissertation/cmjf8-q9b78
- https://arxiv.org/pdf/2604.15328
- https://www.acmgmeeting.net/conference-program/assessment-interpupillary-distance-racially-ethnically-diverse-pediatric-sample-electronic-health-record-data
- https://digi.ub.uni-heidelberg.de/diglit/rimmer1864/0048
- https://www.mybluprint.com/article/childs-face-mastering-proportions
- https://eprints.whiterose.ac.uk/185518/1/polargrowth-v7.pdf
- https://research.bond.edu.au/en/publications/further-experiments-on-the-perception-of-growth-in-three-dimensio/
- https://onlinelibrary.wiley.com/doi/10.1002/ajmg.1320650102
- https://archive.org/details/anthropometryofh0000unse
- https://ncbi.nlm.nih.gov/entrez/query.fcgi?cmd=Retrieve&db=PubMed&dopt=Abstract&list_uids=16077306
- https://www.researchgate.net/figure/Normal-Range-of-Measurements-of-North-American-White-Young-Adult-Population_tbl1_7682966
- https://arxiv.org/pdf/2008.06989
- https://pubmed.ncbi.nlm.nih.gov/1643058/
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8156684/
- https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6384287/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC3404279
- https://academic.oup.com/asj/article/46/4/366/8200963
- https://academic.oup.com/asj/article/43/4/420/6967154
- https://tdr.lib.ntu.edu.tw/handle/123456789/15894
- https://link.springer.com/article/10.1007/s00266-012-9904-3
- https://www.healthday.com/healthpro-news/cosmetic/bony-elements-in-the-mid-face-change-over-time-602251.html
- https://sciencedaily.com/releases/2010/03/100323121836.htm
- https://www.drbicuspid.com/clinical/imaging-cad-cam/3d-printing/article/15359264/jaw-bone-plays-key-role-in-appearance-over-time
- https://utsouthwestern.elsevierpure.com/en/publications/the-fat-compartments-of-the-face-anatomy-and-clinical-implication
- https://surgery.healthsci.mcmaster.ca/wp-content/uploads/2023/05/2007_facial_fat_pads_anatomy.pdf
- https://www.nbcnews.com/health/health-news/cheeky-study-finds-beauty-secret-cadavers-flna1C9462204
- https://jhanley.biostat.mcgill.ca/bios601/Surveys/BigEars.pdf
- https://www.discovermagazine.com/health/british-ears
- https://pmc.ncbi.nlm.nih.gov/articles/PMC10933809
- https://www.springermedicine.com/age-related-changes-in-the-external-noses-of-the-anatolian-men/20766398
- https://pubmed.ncbi.nlm.nih.gov/35994354/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC7984336/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC7875215
- https://pubmed.ncbi.nlm.nih.gov/24267416/
- https://bibliotheek.ehb.be:2400/topics/en-us/859/diagnosis-approach
- https://www.medestheticsmag.com/news/news/22864571/evelab-insight-announces-preliminary-findings-of-skin-aging-with-aidriven-research
- https://repository.upenn.edu/cog_neuro_pubs/11
- https://figshare.com/articles/dataset/_Objective_measures_of_baby_schema_cf_Glocker_et_al_2_/1368417/1
- https://public-pages-files-2025.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2014.00411/pdf
- https://www.pnas.org/doi/10.1073/pnas.0811620106
- https://pmc.ncbi.nlm.nih.gov/articles/PMC4019884/
- https://www.mdpi.com/2073-8994/11/5/664
- https://tips.clip-studio.com/en-us/articles/7059
- https://animeartmagazine.com/how-to-represent-different-ages-in-anime-women/
- https://tvtropes.org/pmwiki/pmwiki.php/Main/AnimationAnatomyAging
- https://www.mouthhealthy.org/all-topics-a-z/eruption-charts
- https://www.cdc.gov/mmwr/volumes/66/wr/mm6603a12.htm
- https://pmc.ncbi.nlm.nih.gov/articles/PMC6394416
- https://www.cosmeticsdesign-europe.com/Article/2012/10/02/50-shades-of-grey-Not-quite-as-L-Oreal-study-finds-grey-hair-less-common-than-previously-thought
- https://donovanmedical.com/hair-blog/50-50-50-rule-revisited
- https://www.nigjdermatology.com/index.php/NJD/article/download/159/123/418
- https://ishrs.org/male-and-female-pattern-hair-loss-are-they-predictable/
- https://www.endotext.org/wp-content/uploads/word/male-androgenetic-alopecia.docx
- https://knowledgebank.epworth.org.au/epworthjspui/handle/11434/552?mode=full
- https://primarycarenotebook.com/en-AU/pages/dermatology/female-pattern-hair-loss-fphl
- https://hub.tmu.edu.tw/en/publications/factors-associated-with-female-pattern-hair-loss-and-its-prevalen/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC3907512/
- https://www.sciencefocus.com/the-human-body/why-dont-eyebrows-go-grey-at-the-same-time-as-head-hair/
- https://www.sciencefocus.com/the-human-body/prominent-nasal-hair-with-age

## Verification

Adversarial check, 2026-10-09. Access in this pass: raw.githubusercontent.com could be reached and the three data files were re-downloaded and re-parsed with Python (`rdata` for `uk90.rda`). WebFetch failed with DNS errors, and the proxy refused CONNECT (403) for pubmed, eutils, cdc.gov, mdpi.com, academic.oup.com, europepmc/ebi and jhanley.biostat.mcgill.ca. The shared WebSearch budget for this turn was already used up, so no new snippets could be fetched. Every claim that rests only on a non-GitHub source is therefore marked **unverifiable**. That does not mean it is wrong; the note gives a plausibility judgement from background knowledge [inference].

| claim | verdict | note | source |
|---|---|---|---|
| UK90 boys median HC 35.2 / 47.7 / 51.5 / 54.5 / 57.3 cm at 0 / 1 / 3 / 10 / 18 y; HC/height 0.690 / 0.632 / 0.539 / 0.394 / 0.323 | confirmed [fetched] | Re-parsed: M.head 35.204, 47.696, 51.471, 54.482, 57.26; M.ht 51.04, 75.51, 95.41, 138.39, 177.09, giving the same ratios. The birth "height" is supine length. Girls' HC ends at 17 y (NaN at 18), as the notes say. HC fractions of adult (61.5 / 83 / 90 / 95%) also check out. | https://raw.githubusercontent.com/cran/sitar/master/data/uk90.rda |
| headScale row of the Section 7 M table (HC/height ÷ 0.323) | corrected [fetched + arithmetic] | The table said 2.10 / 1.93 / 1.65 / 1.40 / 1.20 / 1.05. The stated division gives 2.14 / 1.96 / 1.67 / 1.42 / 1.22 / 1.07. Fixed inline. | uk90.rda |
| WHO 2006 median HC 34.46/33.88 (0), 46.06/44.89 (1 y), 50.74/49.92 (5 y) cm, boys/girls | confirmed [fetched] | hcanthro.txt day 0: 34.4618 / 33.8787; day 365: 46.0637 / 44.894; day 1826: 50.7372 / 49.9225. The boys' length at 0 (49.88) and 1 y (75.74) and height at 5 y (109.96) also match lenanthro.txt. Licence: the R package is GPL-3, while the standards themselves are WHO material. | https://raw.githubusercontent.com/WorldHealthOrganization/anthro/master/data-raw/growthstandards/hcanthro.txt |
| Snyder 1977: nasion depth / head height 0.534 → 0.452 (boys), 0.526 → 0.471 (girls); nose/r 0.375 → 0.570; bizygomatic/r 1.35 → 1.52; head breadth/r ≈ 1.71 | confirmed [fetched], with a binning caveat | Reproduced exactly (0.534, 0.452, 0.526, 0.471, 0.375, 0.570, 1.351, 1.524, 1.70-1.72) using rounded-age bins ("2-3" = 1.5-3.49 y; "18-20" = 17.5 y and over, mostly 17.5-18.9 y). With strict [2, 4) / [18, 21) bins the values move by ≤ 0.006 (for example nasion 0.531 / 0.456, nose/r 0.381 / 0.572, bizygomatic/r 1.37 / 1.53) and the adult cells shrink to 9-14 subjects. Added a note under the Snyder table. Sex code 1 = male (larger HC). The CSV is a third-party GitHub copy of the NIST AnthroKids export, not the NIST original. | https://raw.githubusercontent.com/solcalloni/prediccion-talle-zapatos/main/chicos.csv |
| Neurocranium:face volume 8-9:1 at birth, 5:1 at 2 y, 3:1 at 6 y, 2:1 adult; 2D ratio 4-4.5 → 1.5-2 | unverifiable | The EPOS host was blocked. The classic textbook figure is about 8:1 at birth and about 2-2.5:1 in adults, so the claim is plausible [inference]. It comes from an educational poster, not a primary study. | https://epos.myesr.org/poster/esr/ecr2016/C-1449/background |
| Eyeball axial length 16.8 mm at birth, +3.9 mm in 0-2 y, +1.2 mm in 2-5 y, 23.6 mm young adult | unverifiable | Host not reachable. The values agree with the standard literature (newborn about 16.5-17 mm, adult about 23.5-24 mm) [inference]. Note that 16.8 + 3.9 + 1.2 = 21.9 mm at 5 y, which leaves about 1.7 mm to add after 5 y. | https://reviewofmm.com/whats-normal-whats-not-emmetropisation-and-normal-ocular-growth-in-caucasian-and-asian-children/ |
| Child IPD (far) 46.5 / 47.5 / 49.5 / 51.0 mm at 12-23 / 24-35 / 36-47 / 48-59 months; newborn near 40.5 mm | unverifiable | Pacific University repository not reachable. This is a thesis (not peer reviewed) on a small Caucasian sample. The monotone trend is plausible. | https://commons.pacificu.edu/works/publication-dissertation/cmjf8-q9b78 |
| Farkas 1992 (n = 1,594, 1-18 y): mandible at 1 y is 80.2% of adult width and 66.6% of adult height; face matures at 12-15 y in boys, about 2 y earlier in girls | unverifiable | PubMed was blocked. The citation details (Cleft Palate Craniofac J 29:308-315) are consistent with background knowledge, but the 80.2 / 66.6 numbers could not be re-read. The jawWidth and chin M rows depend on them. | https://pubmed.ncbi.nlm.nih.gov/1643058/ |
| Ear length = 55.9 mm + 0.22 mm × age (Heathcote; slope CI 0.17-0.27) | unverifiable | The McGill PDF was blocked. The 0.22 mm/y slope and the BMJ 1995 Christmas-issue origin match background knowledge [inference]. The intercept 55.9 mm and the CI were not re-checked. The sample is people aged 30+, so do not extrapolate the line below 30 y. | https://jhanley.biostat.mcgill.ca/bios601/Surveys/BigEars.pdf |
| Longitudinal CT (96 adults, about 11 y): pyriform −5°, maxillary −11°, glabellar −6.5°, orbital aperture +91 mm² | unverifiable | OUP was blocked. The directions match Shaw and Kahn 2007 and Mendelson 2012. An 11° maxillary-angle change within about 11 y is large compared with the cross-sectional young-vs-old difference of about 10°, so the angle values may be cross-sectional or mis-summarised. Treat the numbers as provisional. | https://academic.oup.com/asj/article/46/4/366/8200963 |
| Shaw 2010 mandible (n = 120): angle increases, length decreases (young → middle), height decreases (middle → old), women earlier | unverifiable | ScienceDaily was not reachable. The claim matches background knowledge of Shaw et al., PRS 125:332 [inference]. It gives no magnitudes; the ±3-6% values in the M table are guesses. | https://sciencedaily.com/releases/2010/03/100323121836.htm |
| Glocker 2009 baby-schema indices (fw, fol/fal, ew/fw, nl/hl, nw/fw, mw/fw), ±2 SD | unverifiable | figshare was not reachable. The claim is consistent with background knowledge of Glocker et al. (Ethology 2009; PNAS 2009) [inference]. | https://figshare.com/articles/dataset/_Objective_measures_of_baby_schema_cf_Glocker_et_al_2_/1368417/1 |
| Japanese animation eye area 3.4× human; American cartoons 2× | unverifiable | MDPI was blocked. It is unclear whether "eye area" is normalised to face area or absolute; read the methods before using 3.4 as a style gain. Small sample (100 characters). | https://www.mdpi.com/2073-8994/11/5/664 |
| Panhard 2012 (n = 4,192): 6-23% have ≥ 50% grey at 50 y; 74% of 45-65 y have some grey, mean intensity 27% | unverifiable | The trade-press page was not reachable. The figures match background knowledge of Panhard et al., Br J Dermatol 167:865 [inference]. The source is industry-funded (L'Oréal). | https://www.cosmeticsdesign-europe.com/Article/2012/10/02/50-shades-of-grey-Not-quite-as-L-Oreal-study-finds-grey-hair-less-common-than-previously-thought |
| US edentulism 2011-2014: 17.6% at 65+, 13.9% at 65-74, 23.0% at 75+ | unverifiable | cdc.gov was blocked. The figures are consistent with background knowledge of the MMWR QuickStats [inference]. | https://www.cdc.gov/mmwr/volumes/66/wr/mm6603a12.htm |
| Infant eyes only 7-16% larger in head-radius units; toddler IPD/r 0.61 vs adult 0.69 | corrected [inference, arithmetic] | Eyeball: 16.8 / 56.0 = 0.300 vs 23.6 / 91.1 = 0.259 (×1.16) ✓. Palpebral fissure: 19.4 / 56.0 = 0.346 vs 30 / 91.1 = 0.329, which is ×1.05, not ×1.07. Range corrected inline to 5-16%. IPD: 46.5 / 75.9 = 0.613 ✓ and 63 / 91.1 = 0.691 ✓, but 46.5 mm is the 12-23-month mean. Using r at about 1.5 y (about 78 mm) gives about 0.60, so the conclusion still holds. The adult IPD of 63 mm and adult palpebral width of 30 mm are unverified inputs. | computed from uk90.rda + the snippets above |

Net effect on the plan: all the numbers that are fetched and go into code (UK90, WHO, Snyder ratios) hold. One table row (headScale) had arithmetic drift and is now fixed. Every snippet-only number (Farkas 1992, eye and IPD, CT ageing, ear slope, Panhard, CDC) is still unverified and should get a second fetch before it is hard-coded.
