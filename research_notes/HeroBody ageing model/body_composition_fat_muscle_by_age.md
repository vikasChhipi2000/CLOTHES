# Body composition and tissue layers by age: fat, muscle, bone, skin (for the HeroBody fat layer)

Research date: 2026-10-09. Researcher notes for the report writer.

**Access notes.** I re-tested access at the start of this session ("try reaching now"). WebFetch failed with `getaddrinfo ENOTFOUND` for pmc.ncbi.nlm.nih.gov and ajcn.nutrition.org. curl through the proxy returned no connection (HTTP 000) for pmc.ncbi.nlm.nih.gov, www.cdc.gov, wwwn.cdc.gov, ftp.cdc.gov, who.int, journals.plos.org, nature.com, academic.oup.com, cran.r-project.org, europepmc.org and the EBI REST API, api.semanticscholar.org, api.openalex.org, core.ac.uk, scholar.archive.org, zenodo.org, biorxiv.org, openaccess.thecvf.com, frontiersin.org, huggingface.co, cdn.jsdelivr.net, hologic.com, en.wikipedia.org, ismni.org, hit.is.tue.mpg.de and measurement-toolkit.org. Only github.com (git clone), raw.githubusercontent.com, gitlab.com and bitbucket.org answered. The `gh` API was refused (repository access not enabled), so I could not search GitHub code. So the primary data here come from **GitHub copies of published reference tables**, read with Python (`rdata` parser) in the scratchpad:

- **childsds** R package (github.com/cran/childsds, `data/*.ref.rda`): LMS/BCCG/BCPE parameter tables (age, mu, sigma, nu, tau) for: Ofenheimer 2020 adult DXA (LEAD cohort, Austria, 18 to 81 y: FMI, LMI, ALMI, android/gynoid and trunk/limb fat ratios, VAT); Kirk 2021 adult DXA (Australian ABC study, 18 to 83 y: %BF, total fat, total lean, appendicular lean, android fat, gynoid fat); Duran 2019 child DXA (built on NHANES 1999-2004 DXA, 8 to 19.5 y: %BF, FMI, LBMI by ethnicity); Schafmeyer 2022 leg DXA (NHANES, 8 to 20 y: leg fat and lean); WHO 2006 (0 to 5 y: triceps and subscapular skinfold, mid-upper-arm circumference); KiGGS (Germany, 0 to 18 y: triceps skinfold, skinfold sum, %BF); LIFE Child (Leipzig, 3 to 17 y: triceps, subscapular, suprailiac, biceps skinfolds; thigh circumference); UK 1990 (McCarthy BIA body fat, 5 to 18 y).
- **naver/anny** (github.com/naver/anny): the age-conditional Beta priors for the `weight` and `muscle` phenotype knobs in `data/shape_calibration/{boys,girls}.pth` (parsed without torch).
- **MarilynKeller/HIT** README and LICENSE (github.com/MarilynKeller/HIT).
- Stage heights, BMI and thigh/calf/upper-arm circumference ratios are taken from the sister note `proportions_by_age.md` (same folder), so this note is consistent with it.

Everything else comes from search-result snippets. Tags: **[fetched]** = I read the primary file myself (including reference tables read from a GitHub copy of the published parameters). **[computed]** = I computed the number from fetched data; the method is stated. **[snippet]** = search-result snippet only. **[inference]** = my reasoning or recalled textbook knowledge that I could not verify in this session; treat these as defaults to check. Medians are LMS/BCCG `mu` values. Ages are in years. FMI = fat mass / height² (kg/m²), LMI = lean mass / height², ALMI = appendicular lean mass / height². "Skinfold" = caliper double fold in mm.

---

## 1. Fat mass, lean mass and body fat percentage by age and sex across the lifespan

### Takeaway
- **Infancy is the fattest stage of childhood.** Body fat is about 13 to 15% at birth, rises fast to a peak of about 25 to 30% at 3 to 6 months, then falls. Median fat mass at 6 months is about 1.9 kg in both sexes. Skinfolds agree: WHO median triceps skinfold is 9.8 mm at 3 months and falls to 7.7 mm at 2 y.
- **Childhood minimum at about 5 to 7 y.** %BF is lowest (about 15 to 16% boys, 18 to 19% girls by BIA) around 5 to 7 y. The triceps + subscapular skinfold sum (KiGGS) is lowest at 5 to 6 y in boys (14.8 mm) and flat at about 17.2 mm from 2 to 6 y in girls.
- **Puberty splits the sexes.** Boys' %BF peaks at about 10 to 11 y and then falls to about 15% at 16 to 18 y while lean mass index climbs from 12.3 to 17.5 kg/m². Girls' %BF keeps rising to about 25% (BIA) or 36% (NHANES DXA) at 18 y, with lean mass index flattening at about 14 kg/m².
- **Adults gain fat and lose lean mass.** Median %BF (Australian DXA): men 16.5% at 20 y, 20.6% at 40 y, 23.1% at 60 y, 27.0% at 80 y; women 27.3%, 30.3%, 34.8%, 41.1%. Total lean mass peaks at about 40 to 50 y (men 66 kg, women 45.8 kg) and falls to 56.8 kg and 41.7 kg at 80 y.
- **Method matters by several % points.** NHANES DXA gives about 27 to 28% fat for 8- to 10-year-old boys, while BIA or skinfold references give 17 to 18%. Pick one family of references for HeroBody and use it everywhere. I recommend DXA for adults and skinfold/BIA for children, then normalize by volume (Section 5).

### Cited Findings

