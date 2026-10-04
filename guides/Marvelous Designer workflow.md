# Marvelous Designer workflow: anime clothes for an existing 3D body → Unreal Engine 5

Solo creator · no model training · 24GB+ GPU · personal/portfolio use.
Builds on [reports/Solo 3D anime clothes in Unreal.md](../reports/Solo%203D%20anime%20clothes%20in%20Unreal.md).

> **Verify before buying:** MD price, personal-license terms and trial length were **not verified** (the MD support site was blocked during research). Feature claims for 2026.0/2026.1 come from release coverage (The Rookies, Digital Production, CG Channel snippets).

## Versions that matter

| Item | Use | Why |
|---|---|---|
| Marvelous Designer | **2026.1** (Aug 2026) | 2026.0 added VRM/glTF avatar import, 3D Pencil (draw patterns on the body), Toon Shader preview, Unreal LiveSync/USD. 2026.1 added brush-painted folds and keeps UVs on OBJ export. |
| Unreal Engine | **5.8** (or 5.6+) | MD → Chaos Cloth USD workflow needs UE 5.6+; **UE 5.5 is reported incompatible** (snippet). |
| Blender | current 4.x | Cleanup, thickness, weight transfer (Robust Weight Transfer add-on). |

## Phase 0 — Prepare (once per character)

1. **Body export.** Put the body in **A-pose** (arms ~45°), apply scale, units in **cm**. Export **FBX** (or VRM if it is a VRoid/VRM body). Hide/strip hair so it doesn't collide.
2. **Turnaround reference.** Front / side / back of *your* body "dressed" via Qwen-Image-Edit-2511 (ComfyUI, local) with flat colors, no shadows. Clean it into a simple model sheet: one base + one shadow color per garment region (studio settei style).
3. **Simplify the design first (anime rule).** Decide the fold count *before* modeling: anime costumes keep the silhouette and key folds and drop micro-detail. Mark on the sheet which folds are "authored" (painted lines in texture) vs "real" (geometry).

## Phase 1 — Set up in MD

1. **File → Import → Avatar** (FBX/VRM). Let **Auto Fitting** create the fitting suit for humanoid FBX imports — check early how it handles anime proportions (big head, thin limbs); no source tested this.
2. Set **Arrangement Points** on the avatar (MD generates them for humanoids; add missing ones for anime proportions).
3. Load the turnaround images as **reference images in the 2D window** (scale front view to match the avatar height) so you can trace over them.
4. Working **Particle Distance**: start coarse (~15–20 mm) while drafting — faster and fewer micro-wrinkles. Go to ~5–10 mm only for the final simulation (general MD practice, not from research).

## Phase 2 — Build the garments

**Pattern creation**
- Trace panels in 2D with Polygon / Rectangle / Curve tools over the reference, **or** use the **3D Pencil** (2026.0) to draw outlines directly on the body and convert them to patterns — fastest for fitted tops and collars.
- Mirror symmetric panels (**Symmetric Pattern / Clone as Symmetric**) so edits stay paired.
- **Sew** with Segment / Free sewing; check sewing direction (crossed lines = twisted seams).
- Place panels on arrangement points → **Simulate** (spacebar).

**Common anime pieces**

| Piece | How |
|---|---|
| Pleated school skirt | Rectangle panel → **Fold Arrangement / pleat tools** with internal lines; keep 8–16 pleats, set pleat folds stiff. Bones later in Phase 6. |
| Blazer / gakuran jacket | Standard front/back/sleeve panels; add **Fusible / stiff** fabric to collar & lapels so they hold shape. |
| Sailor collar (serafuku) | Separate flat panel sewn to neckline, high bending stiffness, "Freeze" after placement. |
| Ribbon / tie | Separate small panels; often better as a rigid AI-mesh or modeled accessory with 1 bone chain. |
| Long coat / cape | Normal panels; plan for Chaos Cloth or KawaiiPhysics chains. |
| Thigh-highs / tights | Usually **Blender shrinkwrap** from body faces is faster than MD. |

