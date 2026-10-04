# Creating separate, rig-ready anime clothing for an EXISTING 3D anime body from manga references, with no training (solo creator, 24GB+ GPU, personal use, target UE5; as of 2026-10-04)

Scope note: the coordinator changed the scope mid-task. The user already has an unclothed anime body that is rigged or can be rigged. These notes focus on making the clothing. Full-character image-to-3D is condensed into Route 3.

Research method note: ~45 tool calls. The egress proxy BLOCKED these domains, so their content is known only from search-result snippets:
- arxiv.org, huggingface.co
- support.marvelousdesigner.com, cgchannel.com
- help.meshy.ai, www.meshy.ai, developers.tripo3d.ai
- chatgarment.github.io, joseconseco.github.io (Garment Tool docs)
- en.papernotes.org, triposr.org

GitHub repos (StdGEN, TRELLIS.2, VRM4U, GarmentCode, ChatGarment, robust-weight-transfer) were fetched directly. Many 2026 "comparison" pages are vendor-written (Meshy, Marvelous Designer's own "best software" guides) or SEO aggregators; they are flagged where cited.

## Reference prep: clean front/side/back outfit turnarounds from manga panels, including dressing a render of the user's own base body

### Takeaway
The most useful no-training trick is to render the user's own base body in A-pose (front, side and back) in Blender, then use a zero-shot image editor to "dress" each render in the manga outfit. Use Qwen-Image-Edit-2511 locally (it accepts up to 3 reference images: body render + manga panel + color sheet), or Nano Banana Pro / GPT-image in the cloud. The result is a turnaround that already matches the body's proportions. It works as a pattern-tracing guide in Marvelous Designer, a modeling backdrop in Blender, a multi-view input for image-to-3D, and a projection source for flat textures. A community "Character Turnaround Sheet" LoRA for Qwen-Image-Edit-2511/2509 provides the multi-angle layout. It is already trained, so you only download it; the user trains nothing.

### Cited Findings
- Qwen-Image-Edit-2511 (ComfyUI) improves character consistency, reduces image drift across edits, and accepts up to three reference images at once. — [RunComfy workflow page](https://www.runcomfy.com/comfyui-workflows/qwen-image-edit-2511-in-comfyui-precision-instruction-editing)
- The "Character Turnaround Sheet" LoRA for Qwen Image Edit 2511/2509 outputs a composite multi-angle image.
  - Recommended input is 600x1080; output is 2600x1080 or 3000x1080.
  - Needs the Qwen text encoder and VAE. — [Civitai 2149265](https://civitai.com/models/2149265/character-turnaround-sheet-qwen-image-edit-25112509); [HF tarn59/character_turnaround_sheet_qwen_edit_2511](https://huggingface.co/tarn59/character_turnaround_sheet_qwen_edit_2511) (HF blocked)
- An open-source GPU service added a "turnaround mode" on the official Qwen-Image-Edit-2511. It takes a face image plus outfit/hair text and outputs a front/side/back full-body turnaround on white. — [wanly-gpu-docker PR #160](https://github.com/DavidJBarnes/wanly-gpu-docker/pull/160), [issue #157](https://github.com/DavidJBarnes/wanly-gpu-docker/issues/157)
- A free "Anime Character Sheet Creator" workflow exists for Qwen Image Edit 2511. — [Patreon](https://www.patreon.com/posts/free-version-147202249)
- Two-stage pattern: an LLM writes a production character-sheet prompt from the reference image, then a ComfyUI Qwen-Image workflow generates the sheet. — [AGI Hunt](https://agihunt.info/en/p/1a0e83a85020dd39a9298711e37)
- Nano Banana Pro turnaround prompting:
  - Name "front view / side view / back view" explicitly and request a T-pose.
  - Typical layout: 4 full-body views (front, left, right, back) plus 3 face close-ups. — [nanobananapro.photo](https://nanobananapro.photo/prompts/character-sheet), [nanoprompts.org](https://nanoprompts.org/lab/2025/game-character-sheet-design) (prompt-library sites, low authority); [pIXELsHAM Apr 2026](https://www.pixelsham.com/2026/04/18/creating-a-character-sheet-for-ai-videos-using-nano-banana/)
- Scenario.com documents a turnaround workflow. — [Scenario help](https://help.scenario.com/articles/1419523552-generate-character-turnarounds)
- A Daz3D forum post describes exactly the "dress your own render" workflow:
  1. Render a T-posed figure.
  2. Use AI inpainting (Fooocus / Photoshop Generative Fill) to dress it.
  3. Erase everything but the garment.
  4. Convert to a mesh with Meshy or Hunyuan.
  5. Retopologize and UV in Blender. — [Daz3D forum thread](https://www.daz3d.com/forums/discussion/718631/do-you-know-an-ai-to-create-cloth-and-outfit) (via search snippet)
- CharacterGen's multi-view UNet canonicalizes a posed single image into consistent A-pose multi-views. It was trained on 13,746 anime characters and is useful when the only reference is a dynamic manga panel. — [CharacterGen GitHub](https://github.com/zjp-shadow/CharacterGen)

### Inferences
- Concrete prompt pattern for Qwen 2511 / Nano Banana Pro:
  - Inputs: (img1) the user's A-pose body render, front; (img2) the manga panel; (img3) an optional colored reference.
  - Prompt: "Dress the character in image 1 in the exact outfit from image 2, keep image 1's body proportions, pose, camera and framing unchanged, colors: [list], flat cel colors, even lighting, no cast shadows, plain white background."
  - Repeat for the side and back renders, feeding the front result back in as a reference for consistency.
- Also generate an outfit-only "flat lay" sheet (front and back of each garment, no body). It is useful for MD pattern shapes and for texture decals/emblems.
- Requesting "even lighting / no shadows" prevents baked shading. That matters when the images are later projected as textures.

### Gaps
- No benchmark found comparing Qwen 2511, Nano Banana Pro, GPT-image and FLUX.2/Kontext on keeping a supplied body render's proportions while dressing it.
- License of the community turnaround LoRA was not verified (HF blocked).

## Route 1: Marvelous Designer 2026 (or CLO), sewing on the user's avatar and exporting to UE5

### Takeaway
Marvelous Designer (MD) is the most direct way to get truly separate, correctly fitted garments on an existing custom body.
- 2026.0 (Apr 2026) added the features most relevant here: VRM/glTF avatar import, blendshapes on imported avatars, the 3D Pencil for drawing patterns directly on the avatar, a Toon Shader, and Unreal LiveSync/USD.
- 2026.1 (Aug 2026) added brush-based pinching for hand-authored folds and OBJ UV preservation.
- Quad (Optimized) remesh and the EveryWear toolkit (polygon reduction, auto-rig/weights, UV packing, baking) cover game-ready output.
MD has no AI sewing-pattern generator from images. Its "AI" features are texture/graphic generators from 2024.1 onward, so patterns are traced manually from the turnarounds.

### Cited Findings
- MD 2026.0 (Apr 2026):
  - 3D Pencil: draw pattern outlines on or around the avatar.
  - Lacing Tool; Toon Shader with outlines and rim light.
  - glTF and VRM avatar import, with blendshape support on imported avatars to avoid penetration.
  - Export of animation ranges to FBX/glTF.
  - Unreal LiveSync plugin and USD simulation-data export; MetaHuman garment round-trip. — [The Rookies](https://www.therookies.co/blog/headlines/marvelous-designer-2026), [Digital Production](https://digitalproduction.com/2026/04/15/marvelous-designer-2026-0-adds-3d-pencil-and-lacing/), [CG Channel 2026.0](https://www.cgchannel.com/2026/04/clo-virtual-fashion-releases-marvelous-designer-2026-0/) (blocked; snippet)
- MD 2026.1 (Aug 2026):
  - Brush Pinching (paint folds and wrinkles freehand); temporary simulation override while pinching for stable wrinkle shapes.
  - Experimental Seamline Rip (fraying).
  - Blendshape value recording on the avatar.
  - Camera import/export; Garment OBJ UV Preservation.
  - Toolbar customization and time-based autosave; zipper and pattern-mirroring updates; macOS GPU simulation.
  - AI texture/graphic generators date from 2024.1 and are not new in 2026.1. — [MD 2026.1 feature list](https://support.marvelousdesigner.com/hc/en-us/articles/59743730927129-Marvelous-Designer-2026-1-New-Feature-List) (blocked; snippet), [CG Channel 2026.1](https://www.cgchannel.com/2026/08/clo-virtual-fashion-releases-marvelous-designer-2026-1/), [textalks](https://textalks.com/clo-virtual-fashion-has-released-marvelous-designer-2026-1/)
- MD AI Studio plug-in activation page exists. — [MD support: AI Studio Plug-In](https://support.marvelousdesigner.com/hc/en-us/articles/49694260744089-AI-Studio-Plug-In-Activation)
- Mesh, fitting and export:
  - Mesh options are Triangle, Quad (Optimized) and Quad (Grid); Quad (Optimized) is auto retopology with an improved algorithm. Inspect edge flow, deformation, UVs and poly budget afterwards; manual cleanup may be needed.
  - Auto Fitting resizes a garment to a custom avatar and simulates it, with automatic fitting-suit creation for humanoid FBX imports and an option to preserve garment (quad) topology.
  - Export: FBX, Alembic, USD. — [MD Auto Fitting doc](https://support.marvelousdesigner.com/hc/en-us/articles/47358335130649-Auto-Fitting) (blocked; snippet), [MD "best clothing software 2026" guide](https://www.marvelousdesigner.com/explore/guide/best-3d-clothing-cloth-simulation-software-2026) (vendor)
- EveryWear (CLO/MD) optimizes garments for games and metaverse. It covers polygon optimization, auto-rigging, weight painting, UV adjustment, UV packing and texture baking, plus an experimental Rig Template system. A 2026.0 VRChat outfit tutorial uses it. — [CLO Connect: EveryWear](https://connect.clo-set.com/everywear), [What is EveryWear](https://support-connect.clo-set.com/hc/en-us/articles/45304312078873-What-is-EveryWear), [MD news: VRChat outfit with EveryWear 2026.0](https://www.marvelousdesigner.com/support/news/view/b43d71d4ad08419d8f3cff28b8dbd2bc)
- CLO MD Japan publishes a basic MD guide for VRChat outfits, which shows the anime-avatar use case is officially supported. — [note.com clo_md_japan](https://note.com/clo_md_japan/n/n919ce239f960?hl=en)
- MD -> Unreal practicalities:
  - FBX export: switch "thick" to "thin", choose multiple objects, set scale to cm.
  - USD/FBX export in A-pose; USD carries simulation data for UE 5.4+.
  - LiveSync does one-click transfer of mesh, materials, skeletal animation and geometry caches.
  - In UE 5.4+ a Transfer Skin Weights dataflow node (Closest Point on Surface) can skin a garment render mesh from a skeletal mesh. — [virtualfilmer: MD -> Maya for MetaHuman](https://virtualfilmer.com/how-to-export-from-marvelous-designer-to-maya/), [MD -> MetaHuman USD workflow](https://support.marvelousdesigner.com/hc/en-us/articles/52699135975705-Marvelous-Designer-to-MetaHuman-USD-Garment-Integration-Workflow) (blocked; snippet), [MD + Unreal tips](https://support.marvelousdesigner.com/hc/en-us/articles/47358145573401--Tips-Tricks-Discover-Better-Workflow-with-Marvelous-Designer-and-Unreal-Engine), [CLO LiveSync](https://connect.clo-set.com/livesync)
- A third-party Maya tool transfers an MD garment onto a quad mesh with UVs, which suggests MD's own quad output is often reworked for hero assets. — [ArtStation MD/Maya quad-UV tool](https://www.artstation.com/marketplace/p/5jrad/marvelous-designer-and-maya-quad-mesh-uv-transfer-tool)

### Inferences
- Recommended MD steps for this user:
  1. Export the base body as FBX in A-pose (or VRM if it is a VRoid/VRM body) and import it as the avatar. Let MD auto-create the fitting suit.
  2. Put the AI turnaround images on reference planes or in the 2D window. Trace patterns with 2D tools or the 3D Pencil.
  3. Simulate, then "anime-ify" with fewer, bigger folds: stiffer fabric presets, plus 2026.1 Brush Pinching for stylized folds. Preview with the Toon Shader.
  4. Run Quad (Optimized) remesh and set UVs (pattern-based UVs come free).
  5. Export thin (single-layer) garments in cm as FBX, then skin in Blender or UE. EveryWear can auto-rig and weight inside the CLO ecosystem.
- Anime garments are usually heavier, simpler shells than realistic cloth. Expect to simplify the simulated mesh: smooth out micro-wrinkles, keep large silhouette folds, add thickness with Solidify in Blender.
- Pricing: MD is a subscription with personal plans. Not verified this session; see Gaps.

### Gaps
- MD 2026 pricing, personal-license terms and system requirements were not verified (MD support site blocked).
- Whether EveryWear's auto-rig can target an arbitrary custom skeleton (vs. VRChat/humanoid templates) was not confirmed.
- No source on how well MD Auto Fitting handles stylized anime proportions (large head, thin limbs).

## Route 2: Blender modeling (duplicate body / shrinkwrap / solidify, cloth sim, sculpted anime folds, add-ons)

### Takeaway
Blender gives full control and is free. The best stylized results usually come from modeling directly on the body, not from simulation:
1. Duplicate the body's faces for tight garments.
2. Shrinkwrap with an offset, then Solidify.
3. Model loose parts (skirts, coats) with simple poly modeling.
4. Optionally run a short cloth sim, then sculpt simplified anime folds.
Add-ons that bring MD-like sewing into Blender include Simply Cloth Studio 2.0 (Jan 2026; pattern drawing on the character, sewing, adaptive wrinkle system) and Garment Tool (2D-pattern-curve sewing). GarmentCode (MIT, ETH) is a parametric pattern framework with a Warp simulator and OBJ/SVG export. It is not a Blender add-on, and custom-body support goes through body measurements, not arbitrary meshes.

### Cited Findings
- Simply Cloth Studio 2.0 (Vjaceslav Tissen, Jan 2026) is rebuilt from the ground up:
  - Draw clothing parts around a ready-made character in the viewport and sew them, producing a simulation-ready mesh.
  - Fabric presets (cotton, denim, leather, silk, wool, rubber).
  - New adaptive wrinkle system using geometry nodes (curvature-based wrinkle maps with intensity/direction/detail control).
  - Start/Design/Finish staged UI; "Paint Cloth" with sculpt brushes (bend, grab, inflate, twist). — [CG Channel Jan 2026](https://www.cgchannel.com/2026/01/vjaceslav-tissen-releases-simply-cloth-studio-2-0-for-blender/) (blocked; snippet), [Digital Production](https://digitalproduction.com/2026/01/27/simply-cloth-studio-2-0-rebuilds-cloth-in-blender/), [Superhive FAQ](https://superhivemarket.com/products/simply-cloth/faq)
- Garment Tool (Blender add-on by JoseConseco / Bartosz Styperek) has quick-start docs and is sold on Gumroad. — [Garment Tool docs](https://joseconseco.github.io/GarmentToolDocs/quick_guide/) (blocked), [Gumroad](https://bartoszstyperek.gumroad.com/l/GarmentTool?ref=311)
- GarmentCode:
  - Modular programming framework/DSL for parametric sewing patterns. MIT license.
  - Simulation via NVIDIA Warp; outputs OBJ meshes and SVG patterns.
  - Has a local Python GUI and an online configurator (garmentcode.ethz.ch, back online June 3, 2025); v2.0.0 (Aug 2024) added the GarmentCodeData dataset.
  - Body customization via GarmentMeasurements/body presets. Direct import of arbitrary custom body meshes is not documented. No Blender or MD integration is mentioned. — [GarmentCode GitHub](https://github.com/maria-korosteleva/GarmentCode)
- Commercial anime outfit packs exist with welded/unwelded variants and semi-automated low/high-poly retopology versions. They are good starting points to kitbash and refit. — [soheilkianfar outfit pack](https://soheilkianfar.gumroad.com/l/WarriorOutfit-TojiAndMegumiFushiguro); VRoid clothing packs: [cikanindya](https://cikanindya.gumroad.com/l/CasualOutfitsforVirtualStreamers)

### Inferences
These are standard practitioner techniques from prior knowledge, not verified via a fetched source this session.
- Tight garments (shirts, leggings, bodysuits):
  1. Select body faces and Duplicate + Separate (P).
  2. Add Shrinkwrap (offset ~2–5 mm), then Solidify (~3–8 mm).
  3. Add support edge loops at hems.
  This reuses the body's deformation-friendly topology and UV layout, so weights transfer almost perfectly.
- Skirts, capes, coats: start from a cylinder or plane and poly-model the silhouette from the turnaround backdrop. Optionally run a short Blender cloth sim with collision on the body, then apply it and sculpt big readable folds (Crease/Draw Sharp) and remove micro-wrinkles. Anime style favors 3–6 bold folds per garment.
- Retopology (only if you simulated or sculpted densely): Quad Remesher (paid), Instant Meshes (free) or RetopoFlow; then UV with seams along garment seams.
- Blender is free. Simply Cloth Studio and Garment Tool are paid add-ons; prices were not verified.

### Gaps
- No fetched source for current Simply Cloth Studio or Garment Tool pricing or Blender-version compatibility (Blender 4.x/5.x).
- No dedicated anime-clothing Blender tutorial was fetched to cite fold-stylization practice.

## Route 3: AI generation of garment meshes (image-to-3D, part separation, StdGEN clothes layer) and sewing-pattern AI (ChatGarment, GarmentCode, AIpparel, Design2GarmentCode, Dress-1-to-3)

### Takeaway
Image-to-3D generators produce fused, closed meshes. To get a garment you must:
1. Generate from a garment-only image (an outfit on an invisible body, or a "dressed body render" with the body erased), or generate the dressed figure and separate it with segmentation (Tripo segmentation, Hunyuan3D-Part P3-SAM/X-Part, HoloPart, PartCrafter).
2. Then shrinkwrap or refit the result onto the user's body and retopologize.
StdGEN (Apache-2.0) is the only anime-native open model that outputs a separate clothing layer, but it builds its own body, so the clothes must be refit to the user's body.
Sewing-pattern AI (ChatGarment, AIpparel, Design2GarmentCode, Dress-1-to-3) is research code that drapes on SMPL/SMPL-X bodies. It is mostly realistic-garment oriented, and its outputs (patterns) would need re-sewing in MD/GarmentCode on the user's avatar. Interesting, but not yet the easy path.

### Cited Findings
Image-to-3D (condensed):
- TRELLIS.2 (Microsoft):
  - 4B parameters, Dec 2025, MIT.
  - PBR (base color/roughness/metallic/opacity) at 512³–1536³.
  - Needs at least 24GB VRAM, Linux only, CUDA 12.4. Install is moderate-to-high effort (compiles flash-attn, nvdiffrast, cumesh, o-voxel, flexgemm).
  - No part segmentation. GLB exports opaque by default. — [TRELLIS.2 GitHub](https://github.com/microsoft/TRELLIS.2)
- Hunyuan3D:
  - 2.1 (June 2025) is the latest open-weight version: 10GB VRAM for shape, 21GB for paint, 29GB for both. Windows/Linux/macOS. Tencent Hunyuan Community License. Windows portable build available.
  - 2.5, 3.0 and 3.1 are hosted/API only; 3.1 accepts up to 8 views. — [Hunyuan3D-2.1 GitHub](https://github.com/tencent-hunyuan/hunyuan3d-2.1), [WinPortable](https://github.com/YanWenKun/Hunyuan3D-2-WinPortable), [wireflow.ai](https://www.wireflow.ai/blog/best-hunyuan3d-v3-tools-in-2026) (aggregator)
- Hunyuan3D-Part (open source, Sep 2025):
  - P3-SAM: native 3D point-promptable part segmentation, trained on 3.7M shapes.
  - X-Part: part generation/completion (light version open; full version in Hunyuan3D Studio).
  - ComfyUI wrapper available. — [Hunyuan3D-Part GitHub](https://github.com/Tencent-Hunyuan/Hunyuan3D-Part), [ComfyUI-Hunyuan3D-Part](https://github.com/PozzettiAndrea/ComfyUI-Hunyuan3D-Part), [Tencent Hy on X](https://x.com/TencentHunyuan/status/1971491034044694798)
- Tripo (cloud):
  - Smart Mesh: quads in the 500–50K range, claimed engine-ready.
  - Tripo 3.1 adds AI segmentation into components and auto-rig.
  - Smart Mesh P1.0 is a native-3D low-poly generator. — [Meshy blog (competitor)](https://www.meshy.ai/blog/hi3d-vs-meshy-vs-tripo), [3DAIStudio](https://www.3daistudio.com/blog/hitem3d-vs-meshy-vs-tripo-comparison), [Barchart PR](https://www.barchart.com/story/news/936837/tripo-ai-introduces-smart-mesh-p1-0-defining-a-new-phase-for-ai-3d-production)
- Meshy 6 (cloud): quad topology option, A/T-pose control, 1–4 input views. A Meshy 7/7.1 line exists. — [Meshy vs Rodin (vendor)](https://www.meshy.ai/compare/meshy-vs-rodin) (blocked; snippet), [Meshy help 7.1 vs 6](https://help.meshy.ai/en/articles/15697060-meshy-7-vs-meshy-6-vs-meshy-6-lite-which-ai-model-should-you-use) (blocked)
- Rodin Gen-2/2.5 (cloud): highest geometry density; high-poly quads on the Business plan. — [Meshy vs Rodin (vendor)](https://www.meshy.ai/compare/meshy-vs-rodin), [StraySpark Apr 2026](https://www.strayspark.studio/blog/generative-3d-tools-comparison-meshy-rodin-tripo-csm-2026)
- PartCrafter: 2–16 separate meshes from one image. PartPacker (NVIDIA): part-level generation via dual volume packing. HoloPart: segments an existing mesh, then completes each part amodally. — [PartCrafter arXiv](https://arxiv.org/html/2506.05573v1), [PartPacker GitHub](https://github.com/NVlabs/PartPacker), [awesome-3dv-weekly (HoloPart)](https://github.com/FishWoWater/awesome-3dv-weekly)
- StdGEN (CVPR 2025):
  - Apache-2.0. Separate body, clothes and hair from a single anime image.
  - Python 3.9 / PyTorch 2.1; `--low_vram` flag. Full-body input required; weaker on non-anime styles. — [StdGEN GitHub](https://github.com/hyz317/StdGEN)
- StdGEN++ (Jan 2026) adds a dual-branch S-LRM, a facial branch, higher resolution and video-diffusion texture layer decomposition. No public code found. — [StdGEN++ arXiv](https://arxiv.org/pdf/2601.07660) (blocked; snippet)
- Anime-Ready (ICLR 2026) generates body-aligned component-wise garments (hair, upper, lower, accessories) on an Anime-SMPL body. Code "TBD". — [ICLR poster](https://iclr.cc/virtual/2026/poster/10010948), [Liner review](https://liner.com/review/animeready-controllable-3d-anime-character-generation-with-bodyaligned-componentwise-garment)
- SAM 3D Objects (Meta, Nov 2025): single-image object reconstruction under the custom SAM License. — [Roboflow](https://blog.roboflow.com/sam-3d/)

Sewing-pattern AI:
- ChatGarment (CVPR 2025):
  - Apache-2.0. A fine-tuned VLM (LLaVA/LISA-based) turns an image, sketch or text into GarmentCode-style JSON, then a 2D sewing pattern (GarmentCodeRC), then a drape (ContourCraft-CG simulation).
  - The README warns it "may occasionally produce garments with incorrect lengths or widths from input images".
  - Multi-turn chat is "coming soon". — [ChatGarment GitHub](https://github.com/biansy000/ChatGarment)
  - Draping is on an SMPL-X body. — [ChatGarment arXiv html](https://arxiv.org/html/2412.17811v1) (via snippet)
- AIpparel (CVPR 2025): multimodal foundation model for sewing patterns trained on 120k+ garments. MIT; pretrained weights downloadable. Repo: georgeNakayama/AIpparel-Code. — [AIpparel GitHub](https://github.com/georgeNakayama/AIpparel-Code), [CVPR paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Nakayama_AIpparel_A_Multimodal_Foundation_Model_for_Digital_Garments_CVPR_2025_paper.pdf)
- Design2GarmentCode (CVPR 2025): an LMM generates parametric GarmentCode pattern-making programs (Python) from multimodal design concepts. — [CVPR paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Zhou_Design2GarmentCode_Turning_Design_Concepts_to_Tangible_Garments_Through_Program_Synthesis_CVPR_2025_paper.pdf)
- Dress-1-to-3 (2025, ACM TOG): in-the-wild image -> separated, simulation-ready garments with sewing patterns. It combines a pre-trained image-to-pattern model, multi-view diffusion and a differentiable garment simulator. — [project page](https://dress-1-to-3.github.io/), [ACM DL](https://dl.acm.org/doi/10.1145/3731177)
- Image2Garment (Jan 2026): simulation-ready garment from a single image. — [arXiv 2601.09658](https://arxiv.org/html/2601.09658v4) (blocked; title only)
- GarmageNet (2025): multimodal generative framework for sewing patterns. — [arXiv 2504.01483](https://arxiv.org/html/2504.01483v3)

### Inferences
- The practical AI-garment recipe that works on the user's existing body:
  1. Dress the body render in Qwen 2511 / Nano Banana.
  2. Erase the body, leaving the garment only (or keep the body and segment later).
  3. Run image-to-3D (Hunyuan 2.1 local, TRELLIS.2 local, or Tripo/Meshy cloud with front+back views).
  4. Import into Blender, align to the body, cut open the closed volume (AI garments come out as solid closed shells), delete inner faces, and keep the outer shell.
  5. Shrinkwrap/Surface Deform to the body, retopologize, and UV.
  Or import the shell into MD as an "avatar-like" reference and resew it properly.
- AI garment meshes are best for rigid or semi-rigid pieces: armor, belts, shoes, hats, bags, buckles, ribbons and bows, emblems. They are worst for skirts and coats that need clean edge loops for deformation or cloth sim.
- Sewing-pattern AI produces patterns on SMPL-X proportions. Possible use: run ChatGarment or AIpparel on the colored turnaround, export the pattern (SVG/JSON), import the pattern shapes into MD as a starting point, and Auto Fit to the anime avatar. Expect realistic-fashion bias and wrong proportions; anime-specific shapes (puffy sleeves, sailor collars, frilly layers) are likely poorly covered. All these repos need conda setups with research dependencies (moderate-to-high effort).
- License summary for personal use:
  - MIT: TRELLIS.2, GarmentCode, AIpparel.
  - Apache-2.0: StdGEN, ChatGarment.
  - Tencent Hunyuan Community License: Hunyuan3D 2.1 and Part. Prior knowledge, not re-verified: it excludes EU/UK/South Korea.
  - Custom: SAM License.
  - Cloud tools: per vendor ToS.

### Gaps
- No fetched source tested any image-to-3D tool on garment-only anime inputs.
- No source confirmed whether the Tripo or Meshy APIs can generate open (non-watertight) garment shells.
- VRAM for ChatGarment, AIpparel and Design2GarmentCode is undocumented in what was fetched. Whether Design2GarmentCode and Dress-1-to-3 code is public was not confirmed.
- No reports found of anyone using these pattern AIs on anime avatars.

## Texturing clothes in flat anime style, HD, without training

### Takeaway
Anime garments need flat albedo plus a toon shader, not PBR. The easiest high-quality approach is to UV the garment (MD gives pattern UVs for free), then either hand-paint flat fills, lines and emblems, or project the AI-generated "dressed" turnaround images onto the mesh. Projection options: StableProjectorz (free, AGPL-3.0 since Jan 2026, multi-view projection with ComfyUI/Forge backends, bundled Trellis/Hunyuan installers) or Hunyuan3D-Paint (2.1, local, can texture a user-supplied mesh). Cloud retexture (Meshy/Tripo) tends to bake shading and PBR detail into the texture, so it needs flattening.

### Cited Findings
- StableProjectorz:
  - Free desktop app that textures a 3D asset by projecting Stable Diffusion generations while preserving the original UV layout, on one consumer NVIDIA GPU.
  - Up to 6 user-placed view arrangements with per-view weight masks and UV-space inpaint masks.
  - Two modes: 2D texturing via AUTOMATIC1111/Forge/ComfyUI, and bundled one-click 3D generation via Trellis 1/2 and Hunyuan3D 2.0/2.1.
  - First released Jan 2024; open-sourced under AGPL-3.0 in Jan 2026. — [ACM paper listing](https://dl.acm.org/doi/10.1145/3799825.3818726), [stableprojectorz.com](https://www.stableprojectorz.com/), [trellis-stable-projectorz GitHub](https://github.com/IgorAherne/trellis-stable-projectorz), [CGPress](https://cgpress.org/archives/stable-projectorz-an-ai-tool-for-3d-texture-generation.html)
- Hunyuan3D-Paint can texture a provided mesh: generate or supply the mesh first, then run the paint pipeline with mesh + image. ComfyUI texture support needs a wrapper (Kijai Hunyuan3DWrapper, or visualbruno's ComfyUI-Hunyuan3d-2-1) with a compiled custom rasterizer and differentiable renderer; native ComfyUI lacks the paint stage. — [ComfyUI wiki Hunyuan3D-2](https://comfyui-wiki.com/en/tutorial/advanced/3d/huanyuan3d-2), [visualbruno ComfyUI-Hunyuan3d-2-1](https://github.com/visualbruno/ComfyUI-Hunyuan3d-2-1), [Yuan-ManX ComfyUI-Hunyuan3D-2.1](https://github.com/Yuan-ManX/ComfyUI-Hunyuan3D-2.1)
- Hunyuan3D-2.1 paint produces PBR materials and needs about 21GB VRAM (fits 24GB). — [Hunyuan3D-2.1 GitHub](https://github.com/tencent-hunyuan/hunyuan3d-2.1)
- MD's 2026.0 Toon Shader previews cartoon clothing inside MD; EveryWear does UV packing and texture baking. MD has had AI texture/graphic generators since 2024.1. — [CLO EveryWear](https://connect.clo-set.com/everywear), [The Rookies](https://www.therookies.co/blog/headlines/marvelous-designer-2026), [textalks](https://textalks.com/clo-virtual-fashion-has-released-marvelous-designer-2026-1/)
- VRM4U recreates the MToon toon material in UE (shadow color, outlines, MatCap). — [VRM4U GitHub](https://github.com/ruyo/VRM4U)

### Inferences
- Flat-anime texturing recipe:
  1. In Blender, assign material IDs per color region and bake a flat ID map, or paint fills directly. 2K is enough for flat colors; 4K only for fine emblems or lace.
  2. Add line details (seams, trims) as painted lines or a separate "line" texture.
  3. Add emblems/patterns as decals: Substance Painter stencils/projection, or Blender texture-paint stencil from the AI flat-lay sheet.
  4. If projecting AI images, posterize or hand-correct afterwards to remove soft gradients.
  5. In UE5, use a cel/toon material (MToon via VRM4U, or a custom post-process/material toon shader) with shadow-color ramps instead of baked shading.
- Substance Painter (Adobe; paid subscription or Steam perpetual, prior knowledge) is the polished hand-paint option. Blender Texture Paint and the free StableProjectorz cover a zero-cost path.

### Gaps
- No fetched source on Meshy/Tripo "retexture uploaded model" features or their ability to output flat, unlit colors.
- No fetched source documenting an anime-specific UE5 toon-material plugin beyond MToon/VRM4U.

## Fitting to the rig: skin weight transfer, hiding the body under clothes, extra bones for skirts/coats, UE5 physics

### Takeaway
Skin garments to the body's existing skeleton by transferring weights from the body.
- Robust Weight Transfer (free, GPL-3.0 Blender add-on, based on Epic's SIGGRAPH Asia 2023 "Robust Skin Weights Transfer via Weight Inpainting") is the best one-click option. It specifically fixes gaps at armpits, between the legs, on skirts and on loose sleeves, where Blender's Data Transfer fails.
- In UE5, give skirts and coats extra bone chains driven by Kawaii Physics (anime-style bone physics, used in Wuthering Waves, Persona 3 Reload, Stellar Blade; UE 5.3–5.6 noted), or use Chaos Cloth. In 5.8 the Chaos Dataflow Cloth Editor is production-ready and the default cloth editor; 5.6 introduced Outfit Assets with resizing/refitting.

### Cited Findings
- Robust Weight Transfer: one-click weight transfer from body to targets that handles between-legs, chest and armpit areas without manual smoothing. GPL-3.0, free, based on Abdrashitov et al. (Epic Games), SIGGRAPH Asia 2023. — [robust-weight-transfer GitHub](https://github.com/sentfromspacevr/robust-weight-transfer), [paper preprint](https://www.dgp.toronto.edu/~rinat/projects/RobustSkinWeightsTransfer/preprint.pdf), [author reference code](https://github.com/rin-23/RobustSkinWeightsTransferCode)
- Standard weight transfer fails where gaps exist (armpits, inner shoulders, between legs on trousers/skirts, loose sleeves/capes, layered outfits); this add-on addresses those cases. — [3dxdev listing](https://3dxdev.com/assets/robust-weight-transfer-one-click-weights-for-blender/), [YouTube demo](https://www.youtube.com/watch?v=9Bg42lA6TV8)
- A paid "Skin Cloth Transfer Weights" tool is on Fab. — [Fab listing](https://www.fab.com/listings/cd21f4ab-fab7-49c2-a9cf-f6c118d2beb1)
- UE 5.4+ can transfer skin weights in-engine with a Transfer Skin Weights dataflow node (Closest Point on Surface onto the garment render mesh). — [virtualfilmer / MD -> MetaHuman snippet](https://virtualfilmer.com/how-to-export-from-marvelous-designer-to-maya/)
- Kawaii Physics (pafuhana1213, an Epic Games employee) is simple anime-style bone physics for hair, skirts and ears. It preserves bone length, avoids PhysX overhead, is used in major titles, and supports UE 5.3–5.6. — [80.lv](https://80.lv/articles/kawaii-physics-plug-in-now-supports-unreal-engine-5-5), [GitHub](https://github.com/pafuhana1213/KawaiiPhysics), [80.lv tutorial](https://80.lv/articles/learn-how-to-use-fake-physics-plug-in-kawaii-physics-for-unreal-engine-5)
- UE 5.6 Chaos Cloth: Beta Panel Editor on the experimental Unified Dataflow Editor; Outfit Asset with resizing/refitting; cloth-to-cloth constraints; simulation morph targets. In 5.8 the Chaos Dataflow Cloth Editor is production-ready and default. — [Epic forum: Chaos Cloth 5.6](https://forums.unrealengine.com/t/tutorial-chaos-cloth-updates-5-6/2555686), [Chaos Cloth 5.8](https://forums.unrealengine.com/t/tutorial-chaos-cloth-updates-5-8/2729420), [Panel Cloth Editor docs](https://dev.epicgames.com/documentation/unreal-engine/panel-cloth-editor-overview)
- An indie dev log covers skirt animation with Chaos Clothing in UE. — [Absolute Tenebra devlog](https://marcheadroom.itch.io/absolute-tenebra/devlog/1164789/character-skirt-animation-with-chaos-clothing-unreal-engine)
- VRM4U imports VRM with spring bones and auto-generates a Humanoid rig for UE mannequin retargeting. MIT. 2026 updates add UE 5.8 support. Useful if the base body is VRM/VRoid. — [VRM4U GitHub](https://github.com/ruyo/VRM4U)
- MD can import VRM/glTF avatars (2026.0) and EveryWear offers auto-rig/weight painting. — [The Rookies](https://www.therookies.co/blog/headlines/marvelous-designer-2026), [CLO EveryWear](https://connect.clo-set.com/everywear)

### Inferences
- Fit steps for this user:
  1. Parent each garment to the body armature and run Robust Weight Transfer (or Data Transfer with Nearest Face Interpolated, then smooth) from the body.
  2. Check extreme poses and fix the shoulders and crotch.
  3. For skirts and coat tails, add 4–8 radial bone chains (3–5 bones each) and weight the skirt to them with a gradient from the hip. In UE, drive them with Kawaii Physics plus collision capsules on the thighs.
  4. Use Chaos Cloth only where you want real cloth motion. Paint max-distance masks and keep the sim mesh low-res.
- Poke-through prevention:
  - Delete or mask body faces fully covered by tight garments. Keep a separate "body under outfit" mesh variant, or use a material opacity mask per outfit in UE.
  - Add a small Shrinkwrap offset.
  - Weight-transfer so garment and body share the same influences.
- Export to UE5 as one skeletal mesh (body + garments as material sections), or as separate skeletal meshes on the same skeleton for swappable outfits (Leader Pose / Copy Pose component). Separate meshes suit a "dress-up" design.

### Gaps
- Robust Weight Transfer's supported Blender versions are not stated in the README.
- Kawaii Physics UE 5.7/5.8 support was not confirmed (sources say 5.3–5.6).
- Auto-Rig Pro (paid Blender add-on) was not researched this session.

## Which route gives the best HD result with the easiest setup, realistic time per outfit, and pitfalls

### Takeaway
Best quality-to-effort for a solo creator with an existing body:
1. Prep: AI-dressed turnarounds of the user's own body render (Qwen-Image-Edit-2511 local or Nano Banana Pro).
2. Garments: Marvelous Designer 2026.x. Pattern-trace from the turnarounds, use Toon Shader preview, Brush Pinching for stylized folds, Quad (Optimized) remesh, free pattern UVs.
   - Blender alternative: duplicate/shrinkwrap/solidify plus Simply Cloth Studio 2.0. Cheaper, more manual, often better for simple tight anime outfits.
3. Rigid accessories only: AI image-to-3D (Hunyuan3D 2.1 / TRELLIS.2 local, or Tripo/Meshy).
4. Texture: flat colors, hand-painted or projected (StableProjectorz).
5. Weights: Robust Weight Transfer.
6. UE5: Kawaii Physics for skirt/hair bones; Chaos Cloth where needed.
Pure AI garment generation and sewing-pattern AI exist but are not yet the easiest path to clean, deformable anime clothing.

### Cited Findings
- AI meshes for hero characters need a full retopology pass before rigging; irregular dense topology gives poor auto-rig results. — [Meshy auto-rig guide](https://www.meshy.ai/tutorials/character-auto-rigging-workflow) (vendor)
- Fused "monolithic" meshes (skin, hair and clothes fused) are "practically useless for professional gaming and animation pipelines". — [StdGEN++ arXiv](https://arxiv.org/pdf/2601.07660) (snippet)
- Recent image-to-3D models produce fused models, which limits garment animation. Dress-1-to-3's motivation is separable, simulation-ready garments. — [Dress-1-to-3](https://dress-1-to-3.github.io/)
- ChatGarment may produce wrong garment lengths/widths from images. — [ChatGarment GitHub](https://github.com/biansy000/ChatGarment)
- MD's automated quad mesh is a starting point that may need manual cleanup. — [MD guide (vendor)](https://www.marvelousdesigner.com/explore/guide/best-3d-clothing-cloth-simulation-software-2026)
- Sparc3D was announced as open source but is "no longer fully open", an availability pitfall for AI tools. — [Sparc3D issue #4](https://github.com/lizhihao6/Sparc3D/issues/4)
- Hunyuan 2.5/3.x are cloud only, so local users are capped at 2.1. — [wireflow.ai](https://www.wireflow.ai/blog/best-hunyuan3d-v3-tools-in-2026)
- TRELLIS.2 is Linux only and needs 24GB. — [TRELLIS.2 GitHub](https://github.com/microsoft/TRELLIS.2)

### Inferences
- Route comparison:

  | Route | HD / deformation quality | Setup | Separate garments | Notes |
  |---|---|---|---|---|
  | MD 2026 on user avatar | High (real patterns, clean UVs) | Installer + subscription | Yes, native | Must stylize folds; quad cleanup |
  | Blender shrinkwrap/solidify + modeling (+ Simply Cloth) | High if skilled; best for tight anime outfits | Free (add-ons paid) | Yes | Most manual; reuses body topology/UVs |
  | AI image-to-3D garment + refit | Medium-low for soft cloth; good for accessories | Local conda (moderate-high) or cloud | Only after cutting/segmenting | Closed shells, baked light, retopo needed |
  | Sewing-pattern AI (ChatGarment / AIpparel) -> MD | Unproven for anime | Research-code installs | Yes (patterns) | SMPL-X bias, size errors |
  | StdGEN clothes layer -> refit | Medium | Conda, Py3.9/PT2.1 | Yes, but on its own body | Low-res; must refit |

- Realistic time per outfit for a moderately complex anime outfit (top, skirt or pants, jacket, accessories). These are estimates from practitioner experience, not sourced:
  - Reference prep: 0.5–1.5 h.
  - MD or Blender garment build: 3–10 h.
  - Retopo/UV cleanup: 2–6 h.
  - Flat texturing: 2–5 h.
  - Weights + skirt bones + UE physics/material setup: 2–5 h.
  - Total: about 1–3 working days. Simple tight outfits: half a day. AI accessories add ~0.5–1 h each including cleanup.
- Pitfalls:
  - (a) AI garments as closed, solid volumes with interior faces; must be cut open.
  - (b) Baked lighting and gradients in AI textures.
  - (c) Realistic MD micro-wrinkles that look wrong in anime style.
  - (d) Garment interpenetration at armpits and crotch after weight transfer.
  - (e) Skirt-leg clipping with no thigh collision.
  - (f) Proportion drift when AI editors "dress" the render (check the silhouette against the body).
  - (g) License: Tencent's territory restrictions; free cloud tiers' visibility/commercial terms; AGPL for StableProjectorz only matters if redistributing modified code.

### Gaps
- No sourced, measured time-per-outfit data found for any route.
- No head-to-head comparison of MD vs Blender add-ons for stylized anime garments.
- MD and Substance pricing not verified.
