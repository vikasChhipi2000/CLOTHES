# Manga character -> game-ready 3D anime character with separate clothing for UE5, no training (solo creator, 24GB+ GPU, as of 2026-10-04)

Research method note: ~30 tool calls. The egress proxy BLOCKED these domains, so their content is known only from search snippets: arxiv.org, huggingface.co (model cards and Spaces), en.papernotes.org, help.meshy.ai, www.meshy.ai, developers.tripo3d.ai, www.cgchannel.com, triposr.org. GitHub pages (StdGEN, TRELLIS.2, VRM4U) were fetched directly. Many 2026 "comparison" pages are vendor blogs (Meshy, 3DAIStudio, Neural4D) or SEO aggregators. Treat them as low-to-medium reliability; they are flagged where cited.

## Step 0: Reference prep without training (manga panels -> clean colored turnaround / outfit sheets)

### Takeaway
Zero-shot image editors can now turn one manga panel or colored sheet into a front/side/back turnaround on a white background. Nano Banana Pro (Gemini) is the easy cloud option. Qwen-Image-Edit-2511 in ComfyUI is the local option, and a community "Character Turnaround Sheet" add-on exists for it. That add-on is a LoRA someone else already trained; you just download and use it, with no training on your side. Character-specific multi-view generators (the CharacterGen and StdGEN multi-view stages) produce A-pose views as a built-in step, so they double as reference canonicalizers.

