# Age progression systems in games and character creators (CK3, Sims 4, Bannerlord, MakeHuman/Anny, MetaHuman, others) and a data model for HeroBody ageing

Research date: 2026-10-09. Researcher notes for the report writer.

**Access notes.** This session had very limited access. The shared web-search budget for the turn was already used up when this track started (WebSearch returned "budget is used up" on the first call), so there are **no search snippets** in this file. WebFetch failed with DNS errors (`getaddrinfo ENOTFOUND`) for ck3.paradoxwikis.com, apidoc.bannerlord.com, simswiki.info, en.wikipedia.org and manual.reallusion.com. curl through the proxy got a 403 on CONNECT for every non-GitHub host I tested: ck3.paradoxwikis.com, forum.paradoxplaza.com, Wikipedia, sims/spore/fable/bannerlord/blackdesert/onepiece fandom wikis, reallusion.com, daz3d.com, docs.daz3d.com, dev.epicgames.com, metahuman.com, modthesims.info, reddit, gamedeveloper.com, gdcvault.com, ea.com, sakugabooru, animenewsnetwork, cartoonbrew, semanticscholar, dl.acm.org and pmc.ncbi.nlm.nih.gov. **Only github.com and raw.githubusercontent.com could be reached.** The GitHub code-search MCP tool worked across all of GitHub. The GitHub REST API was limited to the session's own repo. So every primary fact below comes from **game data files and decompiled or tool source code that people have published on GitHub**:
- CK3/Jomini gene files: copies of vanilla files in mods (A Game of Thrones mod `Sohex/temp-agot`, `GalacticLiaison/Elf-Destiny`, `csirke128/CBO-Base`, Warcraft: Guardians of Azeroth), plus the **Victoria 3 vanilla** `01_genes_morph.txt` (`tjysdsg/Victoria-3-files`). Vic3 runs on the same Jomini portrait engine. I also used CK3 `00_defines.txt` copies and the `amtep/tiger` validator source, which encodes the file grammar.
- Bannerlord: decompiled `TaleWorlds.Core` and `CampaignSystem` source (`BannerlordCode/bannerlord-1.3.15`) and a mod's `skins.xml`.
- The Sims 4: decompiled Python (`jafffy/sims4-modding-framework`), protobuf stubs (`TURBODRIVER/TS4ProtobufStubs`), the TS4 SimRipper source (`CmarNYC-Tools/TS4SimRipper`), and a tuning override from a mod.
- MakeHuman `human.py`, Anny `phenotype.py`, and the MetaHuman DNA Calibration docs (`EpicGames/MetaHuman-DNA-Calibration`).

Nothing primary could be fetched for **Character Creator 4, Daz Genesis 8/9, Black Desert, Spore, Fable, or anime production practice**. Those parts are background knowledge and are tagged **[inference]**, flagged "unverified". They need a follow-up session with web access.

Tags: **[fetched]** = I read the primary file or code myself, including a GitHub copy of a game file. **[computed]** = I computed the number from fetched data; the script is described. **[inference]** = my reasoning, or background knowledge I could not verify this session. No [snippet] tags appear, because no search ran.

---

## 1. Crusader Kings III DNA/genes system: genes, morph targets and age curves

### Takeaway
CK3 keeps a character's **identity** and their **age** in separate places:
- **Identity is the DNA**: one byte (0–255) per gene, plus a template name for each morph gene. It is stored once for the character's whole life.
- **Age is not stored in the DNA.** At render time, every gene *setting* (one blendshape or bone attribute) can carry an `age = <preset>` curve. The curve's x axis is **age / 100 years** (`NPortrait.MAX_AGE = 100.0`). In `mode = multiply` the curve scales the gene's effect. In `mode = add` it adds an offset.
- **Children** are handled by a separate pair of body types, `boy`/`girl`. They switch to `male`/`female` at **18 years** (`PORTRAIT_*_ADULT_AGE = 18`), even though gameplay adulthood starts at 16.
- **Infant proportions** come from a dedicated `gene_age` gene. It drives infant-head, infant-body and old-age blendshapes and joint attributes, each with its own age curve. Different `gene_age` templates (`old_1`…`old_4`, `old_beauty_1`, `no_aging`) give different ageing *styles*. A trait can force a template, which is how designers override ageing for specific characters.
- **Apparent age can differ from real age.** "Graceful aging" slows visual ageing between 25 and 70 years for characters with higher life expectancy.

