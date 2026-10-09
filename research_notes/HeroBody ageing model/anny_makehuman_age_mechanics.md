# How Anny and MakeHuman model age, why it breaks at the baby end, and how to extend it

Research date: 2026-10-09. Researcher notes for the report writer.

**Access notes.** Fetched and read in full: the naver/anny repository (git clone of `main`, which is tag `v0.6.1`, commit d6fc027, 2026-09-28), the makehumancommunity/mpfb2 repository (clone, commit d0a32e5, 2026-10-04), the makehumancommunity/makehuman repository (clone of `master`, commit a8bc2d5, 2024-06-26), the WHO 0-5 y and CDC 2-20 y length/height and BMI LMS tables as JSON copies in github.com/ewheeler/pygrowup, and the GitHub issue list of naver/anny plus issues #15 and #23. I also **installed Anny 0.6.1 in a CPU-only venv (torch 2.14.1, no GPU) and ran it**. All numeric tables marked [computed] come from running the real Anny model or its data files. A NumPy re-implementation of the phenotype blending that I wrote reproduced Anny's statures exactly (for example 190.87 cm vs 190.9 cm for a young male at height 0.5), so it is correct.
Blocked: arxiv.org, alphaxiv.org, huggingface.co, semanticscholar.org, openaccess.thecvf.com, europe.naverlabs.com, who.int, cdc.gov and pmc.ncbi.nlm.nih.gov (WebFetch DNS errors; proxy 403 for curl). The `gh` GitHub REST API was refused for naver/anny. **The web-search budget for this run was used up before my first search.** So I have **no fetched text from the Anny paper (arXiv 2511.03589, ECCV 2026) or from the Anny-Fit paper (CVPR 2026 Findings)**, and no forum posts. Reference proportions for real children come from the sibling note `proportions_by_age.md` (NL4 Dutch LMS tables and Snyder 1977, which that researcher fetched from GitHub mirrors). I cite those as "sibling note".

Evidence tags: [fetched] = I read the primary source or code. [computed] = I ran Anny code or data from a fetched source. [snippet] = search snippet only. [inference] = my reasoning. [background] = general knowledge, not verified in this session.

---

## 1. How the Anny age phenotype works (code level)

### Takeaway
Anny age is one scalar. It blends **five age anchors** with piecewise-linear weights. The anchors are `newborn, baby, child, young, old` at **age = -1/3, 0, 1/3, 2/3, 1**. The "newborn" anchor is **not a sculpted shape**. It is the MakeHuman baby target with a fixed anisotropic scale: 0.922 in width and depth, 0.75 in height. So between newborn and baby **nothing changes except size**. The 0-1 value has no built-in years. A separate "morphological age" table, calibrated to WHO height-for-age, maps years to Anny age: **0 y→0, 1 y→0.05, 4 y→0.215, 11 y→0.415, 16 y→0.67, 18 y→0.77, 64 y→0.83, 110 y→1.0**. The age prior for weight and muscle in that calibration code is indexed in the wrong units (Anny age is fed into tables keyed by years). That looks like a bug.