### Cited Findings
- Qwen-Image-Edit-2511 runs in ComfyUI. It improves character consistency, reduces drift across edits, and accepts up to three reference images at once. — [RunComfy workflow page](https://www.runcomfy.com/comfyui-workflows/qwen-image-edit-2511-in-comfyui-precision-instruction-editing)
- A community "Character Turnaround Sheet" LoRA exists for Qwen Image Edit 2511/2509. It outputs one composite image of the character from multiple angles. Recommended input is 600x1080 and output 2600x1080 or 3000x1080. It needs the Qwen text encoder and VAE. — [Civitai model 2149265](https://civitai.com/models/2149265/character-turnaround-sheet-qwen-image-edit-25112509); also listed as [tarn59/character_turnaround_sheet_qwen_edit_2511 on HF](https://huggingface.co/tarn59/character_turnaround_sheet_qwen_edit_2511) (HF blocked, seen via search only)
- One open-source GPU service added a "turnaround mode" built on the official Qwen-Image-Edit-2511. It takes a face image plus outfit/hair text and outputs a front, side and back full-body turnaround on white. — [wanly-gpu-docker PR #160](https://github.com/DavidJBarnes/wanly-gpu-docker/pull/160), [issue #157](https://github.com/DavidJBarnes/wanly-gpu-docker/issues/157)
- A free "Anime Character Sheet Creator" workflow for Qwen Image Edit 2511 is distributed on Patreon. — [Patreon post](https://www.patreon.com/posts/free-version-147202249)
- Two-stage pattern: upload the reference image to an LLM with a system prompt that writes a production-ready character-sheet prompt, then feed that prompt into a ComfyUI Qwen-Image workflow. This yields turnarounds, expression studies and detail grids. — [AGI Hunt writeup](https://agihunt.info/en/p/1a0e83a85020dd39a9298711e37)
- Nano Banana / Nano Banana Pro prompting advice:
  - Name "front view", "side view" and "back view" explicitly.
  - Ask for a T-pose, the standard pose for 3D-modeling reference.
  - Typical layout: a top row of four full-body views (front, left, right, back) and a bottom row of three face close-ups. — [nanobananapro.photo prompt library](https://nanobananapro.photo/prompts/character-sheet), [nanoprompts.org templates](https://nanoprompts.org/lab/2025/game-character-sheet-design) (prompt-library sites, low authority)
- Alternative Nano Banana recipe: upload a reference image and ask for front, 3/4, side and back views plus a face close-up, 6 panels on a neutral background. — [pIXELsHAM, Apr 2026](https://www.pixelsham.com/2026/04/18/creating-a-character-sheet-for-ai-videos-using-nano-banana/), [invideo FAQ](https://invideo.io/faq/how-do-you-create-a-character-reference-sheet-using-nano/)
- Scenario.com documents a "Generate Character Turnarounds" workflow. — [Scenario help](https://help.scenario.com/articles/1419523552-generate-character-turnarounds)
- CharacterGen (SIGGRAPH 2024, TOG):
  - Its multi-view UNet generates highly consistent A-pose multi-view images from a single posed input. This is "pose canonicalization", useful for converting a dynamic manga pose into an A-pose.
  - It was trained on 13,746 anime characters. Code: github.com/zjp-shadow/CharacterGen. — [CharacterGen GitHub](https://github.com/zjp-shadow/CharacterGen), [project page](https://charactergen.github.io/)
- StdGEN++ first canonicalizes any input (text or image) into multi-view RGB and normal maps under A-pose, then reconstructs. — [StdGEN++ arXiv 2601.07660](https://arxiv.org/pdf/2601.07660) (arXiv blocked; from search snippet)

### Inferences
- Recommended zero-training Step 0 pipeline:
  1. Colorize and clean the panel with Nano Banana Pro or Qwen-Image-Edit-2511, e.g. "colorize this manga character using these hair/eye/outfit colors, full body, plain white background, neutral A-pose, flat cel shading, no shadows".
  2. Generate the turnaround with the Qwen 2511 turnaround LoRA or a Nano Banana Pro prompt.
  3. Generate a separate outfit-only "flat lay / costume sheet" (garment front and back with no body). This gives Marvelous Designer pattern references and texture references.
- Ask for "flat colors, no cast shadows, even lighting". This avoids baked lighting that would later get projected into textures.
- Feed several views (front plus back) to any image-to-3D tool that accepts multi-view input (Tripo multi-view, Meshy 6 with 1–4 views, Hunyuan 3.1 with up to 8 views). Multi-view input reduces back-side hallucination.

### Gaps
- No primary source fetched for MV-Adapter, Zero123++, Era3D, FLUX.1 Kontext or FLUX.2 turnaround quality on anime. Era3D appears only as a comparison baseline in CharacterGen/StdGEN.
- No objective benchmark found on turnaround consistency (Nano Banana Pro vs GPT-image vs Qwen 2511) for B&W manga input specifically.
- Exact license of the community Qwen turnaround LoRA was not verified, because the HF model card is blocked. Qwen-Image-Edit itself is generally Apache-2.0 per prior knowledge (not verified this session).

## Image-to-3D generators for anime characters: which handle anime style and clothing separation best?

### Takeaway
Only the StdGEN family natively outputs separate body, clothing and hair meshes for anime characters, and only original StdGEN (Apache-2.0, CVPR 2025) has public code. Separately, Anime-Ready (ICLR 2026) would output a rigged anime body with separate garments, but its code is still "TBD". General generators produce one fused mesh: TRELLIS.2 (MIT, local, 24GB, PBR), Hunyuan3D 2.1 (local, community license), Hunyuan 3.x (cloud only), Tripo, Meshy and Rodin. For those, clothing separation means after-the-fact segmentation (Tripo segmentation, Hunyuan3D-Part P3-SAM/X-Part, PartPacker/PartCrafter, HoloPart) or manual work in Blender.

### Cited Findings
StdGEN / StdGEN++ (body, clothes and hair as separate layers):
- StdGEN code (github.com/hyz317/StdGEN):
  - License is Apache 2.0.
  - Inference code, dataset and checkpoints were released March 2025, along with a Gradio demo. Accepted at CVPR 2025.
  - Install: Python 3.9 conda, PyTorch 2.1.0, xformers, torch-scatter, requirements.txt, and HF weights plus a SAM model in ./ckpt/.
  - A `--low_vram` flag exists for the multi-view stage. Exact VRAM is not stated.
  - Limitations: designed for full-body input (half-body underperforms), weaker on 2.5D/realistic styles, background removal is critical, and hair refinement depends on mask accuracy.
  - No rigging. — [StdGEN GitHub](https://github.com/hyz317/StdGEN)
- StdGEN++ (arXiv 2601.07660, Jan 2026) outputs semantically decomposed characters in three categories: (1) a base minimally-clothed body, (2) clothing, (3) hair. It uses:
  - a Dual-Branch Semantic-aware LRM with Fullbody and Facial LoRA branches;
  - a coarse-to-fine proposal scheme that cuts memory and enables high-resolution meshes;
  - a video-diffusion texture decomposition module that produces editable layers;
  - training data: Anime3D-EX, 10,811 curated VRoid-Hub characters.
  Claims: immediate rigging, physics simulation and gaze tracking. — [StdGEN++ arXiv PDF](https://arxiv.org/pdf/2601.07660), [Lacuna summary](https://lacuna.tiptreesystems.com/work/stdgen-a-comprehensive-system-for-semantic-decomposed-3d-character-generation/wrk_01e6509ae847c8f156205ac2afcea46c)
- StdGEN++ code: search found no separate repo. The StdGEN GitHub README mentions no StdGEN++ code. — [StdGEN GitHub](https://github.com/hyz317/StdGEN), [search result listing](https://paperswithcode.co/paper/2601.07660)

Anime-Ready (ICLR 2026):
- Authors: Jiachen Qian, Hongye Yang, Youtian Lin, Tianhao Zhao, Feihu Zhang, Yao Yao, Hengshuang Zhao. Poster on Apr 24, 2026.
- Method: an extended "Anime-SMPL" body with a unified skeleton and blendshape facial expressions, plus body-aligned component-wise garments (hair, upper garment, lower garment, accessories) with skin, face and garment textures.
- Code status is listed as "TBD". — [ICLR poster page](https://iclr.cc/virtual/2026/poster/10010948), [Liner review](https://liner.com/review/animeready-controllable-3d-anime-character-generation-with-bodyaligned-componentwise-garment)

CharacterGen (SIGGRAPH 2024):
- Single image -> A-pose multi-view -> sparse-view reconstruction into one mesh (no clothing separation). Repo: zjp-shadow/CharacterGen. — [GitHub](https://github.com/zjp-shadow/CharacterGen)

TRELLIS.2 (Microsoft):
- 4B parameters, paper arXiv 2512.14692 (Dec 2025).
- Outputs PBR (base color, roughness, metallic, opacity) at 512³–1536³ voxel resolution.
- MIT license; nvdiffrast/nvdiffrec dependencies carry their own terms.
- Needs at least 24GB VRAM (tested on A100/H100). Linux only. CUDA 12.4.
- Install effort is moderate-to-high: flash-attn, nvdiffrast, cumesh, o-voxel and flexgemm are compiled during setup.
- No part segmentation. — [TRELLIS.2 GitHub](https://github.com/microsoft/TRELLIS.2)
- A third-party run reported just over 16GB VRAM during generation and just under 30GB while rendering. — [Medium hands-on](https://medium.com/@ammanakhtar8/one-image-in-3d-out-free-open-source-i-ran-trellis-2-3f7f5ed8e96c), via [search snippet](https://webkul.com/blog/trellis-2/)

Hunyuan3D:
- 2.1 (June 2025) is the latest open-weight version.
- 2.5, PolyGen, 3.0 and 3.1 are hosted only (Tencent platform/API). As of Sep–Oct 2026 there was no announcement of open weights.
- 3.1 (early 2026) improves texture and geometry detail and accepts up to 8 input views. — [wireflow.ai blog](https://www.wireflow.ai/blog/best-hunyuan3d-v3-tools-in-2026), [triposr.org comparison](https://triposr.org/blog/hunyuan3d-versions) (aggregators; triposr.org blocked, snippet only)
- Hunyuan3D 2.1 local:
  - VRAM: 10GB for shape, 21GB for texture (PBR paint), 29GB for both in one process.
  - MMGP offloading variants exist (~3GB for shape, ~6GB for texture).
  - Runs on Windows, Linux and macOS.
  - License: Tencent Hunyuan Community License.
  - A Windows portable build exists (YanWenKun/Hunyuan3D-2-WinPortable). — [Hunyuan3D-2.1 GitHub](https://github.com/tencent-hunyuan/hunyuan3d-2.1), [WinPortable](https://github.com/YanWenKun/Hunyuan3D-2-WinPortable)
- Tencent has launched the Hunyuan 3D Engine globally (hosted creation tools). — [Tencent press release](https://www.tencent.com/en-us/articles/2202235.html)
- Tencent's community license page covers open-source terms and enterprise commercial use. — [Tencent Cloud techpedia](https://www.tencentcloud.com/techpedia/148273?lang=en)

Hunyuan3D-Part (open source, Sep 2025):
- P3-SAM is described as the "industry's first native 3D part segmentation model". It is point-promptable, trained on 3.7M shapes, and outputs semantic labels and boxes.
- X-Part does controllable part generation/completion. The open release is a light version; the full version is in Hunyuan3D Studio.
- A ComfyUI wrapper exists (PozzettiAndrea/ComfyUI-Hunyuan3D-Part). — [Hunyuan3D-Part GitHub](https://github.com/Tencent-Hunyuan/Hunyuan3D-Part), [Tencent Hy on X](https://x.com/TencentHunyuan/status/1971491034044694798), [ComfyUI wrapper](https://github.com/PozzettiAndrea/ComfyUI-Hunyuan3D-Part)
- Hunyuan3D Studio is described as an end-to-end pipeline for game-ready assets. — [arXiv 2509.12815](https://arxiv.org/pdf/2509.12815) (blocked, snippet)

Tripo (cloud):
- Offers text-, image- and multi-view-to-3D in two geometry modes: "High Detail" and "Smart Mesh". Smart Mesh outputs quads in the 500–50K range with clean edge flow, intended to drop into an engine without retopology. — [Meshy blog: Hi3D vs Meshy vs Tripo](https://www.meshy.ai/blog/hi3d-vs-meshy-vs-tripo) (competitor-written)
- Tripo 3.1 has smart segmentation (automatic splitting into components) plus auto-rig and animation. — [3DAIStudio comparison](https://www.3daistudio.com/blog/hitem3d-vs-meshy-vs-tripo-comparison), [Tripo segmentation docs](https://developers.tripo3d.ai/en/docs/mesh-segment) (blocked)
- Smart Mesh P1.0 is a native 3D diffusion architecture for production low-poly assets, generated in as little as 2 seconds. — [Barchart press release](https://www.barchart.com/story/news/936837/tripo-ai-introduces-smart-mesh-p1-0-defining-a-new-phase-for-ai-3d-production)
- Tripo Studio automates modeling, texturing and rigging. — [Technology.org, Jun 2026](https://www.technology.org/2026/06/01/the-future-of-3d-creation-how-tripo-studios-ai-automates-modeling-texturing-and-rigging/)

Meshy (cloud):
- Meshy 6 has a quad-topology option and A-pose/T-pose control, accepts 1–4 reference views, and produces watertight meshes.
- In a vendor-cited blind test by 1,331 artists, Meshy-6 was preferred over Tripo 3.1 by 63.8%.
- A Meshy 7 / 7.1 line exists as of 2026. — [Meshy vs Rodin page](https://www.meshy.ai/compare/meshy-vs-rodin) (vendor; blocked, snippet only), [Meshy help: 7.1 vs 6 vs 6 Lite](https://help.meshy.ai/en/articles/15697060-meshy-7-vs-meshy-6-vs-meshy-6-lite-which-ai-model-should-you-use) (blocked)

Rodin (Hyper3D, cloud):
- Rodin Gen-2 / Gen-2.5 leads on geometry density and polygon ceiling; high-poly quad meshes require the Business plan.
- Meshy is said to win on texture detail. — [Meshy vs Rodin](https://www.meshy.ai/compare/meshy-vs-rodin) (vendor bias), [StraySpark comparison Apr 2026](https://www.strayspark.studio/blog/generative-3d-tools-comparison-meshy-rodin-tripo-csm-2026)

Part-level generators:
- PartPacker (NVIDIA): part-level 3D from one image via "dual volume packing"; code at NVlabs/PartPacker. — [GitHub](https://github.com/NVlabs/PartPacker), [project page](https://research.nvidia.com/labs/cosmos-lab/partpacker/)
- PartCrafter: 2–16 separate, semantically meaningful meshes from one RGB image. — [arXiv 2506.05573](https://arxiv.org/html/2506.05573v1), [VoxelMatters](https://www.voxelmatters.com/impressive-partcrafter-tool-generates-multiple-parts-and-objects-from-a-single-image/)
- HoloPart: amodal segmentation and completion. It segments an existing mesh (SAMesh/SAMPart3D), then completes each part with a DiT. In principle this could recover the occluded body under clothing. — [awesome-3dv-weekly](https://github.com/FishWoWater/awesome-3dv-weekly)

Other geometry models:
- Direct3D-S2: sparse-volume, 1024³, MIT license. — [Direct3D-S2 project](https://nju-3dv.github.io/projects/Direct3D-S2/)
- Sparc3D: about 8GB VRAM for 1024³, but "no longer fully open source despite initial promises". — [Creative Shrimp](https://www.creativeshrimp.com/sparc3d-ai-mesh-from-image.html), [Sparc3D issue #4](https://github.com/lizhihao6/Sparc3D/issues/4)
- Hi3DGen: built on TRELLIS, uses normal maps for high geometric detail. — [awesome-3dv-weekly](https://github.com/FishWoWater/awesome-3dv-weekly)
- UltraShape 1.0 (Dec 2025): geometric refinement. — [arXiv 2512.21185](https://arxiv.org/html/2512.21185)

SAM 3D Objects (Meta):
- Released Nov 19, 2025. Single-image reconstruction of objects (geometry, texture, layout).
- Licensed under the custom SAM License. Code and checkpoints are at facebookresearch/sam-3d-objects. — [Roboflow blog](https://blog.roboflow.com/sam-3d/), [Roboflow licensing page](https://playground.roboflow.com/models/meta/sam-3d-objects)

### Inferences
- Best clothing separation with zero training today:
  - Run StdGEN locally. It is anime-native, outputs separate body/clothes/hair, and is Apache-2.0.
  - It is a 2024–25-era model, so mesh resolution and textures are modest (LRM-based, multi-view-projected colors).
  - Treat its output as a high-quality blockout/proxy that you then retopologize and texture. It is not final HD.
- Best raw HD fidelity (fused mesh):
  - Local: TRELLIS.2 (MIT, 24GB, Linux; Windows likely via WSL2 or community ports, unverified) or Hunyuan3D 2.1 (works on Windows, 29GB for both stages in one process, or run shape and paint separately).
  - Cloud: Hunyuan 3.1, Meshy 6/7, Tripo 3.x, Rodin Gen-2.5.
  - All produce PBR-style textures. Lighting and shading in the input image is often baked into the base color, so for anime you would repaint to flat colors anyway.
- Most practical cloud route for a solo creator who wants separation plus rig in one place: Tripo (segmentation + Smart Mesh quads + auto-rig) or Meshy (quad + A/T-pose + rig). Expect segmentation to split into parts (hair, head, torso, limbs, skirt) rather than produce a correct closed body under the clothes. HoloPart or X-Part style completion would be needed to fill the occluded body.
- Watch PBR outputs for anime. A "metallic/roughness" PBR set is not the target. For UE5 you will usually want a toon/cel material (VRM4U's MToon or a custom cel shader) with flat albedo.
- License summary for personal/portfolio use:
  - MIT: TRELLIS.2, Direct3D-S2.
  - Apache-2.0: StdGEN.
  - Tencent Hunyuan Community License: Hunyuan3D 2.1, Hunyuan3D-Part. Prior knowledge, not re-verified: it excludes the EU, UK and South Korea, so check if the user lives there.
  - Custom SAM License: SAM 3D.
  - Cloud vendors: per their ToS; free tiers often restrict commercial use and privacy (not verified).

### Gaps
- No primary benchmark found comparing these generators specifically on anime characters.
- Polycount and texture resolution numbers for Tripo 3.x, Meshy 7 and Rodin Gen-2.5 could not be confirmed, because vendor docs were blocked.
- StdGEN VRAM is not documented. StdGEN++ code/weights availability is unconfirmed (no repo found).
- No details fetched on PartCrafter/PartPacker license or VRAM, or on how they behave on clothed humans. They target objects; human/anime clothing suitability is unverified.
- Hunyuan3D 3.x pricing was not verified.

## Clean-up: retopology, UVs, texture repaint to flat anime colors, separating garments, fixing hands/face/hair

### Takeaway
Treat any AI mesh as a sculpt or base reference, not a final asset. For hero characters the standard advice is a full retopology pass before rigging. Mixamo and auto-riggers work poorly on dense or irregular AI topology. Tool-side "smart topology" (Tripo Smart Mesh, Meshy quad) reduces but does not remove this work.

### Cited Findings
- Meshy's own auto-rig guide:
  - If the mesh is messy after remeshing, retopologize in Blender.
  - For hero-quality characters, treat the AI output as a base mesh and do a full retopology pass.
  - High-poly or irregular meshes give poor auto-rig results, so retopologize before auto-rigging. — [Meshy auto-rigging tutorial](https://www.meshy.ai/tutorials/character-auto-rigging-workflow) (vendor)
- Tripo Smart Mesh claims engine-ready quads (500–50K) with no retopology. — [Meshy comparison blog](https://www.meshy.ai/blog/hi3d-vs-meshy-vs-tripo) (competitor-written, so the claim is reported, not endorsed)
- Commercial anime base meshes with clean topology are sold on Gumroad. Examples: male base mesh (85 variations), female base mesh (106 variations), anime outfit packs with welded/unwelded meshes and semi-automated low/high-poly retopology versions. — [minimoku anime female basemesh](https://minimoku.gumroad.com/l/anime_female_basemesh), [xxerbexx Blender anime character](https://xxerbexx.gumroad.com/l/blender_anime_character), [soheilkianfar outfit pack](https://soheilkianfar.gumroad.com/l/WarriorOutfit-TojiAndMegumiFushiguro)
- TRELLIS.2 exports GLB in opaque mode by default; transparency needs manual material setup. — [TRELLIS.2 GitHub](https://github.com/microsoft/TRELLIS.2)

### Inferences
These are practitioner-standard steps from prior knowledge, not verified via fetched sources this session.
- Retopology tools:
  - Quad Remesher (paid Blender/Maya add-on, by Exoside) is the fastest automatic quad route.
  - Instant Meshes (free) is the free alternative.
  - Blender's built-in Voxel/Quadriflow remesh is fine for blockouts.
  - For deforming areas (shoulders, elbows, knees, face loops), manual retopology with RetopoFlow or snapping is best.
- Anime faces: AI generators usually blur eyes and mouths. The practical approach:
  - Replace the head with a VRoid head or a base-mesh anime head.
  - Paint eyes and mouth as texture or decal planes (classic anime technique).
  - Add custom normals or a face-shadow SDF in the UE5 toon material.
- Anime hair: rebuild as card/ribbon strips (VRoid hair guides, or Blender curves -> mesh) rather than keeping a blobby AI hair volume. This also gives clean spring-bone chains.
- Textures:
  - Re-project or bake the AI color onto the new UVs (Blender bake, selected-to-active).
  - Flatten to cel colors with posterize/paint-over, or project the clean turnaround images onto the mesh (Blender projection painting, or StableProjectorz-type tools).
  - Upscale with a 4x anime ESRGAN model (prior knowledge).
- Separating garments from a fused mesh in Blender:
  1. Select faces by region or material, then Separate (P).
  2. Duplicate the shell and Solidify it to give the garment thickness.
  3. Shrink or replace the body under the garment with a base body.
  4. Delete hidden body faces to avoid poke-through.
  5. Transfer weights from the body to the garments (Data Transfer modifier).

### Gaps
- No fetched source gave 2026 versions or prices for Quad Remesher, Instant Meshes or RetopoFlow, or Blender version specifics.
- No fetched community post (Reddit) on AI-mesh garment separation; the reddit search returned nothing useful.

## Alternative "construct" routes (no AI training): VRoid Studio, Marvelous Designer, Blender base mesh, combined routes

### Takeaway
The most reliable route to separate, correctly deforming clothing in UE5 has these pieces:
- Body and face/hair: VRoid Studio (free, anime-native, now has a dress-up/XWear system and custom accessory items).
- Hero outfits: Marvelous Designer 2026.x, sewn from the AI-generated turnaround and outfit sheets. It imports VRM/glTF avatars, has a Toon Shader, and has a LiveSync plugin for Unreal.
- UE5 import: VRM4U (MIT).
AI image-to-3D is best used here only for accessories and props (weapons, bags, shoes, hard-surface armor).

### Cited Findings
VRoid Studio:
- v2.0.0 (Nov 2024) officially released the dress-up feature and the XWear format:
  - Auto-fitting plus manual tools to fit clothing to different body types.
  - Export as VRM 1.0 or to VRChat. — [VRoid news v2.0.0](https://vroid.com/en/news/26gn98sTuPJQ53LxDQRyFg), [Steam announcement](https://steamcommunity.com/games/1486350/announcements/detail/4446837338576781356)
- XWear is costume/accessory data supported in VRoid Studio's dress-up and in Unity (via VCC + XWear package). — [VRoid help: What is XWear](https://vroid.pixiv.help/hc/en-us/articles/39513229598233-What-is-XWear)
- Release cadence 2025–26:
  - v2.5.0, Oct 30 2025: KR/zh languages.
  - v2.6.0, Nov 27 2025: 17 new hair presets.
  - v2.7.0, Dec 4 2025: Closet feature for frequently used base models, costumes and accessories.
  - v2.9.0, Jan 15 2026: presets.
  - v2.10.0, Jan 29 2026: custom items for accessories.
  - v2.11.0, Feb 26 2026: new presets and a Sticker tool.
  - v2.8.0 also exists. — [VRoid release notes index](https://vroid.pixiv.help/hc/en-us/sections/900000861806--Release-Notes), [v2.7.0](https://vroid.pixiv.help/hc/en-us/articles/52924741495065--v2-7-0-Added-the-Closet-feature-Dec-4th-2025), [v2.10.0](https://vroid.pixiv.help/hc/en-us/articles/54649904917657--v2-10-0-Custom-items-for-accessories-Jan-29th-2026), [v2.11.0](https://vroid.pixiv.help/hc/en-us/articles/55439895790745--v2-11-0-Added-new-presets-Sticker-tool-and-more-Feb-26-2026)
- A VRM4U GitHub issue asks about XWear/changeable clothes in Unreal. XWear is not a native UE path, so outfits arrive via VRM export or FBX. — [VRM4U issue #592](https://github.com/ruyo/VRM4U/issues/592)
- Clothing packs for VRoid are sold on Gumroad. — [cikanindya casual outfits](https://cikanindya.gumroad.com/l/CasualOutfitsforVirtualStreamers)
- StdGEN++'s dataset is 10,811 VRoid-Hub characters, so VRoid topology is the "native" format these research models are trained on. — [StdGEN++ arXiv](https://arxiv.org/pdf/2601.07660)

VRM4U (Unreal importer):
- MIT license, by ruyo.
- Imports VRM via drag-and-drop with MToon material recreation (shadow colors, outlines, MatCap), bones, morph targets, blendshape groups and spring bones with physics.
- Auto-generates a Humanoid rig for retargeting to the UE mannequin. Supports runtime loading.
- 2026 updates added formal UE 5.8 support, material batch generation and MetaHuman animation curves.
- The README lists 5.5/5.6/5.7 as "preview"; the wording is inconsistent. — [VRM4U GitHub](https://github.com/ruyo/VRM4U)

Marvelous Designer:
- 2026.0 (Apr 2026):
  - 3D Pencil: draw pattern outlines on the avatar.
  - Lacing Tool.
  - Toon Shader with outlines and rim light.
  - glTF and VRM avatar import, with blendshape support on imported avatars to avoid penetration.
  - Export of specific animation ranges to FBX/glTF.
  - LiveSync plugin and USD simulation-data export for Unreal; MetaHuman fitting round-trip. — [The Rookies](https://www.therookies.co/blog/headlines/marvelous-designer-2026), [Digital Production](https://digitalproduction.com/2026/04/15/marvelous-designer-2026-0-adds-3d-pencil-and-lacing/), [CG Channel 2026.0](https://www.cgchannel.com/2026/04/clo-virtual-fashion-releases-marvelous-designer-2026-0/) (blocked; snippet)
- 2026.1 was released Aug 2026. — [MD 2026.1 feature list](https://support.marvelousdesigner.com/hc/en-us/articles/59743730927129-Marvelous-Designer-2026-1-New-Feature-List), [CG Channel 2026.1](https://www.cgchannel.com/2026/08/clo-virtual-fashion-releases-marvelous-designer-2026-1/) (contents not fetched; blocked)
- An official MD + Unreal workflow guide exists. — [MD support: workflow with Unreal](https://support.marvelousdesigner.com/hc/en-us/articles/47358145573401--Tips-Tricks-Discover-Better-Workflow-with-Marvelous-Designer-and-Unreal-Engine), [CLO LiveSync](https://connect.clo-set.com/livesync)

### Inferences
- Combined route A ("VRoid + MD", recommended for best deformation and separate clothing):
  1. Build body, face, eyes and hair in VRoid from the turnaround. Use a near-nude or base outfit.
  2. Export VRM 1.0.
  3. Import the VRM avatar into Marvelous Designer 2026.x.
  4. Sew the outfit from the AI outfit sheet. Use 3D Pencil to trace silhouettes.
  5. Use MD's quad remesh/retopology and UV, then export FBX.
  6. In Blender, skin the garments to the VRoid armature (weight transfer).
  7. Bring the result into UE5 via VRM4U (body), with garments as separate skeletal meshes sharing the skeleton, or via FBX. Use Chaos Cloth or spring bones for skirts and coats.
  - MD's Toon Shader lets you preview the anime look before export.
- Combined route B (fastest, lower fidelity): VRoid alone with custom textures. Paint the outfit onto VRoid's built-in clothing templates, use the v2.11 Sticker tool and custom accessory items (v2.10), then export VRM -> VRM4U. Clothing is texture-painted on body-fitted meshes rather than truly separate garments, except XWear items.
- Combined route C (AI-assisted): generate accessories and hard-surface parts (weapons, shoes, armor plates, bags) with TRELLIS.2, Hunyuan 2.1 or Tripo. Decimate or retopologize, then attach to bones.
- MD cost: subscription (prior knowledge; personal plans exist, roughly tens of USD/month; not verified this session).

### Gaps
- The 2026.1 feature list (e.g., any new auto-retopo/UV or anime features) was not fetched.
- MD 2026 pricing and system requirements were not verified.
- No source confirmed whether VRoid has added FBX export or a direct Unreal path in 2026 (none seen in the release-note titles).
- CLO (fashion-industry sibling) was not researched; MD is the game/VFX product.

## Which route gives the BEST HD result with the EASIEST setup, and realistic time per character?

### Takeaway
For a solo creator who wants HD, separate and correctly deforming clothing in UE5, the best quality-to-effort route is a hybrid with no model training anywhere:
1. AI reference prep (Nano Banana Pro or Qwen-Image-Edit-2511).
2. VRoid body, face and hair.
3. Marvelous Designer clothing.
4. Optional AI image-to-3D (TRELLIS.2 / Hunyuan / Tripo) for accessories only.
5. VRM4U into UE5.
A fully AI route (StdGEN for separation, or Tripo/Meshy with segmentation and auto-rig) is faster to a first result but needs heavy cleanup to reach HD and correct deformation.

### Cited Findings
- AI image-to-3D outputs for hero characters still need a full retopology pass. — [Meshy auto-rig guide](https://www.meshy.ai/tutorials/character-auto-rigging-workflow)
- Most state-of-the-art generators produce "monolithic" meshes with skin, hair and clothes fused, which the StdGEN++ authors call "practically useless for professional gaming and animation pipelines". — [StdGEN++ arXiv](https://arxiv.org/pdf/2601.07660) (snippet)
- VRoid dress-up has auto-fitting, and VRM 1.0 export is the supported interchange format. — [VRoid v2.0.0 news](https://vroid.com/en/news/26gn98sTuPJQ53LxDQRyFg)
- VRM4U brings VRM into UE with MToon, spring bones and mannequin retargeting. — [VRM4U GitHub](https://github.com/ruyo/VRM4U)
- MD 2026 imports VRM avatars and has a toon preview plus Unreal LiveSync. — [The Rookies](https://www.therookies.co/blog/headlines/marvelous-designer-2026)

### Inferences
- Route ranking for this user (24GB GPU, personal use, HD, separate clothes):
  1. Hybrid VRoid + MD (+ AI accessories). Highest deformation quality and truly separate garments; setup is installers only.
  2. StdGEN local, then Blender retopo, re-texture and rig. Anime-native separation, but low-res geometry and textures that need rework; medium install effort (conda, Python 3.9, PyTorch 2.1).
  3. Cloud Tripo/Meshy, then segmentation, Blender separation, retopo and rig. Fast, but clothing is fused and hidden body parts are missing.
  4. Local TRELLIS.2 / Hunyuan 2.1 for a high-detail fused sculpt, used as a reference to model over. Best as a "sculpt reference", not a final asset.
- Realistic time per character. These are estimates from practitioner experience, not sourced; treat them as rough.
  - Step 0 reference sheets: 0.5–2 h.
  - VRoid body, hair and face: 2–6 h.
  - MD outfit, depending on complexity: 4–12 h.
  - Retopo, UV and skinning in Blender: 4–10 h.
  - UE5 import, toon material and physics: 2–4 h.
  - Total: roughly 2–5 working days for a portfolio-quality character.
  - An AI-only route can produce a rough rigged result in under 1 h, but HD cleanup usually adds back days.

### Gaps
- No sourced, measured time-per-character data was found for any route.
- No head-to-head anime-character tests comparing these routes in UE5 were found.

## Known pitfalls (fused clothing, deformation topology, baked lighting, collapsing anime faces)

### Takeaway
Problems with AI meshes:
- Fused body and clothes.
- Missing occluded geometry.
- Triangulated or irregular topology that deforms badly.
- Baked lighting in textures.
- Mushy faces, eyes and hands.
Mitigations: semantic-decomposed generators (StdGEN), part segmentation and completion (Tripo segmentation, P3-SAM/X-Part, HoloPart), retopology, flat-lit input references, and replacing the head and hair with constructed (VRoid) parts.

### Cited Findings
- Fused "monolithic" meshes are the core problem StdGEN/StdGEN++ were built to solve. — [StdGEN++ arXiv](https://arxiv.org/pdf/2601.07660)
- StdGEN fails more on half-body inputs and non-anime styles, depends on background removal, and hair quality depends on mask accuracy. — [StdGEN GitHub](https://github.com/hyz317/StdGEN)
- Irregular, high-poly AI topology gives poor auto-rig results; retopologize first. — [Meshy auto-rig guide](https://www.meshy.ai/tutorials/character-auto-rigging-workflow)
- TRELLIS.2 complex topologies vary in quality, and GLB transparency needs manual setup. — [TRELLIS.2 GitHub](https://github.com/microsoft/TRELLIS.2)
- HoloPart addresses the "missing hidden geometry" problem: it segments, then completes each part amodally. — [awesome-3dv-weekly](https://github.com/FishWoWater/awesome-3dv-weekly)
- Sparc3D was promoted as open source but is no longer fully open, a licensing/availability pitfall. — [Sparc3D issue #4](https://github.com/lizhihao6/Sparc3D/issues/4), [Creative Shrimp](https://www.creativeshrimp.com/sparc3d-ai-mesh-from-image.html)
- Hunyuan 2.5/3.x are cloud only despite the "open" Hunyuan brand, so local users are capped at 2.1. — [wireflow.ai](https://www.wireflow.ai/blog/best-hunyuan3d-v3-tools-in-2026)

### Inferences
- Baked lighting: PBR generators (TRELLIS.2, Hunyuan 2.1 paint) try to de-light, but anime cel shadows drawn in the input usually end up in the albedo. Prepare references with flat, even lighting and no cast shadows, then repaint to flat colors before applying a UE5 toon shader.
- Anime faces collapse because single-image generators reconstruct the face as geometry rather than texture decals. StdGEN++ added a dedicated Facial LoRA branch for this reason, per its abstract. The practical fix is a VRoid or base-mesh head with painted eyes.
- Clothing poke-through in UE5: delete body faces under tight garments, or use material masks. Give skirts their own bone chains (VRM spring bones via VRM4U, or UE Chaos Cloth / Kawaii Physics, prior knowledge).
- License pitfall: Tencent Hunyuan Community License territory restrictions (prior knowledge: EU/UK/South Korea excluded). Free tiers of cloud tools may make outputs public or restrict commercial use. That is fine for personal/portfolio use, but note it.

### Gaps
- No fetched Reddit/80.lv practitioner reports quantifying these failure rates.
- Exact Hunyuan license territory wording was not re-fetched in this session.
