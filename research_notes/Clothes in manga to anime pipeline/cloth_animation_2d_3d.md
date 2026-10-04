# Animating Clothing in Anime-Style Productions: 2D Hand-Drawn, 2D Rig/Deformation, AI In-betweening, and 3D Cel-Shaded Cloth (state as of Oct 2026)

Research note scope: how anime clothing is made to move (folds, skirts, capes, coats, ribbons, wind, secondary motion), with tools, workflows, costs and quality trade-offs. Several primary sources (gamedeveloper.com, ggxrd.com PDF, gameanim.com, docs.live2d.com) were blocked by the network proxy during research; where that happened, details come from search-result excerpts of those pages and are flagged.

---

## 1. 2D traditional principles for cloth, and digital 2D tools that help

### Takeaway
Hand-drawn anime cloth relies on classic follow-through / overlapping action / drag plus an anime-specific vocabulary (nabiki "flapping" for cloth and hair in wind); fabric type dictates fold logic (pleated skirt = accordion bounce, A-line = wave ripple). Among digital tools, Toon Boom Harmony's deformer stack (Envelope, Curve, Deformer-on-Deformer) is the most explicitly cloth-oriented feature set documented; evidence for cloth-specific features in CLIP STUDIO PAINT, TVPaint, OpenToonz and Krita was not gathered.

