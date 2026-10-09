# Stature, weight and head growth over the whole lifespan: growth-curve models, LMS reference data, tracking, adult change, and a growth() design for HeroBody ageing

Research date: 2026-10-09. Researcher notes for the report writer.

Access notes. This session could reach only GitHub (github.com, raw.githubusercontent.com) and PyPI. All of these were BLOCKED, for both WebFetch and curl: who.int and cdn.who.int, cdc.gov (www, stacks, wwwn, ftp), pmc.ncbi.nlm.nih.gov and ncbi.nlm.nih.gov, europepmc.org, academic.oup.com, Wiley, Springer, BMJ/adc.bmj.com, ScienceDirect, MDPI, Frontiers, PLOS, bioRxiv, CRAN, rdrr.io, openlab.psu.edu, math.nist.gov, bd.dbio.uevora.pt, archive.unu.edu, Rothamsted repository, UCL Discovery, Wellcome Open Research, Wikipedia, web.archive.org. So:
- **Fetched (primary data and code):**
  - LMS tables for WHO 2006 (0-5 y), WHO 2007 (5-19 y), CDC 2000 (0-20 y, with the CDC extended-BMI sigma column) and UK90. These came as JSON inside the `rcpchgrowth` 4.6.5 wheel from PyPI (RCPCH, AGPL-3.0 code; the numbers are the WHO/CDC/UK90 values).
  - WHO's own z-score code from the official GitHub repos `WorldHealthOrganization/anthro` and `anthroplus`.
  - The SITAR model code and the Berkeley Growth Study raw data (66 boys, 70 girls, birth to 21 y, born 1928-29), from the CRAN mirror `cran/sitar` on GitHub.
  - NHANES 2009-2012 raw adult anthropometry with survey weights, from the CRAN mirror `cran/NHANES`.
- **Computed here:** every table marked "[fetched; computed here]" was calculated in Python from those files: Preece-Baines, JPA-2, ICP-like and double-logistic fits; tracking correlations; tempo effects; adult LMS by decade; the prototype growth() outputs.
- **Snippet only:** all journal facts (Preece-Baines 1978, Karlberg ICP, JPA-2, SITAR 2010, ALSPAC, Sorkin 1999/BLSA, Dey 1999, Farkas 1992, Bushby 1992, Wright & Cheetham 1999, Mei 2004, Khamis-Roche 1994, de Onis 2007, Cole & Green 1992). These come from search-result snippets and are tagged [snippet].
- Equations that I know from the literature but that no snippet confirmed are tagged [inference] with a note.

## 1. Mathematical growth-curve models for height from birth to adult

### Takeaway
- **Five families matter.**
  - Preece-Baines model 1 (PB1): 5 parameters, valid from about 2-3 y to adult.
  - JPA-2: 7 parameters, birth to adult.
  - Karlberg ICP: infancy + childhood + puberty components, birth to adult.
  - SITAR: one mean spline curve plus per-person size, timing and intensity shifts.
  - Double/triple logistic (Bock-Thissen): 1 y to adult.
- **Best fit to the brief.** HeroBody needs ONE smooth individual trajectory with final height and puberty timing as separate knobs. For that, the best choice is a hybrid:
  - **Shape:** WHO medians from 0 to 3 y, then PB1 in "fraction of adult height" form from about 5 y. PB1 has an explicit asymptote (h1 = final height), so final height becomes a pure multiplier.
  - **Timing:** a SITAR-style age warp τ(t) = t − Δ·s(t). The ramp s(t) was fitted here to the Berkeley data, so a late maturer is only slightly shorter in childhood, much shorter in early adolescence, and reaches the same adult height.
- **Why not just shift θ or D3.**
  - Shifting PB1's θ alone moves the whole curve. For a 1.5-year shift, that makes a late-maturing child about 6% too short at age 8.
  - Shifting JPA-2's D3 alone changes the spurt intensity a lot (peak velocity 12 vs 6 cm/yr for D3 ∓1.5 y) and has no childhood effect.
  - The Berkeley data show both are wrong.
- **Spurt sharpness.** Cross-sectional reference medians (WHO/CDC) blur the pubertal spurt: the median curve has a peak of about 7.3 cm/yr in boys. Real individuals peak at 8-10 cm/yr. A character's trajectory should use an individual-shaped spurt, not the raw median curve.

### Cited Findings

**Model equations** (t = age in years, h = stature in cm):

| Model | Equation | Parameters | Valid range |
|---|---|---|---|
| Preece-Baines 1 | h(t) = h1 − 2(h1 − hθ) / (exp(s0(t−θ)) + exp(s1(t−θ))) | h1 adult height; θ a time constant near the spurt; hθ = h(θ); s0, s1 pre-pubertal and pubertal rate constants | about 2 y to adult |
| PB family ODE | dh/dt = s(t)(h1 − h), with ds/dt = (s1 − s)(s − s0) for model 1 | — | — |
| JPA-2 | h(t) = A [1 − 1/(1 + ((t+E)/D1)^C1 + ((t+E)/D2)^C2 + ((t+E)/D3)^C3)], with E = prenatal duration of growth (about 0.75 y) | A adult height; D1-D3 time scales of the infancy, childhood and puberty terms; C1-C3 exponents | birth (even conception) to adult |
| ICP (Karlberg) | I(t) = a + b(1 − e^(−ct)); C(t) = b1 (t−t0) + b2 (t−t0)² for t > t0 (onset 0.5-1 y); P(t) = A / (1 + e^(−k(t − tv))) | — | birth to adult |
| SITAR | y_i(t) = α_i + h((t − β_i)·exp(γ_i)) [+ δ_i·t] | h = natural cubic spline mean curve; α size; β timing; γ intensity; δ rotation (optional) | usually about 5 y to adult |
| Double logistic (Bock et al. 1973) | h(t) = a1/(1 + e^(−b1(t−c1))) + (A − a1)/(1 + e^(−b2(t−c2))) | — | about 1 y to adult |

