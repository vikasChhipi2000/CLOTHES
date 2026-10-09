# HeroBody Ageing Model

Summary of the HeroBody ageing research (2026-10-09). The final multi-agent synthesis was stopped to save credit, so this is a short summary written from the nine notes files in `research_notes/HeroBody ageing model/`. Read those notes for full numbers, formulas and sources. Five notes have a Verification section from a fact-checker (growth, proportions, body composition, skeleton, face). The other four were not fact-checked: organs, Anny age mechanics, game systems and blueprints.

## Verdict

Ageing fits into HeroBody as **age-driven measurement targets for the knob solver**, plus fixes where Anny breaks at the baby end. Real data exists for nearly every layer. Two layers have no open data: child and elderly organ meshes, and anime-style face ageing.

## Main findings

| Area | Finding | Notes file |
|---|---|---|
| Growth | WHO medians 0–3 y, then the Preece-Baines model with final height as a pure multiplier, plus a SITAR-style age warp for early or late puberty. The LMS formula turns a percentile into height, weight or head size at any age. Real people peak at 8–10 cm/yr at puberty; median curves blur this to about 7 cm/yr. | `growth_stature_trajectories.md` |
| Proportions | Sitting height ÷ height: 0.69 at birth, 0.60 at 2 y, 0.54 at 6 y, 0.51 at 12–14 y, then 0.515 (M) / 0.524 (F) at 18 y. Heads tall: about 4 at birth, 5.3 at 2 y, 6.2 at 6 y, 7.0 at 10 y, 8.0 at 18 y, 7.5–7.7 at 80 y. Shoulder width stays about 0.22 × height and grows only in boys. Pelvis width grows only in girls. Waist ÷ height is lowest (0.41) at 8–12 y and rises to 0.61–0.63 by 65–80 y. | `proportions_by_age.md` |
| Fat and muscle | Body fat peaks at 25–30% at 3–6 months and is lowest at 5–7 y. Puberty splits the sexes. Adults gain fat and lose lean mass; lean mass peaks at 40–50 y. Includes data for the fat layer still to be built. | `body_composition_fat_muscle_by_age.md` |
| Skeleton | Femur ÷ height: 0.15 at birth, 0.20 at 2 y, 0.245 at 8 y, 0.265–0.27 in adults. Per-bone formula: femur_cm = −4.40 + 0.2045·H + 0.000497·H². Legs outgrow arms. The notes also cover posture (kyphosis), joint limits and gait by age. | `skeleton_posture_motion_by_age.md` |
| Organs | ICRP 89 reference masses at 6 ages. The brain is 10.9% of body mass at birth and 2% in adults; the thymus peaks at about 10 y. **No openly licensed child or elderly organ set exists**: ICRP, UF/NCI and XCAT phantoms all require permission or a fee. Scale the adult BodyParts3D/HuBMAP organs using ICRP masses. | `organs_and_anatomy_by_age.md` |
| Face | The cranium grows early and the face late. Real children's eyes are only slightly bigger relative to the head; the main change is that **eye level moves up the head with age**. Head size is already 61.5% of adult at birth. Farkas 1994 norms cover ages 1–18. Teeth and hair timelines are included. | `head_face_teeth_hair_by_age.md` |
| Anny age | One scalar blends 5 anchors (newborn, baby, child, young, old). **Newborn is only the baby shape scaled down**, which is why the baby end breaks. Years-to-Anny-age table: 0 y→0, 1 y→0.05, 4 y→0.215, 11 y→0.415, 16 y→0.67, 18 y→0.77, 64 y→0.83. The weight and muscle age prior looks indexed in the wrong units, which looks like a bug. | `anny_makehuman_age_mechanics.md` |
| Game systems | Crusader Kings 3 stores identity (DNA genes) separately from age (per-gene age curves). Children use separate body types until 18. A gene_age template sets the ageing style, and traits can override it. This is a good model for the HeroBody profile. | `age_progression_systems_games_tools.md` |
| Blueprints | All open face-ageing models are trained on photos, none on anime. Age the anime head **parametrically** (eye height, face length, jaw by stage), with manga images as overrides. FADING is the most usable model, but it has no licence file. | `age_variant_blueprints_2d.md` |

## Recommended design

1. **Identity stays separate from age.** A character profile holds fixed identity knobs (CK3-style DNA) plus adult height, puberty timing and body conditions.
2. **Age is in years.** Stage plus in-stage slider gives years. Years go through the growth model to give stature, and through per-stage ratio tables to give knob targets. The HeroBody knob solver fits them, with height applied last.
3. **New baby-end shapes.** Anny cannot make real infants (newborn is only a scaled baby). Add infant and toddler blendshapes: big head, short legs, round belly, short neck.
4. **Layers by age:** fat-layer thickness from the composition tables, rig bone lengths from the per-bone formulas, rest-pose kyphosis for old age, organs scaled to ICRP masses inside the clamp.
5. **Head module by age:** move eye level up and lengthen the face with age, keeping identity features fixed. Manga references override per stage.

## Next steps

- Check the four notes that were not fact-checked (organs, Anny mechanics, game systems, blueprints) before relying on their numbers.
- In a local session with `D:\anime\herobody`: map these ratio tables onto the 30 knobs in `herobody_spec.json`, and confirm the Anny age-prior bug in the Anny 0.6 code.