### Cited Findings
- Anime animation vocabulary: "nabiki" (なびき, "flapping") applies to any cloth, hair or other light object that flaps in wind or when pulled along; the community sakuga wiki treats it as a distinct effect category. Example given: a skirt does not flare at the start of a run out of excitement, it *drags* because of inertia and air resistance — [Sakuga Wiki: Nabiki](https://sakuga.fandom.com/wiki/Nabiki)
- Follow-through: loosely tied parts keep moving after the character stops, then are "pulled back" toward the center of mass with damped oscillation; "drag" is the related idea that when a character starts moving, loose parts take a few frames to catch up; overlapping action = different parts moving at different rates — [Wikipedia: Follow through and overlapping action](https://en.wikipedia.org/wiki/Follow_through_and_overlapping_action); [Twelve basic principles of animation](https://en.wikipedia.org/wiki/Twelve_basic_principles_of_animation)
- Fold logic by fabric type: a pleated skirt has a distinct accordion-like bounce and fold, while a simple A-line skirt ripples and flows like a wave; flowing fabric forms soft curved folds that follow the direction of body motion or wind (practitioner/hobbyist guide, lower authority) — [Lemon8 skirt movement guide](https://www.lemon8-app.com/@koanlicolors/7460925912599740974?region=us); [Skyrye Design: drawing clothing that moves](https://skyryedesign.com/art/drawing/how-to-draw-clothing-that-flows-and-moves/)
- Toon Boom Harmony Premium ships six deformer types: Bone, Game Bone, Curve, Envelope, Free Form and Shape-Aware — [Harmony 22 docs: How to Use Deformers](https://docs.toonboom.com/help/harmony-22/premium/getting-started/deformation.html); [Harmony 25 docs](https://docs.toonboom.com/help/harmony-25/premium/getting-started/deformation.html)
- Harmony's Envelope deformation wraps an envelope around a drawing and deforms it via the envelope's points/curves; Toon Boom explicitly lists "hair, cloaks, shoulders, and chins" as intended uses (fluid-shape parts) — [Harmony 20 docs: About Envelope Deformations](https://docs.toonboom.com/help/harmony-20/premium/deformation/about-envelope-deformation.html)
- Harmony's Deformer-on-Deformer Wizard puts a Curve deformer over an Envelope deformer, so the Curve animates the centreline (e.g., a cape's spine / ribbon path) while the Envelope tweaks the outer silhouette — [Harmony 24 docs: About the Deformer On Deformer Wizard](https://docs.toonboom.com/help/harmony-24/premium/master-controller/about-deformer-on-deformer.html)
- Toon Boom positions deformers as letting cut-out rigs be animated "in a way that likens the fluidity and flexibility of traditional animation" — [Harmony 22 docs](https://docs.toonboom.com/help/harmony-22/premium/getting-started/deformation.html)

### Inferences
- For hand-drawn TV anime, cloth secondary motion is entirely a key-animator/in-betweener drawing cost; there is no "simulation" — the quality depends on the animator's grasp of drag/follow-through timing and fold logic, which is why flowing coats/capes are frequently simplified in model sheets for TV production (inference; consistent with cost data in §9).
- Harmony's Curve-on-Envelope stack is a good fit for capes, long coats tails, scarves and ribbons in cut-out or hybrid productions, because those are "centreline + silhouette" shapes.

### Gaps
- No sources were gathered on cloth-specific features of CLIP STUDIO PAINT (e.g., its animation timeline, 3D reference models, or mesh transform), TVPaint, OpenToonz or Krita; claims about them should not be made from these notes.
- No primary Japanese-industry source (e.g., a sakuga/genga instruction book or animator interview) on fold-drawing conventions was retrieved; the fold-logic guidance above is from hobbyist guides.

---

## 2. 2D rigged deformation for clothing (Live2D, Spine, Moho, After Effects; VTuber rigs)

### Takeaway
Rigged 2D clothing motion is driven by spring/pendulum physics bound to parameters or bones: Live2D Cubism uses per-region physics groups (e.g., "swinging skirt", "swinging bangs") with input/output parameters and pendulum settings; Spine added physics constraints in 4.2 (strength/damping on bones); Moho relies on Smart Bones/pin bones/Smart Warp; AE Puppet is free but clunky. The limit of all these: they deform pre-drawn art, so they cannot produce genuinely new folds or turnaround views of a garment without extra hand-drawn art.

### Cited Findings
- Live2D Cubism physics: groups are created per body region to be swung, such as "swinging bangs" and "swinging skirt"; each group has Input settings (the parameter that acts as the "part that suspends the thread" of the pendulum), Output settings (which parameters receive the computed swing) and Physical Model settings (the pendulum "weight") — [Live2D Manual: About Physics](https://docs.live2d.com/en/cubism-editor-manual/physics-operation/) (page fetch blocked; details from search excerpt); [Live2D Tutorial: Physics Settings](https://docs.live2d.com/en/cubism-editor-tutorials/physical-calculation-settings/)
- Live2D pendulum parameters: Reaction (1 = neutral; >1 more agile; <1 less agile), Duration/amplitude (range of motion), Shaking influence (weight of the pendulum end point — small = light motion, large = heavy powerful swing), Speed of convergence (how fast the swing settles) — [Live2D Manual: Physics Settings](https://docs.live2d.com/4.2/en/cubism-editor-manual/physics-operation/); [How to Set Up Physics](https://docs.live2d.com/en/cubism-editor-manual/physical-operation-setting/)
- Community feedback highlights bulk-editing physics pendulums as a pain point in the Physics window (practitioner forum) — [Live2D Community: Physics window pendulum settings](https://community.live2d.com/discussion/2283/physics-window-pendulum-settings-better-way-to-bulk-edit-traverse-editable-range)
- Spine 4.2 introduced physics constraints that simulate Newtonian forces on bones for secondary motion of hair, clothing and other items, integrated into the skeleton update loop; key parameters include strength (spring back to pose) and damping; Esoteric markets it as removing the need to hand-animate secondary motion and giving reactive motion across animations and world movement — [Spine User Guide: Physics constraints](http://en.esotericsoftware.com/spine-physics-constraints); [Spine blog: "Spine 4.2: The physics revolution"](https://esotericsoftware.com/blog/Spine-4.2-The-physics-revolution)
- Moho (renamed from Anime Studio in 2016) offers Smart Bones (bone-driven corrective shapes with editable motion graphs), pin bones (one-point bones to reshape assets), Smart Warp mesh deformation (Moho 12) and automatic squash-and-stretch on any bone — [Wikipedia: Moho](https://en.wikipedia.org/wiki/Moho_(software)); [Moho features](https://moho.lostmarble.com/pages/features)
- After Effects' Puppet Pin tool is free and built in but "pretty clunky for character work," producing less-than-clean deformations around joints — [School of Motion: Character Rigging Tools for After Effects](https://schoolofmotion.com/blog/character-rigging-tools-after-effects)

### Inferences
- VTuber-style clothing rigging in Live2D = cut the garment art into layers (skirt front/back panels, ribbons, sleeve ends), give each a deformer/parameter (e.g., "Skirt sway X"), then bind that parameter as the output of a physics group whose input is head/body angle X/Z. Multi-segment pendulums allow ribbons and long hair to whip. This is a well-known community pattern consistent with the official "swinging skirt" example, but no single official VTuber clothing guide was retrieved.
- The core limitation of all 2D rigs: deformations are bounded by the drawn art and its layer cuts; a skirt can sway and flare within a drawn range but cannot convincingly show back-panel reveals, new folds from a sit-down, or large 3/4-to-profile rotation without extra drawn parts (art swaps), which is why Live2D models favor front-facing ±30° angles.

### Gaps
- Cubism 5.x specific physics changes (e.g., new physics features or limits on group counts) could not be verified because docs.live2d.com was blocked.
- No source on Spine/Moho usage in actual Japanese anime TV production (they are mostly used in games/web).

---

## 3. AI in-betweening and interpolation for clothing

### Takeaway
Research has moved from flow/correspondence-based anime interpolation (AnimeInterp 2021, EISAI, AnimeInbet 2023) to generative video-diffusion in-betweening (ToonCrafter 2024, AniDoc CVPR 2025, ToonComposer 2025/ICLR 2026). Flow methods break on cartoons' large non-linear motion and dis-occlusion — exactly the regime of flapping skirts, capes and coats; diffusion methods can hallucinate plausible fabric motion but lose fine line detail in compressed latents, can leak live-action content, and still need dense keys for big motion. None of the retrieved papers evaluates cloth specifically.

### Cited Findings
- AnimeInterp ("Deep Animation Video Interpolation in the Wild", CVPR 2021): Segment-Guided Matching (global matching of flat color pieces to fix the "lack of texture" problem) + Recurrent Flow Refinement (transformer-like recurrent refinement for "non-linear and extremely large motion"); released ATD-12K, 12,000 frame triplets from 30 animated movies (>25 hours) — [CVPR 2021 paper](https://openaccess.thecvf.com/content/CVPR2021/papers/Siyao_Deep_Animation_Video_Interpolation_in_the_Wild_CVPR_2021_paper.pdf); [arXiv 2104.02495](https://arxiv.org/abs/2104.02495)
- EISAI improves perceptual quality of 2D animation interpolation, reducing line destruction and ghosting via forward warping and line-distance constraints — [search excerpt citing EISAI, via AnimeInterp-related results](https://www.researchgate.net/publication/350674122_Deep_Animation_Video_Interpolation_in_the_Wild) (primary EISAI paper not fetched)
- AnimeInbet ("Deep Geometrized Cartoon Line Inbetweening", ICCV 2023): treats line inbetweening as graph fusion with vertex repositioning and predicts a visibility mask to erase vertices/edges occluded in the in-between frame; difficulty rises with frame gap (larger motion, more occlusion) — [ICCV 2023 paper](https://openaccess.thecvf.com/content/ICCV2023/papers/Siyao_Deep_Geometrized_Cartoon_Line_Inbetweening_ICCV_2023_paper.pdf); [arXiv 2309.16643](https://arxiv.org/abs/2309.16643)
- AnimeRun (NeurIPS 2022 Datasets & Benchmarks): 2D-styled correspondence dataset rendered from three open Blender movies (Agent 327, Caminandes 3, Sprite Fright) with pixel-level optical flow and region-level segment matching, colored frames plus line-art; motivated by prior cartoon datasets having "simple frame composition and monotonic movements"; CC-BY-NC 4.0 — [arXiv 2211.05709](https://arxiv.org/abs/2211.05709); [project page](https://lisiyao21.github.io/projects/AnimeRun)
- LinkTo-Anime (2025): a 2D-animation optical-flow dataset rendered from 3D models — [arXiv 2506.02733](https://arxiv.org/pdf/2506.02733)
- Thin-plate-spline-based line inbetweening (AAAI 2025) — [arXiv 2408.09131](https://arxiv.org/html/2408.09131)
- ToonCrafter (SIGGRAPH Asia 2024, ACM TOG 43(6) Art. 245): generative interpolation of two cartoon frames using a pre-trained image-to-video diffusion prior (DynamiCrafter lineage); "toon rectification learning" adapts live-action priors to cartoons; dual-reference 3D decoder recovers detail lost in compressed latents; outputs up to 512x320, up to 16 frames; explicitly motivated by correspondence methods failing on "exaggerated non-linear and large motions with occlusion" — [arXiv 2405.17933](https://arxiv.org/abs/2405.17933); [GitHub](https://github.com/Doubiiu/ToonCrafter)
- ToonCrafter's own paper lists the failure sources: highly compressed latent spaces lose detail (worse in cartoons due to high-contrast regions, fine outlines, no motion blur) and the live-action domain gap can synthesize non-cartoon content or wrong motion — [ToonCrafter arXiv HTML](https://arxiv.org/html/2405.17933v1)
- ToonComposer (2025; listed for ICLR 2026) reports that ToonCrafter-style inbetweening struggles with large motions from sparse sketches, often needs dense keyframes, and produces distorted faces in hard sparse-sketch cases; ToonComposer proposes "generative post-keyframing" with reference-preserving identity — [arXiv 2508.10881](https://arxiv.org/pdf/2508.10881); [Paper note](https://en.papernotes.org/ICLR2026/video_generation/tooncomposer_streamlining_cartoon_production_with_generative_post-keyframing/)
- AniDoc (CVPR 2025): video-diffusion line-art colorization that follows a character design reference, robust to pose/scale differences, and supports sparse sketch input so it does interpolation + colorization together; frames production as design → key animation → in-betweening → coloring — [CVPR 2025 paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Meng_AniDoc_Animation_Creation_Made_Easier_CVPR_2025_paper.pdf); [project](https://yihao-meng.github.io/AniDoc_demo/)
- Other 2025–2026 inbetweening work surfaced: "Generative Inbetweening through Frame-wise Conditions-Driven Video Generation" ([arXiv 2412.11755](https://arxiv.org/pdf/2412.11755)), EF-VI end-frame injection ([arXiv 2505.21205](https://arxiv.org/pdf/2505.21205)), "Workflow-friendly Anime In-betweening Using Video Generation Models" ([DOI 10.1145/3799818.3812070](https://doi.org/10.1145/3799818.3812070)), AniDepth (anime line-art inbetweening with out-of-plane motion and occlusion handling, per search excerpt), SketchKeyAnime (sparse key-sketch synthesis, [arXiv 2606.19958](https://arxiv.org/pdf/2606.19958)); curated list: [Awesome-2D-Animation](https://github.com/MarkMoHR/Awesome-2D-Animation)
- Sakuga-42M (2024): first large-scale cartoon dataset, 42M keyframes from >150,000 public cartoon videos, with video-text captions, anime tags and taxonomies; fine-tuning Video CLIP, Video Mamba and SVD on it improved cartoon comprehension/generation — [project page](https://zhenglinpan.github.io/sakuga_dataset_webpage/); [arXiv abstract via ADS](https://ui.adsabs.harvard.edu/abs/2024arXiv240507425P/abstract)
- MagicAnime (2025): hierarchically annotated multimodal cartoon animation dataset with benchmarks — [arXiv 2507.20368](https://arxiv.org/html/2507.20368v1)

### Inferences
- Cloth is a worst case for correspondence-based interpolators: skirt hems and cape edges undergo large non-linear arcs, fold lines appear/disappear (topology changes), and fabric frequently occludes/disoccludes legs and torso. The visibility-mask approach (AnimeInbet) and generative priors (ToonCrafter/AniDoc) are the two answers to this, but neither guarantees "fold logic" consistent with the key animator's intent.
- Practical 2026 use: AI in-betweening is most credible for small-to-medium secondary motion (ribbon flutter, hem sway on 2s between close keys) with human cleanup; big cape swirls/skirt flips still need more keys or hand in-betweens. This inference matches the "dense keyframes required" finding.
- Licensing/data provenance (e.g., Sakuga-42M built from public videos) is a production adoption concern for studios.

### Gaps
- No paper found that benchmarks interpolation specifically on garments/fabric in anime; claims about cloth performance are inferred.
- No verified evidence of named Japanese studios shipping TV anime using ToonCrafter/AniDoc for in-betweens; industry adoption status as of 2026 not established here.
- EISAI primary paper and AniDepth primary paper not fetched.

---

## 4. 3D cel-shaded cloth: garment creation, simulation, toon shaders, and making 3D cloth look hand-drawn

### Takeaway
The typical 3D anime garment pipeline: author garments as sewing patterns in Marvelous Designer/CLO (now with a Toon Shader preview), then either (a) simulate offline (Houdini Vellum, Blender cloth, Maya nCloth) and bake to caches/shape keys, or (b) for real-time/games, use bone/spring chains (VRM SpringBone, Magica Cloth BoneCloth, Spine-like dynamics) or mesh cloth (Magica MeshCloth, Unreal Chaos Cloth Dataflow editor, production-ready in UE 5.8). Render with toon shaders (Pencil+ 4, UTS3, lilToon, Blender LineArt/Grease Pencil). Stylization comes from simplified geometry, edited normals, vertex-color shadow control, stepped timing (2s/3s, ~15 fps), and hand-keyed or model-swapped cloth instead of raw simulation.

### Cited Findings
**Garment authoring**
- Marvelous Designer builds 3D garments like real clothes by stitching virtual 2D pattern pieces; widely used by games and animation studios — [CG Channel: Creating Clothing for Characters in MD](https://www.cgchannel.com/2024/05/creating-clothing-for-characters-in-marvelous-designer/)
- Marvelous Designer added a Toon Shader so garments for cartoon-style characters can be previewed more accurately inside MD; Marvelous Designer 2026.0 released April 2026 — [CG Channel: MD 2026.0](https://www.cgchannel.com/2026/04/clo-virtual-fashion-releases-marvelous-designer-2026-0/)
- Japanese community material documents a Marvelous Designer → UE5 Chaos Cloth → UEFN workflow — [Docswell: chaoscloth workflow from MD to UE5 to UEFN](https://www.docswell.com/s/moyuki/54VVWQ-2024-09-14-231311)

**Offline simulation**
- Houdini Vellum: fast cloth/hair/soft-body solver; studios favor it for performance/quality balance, adaptive substepping, optional GPU acceleration; a CGI anime short "Henshin!" (Kay John Yim) used Vellum cloth with per-material properties and painted sim masks — [SideFX Vellum](https://www.sidefx.com/products/houdini/vfx/vellum/); [Fox Render Farm: Making of "Henshin!"](https://www.foxrenderfarm.com/news/the-making-of-henshin-a-cgi-anime-fantasy-created-by-kay-john-yim/); [80.lv Vellum insights](https://80.lv/articles/exploring-vellum-cloth-simulation-in-houdini)
- Blender cloth: bake via Cache panel; Quality Steps trade accuracy vs time; free add-on "Cloth to Shape Keys" bakes sims to shape keys (useful for editing/retiming stylized motion); wrinkle maps driven by tension maps on proxy meshes — [RenderGuide Blender cloth tutorial](https://renderguide.com/blender-cloth-simulation-tutorial/); [Lesterbanks: animating wrinkles with cloth in Blender](https://lesterbanks.com/2020/09/how-to-animate-wrinkles-with-cloth-in-blender/)

**Real-time / engine cloth**
- Unreal Chaos Cloth: Panel Cloth node editor introduced in 5.3 (non-destructive, in-engine authoring) with experimental ML cloth data generation (simulate inside UE to geometry cache → feed ML Deformer); 5.6 beta added unified Dataflow editor, Outfit Asset with resizing/refitting, cloth-to-cloth constraints, simulation morph targets; in 5.8 the Chaos Dataflow Cloth Editor is Production-Ready and the default — [Epic: Chaos Cloth Updates 5.8](https://dev.epicgames.com/community/learning/tutorials/Wb2V/unreal-engine-chaos-cloth-updates-5-8); [Epic forum: Chaos Cloth Updates 5.6](https://forums.unrealengine.com/t/tutorial-chaos-cloth-updates-5-6/2555686); [Epic forum: ML Cloth Generation](https://forums.unrealengine.com/t/tutorial-ml-cloth-generation/1284843); [Panel Cloth Editor Overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/panel-cloth-editor-overview)
- Magica Cloth 2 (Unity, DOTS-based): BoneCloth simulates Transforms (bone chains), MeshCloth simulates mesh vertices and is recommended for complex motion like skirts with collision; skirt setup example uses UnityChan KAGURA (waist verts fixed, rest free); runs on all platforms except WebGL and visionOS; works in Built-in/URP/HDRP without special shaders — [MagicaSoft: About MagicaCloth2](https://magicasoft.jp/en/mc2_about/); [MeshCloth guide](https://magicasoft.jp/en/mc2_meshclothstartguide/); [Magica Cloth product page](https://magicasoft.jp/en/magica-cloth-2/)
- VRM SpringBone: simple spring physics on hair/dress bones with body colliders to prevent skin penetration; skirt flipping when sitting is a very common VRM problem arising from mesh shape, skin weights and SpringBone parameters; one fix tool sets skirt SpringBone Gravity Power (default 0) to ~0.3 — [AOUSD forum: springbone standards](https://forum.aousd.org/t/standards-for-springbones-and-colliders-for-hair-and-cloth-physics/799); [pixiv/three-vrm discussion #1464](https://github.com/pixiv/three-vrm/discussions/1464); [VrmSkirtFixTool note](https://note.com/dokokano_usagi/n/n7d49a37bc875?hl=en)

**Toon shaders / lines**
- PSOFT Pencil+ 4: NPR line/material toolset widely used in Japanese animation (Toei Animation, Shirogumi, Marza Animation Planet; used on Khara's EVANGELION:3.0 (-46h)); Pencil+ 4 Line for Blender add-on (2023) interoperates with 3ds Max, Maya and Unity editions; line width, colour, brush angle, opacity, blend mode via UI or node editor; add-on free but designed to use commercial Pencil+ Render — [CG Channel: Pencil+ 4 Line for Blender](https://www.cgchannel.com/2023/10/psoft-releases-pencil-4-line-for-blender/); [GitHub psofthouse](https://github.com/psofthouse/Pencil-4-Line-for-Blender)
- Unity Toon Shader (UTS3): successor to UTS2, designed for cel-shaded 3DCG animation, supports Built-in/URP/HDRP; outlines via inverted-hull (front-culled enlarged normals) — [GitHub com.unity.toonshader](https://github.com/Unity-Technologies/com.unity.toonshader); [UTS manual](https://docs.unity3d.com/Packages/com.unity.toonshader@0.7/manual/GettingStarted.html)
- lilToon: popular Unity toon shader package (VRChat avatar ecosystem; UTS2→lilToon conversion guides exist) — [lilToon](https://lilxyzw.github.io/lilToon/); [note.com UTS2→lilToon](https://note.com/chiffon_pudding/n/n4ed8e23a07ca?hl=en)

**Stylization techniques**
- Arc System Works (Guilty Gear Xrd): animated on a ~15 fps basis; vertex normals edited to simplify light response and remove small polygonal shadows; a vertex-color channel offsets the shading threshold so artists can force areas to darken; "stretchy bones"; multiple 3D models swapped per frame for hair/cloth shapes (e.g., Millia's hair attacks) — [BlenderNation: Motomura behind the scenes](https://www.blendernation.com/2015/07/26/junya-c-motomura-behind-the-scenes-of-guilty-gear-xrd/); [GDC Vault talk page](https://www.gdcvault.com/play/1022031/GuiltyGearXrd-s-Art-Style-The); [Motomura slides PDF (search excerpt; fetch blocked)](https://www.ggxrd.com/Motomura_Junya_GuiltyGearXrd.pdf)
- Spider-Verse (Sony Imageworks): cloth on 2s required simulating with a hidden "ghost in-between frame that the audience never sees and animation never actually animated, but it allows the simulation to actually be continuous"; custom software targeted sims to the stepped animation and removed ghost frames afterward — [Cartoon Brew: Imageworks' approach to Into the Spider-Verse](https://www.cartoonbrew.com/feature-film/if-its-not-broke-break-it-sony-imageworks-renegade-approach-to-spider-man-into-the-spider-verse-167321.html)

### Inferences
- Raw physical simulation produces too many small wrinkles and continuous (on-1s) motion, both of which read as "CG" in anime; the anime look requires either reducing wrinkle frequency (coarser sim mesh, stiffer bending, fewer pattern seams) or bypassing sim with hand-keyed/stepped cloth. Spider-Verse's ghost-frame trick shows the engineering needed to keep sims stable under stepped timing.
- Cloth-to-shape-keys (Blender) or caches → keyframe reduction is the practical way to retime sims onto 2s/3s and hand-correct silhouettes.
- For real-time anime games, bone-chain cloth (SpringBone/BoneCloth) dominates for skirts/ribbons because it is cheap and art-directable, with mesh cloth reserved for hero garments.

### Gaps
- Maya nCloth anime-specific workflows were not researched.
- Blender Grease Pencil / LineArt documentation was not retrieved (only mentioned).
- No retrieved source on "hand-placed shadow shapes" for clothing specifically (e.g., Pencil+ shadow masks, normal transfer from proxy shapes) beyond the Guilty Gear vertex-normal/vertex-color technique.

---

## 5. Case studies (Guilty Gear, Genshin, Orange, Spider-Verse, others)

### Takeaway
The best-documented case is Arc System Works' Guilty Gear Xrd (GDC 2015): cloth and hair are posed frame-by-frame at ~15 fps, with model swaps and stretchy bones rather than physics. Orange (Land of the Lustrous, Trigun Stampede) mixes 3DCG with hand-drawn close-ups and deliberately recreates 2D quirks like variable frame rates. Spider-Verse engineered sims that work under stepped animation. Public, technical detail on Genshin Impact cloth, Blue Lock, Ufotable or Polygon Pictures cloth was not found.

### Cited Findings
- GDC 2015 talk "GuiltyGearXrd's Art Style: The X Factor Between 2D and 3D" by technical artist Junya Christopher Motomura (Arc System Works); goal: recreate 2D animation using 3DCG with custom cel shaders and models made specifically for them; frames are "cut" (limited animation) to reinforce the illusion of 2D — [GDC Vault](https://www.gdcvault.com/play/1022031/GuiltyGearXrd-s-Art-Style-The); [Game Developer article (fetch blocked; excerpt)](https://www.gamedeveloper.com/art/see-i-guilty-gear-xrd-i-s-striking-2d-3d-art-deconstructed-at-gdc-2015)
- Guilty Gear's developers pose character animations frame-by-frame including clothing and hair so the game "always looks the way they intend" — [EventHubs on Guilty Gear Strive](https://www.eventhubs.com/news/2021/jul/07/guilty-gear-costumes/) (secondary source)
- Arc System Works Academy has published "Guilty Gear Toon Line Control Techniques" (English version) on Docswell — [Docswell: GG Toonline (ENG)](https://www.docswell.com/s/ASW_Academy/5LVY67-GG-Toonline-Eng) (not fetched)
- Orange / Land of the Lustrous (2017): chose 3D partly because translucent gems are hard to hand-draw; many close-ups and some facial parts are hand-drawn 2D; Orange won CGWORLD's award for best CG in animation; deliberately engineered 3D to replicate 2D quirks including variable framerates and faces that warp to feel camera-forward — [ANN: Art and Animation of Land of the Lustrous](https://www.animenewsnetwork.com/interest/2018-02-26/a-look-into-the-art-and-animation-of-land-of-the-lustrous/.128275); [Wikipedia: Orange (animation studio)](https://en.wikipedia.org/wiki/Orange_(animation_studio))
- Orange / Trigun Stampede (2023): first full-length Orange series with human characters in 3DCG; ~5 years of development, CG modeling started 3–4 years before air, animation ~1.5 years before; animation producer referenced old Disney cel films like Tarzan for expressive CG; concept designs had to be reinterpreted for animation — [SlashFilm](https://www.slashfilm.com/1184803/how-3d-anime-specialists-studio-orange-created-the-world-of-trigun-stampede/); [ComicBook.com interview](https://comicbook.com/anime/news/trigun-stampede-team-interview-cg-animation-talk/)
- Trigun Stargaze (2026) director Masako Sato interview on "breaking new ground with 3DCG" — [MANTANWEB (Feb 2026)](https://en.mantan-web.jp/e_article/20260203dog00m200106000a.html) (not fetched)
- Spider-Verse: ghost in-between frames for continuous cloth sims under 2s timing (see §4) — [Cartoon Brew](https://www.cartoonbrew.com/feature-film/if-its-not-broke-break-it-sony-imageworks-renegade-approach-to-spider-man-into-the-spider-verse-167321.html); [SideFX: Spider-Verse](https://www.sidefx.com/community/spider-man-into-the-spider-verse/)
- Genshin Impact: retrieved sources only cover fan shader recreations (anisotropic hair highlights, Unity URP breakdowns) and an unverified third-party description of Unity Cloth proxies; no first-party miHoYo/HoYoverse talk on its cloth system was found — [Adrian Mendez shader breakdown](https://adrianmendez.artstation.com/projects/wJZ4Gg); [PrimoToon (GitHub)](https://github.com/festivities/PrimoToon)

### Inferences
- Common thread across successful "3D that reads as 2D" productions: treat cloth as an animated *shape* (keyed, swapped, stepped) rather than a simulated surface, and hand-draw where 3D fails (close-ups, faces, hero cloth moments).
- Orange's Trigun Stampede timeline (years of modeling) indicates very high up-front cost for 3DCG anime asset development, amortized over a season.

### Gaps
- No primary technical source found for Genshin Impact cloth/hair physics (miHoYo has given Unite/GDC talks on rendering, but none retrieved here covering cloth).
- Blue Lock (Eight Bit), Ufotable 3D, Polygon Pictures cloth approaches: not researched due to tool-call limits.
- CEDEC talks by Arc System Works on Strive cloth/hair: not found.
- CGWORLD.jp articles not retrieved (likely the richest Japanese source for Orange/Polygon Pictures cloth workflow).

---

## 6. MMD / VRoid Studio clothing for low-budget anime-like animation

### Takeaway
VRoid Studio (pixiv) now supports a dress-up feature with the XWear format (v2.0.0), auto/manual fitting, a Closet for reusable costumes/accessories (v2.7.0–2.8.0, Dec 2025), a texture Sticker tool (v2.11.0), and a separate "VRoid Clothing Maker" app on Steam; VRoid clothing moves via VRM SpringBone. MMD (PMX) clothing uses Bullet rigid bodies + joints edited in PMXEditor; both are cheap but prone to skirt flipping/clipping.

### Cited Findings
- VRoid Studio v2.0.0 officially released the dress-up feature and XWear format: dress 3D characters including VRChat-style avatars/outfits, VRoid models and VRM models; fitting feature with auto-fitting and manual adjustment for varying body types — [VRoid notice v2.0.0](https://vroid.com/en/studio/notice/3rgHw1j8oCVkdWM0LKHBH6); [VRoid news](https://vroid.com/en/news/26gn98sTuPJQ53LxDQRyFg)
- v2.7.0 (Dec 4, 2025) added the Closet for frequently used base models, costumes and accessories; v2.8.0 (Dec 23, 2025) brought the Closet into the VRoid editor and new presets; accessories can be handled as custom items — [VRoid help v2.7.0](https://vroid.pixiv.help/hc/en-us/articles/52924741495065--v2-7-0-Added-the-Closet-feature-Dec-4th-2025); [VRoid help v2.8.0](https://vroid.pixiv.help/hc/en-us/articles/53437792295065--v2-8-0-Added-new-presets-Closet-feature-added-to-the-VRoid-Editor-Dec-23rd-2025)
- v2.11.0 added a Sticker tool to the texture editor (place any image onto textures) — [Steam: VRoid Studio news](https://steamcommunity.com/app/1486350/allnews/)
- VRoid Clothing Maker exists as a separate Steam app — [Steam store](https://store.steampowered.com/app/2330730/VRoid_Clothing_Maker/)
- VRoid-made VRM skirt SpringBone parameters can be retuned with third-party tools (e.g., nocchi's VRM spring bone adjustment tool); skirt flip-up when sitting is common — [F.Issiki on X](https://x.com/FIssiki/status/1808165770796622106); [VrmSkirtFixTool](https://note.com/dokokano_usagi/n/n7d49a37bc875?hl=en)
- MMD physics uses the Bullet engine; PMXEditor creates rigid bodies (simple collision shapes) and joints connecting them; skirt clipping happens if legs lack rigid bodies; tip: make skirt rigid bodies longer/wider so the shape holds when stretched — [Wikipedia: Bullet](https://en.wikipedia.org/wiki/Bullet_(software)); [PMX Editor Tutorials: skirt physics clipping](https://pmxeditortutorials.tumblr.com/post/171424498415/tip-for-skirt-physics-about-clipping); [LearnMMD: making clothes follow motion](https://learnmmd.com/http:/learnmmd.com/pmde-qa-making-clothes-follow-model-motions/)

### Inferences
- VRoid/MMD give near-zero-cost moving clothing for indie anime-like shorts and VTubers, but the look is generic (SpringBone/rigid-body jiggle, on-1s motion) and needs toon shader tuning, stepped rendering and manual fixes to approach broadcast anime quality.

### Gaps
- No source on VRoid Studio's built-in clothing template limits (e.g., which garment types are supported natively vs. needing external DCC).

---

## 7. AI/ML (neural) cloth simulation and applicability to stylized anime garments

### Takeaway
Neural cloth simulation progressed from SNUG (self-supervised, tight garments only) to HOOD (hierarchical GNN, free-flowing garments, generalizes to unseen garments, real-time) to ContourCraft (SIGGRAPH 2024, resolves multi-layer intersections), plus 2025–2026 work (D-Garment, Neural Garment Dynamic Super-Resolution, UNIC, PhySkin, SimAvatar). All target realistic physics; none found targets anime stylization, so applicability is mainly as a fast physical base that would still need stylization.

### Cited Findings
- SNUG: self-supervised physics-based training with a recurrent network predicting garment deformation sequences; limited to tight-fitting garments and cannot generalize to novel garments — [HOOD paper (related-work discussion)](https://arxiv.org/pdf/2212.07242)
- HOOD (CVPR 2023): GNN + hierarchical graph + multi-level message passing, unsupervised training; real-time prediction for arbitrary garment types and body shapes; handles tight and free-flowing clothes; generalizes to unseen garments; supports changes in material parameters and topology at test time — [arXiv 2212.07242](https://arxiv.org/pdf/2212.07242)
- ContourCraft (SIGGRAPH 2024): adds an intersection-contour loss + collision-avoiding repulsion to a GNN cloth simulator; addresses HOOD's inability to model cloth–cloth interactions/self-collisions, which made multi-layer outfits unrealistic — [ACM DL](https://dl.acm.org/doi/fullHtml/10.1145/3641519.3657408)
- D-Garment (2025): physically grounded latent diffusion for dynamic garment deformations — [arXiv 2504.03468](https://arxiv.org/pdf/2504.03468)
- Neural Garment Dynamic Super-Resolution (Dec 2024) — [arXiv 2412.06285](https://arxiv.org/pdf/2412.06285)
- UNIC (2026): neural deformation field to animate garment meshes in real time from motion sequences — [arXiv 2603.25580](https://arxiv.org/html/2603.25580v1)
- PhySkin (2026): physics-based, bone-driven neural garment simulation — [arXiv 2603.27013](https://arxiv.org/pdf/2603.27013)
- SimAvatar (CVPR 2025): simulation-ready avatars with layered hair and clothing driven by physics simulators — [CVPR 2025 paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Li_SimAvatar_Simulation-Ready_Avatars_with_Layered_Hair_and_Clothing_CVPR_2025_paper.pdf)
- Unreal's ML Deformer + Chaos Cloth ML data generation is the closest production-tool analogue (train on in-engine sim caches) — [Epic forum: ML Cloth Generation](https://forums.unrealengine.com/t/tutorial-ml-cloth-generation/1284843)

### Inferences
- Bone-driven approaches (PhySkin) are interesting for anime games because they map onto existing bone-chain skirt rigs.
- Neural sims inherit realistic wrinkle statistics; for anime they would need post-processing (wrinkle suppression, stepped sampling) or training on stylized target data, which does not appear to exist publicly.

### Gaps
- No paper found on neural cloth trained on or evaluated for stylized/anime garments.

---

## 8. Garment-from-2D pipelines (model sheet → 3D garment / sewing pattern)

### Takeaway
A lineage of sewing-pattern generators now exists: Sewformer (image→pattern), DressCode (SIGGRAPH 2024, text→pattern via SewingGPT), GarmentCode (programmatic JSON pattern representation), ChatGarment (CVPR 2025, VLM outputs GarmentCode JSON from images/sketches/text, with editing dialogue), AIpparel (multimodal foundation model, vector-quantized patterns), GarmentDiffusion (2025, multimodal diffusion transformer, cm-precise vector patterns), Design2GarmentCode (CVPR 2025) and GarmentGPT (ICLR 2026). ChatGarment's ability to estimate from sketches is the most relevant to anime model sheets, but none were evaluated on stylized anime costumes.

### Cited Findings
- ChatGarment (CVPR 2025): VLM takes text or image (incl. in-the-wild images and sketches), outputs JSON decoded by GarmentCode into a 2D sewing pattern then draped on a body; supports interactive editing in dialogue — [CVPR 2025 paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Bian_ChatGarment_Garment_Estimation_Generation_and_Editing_via_Large_Language_Models_CVPR_2025_paper.pdf); [project](https://chatgarment.github.io/); [GitHub](https://github.com/biansy000/ChatGarment)
- GarmentCode: programmatic framework encoding garments as JSON with garment types, styles and numeric attributes — [ChatGarment arXiv](https://arxiv.org/html/2412.17811v1)
- AIpparel (ETH IGL): fine-tuned large multimodal model on a custom sewing pattern dataset; generates complex patterns from text and images; emphasizes text-based generation; outputs vector-quantized patterns rather than JSON — [AIpparel paper](https://igl.ethz.ch/projects/aipparel/aipparel_paper.pdf); [ChatGarment comparison](https://arxiv.org/html/2412.17811v1)
- DressCode (ACM TOG / SIGGRAPH 2024): GPT-like autoregressive SewingGPT generating vector-quantized sewing patterns conditioned on text via cross-attention; reuses Sewformer pattern code — [ACM DL](https://dl.acm.org/doi/10.1145/3658147); [GitHub](https://github.com/IHe-KaiI/DressCode)
- GarmentDiffusion (2025): multimodal (text, image, incomplete pattern) diffusion transformer producing centimeter-precise vectorized 3D sewing patterns with ~10x shorter sequences than DressCode's SewingGPT — [arXiv 2504.21476](https://arxiv.org/html/2504.21476v4)
- Multimodal latent diffusion for complex sewing patterns (Dec 2024) — [arXiv 2412.14453](https://arxiv.org/html/2412.14453v1)
- Design2GarmentCode (CVPR 2025): program synthesis from design concepts to garments — [CVPR 2025 paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Zhou_Design2GarmentCode_Turning_Design_Concepts_to_Tangible_Garments_Through_Program_Synthesis_CVPR_2025_paper.pdf)
- GarmentGPT (ICLR 2026): compositional garment pattern generation via discrete latent tokenization — [Paper note](https://en.papernotes.org/ICLR2026/image_generation/garmentgpt_compositional_garment_pattern_generation_via_discrete_latent_tokeniza/)

### Inferences
- A plausible 2026 pipeline: anime model sheet (front/back/side) → ChatGarment/GarmentDiffusion draft pattern → import to Marvelous Designer/CLO for correction → simplify for toon look → export to engine/DCC. Anime costumes have non-physical elements (impossible ribbons, floating capes, exaggerated volume) that GarmentCode-style parametric templates likely cannot express, so human pattern work remains.

### Gaps
- Sewformer primary paper not fetched; no benchmark of these systems on anime/stylized costume designs; no evidence of studio adoption.

---

## 9. Cost/time and quality comparison across approaches

### Takeaway
Hard, comparable per-approach cost data for cloth animation does not exist publicly. Available anchors: TV anime episodes reportedly cost roughly US$160k–320k (aggregator sources, low reliability), with ~300 cuts per episode and key-animator per-cut prices as low as ¥3,800 in a criticized Netflix offer; 3DCG anime requires multi-year asset development (Trigun Stampede: CG modeling 3–4 years before airing). The table below synthesizes qualitative trade-offs from the cited findings; numbers are only those sourced.

### Cited Findings
- A Netflix-suggested unit price of ¥3,800 per cut (~US$34 at the time) was criticized by an animator quoted by Cartoon Brew; average TV episode ~300 cuts (via aggregator) — [Medium: Anime and the cut unit price](https://medium.com/@emiliahoarfrost/anime-and-the-cut-unit-price-on-the-interplay-between-producers-animators-and-anime-workers-in-a-ac38c02932c1) (secondary)
- Episode cost US$160k–320k, >US$500k for high-profile streaming productions; 300–500 cuts per half-hour episode; animation labor ~60% of budget — [Magic Motion Studio pricing guide](https://magicmotionstudio.com/how-much-does-anime-animation-cost/); [Dark Skies](https://darkskiesfilm.com/how-much-does-a-single-anime-episode-cost/) (low-reliability commercial/aggregator sources; one aggregator's claim of "300 million yen per episode" at top studios is unverified and likely erroneous)
- Trigun Stampede: ~5 years development; CG modeling began 3–4 years before release; animation ~1.5 years — [ComicBook.com](https://comicbook.com/anime/news/trigun-stampede-team-interview-cg-animation-talk/)
- Spider-Verse needed custom software to keep sims stable on 2s — [Cartoon Brew](https://www.cartoonbrew.com/feature-film/if-its-not-broke-break-it-sony-imageworks-renegade-approach-to-spider-man-into-the-spider-verse-167321.html)
- Tool pricing anchors: Pencil+ 4 Line for Blender add-on free, needs commercial Pencil+ Render ([CG Channel](https://www.cgchannel.com/2023/10/psoft-releases-pencil-4-line-for-blender/)); AE Puppet free but clunky ([School of Motion](https://schoolofmotion.com/blog/character-rigging-tools-after-effects)); ToonCrafter open-source on GitHub, 512x320 / 16 frames max ([GitHub](https://github.com/Doubiiu/ToonCrafter))

### Comparison table (synthesis; qualitative ratings are inferences from the findings above)

| Approach | Typical tools | Up-front cost | Per-shot cost | Cloth quality / anime authenticity | Main failure modes | Best for |
|---|---|---|---|---|---|---|
| 2D hand-drawn | Paper, CSP, TVPaint, Harmony, OpenToonz | Low | High (every fold drawn; per-cut labor) | Highest; full control of fold logic, nabiki, smears | Off-model folds, cost pressure → simplified garments | TV/film anime hero moments |
| 2D cut-out + deformers | Toon Boom Harmony (Envelope/Curve) | Medium (rig build) | Low–medium | Good for capes/ribbons; limited new folds/turns | Rubbery look, limited angles | Kids' TV, web series |
| 2D rig + physics | Live2D Cubism, Spine 4.2, Moho | Medium (art cut + rig) | Very low (runtime physics) | Good sway/bounce within drawn range | No true rotation, no new folds; floaty pendulums | VTubers, gacha/visual novels, games |
| AI in-betweening | ToonCrafter, AniDoc, ToonComposer | Low (open models) + GPU | Low per frame + cleanup | Variable; plausible on small motion, breaks on large cape/skirt arcs | Detail loss, line wobble, hallucinated content, needs dense keys, data/IP concerns | Assisting in-betweens, previz |
| 3D sim + toon shading | MD/CLO → Vellum / Blender / nCloth; Pencil+ 4 | High (assets, pipeline) | Medium (sim + fixes) | Can look "CG" (too many wrinkles, on-1s) unless stylized | Wrinkle noise, sim instability under stepped timing | Feature films, 3DCG anime with big crowds/action |
| 3D hand-keyed/swapped cloth | Maya/Blender bones, model swaps, stretchy bones, edited normals | High | Medium–high | Highest 3D authenticity (Guilty Gear) | Labor-intensive per frame | Fighting games, signature 3DCG anime |
| Real-time bone/mesh cloth | VRM SpringBone, Magica Cloth 2, UE Chaos Cloth | Medium | ~Zero at runtime | Acceptable; generic jiggle | Skirt flipping, clipping, floaty motion | Games, VTubers, MMD/VRoid content |
| Neural cloth sim | HOOD, ContourCraft, UE ML Deformer | High (R&D/training) | Very low inference | Realistic, not stylized | No anime training data; multi-layer issues (pre-ContourCraft) | Research / real-time realism |

### Inferences
- The decision axis is less "2D vs 3D" than "who authors the fold shapes": a human (hand-drawn, hand-keyed 3D, model swaps) gives anime authenticity at labor cost; a solver (physics, neural, pendulum, diffusion) gives low marginal cost but generic or realistic motion that must be stylized.
- Hybrid approaches (3D base + 2D hand-drawn close-ups as with Orange; sims retimed to 2s as with Spider-Verse; AI in-betweens with human keys) are the dominant 2026 pattern based on the case studies.

### Gaps
- No reliable per-approach cost/time numbers (e.g., hours per second of cloth animation) were found for any method; all ratings in the table are qualitative inferences.
- Japanese primary sources on cut unit prices (e.g., JAniCA surveys) were not retrieved; figures above come from secondary/aggregator sources.