**Infants, 0 to 2 y**
- Fat mass "peaks near 6 postnatal months", after which fat mass growth slows relative to fat-free mass and %BF declines — [search result, review of infant body composition](https://www.nature.com/articles/ejcn2015117) [snippet; attribution to this exact review uncertain].
- An air-displacement plethysmography (ADP) study reported %BF rising from **13.3% at birth to 24.5% at 2 months and 31.2% at 4 months** — [search result, likely "Body composition from birth to 4.5 months in infants born to non-obese women"](https://www.nature.com/articles/pr2010136) [snippet; attribution uncertain].
- Multicenter Infant Body Composition Reference Study (MIBCRS; ADP 0 to 6 mo, deuterium dilution 3 to 24 mo): **median fat mass 1.9 kg in both sexes at 6 months; fat-free mass 5.6 kg (boys) vs 5.1 kg (girls)**; %FM rises early and then falls until 24 months; girls have higher %FM at all ages; %FM is roughly 20 to 23% at 3 and 24 months — [Murphy-Alford et al., AJCN 2023](https://ajcn.nutrition.org/article/S0002-9165(23)07687-6/fulltext), [PMC11537950](https://pmc.ncbi.nlm.nih.gov/articles/PMC11537950/) [snippet; the 3- and 24-month %FM values were garbled in the snippet].
- Fomon et al. 1982 "reference child", birth to 10 y (AJCN 35:1169) — [EPA HERO record](https://heronext.epa.gov/reference/184446) [snippet: citation only]. Values I recall but could NOT verify this session: boys %BF 13.7 (birth), about 25 (4 to 6 mo), 22.5 (12 mo), 14.6 (5 y), 13.7 (10 y); girls 14.9 (birth), about 26 (6 mo), 23.7 (12 mo), 16.7 (5 y), 19.4 (10 y) [inference: recalled, verify against the paper before use].

**WHO 2006 skinfolds and arm circumference, 0.25 to 5 y (medians, mm or cm) [fetched: childsds who.ref]**

| age y | triceps M | triceps F | subscap M | subscap F | MUAC M cm | MUAC F cm |
|---|---|---|---|---|---|---|
| 0.25 | 9.8 | 9.8 | 7.7 | 7.8 | 13.5 | 13.0 |
| 0.5 | 9.2 | 9.1 | 7.2 | 7.2 | 14.2 | 13.8 |
| 1 | 8.1 | 8.0 | 6.5 | 6.5 | 14.6 | 14.2 |
| 2 | 7.7 | 7.8 | 5.9 | 6.1 | 15.2 | 14.9 |
| 3 | 7.8 | 8.2 | 5.7 | 6.1 | 15.7 | 15.7 |
| 5 | 7.6 | 8.8 | 5.4 | 6.1 | 16.5 | 16.9 |

10th to 90th percentile at 1 y: triceps 6.3 to 10.4 mm (M), subscapular 5.2 to 8.2 mm (M) [computed from the BCCG parameters].

**Children and adolescents: %BF by age, three references (medians) [fetched: childsds duran_bf.ref (NHANES DXA, white), uk1990.ref bodyfat (McCarthy 2006 BIA), kiggs.ref bodyfat]**

| age y | DXA M | DXA F | BIA UK M | BIA UK F | KiGGS M | KiGGS F |
|---|---|---|---|---|---|---|
| 5 | – | – | 15.6 | 18.0 | – | – |
| 6 | – | – | 16.0 | 19.1 | – | – |
| 8 | 27.3 | 33.2 | 17.0 | 21.2 | 15.5 | 18.1 |
| 10 | 28.3 | 33.4 | 17.8 | 22.8 | 18.0 | 20.2 |
| 11 | 28.0 | 33.1 | 17.7 | 23.2 | 18.8 | 21.2 |
| 12 | 27.2 | 32.8 | 17.4 | 23.6 | 18.8 | 22.3 |
| 14 | 24.6 | 33.0 | 16.2 | 24.0 | 16.2 | 24.4 |
| 16 | 22.0 | 34.3 | 15.5 | 24.3 | 14.9 | 26.5 |
| 18 | 22.0 | 36.3 | 15.4 | 24.6 | 16.4 | 28.6 |
| 19 | 22.4 | 37.2 | 15.4 | 24.9 | – | – |

- Spread is wide: NHANES-DXA boys at 12 y P10/P90 = 19.5/40.9%; girls 24.5/44.0% [computed].
- **Fat and lean mass indices (NHANES DXA, white, medians, kg/m²) [fetched: duran_bf.ref]**: FMI boys 4.6 (8 y), 5.0 (10), 5.1 (12), 5.1 (14), 5.0 (16), 5.2 (18); girls 5.7, 6.1, 6.6, 7.1, 7.7, 8.4. LBMI boys 12.0, 12.3, 13.2, 14.8, 16.5, 17.5; girls 11.0, 12.0, 12.8, 13.4, 13.8, 14.1. So boys' pubertal change is almost all lean gain (+5.5 kg/m² from 10 to 18 y) with flat fat; girls add both (+2.3 FMI, +2.1 LBMI).
- Ogden et al. 2011 (NHANES 1999-2004 skinfold-based curves, 5 to 18 y): curves are similar for boys and girls until about 9 y; boys' %BF peaks at about 11 y, girls' keeps rising — [PubMed 21961617](https://pubmed.ncbi.nlm.nih.gov/21961617/) [snippet].
- A 2025 NHANES 2021-2023 analysis found that around adiposity rebound (BMI minimum, about 5 to 6 y) waist-to-height ratio keeps falling, so the BMI rise there is mainly lean tissue, not fat — [ASN news on the J Nutr paper](https://nutrition.org/study-challenges-decades-old-puzzle-about-childhood-body-fat/) [snippet].

**Adults 18 to 83 y: DXA medians (P10/P90 in brackets) [fetched: childsds kirk_bf.ref (Australia) and ofenheimer_bf.ref (Austria)]**

| age y | %BF M (Kirk) | %BF F (Kirk) | total fat kg M | total fat kg F | total lean kg M | total lean kg F | ALM kg M | ALM kg F |
|---|---|---|---|---|---|---|---|---|
| 20 | 16.5 (10.9/24.1) | 27.3 (19.2/36.5) | 13.2 | 17.4 | 64.3 | 45.2 | 30.0 | 19.8 |
| 30 | 18.9 | 28.6 | 15.5 | 18.7 | 65.3 | 45.4 | 30.1 | 19.8 |
| 40 | 20.6 | 30.3 | 17.5 | 20.3 | 65.9 | 45.8 | 30.0 | 19.6 |
| 50 | 21.7 | 32.4 | 18.7 | 22.2 | 66.0 | 45.4 | 29.6 | 19.1 |
| 60 | 23.1 | 34.8 | 19.6 | 24.3 | 63.8 | 44.2 | 28.0 | 18.3 |
| 70 | 25.0 | 37.7 | 20.5 | 26.8 | 60.1 | 42.9 | 25.9 | 17.3 |
| 80 | 27.0 (19.0/35.7) | 41.1 (32.6/48.3) | 21.4 | 29.6 | 56.8 | 41.7 | 24.2 | 16.2 |

| age y | FMI M | FMI F | LMI M | LMI F | ALMI M | ALMI F | VAT g M | VAT g F |
|---|---|---|---|---|---|---|---|---|
| 18 | 4.58 | 7.01 | 16.83 | 13.60 | 8.09 | 6.09 | 164 | 138 |
| 25 | 5.50 | 7.26 | 17.24 | 13.93 | 8.29 | 6.36 | 348 | 168 |
| 35 | 6.64 | 7.74 | 17.67 | 14.36 | 8.39 | 6.53 | 683 | 255 |
| 45 | 7.54 | 8.62 | 17.94 | 14.66 | 8.41 | 6.57 | 1088 | 408 |
| 55 | 8.27 | 9.80 | 18.12 | 14.82 | 8.40 | 6.52 | 1486 | 644 |
| 65 | 8.77 | 10.91 | 18.03 | 14.94 | 8.18 | 6.56 | 1796 | 896 |
| 75 | 9.05 | 11.72 | 17.70 | 15.06 | 7.89 | 6.56 | 2036 | 1057 |
| 80 | 9.14 | 12.13 | 17.53 | 15.12 | 7.75 | 6.62 | 2153 | 1123 |
(First table: Kirk 2021, ABC study. Second table: Ofenheimer 2020, LEAD cohort.)

- Kelly, Wilson & Heymsfield 2009 (PLoS ONE 4:e7038) give NHANES 1999-2004 DXA reference values for %fat, FMI and LMI, by sex and ethnicity, 8 to 85 y — [PMC2737140](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2737140/) [snippet: scope only; I could not read the tables]. The Duran 2019 and Schafmeyer 2022 child tables above are built on the same NHANES DXA scans.

### Inferences
- **Recommended %BF curve for HeroBody** (median, %; mixes the sources above) [inference]:

| age y | 0 | 0.25 | 0.5 | 1 | 2 | 3.5 | 6 | 8 | 10 | 12 | 14 | 16 | 18 | 25 | 45 | 65 | 80 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| M | 14 | 24 | 25 | 23 | 21 | 18 | 16 | 17 | 18 | 17.5 | 16 | 15.5 | 15.5 | 18 | 21 | 24 | 27 |
| F | 15 | 25 | 26 | 24 | 22 | 19 | 19 | 21 | 23 | 23.5 | 24 | 24.5 | 25 | 28 | 31.5 | 36 | 41 |

  Ages 0 to 3.5 use the infant ADP/Fomon shape; 5 to 18 use UK BIA (closest to skinfold references); adults use Kirk DXA. If you prefer DXA throughout, add about +8 to +10 points at 8 to 12 y (NHANES DXA vs BIA).
- %BF can be rebuilt from the index pair: %BF ≈ FMI / (FMI + LMI + BMCI), with BMCI (bone mineral / H²) ≈ 0.9 to 1.0 kg/m² in adults [inference]. Ofenheimer medians then give men 23% (25 y) → 33% (80 y), women 33% → 43%, a few points above Kirk because the Austrian cohort is heavier [computed].
- Use the LMS form for a "fatness" knob: x(z) = μ·(1 + ν·σ·z)^(1/ν), or μ·exp(σ·z) when ν = 0. A character's fatness z-score then gives FMI at every age and the body ages along its own centile [inference]. Note: BCPE tables (Duran %BF/FMI, LIFE skinfolds) also have τ; the BCCG formula gives the median exactly but only approximate centiles for them.

### Gaps
- Kelly 2009 tables, Fomon 1982 tables and Butte 2000 infant tables were not read (blocked). Infant numbers above are snippet or recalled.
- No adult reference in this note covers > 83 y. Childhood 2 to 5 y %BF has no direct reference here (only skinfolds).
- Ofenheimer (Austria) and Kirk (Australia) are white-majority cohorts; NHANES adds ethnic variation not tabulated here.

---

## 2. Regional fat distribution: subcutaneous thickness by site and age, visceral vs subcutaneous, gynoid/android, elderly limb loss vs central gain

### Takeaway
- **Children are "limb-fat" bodies.** From 3 to 10 y the subscapular (back) skinfold is only 0.57 to 0.63 of the triceps (arm) skinfold in both sexes. Infants have relatively thick trunk fat too (subscapular/triceps 0.78 at 0.5 y).
- **Puberty sets the adult pattern.** In boys the subscapular/triceps ratio climbs from 0.63 (10 y) to 1.08 (17 y) and the suprailiac/triceps ratio from 0.60 to 1.17: arm fat thins while trunk fat holds. In girls the ratios stay at 0.7 and 0.9 while every site thickens; triceps goes 13.4 → 18.3 mm from 10 to 17 y.
- **Adults centralize with age.** Trunk/limb fat ratio (DXA) rises in men from 0.88 (20 y) to 1.64 (60 y) and 1.75 (80 y); in women from 0.77 to 1.13 and 1.19. Android/gynoid fat (ratio of Kirk medians) rises from 0.36 to 0.70 (men) and 0.26 to 0.48 (women) between 20 and 80 y.
- **Visceral fat rises about 6-fold in men and 7-fold in women** from 25 to 80 y (median VAT 0.35 → 2.15 kg men, 0.17 → 1.12 kg women).
- **Old-age limb fat loss is relative before it is absolute.** Cross-sectional DXA medians keep limb fat roughly flat to 80 y, but lower-body subcutaneous fat falls relative to abdominal fat, and many older people lose subcutaneous fat when they lose weight. Intermuscular fat rises in all cases.
- **Skinfold ≠ ultrasound thickness.** Calipers compress fat. For a 3D layer, take site *ratios* and age *shapes* from skinfolds, then scale absolute thickness so the layer's volume equals subcutaneous fat mass / 0.9 g/cm³ (Section 6).

### Cited Findings

**LIFE Child (Leipzig) skinfold medians, 3 to 17 y, mm, with site ratios [fetched: childsds life_skinfold.ref; ratios computed]**

| age | M tri | M subs | M iliac | M biceps | M subs/tri | M iliac/tri | F tri | F subs | F iliac | F biceps | F subs/tri | F iliac/tri |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 3 | 10.3 | 5.9 | 4.8 | 5.9 | 0.57 | 0.47 | 10.0 | 6.1 | 5.6 | 6.0 | 0.60 | 0.56 |
| 4 | 9.6 | 5.5 | 4.6 | 5.5 | 0.58 | 0.48 | 9.7 | 5.9 | 5.8 | 5.9 | 0.60 | 0.59 |
| 5 | 9.2 | 5.4 | 4.5 | 5.3 | 0.58 | 0.49 | 9.6 | 5.7 | 5.9 | 5.7 | 0.60 | 0.62 |
| 6 | 9.0 | 5.4 | 4.7 | 5.2 | 0.60 | 0.51 | 9.6 | 5.8 | 6.2 | 5.7 | 0.60 | 0.64 |
| 7 | 9.3 | 5.6 | 5.0 | 5.3 | 0.61 | 0.54 | 10.1 | 6.1 | 6.7 | 6.0 | 0.61 | 0.66 |
| 8 | 9.9 | 6.1 | 5.6 | 5.6 | 0.61 | 0.56 | 11.1 | 6.7 | 7.5 | 6.6 | 0.61 | 0.68 |
| 9 | 10.9 | 6.8 | 6.3 | 6.1 | 0.62 | 0.58 | 12.3 | 7.5 | 8.7 | 7.4 | 0.61 | 0.71 |
| 10 | 12.1 | 7.7 | 7.3 | 6.7 | 0.63 | 0.60 | 13.4 | 8.3 | 9.9 | 8.1 | 0.62 | 0.74 |
| 11 | 13.2 | 8.6 | 8.3 | 7.3 | 0.65 | 0.63 | 14.3 | 9.2 | 11.2 | 8.6 | 0.65 | 0.78 |
| 12 | 13.8 | 9.4 | 9.3 | 7.7 | 0.68 | 0.67 | 15.2 | 10.2 | 12.6 | 9.0 | 0.67 | 0.83 |
| 13 | 13.6 | 10.1 | 10.1 | 7.5 | 0.74 | 0.74 | 16.1 | 11.2 | 14.0 | 9.4 | 0.70 | 0.87 |
| 14 | 12.8 | 10.5 | 10.7 | 7.0 | 0.82 | 0.84 | 16.9 | 11.9 | 15.1 | 9.7 | 0.71 | 0.89 |
| 15 | 11.8 | 10.7 | 11.1 | 6.3 | 0.90 | 0.94 | 17.6 | 12.4 | 15.7 | 9.8 | 0.71 | 0.89 |
| 16 | 10.9 | 10.8 | 11.4 | 5.6 | 1.00 | 1.05 | 18.0 | 12.7 | 15.9 | 9.6 | 0.70 | 0.88 |
| 17 | 10.1 | 10.9 | 11.8 | 5.1 | 1.08 | 1.17 | 18.3 | 12.7 | 16.0 | 9.4 | 0.69 | 0.87 |

**KiGGS (Germany) triceps skinfold and triceps+subscapular sum, medians, mm [fetched: kiggs.ref]**

| age y | 0.5 | 1 | 2 | 4 | 6 | 8 | 10 | 12 | 14 | 16 | 18 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| triceps M | 11.2 | 10.7 | 10.0 | 9.9 | 9.7 | 10.3 | 12.0 | 13.3 | 11.9 | 10.8 | 11.1 |
| triceps F | 10.8 | 10.9 | 10.9 | 11.0 | 11.3 | 12.3 | 14.0 | 15.3 | 17.0 | 19.0 | 20.8 |
| sum M | 18.8 | 17.5 | 16.1 | 15.4 | 14.9 | 15.8 | 18.7 | 21.2 | 19.9 | 19.5 | 21.1 |
| sum F | 18.5 | 17.9 | 17.2 | 17.3 | 17.4 | 18.9 | 21.8 | 24.5 | 27.8 | 31.4 | 34.7 |

KiGGS values run 1 to 2 mm above WHO and LIFE at the same age (different observers and populations). The skew is large: at 12 y the boys' triceps P90 is 24.6 mm vs median 13.3 [computed].

**Adult regional DXA ratios (medians) [fetched: ofenheimer_bf.ref fmrtl, fmrag; kirk_bf.ref afat, gf; ratios of medians computed]**

| age y | trunk/limb fat M | trunk/limb fat F | android/gynoid M (Ofenheimer) | android/gynoid F (Ofenheimer) | android fat kg M (Kirk) | gynoid fat kg M | android fat kg F | gynoid fat kg F |
|---|---|---|---|---|---|---|---|---|
| 20 | 0.88 | 0.77 | 0.35 | 0.29 | 0.89 | 2.47 | 0.97 | 3.71 |
| 30 | 1.12 | 0.81 | 0.48 | 0.31 | 1.22 | 2.85 | 1.12 | 3.88 |
| 40 | 1.33 | 0.87 | 0.60 | 0.35 | 1.52 | 3.03 | 1.29 | 4.05 |
| 50 | 1.49 | 0.99 | 0.71 | 0.41 | 1.73 | 3.01 | 1.49 | 4.22 |
| 60 | 1.64 | 1.13 | 0.80 | 0.50 | 1.92 | 2.97 | 1.73 | 4.41 |
| 70 | 1.73 | 1.20 | 0.85 | 0.54 | 2.07 | 3.01 | 2.00 | 4.60 |
| 80 | 1.75 | 1.19 | 0.88 | 0.56 | 2.17 | 3.11 | 2.31 | 4.80 |

- Kuk, Saunders, Davidson & Ross 2009 (Ageing Res Rev 8:339): with age, abdominal and especially visceral fat rise preferentially while **lower-body subcutaneous fat falls**, and this can happen **without change in total fat, weight or waist** — [abstract record](https://scinapse.io/papers/1974545118) [snippet].
- Health ABC (70 to 79 y, mid-thigh CT, 5-y follow-up, n ≈ 1,678): thigh subcutaneous fat rose only in people who gained weight and fell in those who lost weight; **intermuscular fat rose in every group** (losers, gainers, weight-stable) — [PMC2777469 search record](https://pmc.ncbi.nlm.nih.gov/articles/2777469) [snippet; paper identity inferred from the result list]. Over longer follow-up both sexes lost visceral fat area, thigh muscle area, lean mass and fat mass but gained intermuscular thigh fat — same search [snippet].
- Pubertal subcutis by ultrasound (NZ, type 1 diabetes, otherwise healthy, 5 to 19 y): **girls 16.7 mm vs boys 7.5 mm at lateral mid-thigh; 16.7 vs 8.8 mm at the abdomen**; girls' subcutis grew 54% (thigh) and 68% (abdomen) during puberty; in adults (20 to 85 y) dermis and subcutis thinned with age — [Derraik et al. 2014, PMC3897752](https://pmc.ncbi.nlm.nih.gov/articles/PMC3897752/) [snippet].
- Adult injection-site ultrasound (293 Filipino adults with diabetes): mean subcutaneous thickness 6.9 to 19.1 mm across sites; anterior thigh thinnest — [JAFES](https://www.asean-endocrinejournal.org/index.php/JAFES/article/view/112) [snippet]. Gibney et al. 2010 (341 adults): skin-surface-to-fascia distance varies by site, BMI and sex and is lowest at the thigh — [summary, PMC4021299](https://pmc.ncbi.nlm.nih.gov/articles/4021299) [snippet].
- Standardized 8-site ultrasound method (Müller et al. 2016, IOC working group): sites upper abdomen, lower abdomen, erector spinae, distal triceps, brachioradialis, lateral thigh, front thigh, medial calf; 18 MHz probe; 0.2 mm accuracy; speed of sound 1450 m/s — [BJSM 50:45](https://bjsm.bmj.com/content/50/1/45.abstract) [snippet]. The mean of these 8 sites overestimates the mean of 216 random whole-body sites; **calibration factor 0.65** (n = 10, BMI < 28.5) — [Störchle et al. 2018, PMC6214952](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6214952/) [snippet].
- Older adults, NHANES III, non-Hispanic white, mean age about 66 y: mean triceps skinfold **13.5 mm (men), 23.3 mm (women)** — [PMC1097739 table](https://pmc.ncbi.nlm.nih.gov/articles/PMC1097739/table/T1) [snippet]. Brazilian and Chilean surveys of people ≥ 60 y show mean and median skinfolds falling with age; arm fat area falls with age in NZ adults ≥ 70 y — [SciELO](https://scielosp.org/pdf/csp/2007.v23n12/2887-2895/pt) [snippet].
- NHANES 2003-2006 percentile tables of triceps and subscapular skinfold for adults by age exist (McDowell et al. 2008, NHSR 10, Tables 31-32) — [CDC PDF](https://www.cdc.gov/nchs/data/nhsr/nhsr010.pdf) [snippet: table titles only, values not read].

### Inferences
- **Adult site medians to use until the NHANES tables are read** (mm, caliper) [inference from the snippets above and textbook values]: triceps men 11 to 12 (20 to 40 y), 13 to 14 (50 to 70 y), 11 to 12 (80 y); women 20 to 22, 24 to 25, 19 to 21. Subscapular men 15 to 20, women 14 to 20, both rising to about 60 y. Suprailiac men 15 to 22, women 15 to 22.
- **Sex pattern for the layer** [inference]: male = trunk-weighted (flank, upper abdomen, back), female = gynoid (buttock, lateral and medial thigh, lower abdomen, breast) plus thicker everywhere (×1.6 to 2 at the arm). Use the DXA android/gynoid ratio as the global weight between an "android" thickness map and a "gynoid" map.
- **Elderly** [inference]: keep total subcutaneous volume from the %BF curve, shift volume from limbs (especially thigh and calf) to abdomen with the trunk/limb ratio, add visceral volume inside the abdominal wall (pushes the belly out without thickening the skin layer), and from about 75 y thin the limb layer by 5 to 15% if the character is frail or losing weight.

### Gaps
- No adult per-site ultrasound reference values (mm by age and sex) were read; only the method papers.
- No infant ultrasound maps; infant thigh and cheek fat thickness are not measured here.
- Breast, buttock and cheek fat volumes by age are not covered.

---

## 3. Muscle: total and regional muscle mass by age, sarcopenia rates, limb cross-section composition

### Takeaway
- **Adult skeletal muscle**: men 33.0 kg (38.4% of body mass), women 21.0 kg (30.6%) on whole-body MRI; the male excess is larger in the upper body (40%) than the lower body (33%).
- **Decline**: relative muscle mass falls from the 30s; absolute muscle falls clearly from the end of the 40s, mostly in the legs. Men 18-29 vs > 70 y: lower body −25% (−4.7 kg), upper body −5.6% (−0.8 kg).
- **Rates**: median loss 0.47%/y (men) and 0.37%/y (women) across studies; at 75 y 0.80 to 0.98%/y (men) and 0.64 to 0.70%/y (women). Strength falls 2 to 5 times faster than mass.
- **DXA appendicular lean (ALM)**: men about 30 kg flat from 20 to 45 y, 28.0 kg at 60, 24.2 kg at 80 (−19%); women about 19.8 kg to 40 y, 18.3 at 60, 16.2 at 80 (−18%). Convert to skeletal muscle with SM ≈ 1.12 × ALM − 0.63 kg.
- **Growth**: lean mass index (NHANES DXA) is about 11 to 12 kg/m² at 8 y in both sexes; boys reach 17.5 and girls 14.1 kg/m² at 18 y. Leg lean mass (one leg, DXA) goes 3.2 → 9.5 kg (boys) and 2.9 → 6.8 kg (girls) from 8 to 20 y.
- **Limb cross-sections** follow from circumference + fat thickness + bone (Section 6). Reference values: adult mid-upper-arm muscle (incl. bone) about 65 to 70 cm² men, 44 cm² women; calf muscle at the widest point about 80 cm² men, 57 cm² women; mid-thigh CT muscle 157 cm² (men 27 y) vs 137 cm² (men 71 y).

### Cited Findings
- Janssen, Heymsfield, Wang & Ross 2000 (J Appl Physiol 89:81; whole-body MRI, n = 468, 18 to 88 y): men 33.0 vs women 21.0 kg SM; 38.4% vs 30.6% of body mass; sex difference 40% upper vs 33% lower body; relative SM falls from the 3rd decade, absolute SM from the end of the 5th decade, mostly lower body; weight and height explain about 50% of SM variance — [Bioblast summary](https://bioblast.at/index.php/Janssen_2000_J_Appl_Physiol) [snippet]. Lower-body loss 4.7 kg (25%) vs upper 0.8 kg (5.6%), men 18-29 vs > 70 — [table in PMC3429036](https://pmc.ncbi.nlm.nih.gov/articles/PMC3429036/table/T1) [snippet].
- Mitchell et al. 2012 (Front Physiol 3:260, quantitative review): median loss 0.47%/y men, 0.37%/y women; at 75 y 0.80 to 0.98%/y men, 0.64 to 0.70%/y women; strength at 75 y −3 to −4%/y men, −2.5 to −3%/y women; strength loss 2 to 5× mass loss — [PMC3429036](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3429036/) [snippet].
- DXA ALM to MRI muscle: Kim et al. 2002 built SM-from-ALM equations on 321 adults (revised 2004 after removing intermuscular fat); McCarthy et al. 2023 proposed **SM = 1.12 × ALM − 0.63 (kg)** — [PMC9929067](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9929067/) [snippet].
- Kirk 2021 ALM and total lean tables above [fetched]. Ofenheimer ALMI: men 8.09 (18 y) → 8.42 (50 y) → 7.75 (80 y) kg/m²; women about 6.1 → 6.6, flat after 35 y [fetched]. The two cohorts disagree on women's appendicular loss (Kirk −18%, Ofenheimer flat as an index; part of the gap is height loss) [computed].
- Duran 2019 LBMI and Schafmeyer 2022 leg lean [fetched]: leg lean (white, median, kg) boys 3.22 (8 y), 4.00 (10), 5.30 (12), 7.24 (14), 8.77 (16), 9.31 (18), 9.50 (20); girls 2.87, 4.04, 5.09, 5.86, 6.28, 6.52, 6.78. Leg fat boys 1.49 → 3.27 kg, girls 1.87 → 5.05 kg from 8 to 20 y. I read these as one leg (an adult man's both-leg lean is about 22 kg) [inference: unit check needed].
- Thigh CT, 10 young (≈ 27 y) vs 10 older (≈ 71 y) men: **157 ± 24 vs 137 ± 17 cm²** (−13%); lower-body muscle volume (MRI) −20% — [search result, PMC5111551 or nearby record](https://pmc.ncbi.nlm.nih.gov/articles/PMC5111551) [snippet; attribution uncertain]. Health ABC (70 to 79 y): mean 5-year change in mid-thigh muscle CSA **−9.8 cm²** (about −2 cm²/y) — [Verreijen et al. 2019, PMC6408207](https://pmc.ncbi.nlm.nih.gov/articles/PMC6408207) [snippet]. Over 5 y, knee-extensor torque fell 16.1% (men) and 13.4% (women) — [search record](https://pmc.ncbi.nlm.nih.gov/articles/2777469) [snippet].
- pQCT definitions (Stratec XCT): the 66% tibia site is the calf level with the largest muscle area; **muscle CSA = (muscle + bone area) − bone area; fat CSA = total area − (muscle + bone area)** — [Pediatr Blood Cancer 2023](https://onlinelibrary.wiley.com/doi/pdfdirect/10.1002/pbc.30705) [snippet]. Child references with muscle and fat CSA percentiles exist: Jaworski & Graff, lower leg 4/14/38/66% tibia, 222 subjects 4.3 to 19.4 y, LMS curves by age and by height — [PMC8185261](https://pmc.ncbi.nlm.nih.gov/articles/PMC8185261) [snippet]; forearm, 5 to 19 y — [JMNI 18:237](https://www.ismni.org/jmni/article/18/237) [snippet]; Rauch & Schoenau proximal radius (65%), 469 subjects 6 to 40 y — [JMNI 8:217](https://www.ismni.org/jmni/article/8/217) [snippet]. I could not read their numbers.
- Upper-arm area method (Frisancho 1981, AJCN 34:2540; 19,097 white subjects 1 to 74 y, NHANES I): percentiles of arm muscle circumference, arm muscle area and arm fat area by age and sex — [QxMD record](https://www.qxmd.com/r/6975564) [snippet]. Formula: **AMA = (MUAC − π·TSF)² / (4π)** with MUAC and TSF in cm; bone-corrected AMA subtracts **10 cm² (men) or 6.5 cm² (women)**; arm fat area = MUAC²/(4π) − AMA [inference: standard formula, constants not confirmed in the snippet]. It assumes a circular arm with a uniform fat ring.
- Adult femur mid-shaft transverse diameter: **26.5 mm (men), 25.6 mm (women)** (104 Anatolian adults, radiographs) — [Diagn Interv Radiol](https://dirjournal.org/articles/femoral-shaft-bowing-with-age-a-digital-radiological-study-of-anatolian-caucasian-adults/56956) [snippet].

### Inferences
- **Muscle-mass schedule for a "muscle" knob** [inference from the above]: keep each character's muscle z-score; adult muscle = reference(age) × exp(σ·z). Reference: flat 20 to 45 y; then −0.3%/y (45 to 60), −0.6%/y (60 to 75), −0.9%/y (> 75) in men; women −0.25, −0.5, −0.7%/y. Apply 80% of the loss to the legs and lower trunk and 20% to arms and upper trunk (Janssen pattern).
- Per-limb muscle volume from ALM: SM_limbs ≈ 1.12·ALM − 0.63 kg total, split arms : legs ≈ 25 : 75 (men) and 22 : 78 (women) [inference]. Volume = mass / 1.06 g/cm³ (muscle density) [inference].
- Children's muscle can be driven by LBMI(age) (Duran) and leg lean (Schafmeyer).

### Gaps
- No pQCT or MRI muscle/fat area numbers by age were read (all blocked). The calf and arm values in Section 6 are model outputs checked only against a few snippet anchors.
- No muscle data for 0 to 7 y except arm geometry (WHO MUAC + skinfolds).
- Intermuscular and intramuscular fat (rises with age) is not quantified here.

---

## 4. Skin thickness, elasticity and sagging by age

### Takeaway
- **Skin (epidermis + dermis) is about 2 mm thick in adults at the arm, thigh, abdomen and buttock** (ultrasound means 1.8 to 2.75 mm) and varies little with BMI or sex. It is thinner in children and thickens steadily from birth to adulthood.
- **Adults thin the dermis and subcutis with age** (significant in a 20- to 85-year sample). After menopause skin thickness falls about 1 to 2%/y (one estimate 1.13%/y), in step with collagen.
- **Face is an exception in some data**: one 20 MHz study found elderly facial skin thicker and more echogenic than young skin at all sites except under the eye.
- **Elasticity** (Cutometer R2, R7) falls with age at every site tested; R7 correlates about −0.62 with age in women 18 to 70 y; the main decline is in the midface; crow's-feet region drops most after 30 y.
- **Visible old-age shape** comes less from skin thickness and more from: fat redistribution between separate facial fat compartments, loss of deep fat-pad elasticity (deep medial cheek fat), lower skin recoil (gravity sag: jowls, under-chin, upper-arm "wings", inner thigh, buttock fold drop, breast ptosis), muscle loss in the legs, visceral belly growth, and stature loss.

### Cited Findings
- Derraik et al. 2014 (103 children 5 to 19 y, 140 adults 20 to 85 y; abdomen and thigh ultrasound): dermis and subcutis thicken with age in children and thin with age in adults (dermis p = 0.021, subcutis p = 0.009); skin thickens with BMI — [PMC3897752](https://pmc.ncbi.nlm.nih.gov/articles/PMC3897752/) [snippet].
- Gibney et al. 2010: skin "on average 2 mm thick with little variation, regardless of age, gender, BMI and ethnicity" (as summarized by a secondary source) — [PMC4021299](https://pmc.ncbi.nlm.nih.gov/articles/4021299) [snippet]. Filipino adults: skin 1.76 to 2.75 mm across injection sites, anterior thigh thinnest — [JAFES](https://www.asean-endocrinejournal.org/index.php/JAFES/article/view/112) [snippet]. South African children 4 to 18 y: maximum skin thickness at any site 2.93 mm — [SAJCH](https://scielo.org.za/pdf/sajch/v8n3/04.pdf) [snippet].
- Seidenari et al. 2000 (20 MHz, 42 children and 30 young adults, 8 sites): skin thickness increases gradually from birth to adulthood; echogenicity falls with maturation on face and trunk and rises on the limbs — [IRIS Unimore](https://iris.unimo.it/handle/11380/612798) [snippet].
- Facial skin (40 women, 12 sites, 20 MHz): thicker and more echogenic in the elderly except infraorbital; lower face thicker than upper face — [Acta Derm Venereol](https://medicaljournalssweden.se/actadv/article/view/14205) [snippet].
- Postmenopausal women: skin collagen, skin thickness and forearm bone fall 1 to 2%/y — [PubMed 3120067](https://pubmed.ncbi.nlm.nih.gov/3120067/) [snippet]; a skin-mechanics review cites 1.13%/y thickness loss with 2%/y collagen loss after menopause, faster in women — [arXiv 1709.03752](https://arxiv.org/pdf/1709.03752) [snippet].
- Cutometer: age correlates negatively with R2 and R7 (129 East Asian women, main decline midface); R7 r = −0.62 with age (60 Chinese women, 18 to 70 y); in 669 volunteers R2, R5 and R7 fell with age at 8 sites, most at the crow's feet after 30 y — [PMC6615427](https://pmc.ncbi.nlm.nih.gov/articles/PMC6615427) and related results [snippet].
- Facial fat is split into discrete compartments (nasolabial, medial/middle/lateral-temporal cheek, forehead, orbital, jowl); the face "does not age as a confluent mass"; shear between compartments may cause malposition — [Rohrich & Pessa 2007](https://utsouthwestern.elsevierpure.com/en/publications/the-fat-compartments-of-the-face-anatomy-and-clinical-implication) [snippet]. Deep medial cheek fat elasticity (shear-wave elastography, 89 women) falls with age; age is its only independent determinant; linked to midface pseudoptosis — [Postepy Dermatol Alergol](https://www.termedia.pl/Age-related-changes-in-elastographically-determined-strain-of-the-facial-fat-compartments-a-new-frontier-of-research-on-face-aging-processes,7,34200,0,1.html) [snippet].

### Inferences
- **Skin layer thickness for HeroBody** (mm, body average; face 1.2 to 1.8, eyelid 0.5, palms/soles 2 to 4 not modelled here) [inference]: 0.5 y 1.2; 1 y 1.3; 2 y 1.4; 3.5 y 1.5; 6 y 1.6; 8 y 1.7; 10 y 1.8; 12 y 1.9; 14 to 50 y 2.0; 65 y 1.9; 80 y 1.7. It is only 1 to 2% of a limb radius, so its main job is as a sliding/sag layer, not as volume.
- **Sag model** [inference]: treat skin + fat as one shell with an age-dependent "laxity" a(age) ≈ 0 to 30 y, rising linearly to 1 at 85 y (match R7 decline). Under gravity in the rest pose, displace the shell along −Z by k·a·t_fat in regions with loose attachment (submental, jowl, upper-arm posterior, inner thigh, lower abdomen, breast, buttock fold), with k ≈ 0.3 to 0.6. Anchor lines (retaining ligaments: zygomatic, mandibular, inguinal, gluteal fold) get zero displacement. This is a corrective blendshape per stage, not a simulation, which suits the CPU-only constraint.

### Gaps
- No site-by-site dermis thickness table by age (e.g. per decade) was read.
- Infant skin thickness numbers (Ploin 2011, Lo Presti 2012, Lim 2018) were not reached.
- No quantitative sag displacement data (mm of jowl drop per decade) were found.

---

## 5. Methods to build a fat layer for a parametric body

### Takeaway
- Three families exist: **(a) layered offset surfaces** (skin, muscle, skeleton shells with fat as the gap; Komaritzan/Botsch "Inside Humans", Kadlecek 2016, classic Wilhelms & Van Gelder / Scheepers 1997), **(b) volumetric growth simulation** (Saito, Zhou & Kavan 2015 "Computational Bodybuilding": grow muscle and fat with physics, Projective Dynamics), **(c) learned implicit tissue** (HIT 2024: point → lean / adipose / bone / empty, conditioned on SMPL shape, trained on MRI of 127 men and 191 women).
- For HeroBody, (a) is the right first build: it is CPU-cheap, deterministic, editable and fits the existing "anatomy inside skin + clamp" design. Use (b) only for offline hero shots, and (c) only as a reference (SMPL-bound, adult-only, non-commercial terms, GPU training).
- **Make it age-dependent and knob-driven** with one rule: thickness map t(site) = scale × template_ratio(site, stage, sex, fatness) so that ∫ t dA = subcutaneous fat volume(age, sex, fatness z). Skinfold curves give the ratios; DXA %BF gives the volume.

### Cited Findings
- **Inside Humans** (Komaritzan, Wenninger & Botsch, Frontiers in VR 2021): fits a volumetric template to a surface scan in seconds; **three surface layers (skin, muscle, skeleton) enclose volumetric muscle and fat between them**; a data-driven estimate of the amount of muscle and fat from the scan; high-resolution skeleton and muscle models can be embedded in the layered fit — [TU Dortmund PDF](https://cg.cs.tu-dortmund.de/publications/2021-insidehumans.pdf) [snippet]. It notes Saito et al. showed that a layer enveloping the muscles gives more convincing growth with fewer tetrahedra — same PDF [snippet].
- **Computational Bodybuilding** (Saito, Zhou & Kavan, SIGGRAPH 2015): from one anatomy template, physics-based growth of muscles and subcutaneous fat with elasticity; user sets hypertrophy/atrophy per muscle and amount of fat; near-interactive with Projective Dynamics; outputs volumetric models for simulation — [project page](https://users.cs.utah.edu/~ladislav/saito15computational/saito15computational.html) [snippet].
- **Reconstructing personalized anatomical models** (Kadleček et al., SIGGRAPH Asia 2016): average-male anatomical template whose parameters cover bones, muscles and **adipose tissue** plus pose; fitted by a large optimization to multi-pose scans; physics-ready — [project page](https://users.cs.utah.edu/~ladislav/kadlecek16reconstructing/kadlecek16reconstructing.html) [snippet].
- **HIT** (Keller et al., CVPR 2024): input SMPL β, θ and a 3D point; output class probabilities for lean tissue (muscle, organs), **adipose tissue (subcutaneous)**, bone and empty space; points are canonicalized to a template first; meshes by marching cubes; dataset 127 males and 191 females with MRI segmentations and SMPL fits; licence: "Before commercial usage of source code, the copyright holder must be contacted"; install expects CUDA torch — [GitHub README and LICENSE](https://github.com/MarilynKeller/HIT) [fetched].
- **Anny phenotype priors**: Anny samples `weight` and `muscle` knobs from age-conditional Beta distributions anchored at ages 0, 1, 4, 11, 16, 18, 64, 110 y. Mean `weight`: boys 0.62, 0.97, 0.43, 0.24, 0.26, 0.36, 0.59, 0.59; girls 0.63, 0.97, 0.71, 0.54, 0.57, 0.59, 0.73, 0.74. Mean `muscle`: boys 0.56, 0.03, 0.48, 0.19, 0.47, 0.34, 0.47, 0.55; girls 0.50, 0.03, 0.54, 0.48, 0.49, 0.44, 0.19, 0.19 — [naver/anny shape_calibration/{boys,girls}.pth](https://github.com/naver/anny/tree/main/src/anny/data/shape_calibration) [fetched; means computed as α/(α+β)]. The age-1 anchor (weight ≈ 0.97, muscle ≈ 0.03) is extreme and matches the "age knob breaks at the baby end" finding of the Phase 1 atlas [inference].
- Classic anatomy-based modelling (Wilhelms & Van Gelder 1997; Scheepers et al. 1997) built skin as a surface offset from or attached to the muscle layer, with fat thickness as a spring rest length or offset [inference: recalled, not fetched this session].

### Inferences
- **Recommended HeroBody fat-layer construction (CPU only)** [inference]:
  1. Inputs per character and stage: sex s, age a, fatness z_f (FMI z-score), muscle z_m (ALMI or LBMI z-score), skin outer mesh (from the knob solver).
  2. Fat volume: V_SAT = FM(a, s, z_f) × share_SAT / 0.9 g/cm³, with share_SAT = 0.93 (infant) → 0.90 (child) → 0.88 (teen) → 1 − VAT/FM − 0.08 (adult; 0.08 covers intermuscular, marrow and other internal fat).
  3. Region template: 16 to 20 painted regions on the hm08 skin (cheek, submental, neck, deltoid, triceps, biceps, forearm, chest/breast, upper back/subscapular, flank/suprailiac, upper abdomen, lower abdomen, buttock, thigh anterior, thigh lateral, thigh medial, calf, knee/shin, hands/feet). Each region has ratio r_i(a, s) from skinfold ratios (Section 2 table) and DXA android/gynoid and trunk/limb ratios.
  4. Thickness: t_i = λ · r_i · exp(σ_i·z_f) with λ solved so that Σ t_i·A_i = V_SAT (A_i = region area on the skin mesh). Smooth across region borders with a heat-diffusion or Laplacian smoothing of the per-vertex value (5 to 10 iterations).
  5. Muscle shell = skin vertices − (t_skin + t_i)·n̂ (vertex normal), then clamp so it never crosses the registered BodyParts3D muscles + bones (reuse the existing per-body clamp). Where the shell would cross bone (shin, elbow, knee, skull, iliac crest), set t_i → 1 to 3 mm.
  6. Store per vertex: t_skin, t_fat, and the muscle-shell position. Export to Unreal as a second mesh or as a vertex attribute for a deformer.
- **Blender implementation**: per-vertex thickness in a vertex group or attribute; a Geometry Nodes "Set Position" offset along −normal by the attribute; or the Solidify modifier with a vertex-group thickness factor (inward, with "Even thickness") to get a closed fat shell. Volume check by `bmesh.calc_volume()` of skin minus muscle shell. All CPU.
- **Optional volumetric sim (offline)**: tetrahedralize the fat shell (TetGen or fTetWild, CPU), assign fat as a soft neo-Hookean (E ≈ 1 to 3 kPa), muscle stiffer (E ≈ 10 to 50 kPa passive), skin as a membrane; quasi-static solve with Projective Dynamics or XPBD on CPU for jiggle/compression. In Unreal, Chaos Flesh exists for tetrahedral flesh but is experimental [inference; flag before use].
- **Age dependence**: r_i(a, s) comes from data tables by stage and is interpolated within a stage by the in-stage slider; V_SAT(a, s, z_f) from the %BF curve. Elderly sag is a separate corrective (Section 4), and visceral fat is a separate interior volume that pushes the abdominal muscle wall outward.

### Gaps
- No method paper gives fat thickness maps for children; all published layered models are adult.
- The HIT dataset (MRI segmentations) could give adult regional thickness maps but is on huggingface.co and the project site, both blocked here, and is bound to SMPL (licence must be checked).
- Wilhelms/Scheepers details were not re-read.

---

## 6. Layered cross-section model: bone radius, muscle thickness, fat thickness → outer circumference, and fat-thickness maps per stage

### Takeaway
- A concentric-ring model per limb segment reproduces the reference circumferences exactly (they are its input) and gives muscle and fat areas within the range of published anchors: adult upper-arm muscle (incl. bone) 67 cm² men / 44 cm² women; widest-calf muscle 80 / 57 cm²; adult fat ring at the arm 4 mm (men) vs 8.5 mm (women); thigh 14 vs 22 mm.
- **Fat ring thickness by stage (men / women, mm)**: arm 3.4/3.3 at 0.5 y, 2.4/2.7 at 3.5 y, 4.2/4.9 at 10 y, 3.5/8.4 at 18 y, 4.0/8.5 at 25 y, 4.5/10.5 at 45 y, 4.3/8.3 at 80 y. Thigh 8/8 at 1 to 3.5 y, 13/16 at 10 y, 13/21 at 18 y, 14/22 at 25 y, 16/25 at 80 y.
- **Mean subcutaneous fat thickness over the whole skin** (volume / skin area): about 5 mm in infants, 4 to 4.5 mm at 3.5 to 6 y (minimum), 5.5 mm (boys) / 8.5 mm (girls) at 18 y, 7 / 10.6 mm at 25 y, 8.5 / 12.5 at 45 y, 10 / 16 mm at 80 y.
- For fat-layer *maps*, use the skinfold site ratios for the shape and the mean-thickness table for the scale. Do not use skinfold/2 directly as absolute thickness: it is a compressed value.

### Cited Findings
- Ring and skinfold conventions from Frisancho (Section 3) [snippet/inference]. Circumference inputs from the sister note's stage tables, built from fetched Snyder 1977, WHO, NL4, LIFE, ANSUR and NHANES data — `proportions_by_age.md` Tables 6a/6b [computed in that note].
- Limb fat fractions from DXA (this note) [computed from fetched tables]:
  - Adults (Ofenheimer): limb fat volume fraction f = (LF/0.9) / (LF/0.9 + ALM/1.06), LF = 0.95·FMI/(1 + trunk/limb ratio) (per m²). Men 0.26 (20 y), 0.27 (25), 0.29 (45), 0.31 (65), 0.33 (80); women 0.42, 0.42, 0.43, 0.46, 0.48.
  - Children (Schafmeyer, one leg): f = 0.35 (8 y), 0.38 (10), 0.37 (12), 0.32 (14), 0.29 (16), 0.29 (18) in boys; 0.43, 0.43, 0.43, 0.43, 0.44, 0.45 in girls.

### Equations (for code)
- Outer radius: R = C / (2π).
- Skinfold route (arm): t_skin + t_fat = TSF/2, so t_fat = TSF/2 − t_skin; muscle outer radius R_m = R − t_skin − t_fat.
- Fat-fraction route (thigh, calf): with inner skin radius R_i = R − t_skin, bone radius r_b and soft-tissue fat fraction f: t_fat = R_i − sqrt(R_i² − f·(R_i² − r_b²)).
- Areas: A_muscle = π(R_m² − r_b²); A_fat = π(R_i² − R_m²); A_bone = π r_b²; muscle thickness = R_m − r_b.
- Bone radius: r_b = k·H with k = 0.0062 (humerus mid-shaft), 0.0075 (femur, men), 0.0078 (femur, women), 0.0074 (tibia + fibula combined, widest calf) [inference, calibrated to femur 26.5/25.6 mm]. For HeroBody use the actual registered BodyParts3D bone section instead when available.
- Mean SAT thickness: t̄ = SAT_mass / (0.9 g/cm³ × BSA), BSA (Mosteller) = sqrt(H_cm × W_kg / 3600) m².
- Volume closure for a thickness map: λ = V_SAT / Σ_i (r_i·A_i); t_i = λ·r_i.

### Cross-section table [computed with the equations above; inputs listed under Inferences]
C = circumference at mid-upper-arm (arm), at the thigh level used by the stage tables (gluteal-furrow/upper thigh for 2 y and older), and at maximum calf. Thickness in mm, radius in cm, areas in cm².

| age | sex | H | seg | C cm | R cm | skin mm | fat mm | muscle mm | bone r cm | muscle cm2 | fat cm2 | bone cm2 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 0.5 | M | 67.6 | arm | 14.2 | 2.26 | 1.2 | 3.4 | 13.8 | 0.42 | 9.6 | 4.2 | 0.6 |
| 1 | M | 75.7 | arm | 14.6 | 2.32 | 1.3 | 2.8 | 14.5 | 0.47 | 10.9 | 3.6 | 0.7 |
| 1 | M | 75.7 | thigh | 23.5 | 3.73 | 1.3 | 7.9 | 22.5 | 0.57 | 23.9 | 15.9 | 1.0 |
| 1 | M | 75.7 | calf | 18.2 | 2.89 | 1.3 | 5.9 | 16.1 | 0.56 | 13.8 | 9.2 | 1.0 |
| 2 | M | 86.5 | arm | 15.2 | 2.42 | 1.4 | 2.5 | 15.0 | 0.54 | 12.1 | 3.3 | 0.9 |
| 2 | M | 86.5 | thigh | 26.7 | 4.25 | 1.4 | 8.0 | 26.7 | 0.65 | 33.2 | 18.7 | 1.3 |
| 2 | M | 86.5 | calf | 19.3 | 3.07 | 1.4 | 5.5 | 17.4 | 0.64 | 16.4 | 9.2 | 1.3 |
| 3.5 | M | 98.7 | arm | 15.9 | 2.53 | 1.5 | 2.4 | 15.3 | 0.61 | 13.2 | 3.4 | 1.2 |
| 3.5 | M | 98.7 | thigh | 29.5 | 4.70 | 1.5 | 8.0 | 30.1 | 0.74 | 42.4 | 20.9 | 1.7 |
| 3.5 | M | 98.7 | calf | 20.9 | 3.33 | 1.5 | 5.4 | 19.1 | 0.73 | 20.2 | 9.9 | 1.7 |
| 6 | M | 115.4 | arm | 17.2 | 2.74 | 1.6 | 2.9 | 15.7 | 0.72 | 14.8 | 4.4 | 1.6 |
| 6 | M | 115.4 | thigh | 33.2 | 5.29 | 1.6 | 9.0 | 33.6 | 0.87 | 53.8 | 26.5 | 2.4 |
| 6 | M | 115.4 | calf | 23.0 | 3.65 | 1.6 | 5.9 | 20.5 | 0.85 | 24.2 | 11.9 | 2.3 |
| 8 | M | 127.9 | arm | 18.4 | 2.93 | 1.7 | 3.2 | 16.4 | 0.79 | 16.7 | 5.3 | 2.0 |
| 8 | M | 127.9 | thigh | 36.6 | 5.82 | 1.7 | 10.7 | 36.2 | 0.96 | 63.0 | 34.5 | 2.9 |
| 8 | M | 127.9 | calf | 25.3 | 4.03 | 1.7 | 7.1 | 22.1 | 0.95 | 28.4 | 15.6 | 2.8 |
| 10 | M | 138.6 | arm | 20.0 | 3.18 | 1.8 | 4.2 | 17.1 | 0.86 | 18.5 | 7.4 | 2.3 |
| 10 | M | 138.6 | thigh | 40.3 | 6.42 | 1.8 | 12.8 | 39.2 | 1.04 | 74.0 | 44.9 | 3.4 |
| 10 | M | 138.6 | calf | 27.6 | 4.39 | 1.8 | 8.3 | 23.5 | 1.03 | 32.6 | 19.8 | 3.3 |
| 12 | M | 149.1 | arm | 21.9 | 3.49 | 1.9 | 5.0 | 18.7 | 0.92 | 21.9 | 9.6 | 2.7 |
| 12 | M | 149.1 | thigh | 44.1 | 7.02 | 1.9 | 13.5 | 43.7 | 1.12 | 90.5 | 52.3 | 3.9 |
| 12 | M | 149.1 | calf | 29.7 | 4.72 | 1.9 | 8.6 | 25.7 | 1.10 | 38.5 | 22.2 | 3.8 |
| 14 | M | 163.8 | arm | 24.6 | 3.91 | 2.0 | 4.4 | 22.5 | 1.02 | 30.4 | 9.6 | 3.2 |
| 14 | M | 163.8 | thigh | 49.5 | 7.87 | 2.0 | 13.3 | 51.2 | 1.23 | 121.8 | 58.4 | 4.7 |
| 14 | M | 163.8 | calf | 33.3 | 5.29 | 2.0 | 8.5 | 30.3 | 1.21 | 51.9 | 24.9 | 4.6 |
| 16 | M | 173.5 | arm | 27.4 | 4.36 | 2.0 | 3.5 | 27.4 | 1.08 | 42.2 | 8.7 | 3.6 |
| 16 | M | 173.5 | thigh | 52.0 | 8.28 | 2.0 | 12.5 | 55.4 | 1.30 | 141.6 | 58.4 | 5.3 |
| 16 | M | 173.5 | calf | 35.2 | 5.61 | 2.0 | 8.0 | 33.2 | 1.28 | 61.3 | 25.3 | 5.2 |
| 18 | M | 176.2 | arm | 29.1 | 4.63 | 2.0 | 3.5 | 29.8 | 1.09 | 48.3 | 9.5 | 3.7 |
| 18 | M | 176.2 | thigh | 54.6 | 8.69 | 2.0 | 12.8 | 58.9 | 1.32 | 158.1 | 63.0 | 5.5 |
| 18 | M | 176.2 | calf | 36.5 | 5.80 | 2.0 | 8.1 | 34.9 | 1.30 | 66.7 | 26.6 | 5.3 |
| 25 | M | 176.7 | arm | 33.6 | 5.34 | 2.0 | 4.0 | 36.5 | 1.10 | 66.9 | 12.4 | 3.8 |
| 25 | M | 176.7 | thigh | 62.2 | 9.90 | 2.0 | 13.8 | 69.9 | 1.33 | 211.7 | 78.3 | 5.5 |
| 25 | M | 176.7 | calf | 39.2 | 6.24 | 2.0 | 8.4 | 39.0 | 1.31 | 79.8 | 29.5 | 5.4 |
| 45 | M | 176.2 | arm | 33.8 | 5.38 | 2.0 | 4.5 | 36.4 | 1.09 | 66.7 | 14.0 | 3.7 |
| 45 | M | 176.2 | thigh | 63.4 | 10.10 | 2.0 | 15.4 | 70.3 | 1.32 | 213.6 | 88.5 | 5.5 |
| 45 | M | 176.2 | calf | 39.5 | 6.28 | 2.0 | 9.2 | 38.6 | 1.30 | 78.4 | 32.5 | 5.3 |
| 65 | M | 173.4 | arm | 32.6 | 5.19 | 1.9 | 4.8 | 34.4 | 1.08 | 60.4 | 14.5 | 3.6 |
| 65 | M | 173.4 | thigh | 60.7 | 9.66 | 1.9 | 15.6 | 66.1 | 1.30 | 191.3 | 85.1 | 5.3 |
| 65 | M | 173.4 | calf | 37.3 | 5.93 | 1.9 | 9.1 | 35.5 | 1.28 | 68.1 | 30.3 | 5.2 |
| 80 | M | 170.3 | arm | 30.7 | 4.88 | 1.7 | 4.3 | 32.2 | 1.06 | 54.0 | 12.1 | 3.5 |
| 80 | M | 170.3 | thigh | 57.9 | 9.22 | 1.7 | 15.8 | 61.9 | 1.28 | 170.0 | 81.9 | 5.1 |
| 80 | M | 170.3 | calf | 34.9 | 5.56 | 1.7 | 9.0 | 32.2 | 1.26 | 58.2 | 28.0 | 5.0 |
| 0.5 | F | 65.7 | arm | 13.8 | 2.20 | 1.2 | 3.3 | 13.3 | 0.41 | 9.0 | 4.0 | 0.5 |
| 1 | F | 74.0 | arm | 14.2 | 2.26 | 1.3 | 2.7 | 14.0 | 0.46 | 10.2 | 3.4 | 0.7 |
| 1 | F | 74.0 | thigh | 22.9 | 3.65 | 1.3 | 7.9 | 21.5 | 0.58 | 22.4 | 15.5 | 1.0 |
| 1 | F | 74.0 | calf | 17.8 | 2.83 | 1.3 | 6.0 | 15.5 | 0.55 | 12.9 | 9.0 | 0.9 |
| 2 | F | 85.0 | arm | 14.9 | 2.37 | 1.4 | 2.5 | 14.5 | 0.53 | 11.5 | 3.3 | 0.9 |
| 2 | F | 85.0 | thigh | 27.2 | 4.33 | 1.4 | 8.7 | 26.6 | 0.66 | 33.3 | 20.4 | 1.4 |
| 2 | F | 85.0 | calf | 19.5 | 3.10 | 1.4 | 6.0 | 17.3 | 0.63 | 16.3 | 10.0 | 1.2 |
| 3.5 | F | 97.4 | arm | 15.9 | 2.53 | 1.5 | 2.7 | 15.1 | 0.60 | 12.8 | 3.8 | 1.1 |
| 3.5 | F | 97.4 | thigh | 30.5 | 4.85 | 1.5 | 9.1 | 30.3 | 0.76 | 43.3 | 24.4 | 1.8 |
| 3.5 | F | 97.4 | calf | 20.8 | 3.32 | 1.5 | 6.0 | 18.5 | 0.72 | 19.1 | 10.8 | 1.6 |
| 6 | F | 114.7 | arm | 17.1 | 2.72 | 1.6 | 3.2 | 15.3 | 0.71 | 14.2 | 4.8 | 1.6 |
| 6 | F | 114.7 | thigh | 34.3 | 5.46 | 1.6 | 10.9 | 33.1 | 0.89 | 53.1 | 32.6 | 2.5 |
| 6 | F | 114.7 | calf | 22.9 | 3.65 | 1.6 | 6.9 | 19.5 | 0.85 | 22.3 | 13.7 | 2.3 |
| 8 | F | 127.6 | arm | 18.6 | 2.96 | 1.7 | 3.8 | 16.2 | 0.79 | 16.3 | 6.3 | 2.0 |
| 8 | F | 127.6 | thigh | 39.2 | 6.23 | 1.7 | 14.6 | 36.1 | 1.00 | 63.6 | 48.8 | 3.1 |
| 8 | F | 127.6 | calf | 25.6 | 4.08 | 1.7 | 9.0 | 20.6 | 0.94 | 25.6 | 19.6 | 2.8 |
| 10 | F | 138.0 | arm | 20.4 | 3.25 | 1.8 | 4.9 | 17.2 | 0.86 | 18.6 | 8.7 | 2.3 |
| 10 | F | 138.0 | thigh | 42.5 | 6.76 | 1.8 | 15.7 | 39.4 | 1.08 | 75.3 | 57.3 | 3.6 |
| 10 | F | 138.0 | calf | 27.6 | 4.39 | 1.8 | 9.7 | 22.2 | 1.02 | 29.8 | 22.7 | 3.3 |
| 12 | F | 151.2 | arm | 22.7 | 3.61 | 1.9 | 5.7 | 19.1 | 0.94 | 22.8 | 11.2 | 2.8 |
| 12 | F | 151.2 | thigh | 46.3 | 7.36 | 1.9 | 16.9 | 43.1 | 1.18 | 90.1 | 67.2 | 4.4 |
| 12 | F | 151.2 | calf | 29.9 | 4.76 | 1.9 | 10.4 | 24.2 | 1.12 | 35.4 | 26.4 | 3.9 |
| 14 | F | 160.4 | arm | 24.5 | 3.91 | 2.0 | 6.4 | 20.7 | 0.99 | 26.3 | 13.7 | 3.1 |
| 14 | F | 160.4 | thigh | 52.3 | 8.32 | 2.0 | 19.4 | 49.4 | 1.25 | 115.3 | 87.0 | 4.9 |
| 14 | F | 160.4 | calf | 32.7 | 5.21 | 2.0 | 11.5 | 26.7 | 1.19 | 42.4 | 32.0 | 4.4 |
| 16 | F | 162.5 | arm | 26.0 | 4.14 | 2.0 | 7.0 | 22.3 | 1.01 | 29.8 | 15.8 | 3.2 |
| 16 | F | 162.5 | thigh | 53.8 | 8.56 | 2.0 | 20.4 | 50.5 | 1.27 | 120.4 | 94.2 | 5.0 |
| 16 | F | 162.5 | calf | 33.8 | 5.38 | 2.0 | 12.2 | 27.6 | 1.20 | 44.7 | 35.0 | 4.5 |
| 18 | F | 163.1 | arm | 26.9 | 4.28 | 2.0 | 8.4 | 22.3 | 1.01 | 29.8 | 19.3 | 3.2 |
| 18 | F | 163.1 | thigh | 54.1 | 8.62 | 2.0 | 21.3 | 50.2 | 1.27 | 119.2 | 98.3 | 5.1 |
| 18 | F | 163.1 | calf | 33.9 | 5.40 | 2.0 | 12.7 | 27.3 | 1.21 | 44.0 | 36.3 | 4.6 |
| 25 | F | 163.1 | arm | 31.0 | 4.93 | 2.0 | 8.5 | 28.7 | 1.01 | 44.1 | 23.0 | 3.2 |
| 25 | F | 163.1 | thigh | 61.5 | 9.79 | 2.0 | 22.1 | 61.0 | 1.27 | 165.6 | 118.0 | 5.1 |
| 25 | F | 163.1 | calf | 37.2 | 5.92 | 2.0 | 12.8 | 32.3 | 1.21 | 57.3 | 40.8 | 4.6 |
| 45 | F | 161.8 | arm | 32.4 | 5.15 | 2.0 | 10.5 | 29.0 | 1.00 | 44.6 | 29.2 | 3.2 |
| 45 | F | 161.8 | thigh | 62.5 | 9.94 | 2.0 | 23.6 | 61.2 | 1.26 | 166.1 | 126.9 | 5.0 |
| 45 | F | 161.8 | calf | 37.2 | 5.92 | 2.0 | 13.4 | 31.8 | 1.20 | 55.8 | 42.6 | 4.5 |
| 65 | F | 159.8 | arm | 31.6 | 5.04 | 1.9 | 10.1 | 28.4 | 0.99 | 43.1 | 27.5 | 3.1 |
| 65 | F | 159.8 | thigh | 60.7 | 9.66 | 1.9 | 24.7 | 57.6 | 1.25 | 149.4 | 127.8 | 4.9 |
| 65 | F | 159.8 | calf | 35.2 | 5.60 | 1.9 | 13.6 | 28.7 | 1.18 | 47.1 | 40.3 | 4.4 |
| 80 | F | 156.2 | arm | 28.9 | 4.60 | 1.7 | 8.3 | 26.3 | 0.97 | 37.7 | 20.9 | 2.9 |
| 80 | F | 156.2 | thigh | 57.8 | 9.20 | 1.7 | 24.8 | 53.3 | 1.22 | 130.0 | 121.4 | 4.7 |
| 80 | F | 156.2 | calf | 32.8 | 5.22 | 1.7 | 13.3 | 25.6 | 1.16 | 39.3 | 36.7 | 4.2 |

### Mean subcutaneous fat thickness over the whole body [computed]

| age | sex | H cm | W kg | %BF | FM kg | VAT kg | SAT kg | BSA m2 | mean SAT mm |
|---|---|---|---|---|---|---|---|---|---|
| 0.5 | M | 67.6 | 7.9 | 25 | 2.0 | 0 | 1.8 | 0.39 | 5.3 |
| 1 | M | 75.7 | 9.6 | 23 | 2.2 | 0 | 2.1 | 0.45 | 5.1 |
| 2 | M | 86.5 | 12.4 | 21 | 2.6 | 0 | 2.4 | 0.55 | 4.9 |
| 3.5 | M | 98.7 | 15.5 | 18 | 2.8 | 0 | 2.5 | 0.65 | 4.3 |
| 6 | M | 115.4 | 20.5 | 16.0 | 3.3 | 0 | 3.0 | 0.81 | 4.0 |
| 8 | M | 127.9 | 25.7 | 17.0 | 4.4 | 0 | 3.9 | 0.96 | 4.6 |
| 10 | M | 138.6 | 31.9 | 17.8 | 5.7 | 0 | 5.1 | 1.11 | 5.1 |
| 12 | M | 149.1 | 39.1 | 17.4 | 6.8 | 0 | 6.1 | 1.27 | 5.3 |
| 14 | M | 163.8 | 51.2 | 16.2 | 8.3 | 0 | 7.3 | 1.53 | 5.3 |
| 16 | M | 173.5 | 61.7 | 15.5 | 9.6 | 0 | 8.4 | 1.72 | 5.4 |
| 18 | M | 176.2 | 68.0 | 15.4 | 10.5 | 0 | 9.2 | 1.82 | 5.6 |
| 25 | M | 176.7 | 78.1 | 17.7 | 13.8 | 0.35 | 12.4 | 1.96 | 7.0 |
| 45 | M | 176.2 | 85.4 | 21.2 | 18.1 | 1.09 | 15.6 | 2.04 | 8.5 |
| 65 | M | 173.4 | 85.7 | 24.0 | 20.6 | 1.8 | 17.1 | 2.03 | 9.4 |
| 80 | M | 170.3 | 78.6 | 27.0 | 21.2 | 2.15 | 17.4 | 1.93 | 10.0 |
| 0.5 | F | 65.7 | 7.3 | 26 | 1.9 | 0 | 1.8 | 0.36 | 5.4 |
| 1 | F | 74.0 | 9.0 | 24 | 2.2 | 0 | 2.0 | 0.43 | 5.2 |
| 2 | F | 85.0 | 11.8 | 22 | 2.6 | 0 | 2.4 | 0.53 | 5.0 |
| 3.5 | F | 97.4 | 14.9 | 19 | 2.8 | 0 | 2.6 | 0.63 | 4.5 |
| 6 | F | 114.7 | 20.0 | 19.1 | 3.8 | 0 | 3.4 | 0.80 | 4.8 |
| 8 | F | 127.6 | 25.7 | 21.2 | 5.5 | 0 | 4.9 | 0.95 | 5.7 |
| 10 | F | 138.0 | 32.0 | 22.8 | 7.3 | 0 | 6.6 | 1.11 | 6.6 |
| 12 | F | 151.2 | 41.2 | 23.6 | 9.7 | 0 | 8.6 | 1.31 | 7.3 |
| 14 | F | 160.4 | 49.7 | 24.0 | 11.9 | 0 | 10.5 | 1.49 | 7.8 |
| 16 | F | 162.5 | 53.9 | 24.3 | 13.1 | 0 | 11.5 | 1.56 | 8.2 |
| 18 | F | 163.1 | 56.7 | 24.6 | 13.9 | 0 | 12.3 | 1.60 | 8.5 |
| 25 | F | 163.1 | 63.8 | 27.9 | 17.8 | 0.17 | 16.2 | 1.70 | 10.6 |
| 45 | F | 161.8 | 70.7 | 31.4 | 22.2 | 0.41 | 20.0 | 1.78 | 12.5 |
| 65 | F | 159.8 | 74.1 | 36.2 | 26.8 | 0.9 | 23.8 | 1.81 | 14.6 |
| 80 | F | 156.2 | 67.1 | 41.1 | 27.6 | 1.12 | 24.3 | 1.71 | 15.8 |

Inputs: H and BMI from the stage tables of `proportions_by_age.md`; %BF from the recommended curve in Section 1; VAT from Ofenheimer; SAT share as in Section 5. Cross-check: infant fat mass 2.0 kg at 0.5 y matches the MIBCRS median 1.9 kg [snippet].

### Fat-layer anchor thickness from skinfolds (SF/2 − skin, mm; shape only, not absolute) [computed from fetched WHO and LIFE medians]

| age y | M triceps | M subscap | M suprailiac | M biceps | F triceps | F subscap | F suprailiac | F biceps |
|---|---|---|---|---|---|---|---|---|
| 0.5 | 3.4 | 2.4 | – | – | 3.4 | 2.4 | – | – |
| 1 | 2.8 | 1.9 | – | – | 2.7 | 2.0 | – | – |
| 2 | 2.4 | 1.6 | – | – | 2.5 | 1.7 | – | – |
| 3.5 | 3.2 | 1.4 | 0.8 | 1.4 | 3.4 | 1.5 | 1.4 | 1.5 |
| 6 | 2.9 | 1.1 | 0.8 | 1.0 | 3.2 | 1.3 | 1.5 | 1.2 |
| 8 | 3.2 | 1.3 | 1.1 | 1.1 | 3.8 | 1.7 | 2.0 | 1.6 |
| 10 | 4.2 | 2.0 | 1.8 | 1.6 | 4.9 | 2.4 | 3.2 | 2.2 |
| 12 | 5.0 | 2.8 | 2.8 | 2.0 | 5.7 | 3.2 | 4.4 | 2.6 |
| 14 | 4.4 | 3.2 | 3.3 | 1.5 | 6.4 | 4.0 | 5.5 | 2.8 |
| 16 | 3.5 | 3.4 | 3.7 | 0.8 | 7.0 | 4.3 | 6.0 | 2.8 |
| 17 | 3.0 | 3.5 | 3.9 | 0.5 | 7.2 | 4.3 | 6.0 | 2.7 |

These are 2 to 4 times smaller than the volume-based mean thickness, because calipers compress fat and because the caliper sites are not the thick sites (buttock, lower abdomen, thigh). Ultrasound in pubertal girls gives 16.7 mm at the thigh and abdomen (Derraik), versus 7 mm from triceps SF/2. So scale the map by volume.

### Inferences
- **Inputs used** [inference unless marked]: stage H, thigh/H and calf/H from the sister note [computed there]; MUAC: WHO medians to 3.5 y [fetched], Snyder upper-arm/H for 6 to 18 y (boys' ratio also used for girls), adults 0.190·H (men 25 y), 0.192 (45), 0.188 (65), 0.180 (80); women 0.190, 0.200, 0.198, 0.185 [inference; NHANES tables not read]. Triceps medians: WHO (0.5 to 2 y), LIFE (3.5 to 16 y), KiGGS (18 y) [fetched]; adults 12/21 (25 y), 13/25 (45), 13.5/24 (65), 12/20 (80) mm M/F [inference anchored on the NHANES III snippet]. Leg fat fraction: Schafmeyer (8 to 18 y), Ofenheimer (adults) [computed]; 1 to 6 y set to 0.33 to 0.40 (boys) and 0.36 to 0.41 (girls) [inference]. Skin thickness as in Section 4.
- **Per-stage fat-thickness map recipe for HeroBody** [inference]:
  1. For each stage anchor (0.5, 1, 2, 3.5, 6, 8, 10, 12, 14, 16, 18, 25, 45, 65, 80 y) and sex, compute V_SAT (table above) and the region ratios: arm = triceps, back = subscapular, flank = suprailiac, anterior arm = biceps (children: LIFE table; adults: hold the 17 y ratios and multiply trunk regions by the DXA trunk/limb ratio relative to 20 y).
  2. Lower-body regions use the leg fat fraction route (thigh ring and calf ring from the cross-section table) and the android/gynoid ratio: buttock and lateral thigh = gynoid weight × 1.3; lower abdomen = android weight × 1.2 (women) or × 1.5 (men) [inference].
  3. Face: infants and toddlers get a thick buccal/cheek pad (about 1.5× the trunk mean, no data) [inference]; adults use the compartment layout; elderly reduce the deep medial cheek and temporal compartments by 20 to 40% and add jowl and submental sag (Section 4) [inference].
  4. Solve λ for volume closure; smooth; clamp against anatomy; store the map per stage. In-stage sliders interpolate the maps linearly between stage anchors (both ratio and λ), then re-close volume.
- **Consistency checks the solver can run** [inference]: (1) muscle area at mid-arm and calf within ±15% of the cross-section table for z_m = 0; (2) mean SAT thickness within ±10% of the volume table; (3) circumference error ≤ 3% (the HeroBody blueprint rule); (4) fat thickness ≥ 1 mm everywhere except over bone landmarks.

### Gaps
- The concentric-ring model ignores shape: real limbs are elliptical, fat is thicker posteriorly on the arm and laterally/medially on the thigh, and muscle bellies are eccentric. Upper-thigh fat in women is underestimated by a whole-leg fat fraction (true upper-thigh fat fraction is probably about 0.5).
- No direct pQCT/MRI validation tables by age were read; children under 8 y have no leg DXA.
- The thigh input jumps between 18 y (Snyder 1977 thigh/H 0.31 M, 0.33 F) and 25 y (ANSUR 0.35, 0.38) because the two sources measure at different levels and populations. That jump, not biology, makes the girls' thigh muscle go 119 → 166 cm². Smooth the thigh/H input across 16 to 25 y before using the table.
- Bone radius constants are inferred; the registered BodyParts3D bones should replace them.

---

## How this plugs into HeroBody

- **Build item: THE FAT LAYER (open).** Implement as layered offset surfaces (Section 5): skin mesh → per-vertex fat thickness attribute → muscle shell, with volume closure to V_SAT(a, s, z_f). New module, e.g. `hb/fatlayer.py`: `fat_volume(age, sex, z_f)`, `region_ratios(age, sex)`, `thickness_map(mesh, age, sex, z_f) → per-vertex mm`, `muscle_shell(mesh, map, clamp=anatomy)`. CPU only (numpy, bmesh, Geometry Nodes).
- **Reference tables as data files**: export the childsds LMS tables used here (WHO skinfolds/MUAC, KiGGS, LIFE skinfolds and thigh, Duran, Schafmeyer, Kirk, Ofenheimer) to JSON or CSV under the project data folder, with source and licence fields. Evaluate them with the LMS formula in Section 1 so a character keeps its own centile (z) across all ages; this fits "every character is solved from its own numbers".
- **Knob targets** (add to the 30 measurement knobs, as body-condition knobs in the character profile): `fatness_z` (FMI z for age/sex), `muscle_z` (LBMI/ALMI z), optional `fat_pattern` (android ↔ gynoid weight, default from sex and age), `laxity` (0 to 1, default from age). Map `fatness_z` and `muscle_z` onto Anny `weight` and `muscle` by regression against the measured circumferences (Phase 1 knob atlas), not by Anny's own age priors, which are extreme at 1 y.
- **Knob solver (height last)**: fit circumferences first with Anny weight/muscle, then run the fat layer and check the Section 6 consistency rules. Report muscle area, fat area and mean SAT per segment in the character profile so the human can verify numbers instead of meshes.
- **Ageing stages**: use the stage anchors of this note (0.5, 1, 2, 3.5, 6, 8, 10, 12, 14, 16, 18, 25, 45, 65, 80 y); the in-stage slider interpolates fat maps and muscle reference. Elderly: apply the muscle loss schedule (Section 3), the trunk/limb redistribution (Section 2), visceral volume, and the sag corrective (Section 4).
- **Anatomy clamp**: the existing clamp (1 to 5% of anatomy pokes out on extreme bodies) should now clamp BodyParts3D muscles to the muscle shell, not to the skin, and bones to the muscle shell minus 1 mm. That also fixes X-ray/cutaway shots: the cutaway shows skin (≈ 2 mm), fat (map), muscle, bone with plausible thickness for the age.
- **Garments**: the fat map changes outer shape only through the solved skin; garments keep following the skin as now.
- **Female organ rig binding and pose correctives**: fat thickness tells the corrective system where volume can slide (thick fat at buttock, abdomen, inner thigh = larger soft corrective; thin fat at knee, elbow, shin = bone-driven).

## Sources

- childsds R package data (WHO 2006, KiGGS, LIFE Child skinfold and circumference, UK 1990 body fat, Duran 2019, Schafmeyer 2022, Kirk 2021, Ofenheimer 2020): https://github.com/cran/childsds/tree/master/data ; documentation in https://github.com/cran/childsds/blob/master/R/data.R
- Duran et al. 2019, J Clin Densitom: https://linkinghub.elsevier.com/retrieve/pii/S1094695018302622
- WHO anthro R package: https://github.com/WorldHealthOrganization/anthro
- naver/anny (shape calibration priors): https://github.com/naver/anny/tree/main/src/anny/data/shape_calibration
- HIT (Keller et al. CVPR 2024) code, README, licence: https://github.com/MarilynKeller/HIT
- Kelly, Wilson & Heymsfield 2009, NHANES DXA reference values: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2737140/
- Ogden et al. 2011, body fat percentile curves for US children: https://pubmed.ncbi.nlm.nih.gov/21961617/
- Adiposity rebound and WHtR (J Nutr 2025, ASN news): https://nutrition.org/study-challenges-decades-old-puzzle-about-childhood-body-fat/
- Infant fat peak review: https://www.nature.com/articles/ejcn2015117
- Infant ADP birth to 4.5 months: https://www.nature.com/articles/pr2010136
- MIBCRS reference charts (Murphy-Alford et al. 2023): https://ajcn.nutrition.org/article/S0002-9165(23)07687-6/fulltext ; https://pmc.ncbi.nlm.nih.gov/articles/PMC11537950/
- Fomon et al. 1982 reference child: https://heronext.epa.gov/reference/184446
- Janssen et al. 2000: https://bioblast.at/index.php/Janssen_2000_J_Appl_Physiol
- Mitchell et al. 2012: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3429036/ ; table: https://pmc.ncbi.nlm.nih.gov/articles/PMC3429036/table/T1
- DXA ALM to skeletal muscle (Kim 2002; McCarthy 2023): https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9929067/
- Thigh CT young vs old men: https://pmc.ncbi.nlm.nih.gov/articles/PMC5111551
- Health ABC 5-year thigh CSA change (Verreijen 2019): https://pmc.ncbi.nlm.nih.gov/articles/PMC6408207
- Health ABC thigh fat and muscle change: https://pmc.ncbi.nlm.nih.gov/articles/2777469
- Kuk et al. 2009: https://scinapse.io/papers/1974545118
- Derraik et al. 2014 skin and subcutis thickness: https://pmc.ncbi.nlm.nih.gov/articles/PMC3897752/
- Gibney 2010 summary: https://pmc.ncbi.nlm.nih.gov/articles/4021299
- Filipino injection-site ultrasound: https://www.asean-endocrinejournal.org/index.php/JAFES/article/view/112
- Marran & Segal children skin thickness: https://scielo.org.za/pdf/sajch/v8n3/04.pdf
- Müller et al. 2016 8-site ultrasound: https://bjsm.bmj.com/content/50/1/45.abstract
- Störchle et al. 2018: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6214952/
- NHANES III older adults triceps: https://pmc.ncbi.nlm.nih.gov/articles/PMC1097739/table/T1
- Elderly skinfold decline (Brazil): https://scielosp.org/pdf/csp/2007.v23n12/2887-2895/pt
- McDowell et al. 2008 NHSR 10: https://www.cdc.gov/nchs/data/nhsr/nhsr010.pdf
- Fryar et al. 2021 anthropometric reference data 2015-2018: https://stacks.cdc.gov/view/cdc/100478
- Frisancho 1981: https://www.qxmd.com/r/6975564
- pQCT muscle/fat CSA definitions: https://onlinelibrary.wiley.com/doi/pdfdirect/10.1002/pbc.30705
- Jaworski & Graff pQCT lower leg (children): https://pmc.ncbi.nlm.nih.gov/articles/PMC8185261
- Jaworski & Graff pQCT forearm: https://www.ismni.org/jmni/article/18/237
- Rauch & Schoenau proximal radius: https://www.ismni.org/jmni/article/8/217
- Femur mid-shaft diameter: https://dirjournal.org/articles/femoral-shaft-bowing-with-age-a-digital-radiological-study-of-anatolian-caucasian-adults/56956
- Seidenari 2000 children skin ultrasound: https://iris.unimo.it/handle/11380/612798
- Facial skin thickness and echogenicity with age: https://medicaljournalssweden.se/actadv/article/view/14205
- Postmenopausal skin collagen and thickness: https://pubmed.ncbi.nlm.nih.gov/3120067/
- Skin mechanics review (1.13%/y): https://arxiv.org/pdf/1709.03752
- Cutometer and age: https://pmc.ncbi.nlm.nih.gov/articles/PMC6615427
- Deep medial cheek fat elastography: https://www.termedia.pl/Age-related-changes-in-elastographically-determined-strain-of-the-facial-fat-compartments-a-new-frontier-of-research-on-face-aging-processes,7,34200,0,1.html
- Rohrich & Pessa 2007 facial fat compartments: https://utsouthwestern.elsevierpure.com/en/publications/the-fat-compartments-of-the-face-anatomy-and-clinical-implication
- Komaritzan, Wenninger & Botsch 2021 Inside Humans: https://cg.cs.tu-dortmund.de/publications/2021-insidehumans.pdf
- Saito, Zhou & Kavan 2015 Computational Bodybuilding: https://users.cs.utah.edu/~ladislav/saito15computational/saito15computational.html
- Kadleček et al. 2016: https://users.cs.utah.edu/~ladislav/kadlecek16reconstructing/kadlecek16reconstructing.html
- Sister notes: `proportions_by_age.md`, `growth_stature_trajectories.md` (same folder)