Evidence for these rows:
- **PB1.** Preece & Baines 1978 (Ann Hum Biol 5:1-24) say model 1 was "especially accurate and robust", with "only five parameters to describe growth in stature from age two to maturity". The closed form is the standard published form, confirmed by a course handout snippet. — [Preece & Baines 1978 PDF](https://bd.dbio.uevora.pt/Crescimento/Preece-Baines_Human_Growth.pdf) [snippet]; [SFU lab handout](https://www.sfu.ca/~dmackey/Lab%207%20Spring%202013%20Growth%20Curve%20Modeling.doc) [snippet]
- **PB family ODE.** Same source. — [Preece & Baines 1978](https://bd.dbio.uevora.pt/Crescimento/Preece-Baines_Human_Growth.pdf) [snippet]
- **JPA-2.** A later fitting paper names the terms: adult height a, time scales b1-b3, exponents c1-c3, and e = "estimated pre-natal duration of growth". The exact closed form is from my knowledge of Jolicoeur, Pontier & Abidi 1992 (Am J Hum Biol 4:461-468) and is not confirmed by any snippet. — [Wiley abstract](https://onlinelibrary.wiley.com/doi/abs/10.1002/ajhb.1310040405) [snippet]; [del Pino et al. 2020, achondroplasia JPA-2](https://doi.org/10.1515/jpem-2020-0298) [snippet]; closed form [inference]
- **ICP.** The three additive, partly overlapping components (infancy sharply decelerating; childhood starting at 6-12 months; puberty sigmoid) are [snippet]. Karlberg's later work fitted "a second-degree polynomial" to infancy+childhood from 3 y. The exact a, b, c parameterisation is my recollection of Karlberg 1987/1989 and is not verified. — [Karlberg 1989](https://onlinelibrary.wiley.com/doi/abs/10.1111/j.1651-2227.1989.tb11199.x) [snippet]; [Gambia ICP study](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7105771/) [snippet]; [Karlberg 1987 I](https://pubmed.ncbi.nlm.nih.gov/3604665/) [snippet]; exact formula [inference]
- **SITAR.** The R code builds `ex <- (x - b) * exp(c)` and `y = a + spline(ex) (+ d*x)`. — [sitar.R](https://raw.githubusercontent.com/cran/sitar/master/R/sitar.R) [fetched]; Cole, Donaldson & Ben-Shlomo 2010 IJE 39:1558-66 explains 99% of variance, RSD 6-7 mm, "matching the fit of individually-fitted PB curves" — [HAL record](https://hal.archives-ouvertes.fr/hal-00609750) [snippet]
- **Double logistic.** Standard form, [inference]; it was fitted here (see below).

**Typical parameter values, Preece-Baines 1, fitted to each Berkeley child, 2-21 y** (66 boys, 70 girls; mean ± SD; rms fit error 0.67 / 0.53 cm). [fetched; computed here] from [Berkeley data](https://raw.githubusercontent.com/cran/sitar/master/data/berkeley.rda):

| Parameter | Boys | Girls |
|---|---|---|
| h1 (cm) | 180.0 ± 6.6 | 166.6 ± 6.2 |
| hθ (cm) | 166.5 ± 5.9 | 154.3 ± 5.6 |
| s0 (1/yr) | 0.103 ± 0.007 | 0.120 ± 0.011 |
| s1 (1/yr) | 1.036 ± 0.170 | 1.001 ± 0.147 |
| θ (yr) | 14.11 ± 1.13 | 11.99 ± 0.91 |
| Age at peak height velocity, APHV (yr) | 13.47 ± 1.19 | 11.14 ± 0.91 |
| Peak height velocity, PHV (cm/yr) | 8.04 ± 1.19 | 7.37 ± 0.93 |
| Age at take-off (yr) | 9.74 ± 1.13 | 7.93 ± 0.85 |
| Velocity at take-off (cm/yr) | 4.92 ± 0.69 | 5.54 ± 0.65 |
| Height at take-off / at PHV (cm) | 138.7 / 161.7 | 128.1 / 148.4 |
| corr(θ, APHV) | 0.95 | 0.94 |
| corr(APHV, PHV) | −0.47 | −0.53 |
| corr(h1, APHV) | −0.15 | −0.10 |

Note: the Berkeley children were born 1928-29, of north European ancestry, and were tall for their era.

**JPA-2 fitted to each Berkeley child, 0-21 y** (medians; rms 0.60 / 0.64 cm). [fetched; computed here]

| | A | D1 | D2 | D3 | C1 | C2 | C3 | APHV (mean) | PHV (mean) |
|---|---|---|---|---|---|---|---|---|---|
| Boys | 179.7 | 3.16 | 9.13 | 13.71 | 0.58 | 3.32 | 21.05 | 13.82 | 8.69 |
| Girls | 166.3 | 2.51 | 8.27 | 11.42 | 0.63 | 3.63 | 16.85 | 11.53 | 7.49 |

D3 sits close to APHV and C3 (about 17-21) sets how sharp the spurt is. [fetched; computed here]

**Models fitted to the WHO median curve** (WHO 2006 + 2007; lengths below 2 y converted to height with −0.7 cm). These are "population-median" parameter sets. [fetched; computed here]

| Model (age range) | Boys | Girls | Fit error vs WHO median |
|---|---|---|---|
| PB1 fraction form, h1 = 1 (3-19 y), free | pθ = 0.9204, s0 = 0.0971, s1 = 0.8743, θ = 13.887 (→ APHV 13.11, PHV 7.28 cm/yr) | pθ = 0.9231, s0 = 0.1123, s1 = 0.8412, θ = 11.869 (→ APHV 10.76, PHV 6.63) | rms 0.22 / 0.13 cm |
| PB1 fraction form, s1 fixed at Berkeley individual mean | pθ = 0.9246, s0 = 0.1017, s1 = 1.036, θ = 13.988 (→ APHV 13.44, PHV 7.95) | pθ = 0.9297, s0 = 0.1209, s1 = 1.001, θ = 12.048 (→ APHV 11.27, PHV 6.97) | max 0.96 / 0.72 cm |
| ICP-like (0-19 y): a, b, c, t0, b1, b2, A, k, tv | 49.89, 30.02, 1.696, 0.98, 8.443, −0.2478, 25.14, 0.765, 13.43 | 49.24, 30.85, 1.456, 1.06, 8.376, −0.2584, 15.35, 0.901, 11.10 | rms 0.20 / 0.27, max 0.71 / 0.79 cm |
| Double logistic (1-19 y): a1, b1, c1, A, b2, c2 | 139.57, 0.280, 0.29, 179.02, 0.604, 12.79 | 124.40, 0.380, −0.02, 163.96, 0.607, 10.45 | rms 0.60 / 0.39 cm |
| JPA-2 (0-19 y): A, D1, D2, D3, C1, C2, C3 | 176.90, 2.894, 10.220, 13.319, 0.626, 3.864, 15.479 | 163.32, 2.380, 8.754, 11.245, 0.678, 3.939, 12.936 | rms 0.55 / 0.40, worst at birth (4.0 / 2.8 cm) |

In the ICP-like fit, the quadratic childhood term turns over at t0 + b1/(2|b2|), about 18 y. Clamp it after 18 y.

**Effect of moving only one timing parameter**, using the median Berkeley JPA-2 curve. [fetched; computed here]
- D3 −1.5 y: APHV 12.2 y and PHV rises to 12.0 cm/yr.
- D3 +1.5 y: PHV falls to 6.0 cm/yr and the peak almost disappears.
- In both cases height at 8 y changes by less than 0.2%.
- Real late maturers in the same data are 0.6% (boys) to 1.4% (girls) shorter at age 8 per year of APHV delay (see section 3).
- So D3 alone is a poor timing knob.

**Literature values for comparison:**
- Population studies fitted with PB1 report widely varying means. In a cross-sectional Colombian study: final height 170.8 / 157.9 cm, APHV 12.71 / 10.4 y, PHV 7.4 / 7.0 cm/yr. — [Colombia, PMC8485727](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8485727/) [snippet]
- Chile: APHV 13.6 / 11.0 y, PHV 6.4 cm/yr, h1 173.7 / 160.0 cm. — [Chilean PB1 study](https://andespediatrica.cl/index.php/rchped/article/download/2066/3204/22569) [snippet]
- A flood-exposure study: boys s0 0.11, s1 0.87, θ 14.4, hθ 152.5, h1 163.8; girls s0 0.12, s1 0.93, θ 11.07, hθ 145.07, h1 156.62. — [PMC12373937 Table 4](https://pmc.ncbi.nlm.nih.gov/articles/PMC12373937/table/Tab4) [snippet]
- Cross-sectional PB1 fits give lower PHV than longitudinal individual fits, because averaging across tempo blurs the spurt. [inference, supported by the WHO-median fit above: PHV 7.28 vs individual Berkeley 8.04 and ALSPAC 10.0 cm/yr]

### Inferences
- **Which model generates ONE adjustable individual trajectory best:**
  - **PB1:** best for the 3-21 y range. The asymptote h1 is exactly "adult height", and s1 is an intuitive "spurt sharpness" knob. Its weakness is the infancy range. Patch that with WHO medians.
  - **JPA-2:** one closed form from birth, but its timing parameter D3 also changes spurt intensity, and its birth fit is poor (up to 4 cm off on WHO medians).
  - **ICP:** the most "biological" (separate infancy/childhood/puberty). It is good for explaining stages, but it has 9 parameters and needs clamping.
  - **SITAR:** the best statistical framework for timing (β) and intensity (γ), but it needs the R package and a fitted spline. We reproduce its idea with a closed-form age warp instead.
- **Recommended:** fraction-of-adult-height template P(t) (WHO 0-3 y, PB1 from 5 y, smoothstep blend 3-5 y) × adult height, with a tempo warp. See section 6.
- **Spurt sharpness.** Use s1 = 1.0-1.04 (Berkeley individual means) as default. Expose a "spurt sharpness" knob, s1 in 0.85-1.4. The Berkeley SD of s1 is 0.15-0.17.

### Gaps
- Exact Karlberg ICP coefficients for Swedish children, and the exact JPA-2 notation, could not be read: Wiley was blocked.
- The original Preece-Baines 1978 tables of mean parameters were not read.
- No modern longitudinal dataset (ALSPAC, Harpenden) was available as raw data. Individual-curve parameters come from the 1930s Berkeley cohort. Modern children have similar fraction-of-adult curves (see section 3), but this is not proven for spurt sharpness.

## 2. Reference data with LMS parameters: WHO 0-5, WHO 5-19, CDC 2-20

### Takeaway
- **LMS formula** (Cole 1990; Cole & Green 1992):
  - X = M·(1 + L·S·Z)^(1/L) when L ≠ 0, and X = M·exp(S·Z) when L = 0.
  - Inverse: Z = ((X/M)^L − 1)/(L·S).
  - Interpolate L, M, S linearly between tabulated ages: WHO tables are daily (0-5 y) or monthly (5-19 y); CDC tables are half-monthly.
- **Coverage gaps:**
  - WHO gives height and BMI to 19 y, but weight-for-age only to 10 y.
  - So weight for a teen or adult must come from BMI × height².
  - WHO 0-2 y is recumbent length; height = length − 0.7 cm.
- **Tail rules:**
  - WHO uses a special linear rule beyond ±3 SD for weight and BMI.
  - CDC uses a "sigma" extension above the 95th BMI percentile.
- **Sources and licences:**
  - The CDC data are a US-government work. Free reuse is likely; check the CDC website policy.
  - WHO tables are WHO copyright with acknowledgment conditions.
  - For a later commercial release, CDC is the lower-risk source, or the user should ask WHO. [inference]

### Cited Findings
- **LMS (Cole & Green 1992, Stat Med 11:1305-1319):** after a Box-Cox power transform the measurement is Normal at each age. Z = ((y/M)^L − 1)/(L·S); limit Z = ln(y/M)/S at L = 0; centile y = M(1 + L·S·z)^(1/L). — [Cole & Green PDF](https://bd.dbio.uevora.pt/Crescimento/Smoothing_Reference_Centile_Curves_(LMS_Method).pdf) [snippet]
- **WHO's official R code:**
  - `compute_zscore <- ((y/m)^l - 1)/(s*l)`.
  - The adjusted version applies to weight-based indicators. If z > 3: z = 3 + (y − SD3pos)/(SD3pos − SD2pos). If z < −3: z = −3 + (y − SD3neg)/(SD2neg − SD3neg). Here SDk = M(1 + L·S·k)^(1/L).
  - Length/height rule: below 731 days, a standing height gets +0.7 cm; from 731 days, a recumbent length gets −0.7 cm.
  - — [anthro R/z-score-helper.R](https://raw.githubusercontent.com/WorldHealthOrganization/anthro/master/R/z-score-helper.R) [fetched]
- **WHO 2007 reference (de Onis et al., Bull WHO 85:660-667):**
  - Built by merging the NCHS/WHO 1977 data (1-24 y) with the under-5 standards sample (18-71 months), fitted with BCPE.
  - Smooth transition at 5 y.
  - Height-for-age and BMI-for-age run to 19 y; weight-for-age to 10 y.
  - At 19 y, +1 SD BMI = 25.4 (boys) / 25.0 (girls), and +2 SD = 29.7.
  - — [DOAJ record](https://doaj.org/article/d41a82701e01405fa1aa6f9fc82d985c) [snippet]
- **LMS tables actually read** (WHO 0-2 y daily, 731 rows; WHO 2-5 y daily, 1126 rows; WHO 5-19 y monthly, 169 rows for height/BMI and 61 rows for weight 5-10 y; CDC 2-20 y, 218-219 rows; CDC 0-36 months; UK90 4-20 y). — [rcpchgrowth 4.6.5 wheel on PyPI](https://pypi.org/project/rcpchgrowth/) [fetched]
- **Key LMS values** [fetched; computed here]:

| Age (y) | WHO boys height M (S) | WHO girls height M (S) | WHO boys BMI L / M / S | WHO girls BMI L / M / S | WHO boys weight M | WHO girls weight M |
|---|---|---|---|---|---|---|
| 0 | 49.9 (0.0380) | 49.1 (0.0379) | −0.305 / 13.41 / 0.0956 | −0.063 / 13.34 / 0.0927 | 3.35 | 3.23 |
| 0.5 | 67.6 (0.0317) | 65.7 (0.0345) | −0.191 / 17.3 / 0.0823 | −0.143 / 16.9 / 0.0904 | 7.9 | 7.3 |
| 1 | 75.7 (0.0314) | 74.0 (0.0348) | −0.412 / 16.8 / 0.0801 | −0.367 / 16.4 / 0.0880 | 9.6 | 8.9 |
| 2 (height) | 87.1 (0.0351) | 85.7 (0.0376) | −0.619 / 16.0 / 0.0779 | −0.568 / 15.7 / 0.0845 | 12.2 | 11.5 |
| 5 | 110.0 (0.0421) | 109.6 (0.0431) | −0.689 / 15.2 / 0.0870 | −0.568 / 15.3 / 0.0979 | 18.3 | 18.2 |
| 8 | 127.3 (0.0444) | 126.6 (0.0458) | −1.463 / 15.7 / 0.0953 | −1.388 / 15.7 / 0.1129 | 25.4 | 25.0 |
| 10 | 137.8 (0.0463) | 138.6 (0.0461) | −1.741 / 16.4 / 0.1057 | −1.486 / 16.6 / 0.1231 | 31.2 | 31.9 |
| 12 | 149.1 (0.0475) | 151.2 (0.0452) | −1.775 / 17.5 / 0.1152 | −1.401 / 18.0 / 0.1313 | — | — |
| 14 | 163.2 (0.0471) | 159.8 (0.0434) | −1.621 / 19.0 / 0.1219 | −1.227 / 19.6 / 0.1370 | — | — |
| 16 | 172.9 (0.0449) | 162.5 (0.0418) | −1.353 / 20.5 / 0.1258 | −1.037 / 20.7 / 0.1407 | — | — |
| 19 | 176.5 (0.0413) | 163.2 (0.0401) | −0.842 / 22.2 / 0.1295 | −0.750 / 21.4 / 0.1444 | — | — |

  WHO height L = 1 at all ages, so height is Normal.

- **CDC 2000 at 20 y** [fetched; computed here]:

| Measure | Male L / M / S | Female L / M / S |
|---|---|---|
| Height | 1.167 / 176.85 / 0.0404 | 1.108 / 163.34 / 0.0396 |
| Weight | −0.916 / 70.6 / 0.162 | −1.513 / 58.2 / 0.167 |
| BMI | −1.842 / 23.0 / 0.1345 | −2.345 / 21.7 / 0.153 |

  CDC infant OFC at birth is M = 35.8 (boys) / 34.7 (girls), against WHO's 34.5 / 33.9.

- **Peak in S.** WHO height S peaks at 12-13 y in boys (0.0476) and 9-10 y in girls (0.0461). This peak reflects differences in tempo across children. [fetched; computed here; interpretation inference]
- **CDC extended BMI** (above the 95th percentile): P95 = M(1 + L·S·1.645)^(1/L); percentile = 90 + 10·Φ((BMI − P95)/σ); BMI = P95 + σ·Φ⁻¹((p − 90)/10). The CDC 2-20 y BMI table carries `sigma` (e.g., 1.3756 for boys and 1.5714 for girls at 2 y). — [rcpchgrowth global_functions.py](https://pypi.org/project/rcpchgrowth/) [fetched]; [CDC extended BMI data page](https://www.cdc.gov/growthcharts/extended-bmi-data-files.htm) [snippet]
- **WHO download pages:**
  - 0-5 y length/height-for-age has Excel and PDF tables (z-scores, percentiles) for 0-13 weeks, 0-2 y and 2-5 y. — [WHO length/height-for-age](https://www.who.int/tools/child-growth-standards/standards/length-height-for-age) [snippet]
  - 5-19 y height-for-age has "Expanded tables for constructing national health cards": xlsx z-scores and percentiles for boys and girls, about 30 kB each, plus a computation PDF. — [WHO height-for-age 5-19](https://www.who.int/tools/growth-reference-data-for-5to19-years/indicators/height-for-age) [snippet]
  - Computation instructions, including the ±3 SD rule. — [WHO computation.pdf](https://cdn.who.int/media/docs/default-source/child-growth/growth-reference-5-19-years/computation.pdf) (URL as cited inside rcpchgrowth code [fetched]; page itself blocked)
- **CDC data files.** 8 Excel/CSV files: weight-, stature- and BMI-for-age 2-20 y; weight-, length-, weight-for-length and head-circumference-for-age 0-36 months. Each has L, M, S and selected percentiles. Unchanged since 30 May 2000. Infant files are named like WTAGEINF and LENAGEINF. — [CDC data files](https://www.cdc.gov/growthcharts/cdc-data-files.htm) [snippet]; [CDC data tables](https://www.cdc.gov/growthcharts/data_tables.htm) [snippet]. The exact CSV paths I believe exist (statage.csv, wtage.csv, bmiagerev.csv, hcageinf.csv, lenageinf.csv, wtageinf.csv under /growthcharts/data/zscore/) could not be verified [inference].
- **Terms of use:**
  - WHO holds copyright on the growth-standard publications.
  - The AnthroPlus software licence allows free copying "with an identification of the source" but "not in part nor for sale or for use in conjunction with any commercial or promotional purpose". Modification needs WHO permission. — [WHO AnthroPlus manual](https://cdn.who.int/media/docs/default-source/child-growth/growth-reference-5-19-years/who-anthroplus-manual.pdf) [snippet]
  - RCPCH's conditions for UK-WHO/UK90 require that WHO be acknowledged and forbid use for advertising; commercial printing needs permission. — [RCPCH terms](https://www.rcpch.ac.uk/resources/how-get-growth-charts-data-terms-conditions-use) [snippet]
  - Licences on the code (not the data): WHO `anthro` is GPL-3, `anthroplus` is GPL (≥ 3), `rcpchgrowth` is AGPL-3.0. — [anthro DESCRIPTION](https://raw.githubusercontent.com/WorldHealthOrganization/anthro/master/DESCRIPTION) [fetched]; [anthroplus DESCRIPTION](https://raw.githubusercontent.com/WorldHealthOrganization/anthroplus/master/DESCRIPTION) [fetched]; [rcpchgrowth METADATA](https://pypi.org/project/rcpchgrowth/) [fetched]

### Inferences
- **Do not ship AGPL code.** For HeroBody, download the original WHO xlsx and CDC CSV files on the user's Windows machine, where those hosts are probably reachable. Convert them once to a small JSON (`hb/data/growth_lms.json`) and write a ~20-line LMS evaluator. Do not ship rcpchgrowth: AGPL would infect the pipeline code.
- **Reference choice.** Use WHO for height 0-19 y (one family, smooth at 5 y) and CDC or WHO for BMI.
- **Weight past 10 y:** use BMI-for-age × height².
- **Height z-scores from WHO 19 y vs CDC 20 y** differ by at most 0.05 SD for normal heights (Luffy 174 cm: −0.35 vs −0.40; Shanks 199 cm: +3.08 vs +3.13).
- **Gap handling.** The WHO 0-2 and 2-5 tables leave a gap between 1.9986 y and 2.0014 y, and length vs height steps by 0.7 cm. Use the 2-5 y table from exactly 2.0 y and subtract 0.7 cm below 2 y. That gives a continuous standing-height curve.

### Gaps
- WHO and CDC web pages, and the current WHO licence wording (possibly CC BY-NC-SA 3.0 IGO), could not be read.
- No official adult LMS exists for height, weight or BMI after 19-20 y. Section 4 derives one from NHANES.

## 3. Individual tracking: canalization, catch-up, infancy channel shifts, mid-parental height, APHV, final-height prediction

### Takeaway
- **Infancy.** Children are NOT locked to one percentile in infancy. Birth length says almost nothing about adult height: r ≈ 0 to 0.3. Many infants cross two major percentiles before 2 y.
- **From about 3 y.** Height tracks strongly: r with adult height 0.7-0.86 at ages 3-10 in Berkeley. It dips at 12-14 y because of tempo differences, then reaches r ≈ 0.95-0.99 by 16-17 y.
- **Mid-parental height.** Tanner's formula is (mother + father ± 13)/2. The regression-adjusted version is expected child SDS ≈ 0.5 × mid-parental SDS, with a 90% band of ±1.4 SDS.
- **APHV:** about 13.5-13.9 y (SD 0.9-1.2) in boys and 11.1-12.0 y (SD 0.8-0.9) in girls. PHV is 8-10 cm/yr in boys and 7.4-7.7 cm/yr in girls.
- **Prediction methods:** Bayley-Pinneau (bone age), Tanner-Whitehouse, Roche-Wainer-Thissen and Khamis-Roche (no bone age). HeroBody needs none of these for designed characters, because adult height is given. Their logic ("percent of adult height reached at a given maturity") is exactly what section 6 uses.

### Cited Findings

**Tracking correlations and percent of adult height, Berkeley Growth Study** [fetched; computed here]. Adult height = PB1 asymptote h1. The last column is the regression of (height/adult) on APHV, which measures how much late maturers lag.

| Age (y) | Boys r(h, adult) | Boys mean % adult (SD) | Boys Δ% per +1 y APHV | Girls r(h, adult) | Girls mean % adult (SD) | Girls Δ% per +1 y APHV |
|---|---|---|---|---|---|---|
| 0 | — | — | — | −0.04 (n = 23) | 31.0 (2.3) | +0.16 |
| 1 | 0.33 | 42.3 (1.8) | +0.10 | 0.55 | 44.5 (1.7) | −0.66 |
| 2 | 0.59 | 49.1 (1.6) | −0.27 | 0.66 | 52.4 (1.6) | −0.47 |
| 3 | 0.73 | 53.7 (1.5) | −0.29 | 0.71 | 57.3 (1.5) | −0.48 |
| 5 | 0.76 | 61.7 (1.5) | −0.33 | 0.71 | 66.2 (1.9) | −0.73 |
| 6 | 0.78 | 65.3 (1.6) | −0.40 | 0.73 | 70.4 (2.0) | −1.04 |
| 8 | 0.82 | 72.3 (1.6) | −0.58 | 0.78 | 77.6 (2.0) | −1.38 |
| 10 | 0.86 | 78.5 (1.6) | −0.84 | 0.75 | 84.6 (2.5) | −2.20 |
| 12 | 0.78 | 84.5 (2.5) | −1.72 | 0.68 | 92.8 (3.2) | −3.02 |
| 13 | 0.72 | 88.2 (3.4) | −2.44 | 0.77 | 95.9 (2.6) | −2.32 |
| 14 | 0.70 | 92.1 (3.7) | −2.74 | 0.90 | 97.9 (1.7) | −1.35 |
| 15 | 0.73 | 95.5 (3.1) | −2.26 | 0.97 | 99.0 (0.9) | −0.66 |
| 16 | 0.85 | 97.7 (2.0) | −1.35 | 0.99 | 99.5 (0.5) | −0.33 |
| 17 | 0.95 | 98.8 (1.2) | −0.68 | 0.99 | 99.8 (0.4) | −0.19 |
| 18 | 0.98 | 99.4 (0.7) | −0.31 | 1.00 | 100.0 (0.3) | −0.10 |

Boys reach 50% of adult height at about 2 y and girls at about 1.75 y. The SD of percent-of-adult is about 1.5-2% in childhood and 3-3.7% at the peak of tempo spread.

- **Percent of adult height, modern vs old.** The modern WHO median fraction M(t)/M(19 y) agrees with the Berkeley "typical-timing individual" curve (21 boys and 34 girls with APHV within ±0.6 y of the mean) within 0.7 percentage points from 0.5 to 19 y. At 8 y: WHO 72.1% vs Berkeley 72.4% (boys); 77.6% vs 77.6% (girls). So fraction-of-adult curves are stable across 80 years of secular change. [fetched; computed here]
- **ALSPAC (SITAR, N = 5,707):** APHV 13.6 (SD 0.9) in males and 11.7 (0.8) in females; PHV 10.0 (1.1) and 7.7 (0.8) cm/yr. — [Frysz et al. 2018, PMC6171559](https://pmc.ncbi.nlm.nih.gov/articles/PMC6171559/) [snippet]
- **Harpenden (Cole 2020):** mean APV 12.0 y in girls and 13.9 y in boys; peak velocity "expressed as percent per year lay in the narrow range 4-8%". — [Cole 2020, Ann Hum Biol](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7391859/) [snippet]
- **Infancy shifts (Mei et al. 2004)**, California Child Health and Development Study, 10,844 children, CDC 2000 charts. The share crossing 2 major height percentiles was:
  - 32% between birth and 6 months
  - 13-15% between 6 and 24 months
  - 2-10% between 24 and 60 months
  - — [PubMed 15173545](https://www.ncbi.nlm.nih.gov/pubmed/15173545) [snippet]
- **Smith et al. 1976 (J Pediatr 89:225-230)** first described "shifting linear growth during infancy" as genetic potential replacing maternal and uterine influences. A review puts it at 40-50% of infants shifting significantly in the first 2 y. — [search snippet citing Smith 1976 and review](https://pmc.ncbi.nlm.nih.gov/articles/PMC1793269/?page=0) [snippet]
- **Wright & Cheetham 1999 (ADC 81:257):** the regression of child height SDS on mid-parental height SDS had slope 0.51. Rounded rule: expected SDS = 0.5 × mid-parental SDS. "Ninety per cent of children had values within 1.4 SDS of their expected SDS." — [ADC 81:257](https://adc.bmj.com/content/81/3/257) [snippet]
- **rcpchgrowth's implementation** (citing Tanner 1966 and Wright & Cheetham 1999):
  - Mid-parental height = (mother + father + 13)/2 for boys and (mother + father − 13)/2 for girls.
  - Mid-parental z = (z_mother + z_father)/4, using adult reference z at 19 y (WHO) or 20 y (UK).
  - Expected range = ±1.4 SDS.
  - — [rcpchgrowth mid_parental_height.py](https://pypi.org/project/rcpchgrowth/) [fetched]
  - The same code then multiplies by 0.5 again in `expected_height_z_from_mid_parental_height_z`. That looks like a double regression; treat it with care. [inference]
- **Khamis-Roche 1994 (Pediatrics 94:504-507):** a modified Roche-Wainer-Thissen model without skeletal age, from 223 boys and 210 girls in the Fels study. Errors are "only slightly larger" than RWT with bone age. Valid for white American children without growth pathology. — [PubMed 7936860](https://pubmed.ncbi.nlm.nih.gov/7936860/) [snippet]. A ±5.3 cm (90%) accuracy figure appears only on a calculator site [snippet, unverified].
- **Bayley-Pinneau and Roche-Wainer-Thissen** are the two height-prediction references implemented in rcpchgrowth (`height_predictions_constants.py`). — [rcpchgrowth](https://pypi.org/project/rcpchgrowth/) [fetched]

### Inferences
- **Designed characters.** The adult height is a fact from the character sheet. "Final height prediction" turns into "back-projection": child height(t) = adult × fraction(biological age).
- **Default tracking.** For a designed character, use perfect tracking from about 3 y (same fraction curve, scaled). Real children regress toward the mean (r ≈ 0.75-0.85), but a recognisable character should keep its rank: a tall adult was a tall child.
- **Infancy.** Shrink the size deviation toward the median: birth size keeps about 35% of the log-deviation, rising to about 76% at 1 y, 91% at 2 y and 97% at 3 y (k(t) = 1 − 0.65·e^(−t/1 y)). This mirrors r ≈ 0.3 → 0.6 → 0.7 in Berkeley and the infancy-shift literature. It stops a 199 cm Shanks from being a 55 cm (z ≈ +3.9) newborn.
- **Mid-parental height in profiles.** If parents are known (e.g., Garp and Dragon for Luffy's family), the profile can show a "plausibility" band: expected adult z = 0.5 × mean parental z ± 1.4. It should only warn, never override: sex is a fact and canon height wins.

### Gaps
- No raw modern longitudinal data with infancy were available. The infancy shrink factor k(t) is an assumption, set to match the literature r values. It has not been fitted to individual data.
- The Aberdeen Growth Study (Tanner et al. 1956) correlations could not be read (repository blocked).

## 4. Adult and old-age change: height loss, weight and BMI trajectories, sex differences

### Takeaway
- **Height (BLSA, Sorkin et al. 1999, 2,084 people aged 17-94):** loss starts at about 30 y and accelerates.
  - Cumulative loss 30 → 70 y: about 3 cm (men) and 5 cm (women).
  - By 80 y: 5 cm and 8 cm.
- **Other cohorts:**
  - Gothenburg, 70 → 95 y: −4.0 cm (men) and −4.9 cm (women).
  - ELSA, after 50 y: 0.08-0.10%/yr (men) and 0.12-0.14%/yr (women).
- **Where the loss happens:** in the trunk (sitting height). Leg length does not change with age. So in HeroBody, old-age height loss must be applied to the spine/neck segment and posture (kyphosis), NOT through the uniform height scale.
- **Weight and BMI:** rise from 20 y to a plateau at 40-60 y (men) or 60-75 y (women), then fall after about 70 y. Gothenburg, 70 → 95 y: −3.2 kg (men) and −5.1 kg (women).
- **The BMI artefact.** Part of the old-age BMI rise comes from shrinking height: +0.7 / +1.6 kg/m² by 70 y and +1.4 / +2.6 by 80 y (men / women).

### Cited Findings
- **Sorkin, Muller & Andres 1999, Am J Epidemiol 150:969** (Baltimore Longitudinal Study of Aging, 2,084 men and women aged 17-94, enrolled 1958-1993):
  - Height loss "began at about age 30 years and accelerated", and was faster in women.
  - Cumulative loss 30-70 y: about 3 cm (men) and 5 cm (women); by 80 y, 5 cm and 8 cm.
  - The resulting artifactual BMI increase is about 0.7 (men) and 1.6 (women) kg/m² by 70 y, and 1.4 and 2.6 by 80 y.
  - — [PubMed 10547143](https://pubmed.ncbi.nlm.nih.gov/10547143/) [snippet]
- **Dey et al. 1999, Eur J Clin Nutr 53:905-914** (Gothenburg, 449 men and 524 women first examined at 70 y in 1971-72, 11 examinations over 25 y):
  - Mean height fell 4.0 cm (men) and 4.9 cm (women) between 70 and 95 y.
  - From 70 to 75 y, the tallest quintile lost 0.4 (men) and 0.3 (women) cm/yr, and the shortest quintile 0.1 cm/yr.
  - Mean weight fell 3.2 kg (men) and 5.1 kg (women) between 70 and 95 y.
  - — [Nature EJCN 1600852](https://www.nature.com/articles/1600852) [snippet]
- **ELSA (Fernihough & McGovern 2015):** after 50 y, stature declines by 0.08-0.10%/yr (men) and 0.12-0.14%/yr (women), about 2-4 cm over the remaining life course. — [IFS page](https://ifs.org.uk/journals/physical-stature-decline-and-health-status-elderly-population-england) [snippet]
- **VA Normative Aging Study, Boston** (1,212 men aged 22-82): the 7.27 cm height gap between the youngest and oldest groups split into 4.27 cm from ageing and 3.00 cm from secular trend. This is why cross-sectional data overstate loss. — [Human Biology record](https://digitalcommons.wayne.edu/humbiol/vol61/iss3/8) [snippet; which Human Biology volume holds this result is uncertain]
- **Where the loss happens, cross-sectional evidence** (healthy adults aged 18-92): sitting height falls with age (r = −0.37 to −0.41); "leg length was independent of age in both sexes". — [Osteoporos Int 10.1007/s00198-003-1496-y](https://www.doi.org/10.1007/s00198-003-1496-y) [snippet]
- **Where the loss happens, longitudinal evidence** (Furano, Japan, 53 adults followed 34 years from mean age 44.4 to 78.6): mean loss 3.8 cm. Excess loss came from spinal kyphosis/scoliosis, more often in women. — [Shimizu et al. 2020, BMC Musculoskelet Disord](https://link.springer.com/10.1186/s12891-020-03464-2) [snippet]
- **BMI trajectory with age:**
  - ELSA: mean BMI rose up to the 60-69 band, then declined, faster in people who later died. — [BMJ Open 2022 record](https://doaj.org/article/25347604c16f468fbf0bbe37e45bc275) [snippet; attribution to this record is from the search result list]
  - Japan (8.15 M adults, 35-69 y): BMI rose in all age groups; in men 65+, weight fell but height loss still raised BMI. — [Int J Obes 2024](https://www.nature.com/articles/s41366-024-01694-1) [snippet]; [Keio press release](https://www.keio.ac.jp/en/press-release/20250212-1/) [snippet]
- **NHANES 2009-2012 adults, survey-weighted** (cross-sectional; includes secular trend; 80 = 80+). [fetched; computed here] from [NHANESraw.rda](https://raw.githubusercontent.com/cran/NHANES/master/data/NHANESraw.rda):

| Age (y) | Men height mean (SD) | Men weight mean / median | Men BMI median | Women height mean (SD) | Women weight mean / median | Women BMI median |
|---|---|---|---|---|---|---|
| 20-24 | 175.8 (7.9) | 81.2 / 77.3 | 24.9 | 162.7 (7.3) | 71.0 / 65.3 | 24.8 |
| 30-34 | 176.6 (7.5) | 88.9 / 84.9 | 27.4 | 163.8 (7.1) | 78.0 / 72.8 | 26.7 |
| 40-44 | 177.0 (7.9) | 92.4 / 88.8 | 28.4 | 162.8 (7.2) | 75.7 / 70.4 | 27.0 |
| 50-54 | 176.0 (7.5) | 92.0 / 89.0 | 28.5 | 161.9 (7.3) | 76.7 / 72.2 | 27.7 |
| 60-64 | 175.7 (7.9) | 90.1 / 85.9 | 28.0 | 161.7 (6.7) | 78.8 / 76.1 | 29.3 |
| 70-74 | 173.0 (7.3) | 86.2 / 84.9 | 28.2 | 159.7 (7.0) | 78.4 / 75.1 | 29.4 |
| 75-79 | 172.3 (7.2) | 84.2 / 82.4 | 27.8 | 159.0 (6.7) | 74.1 / 71.0 | 27.5 |
| 80+ | 171.2 (7.1) | 79.4 / 78.6 | 26.7 | 155.7 (6.5) | 64.1 / 62.9 | 26.1 |

- **Adult BMI LMS by decade, derived here** from NHANES 2009-2012 (weighted Box-Cox maximum likelihood; pregnant women excluded). [fetched; computed here]

| Age band | Men BMI L / M / S | Men weight L / M / S | Women BMI L / M / S | Women weight L / M / S |
|---|---|---|---|---|
| 20-29 | −0.63 / 25.8 / 0.197 | −0.39 / 81.0 / 0.216 | −0.99 / 25.2 / 0.226 | −0.78 / 67.8 / 0.243 |
| 30-39 | −0.87 / 27.6 / 0.190 | −0.58 / 86.6 / 0.217 | −0.84 / 27.3 / 0.232 | −0.85 / 72.4 / 0.241 |
| 40-49 | −0.74 / 28.4 / 0.178 | −0.51 / 88.1 / 0.194 | −0.64 / 27.7 / 0.241 | −0.68 / 72.8 / 0.245 |
| 50-59 | −0.60 / 28.3 / 0.185 | −0.25 / 88.6 / 0.208 | −0.62 / 28.1 / 0.227 | −0.58 / 73.4 / 0.232 |
| 60-69 | −0.65 / 28.0 / 0.191 | −0.29 / 87.1 / 0.218 | −0.29 / 29.0 / 0.219 | −0.07 / 75.5 / 0.233 |
| 70-79 | −0.23 / 28.0 / 0.160 | 0.07 / 83.5 / 0.179 | −0.34 / 29.0 / 0.233 | −0.07 / 73.5 / 0.243 |
| 80+ | 0.26 / 26.7 / 0.165 | 0.43 / 78.6 / 0.183 | −0.04 / 26.1 / 0.191 | 0.00 / 62.9 / 0.211 |

- **Newer official NHANES tables:** Fryar et al. 2021 (Vital Health Stat 3(46), NHANES 2015-2018, 18,061 people) gives means and percentiles by age; a 2025 edition (Series 3 No. 50, Aug 2021-Aug 2023) exists. — [CDC stacks 100478](https://stacks.cdc.gov/view/cdc/100478) [snippet]; [CDC stacks 174595](https://stacks.cdc.gov/view/cdc/174595) [snippet]

### Inferences
- **Height-loss schedule (cm/yr by decade), calibrated to BLSA** and capped after 80 y to match Gothenburg:

| Decade (y) | Men cm/yr | Women cm/yr | Men cumulative at decade end | Women cumulative at decade end |
|---|---|---|---|---|
| 30-40 | 0.03 | 0.05 | 0.3 | 0.5 |
| 40-50 | 0.06 | 0.10 | 0.9 | 1.5 |
| 50-60 | 0.08 | 0.14 | 1.7 | 2.9 |
| 60-70 | 0.12 | 0.18 | 2.9 (BLSA 3) | 4.7 (BLSA 5) |
| 70-80 | 0.20 | 0.30 | 4.9 (BLSA 5) | 7.7 (BLSA 8) |
| 80+ | 0.15 | 0.15 | 6.4 at 90 | 9.2 at 90 |

  Check: 70 → 95 y gives 4.25 cm (men) and 5.25 cm (women), close to Gothenburg's 4.0 and 4.9.

- **Profile modifiers** ("body conditions"): osteoporosis with vertebral fractures adds 2-6 cm of extra trunk loss plus kyphosis. "Fit elderly" uses half the rates. [inference; magnitudes not sourced]
- **Weight, two presets:**
  - "realistic US" uses the NHANES LMS above.
  - "lean / anime-realistic" keeps the WHO-19 BMI median (22.2 men, 21.4 women) and adds half of the NHANES rise: about +1.5 kg/m² by 45 y, then −1 to −2 kg/m² after 75 y. [inference]
- **Sex differences:**
  - Women lose height faster (about 1.6× the men's rate).
  - Men's weight peaks earlier (40-55 y) than women's (60-75 y).
  - Women's old-age weight loss is larger (−5.1 vs −3.2 kg, 70 → 95 y).
- **Use the shrunk height for weight.** NHANES BMI medians already include height loss, so weight = BMI(age) × h(age)² stays consistent.

### Gaps
- No primary source for per-decade weight change in kg within individuals was read in full.
- NHANES is cross-sectional. Old-age decline mixes ageing with survivor and cohort effects.
- No longitudinal Asian/Japanese adult height-loss table (relevant to anime-styled populations) was found.

## 5. Head circumference and head height from birth to old age

### Takeaway
- **Head circumference (OFC) is strongly front-loaded.**
  - At birth it is already about 60% of adult size. Stature at birth is 28-30%.
  - At 1 y: about 81% (stature 43-45%).
  - At 2 y: about 86% (stature 50-53%).
  - At 5 y: 92-93% (stature 62-67%).
  - At 10 y: 95-97% (stature 78-85%).
- **Absolute numbers:**
  - First-year gain: +11.6 cm (boys) and +11.0 cm (girls).
  - From 5 y to adult only +4.5 / +3.8 cm more (UK90).
  - Adult OFC is about 57.3 cm (men) and 55.5 cm (women) at the UK90 level. WHO-level references run about 2 cm (4%) lower at 4-5 y.
- **Head height** (vertex to chin) is less front-loaded than OFC, because the face grows late:
  - Farkas: cranial "head height" is near adult by 13 y.
  - Face height matures at about 15 y in boys and about 2 years earlier in girls.
  - Mandible height is only 66.6% of adult at 1 y.
- **"Heads tall"** (snippet-level sources only): about 4 at birth, about 5 at 2 y, about 6 at 5-6 y, about 7 at 15 y, 7-7.5 in adults.
- **Old age.** Adult OFC is usually treated as stable. No good longitudinal data on old-age change were found.

### Cited Findings
- **OFC trajectory:** WHO 2006 for 0-5 y; UK90 for 4-18 y (boys) and 4-17 y (girls). [fetched; computed here]

| Age (y) | Boys WHO M (S) | Boys UK90 M | Girls WHO M (S) | Girls UK90 M |
|---|---|---|---|---|
| 0 | 34.5 (0.0369) | — | 33.9 (0.0350) | — |
| 0.25 | 40.5 | — | 39.5 | — |
| 0.5 | 43.3 | — | 42.2 | — |
| 1 | 46.1 (0.0279) | — | 44.9 (0.0303) | — |
| 2 | 48.3 | — | 47.2 | — |
| 3 | 49.5 | — | 48.5 | — |
| 4 | 50.2 | 52.2 | 49.3 | 51.1 |
| 5 | 50.7 | 52.75 | 49.9 | 51.7 |
| 8 | — | 53.9 | — | 53.0 |
| 10 | — | 54.5 | — | 53.7 |
| 12 | — | 55.1 | — | 54.4 |
| 14 | — | 55.9 | — | 54.9 |
| 16 | — | 56.6 | — | 55.3 |
| 17 | — | 56.9 | — | 55.5 (S 0.025) |
| 18 | — | 57.3 (S 0.030) | — | — |

  OFC L = 1 at all ages (Normal). OFC gain [fetched; computed here]:

  | Interval (y) | Boys (cm) | Girls (cm) |
  |---|---|---|
  | 0-1 | +11.6 | +11.0 |
  | 1-2 | +2.2 | +2.3 |
  | 2-5 | +2.5 | +2.7 |
  | 5 to adult (UK90) | +4.5 | +3.8 |

- **Level mismatch.**
  - CDC 2000 infant OFC sits between: birth 35.8 / 34.7, 3 y 49.7 / 48.6.
  - The WHO → UK90 step at 4-5 y is about +2.0 cm, a known discontinuity in UK-WHO charts. [fetched; computed here; "known discontinuity" is inference]
- **Adult OFC depends on height** (Bushby et al. 1992, ADC 67:1286, 354 British adults). The mean head size of an average-height man lies above the 97th centile of the Tanner chart at 16 y, so paediatric charts are unsuitable for adult men. — [ADC 67:1286](https://adc.bmj.com/content/67/10/1286) [snippet]
- **Men's heads are about 1.33 cm larger than women's on average** (Turkish adult centiles, Örmeci 1997). Sources quoting "57 cm men / 55 cm women" are blogs. — [search result list](https://abakus.inonu.edu.tr/items/5702a7db-f6ea-4159-9881-370823a426d0/full) [snippet]
- **US OFC reference birth to 21 y** (Rollins, Collins & Holden 2010, J Pediatr 156:907-913): exists; values not read. — [em-consulte record](https://em-consulte.com/article/876532) [snippet]
- **Older adults:** OFC is affected by "skull thickness, temporal wasting, and estrogen deficiency", which "become more prominent" with age. OFC correlates with intracranial volume at r ≈ 0.63-0.73. — [PMC4669896](https://pmc.ncbi.nlm.nih.gov/articles/PMC4669896) [snippet]. A letter notes that adult skull circumference "is generally thought to stay stable" and that no study had measured change over time in older people. — [Cambridge core reader](https://www.cambridge.org/core/product/CA3BA47F810ADC8A29BF2C264FAF24B4/core-reader) [snippet]
- **Farkas, Posnick & Hreczko 1992** (Cleft Palate-Craniofac J 29):
  - Head (1,537 North American Caucasians, 1-18 y): adult head height is approached at 13 y (113.3 mm boys, 109.8 mm girls), with rapid growth of head height and length at 1-4 y. — [Head growth study](https://doi.org/10.1597/1545-1569_1992_029_0303_agsoth_2.3.co_2) [snippet]
  - Face (1,594 subjects): at 1 y, mandible width is 80.2% and mandible height 66.6% of adult. Face height keeps growing after 5 y and matures at 15 y in boys. Facial maturity comes 12-15 y in boys and about 2 years earlier in girls. — [Face growth study](https://journals.sagepub.com/doi/abs/10.1597/1545-1569_1992_029_0308_gpotfa_2.3.co_2) [snippet]
- **Head-to-body proportion:**
  - About 1/4 of body length at birth and about 1/7 in adults. — [Montreal Children's Hospital](https://montrealchildrenshospital.ca/health-info/true-or-false-by-age-six-childrens-bodies-are-proportionately-not-very-different-from-those-of-adults/) [snippet]
  - About 1/5 at 2 y and about 1/6 at 5-6 y. — educational pages [snippet, low authority]
- **Data sources for total head height:**
  - GB/T 26160-2010 (Chinese minors 4-17 y) defines total head height as vertex to chin, but the tables are paid. — [chinesestandard.net](https://www.chinesestandard.net/PDF.aspx/GBT26160-2010) [snippet]
  - Snyder et al. 1977 (4,127 US children, 2 weeks to 18 y, 87 measures) is free at NIST but was blocked here. — [NIST AnthroKids PDF](https://math.nist.gov/~SRessler/anthrokids/child77lnk.pdf) [snippet]

### Inferences
- **OFC model.** OFC(t) = OFC_adult × Q(t):
  - Q is the WHO 0-5 y shape, ramped by smoothstep from the WHO level at birth to the UK90 level at 4.5 y, then the UK90 shape to 18 y.
  - OFC_adult = 57.3 (men) / 55.5 (women) × (1 + S·head_z), with S = 0.030 / 0.025.
  - This gives 34.5 → 46.3 → 49.1 → 52.7 → 54.5 → 57.3 cm (boys at 0, 1, 2, 5, 10, 18 y).
  - Keep OFC flat after 20 y.
- **HeroBody head size.** HeroBody's head size comes from the blueprint, not from population OFC. So use Q(t) (or a head-height version of it) as the age multiplier relative to the character's adult head, not absolute centimetres.
- **Head-height model, proposed (to be fitted when data arrive):**
  - head_height(t) = cranial_adult × Q_ofc(t) + face_adult × Q_face(t).
  - Q_face(t) follows a stature-like curve that matures at 15 y (boys) / 13 y (girls), with Q_face(1 y) ≈ 0.67.
  - Adult split cranial/face ≈ 45/55 of vertex-to-chin.
  - This gives "heads tall" of about 4 at birth rising to about 7.3-7.6 in adults. Check it against the snippet milestones (≈5 at 2 y, ≈6 at 6 y).
- **Anime ranges.** Anime "head bigger than 2×" or chibi ranges are a style offset on top of this realistic curve. They belong to the per-character style layer (HeroBody's planned new blendshapes), not to the growth function.

### Gaps
- No primary table of total head height (vertex-menton) by age was read: Snyder 1977, Farkas 1994 and GB/T 26160 were not accessible.
- No adult head-height or OFC data by decade of old age were found.
- WHO vs UK90 OFC level mismatch: which adult OFC to use is a design choice. It matters little because the blueprint sets the adult head.

## 6. Concrete function design: growth(age_years, sex, height_z, timing_offset, weight_z) → stature_cm, weight_kg, head_circumference_cm

### Takeaway
- **Model in one line:** stature(t) = H_med × r^k(t) × P_sex(τ(t)) − loss_sex(t).
  - H_med is the adult median (WHO 19 y: 176.5 cm men, 163.2 cm women).
  - r = adult_height / H_med; adult_height is given in cm or derived from height_z.
  - k(t) is the infancy channel-shift share.
  - P_sex is the fraction-of-adult-height template.
  - τ(t) = t − Δ / (1 + e^(−(t − c)/w)) is biological age, with Δ = timing_offset (years, + = late).
  - loss is the adult height-loss schedule.
- **Weight:** BMI(age, weight_z) × (stature/100)². BMI comes from WHO BMI-for-age to 19 y, then a blend into adult NHANES-derived LMS by 30 y.
- **Head circumference:** OFC_median_sex(t) × (1 + S·head_z).
- **Luffy (174 cm at 19 y):** about 120.7 cm at 7 y, 135.6 at 10, 161.0 at 14, 172.9 at 17. Canon also gives 172 cm at 17; using both pins, the solver returns adult 174.3 cm and timing_offset +0.9 y (APHV ≈ 14.1 y, a slightly late maturer).
- **Shanks (199 cm, z = +3.08):** about 138.0 cm at 7 y and 184.1 cm at 14 y.
- **Non-human heights.** The method is ratio-based, so it also works for extreme heights where z-scores are meaningless (e.g., 300-600 cm giants).

### Cited Findings
- All tables used by the design are in sections 1-5. [fetched; computed here]
- **Prototype outputs** (code below). `infancy_shift = True` applies k(t); the other columns use full tracking. "const-z LMS" holds the WHO height z constant at all ages, for comparison. [fetched; computed here]

Luffy (male, adult 174.0 cm, WHO-19 z = −0.35, 36th percentile):

| Age | Stature | With infancy shift | Constant-z LMS | Weight (weight_z = 0) | BMI | OFC |
|---|---|---|---|---|---|---|
| 0 | 48.5 | 48.9 | 48.5 | 3.2 | 13.4 | 34.5 |
| 1 | 74.0 | 74.2 | 74.2 | 9.2 | 16.8 | 46.3 |
| 2 | 85.9 | 86.0 | 86.4 | 11.7 | 15.9 | 49.1 |
| 3 | 94.7 | 94.7 | 94.8 | 14.0 | 15.6 | 50.9 |
| 5 | 108.6 | 108.6 | 108.3 | 17.9 | 15.2 | 52.7 |
| 7 | 120.7 | 120.7 | 119.9 | 22.5 | 15.5 | 53.5 |
| 10 | 135.6 | 135.6 | 135.6 | 30.2 | 16.4 | 54.5 |
| 12 | 146.2 | 146.2 | 146.6 | 37.5 | 17.5 | 55.1 |
| 14 | 161.0 | 161.0 | 160.5 | 49.2 | 19.0 | 55.9 |
| 16 | 171.2 | 171.2 | 170.2 | 60.0 | 20.5 | 56.6 |
| 17 | 172.9 | 172.9 | 172.5 | 63.2 | 21.1 | 56.9 |
| 19 | 173.9 | 173.9 | 174.0 | 67.1 | 22.2 | 57.3 |
| 40 | 173.7 | | | 84.5 | 28.0 | 57.3 |
| 60 | 172.3 | | | 83.6 | 28.1 | 57.3 |
| 80 | 169.1 | | | 78.2 | 27.4 | 57.3 |
| 90 | 167.6 | | | 75.0 | 26.7 | 57.3 |

Shanks (male, adult 199.0 cm, z = +3.08, 99.9th percentile):

| Age | 0 | 0.5 | 1 | 2 | 3 | 5 | 7 | 10 | 12 | 13 | 14 | 15 | 16 | 17 | 19 | 80 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Stature, full tracking | 55.4 | 75.4 | 84.6 | 98.2 | 108.3 | 124.2 | 138.0 | 155.0 | 167.2 | 175.3 | 184.1 | 191.4 | 195.8 | 197.8 | 198.8 | 194.1 |
| With infancy shift | 51.3 | 72.0 | 82.2 | 97.2 | 107.9 | 124.1 | 138.0 | 155.0 | 167.2 | 175.3 | 184.1 | 191.4 | 195.8 | 197.8 | 198.8 | 194.1 |
| Constant-z LMS | 55.0 | 73.5 | 82.4 | 96.9 | 107.5 | 124.2 | 138.0 | 157.4 | 170.9 | 178.9 | 186.9 | 193.0 | 196.8 | 198.7 | 199.0 | — |

Nami (female, adult 170.0 cm, z = +1.05), full tracking: 0 y 50.5, 1 y 76.4, 3 y 99.0, 5 y 114.1, 7 y 126.5, 10 y 143.7, 12 y 157.7, 14 y 167.1, 16 y 169.6, 19 y 170.0, 60 y 167.1, 80 y 162.3, 90 y 160.8 cm.

- **Constant-z vs fraction.** Holding the LMS z constant makes tall characters even taller at 10-14 y (Shanks +2.4 to +3.7 cm). The reference S at 10-14 y is inflated by tempo spread. The fraction method plus an explicit tempo knob avoids double-counting tempo. [fetched; computed here; interpretation inference]
- **Tempo demo, boys** (adult 176.5 cm; heights in cm). [fetched; computed here]

| timing_offset | APHV (y) | PHV (cm/yr) | 6 y | 8 y | 10 y | 12 y | 13 y | 14 y | 15 y | 16 y | 18 y | 20 y |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| −2 | 12.31 | 9.44 | 117.8 | 129.7 | 140.9 | 155.9 | 165.1 | 171.7 | 174.8 | 175.9 | 176.4 | 176.5 |
| −1 | 12.82 | 8.67 | 117.2 | 128.8 | 139.2 | 151.9 | 160.3 | 168.1 | 173.0 | 175.2 | 176.3 | 176.5 |
| 0 | 13.45 | 7.95 | 116.6 | 127.7 | 137.5 | 148.3 | 155.4 | 163.3 | 169.8 | 173.6 | 176.1 | 176.4 |
| +1 | 14.21 | 7.34 | 115.9 | 126.7 | 135.9 | 145.1 | 150.9 | 157.8 | 164.9 | 170.6 | 175.5 | 176.4 |
| +2 | 15.13 | 6.92 | 115.3 | 125.7 | 134.2 | 142.2 | 146.8 | 152.4 | 158.9 | 165.7 | 174.0 | 176.1 |

- **Tempo demo, girls** (adult 163.2 cm; heights in cm). [fetched; computed here]

| timing_offset | APHV (y) | PHV (cm/yr) | 6 y | 8 y | 10 y | 11 y | 12 y | 13 y | 14 y | 16 y | 18 y |
|---|---|---|---|---|---|---|---|---|---|---|---|
| −2 | 9.98 | 8.28 | 118.6 | 131.3 | 146.1 | 153.9 | 159.2 | 161.7 | 162.7 | 163.1 | 163.2 |
| −1 | 10.56 | 7.58 | 117.2 | 129.0 | 141.9 | 149.4 | 156.0 | 160.0 | 162.0 | 163.0 | 163.2 |
| 0 | 11.27 | 6.97 | 115.8 | 126.8 | 138.0 | 144.6 | 151.4 | 157.0 | 160.4 | 162.8 | 163.1 |
| +1 | 12.12 | 6.50 | 114.3 | 124.6 | 134.3 | 139.9 | 146.1 | 152.4 | 157.5 | 162.1 | 163.0 |
| +2 | 13.11 | 6.23 | 112.8 | 122.3 | 130.9 | 135.5 | 140.8 | 146.8 | 152.9 | 160.7 | 162.8 |

- **Behaviour of the timing knob:**
  - Early maturers get taller in childhood and have a sharper spurt; late maturers the reverse.
  - This matches Berkeley corr(APHV, PHV) = −0.47 / −0.53 and the childhood lag table in section 3.
  - The APHV shift is about 0.6-0.85 per unit offset, because the fitted ramp is below 1 at APHV. Expose "target APHV" in the UI and solve for the offset numerically.

**Template table P_sex(t)** = fraction of adult stature (prototype: WHO 0-3 y, blend, PB1 with individual s1 from 5 y). [fetched; computed here]

| Age (y) | Boys % adult | Girls % adult | Age (y) | Boys % adult | Girls % adult |
|---|---|---|---|---|---|
| 0 | 27.9 | 29.7 | 9 | 75.2 | 81.0 |
| 0.25 | 34.4 | 36.2 | 10 | 77.9 | 84.5 |
| 0.5 | 37.9 | 39.9 | 11 | 80.7 | 88.6 |
| 1 | 42.5 | 44.9 | 12 | 84.0 | 92.8 |
| 1.5 | 46.2 | 49.0 | 13 | 88.1 | 96.2 |
| 2 | 49.5 | 52.8 | 14 | 92.5 | 98.3 |
| 3 | 54.4 | 58.3 | 15 | 96.2 | 99.3 |
| 4 | 58.4 | 62.9 | 16 | 98.4 | 99.7 |
| 5 | 62.4 | 67.1 | 17 | 99.4 | 99.9 |
| 6 | 66.0 | 70.9 | 18 | 99.8 | 100.0 |
| 7 | 69.4 | 74.4 | 19 | 99.9 | 100.0 |
| 8 | 72.4 | 77.7 | | | |

Median-character velocity (cm/yr) [fetched; computed here]:

| Age (y) | 0.5 | 1 | 2 | 3 | 5 | 8 | 10 | 12 | 13 | 14 | 15 | 16 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Boys | 19.4 | 14.3 | 10.4 | 7.9 | 6.8 | 5.1 | 4.8 | 6.4 | 7.7 | 7.6 | 5.2 | 2.6 |
| Girls | 19.4 | 14.7 | 10.6 | 8.3 | 6.6 | 5.3 | 6.2 | 6.5 | 4.5 | 2.4 | 1.0 | 0.4 |

Girls also peak at 6.9 cm/yr at 11 y.

### Inferences

**Function contract** (Python, CPU-only, microseconds per call):

```python
growth(age_years: float, sex: "male"|"female",
       height_z: float | None = None, adult_height_cm: float | None = None,  # one of the two
       timing_offset: float = 0.0,      # years; + = late maturer; UI may expose target APHV instead
       weight_z: float = 0.0,           # BMI-for-age z (WHO to 19 y, adult LMS after)
       head_z: float = 0.0,             # adult OFC z (S = 0.030 men / 0.025 women)
       spurt_s1: float | None = None,   # PB1 s1, default 1.036 boys / 1.001 girls
       infancy_shift: bool = True, loss_preset: str = "BLSA", bmi_preset: str = "NHANES")
 -> dict(stature_cm, weight_kg, bmi, head_circumference_cm, bio_age, trunk_loss_cm)
```

**Maths, step by step:**
1. **Adult height.** adult = adult_height_cm, or M19·(1 + L19·S19·height_z)^(1/L19). WHO 19 y: men L = 1, M = 176.54, S = 0.04134; women L = 1, M = 163.15, S = 0.04009. Store height_z = (adult/M19 − 1)/S19 for display only.
2. **Biological age.** τ = t − Δ / (1 + exp(−(t − c)/w)), with c = 11.70 and w = 2.64 (boys), c = 8.83 and w = 2.53 (girls). These are fitted to the Berkeley lag table.
3. **Fraction template P_sex(τ).**
   - For τ ≤ 3 y: (WHO median height(τ), lengths −0.7 cm below 2 y) / WHO median at 19 y.
   - For τ ≥ 5 y: P = 1 − 2(1 − pθ)/(exp(s0(τ − θ)) + exp(s1(τ − θ))), with PB_INDIV parameters (boys 0.9246, 0.1017, 1.036, 13.988; girls 0.9297, 0.1209, 1.001, 12.048).
   - Between 3 and 5 y: smoothstep blend.
4. **Infancy share.** k(t) = 1 − 0.65·exp(−t/1.0). Stature = H_med·r^k·P(τ), with r = adult/H_med (log-scale shrink of the size deviation in infancy).
5. **Adult loss.** Subtract the piecewise loss schedule (section 4) for t > 30. Report it also as trunk_loss_cm, so the rig shortens the spine and adds kyphosis instead of scaling the legs.
6. **BMI.**
   - t ≤ 19: WHO BMI-for-age LMS with weight_z (use the WHO ±3 SD rule if |z| > 3).
   - t ≥ 30: adult LMS by decade (section 4 table, interpolated at band midpoints 25, 35, ... 85).
   - 19 < t < 30: smoothstep blend.
   - Weight = BMI·(stature/100)².
7. **OFC.** OFC_median(t)·(1 + S·head_z), with the splice from section 5.

**Data tables needed** (put in `hb/data/growth_lms.json`):
- WHO height LMS, 0-19 y (daily to 5 y, monthly after).
- WHO BMI LMS, 0-19 y.
- WHO OFC LMS, 0-5 y.
- UK90 OFC M, 4-18 y (or CDC/Nellhaus if UK90 licensing is a concern).
- The adult BMI LMS by decade (section 4).
- The loss schedule.
- PB, RAMP and k constants.
- Total size about 200 kB.

**Back-computing a character from canon pins:**
- Given pins (age_i, height_i) from the character profile (canon heights, approved manga images at an age), solve for adult_height (and timing_offset if 2+ pins) by least squares.
- **Luffy:** pins (17 y, 172 cm) and (19 y, 174 cm) give adult = 174.34 cm and Δ = +0.90 y, so APHV ≈ 14.1 y and PHV ≈ 7.3 cm/yr. Resulting heights: 7 y 120.2, 10 y 134.4, 12 y 143.6, 14 y 156.4, 15 y 163.5, 16 y 168.9, 21 y 174.3 cm.
- With only one adult pin (Shanks 199 cm at 39 y), set Δ = 0 and add back the adult height loss for ages over 30. At 39 y, BLSA loss is about 0.3 cm, so the pre-loss adult height is about 199.3 cm.
- **Guardrails:**
  - Flag |Δ| > 2.5 y.
  - Flag any pinned child height whose implied fraction lies outside P(τ) ± 3·SD (SD ≈ 1.6% in childhood, 3.5% at 12-14 y). Treat such pins as a style override (manga exaggeration): show the realistic value next to the drawn one and let the human approve.
- **Every character is computed from its own numbers.** Nothing is copied from a pilot character: the inputs are sex, adult height, timing and weight z.

**Stage mapping for the profile:**
- Baby 0-2 y; child 2 y to puberty onset (τ = take-off: about 9.7 y boys, 7.9 y girls in Berkeley PB1); teen to 19 y; adult 19-65 y; old 65+.
- The in-stage slider maps linearly in chronological years, except the teen stage. There it should map in biological age τ, so a 50% slider position means half-way through the spurt for both early and late maturers.

**Prototype code** (tested here; ~110 lines; uses the LMS JSON and numpy). Key parts:

```python
PB_INDIV = {'male': (0.9246, 0.1017, 1.0360, 13.988), 'female': (0.9297, 0.1209, 1.0010, 12.048)}
RAMP     = {'male': (11.70, 2.64), 'female': (8.83, 2.53)}
LOSS_RATE = {'male':   [(30,.03),(40,.06),(50,.08),(60,.12),(70,.20),(80,.15)],
             'female': [(30,.05),(40,.10),(50,.14),(60,.18),(70,.30),(80,.15)]}

def lms_x(z, L, M, S):  return M*math.exp(S*z) if abs(L) < 1e-6 else M*(1+L*S*z)**(1/L)
def lms_z(x, L, M, S):  return math.log(x/M)/S if abs(L) < 1e-6 else ((x/M)**L-1)/(L*S)
def pb_frac(t, p):
    pth, s0, s1, th = p
    return 1 - 2*(1-pth)/(math.exp(s0*(t-th)) + math.exp(s1*(t-th)))
def who_height_median(t, sex):                     # standing-height equivalent
    if 1.998 < t < 2.0014: t = 2.0014
    M = WHO_HEIGHT_M(sex, min(max(t, 0), 19))      # interpolated from LMS table
    return M - 0.7 if t < 2 else M
def frac_adult(t, sex):
    Mad = who_height_median(19, sex); f_pb = pb_frac(max(t, 3), PB_INDIV[sex])
    if t <= 3: return who_height_median(t, sex)/Mad
    if t >= 5: return min(f_pb, 1.0)
    w = (t-3)/2; w = w*w*(3-2*w)
    return (1-w)*who_height_median(t, sex)/Mad + w*f_pb
def bio_age(t, sex, d):
    c, w = RAMP[sex]; return t - d/(1+math.exp(-(t-c)/w))
def size_share(t, k0=0.35, tau=1.0): return 1-(1-k0)*math.exp(-max(t, 0)/tau)
def stature(t, sex, adult_cm, d=0.0, infancy_shift=True):
    Hm = who_height_median(19, sex); r = adult_cm/Hm
    k = size_share(t) if infancy_shift else 1.0
    return Hm * r**k * frac_adult(max(bio_age(t, sex, d), 0), sex) - height_loss(t, sex)
```

The full version (with BMI, OFC, adult LMS and the canon-pin solver) was run in this session. The report writer can rebuild it from these notes and tables. All outputs above come from it.

**Extreme or non-human sizes:**
- Use the same fraction curve, r = adult/176.5 (or 163.2).
- Turn infancy_shift off or keep it (it only shrinks the newborn).
- Do not compute z.
- Puberty timing for non-humans (fish-men, giants, minks) is a profile field: species-specific APHV and spurt sharpness, defaulting to human. [inference]

### Gaps
- **Ramp calibration is approximate.** The model APHV shift is 0.6-0.85 per unit Δ, not 1. Childhood lag at 8-10 y is about 1.5× the Berkeley per-APHV-year value. A better fit would use SITAR random effects estimated on modern longitudinal data (ALSPAC is not public; the Berkeley data are 1930s).
- The infancy shrink k(t) is assumed, not fitted.
- Adult BMI presets are US 2009-2012. No "lean" reference was derived from data.

## How this plugs into HeroBody

- **Knob targets.**
  - growth() produces the age targets for the stature knob, which HeroBody applies LAST as a pure uniform scale.
  - It also produces targets for the weight/fat knobs, via BMI → the planned fat layer, and for the head-size knob, via the OFC fraction Q(t) times the blueprint's adult head size.
  - Old-age trunk_loss_cm goes to the spine-length and posture correctives, not to the uniform scale. Leg length does not change with age.
- **Build items:**
  - **Knob solver (height last):** per age it receives stature_cm. Fit all ratio knobs first (age-proportion targets come from the separate proportions track), then scale to stature_cm.
  - **Fat layer:** consumes BMI/weight_z per age.
  - **New blendshapes** (head > 2×, chibi legs): the growth curve sets the realistic head-to-height ratio. Style offsets sit on top, per character.
  - **Anny age knob:** breaks at the baby end. The stature targets for 0-2 y (48-88 cm) will need the new baby-end blendshapes, or a dedicated infant base, before the solver can reach them.
- **Files and stages:**
  - `hb/data/growth_lms.json`: WHO height, BMI and OFC LMS; UK90 OFC median; adult BMI LMS; loss schedule; PB, RAMP and k constants.
  - `hb/age/growth.py`: growth(), the canon-pin solver (least squares on adult_height and timing_offset) and the stage↔years mapping.
  - **Character profile fields:** sex (fact), adult height cm (canon), height pins [(age, cm, source)], timing_offset or target APHV, spurt_s1, weight_z per stage or a BMI preset, head_z, loss preset (BLSA / fit / osteoporotic), species APHV override.
  - **Profile verification step:** show the curve, the flags (|Δ| > 2.5, pins outside ±3 SD, z > 3 for humans) and the realistic-vs-drawn comparison for manga overrides. The human approves; no mesh edits.
- **2D-first link.**
  - For a stage blueprint, hb/measure2d.py measures the drawn heights in head units.
  - growth() supplies the realistic stature for that age, so the drawn head ratio can be checked against the age-appropriate realistic ratio. Anime deviations are allowed but recorded as style offsets.
  - The approval rule "body fit to blueprint: every ratio within 3%" stays unchanged. Stature is not a ratio and is set by growth() or the pins.
- **Data licensing.**
  - Ship your own JSON derived from the WHO xlsx and CDC CSV files (downloaded on the user's machine) with an acknowledgment string. Do not vendor AGPL rcpchgrowth or GPL anthro code.
  - For a future commercial release, prefer CDC (US government) tables or ask WHO and RCPCH (UK90) for permission. [inference]

## Sources

Fetched (primary data/code):
- rcpchgrowth 4.6.5 wheel (WHO/CDC/UK90 LMS JSON, mid-parental code, CDC extended BMI code): https://pypi.org/project/rcpchgrowth/ ; https://github.com/rcpch/rcpchgrowth-python
- WHO anthro R package (z-score helper, ±3 SD rule, 0.7 cm rule): https://raw.githubusercontent.com/WorldHealthOrganization/anthro/master/R/z-score-helper.R ; https://raw.githubusercontent.com/WorldHealthOrganization/anthro/master/DESCRIPTION ; https://github.com/WorldHealthOrganization/anthro
- WHO anthroplus R package: https://raw.githubusercontent.com/WorldHealthOrganization/anthroplus/master/README.md ; https://raw.githubusercontent.com/WorldHealthOrganization/anthroplus/master/DESCRIPTION
- SITAR code and Berkeley Growth Study data: https://raw.githubusercontent.com/cran/sitar/master/R/sitar.R ; https://raw.githubusercontent.com/cran/sitar/master/data/berkeley.rda ; https://raw.githubusercontent.com/cran/sitar/master/man/berkeley.Rd ; https://raw.githubusercontent.com/cran/sitar/master/DESCRIPTION
- NHANES 2009-2012 (R package NHANES): https://raw.githubusercontent.com/cran/NHANES/master/data/NHANESraw.rda ; https://raw.githubusercontent.com/cran/NHANES/master/man/NHANES.Rd
- ANSUR II subset (no head measures): https://raw.githubusercontent.com/cran/psyntur/master/man/ansur.Rd

Snippet only (search results; primary pages blocked):
- Preece & Baines 1978: https://bd.dbio.uevora.pt/Crescimento/Preece-Baines_Human_Growth.pdf
- SFU growth-curve lab handout: https://www.sfu.ca/~dmackey/Lab%207%20Spring%202013%20Growth%20Curve%20Modeling.doc
- PB1 population studies: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8485727/ ; https://pmc.ncbi.nlm.nih.gov/articles/PMC12373937/table/Tab4 ; https://andespediatrica.cl/index.php/rchped/article/download/2066/3204/22569
- JPA-2: https://onlinelibrary.wiley.com/doi/abs/10.1002/ajhb.1310040405 ; https://doi.org/10.1515/jpem-2020-0298
- Karlberg ICP: https://onlinelibrary.wiley.com/doi/abs/10.1111/j.1651-2227.1989.tb11199.x ; https://pubmed.ncbi.nlm.nih.gov/3604665/ ; https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7105771/ ; https://onlinelibrary.wiley.com/doi/10.1111/j.1651-2227.1989.tb11237.x
- SITAR 2010: https://hal.archives-ouvertes.fr/hal-00609750
- ALSPAC SITAR: https://pmc.ncbi.nlm.nih.gov/articles/PMC6171559/
- Cole 2020 Harpenden/ALSPAC: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7391859/
- Cole & Green 1992 LMS: https://bd.dbio.uevora.pt/Crescimento/Smoothing_Reference_Centile_Curves_(LMS_Method).pdf
- de Onis 2007 WHO reference: https://doaj.org/article/d41a82701e01405fa1aa6f9fc82d985c
- WHO pages: https://www.who.int/tools/child-growth-standards/standards/length-height-for-age ; https://www.who.int/tools/growth-reference-data-for-5to19-years/indicators/height-for-age ; https://cdn.who.int/media/docs/default-source/child-growth/growth-reference-5-19-years/computation.pdf ; https://cdn.who.int/media/docs/default-source/child-growth/growth-reference-5-19-years/who-anthroplus-manual.pdf
- CDC pages: https://www.cdc.gov/growthcharts/cdc-data-files.htm ; https://www.cdc.gov/growthcharts/data_tables.htm ; https://www.cdc.gov/growthcharts/extended-bmi-data-files.htm
- RCPCH terms: https://www.rcpch.ac.uk/resources/how-get-growth-charts-data-terms-conditions-use
- Wright & Cheetham 1999: https://adc.bmj.com/content/81/3/257
- Mei et al. 2004: https://www.ncbi.nlm.nih.gov/pubmed/15173545
- Infancy shifting (Smith 1976, review): https://pmc.ncbi.nlm.nih.gov/articles/PMC1793269/?page=0
- Khamis-Roche 1994: https://pubmed.ncbi.nlm.nih.gov/7936860/
- Aberdeen Growth Study 1956: https://repository.rothamsted.ac.uk/item/96w46
- Sorkin et al. 1999 BLSA: https://pubmed.ncbi.nlm.nih.gov/10547143/
- Dey et al. 1999 Gothenburg: https://www.nature.com/articles/1600852
- ELSA stature decline: https://ifs.org.uk/journals/physical-stature-decline-and-health-status-elderly-population-england
- VA Normative Aging Study (attribution uncertain): https://digitalcommons.wayne.edu/humbiol/vol61/iss3/8
- Sitting height vs age: https://www.doi.org/10.1007/s00198-003-1496-y
- Furano 34-year study: https://link.springer.com/10.1186/s12891-020-03464-2
- ELSA BMI with age (attribution from result list): https://doaj.org/article/25347604c16f468fbf0bbe37e45bc275
- Japanese 8.15 M adults: https://www.nature.com/articles/s41366-024-01694-1 ; https://www.keio.ac.jp/en/press-release/20250212-1/
- NHANES reference reports: https://stacks.cdc.gov/view/cdc/100478 ; https://stacks.cdc.gov/view/cdc/174595
- Bushby et al. 1992 adult OFC: https://adc.bmj.com/content/67/10/1286
- Örmeci adult OFC (Turkish): https://abakus.inonu.edu.tr/items/5702a7db-f6ea-4159-9881-370823a426d0/full
- Rollins et al. 2010 US OFC: https://em-consulte.com/article/876532
- OFC in older adults: https://pmc.ncbi.nlm.nih.gov/articles/PMC4669896 ; https://www.cambridge.org/core/product/CA3BA47F810ADC8A29BF2C264FAF24B4/core-reader
- Farkas 1992 head and face growth: https://doi.org/10.1597/1545-1569_1992_029_0303_agsoth_2.3.co_2 ; https://journals.sagepub.com/doi/abs/10.1597/1545-1569_1992_029_0308_gpotfa_2.3.co_2
- Head-body proportion: https://montrealchildrenshospital.ca/health-info/true-or-false-by-age-six-childrens-bodies-are-proportionately-not-very-different-from-those-of-adults/
- Head-face standard and child anthropometry: https://www.chinesestandard.net/PDF.aspx/GBT26160-2010 ; https://math.nist.gov/~SRessler/anthrokids/child77lnk.pdf