**Make it read as anime, not realistic**
- Fabric: raise **Bending/Buckling stiffness** and use thicker presets (e.g., denim/canvas-like) → fewer, larger folds.
- Use **Pins** and **Freeze** to lock clean silhouettes; use **Pressure** for puffy sleeves.
- Use **Brush Pinching / brush folds** (2026.1) to author 2–4 bold folds where the settei shows them.
- Judge the look with the **Toon Shader preview** (2026.0), not the realistic viewport.
- Delete micro-wrinkles: re-simulate at coarser particle distance, or smooth in Blender later.

## Phase 3 — Game-ready mesh

1. **Retopology:** Mesh type **Quad (Optimized)** (auto retopo). Inspect edge flow at joints (elbows, knees, armpits) — the vendor says manual cleanup may be needed.
2. **Poly budget (suggestion, unsourced):** ~5k–20k tris per outfit for games; higher for cinematic-only.
3. **UVs:** pattern-based UVs come for free → **UV Editor**, pack all panels into one 0–1 tile per material. Group by material (cloth / trim / buttons).
4. Optional: **CLO EveryWear** for poly reduction, UV packing and baking (whether its auto-rig targets a custom skeleton is unconfirmed — prefer Blender for weights).

## Phase 4 — Export from MD

- **FBX**, **cm**, **A-pose (same pose as the body)**, single-layer ("thin") garments, *include* UVs, combine or keep per-garment objects (keep per-garment for outfit switching).
- For hero cinematics only: also export an **Alembic** cache, or **USD with simulation data** → Chaos Cloth Asset (UE 5.6+). Note a reported issue: MD materials can look different after USD import — rebuild materials in UE anyway (toon).
- **LiveSync 2** plugin can round-trip mesh/materials/animation with UE 5.6+ (MetaHuman-focused documentation; test with your skeleton).

## Phase 5 — Blender finishing

1. Import body + garment FBX (same scale).
2. **Solidify** (~3–8 mm) for visible thickness at hems/collars; avoid zero-thickness backfaces in UE.
3. **Texture:** flat 2K albedo, paint fold and seam lines into the texture (anime look). StableProjectorz (free) can project the turnaround onto the existing UVs. No baked lighting.
4. **Swing bones:** add skirt/coat/ribbon bone chains to the *body's* armature (once, e.g., 6–8 skirt chains × 3–4 bones).
5. **Weights:** **Robust Weight Transfer** (free, GPL-3.0) from body → garment; it handles armpits, crotch, skirts. Then weight the skirt to the new chains.
6. **Delete / mask hidden body faces** under tight clothing (or do it in UE via Mutable clipping) to stop poke-through.
7. Export each garment as a **Skeletal Mesh FBX** on the same armature.

## Phase 6 — Unreal Engine 5.8

1. Import each garment → **choose the body's existing Skeleton asset**.
2. Attach to the character Blueprint: **Leader Pose** for garments that just follow the body; **Copy Pose** for garments running their own physics.
3. **Physics:** **KawaiiPhysics** (free, MIT, UE 5.3–5.8) on skirt/ribbon chains; use v1.20+ **SyncBone** to stop skirts clipping legs; add capsule colliders on thighs. **Chaos Cloth** only for long capes/dresses or hero shots. (If animating in Sequencer, drive KawaiiPhysics via the Anim Blueprint — known Sequencer quirk.)
4. **Toon shading:** Substrate Toon (experimental in 5.8, no outlines yet) or a free toon material + **inverted-hull outlines** + a vertex-color mask that pushes folds into shadow (Guilty Gear technique).
5. Render: Movie Render Queue at 4K; step animation on 2s/3s for anime timing.

## Time estimate (unsourced practitioner estimate)

| | First outfit | Later outfits |
|---|---|---|
| MD learning + build | 3–6 days | 0.5–1.5 days |
| Retopo/UV/texture | 1–2 days | 0.5–1 day |
| Bones/weights/UE setup | 2–4 days (one-time systems) | 0.5 day |

## Pitfalls checklist

- [ ] Body and garment exported in the **same pose and units** (A-pose, cm)
- [ ] Realistic micro-wrinkles removed (stiffer fabric, coarse particles, brush folds)
- [ ] Quad mesh checked at joints; no interior/duplicate faces
- [ ] Garments have thickness (Solidify) — no backface holes
- [ ] Hidden body faces deleted/masked
- [ ] UE **not 5.5** for the MD USD workflow
- [ ] KawaiiPhysics colliders on thighs; SyncBone on skirt
- [ ] Flat textures, no baked shadows