### Cited Findings
- **File grammar.** The gene database has these top-level blocks: `age_presets`, `color_genes`, `morph_genes`, `accessory_genes`, `special_genes`, `decal_atlases`. Body types are `male`, `female`, `boy`, `girl`. EU5 adds `adolescent_boy`/`adolescent_girl`, and Imperator/EU5 add `infant`. An age preset must have `mode` and `curve`. Each curve point is `{ x y }` with **x in 0.0–1.0 and y in −1.0–1.0**. A gene `setting` must have `attribute` and either `value = { min max }` or `curve`, and may have `age` (a preset name or an inline `{ mode curve }` block) and `required_tags` — [amtep/tiger src/data/genes.rs](https://github.com/amtep/tiger/blob/HEAD/src/data/genes.rs) [fetched].
- **Age normalisation.** `NPortrait = { MAX_AGE = 100.0 # At this age portraits will use the special age gene at full strength; PORTRAIT_MALE_ADULT_AGE = 18 # The boy -> male portrait change happens at this age; PORTRAIT_FEMALE_ADULT_AGE = 18 }` — [CK3 00_defines.txt copy, AVE_MARIA_CK3](https://github.com/alltheatreides/AVE_MARIA_CK3/blob/HEAD/common/defines/00_defines.txt) [fetched].
- **Graceful ageing.** `GRACEFUL_AGING_START = 25 # After this age, added life expectancy will make a character look younger than they are`; `GRACEFUL_AGING_END = 70 # apparent age at which life expectancy stops slowing down visual aging (each year onwards ages you visually 1 year)` — [same file](https://github.com/alltheatreides/AVE_MARIA_CK3/blob/HEAD/common/defines/00_defines.txt) [fetched].
- **Weight drifts slowly toward a target.** `WEIGHT_UPDATE_YEAR_INTERVAL = 3` years between portrait weight updates (1 for players), `WEIGHT_UPDATE_LERP_SCALAR = 0.40` (0.30 for players), and starting base weight random in `[-35, 35]` — [same file](https://github.com/alltheatreides/AVE_MARIA_CK3/blob/HEAD/common/defines/00_defines.txt) [fetched].
- **Gameplay life stages** (not portrait stages): `TODDLER_AGE = 3`, `CHILDHOOD_AGE = 6`, `ADOLESCENCE_AGE = 12`, `MALE/FEMALE_ADULT_AGE = 16`. Mod copies show `MALE/FEMALE_ELDERLY_AGE = 50`; Warcraft GoA "bumped this up a decade" to 60 — [CK3 defines copies via GitHub code search: AVE_MARIA_CK3, Calradian_Kings, LuxRenataCK3, Warcraft GoA](https://github.com/alltheatreides/AVE_MARIA_CK3/blob/HEAD/common/defines/00_defines.txt) [fetched].
- **The DNA record.** A DNA block (`common/dna_data`) holds `portrait_info = { type=male id=… age=0.480000 genes={ … } }`.
  - Colour genes hold 4 bytes, e.g. `hair_color={ 255 168 236 155 }`.
  - Morph genes hold *two* (template, byte) pairs, e.g. `gene_chin_forward={ "chin_forward" 138 "chin_forward" 138 }`.
  - Accessory genes use the same pattern, e.g. `hairstyles={ "european_hairstyles" 255 "european_hairstyles" 119 }`.
  - The `age=0.48` field uses the same /100 normalisation (48 years).
  - Source: [Victorian-Flavor-Mod dna_data/carlos_v.txt (Vic3, same Jomini format)](https://github.com/Radsterman/Victorian-Flavor-Mod/blob/HEAD/common/dna_data/carlos_v.txt) [fetched].
- **How one gene template maps to blendshapes.** In Vic3 vanilla, `gene_cheek_fat` → template `cheek_fat`. For `male` it sets `head_bs_cheek_fat_max` with `curve = { {0.0 0.0} {0.5 0.0} {1.0 @maleBsMax} }` and `head_bs_cheek_fat_min` with `curve = { {0.0 @maleBsMax} {0.5 0.0} {1.0 0.0} }`, both with `age = age_preset_child_features`. Then `boy = male` and `girl = female`. So **one gene byte drives a pair of opposing blendshapes** through a piecewise-linear curve, and **children reuse the adult mapping** but with the age multiplier applied — [Vic3 01_genes_morph.txt](https://github.com/tjysdsg/Victoria-3-files/blob/HEAD/game/common/genes/01_genes_morph.txt) [fetched].
- **Range constants.** Vic3: `@maleBsMin = -1.0, @maleBsMax = 1.0, @femaleBsMin = -0.8, @femaleBsMax = 0.8, @boyMin = 0.0, @boyMax = 1.0, @girlMin = 0.2, @girlMax = 0.8`. CK3 (per the Elf Destiny override comments, which copy vanilla): `@maleMin = -0.5, @maleMax = 0.499, @femaleMin = -0.4, @femaleMax = 0.4, @maleBsMax = 1.0, @femaleBsMax = 0.8`. **Female ranges are narrower (0.8× the male range)**, a built-in dimorphism — [Vic3 file](https://github.com/tjysdsg/Victoria-3-files/blob/HEAD/game/common/genes/01_genes_morph.txt), [Elf-Destiny overrides](https://github.com/GalacticLiaison/Elf-Destiny/blob/HEAD/common/genes/zzz_elf_destiny_gene_overrides.txt) [fetched].
- **`gene_age` templates.** The gene has templates `old_1` (index 0), `old_2`, `old_3`, `old_4`, `old_beauty_1` (index 4, `generic = no # Can't be selected unless the character has a trait with forced_portrait_age_index matching this`) and `no_aging` (index 5). Inside each template:
  - `head_infant_proportions` and `body_infant_proportions` (joint attributes, value @maleMax) with `age_preset_infant_joints(_body)`.
  - `bs_infant_1`, `bs_infant_1_body`, `bs_body_infant_1` (blendshapes, value @maleBsMax) with `age_preset_child_bs_head/body`.
  - `bs_old_1`, `bs_old` (beards), `bs_body_old_1` with `age_preset_aging_primary`.
  - `jaw_forward`, `bs_nose_size_max`, `bs_ear_size_max` with `age_preset_aging_secondary`.
  - `jaw_angle`, `mouth_*`, lip sizes with `age_preset_aging_tertiary(_reversed)`.
  - Infant skin decals (head normal map fading out by x = 0.25) and old-age head/torso decals (fade in from x = 0.25 → 0.7 and 0.32 → 0.7).
  - `hair_hsv_shift_curve { 0.0 { 0.0 -1.0 0.3 } }` (greying: saturation −1, value +0.3), eye and skin HSV shifts, all on `age_preset_aging_hsv_curve`.
  - Source: [Elf-Destiny gene_age override (copy of vanilla + edits)](https://github.com/GalacticLiaison/Elf-Destiny/blob/HEAD/common/genes/zzz_elf_destiny_gene_overrides.txt) [fetched].
- **`no_aging` is empty in vanilla.** The Elf Destiny mod fills it with infant proportions "for characters whose age is frozen; 'no more adult babies'". Its header says that without the `@` constants "every age setting fails to parse … and children get adult proportions" — [Elf-Destiny overrides header](https://github.com/GalacticLiaison/Elf-Destiny/blob/HEAD/common/genes/zzz_elf_destiny_gene_overrides.txt) [fetched].
- **Body shape and musculature.** `gene_bs_body_shape` "is used for two things: Basic body shape and gradual musculature (the latter tied to gene strength and controlled by modifiers)". Templates: `body_shape_average(_clothed)`, `apple/hourglass/pear/rectangle/triangle` × `half/full`. Decal `alpha_curve` comments read "#character age%, decal alpha" — [same file](https://github.com/GalacticLiaison/Elf-Destiny/blob/HEAD/common/genes/zzz_elf_destiny_gene_overrides.txt) [fetched].
- **Height ages in too.** Vic3 `gene_height` → `normal_height`: `body_height` and `head_height` attributes `value = { min = @maleAnimMin(-0.5) max = @maleAnimMax(0.5) }` with `age = age_preset_height`. So the *individual's* height deviation grows in from 0 at birth to full at 18 years and shrinks to 0.6 in old age — [Vic3 01_genes_morph.txt](https://github.com/tjysdsg/Victoria-3-files/blob/HEAD/game/common/genes/01_genes_morph.txt) [fetched].
- **Ethnicities** are weighted ranges per gene, e.g. `gene_height = { 10 = { name = normal_height range = { 0.60 0.70 } } }`, `hair_color = { 30 = { 0.1 0.6 0.6 0.8 } }` (weight = { x0 y0 x1 y1 } in a colour-ramp texture) — [CK3-Human-Phenotype-Project 01_brunn.txt](https://github.com/Metalhead33/CK3-Human-Phenotype-Project/blob/HEAD/common/ethnicities/01_brunn.txt) [fetched].
- **Mods add their own age presets.** Warcraft GoA's `gene_sexual_dimorphism` uses presets `age_preset_puberty`, `age_preset_child_fat`, `age_preset_child_features(_wide_range)` on attributes such as `jaw_width`, `head_width`, `bs_nose_*`, `neck_length`, `eye_distance` — [wc_genes_sexual_dimorphism.txt](https://github.com/Warcraft-GoA-Development-Team/Warcraft-Guardians-of-Azeroth-2/blob/HEAD/common/genes/wc_genes_sexual_dimorphism.txt) [fetched].

**Age preset curves.** Piecewise-linear, x = age/100, y clamped to the end points. Sampled at whole years [computed: parsed the `age_presets` blocks and interpolated linearly; my parser script is `scratchpad/scripts/ck3tab.py`]. The CK3 rows come from the AGOT mod's override file, which redefines the vanilla preset names ([Sohex/temp-agot gene_age_override.txt](https://github.com/Sohex/temp-agot/blob/HEAD/2962333032/common/genes/gene_age_override.txt) [fetched]). Its `aging_primary` and `infant_joints` points are identical to Vic3 vanilla, so they are very likely the vanilla CK3 values. I could not confirm that against an untouched CK3 install [inference]. The Vic3 rows are vanilla ([Vic3 01_genes_morph.txt](https://github.com/tjysdsg/Victoria-3-files/blob/HEAD/game/common/genes/01_genes_morph.txt) [fetched]).

Raw curve points (x = age/100):

| preset (file) | mode | points `{x y}` |
|---|---|---|
| aging_primary (CK3/AGOT = Vic3) | multiply | {0 0} {0.25 0} {0.35 0.2} {0.75 1.0} |
| aging_secondary (CK3/AGOT) | multiply | {0 0} {0.5 0} {0.85 0.5} |
| aging_secondary (Vic3) | multiply | {0 0} {0.55 0} {0.90 0.5} |
| aging_tertiary (CK3/AGOT) | multiply | {0 0} {0.7 0} {0.95 0.5} |
| aging_tertiary_reversed (CK3/AGOT) | multiply | {0 0} {0.7 0} {0.95 −0.5} |
| aging_hunchback (CK3/AGOT; commented out in gene_age) | multiply | {0 0} {0.7 0} {0.95 1.0} |
| aging_hsv_curve (CK3/AGOT) | multiply | {0 0} {0.35 0} {0.7 1.0} |
| aging_hsv_curve (Vic3) | multiply | {0 0} {0.35 0} {0.7 0} {0.8 1.0} |
| infant_joints | multiply | {0 −1} {0.03 −0.6} {0.07 −0.43} {0.10 −0.28} {0.15 −0.05} {0.18 0} {1.0 1.0} |
| infant_joints_body | multiply | {0 −1} {0.03 −0.7} {0.07 −0.45} {0.15 −0.05} {0.18 0} {1.0 1.0} |
| child_bs_head (CK3/AGOT) | multiply | {0.03 1} {0.05 0.75} {0.07 0.55} {0.10 0.35} {0.12 0.25} {0.14 0.15} {0.16 0.05} {0.18 0} |
| child_bs_body (CK3/AGOT) | multiply | {0 1} {0.05 0.4} {0.10 0.1} {0.15 0} |
| child_skin (CK3/AGOT) | multiply | {0 0.5} {0.15 0} |
| pre_puberty (CK3/AGOT) | multiply | {0 1} {0.10 1} {0.15 0} |
| child_features (Vic3) | multiply | {0 1} {0.05 0.5} {0.10 0.65} {0.22 1.0} |
| child_fat (Vic3) | multiply | {0 0.2} {0.10 0.5} {0.18 1.0} {0.7 1.0} {0.95 0.2} |
| height (Vic3) | multiply | {0 0} {0.18 1.0} {0.7 1.0} {0.9 0.6} |
| beard_growth (Vic3) | multiply | {0 0} {0.15 0} {0.22 1.0} |
| aging_gauntness (Vic3) | **add** | {0 0} {0.7 0} {0.95 0.4} |
| eyebrows_fullness (Vic3) | multiply | {0 0.5} {0.15 0.75} {0.2 1.0} {0.5 1.0} {0.9 0.2} |

Sampled values (age in years):

| preset | 0 | 1 | 3 | 6 | 10 | 12 | 14 | 16 | 18 | 25 | 35 | 50 | 60 | 70 | 80 | 90 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| aging_primary | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0.20 | 0.50 | 0.70 | 0.90 | 1.00 | 1.00 |
| aging_secondary (CK3) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0.14 | 0.29 | 0.43 | 0.50 |
| aging_tertiary (CK3) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0.20 | 0.40 |
| aging_hsv (CK3) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0.43 | 0.71 | 1.00 | 1.00 | 1.00 |
| infant_joints | −1.00 | −0.87 | −0.60 | −0.47 | −0.28 | −0.19 | −0.10 | −0.03 | 0.00 | 0.09 | 0.21 | 0.39 | 0.51 | 0.63 | 0.76 | 0.88 |
| child_bs_head (CK3) | 1.00 | 1.00 | 1.00 | 0.65 | 0.35 | 0.25 | 0.15 | 0.05 | 0.00 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| child_bs_body (CK3) | 1.00 | 0.88 | 0.64 | 0.34 | 0.10 | 0.06 | 0.02 | 0.00 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| child_features (Vic3) | 1.00 | 0.90 | 0.70 | 0.53 | 0.65 | 0.71 | 0.77 | 0.82 | 0.88 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| child_fat (Vic3) | 0.20 | 0.23 | 0.29 | 0.38 | 0.50 | 0.62 | 0.75 | 0.88 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 0.68 | 0.36 |
| height (Vic3) | 0.00 | 0.06 | 0.17 | 0.33 | 0.56 | 0.67 | 0.78 | 0.89 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | 0.80 | 0.60 |
| aging_gauntness (Vic3, add) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0.16 | 0.32 |

### Inferences
- **Evaluation equation** (implied by the grammar and the comments, not documented in what I could read) [inference]. For one setting with DNA byte b on template T at age A in years:
  - `g = b / 255` (0–1)
  - `base = lerp(min, max, g)` for `value = {min max}`, or `base = piecewise_linear(curve, g)` for `curve = {…}`
  - `x = clamp(A / MAX_AGE, 0, 1)`, with MAX_AGE = 100
  - `mode = multiply`: `w = base × agecurve(x)`; `mode = add`: `w = base + agecurve(x)`
  - Final attribute value = Σ over all genes touching that attribute (blendshape weight or joint/bone attribute).
  - `w` uses the second body-type block (`boy`/`girl`) when A < 18.
- **Joint attributes form one axis from infant to old.** `infant_joints` is a joint attribute curve that runs from −1 at birth through 0 at 18 years to +1 at 100 years. So `head_infant_proportions` (value 0.499) goes −0.5 → 0 → +0.5. A single bone-scale axis covers infant proportions and some old-age change (my reading: the +0.5 end likely shrinks or rounds the head and body slightly in old age; unverified) [inference].
- **Child faces are partly muted identity.** Settings tagged `age_preset_child_features` show the identity shape at full strength at birth, dip to about 0.5 around 5 years, then return to 1.0 by 22 years. So a child's face shows a *weaker* version of the adult identity, plus the infant blendshapes. That is the exact pattern HeroBody needs: **identity is constant and its expression is a function of age** [inference].
- **Stored DNA versus visible DNA.** The two (template, byte) pairs per morph gene look like a "visible" pair and a "recessive/inheritable" pair. In the Carlos V example they are identical for faces and differ for accessories (beards 179 vs 32). I could not verify the exact inheritance rule [inference].
- **Graceful ageing as a formula** (exact CK3 formula not seen) [inference]. A usable stand-in:
  - `A_app = A` for A ≤ 25.
  - For A > 25: `A_app = 25 + (A − 25) × r`, with r < 1 derived from life expectancy, until `A_app` reaches 70.
  - After that, A_app advances 1 year per year.

### Gaps
- I could not read the CK3 wiki pages ("Genes", "Portrait modding", "Character modding") or an untouched CK3 install file. The exact vanilla CK3 values of `age_preset_aging_secondary`/`tertiary` may differ slightly from the AGOT copy.
- The exact meaning of `multiply` when the curve value is negative (infant joints) is unknown, and so is the internal graceful-ageing formula.
- I did not confirm whether the "special age gene at full strength" at MAX_AGE refers only to `gene_age` or to all age presets.

---

## 2. Other systems: The Sims 4, Bannerlord, MakeHuman/Anny, MetaHuman, CC4, Daz, Black Desert, Spore, Fable

### Takeaway
Two patterns cover almost every system:
- **(a) Discrete maturity classes with separate meshes or rigs, plus continuous sliders inside the adult class.** The Sims 4 uses Baby/Infant/Toddler/Child/Teen–Elder. Bannerlord uses Toddler/Child/Tween/Teenager/Adult meshes plus a continuous 3–128-year age key.
- **(b) One topology with a continuous age parameter that blends a few age anchor targets.** MakeHuman/Anny blend baby/child/young/old targets. CK3 multiplies age curves over identity genes.

Every system keeps identity as a separate block that does not change with age:
- CK3: DNA.
- Sims 4: `facial_attributes` (sculpts + modifiers) + `physique`.
- Bannerlord: `StaticBodyProperties` (8 × 64-bit face keys) versus `DynamicBodyProperties` (age, weight, build).
- MetaHuman: DNA (geometry/behaviour), where age is only a descriptor field.

### Cited Findings
**The Sims 4**
- **Age enum:** `BABY=1, TODDLER=2, CHILD=4, TEEN=8, YOUNGADULT=16, ADULT=32, ELDER=64`. Animation cache ages are only `ADULT, CHILD, TODDLER`; anything ≤ TODDLER uses toddler animations, ≤ CHILD uses child, and everything older uses adult — [sims4 decompiled sim_info_types.py](https://github.com/jafffy/sims4-modding-framework/blob/HEAD/data/sims4-decompiled/simulation/sims/sim_info_types.py) [fetched].
- **Resource flags** add `Infant = 0x80`, and gender bits `Male = 0x1000, Female = 0x2000` are combined into one `AgeGender` field — [TS4SimRipper Enums.cs](https://github.com/CmarNYC-Tools/TS4SimRipper/blob/HEAD/src/Enums.cs) [fetched].
- **Identity storage** (`BlobSimFacialCustomizationData`): `sculpts` (uint64 ids), `face_modifiers` and `body_modifiers` (key, amount float), **and separate `aged_face_modifiers` and `aged_body_modifiers`** — [TS4ProtobufStubs PersistenceBlobs_pb2.pyi](https://github.com/TURBODRIVER/TS4ProtobufStubs/blob/HEAD/protocolbuffers/PersistenceBlobs_pb2.pyi) [fetched].
- **SimInfo** persists `physique` as a comma-separated string of HEAVY/LEAN/FIT/BONY blend values, truncated to 1/1000, plus `facial_attr` bytes — [sims4 decompiled sim_info.py](https://github.com/jafffy/sims4-modding-framework/blob/HEAD/data/sims4-decompiled/simulation/sims/sim_info.py) [fetched].
- **A morph resource (SMOD/SimModifier) is tied to one age and gender.** It holds `ageGender`, `region`, `subRegion`, `bonePoseKey`, `deformerMapShapeKey`, `deformerMapNormalKey` and BGEO keys — [TS4SimRipper SMOD.cs](https://github.com/CmarNYC-Tools/TS4SimRipper/blob/HEAD/src/SMOD.cs) [fetched].
- **Teen to Elder share the adult rig.** SimRipper maps every age from Teen to Elder to the Adult rig (`adjustedAge = currentAge >= Teen && <= Elder ? Adult : currentAge`). For Elders it *adds* a global deformer map `e{m|f}Body_Average_Shape` / `_Normals` on top of the same face and body modifiers — [TS4SimRipper Form1.cs](https://github.com/CmarNYC-Tools/TS4SimRipper/blob/HEAD/src/Form1.cs) [fetched].
- **Ageing transitions.** `change_age()` sets `age_progress = 0`, calls the native `apply_age(new_age)`, re-evaluates appearance modifiers, resends physical attributes, and updates the rig. Age progress is measured in Sim days. Each transition has `_age_durations` for fast/normal/slow and a `trait_age_duration_mutliplier` mapping — [aging_mixin.py](https://github.com/jafffy/sims4-modding-framework/blob/HEAD/data/sims4-decompiled/simulation/sims/aging/aging_mixin.py), [aging_transition.py](https://github.com/jafffy/sims4-modding-framework/blob/HEAD/data/sims4-decompiled/simulation/sims/aging/aging_transition.py) [fetched].
- **Child duration example:** `age_fast 7, age_normal 14, age_slow 56` Sim days, `_initial_age_randomization_limit 14.29`. This comes from a **mod's override** of `agingTransition_Human_Child`, so the vanilla value may differ — [IWNBedwetting-Extended-Plus agingTransition_Human_Child.xml](https://github.com/LilNinthel/IWNBedwetting-Extended-Plus/blob/HEAD/src/Overrides/Snippet/agingTransition_Human_Child.xml) [fetched].

**Mount & Blade II: Bannerlord**
- **Identity versus dynamic state.** `BodyProperties = DynamicBodyProperties(Age, Weight, Build) + StaticBodyProperties(KeyPart1..KeyPart8, each ulong)`. Constants: `MaxAge = 128f`, `MaxAgeTeenager = 21f`, `DefaultAge = 30f`. `ClampForMultiplayer` clamps age to `[22, 128]`. Face sliders are bit-packed into the KeyParts (`(num & ~mask) | (value << startBit)`) — [BodyProperties.cs](https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/BodyProperties.cs), [DynamicBodyProperties.cs](https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/DynamicBodyProperties.cs) [fetched].
- **Mesh maturity classes:** `enum BodyMeshMaturityType { Toddler, Child, Tween, Teenager, Adult }`, chosen by `FaceGen.GetMaturityTypeWithAge(age)` in native code — [BodyMeshMaturityType.cs](https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/BodyMeshMaturityType.cs), [FaceGen.cs](https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/FaceGen.cs) [fetched].
- **Face generator age slider:** min 3 years (25 in multiplayer), max 128. Changing age calls `EnforceConstraints` and re-picks voice when the maturity type changes. Deform-key data, hair, beard and colour gradients are queried per `(race, gender, age)` — [FaceGenVM.cs](https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.MountAndBlade.ViewModelCollection/FaceGenerator/FaceGenVM.cs) [fetched].
- **Campaign ages:** `BecomeInfantAge 3, BecomeChildAge 6, BecomeTeenagerAge 14, HeroComesOfAge 18, MiddleAdultHoodAge 35, BecomeOldAge 55, MaxAge 128` — [DefaultAgeModel.cs](https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.CampaignSystem/GameComponents/DefaultAgeModel.cs) [fetched].
- **Age as a morph key.** In `skins.xml`, each skin has `mesh_maturity_type="adult"` and a list of `deform_key`s that address `key_time_point` slots on a morph track (face keys 1–61), with `height` at `key_time_point="62"` and **`age` at `key_time_point="63"`** — [BRE_RP ModuleData/skins.xml (mod copy)](https://github.com/BRE-Devlopment/BRE_RP/blob/HEAD/ModuleData/skins.xml) [fetched].

**MakeHuman / Anny (HeroBody's base)**
- **MakeHuman age macro.** `MIN_AGE = 1.0, MID_AGE = 25.0, MAX_AGE = 90.0` years, with slider a ∈ [0,1] and anchors baby 1 y (a = 0), child 10 y (a = 0.1875), young 25 y (a = 0.5), old 90 y (a = 1).
  - Years → a: `a = (y−1)/48` for y < 25, else `a = 0.5 + (y−25)/130`.
  - Weights for a < 0.5: `baby = max(0, 1−5.333a)`, `young = max(0, 3.2(a−0.1875))`, `child = max(0, min(1, 5.333a) − young)`, `old = 0`.
  - Weights for a ≥ 0.5: `old = 2a−1`, `young = 1−old`.
  - The docstring of `setAge` still says "1 is 70", a stale comment.
  - Source: [makehuman apps/human.py](https://github.com/makehumancommunity/makehuman/blob/HEAD/makehuman/apps/human.py) [fetched].
- **Anny anchors.** Anny 0.6 places the age anchors at `torch.linspace(-1/3, 1.0, 5)` = −1/3, 0, 1/3, 2/3, 1 for newborn, baby, child, young, old. That adds a newborn anchor below MakeHuman's baby — [naver/anny phenotype.py](https://github.com/naver/anny/blob/HEAD/src/anny/models/phenotype.py) [fetched]. The sibling note `proportions_by_age.md` gives Anny's WHO-calibrated morphological-age map: Anny age 0, 0.05, 0.215, 0.415, 0.67, 0.77, 0.83, 1.0 ↔ 0, 1, 4, 11, 16, 18, 64, 110 years.

MakeHuman target weights by age [computed from the formulas above]:

| age (y) | slider a | baby | child | young | old |
|---|---|---|---|---|---|
| 1 | 0.000 | 1.00 | 0.00 | 0.00 | 0.00 |
| 3 | 0.042 | 0.78 | 0.22 | 0.00 | 0.00 |
| 5 | 0.083 | 0.56 | 0.44 | 0.00 | 0.00 |
| 7 | 0.125 | 0.33 | 0.67 | 0.00 | 0.00 |
| 10 | 0.1875 | 0.00 | 1.00 | 0.00 | 0.00 |
| 12 | 0.229 | 0.00 | 0.87 | 0.13 | 0.00 |
| 15 | 0.292 | 0.00 | 0.67 | 0.33 | 0.00 |
| 18 | 0.354 | 0.00 | 0.47 | 0.53 | 0.00 |
| 21 | 0.417 | 0.00 | 0.27 | 0.73 | 0.00 |
| 25 | 0.500 | 0.00 | 0.00 | 1.00 | 0.00 |
| 35 | 0.577 | 0.00 | 0.00 | 0.85 | 0.15 |
| 45 | 0.654 | 0.00 | 0.00 | 0.69 | 0.31 |
| 60 | 0.769 | 0.00 | 0.00 | 0.46 | 0.54 |
| 75 | 0.885 | 0.00 | 0.00 | 0.23 | 0.77 |
| 90 | 1.000 | 0.00 | 0.00 | 0.00 | 1.00 |

**MetaHuman**
- **Age is metadata only.** The MetaHuman DNA file has layers Descriptor, Definition, Behavior and Geometry. The Descriptor holds "Name of the character, **Age**, Facial archetype, arbitrary key/value metadata, compatibility parameters". The API has `getAge()` and `setAge(age)`. Age is a descriptor field, not a deformation parameter. The repo notes that characters made in UE 5.6 need the new MetaHuman for Maya plugin — [MetaHuman-DNA-Calibration docs/dna.md](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/HEAD/docs/dna.md), [docs/dna_api.md](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/HEAD/docs/dna_api.md), [README](https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/HEAD/README.md) [fetched].
- **No children.** MetaHuman Creator does not officially support children. Its body presets are adult heights and builds [inference, unverified this session; the brief states it as known].

**Character Creator 4 (Reallusion), Daz Genesis 8/9, Black Desert, Spore, Fable.** Background knowledge only; I could not verify any of it this session [inference].
- **CC4:** age is handled with head/body morph sliders and paid content packs (age and kid/teen morph packs, old-age skin and wrinkle textures). Identity is the morph slider set, and age is applied as extra morphs on top. Age is not a first-class continuous parameter with curves.
- **Daz Genesis 8/9:** children and teens are separate morph products layered on the same base figure. The rig adapts through "auto-follow" joint-centre correction (ERC/rigging follows the shape). The face identity is a separate head morph, so a child version = identity morph + child body/head morph, often with the identity dialled down. This is the same idea as CK3's `child_features` multiplier, done by hand.
- **Black Desert:** a very detailed adult-only creator with per-class fixed body bases. No ageing.
- **Spore:** creatures have baby forms (smaller, with a larger head relative to the body, made procedurally from the adult creature). This is an example of making a juvenile from the adult identity by rule.
- **Fable (I–III):** the hero ages through the story in fixed steps (boy → young adult → older), plus morality and lifestyle morphs (fat, muscle, corruption). Identity is fixed by the story, and ageing is a set of discrete authored stages.

### Inferences
- **Lessons from the Sims 4 and Bannerlord** [inference]:
  - Both switch **mesh or rig class** at a few fixed ages: toddler/child/teen-adult in the Sims, and five maturity types in Bannerlord.
  - Within the adult class, age is continuous: Bannerlord's age key on time point 63, and the Sims elder deformer layered on adult.
  - HeroBody does not need separate rigs because Anny covers 0–110 years on one topology. It does need **one bone set with age-dependent joint positions** (already "joints follow the shape") and **age-banded pose correctives**, because infant joint ranges differ.
- **The Sims 4 keeps an "aged" copy of the modifiers.** The protobuf stores `aged_face_modifiers`/`aged_body_modifiers` *next to* the young ones. So when a Sim crosses into a new stage, the game can keep a separately authored or derived face for the aged stage instead of recomputing it. That is a direct precedent for **per-stage overrides stored alongside identity** [inference; field semantics not documented in the stub].
- **MakeHuman's age weights are piecewise linear with kinks at exactly 10, 25 and 90 years.** That explains why a pure "age knob" looks wrong at the baby end (Phase 1 finding). Real growth is very non-linear in 0–2 years. HeroBody should drive the Anny age parameter from a *morphological* age map (Anny's WHO calibration), not from calendar years directly [inference].

### Gaps
- Primary documentation for CC4 age morphs, Daz child/teen morph packs, MetaHuman age limits, Black Desert, Spore baby rules and Fable ageing could not be fetched.
- The vanilla Sims 4 durations per stage (baby, infant, toddler, teen, young adult, adult, elder) could not be fetched; only one mod-override value (child) was found.
- Bannerlord's native `GetMaturityTypeWithAge` thresholds live in native code and were not visible. Presumably they line up with the campaign model (3/6/14/18), but that is unverified.

---

## 3. Data design: identity separate from age, and handcrafted age overrides

### Takeaway
All the systems examined store **identity once** and **compute the age look on demand**:
- CK3: DNA bytes × per-setting age curves.
- Bannerlord: static face keys + a dynamic age float.
- Sims 4: modifiers + a stage enum, with aged modifier lists kept beside them.
- MakeHuman: slider values + an age macro.

Designer overrides come in four forms:
- **(1) Fixed authored DNA for named characters:** CK3/Vic3 `dna_data` and Bannerlord body-property strings.
- **(2) A forced ageing template:** CK3 `forced_portrait_age_index` → `gene_age` template, and `no_aging` for frozen age.
- **(3) Stage-specific modifier sets:** Sims 4 `aged_*_modifiers`, and per-age SMOD/BGEO resources.
- **(4) Gameplay modifiers that fade in on age:** CK3 portrait modifiers and decals with `alpha_curve` keyed to "character age%".

### Cited Findings
- **CK3 named-character DNA** is a full gene dump with a `portrait_info.age` field (normalised /100). It is the handcrafted identity for a historical person, and the game ages it with the same curves as everyone else — [carlos_v.txt](https://github.com/Radsterman/Victorian-Flavor-Mod/blob/HEAD/common/dna_data/carlos_v.txt) [fetched].
- **CK3 forced ageing template:** `old_beauty_1 … generic = no # Can't be selected unless the character has a trait with forced_portrait_age_index matching this` — [Elf-Destiny override](https://github.com/GalacticLiaison/Elf-Destiny/blob/HEAD/common/genes/zzz_elf_destiny_gene_overrides.txt) [fetched].
- **CK3 frozen age:** `no_aging` (index 5) exists in vanilla but is empty; a mod fills it so that "characters whose age is frozen" keep infant proportions — [same](https://github.com/GalacticLiaison/Elf-Destiny/blob/HEAD/common/genes/zzz_elf_destiny_gene_overrides.txt) [fetched].
- **CK3 versioning:** `GENE_DATABASE_VERSION = 3 # If the version saved in a file does not match this number, the portrait DNAs will get regenerated`. Gene schema changes invalidate stored DNA — [CK3 defines](https://github.com/alltheatreides/AVE_MARIA_CK3/blob/HEAD/common/defines/00_defines.txt) [fetched].
- **Bannerlord** separates `StaticBodyProperties` (identity keys) from `DynamicBodyProperties` (age, weight, build). Equality and hashing are defined per part — [BodyProperties.cs](https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/BodyProperties.cs) [fetched].
- **Bannerlord location age ranges.** `DefaultAgeModel.GetAgeLimitForLocation` gives authored age ranges per role, e.g. 20–28, 20–60, 50–70, 60–90. Designers constrain the age of generated characters per role instead of authoring each one — [DefaultAgeModel.cs](https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.CampaignSystem/GameComponents/DefaultAgeModel.cs) [fetched].
- **Sims 4:** separate `aged_face_modifiers`/`aged_body_modifiers`, and SMODs tagged with one `ageGender` each — [PersistenceBlobs stub](https://github.com/TURBODRIVER/TS4ProtobufStubs/blob/HEAD/protocolbuffers/PersistenceBlobs_pb2.pyi), [SMOD.cs](https://github.com/CmarNYC-Tools/TS4SimRipper/blob/HEAD/src/SMOD.cs) [fetched].

### Inferences
Design rules HeroBody can copy [inference]:
1. **Identity in a unit-free, age-free space.** Store it as z-scores, or as 0–1 or −1..1 "gene" values relative to the population at the same age and sex. Do not store it as centimetres at one age. CK3 stores bytes, and the age curve turns them into an effect at each age. HeroBody's 30 knobs are in measurement terms, so identity = per-knob *deviation* from the population curve, in SD units or as a log-ratio.
2. **An age curve per parameter, with two modes:** `multiply` (identity expression fades in or out) and `add` (an age effect independent of identity, such as gauntness or greying).
3. **Templates for ageing style:** one character ages "graciously", another "hard". Each is a named set of curves, chosen per character (CK3 `gene_age` templates).
4. **Overrides are data, not edits.** A frozen age (`no_aging`), a forced template, or a per-stage set of modifier deltas (Sims `aged_*`). They live beside identity and are versioned (`GENE_DATABASE_VERSION`).
5. **Version the parameter schema.** When knobs change, regenerate derived bodies from the profile. Never patch meshes (this matches "zero manual mesh cleanup").

### Gaps
- No primary design document (GDC talk or dev diary) on why CK3 or the Sims chose these structures could be read. The designs above are reverse-engineered from data files.

---

## 4. Animation and anime practice: younger and older versions of a character

### Takeaway
No primary production source could be fetched this session. The practice below is background knowledge, tagged **[inference, unverified]**. The core idea matches the game systems:
- Studios draw a **separate settei (model sheet) per age variant**: turnaround, expressions, height chart against other characters.
- They keep a short list of **identity markers fixed**: hair silhouette and colour, eye shape and colour, signature marks, and the costume motif.
- They let **proportions change by rule**: head-to-body ratio, eye size relative to the head, nose and jaw definition, line count, limb length.

### Cited Findings
- None fetched. Sakugabooru, Anime News Network, Cartoon Brew and the One Piece wiki were all blocked (proxy 403).

### Inferences
Background knowledge, unverified this session:
- **Model sheets per age.** Anime studios issue *settei* sheets for each distinct look: child flashback, teen, adult, post-timeskip. Each has front/side/back turnarounds, expression sheets and a size comparison (*kurabe*, height lineup). A child version is a new design that the character designer approves, not a scaled adult [inference].
- **What stays constant:** hair silhouette and colour, eye colour and basic eye shape (corner angle, lash style), signature marks, colour palette, and personality-linked expressions. **What changes:** heads tall (children about 3–5 heads; adults 6–8+ in realistic shows; chibi 2–3), eye size relative to the face (larger in children), face shape (rounder and shorter lower face in children; longer jaw and more defined cheekbones in adults), neck length, fewer lines and less muscle definition in children, wrinkles, sag and line accents in the elderly [inference].
- **Scars and acquired marks have an age of onset.** One Piece examples:
  - Luffy's scar under his left eye dates from his childhood (about 7 years old in the manga's flashback); his chest X-scar dates from Marineford (17 years).
  - Zoro's left-eye scar appears only after the two-year timeskip (19+ years).
  - Shanks loses his left arm in Luffy's childhood flashback.
  - So *identity features are not all constant: some begin at an event age*. HeroBody needs an `acquired_features` list with `from_age` [inference; background knowledge of the manga].
- **Studio and manga override.** For a specific age, the drawn design (manga panel or model sheet) is ground truth, even when it is not "realistic". This matches the user's rule: manga images override the realistic curve per stage [inference].
- **One Piece style ranges.** Ages and body sizes vary wildly. Some characters are drawn huge or tiny, and the very old are sometimes drawn spry (Dr. Kureha, said to be 139 years old). So **apparent age must be a separate field from chronological age**, as in CK3 graceful ageing [inference].

### Gaps
- No primary production references: settei examples, studio style-guide pages, interviews on age-variant design. Follow-up targets: published *settei* art books (One Piece Animation Settei), Toei production interviews, and the Sakugabooru wiki "settei" page.
- No measured "heads tall by age" data for anime designs. The anime ratio of an age variant has to be measured from the user's approved blueprints with `hb/measure2d.py`.

---

## 5. Recommended data model for HeroBody ageing

### Takeaway
Combine:
- CK3's **identity × age-curve** structure (multiply/add per parameter, x = years),
- Anny/MakeHuman's **continuous age morph** (driven from a morphological age map),
- the Sims and Bannerlord **stage classes** (for rig correctives and UI only),
- **per-stage artist overrides** stored as sparse keyframes measured from approved blueprints or manga panels (Sims `aged_*` and CK3 `dna_data` precedent).

Height stays last as a uniform scale. Sex is a fixed fact. Everything is solved from the character's own numbers, with nothing copied from another character.

### Cited Findings
- The precedents are cited in sections 1–3. The Anny anchors are cited in section 2 and the sibling note `proportions_by_age.md`.

### Inferences — the model (code-ready)

**5.1 Time axis and stages**

Chronological age `t` is in years (float). Stage anchors, with each stage running from its start to the next stage's start:

| stage | start (y) | end (y) | in-stage slider u = 0 | u = 1 | why these numbers |
|---|---|---|---|---|---|
| baby | 0.0 | 3.0 | newborn | 3 y | CK3 TODDLER_AGE 3, Bannerlord BecomeInfantAge 3; growth is very fast 0–2 y |
| child | 3.0 | 12.0 | 3 y | 12 y | CK3 ADOLESCENCE_AGE 12; pre-puberty curve ends 10–15 y |
| teen | 12.0 | 18.0 | 12 y | 18 y | CK3 PORTRAIT_ADULT_AGE 18, Bannerlord HeroComesOfAge 18; infant joints reach 0 at 18 y |
| adult | 18.0 | 60.0 | 18 y | 60 y | CK3 aging_primary starts at 25 y and reaches 0.7 at 60 y |
| old | 60.0 | 100.0 | 60 y | 100 y | CK3 MAX_AGE 100; Bannerlord BecomeOldAge 55 |

- Mapping: `t = start[s] + u × (end[s] − start[s])`, with u ∈ [0,1].
- Stage starts are **per-profile overridable**. A One Piece character can have `old.start = 70`, and a long-lived race different values.
- Optional per-stage easing `u' = ease(u)` for UI only. Never use easing inside the curves.

**5.2 Apparent age (graceful ageing)**

```
A_app(t) = t                                    if t <= a0
         = a0 + (t - a0) * r                    while a0 + (t-a0)*r < a1
         = a1 + (t - t1)                        after that, where t1 = a0 + (a1-a0)/r
defaults: a0 = 25 y, a1 = 70 y, r = 1.0 (no slowdown); profile field ageing_rate r in (0, 2]
frozen:   profile.freeze_age = A_f  ->  A_app(t) = min(t, A_f)   (CK3 no_aging)
```
All body curves read **A_app**. Gameplay and story read **t**.

**5.3 Parameters and curves**

For every HeroBody knob k (the 30 measurement knobs, plus face-module params, fat/muscle condition knobs and skin/hair appearance params):
```
value_k(t) = P_k(A_app, sex)                       # population curve (WHO/CDC/NHANES etc., sibling notes)
           + E_k(A_app) * I_k * SD_k(A_app, sex)    # identity: z-score I_k, expression curve E_k ("multiply")
           + G_k(A_app; template)                   # age effect independent of identity ("add"), e.g. sag, gauntness
           + S_k(A_app)                             # per-character style offset (anime look), may itself fade by age
           + O_k(t)                                 # artist override correction (5.5)
then clamp to [min_k(A_app), max_k(A_app)]  (Phase 1 atlas safe range)
height applied last: uniform scale so that stature = value_height(t)
```
- `I_k` is **fixed for life** (identity). It is a z-score fitted on the approved adult blueprint, or on the stage the user approved first.
- `E_k` default shapes (copy the CK3 numbers, then tune):
  - Face shape knobs: `E = child_features` (1.0 at 0 y → 0.53 at 6 y → 1.0 at 22 y). Or simpler: `E = 0.5 + 0.5·smoothstep(3, 18, A)`.
  - Stature and limb ratios: `E = height curve` (0 at birth → 1.0 at 18 y → 0.6 at 90 y), i.e. tall people are not tall babies.
  - Fat and muscle condition: `E = child_fat` (0.2 at 0 y → 1.0 at 18 y → 0.2 at 95 y).
- `G_k` default shapes (CK3 ageing presets): `aging_primary` (0 → 1.0 between 25 and 75 y) for global old-age blendshapes. `aging_secondary` (0 → 0.5 between 50/55 and 85/90 y) for nose and ear growth and jaw forward. `aging_tertiary` (0 → ±0.5 between 70 and 95 y) for mouth and lip thinning. `hunchback` (0 → 1.0 between 70 and 95 y), which drives the posture corrective (kyphosis), not a mesh edit. `aging_hsv` (0 → 1.0 between 35 and 70 y) for hair greying, which is per-character adjustable.
- **Ageing templates:** `gracious`, `average`, `hard`, `frozen`. Each is a dict of curve scalings (CK3 `old_1..4`, `old_beauty_1`, `no_aging`).

**5.4 Curve representation**

```json
{"curve_id": "child_features", "mode": "multiply", "x_unit": "years",
 "points": [[0,1.0],[5,0.5],[10,0.65],[22,1.0]], "interp": "pchip", "extrapolate": "clamp"}
```
- Use years on the x axis, not age/100, so artists can read it.
- Use monotone cubic (PCHIP) interpolation, not linear, so that knob velocities are smooth for in-between animation. Linear (CK3 style) is fine for v1.

**5.5 Artist overrides per stage (manga image wins)**

- An override is a **sparse keyframe set**: `{at_age: t_i, source: "manga vol 1 ch 1 p 3" | blueprint id, knobs: {k: target_value_k}, weight: 1.0, locked: [k...]}`.
- Targets come from measuring an approved image with `hb/measure2d.py`, never from mesh edits.
- Correction: `D_i,k = target_i,k − model_k(t_i)` without O.
- Blend: `O_k(t) = Σ_i w_i(t) · D_i,k`, where w_i is a piecewise-linear "hat" partition of unity between neighbouring override ages. Weight is 1 at t_i and falls to 0 at the neighbouring override age or at the stage boundary, whichever is nearer, if `scope = "stage"`. With `scope = "global"` it decays over a profile-set half-width (default 3 years).
- Optional scope `exact` = only at t_i (a one-off cutaway).
- **Locked knobs** (identity markers) cannot be moved by overrides.
- **Approval gate:** an override is accepted only if the resulting body still passes the HeroBody blueprint checks (every ratio within 3%, cross-view agreement ≤ 1% of height).

**5.6 Character profile (human verifies; nothing else is manual)**

```yaml
profile_version: 1            # bump -> regenerate all bodies (CK3 GENE_DATABASE_VERSION)
id: zoro
basic:
  name: Roronoa Zoro
  sex: male                   # fact, never fitted; Anny gender 0 = male
  species: human              # or fishman, mink, giant... -> population curve set
  birth: {year: null, birthday: "11-11"}
  canon_ages: [9, 19, 21]     # ages with approved references
  story_age_now: 21
ageing:
  stages: {baby: 0, child: 3, teen: 12, adult: 18, old: 60}   # overridable
  ageing_template: average     # gracious | average | hard | frozen
  ageing_rate: 1.0             # r in A_app; <1 looks younger
  freeze_age: null             # years; CK3 no_aging
  greying_onset: 35            # years, shifts aging_hsv
identity:                      # fixed for life, z-scores vs population at same age/sex
  body_z: {stature: +1.2, leg_ratio: +0.8, shoulder_width: +1.5, ...}   # the 30 knobs
  face_module: {eye_size: ..., eye_spacing: ..., jaw_width: ..., chin: ...}  # anime head module, head-radius units
  style_offsets: {head_scale: 1.0, waist: -0.1, ...}  # anime look, per-knob, with fade curve id
  markers_locked: [hair_silhouette, eye_shape, eye_color, hair_color]
acquired_features:             # identity features with an onset age
  - {id: scar_left_eye, type: decal+normal, socket: face, from_age: 19}
  - {id: chest_scar, from_age: 19}
body_conditions:               # time ranges; each modifies knobs or rig
  - {type: weight, from_age: 30, to_age: null, fatness_z: +1.0}
  - {type: muscle, from_age: 15, muscle_z: +2.5}
  - {type: amputation, limb: left_arm, level: above_elbow, from_age: 27}   # Shanks-type
  - {type: prosthesis, limb: left_leg, kind: peg, from_age: 40}
  - {type: mobility_aid, kind: cane|crutch|wheelchair|walker, from_age: 75, side: right}
  - {type: posture, kind: kyphosis, severity: 0.5, from_age: 80}   # overrides hunchback curve
  - {type: paralysis|contracture|limb_length_diff|scoliosis, ...}
  - {type: pregnancy, from_age: 25.2, to_age: 25.9}
  - {type: devil_fruit_body, kind: rubber}                # links Rubber Body Lab stretch rules
addons: [{socket: ..., asset: horns_01, scale_with: head_size, from_age: 0}]
hair: {style_by_stage: {child: hair_child_01, adult: hair_adult_02}, color: ...}
garments: [{id: coat_01, from_age: 19, to_age: null}]    # resize via body
overrides:                     # 5.5, manga/blueprint wins
  - {at_age: 9, scope: stage, source: "blueprint zoro_child_v2", knobs: {head_to_height: 0.19, leg_ratio: 0.44}}
  - {at_age: 19, scope: stage, source: "blueprint zoro_v3 (approved)"}
approval: {by: user, date: 2026-10-09, checks_passed: true}
```

**5.7 Evaluation order (per frame or per requested age)**

1. Get t, A_app.
2. Population P (sex, species).
3. Add identity `E·I·SD`.
4. Add the template add-curves G.
5. Add the style offsets S.
6. Apply overrides O.
7. Apply body conditions (fat/muscle z, amputations remove or replace geometry, posture correctives).
8. Clamp to the atlas safe range.
9. Knob solver → 44 Anny params. The Anny age param comes from **morphological age** (WHO-calibrated map), not t.
10. Anime head module with its own age curves (eye size up, lower face shorter in children).
11. Anatomy registration + per-body clamp.
12. Height last (uniform scale).
13. Sockets, hair, garments refit.

Cache bodies at stage anchors and at override ages. For in-between frames, interpolate *knob values*, not meshes.

### Gaps
- The `E_k` and `G_k` defaults are CK3 artist curves, not measured data. They should be refitted to measured longitudinal data (sibling notes: Berkeley growth, NHANES) where it exists, and to the anime blueprints for style.
- The SD_k(age) tables have to come from the sibling growth notes. I did not derive them here.

---

## How this plugs into HeroBody

- **New data file** `D:\anime\herobody\data\age_curves.json`: curve library with ids `child_features`, `child_fat`, `height`, `aging_primary`, `aging_secondary`, `aging_tertiary(_reversed)`, `hunchback`, `aging_hsv`, `infant_joints`. Start from the CK3/Vic3 numbers in section 1, with x converted to years (x × 100).
- **Character profile schema** (`profile.yaml`, section 5.6). It extends the existing "character profile" that already stores hair style. Add `ageing`, `identity` (z-scores), `acquired_features`, `body_conditions` (incl. mobility aids), `overrides` and `profile_version`.
- **Knob solver (open build item):** its input becomes `value_k(t)` from 5.3 instead of fixed targets. Height stays last. Identity z-scores I_k are fitted once from the approved stage blueprint: solve the blueprint ratios, then convert them to z against P_k and SD_k at that age.
- **Anny age parameter:** drive it from a morphological-age map (Anny `shape_calibration` anchors 0, 1, 4, 11, 16, 18, 64, 110 y), with a **baby-end corrective blendshape** for 0–3 years (Phase 1 found that the age knob breaks at the baby end). MakeHuman's linear age weights have kinks at 10/25/90 y (table in section 2). Do not feed calendar years straight in.
- **Four new blendshapes:** "head bigger than 2×" and "chibi short legs" become `style_offsets` with their own fade curve per stage. A child-stage manga override usually needs them more than an adult override does.
- **The fat layer (open build item):** driven by `child_fat`-type E curves × the `fatness_z` body condition, plus the `aging_gauntness` add-curve for the elderly.
- **Pose correctives (open build item):** add an age-banded set (infant 0–3, child 3–12, adult, old with kyphosis from the `hunchback` curve or a posture condition). This mirrors the Sims and Bannerlord maturity classes without separate rigs. The same 53-bone Unreal game rig is used at every age, with joints from the shape.
- **Anime head module:** gets its own E/G curves (eye size, lower face height, jaw definition) and a list of locked identity markers. Acquired features (scars) are decals on the head module with `from_age`.
- **2D-first loop:** a manga or blueprint image at an age → `hb/measure2d.py` → override keyframe (5.5) → the solver re-runs → approval checks (ratios within 3%, view agreement ≤ 1% H). No mesh edits.
- **Anatomy clamp:** runs after every age evaluation. Organs scale by the sibling `organs_and_anatomy_by_age.md` rules.

---

## Sources

- amtep/tiger, CK3/Jomini validator gene grammar: https://github.com/amtep/tiger/blob/HEAD/src/data/genes.rs
- CK3 00_defines.txt (copy in AVE_MARIA_CK3): https://github.com/alltheatreides/AVE_MARIA_CK3/blob/HEAD/common/defines/00_defines.txt
- CK3 defines copies with life stages (Warcraft GoA, Calradian Kings, LuxRenata), found by GitHub code search: https://github.com/Warcraft-GoA-Development-Team/Warcraft-Guardians-of-Azeroth-2/blob/HEAD/common/defines/00_defines.txt ; https://github.com/Calradian-Kings-Mod-Team/Calradian_Kings/blob/HEAD/common/defines/00_defines.txt ; https://github.com/Cameron122HGG/LuxRenataCK3/blob/HEAD/common/defines/00_defines.txt
- CK3 gene_age + age presets (AGOT mod override): https://github.com/Sohex/temp-agot/blob/HEAD/2962333032/common/genes/gene_age_override.txt
- CK3 gene_age, body shape (Elf Destiny overrides): https://github.com/GalacticLiaison/Elf-Destiny/blob/HEAD/common/genes/zzz_elf_destiny_gene_overrides.txt
- CK3 aging gene (CBO Unofficial Base): https://github.com/csirke128/CBO-Base/blob/HEAD/CBO%20Unofficial%20Base/common/genes/aging%20gene.txt
- Warcraft GoA gene_sexual_dimorphism: https://github.com/Warcraft-GoA-Development-Team/Warcraft-Guardians-of-Azeroth-2/blob/HEAD/common/genes/wc_genes_sexual_dimorphism.txt
- CK3 ethnicity example (CK3-Human-Phenotype-Project): https://github.com/Metalhead33/CK3-Human-Phenotype-Project/blob/HEAD/common/ethnicities/01_brunn.txt
- Victoria 3 vanilla 01_genes_morph.txt (Jomini age presets): https://github.com/tjysdsg/Victoria-3-files/blob/HEAD/game/common/genes/01_genes_morph.txt
- Jomini DNA record example (Vic3 mod): https://github.com/Radsterman/Victorian-Flavor-Mod/blob/HEAD/common/dna_data/carlos_v.txt
- Bannerlord decompiled source 1.3.15: https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/BodyProperties.cs ; https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/DynamicBodyProperties.cs ; https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/StaticBodyProperties.cs ; https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/BodyMeshMaturityType.cs ; https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.Core/FaceGen.cs ; https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.MountAndBlade.ViewModelCollection/FaceGenerator/FaceGenVM.cs ; https://github.com/BannerlordCode/bannerlord-1.3.15/blob/HEAD/TaleWorlds.CampaignSystem/GameComponents/DefaultAgeModel.cs
- Bannerlord skins.xml (mod copy): https://github.com/BRE-Devlopment/BRE_RP/blob/HEAD/ModuleData/skins.xml
- The Sims 4 decompiled Python: https://github.com/jafffy/sims4-modding-framework/blob/HEAD/data/sims4-decompiled/simulation/sims/sim_info_types.py ; https://github.com/jafffy/sims4-modding-framework/blob/HEAD/data/sims4-decompiled/simulation/sims/aging/aging_mixin.py ; https://github.com/jafffy/sims4-modding-framework/blob/HEAD/data/sims4-decompiled/simulation/sims/aging/aging_transition.py ; https://github.com/jafffy/sims4-modding-framework/blob/HEAD/data/sims4-decompiled/simulation/sims/sim_info.py
- The Sims 4 protobuf stubs: https://github.com/TURBODRIVER/TS4ProtobufStubs/blob/HEAD/protocolbuffers/PersistenceBlobs_pb2.pyi
- TS4 SimRipper source: https://github.com/CmarNYC-Tools/TS4SimRipper/blob/HEAD/src/SMOD.cs ; https://github.com/CmarNYC-Tools/TS4SimRipper/blob/HEAD/src/Enums.cs ; https://github.com/CmarNYC-Tools/TS4SimRipper/blob/HEAD/src/Form1.cs
- Sims 4 child ageing tuning (mod override): https://github.com/LilNinthel/IWNBedwetting-Extended-Plus/blob/HEAD/src/Overrides/Snippet/agingTransition_Human_Child.xml
- MakeHuman human.py (age macro): https://github.com/makehumancommunity/makehuman/blob/HEAD/makehuman/apps/human.py
- Anny phenotype.py (age anchors): https://github.com/naver/anny/blob/HEAD/src/anny/models/phenotype.py ; README: https://github.com/naver/anny/blob/HEAD/README.md
- MetaHuman DNA Calibration: https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/HEAD/README.md ; https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/HEAD/docs/dna.md ; https://github.com/EpicGames/MetaHuman-DNA-Calibration/blob/HEAD/docs/dna_api.md
- Blocked this session (not read): https://ck3.paradoxwikis.com/Genes ; https://apidoc.bannerlord.com ; https://simswiki.info ; https://en.wikipedia.org ; https://manual.reallusion.com ; daz3d.com ; dev.epicgames.com ; fandom wikis (sims, spore, fable, bannerlord, blackdesert, onepiece) ; sakugabooru.com ; animenewsnetwork.com ; cartoonbrew.com