### Cited Findings
- Age variations are `["newborn", "baby", "child", "young", "old"]`. Gender is `["male", "female"]`, so **gender 0 = male**. Proportions are `["idealproportions", "uncommonproportions"]` and height is `["minheight", "maxheight"]` — [model_data.py L40-55](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/model_data.py#L40-L55) [fetched].
- Anchors: `age = torch.linspace(-1/3, 1.0, 5)` = {-0.3333, 0, 0.3333, 0.6667, 1.0}. Every other phenotype uses `linspace(0, 1, n)`: gender and height use 2 anchors (0, 1); muscle, weight, cupsize and firmness use 3 (0, 0.5, 1); proportions uses 2 — [phenotype.py L187-213](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/phenotype.py#L187-L213) [fetched].
- Interpolation: hat functions. For a value v between anchors a_{i-1} and a_i: alpha = (v − a_{i-1}) / (a_i − a_{i-1}), weight(a_{i-1}) = 1 − alpha, weight(a_i) = alpha, and all other weights are 0. When `extrapolate=False` (the default), v is clamped to [a_0, a_n] — [utils/interpolation.py L7-47](https://github.com/naver/anny/blob/v0.6.1/src/anny/utils/interpolation.py#L7-L47) [fetched]. In v0.6.1 the clamp was changed to `torch.where` so the gradient stays non-zero at anchors — [CHANGELOG v0.6.1](https://github.com/naver/anny/blob/v0.6.1/CHANGELOG.md) [fetched].
- Blend-shape coefficient: each stored blend shape carries a mask of the variation labels it belongs to, for example `universal:male-baby-averagemuscle-averageweight`. Its coefficient is the **product** of the hat weights of those labels: `wi = prod(phens * mask + (1 - mask))`. Race weights are normalised to sum to 1 (NaN → 1/3 each) — [phenotype.py L296-373](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/phenotype.py#L296-L373) [fetched]. Mesh: `V = T + Σ_i wi · B_i` (linear blend shapes).
- Blend-shape blocks loaded per age: universal (gender × age × muscle × weight), race (race × gender × age), height (gender × age × muscle × weight × min/max height), proportions (gender × age × muscle × weight × ideal/uncommon), **except for newborn and baby**, and breast (female × age × muscle × weight × cup × firmness), **only where the file exists, and never for newborn or baby** — [full_model.py L86-240](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/full_model.py#L86-L240) [fetched].
- The newborn construction, quoted from [full_model.py L91-95, L118-126](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/full_model.py#L91-L126) [fetched]:
  ```python
  # Newborn blend shapes are created as a scaled down version of the baby blend shapes
  newborn_blend_shape_scaling = torch.as_tensor([0.922, 0.922, 0.75])  # Empirical values
  normalizing_factor = 3.0  # the cumulated weight of newborn blend shapes when the age is set to newborn
  blend_shape = S * blend_shape + ((S - 1) / normalizing_factor) * template_vertices
  ```
  Three blocks (universal, race, height) each sum to weight 1 at age = −1/3. So the total is `S·(T + ΣB_baby)`: the whole baby body scaled by S = (0.922, 0.922, 0.75) in world (x, y, z), with z up. The scale is about the world origin, which in MakeHuman coordinates sits near the pelvis [computed / inference].
- World transform: MakeHuman decimetres, Y up → metres, Z up: `roma.Linear(0.1 * Rx(90°))` — [full_model.py L786-791](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/full_model.py#L786-L791) [fetched].
- **Morphological age (years) ↔ Anny age**, from `data/shape_calibration/boys.pth` and `girls.pth`. The two tables are identical, and the code asserts this. Interpolation is piecewise linear and extrapolates at both ends — [shape_distribution.py L17-51, L233-243](https://github.com/naver/anny/blob/v0.6.1/src/anny/shape_distribution.py#L17-L51) [fetched; values read from the .pth files, computed]:

  | years | 0 | 1 | 4 | 11 | 16 | 18 | 64 | 110 |
  |---|---|---|---|---|---|---|---|---|
  | Anny age | 0.000 | 0.050 | 0.215 | 0.415 | 0.670 | 0.770 | 0.830 | 1.000 |

- `SimpleShapeDistribution` is "a handcrafted 'morphological age' mapping, calibrated to match the height vs age distribution of WHO data". It samples morphological age ~ U(0, 90) y and gender ~ U(0, 1), then picks boys' tables when gender ≤ 0.5 — [shape_distribution.py L129-314](https://github.com/naver/anny/blob/v0.6.1/src/anny/shape_distribution.py#L129-L314) [fetched].
- Height-given-age prior: a Beta(α, β) on the `height` phenotype with anchors in **Anny-age units** [0, 0.05, 0.215, 0.415, 0.67]. Above 0.67 it is clamped (no extrapolation) [computed from .pth]:

  | Anny age (years) | boys α | boys β | boys mean | girls α | girls β | girls mean |
  |---|---|---|---|---|---|---|
  | 0.000 (0 y) | 21.596 | 58.601 | 0.269 | 29.521 | 96.722 | 0.234 |
  | 0.050 (1 y) | 51.978 | 57.360 | 0.475 | 66.282 | 74.541 | 0.471 |
  | 0.215 (4 y) | 22.147 | 45.859 | 0.326 | 31.816 | 59.491 | 0.348 |
  | 0.415 (11 y) | 13.405 | 22.105 | 0.377 | 21.264 | 23.975 | 0.470 |
  | 0.670 (16 y+) | 16.496 | 25.656 | 0.391 | 13.684 | 25.748 | 0.347 |

- Weight, muscle and proportions priors use anchors **[0, 1, 4, 11, 16, 18, 64, 110]**, which are years. Proportions is Beta(1, 1) everywhere, i.e. uniform. Weight at 1 y: boys Beta(0.273, 0.0094), mean 0.97; muscle at 1 y: boys Beta(0.010, 0.293), mean 0.03. So the calibration says **a 1-year-old needs weight ≈ 1 and muscle ≈ 0** [computed from .pth].
- But `sample()` and the `AnnyInverter` prior call `get_torch_distribution(age)` with **Anny age** (0-1), not years — [shape_distribution.py L254-301](https://github.com/naver/anny/blob/v0.6.1/src/anny/shape_distribution.py#L254-L301), [anny_inverter.py L1040-1055](https://github.com/naver/anny/blob/v0.6.1/src/anny/anny_inverter.py#L1040-L1055) [fetched]. Running it confirms the effect: sampled weight means drift from 0.57 to 0.84 and muscle means from 0.63 to 0.20 as Anny age goes 0 → 0.9. That equals linear interpolation between the "0 y" and "1 y" Beta parameters [computed]. → Likely bug [inference]: the weight and muscle priors are not really age-conditioned in years.
- `AnnyInverter` defaults: age regularisation weight 10.0 ("freeze or near-constant"), height 1e-3, and fitting starts at age = 0.8 ("adult average age") — [anny_inverter.py L20-32, L258-260](https://github.com/naver/anny/blob/v0.6.1/src/anny/anny_inverter.py#L20-L32) [fetched].
- `extrapolate_phenotypes=True` lets values go outside [0, 1]. The maintainer warns it "is likely to produce very deformed mesh" — [issue #15](https://github.com/naver/anny/issues/15) [fetched].

**WHO calibration check: real Anny, 6,000 samples from `SimpleShapeDistribution`, median stature (cm) vs WHO (0-4 y) / CDC (2-20 y) medians [computed]**

| age y | Anny boys | WHO/CDC boys | Anny girls | WHO/CDC girls |
|---|---|---|---|---|
| 0-0.5 | 63.1 | 49.9 (0 mo) / 67.6 (6 mo) | 58.7 | 49.1 / 65.7 |
| 1 | 74.0 | 75.7 | 75.2 | 74.0 |
| 2 | 86.5 | 87.1 | 84.2 | 85.7 |
| 4 | 103.0 | 103.3 | 100.2 | 102.7 |
| 6 | 114.6 | 115.7 | 113.7 | 115.0 |
| 8 | 129.9 | 128.1 | 127.1 | 127.8 |
| 11 | 141.6 | 143.7 | 145.6 | 144.3 |
| 13 | 157.6 | 156.4 | 152.7 | 157.3 |
| 16 | 173.9 | 173.6 | 162.0 | 162.6 |
| 18 | 177.4 | 176.2 | 165.8 | 163.1 |
| 25-35 | 175.6 | 176.8 (20 y) | 163.8 | 163.3 (20 y) |
| 80-90 | 176.1 | — | 162.7 | — |

WHO/CDC source: LMS JSON in [pygrowup tables](https://github.com/ewheeler/pygrowup/tree/master/pygrowup/tables) [fetched]. Stature calibration is good from 1 to 20 y (within about 2%). **At 0 y the calibrated body at its mean height parameter is 56.0 cm (boys) / 54.4 cm (girls), against 49.9 / 49.1 cm (+12%)**, and there is **no stature loss in old age** (80-90 y stays at 176 cm).

### Inferences
- Read in years, Anny's anchors sit here: **baby target ≈ 0 y** (not MakeHuman's 1 y), **child target ≈ 8.1 y**, **young target ≈ 15.9 y**, **old target ≈ 110 y**. The whole adult span 18-64 y occupies only 0.77-0.83. A 40-year-old is Anny age 0.799, which is **40% "old" target** (old weight = (0.799 − 0.667)/0.333).
- The newborn region (−1/3..0) is never used by Anny's own WHO calibration: 0 y maps to 0, and only negative years extrapolate there.
- The blend is C0 only. d(shape)/d(age) jumps at every anchor and at every calibration knot. An age slider animated through years will show visible speed changes at 0, 1, 4, 11, 16, 18 and 64 y.

### Gaps
- Not read: the Anny paper's description of how the morphological age table and the newborn scale (0.922/0.75) were chosen. The paper text was not reachable.

---

## 2. MakeHuman / MPFB2 macro targets and how age combines with the other macros

### Takeaway
MakeHuman has **four** sculpted age targets: baby, child, young and old. Nominally they are **1 y, 10 y (code) / ~11 y (common lore), 25 y and 90 y** at age = 0, 0.1875, 0.5 and 1.0. Age multiplies with gender, muscle and weight in a full tensor product of targets. Ethnicity (race) targets are per gender × age. Height and proportions targets are per gender × age × muscle × weight. MakeHuman ships **no baby proportions targets** and no baby breast targets.

### Cited Findings
- `macro.json` age parts: [−0.01, 0.1874998] baby→child; [0.1874999, 0.49998] child→young; [0.49999, 1.01] young→old — [mpfb2 macro.json](https://github.com/makehumancommunity/mpfb2/blob/master/src/mpfb/data/targets/macrodetails/macro.json) [fetched]. Linear interpolation inside each part — [targetservice.py L896-930](https://github.com/makehumancommunity/mpfb2/blob/master/src/mpfb/services/targetservice.py#L896-L930) [fetched].
- MakeHuman 1.x weights, quoted from [human.py L574-600](https://github.com/makehumancommunity/makehuman/blob/master/makehuman/apps/human.py#L574-L600) [fetched]:
  ```
  1y       10y       25y            90y
  baby    child     young           old
  0      0.1875      0.5             1
  if age < 0.5: old=0; baby=max(0,1-age*5.333); young=max(0,(age-0.1875)*3.2); child=max(0,min(1,5.333*age)-young)
  else: child=baby=0; old=max(0,age*2-1); young=1-old
  ```
- Years ↔ value: MIN_AGE = 1, MID_AGE = 25, MAX_AGE = 90. `years = 1 + 48·age` for age < 0.5, and `years = 25 + 130·(age − 0.5)` otherwise. So 0.1875 → **10 y** — [human.py L66-69, L552-572](https://github.com/makehumancommunity/makehuman/blob/master/makehuman/apps/human.py#L552-L572) [fetched]. Modifier description: "Age of the human (range from 1 year to 90 years old, with center position 25 years)" — [modeling_modifiers_desc.json](https://github.com/makehumancommunity/makehuman/blob/master/makehuman/data/modifiers/modeling_modifiers_desc.json) [fetched]. MPFB's "new human" presets use the same values: baby 0.0, child 0.1875, young 0.5, old 1.0 — [createhuman.py L64-75](https://github.com/makehumancommunity/mpfb2/blob/master/src/mpfb/ui/new_human/newhuman/operators/createhuman.py#L64-L75) [fetched].
- Gender in MakeHuman: low = **female**, high = **male** (`maleVal = gender`) — [macro.json](https://github.com/makehumancommunity/mpfb2/blob/master/src/mpfb/data/targets/macrodetails/macro.json), [human.py L517-519](https://github.com/makehumancommunity/makehuman/blob/master/makehuman/apps/human.py#L517-L519) [fetched]. Anny reverses this (male first).
- Height in MakeHuman is **three-state with an implicit average**: minheight weight = max(0, 1 − 2h), maxheight weight = max(0, 2h − 1), and **no height target at h = 0.5** — [human.py L708-714](https://github.com/makehumancommunity/makehuman/blob/master/makehuman/apps/human.py#L708-L714) [fetched]. Proportions works the same way: uncommon below 0.5, ideal above 0.5 — [human.py L782-786](https://github.com/makehumancommunity/makehuman/blob/master/makehuman/apps/human.py#L782-L786) [fetched].
- **Anny differs here.** Anny uses 2-anchor linear weights, so at h = 0.5 it applies 0.5·minheight + 0.5·maxheight. These do not cancel: the young-male maxheight target alone gives 238.8 cm and minheight alone 130.0 cm, from a 166.6 cm template [computed]. Anny's proportions are also reversed (0 = ideal). **MakeHuman (age, height, proportions) values are therefore not interchangeable with Anny values** [inference from fetched code].
- Combination stack (MPFB `calculate_target_stack_from_macro_info_dict`): race-gender-age (weight = race · gender · age); universal-gender-age-muscle-weight; gender-age-muscle-weight-height; gender-age-muscle-weight-cupsize-firmness; gender-age-muscle-weight-proportions, with "There are no baby proportions targets on disk" — [targetservice.py L933-1104](https://github.com/makehumancommunity/mpfb2/blob/master/src/mpfb/services/targetservice.py#L933-L1104) [fetched].
- Target files: `universal-{gender}-{age}-averagemuscle-averageweight` are **empty** (75-byte gz holding the line "0"). So the age shape itself lives in the **race targets**. For example, `caucasian-male-baby` alone turns the 166.6 cm template into 60.2 cm, and `caucasian-male-young` gives 174.8 cm. 99 macro target files exist in `macrodetails/` [computed from fetched data].
- Ethnic modifiers are normalised to sum to 1, default 1/3 each — [humanmodifier.py L615-642](https://github.com/makehumancommunity/makehuman/blob/master/makehuman/apps/humanmodifier.py#L615-L642) [fetched].
- MPFB's randomiser has a discrete "middleage" category, but there is no middle-age target — [randomizeproperties.py L66](https://github.com/makehumancommunity/mpfb2/blob/master/src/mpfb/ui/new_human/randomize/randomizeproperties.py#L66) [fetched].

**Anchor ages compared**

| target | MakeHuman value → years | Anny anchor | Anny calibrated years |
|---|---|---|---|
| newborn | — (none) | −1/3 | < 0 (unused) |
| baby | 0.0 → 1 y | 0 | 0 y |
| child | 0.1875 → 10 y | 1/3 | ≈ 8.1 y |
| young | 0.5 → 25 y | 2/3 | ≈ 15.9 y |
| old | 1.0 → 90 y | 1 | 110 y |

### Inferences
- Anny remapped the same four MakeHuman targets to new positions and added a scaled newborn. The baby target is used as a "0 y" shape, but it was sculpted as a 1-year-old. That explains part of the baby-end mismatch (see 3).

### Gaps
- No MakeHuman documentation was found that states how the baby target was sculpted (references, or the age it represents beyond "1 y").

---

## 3. Reported problems at the infant end, and what the numbers show

### Takeaway
I found no public bug report about age in naver/anny (24 issues listed; none mention age, baby or child). The paper and forum evidence was unreachable. But measuring the real model against Dutch NL4 and Snyder data shows the failure clearly. **From 0 to 2 y the Anny body has a head that is too small (0.4-0.6 extra "heads") and legs that are too long (sitting-height ratio 0.04-0.06 too low). Infants are too thin: at 6 months the default body has BMI 11.9 vs the WHO median of 17.3, and even weight = 1 / muscle = 0 reaches only 14.7, which is the WHO −2 SD line.** The newborn anchor adds no proportion change at all. And the `height` phenotype is strongly entangled with proportions at the baby end. When HeroBody fixes Anny height and scales afterwards, the baby becomes a 5-heads, long-legged toddler.

### Cited Findings
- naver/anny issue list: 24 issues (#3-#28), none about age or infants — [issues](https://github.com/naver/anny/issues?q=is%3Aissue) [fetched]. On pose correctives the maintainer said: "Ideally we would need to come up with a solution that generalizes well across body shapes and ages" — [issue #23](https://github.com/naver/anny/issues/23) [fetched].
- The README claims "a large variety of human body shapes, from infants to elders, using a common topology and parameter space" — [README](https://github.com/naver/anny/blob/v0.6.1/README.md) [fetched].
- Reference values (sibling note, from fetched NL4 LMS tables and Snyder 1977 individual data): SH/H 0.692 (0 y), 0.671 (0.5 y), 0.648 (1 y), 0.604 (2 y), 0.555 (4 y), 0.541 (6 y), 0.531 (8 y), 0.513 (12 y), 0.515 (18 y M). Heads tall: about 4.0-4.2 at birth (textbook), 4.8 (1 y), 5.3 (2 y), 5.7 (4 y), 6.2 (6 y), 6.7 (8 y), 7.0 (10 y), 7.6 (14 y), 8.0 (18 y) — [proportions_by_age.md §1](./proportions_by_age.md) [fetched by sibling researcher; not re-fetched by me].

**Real Anny 0.6.1 measured along its own calibrated path. Height phenotype = calibrated Beta mean, weight = muscle = 0.5, rest pose. Heads = stature / (vertex to menton, vertex 747). SH/H here = 1 − crotch height / H (midline crotch vertex, an approximation of SH) [computed]**

| age y | sex | Anny age | h param | H cm | heads (Anny) | heads (ref) | SH/H (Anny) | SH/H (NL4) | hip-joint/H | shoulder-joint width/H |
|---|---|---|---|---|---|---|---|---|---|---|
| 0 | M | 0.000 | 0.269 | 56.0 | 4.55 | 4.0-4.2 | 0.648 | 0.692 | 0.476 | 0.190 |
| 1 | M | 0.050 | 0.475 | 77.1 | 5.42 | 4.8 | 0.591 | 0.648 | 0.497 | 0.187 |
| 2 | M | 0.105 | 0.440 | 87.9 | 5.73 | 5.3 | 0.578 | 0.604 | 0.502 | 0.193 |
| 4 | M | 0.215 | 0.326 | 104.5 | 6.10 | 5.7 | 0.572 | 0.555 | 0.502 | 0.204 |
| 6 | M | 0.272 | 0.335 | 117.5 | 6.37 | 6.2 | 0.564 | 0.541 | 0.506 | 0.206 |
| 8 | M | 0.329 | 0.347 | 131.0 | 6.63 | 6.7 | 0.556 | 0.531 | 0.510 | 0.207 |
| 11 | M | 0.415 | 0.377 | 145.2 | 6.95 | 7.0-7.4 | 0.543 | 0.52 | 0.518 | 0.210 |
| 13 | M | 0.517 | 0.384 | 159.0 | 7.24 | 7.5 | 0.533 | 0.51 | 0.524 | 0.215 |
| 16 | M | 0.670 | 0.391 | 179.0 | 7.62 | 7.9 | 0.521 | 0.512 | 0.531 | 0.221 |
| 40 | M | 0.799 | 0.391 | 177.8 | 7.59 | 7.8-7.9 | 0.518 | 0.513 | 0.529 | 0.217 |
| 80 | M | 0.889 | 0.391 | 177.0 | 7.56 | 7.5-7.6 | 0.516 | ≈0.512 | 0.528 | 0.214 |
| 0 | F | 0.000 | 0.234 | 54.4 | 4.47 | 4.0-4.2 | 0.656 | 0.693 | 0.474 | 0.192 |
| 1 | F | 0.050 | 0.471 | 75.4 | 5.41 | 4.8 | 0.587 | 0.648 | 0.497 | 0.184 |
| 2 | F | 0.105 | 0.441 | 85.0 | 5.75 | 5.4 | 0.576 | 0.601 | 0.503 | 0.186 |
| 4 | F | 0.215 | 0.348 | 100.1 | 6.21 | 5.7 | 0.566 | 0.553 | 0.506 | 0.191 |
| 16 | F | 0.670 | 0.347 | 160.2 | 7.63 | 7.9 | 0.542 | 0.524 | 0.515 | 0.197 |
| 80 | F | 0.889 | 0.347 | 159.3 | 7.63 | 7.5-7.6 | 0.530 | ≈0.513 | 0.524 | 0.198 |

**Newborn region and height entanglement (male, real Anny) [computed]**

| setting | H cm | heads | SH/H |
|---|---|---|---|
| age −1/3 (newborn), h 0.5 | 49.8 | 5.05 | 0.607 |
| age −1/6, h 0.5 | 58.1 | 5.05 | 0.604 |
| age 0 (baby), h 0.5 | 66.4 | 5.05 | 0.604 |
| age 0, h 0.0 | 43.8 | 3.87 | 0.697 |
| age 0, h 0.25 | 55.1 | 4.50 | 0.652 |
| age 0, h 0.75 | 77.8 | 5.53 | 0.573 |
| age 0, h 1.0 | 89.1 | 5.95 | 0.549 |
| adult (0.79), h 0.0 | 135.3 | 6.80 | 0.555 |
| adult (0.79), h 0.5 | 189.7 | 7.77 | 0.511 |
| adult (0.79), h 1.0 | 244.2 | 8.43 | 0.487 |

**Infant mass/BMI from Anny's `Anthropometry` (density 980 kg/m³), male, calibrated h [computed] vs WHO BMI-for-age (−2 SD / median / +2 SD) [fetched tables]**

| age | Anny BMI m0.5/w0.5 | Anny BMI muscle 0 / weight 1 | Anny BMI m0/w0 | WHO boys BMI −2SD / median / +2SD |
|---|---|---|---|---|
| 0 y (56 cm) | 11.7 (3.68 kg) | 14.2 (4.45 kg) | 8.2 | 11.1 / 13.4 / 16.3 |
| 6 mo (67 cm) | 11.9 (5.35 kg) | 14.7 (6.60 kg) | 8.5 | 14.7 / 17.3 / 20.5 |
| 1 y (77 cm) | 12.3 (7.29 kg) | 15.5 (9.19 kg) | 8.8 | 14.4 / 16.8 / 19.8 |
| 2 y (88 cm) | 13.2 (10.19 kg) | 17.5 (13.56 kg) | 9.6 | 13.6 / 15.7 / 18.5 |
| 4 y (105 cm) | 15.1 (16.61 kg) | 22.1 (24.25 kg) | 11.2 | (CDC 2-20 range) |

- At the baby anchor, **muscle has almost no effect** (waist 0.32-0.36 m, mass ±0.5%), and **proportions has exactly zero effect** because no baby proportions targets exist [computed; fetched code full_model.py L187-208].
- Phrases like "under-aged bodybuilder" and forum reports about MakeHuman baby heads and limbs: **not verified**. I could not search or reach the MakeHuman forum.

### Inferences
- **Why the Phase-1 atlas saw the age knob break at the baby end** (HeroBody fixes Anny height and scales last): with h fixed at 0.5, every age from −1/3 to 0 gives one body shape (5.05 heads, SH/H 0.60), only smaller. A real newborn is about 4.0 heads with SH/H 0.69. Going towards 1 y, heads rise to 5.4 while reality is 4.8. So the baby end shows toddler proportions shrunk to newborn size. Low h partly rescues this (age 0, h 0 gives 3.87 heads and SH/H 0.70), because in MakeHuman's baby targets the minheight target mostly shortens legs and trunk and leaves the head alone.
- The infant body is also too lean: average Anny babies sit at about the WHO −2 SD BMI line, and the maximum reachable is about the median at 0 y and −2 SD at 6 months. Chubby-baby fat pads (cheeks, thighs, wrists, belly) are missing from the targets.
- From 4 to 13 y the trend reverses: legs become slightly **short** (Anny SH/H is 0.02-0.025 too high) and the head is about right. Anny's adult head is about 3% small (7.6 vs 7.8-7.9 heads).
- Old age: Anny gives about 1% stature loss and an unchanged SH/H. Real ageing loses about 3-5 cm of stature, mostly trunk (sibling note). The old target is weak on stature and posture.

### Gaps
- Anny paper and Anny-Fit evaluation numbers on children or infants were not reachable (arXiv, CVF and alphaxiv blocked; search budget exhausted).
- My SH proxy (crotch vertex) is not a seated measurement. The direction and size of the errors are robust (0.04-0.06 at 0-1 y), but the exact values may shift by about 0.01.

---

## 4. Extending a blendshape body model to infants

### Takeaway
Reuse Anny's machinery: hat-weight anchors, product masks and joints regressed from vertices. Then (a) **replace the synthetic newborn with a sculpted newborn delta**, (b) **add an infant-and-toddler corrective family with age-bump weights** over 0-6 y, (c) **add an infant fat target**, and (d) **decouple proportions from stature** by driving Anny's `height` phenotype as a hidden, age-scheduled proportion control while stature comes from uniform scale. This is the minimum. The four HeroBody-planned shapes (head > 2×, chibi short legs, anime waist, round torso) overlap with (a) and (b) but are not substitutes: they are style axes, not age axes.

### Cited Findings
- Anny's age mechanism is extensible by data: `PHENOTYPE_VARIATIONS["age"]` sets the anchor count, the anchors are `linspace(-1/3, 1, n)`, and a blend shape joins any label set through its mask — [model_data.py L40-55](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/model_data.py#L40-L55), [phenotype.py L187-213, L363-373](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/phenotype.py#L187-L373) [fetched]. Adding an anchor changes every anchor position, because they are evenly spaced. Custom anchors need a code change at L189-195 [inference].
- The newborn is a closed-form scale of the baby (see 1). Replacing it only means loading a real newborn delta file instead of `age_to_load = "baby"` in [full_model.py L107-126, L134-152, L162-185](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/full_model.py#L107-L185) [fetched].
- Joints follow blend shapes automatically, because each joint is the mean of fixed vertex groups (see 5). So new targets need **no rig work** [fetched code; inference].
- SMIL / AGORA kid models and AionHMR / SMPL-A: **not fetched in this session**. Background I could not verify: SMIL (Hesse et al., 2018-2019) is an infant SMPL learned from RGB-D sequences of infants. AGORA and SMPL-X "kid" models add an age-like extra shape coefficient that interpolates between the SMPL-X adult template and the SMIL infant template. That gives a linear adult↔infant blend, with known artefacts at intermediate ages [background]. Treat AionHMR/SMPL-A claims as unverified.

### Inferences
**Minimal new blendshape set for ageing (sex-neutral below 2 y; author on the hm08 topology as deltas on top of the Anny baby target):**

| # | name | where it acts | weight function | target numbers (ref) |
|---|---|---|---|---|
| A1 | `age_newborn` | replaces Anny's scaled newborn: head bigger relative to the body, cranium larger than face, short neck, short bowed legs, long trunk, big belly, narrow shoulders | Anny anchor −1/3 → **0 y** (re-anchored) | 4.0-4.2 heads; SH/H 0.69; head circumference ≈ chest + 2 cm; crown-rump ≈ head circumference |
| A2 | `age_infant_corr` | corrective on the baby target: head +10-12% vs Anny, legs −0.05 H | bump: 1 at 0.5-1 y, 0 by 3 y | 1 y: 4.8 heads, SH/H 0.648 |
| A3 | `age_toddler_corr` | head +6%, legs −0.03 H, round belly, lumbar lordosis | bump: peak 2 y, 0 at 0.5 y and 5 y | 2 y: 5.3 heads, SH/H 0.60 |
| A4 | `infant_fat` | cheek, chin, wrist, thigh and ankle rolls, belly volume | profile-driven (BMI z-score), active 0-3 y, fades by 5 y | lets 6 mo reach BMI 17.3 (median) to 20.5 (+2 SD) |
| A5 | `child_legs_corr` | small leg lengthening, 4-13 y | bump peak 8-10 y, amplitude 0.02-0.025 H | SH/H 0.531 at 8 y, 0.513 at 12 y |
| A6 | `old_stoop` | thoracic kyphosis, forward head, disc-height loss in the trunk, slight knee flexion in the rest pose | ramp 60→90 y | −3 to −5 cm stature, mostly trunk; SH/H ≈ 0.512 at 80 y |

- Age-conditioned correctives: `w_k(y) = bump((y − c_k) / s_k)` with a smooth bump, for example a quartic `(1 − t²)²` for |t| < 1. They add to Anny's blend. Fit amplitudes by least squares so that heads, SH/H, leg/H, shoulder/H and head circumference/H hit the reference table at the anchor years [inference].
- The infant fat target must be a weight-like axis, not a muscle axis. Infant "muscle" should be pinned near 0, which matches Anny's own 1-y prior (muscle mean 0.03, weight mean 0.97) [computed + inference].
- Why the anime shapes do not cover infants: the "head > 2×" blendshape scales the head and keeps adult cranio-facial ratios. An infant head has a large cranium, a small low face, eyes at mid-head height or lower, and almost no neck. The face part belongs to the separate anime face model. The body still needs A1-A3 for the neck, shoulders, trunk and legs [inference].
- Don't add extra anchors to `linspace`. Use a HeroBody-side **year → weights table** (section 6). That keeps Anny's code untouched and gives smooth (C1) curves [inference].

### Gaps
- No fetched infant 3D shape data (SMIL is non-commercial and was not reachable). A1-A4 must be sculpted from proportion tables and artist reference, then checked numerically.
- No fetched data on measured infant head height (vertex to menton) below 2 y. The 0-1 y "heads" values are textbook or head-circumference based (sibling note).

---

## 5. Anny topology and rig facts relevant to ageing

### Takeaway
The **skeleton rescales with age automatically**. Every bone head and tail is the mean of a fixed vertex set ("CUBE" joint-helper groups, single "VERTEX" points or "MEAN" vertex lists), applied to the same blend shapes. So joints are linear in the blend coefficients. The **53-bone `game_engine` rig (Unreal names) is built in** and survives every age from newborn to 110 y: all 53 joint heads stay inside the mesh and no bone collapses. Skinning weights are **fixed** (adult-derived) for all ages. Rest shapes for any age are available directly as `rest_vertices`, `rest_bone_heads` and `rest_bone_tails`.

### Cited Findings
- Joint regression: `template_bone_heads = mean(template_vertices[idx])`, `heads_blend_shapes = mean(blendshapes[:, idx, :])`, and the same for tails — [full_model.py L522-578](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/full_model.py#L522-L578) [fetched]. Strategies VERTEX/CUBE/MEAN — [full_model.py L243-256](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/full_model.py#L243-L256) [fetched]. Rest model: `rest_bone_heads = T_heads + B_heads · coeffs` — [rigged_model.py L279-318](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/rigged_model.py#L279-L318) [fetched].
- Rig presets: `anny` (rig.default.json, 163 bones in the file, pruned to **104** in the default model), `makehuman`, `cmu_mb`, `game_engine`, `mixamo` — [model_data.py L222-230](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/model_data.py#L222-L230) [fetched]. `rig.game_engine.json` has **53 bones** (Root, pelvis, spine_01-03, neck_01, head, clavicle/upperarm/lowerarm/hand, 15 finger bones per hand, thigh/calf/foot/ball). 52 joints use CUBE and 1 uses MEAN [computed].
- v0.6 "anny" rig: "Rest bone orientations are anchored to the mesh: each bone follows the vertices it skins … recovered at runtime by a single batched Procrustes alignment" — [CHANGELOG v0.6](https://github.com/naver/anny/blob/v0.6.1/CHANGELOG.md) [fetched].
- Skinning weights: one weight file per rig (for example `weights.game_engine.json`), loaded once and normalised, with no age dependence — [full_model.py L541-558](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/full_model.py#L541-L558) [fetched].
- Test with real Anny, `rig="game_engine"`, male [computed]:

  | body | H m | bone heads outside skin (trimesh `contains`) | shortest parent-child distance / H |
  |---|---|---|---|
  | newborn (−1/3, h 0.5) | 0.498 | none | 0.0104 (pinky_03_l) |
  | 0 y | 0.560 | none | 0.0103 |
  | 1 y | 0.771 | none | 0.0097 |
  | 4 y | 1.048 | none | 0.0094 |
  | adult | 1.777 | none | 0.0095 |
  | 110 y | 1.759 | none | 0.0087 |

- No pose correctives in Anny; none planned. The maintainer cites generalisation "across body shapes and ages" — [issue #23](https://github.com/naver/anny/issues/23) [fetched].

### Inferences
- The game rig keeps its topology at baby proportions. Its **range of motion** does not: infant joints (short neck, fat folds, big head on a short neck) will collide sooner. HeroBody's planned 90° pose correctives should take age as an input (for example corrective = f(pose) · g(age)). They also need infant-specific fold shapes in the elbow, knee, groin and neck.
- Adult skinning weights on a newborn: the neck region is short, so head and neck weights will bleed into the shoulders. Recompute or retarget weights on the A1 newborn shape once, then blend the weight sets by age (two weight sets, linear blend), or accept it for the game rig [inference].
- Because joints are means of vertex groups, any new target (A1-A6) must also move the joint-helper cubes (`joint-*` groups in base.obj) sensibly. Sculpt with helpers visible, or regenerate helper cube offsets from nearby body vertices [inference].

### Gaps
- `trimesh.contains` on a mesh with open eye sockets is approximate. A per-bone check against the HeroBody anatomy registration (BodyParts3D bones) was not done.

---

## 6. Plan: making age work in HeroBody from baby to very old

### Takeaway
Age is a **profile fact in years**, like sex. The solver never fits it. HeroBody owns a year → (Anny age, hidden proportion parameter, corrective weights, weight/muscle priors) table. Stature is applied last as a uniform scale from WHO/CDC (or the blueprint). Anny's `height` phenotype is repurposed as an internal, age-scheduled **proportion** control, `h_prop(y)`, that the solver may nudge within bounds. New targets A1-A6 fix the 0-6 y and old-age gaps.

### Cited Findings (code anchors this plan reuses)
- Phenotype blend: [phenotype.py L296-373](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/phenotype.py#L296-L373). Anchors: [L187-213](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/phenotype.py#L187-L213). Target loading: [full_model.py L86-240](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/full_model.py#L86-L240). Joint regression: [full_model.py L560-578](https://github.com/naver/anny/blob/v0.6.1/src/anny/models/full_model.py#L560-L578). Calibration tables: `src/anny/data/shape_calibration/{boys,girls}.pth`. Anthropometry (height, waist, volume, mass at 980 kg/m³, BMI): [anthropometry.py](https://github.com/naver/anny/blob/v0.6.1/src/anny/anthropometry.py) [all fetched].

### Inferences — concrete design
1. **Year → Anny age.** Start from Anny's table, then change it:
   - Re-anchor the infant end. Put 0 y at the new sculpted newborn (A1, Anny age −1/3) and 1 y at the baby target (Anny age 0), which matches MakeHuman's nominal "baby = 1 y".
   - Keep 4 y → 0.215, 11 y → 0.415 and 16 y → 0.67.
   - Widen the adult plateau: 18 y → 0.667 ("young" weight 1.0), 40 y → 0.70, 64 y → 0.80, 90 y → 0.95, 110 y → 1.0, so a 40-year-old is about 10% "old" target, not 40%.
   - Use monotone cubic (PCHIP) interpolation in years for C1 slider motion.
   - These numbers are a starting proposal. Refit the adult part against adult ageing references (sibling notes) [inference].
2. **Hidden proportion schedule `h_prop(y)`.** At each anchor year, solve for the Anny `height` value that hits the reference SH/H and heads. Example from the table in 3: at the baby anchor, h 0.0 gives SH/H 0.697 / 3.87 heads, and h 0.25 gives 0.652 / 4.50. So a 1 y target of SH/H 0.648 → h ≈ 0.26 before A2 corrections. Store it as a table per sex. The knob solver may move h within ±0.1 of `h_prop(y)` to fit a blueprint's leg/H. Never use h for stature.
3. **Stature last.** Scale s = H_target / H_mesh, with H_target = WHO (0-5 y) or CDC (2-20 y) LMS median × the profile z-score, or the blueprint height. Adults use profile height. Old age subtracts the age loss (sibling note `growth_stature_trajectories.md`). Apply s to vertices, joints and sockets; it is uniform, so ratios survive. This matches the existing HeroBody rule "height applied LAST as a pure uniform scale".
4. **Weight/muscle priors in years.** Use Anny's Beta tables, but index them in **years** (fix the unit bug). For 0-3 y pin muscle ≤ 0.1 and drive A4 `infant_fat` from the profile BMI z-score against WHO BMI-for-age (median 13.4 at 0 mo, 17.3 at 6 mo, 16.8 at 12 mo, 15.7 at 24 mo, boys).
5. **Stages and sliders.** Profile stages baby / child / teen / adult / old map to year spans, for example [0, 3), [3, 12), [12, 18), [18, 60), [60, 100]. The in-stage slider t ∈ [0, 1] gives y = y_start + t·(y_next − y_start). Everything downstream runs on years.
6. **Identity across ages.** The per-character style offset (anime knobs) is stored as **ratios relative to the age-appropriate realistic body** (for example leg/H_char − leg/H_real(y)), not as absolute Anny values. The same character then ages with the population curve and keeps its exaggeration. Manga overrides at a given age replace the realistic target ratios at that age before solving.
7. **Rig.** Use Anny `rig="game_engine"` (53 bones, Unreal names). Joints come free from the blend. Add age as an input to the pose correctives. Re-weight once on the A1 newborn and blend weight sets for ages under 3 y (optional for the game, needed for film).
8. **Validation (zero manual cleanup).** For each anchor year and sex, assert:
   - heads within ±0.15 of the reference;
   - SH/H within ±0.01 (NL4);
   - shoulder/H, head circumference/H and BMI within reference ±1 SD;
   - all 53 joints inside the skin;
   - anatomy clamp penetration below 1% of anatomy (the existing check).

   Run it CPU-only: one Anny forward pass on CPU takes milliseconds per body, and the sampling script for 6,000 bodies ran in well under a minute on this CPU [computed].

### Gaps
- Exact numeric `h_prop(y)` and A2/A3 amplitudes need a fitting run on the final targets.
- Old-age shape references (kyphosis angle by age) are not in this note; see `skeleton_posture_motion_by_age.md`.

---

## How this plugs into HeroBody

- **Knob solver (open build item "the knob solver, height last"):** add `age_years` as a fixed input, like sex. New module `hb/age_schedule.py` with pure functions:
  - `anny_age(y)`: PCHIP over the table in 6.1;
  - `h_prop(y, sex)`: table from 6.2;
  - `corrective_weights(y)`: bumps for A2, A3, A5 and the A6 ramp;
  - `priors(y, sex)`: Beta tables indexed in years;
  - `stature(y, sex, z)`: WHO/CDC LMS: H = M·(1 + L·S·z)^(1/L).

  The solver optimises the 44 Anny parameters minus {age, height}, plus `h_prop` within ±0.1, plus A4 `infant_fat`, then applies uniform scale.
- **New blendshapes (open build item "the four new blendshapes"):** extend the list to 4 style shapes + 6 age shapes (A1 `age_newborn`, A2 `age_infant_corr`, A3 `age_toddler_corr`, A4 `infant_fat`, A5 `child_legs_corr`, A6 `old_stoop`). Store them as `.target.gz` in the MakeHuman format (`index dx dy dz`, decimetres, Y up) so Anny's `load_blend_shape` reads them unchanged. A1 loads in place of the scaled baby at the newborn anchor (patch `full_model.py` L107-126 or post-process `model.blendshapes` rows labelled `*newborn*`).
- **Fat layer (open build item):** A4 is the infant part of the fat layer. Keep it separate from Anny `weight` so muscle and weight stay adult-meaningful.
- **Pose correctives (open build item):** corrective weight = f(pose) · g(age). Author infant fold shapes for the neck, elbow, knee and groin.
- **Rig binding:** use `Anny(rig="game_engine")`. Joints need no age work. Optionally blend two skin-weight sets for ages under 3 y.
- **Phase-1 knob atlas:** re-run the age row with the new schedule. Acceptance: the table in 3 matches the references within the tolerances in 6.8.
- **Character profile:** fields `age_years` (or stage + slider), `height_z`, `bmi_z`, plus manga overrides per stage. Stage boundaries live in the profile, not in the mesh code.

## Sources

- naver/anny v0.6.1 source: https://github.com/naver/anny (files: src/anny/models/phenotype.py, src/anny/models/model_data.py, src/anny/models/full_model.py, src/anny/models/rigged_model.py, src/anny/utils/interpolation.py, src/anny/shape_distribution.py, src/anny/anny_inverter.py, src/anny/anthropometry.py, src/anny/data/shape_calibration/boys.pth and girls.pth, src/anny/data/mpfb2/rigs/standard/rig.game_engine.json, CHANGELOG.md, README.md)
- naver/anny issues: https://github.com/naver/anny/issues?q=is%3Aissue , https://github.com/naver/anny/issues/15 , https://github.com/naver/anny/issues/23
- MPFB2: https://github.com/makehumancommunity/mpfb2 (src/mpfb/data/targets/macrodetails/macro.json, src/mpfb/services/targetservice.py, src/mpfb/ui/new_human/newhuman/operators/createhuman.py, src/mpfb/ui/new_human/randomize/randomizeproperties.py)
- MakeHuman 1.x: https://github.com/makehumancommunity/makehuman (makehuman/apps/human.py, makehuman/apps/humanmodifier.py, makehuman/data/modifiers/modeling_modifiers.json, modeling_modifiers_desc.json)
- WHO 0-5 y and CDC 2-20 y LMS tables (JSON copies): https://github.com/ewheeler/pygrowup/tree/master/pygrowup/tables
- Sibling note (NL4 and Snyder reference proportions): ./proportions_by_age.md
- Not reachable (cited for completeness): Anny paper arXiv 2511.03589 https://arxiv.org/abs/2511.03589 ; Anny blog https://europe.naverlabs.com/blog/anny-a-free-to-use-3d-human-parametric-model-for-all-ages/
