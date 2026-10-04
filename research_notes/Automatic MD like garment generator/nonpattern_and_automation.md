# Non-pattern garment generation fitted to an existing custom 3D (anime) body + automation glue for UE5

Scope: ways to get a garment fitted to a user's existing unclothed anime body from images/text without a sewing-pattern + cloth-simulation ("automatic Marvelous Designer") pipeline, plus scripts and tools that make the result game-ready. Assumes one 24GB+ NVIDIA GPU, personal use, pretrained weights only, UE5 as the target engine. Current as of 2026-10-04.

**Access notes (for the report writer):** These were blocked by the research proxy: arxiv.org (abs, html and export API), huggingface.co (papers, model cards), dev.epicgames.com, support.marvelousdesigner.com and planaria.github.io (Alterith docs). The GitHub search API was also blocked ("sessions are bound to their configured repositories"). raw.githubusercontent.com worked, so every README and LICENSE claim below was read directly from the repo files. Claims about arXiv papers, Epic docs and MD docs come from search-engine snippets only and are marked **(snippet-only)**.

---

## 1. Template-deformation and body-conditioned generators (image/text → garment fitted to a body)

### Takeaway
Only a few methods in this family have usable code. **Garment3DGen** (deforms a garment template toward an image-to-3D target, CC BY-NC) and **GarmentDreamer** (deforms a template from text using 3DGS, CC BY-NC) are the practical ones. Both keep the template's clean, simulatable topology, and both are fine for personal use. Neither fits a garment to a *user-supplied* body on its own: you supply or fit the template to your body first. **StdGEN** (CVPR 2025, Apache-2.0, weights released) is the only open method trained on anime (VRoid) data that outputs a separate clothing layer. However, it generates its own body, not yours. Most other papers (TELA, HumanCoser, WordRobe, BAG, Tailor, StdGEN++, GarmentCrafter) have no released code, or depend on SMPL/SMPL-X.

### Cited Findings
**Garment3DGen (Meta Reality Labs; arXiv 2403.18816; accepted to 3DV 2025)**
- Repo: github.com/nsarafianos/Garment3DGen. TL;DR: "stylizes the geometry and textures of real and fantastical garments that we can fit on top of parametric bodies and simulate." Tested on Windows 10, Python 3.8, CUDA 11.8, torch 2.2.2. Depends on nvdiffrast, PyTorch3D and Fashion-CLIP. Run with `python main.py`. — [README](https://raw.githubusercontent.com/nsarafianos/Garment3DGen/main/README.md)
- Workflow: put source garment templates in `./meshes/` and target geometry in `./meshes_target/`. The README says targets are obtained "very easily by passing an RGB image to InstantMesh". The paper originally used Wonder3D + Instant-NSR. Guidance: targets should have no intersections, ideally with stretched arms, and "source and target geometry [should] be reasonably close… Going from a skirt to a shirt won't work well." — [README](https://raw.githubusercontent.com/nsarafianos/Garment3DGen/main/README.md)
- Built on TextDeformer and Neural Jacobian Fields, so it is a per-asset optimization with no training. — [README](https://raw.githubusercontent.com/nsarafianos/Garment3DGen/main/README.md)
- License: Creative Commons Attribution-NonCommercial 4.0 (LICENSE.md). — [LICENSE](https://github.com/nsarafianos/Garment3DGen)
- Paper claims: outputs "preserve the structure and topology of the input geometry, contain holes in the neck/arms/waist areas such that they can be fit to bodies and are of good mesh quality to be physically simulated". It also generates high-resolution UV texture maps faithful to the input image. Authors: Sarafianos, Stuyck, Xiang, Li, Popovic, Ranjan. 3DV 2025. — [project page / search snippet](https://nsarafianos.github.io/garment3dgen) **(snippet-only)**

**GarmentDreamer (3DV 2025; arXiv 2405.12420; UCLA/Utah/Style3D-related authors incl. Chenfanfu Jiang, Huamin Wang)**
- Repo: github.com/boqian-li/GarmentDreamer. Requires Ubuntu 20.04, CUDA 11.8, torch 2.3.1, PyTorch3D and 3DGS rasterizer submodules. Usage: `python launch_garmentdreamer.py --template_path template.obj --prompt "a {style} {garment type} made of {color} {material}"`. Single GPU only. — [README](https://raw.githubusercontent.com/boqian-li/GarmentDreamer/main/README.md)
- Template meshes come from a Google Drive link, or you can supply your own, but they must match the orientation of the provided ones. "If you got not good results, please try another mesh template, it's very likely that the mesh template is not suitable for your prompt." The AutoEncoder_dgcnn + Garment_Diffusion pretrained models are released. — [README](https://raw.githubusercontent.com/boqian-li/GarmentDreamer/main/README.md)
- License: CC BY-NC 4.0. — [LICENSE](https://github.com/boqian-li/GarmentDreamer)
- Text only (SDS-style 3DGS guidance), with no image input. An open TODO reads "Improve Garment_3DGS to obtain more significant deformation". — [README](https://raw.githubusercontent.com/boqian-li/GarmentDreamer/main/README.md)

**ClotheDreamer (arXiv 2406.16815; journal version in Applied Intelligence 2025)**
- Text → garment as 3D Gaussians via "Disentangled Clothe Gaussian Splatting" (DCGS), which freezes body Gaussians. It uses bidirectional SDS and a pruning strategy for loose clothing, and claims support for try-on and physically accurate animation. — [arXiv snippet](https://arxiv.org/abs/2406.16815) **(snippet-only)**; [Springer](https://link.springer.com/article/10.1007/s10489-025-06596-x)
- Repo github.com/ggxxii/clothedreamer has an MIT LICENSE, but the README fetched on 2026-10-04 contains only the title, authors and method figure, with no install or run instructions. Code usability is unverified. — [README](https://raw.githubusercontent.com/ggxxii/clothedreamer/main/README.md)

**DressCode (SIGGRAPH 2024 / TOG)**: pattern-based (SewingGPT generates sewing patterns from text, plus Stable Diffusion tile PBR textures). It falls on the pattern side, so it is listed here only for completeness. Repo github.com/ihe-kaii/DressCode; no LICENSE file found at the repo root. — [README](https://raw.githubusercontent.com/ihe-kaii/DressCode/main/README.md)

**StdGEN (CVPR 2025) / StdGEN++ (arXiv 2601.07660, Jan 2026)**
- StdGEN repo github.com/hyz317/StdGEN. Inference code, dataset lists and pretrained checkpoints were released 2025-03-04, with an HF Gradio demo on 2025-03-17. Pipeline: reference image → A-pose image (`infer_canonicalize.py`) → decomposed multi-view (`infer_multiview.py`, has a `--low_vram` flag) → S-LRM reconstruction → multi-layer refinement (`infer_refine.py`). Outputs body, hair and clothing as separate layers. Uses `rm_anime_bg` for anime background removal. Trained on Anime3D++, rendered from VRoid data. Full-body input is needed. — [README](https://raw.githubusercontent.com/hyz317/StdGEN/main/README.md)
- Code license: Apache-2.0 (repo LICENSE). The license of the HF weights was not verifiable because huggingface.co was blocked. — [LICENSE](https://github.com/hyz317/StdGEN)
- StdGEN++ claims body, clothing and hair separated as independent layers "support[ing] immediate rigging, physics simulation". It generates clothing "as a standalone, internally hollow mesh". Built on a Dual-Branch S-LRM. — [arXiv snippet](https://arxiv.org/pdf/2601.07660) **(snippet-only)**. No StdGEN++ code release was found; the search showed only the original StdGEN repo. — [search](https://github.com/hyz317)

**TELA / HumanCoser / LayerAvatar / WordRobe / SimAvatar (text → layered clothed human)**
- TELA (arXiv 2404.16748) repo README says only "Coming soon...". No code. — [README](https://raw.githubusercontent.com/DongJT1996/TELA/main/README.md)
- HumanCoser (arXiv 2408.11357): NeRF-based, generates the minimally clothed body and then layer-wise clothes. No code repo found. — [arXiv snippet](https://arxiv.org/pdf/2408.11357) **(snippet-only)**
- LayerAvatar (ICCV 2025 Highlight, "Disentangled Clothed Avatar Generation with Layered Representation", arXiv 2501.04631): code is released (CUDA 11.7, PyTorch 1.13, MMCV/MMGeneration). It requires SMPL-X and FLAME registrations plus component templates from Google Drive. No LICENSE file was found. — [README](https://raw.githubusercontent.com/olivia23333/LayerAvatar/main/README.md)
- WordRobe (arXiv 2403.17541) is listed in the Awesome-3D-Garments list with a paper link only. — [Awesome-3D-Garments](https://github.com/Shanthika/Awesome-3D-Garments)
- SimAvatar (arXiv 2412.09545) claims to be "the first work that generates fully simulation-ready 3D avatars with separate layers for the body, garment, and hair". — [arXiv snippet](https://arxiv.org/html/2412.09545v1) **(snippet-only)**

**Body-conditioned generation (closest to "fit to MY body")**
- **BAG: Body-Aligned 3D Wearable Asset Generation** (arXiv 2501.16177; IEEE TVCG 2026). A ControlNet conditions a multiview diffusion model on renders of the target body, with XYZ-coordinate maps in canonical space. A native 3D diffusion model then lifts the views to 3D. The similarity transform is recovered from multiview silhouettes, and physics simulation removes body–asset penetration. "Output 3D wearable assets that can be automatically dressed on given 3D human bodies." No code repo was found. — [project page snippet](https://bag-3d.github.io/) **(snippet-only)**
- **Tailor** (arXiv 2503.12052; Sun et al., Tsinghua/Yong-Jin Liu group). An LLM parses text into a parametric avatar plus matched garment templates. "Topology-preserving deformation with novel geometric losses to generate body-aligned garments". Multi-view diffusion texturing. Standard CG polygon meshes. No code was found. — [project page snippet](https://human-tailor.github.io/) **(snippet-only)**
- **GarmentCrafter** (arXiv 2503.08678; reported as 3DV 2026 oral / best paper candidate): single image → progressive novel-view synthesis (RGB + depth) → garment mesh, with 2D edits propagated to 3D. The repo github.com/humansensinglab/garment-crafter README contains only "# GarmentCrafter" as of 2026-10-04, so no usable code. — [README](https://raw.githubusercontent.com/humansensinglab/garment-crafter/main/README.md); [search snippet](https://humansensinglab.github.io/garment-crafter/) **(snippet-only)**
- **Dress-1-to-3** (ACM TOG 2025; arXiv 2502.03449) and **Image2Garment** (arXiv 2601.09658) are simulation-ready but go through sewing patterns or physical parameters. No code repo was found for Dress-1-to-3, only the project-page repo. An Image2Garment course proposal says code was planned "upon acceptance to CVPR26". — [Dress-1-to-3](https://dress-1-to-3.github.io/); [Image2Garment proposal](https://web.stanford.edu/class/ee367/Winter2026/proposals/proposal_can.pdf)
- **GarmageNet** (Style3D; SIGGRAPH Asia 2025) code exists but is pattern-oriented and CC BY-NC-ND 4.0. — [LICENSE](https://github.com/Style3D/garmagenet-impl)

**SAM 3D Body / SAM 3D Objects (Meta, Nov 2025)**
- SAM 3D Body is single-image human mesh recovery onto the **Momentum Human Rig (MHR)** parametric body. It outputs a body, with no garments or clothing layer. Checkpoints (DINOv3-H+ 840M, ViT-H 631M) are on HF behind access requests. "SAM License" dated Nov 19, 2025. — [README](https://raw.githubusercontent.com/facebookresearch/sam-3d-body/main/README.md)
- SAM 3D Objects is the sibling object-reconstruction model under the same SAM License. — [repo](https://github.com/facebookresearch/sam-3d-objects)

### Inferences
- In practice, **Garment3DGen's recipe can be built around your own body.** (1) Fit a generic template (T-shirt, skirt, coat) once to your body: shrinkwrap/cloth-fit (section 3), or MD with one manual drape. (2) Generate the target shell from the anime turnaround with TRELLIS.2/Hunyuan3D (section 2). (3) Let Garment3DGen's NJF deformation pull the clean template toward the target. The result keeps the template's topology and UVs, so it stays sim-ready and outline-friendly. This is inference: Garment3DGen does not check body penetration itself, so a collision/offset pass against your body is still needed.
- StdGEN is the best *anime-domain* generator, but its body is not yours. Use its clothing layer as a "target shell" or "reference garment", then retarget it onto your body with cloth-fit, shrinkwrap or Robust Weight Transfer.
- All template-deformation methods are limited by the template library: a garment far from any template, such as an asymmetric anime coat with capes, will fail. That is where pattern+sim or manual modeling wins.
- VRAM (inference): Garment3DGen and GarmentDreamer are per-asset optimizations (nvdiffrast/3DGS + CLIP/SD) that should fit in 24GB. Exact numbers are not published in either README.

### Gaps
- Exact VRAM/runtime for Garment3DGen, GarmentDreamer and StdGEN on a 24GB card: not stated in the READMEs. Papers were inaccessible (arXiv blocked).
- StdGEN HF weight license, and whether StdGEN++ code or weights will be released: unverified.
- BAG, Tailor, HumanCoser and WordRobe code availability: no repos found by search. Treat as unavailable.
- ClotheDreamer: the MIT-licensed repo appears to contain no runnable code per its README. This is unconfirmed without browsing the repo tree (GitHub API blocked).

---

## 2. Image-to-3D generators + automatic fitting onto a custom body

### Takeaway
**TRELLIS.2** (Microsoft, MIT, 4B, needs ≥24GB) explicitly supports **open surfaces "e.g., clothing"** and non-manifold geometry. It is the best open generator for producing a single garment shell from a cleaned outfit image. **Hunyuan3D-2.1** (10GB shape / 21GB texture) gives closed shells plus a PBR paint model. Part segmentation (Hunyuan3D-Part P3-SAM/X-Part, Tripo Segmentation, HoloPart) can split a generated clothed character into pieces. None of these generators fits the output to *your* body: that step is always a separate registration/retargeting step (section 3), followed by retopology.

### Cited Findings
- **TRELLIS.2** (github.com/microsoft/TRELLIS.2): 4B-param image-to-3D using a "field-free" sparse voxel structure, **O-Voxel**. "Arbitrary Topology Handling: ✅ Open Surfaces (e.g., clothing, leaves) ✅ Non-manifold Geometry ✅ Internal Enclosed Structures". It outputs PBR attributes (base color, roughness, metallic, opacity). It takes ~60 s at 1536³ (35 s shape + 25 s material, tested on H100). "An NVIDIA GPU with at least 24GB of memory is necessary." Uses flash-attn by default, with an xformers fallback. Code and model are MIT. Checkpoints, a shape-conditioned texture generation inference script and training code are all released. — [README](https://raw.githubusercontent.com/microsoft/TRELLIS.2/main/README.md)
- **Hunyuan3D-2.1** (Tencent, released Jun 13, 2025): full weights and training code. "It takes 10 GB VRAM for shape generation, 21GB for texture generation and 29GB for shape and texture generation in total." It has a `--low_vram_mode`. Paint model Hunyuan3D-Paint-v2-1 (2B) is PBR and can be called standalone on any mesh via `Hunyuan3DPaintPipeline(mesh_path, image_path=…)`. — [README](https://raw.githubusercontent.com/Tencent-Hunyuan/Hunyuan3D-2.1/main/README.md)
- **Hunyuan3D-Part** (Sep 2025): P3-SAM (native 3D part segmentation, trained on ~3.7M shapes) + X-Part (part generation/completion). "The current release is a light version of X-Part. The full version is available on Hunyuan3D-Studio". Weights are on HF. License: "Tencent Hunyuan 3D-Part Community License… does not apply in the European Union, United Kingdom and South Korea". — [README](https://raw.githubusercontent.com/Tencent-Hunyuan/Hunyuan3D-Part/main/README.md); [LICENSE](https://github.com/Tencent-Hunyuan/Hunyuan3D-Part); [P3-SAM arXiv snippet](https://arxiv.org/pdf/2509.06784)
- Hunyuan3D-Omni (Sep 26, 2025) uses the same family of Tencent community licenses with the EU/UK/South Korea exclusion. — [LICENSE](https://github.com/Tencent-Hunyuan/Hunyuan3D-Omni)
- **HoloPart** (VAST-AI-Research) has an MIT LICENSE (part amodal segmentation/completion). **UniRig** (VAST, auto-rigging) is also MIT. — [HoloPart](https://github.com/VAST-AI-Research/HoloPart); [UniRig](https://github.com/VAST-AI-Research/UniRig)
- **Tencent XR 3DGen / Pandora3D** (arXiv 2502.14247): its all-in-one release includes `character/phy_cage (PhyCAGE_release)` and `misc/quad_remesh (quad_remesh_utils)`. License is "MIT… with additional restrictions", including prohibition of use in the EU. — [README](https://raw.githubusercontent.com/Tencent/Tencent-XR-3DGen/main/README.md)
- **Tripo Segmentation** (cloud API): v1 geometry-based (default) and v2 semantic labeling + geometry (beta). Accepts GLB/GLTF/FBX/OBJ/STL up to 150 MB. Part Completion fills missing geometry after a split. A Tripo blog states 40 credits per segmentation. — [Tripo developer docs](https://developers.tripo3d.ai/en/docs/mesh-segment); [Tripo blog](https://www.tripo3d.ai/blog/auto-split-model-into-parts)
- Garment3DGen's README itself recommends generating target garment geometry with an image-to-3D model (InstantMesh) and then deforming a template to it, which validates the "generator → registration" pattern. — [README](https://raw.githubusercontent.com/nsarafianos/Garment3DGen/main/README.md)
- **See-through** (SIGGRAPH 2026, Apache-2.0): decomposes a single anime illustration into up to 23 inpainted semantic layers (hair, face, clothing, accessories) with drawing order, using SDXL LayerDiff 3D + anime-finetuned Marigold depth + SAM body parsing. Weights are on HF. Useful to **isolate the outfit from the character art** (body inpainted under clothing) before image-to-3D. — [README](https://raw.githubusercontent.com/shitagaki-lab/see-through/main/README.md); [LICENSE](https://github.com/shitagaki-lab/see-through)

### Inferences
- Recommended non-pattern image route for this user (inference):
  1. Clean the turnaround: run See-through or a manual mask to get "outfit only, on a mannequin-like pose matching your body's A/T-pose".
  2. Generate with TRELLIS.2, which outputs open surfaces (a 24GB card is at the minimum). Alternatively use Hunyuan3D-2.1 shape (10GB) and then P3-SAM to cut garment parts off a whole-character generation.
  3. Align and fit to your body: run a similarity transform (scale/translate) and then a non-rigid fit. Options are Blender Shrinkwrap with offset plus Surface Deform, Garment3DGen-style template deformation, or cloth-fit (section 3).
  4. Open closed shells by deleting faces inside the body. A simple heuristic: delete faces whose normals point inward or that lie within ε of the body's SDF.
  5. Retopo (section 6), paint/project textures, transfer weights, export.
- Generators produce baked-in folds and lumpy, thick shells. For anime style that is often *worse* than a template, because anime clothing is fold-light (section 7). Generated meshes are therefore best used as **targets or references**, not final topology.
- On VRAM: TRELLIS.2's ≥24GB requirement means a 24GB card is at the edge. Hunyuan3D-2.1 shape+paint at 29GB total needs to run in stages or with `--low_vram_mode` on 24GB.

### Gaps
- No published benchmark of TRELLIS.2 or Hunyuan3D on anime outfit turnarounds. Anime-domain quality is unverified.
- Meshy's segmentation/clothing features were not researched in detail (no primary doc fetched).
- No open "garment shell → custom body" auto-registration tool exists as a turnkey package. This step must be assembled from cloth-fit, shrinkwrap, NJF etc.

---

## 3. Garment retargeting between bodies (library garment → user's body)

### Takeaway
This is the most mature non-pattern route. **cloth-fit** ("Intersection-free Garment Retargeting", SIGGRAPH 2025, NYU + Roblox) is open-source (PolyFEM-based, MIT), training-free, and retargets artist garments onto avatars with extreme proportions (examples include a *foxgirl skirt*). Its output is guaranteed intersection-free. In VRChat, commercial tools **Mochifitter** and **Alterith** routinely auto-refit Booth outfits between anime avatars. Inside engines/DCCs, **MD Auto Fitting**, **UE 5.6+ Chaos Outfit Asset resizing** and **Mutable Mesh Reshape** do the same for their own pipelines. **Robust Skin Weights Transfer** (SIGGRAPH Asia 2023, open code + Blender addon) automates the skinning step.

### Cited Findings
**cloth-fit (Intersection-free Garment Retargeting, SIGGRAPH 2025)**
- "Opensource reference implementation… modified based on PolyFEM". C++/CMake build. Run: `PolyFEM_bin -j setup.json --max_threads 16`. Example cases: `foxgirl_skirt`, `Goblin_Jacket`, `Goblin_Jumpsuit`, `Trex_Jacket`. — [README](https://raw.githubusercontent.com/Huangzizhou/cloth-fit/main/README.md)
- Inputs: target avatar `.obj` (triangles only), source garment `.obj`, and **source and target skeleton edge meshes with identical connectivity and joint order**, plus optional avatar skin weights. "Ideally, the source and target skeletons are in the same pose." Losses: surface similarity, curve curvature/torsion (hems, cuffs), curve-loop position, and an IPC-style contact barrier. Outputs a step sequence of garment/avatar OBJs. — [README](https://raw.githubusercontent.com/Huangzizhou/cloth-fit/main/README.md)
- License file: MIT (inherited PolyFEM). Snippet: training-free, "retargets artist-designed garments onto avatars with extreme body proportions, with a guarantee of no intersections". Author Zizhou Huang (NYU and Roblox). — [LICENSE](https://github.com/Huangzizhou/cloth-fit); [search snippet](https://huangzizhou.github.io/research/cloth.html) **(snippet-only for affiliation)**

**Other research**
- **Dress Anyone** (CGF 2026; Naik et al.): isomap-based garment–body correspondences → coarse retarget → physics-constrained neural optimization. It "generalization to biped cartoon characters and non-parametric human meshes in arbitrary poses" and does not need canonicalized garments or parametric bodies. Code status unknown. — [Wiley](https://onlinelibrary.wiley.com/doi/10.1111/cgf.70507) **(snippet-only)**; [project](https://shanthika.github.io/projects/dressmeup/)
- **LoBoFit** (arXiv 2605.07450, 2026): "Flexible Garment Refitting via Local Bone Mapping Blending". Refits garments in geometry across diverse virtual characters. — [arXiv snippet](https://arxiv.org/html/2605.07450) **(snippet-only; code unknown)**
- **LUIVITON** (arXiv 2509.05030, Sep 2025): "Learned Universal Interoperable Virtual Try-On", garment retargeting from a source to a target body. — [arXiv snippet](https://arxiv.org/pdf/2509.05030) **(snippet-only)**
- Other retargeting/draping works in the Awesome-3D-Garments list: "Progressive Outfit Assembly and Instantaneous Pose Transfer" (SIGGRAPH Asia 2025, ACM), DrapeNet (code), ISP multi-layer draping (code), ClothCombo, ULNeF, LayGA. — [Awesome-3D-Garments](https://github.com/Shanthika/Awesome-3D-Garments)

**Skinning transfer**
- **Robust Skin Weights Transfer via Weight Inpainting** (Abdrashitov et al., SIGGRAPH Asia 2023 Tech Comms): Python sample code using libigl + Polyscope. The README says "the code contains the full implementation of the method, and you can swap the meshes". The body→garment FBX example is listed as "(Coming soon)". — [README](https://raw.githubusercontent.com/rin-23/RobustSkinWeightsTransferCode/main/README.md)
- **Blender addon** by SentFromSpaceVR (github.com/sentfromspacevr/robust-weight-transfer). "One click" weight transfer from body, with no smoothing needed between legs, chest or armpits. Adds flipped-normal handling and a point-cloud Laplacian for disconnected meshes. Tutorial on Jinxxy. — [README](https://raw.githubusercontent.com/sentfromspacevr/robust-weight-transfer/main/README.md)

**Commercial / DCC / engine**
- **Marvelous Designer 2025.x**: has an "Auto Fitting" help page (contents not reachable through the proxy). 2025.0 adds drawing patterns on the avatar, soft-body simulation on unrigged custom characters, an AI Pose Generator (beta) and Auto Sewing for tops/pants/skirts. — [CG Channel 2025.0](https://www.cgchannel.com/2025/04/clo-virtual-fashion-releases-marvelous-designer-2025-0/); [CG Channel 2025.1](https://www.cgchannel.com/2025/08/clo-virtual-fashion-releases-marvelous-designer-2025-1/); [MD Auto Fitting page (blocked)](https://support.marvelousdesigner.com/hc/en-us/articles/47358335130649-Auto-Fitting) **(snippet-only)**
- **UE 5.6 Chaos Outfit Asset**: resizing/refitting of garments for the parametric MetaHuman Creator, via the Chaos Cloth Panel Editor (Beta) and Dataflow. Per the forum: resizing **cannot run at runtime** (a new Cloth Asset must be created and cooked in the editor). In 5.6, resizable clothing does not support custom skinning, Control Rig or RBAN, because "when it resizes, the clothing does a simple skin weight transfer from the body, overwriting all of it." Mutable is suggested for runtime resizing. — [Epic forum: Outfit Asset resizing addendum](https://forums.unrealengine.com/t/tutorial-chaos-cloth-outfit-asset-resizing-addendum/2647376); [Chaos Cloth 5.6 updates](https://forums.unrealengine.com/t/tutorial-chaos-cloth-updates-5-6/2555686) **(snippet-only)**; [Epic doc "Getting Started with Parametric Clothing" (blocked)](https://dev.epicgames.com/documentation/unreal-engine/getting-started-with-parametric-clothing?lang=en-US)
- **Mutable "Mesh Reshape"** node "acts like a wrap deformer and can be used to morph clothing to a new body shape". Limitation: it does not work on sections with cloth simulation, because the sim pulls vertices back. No timeline for a fix. — [Epic forum](https://forums.unrealengine.com/t/is-mutable-mesh-reshape-possible-with-cloth-simulation/2689499) **(snippet-only)**
- **VRChat ecosystem**:
  - **Mochifitter** (もちふぃった～, Booth item 7657840) uses "conversion profiles" to rebuild both the shape *and* the weights of an outfit (including transfer of chest shape keys) from the avatar it was made for onto another avatar. Per a technical write-up, a template is fitted to avatar A with bone moves + shrinkwrap, and the deformation is baked into a lattice-like profile. — [vrcfinder](https://vrcfinder.net/en/collection/mochifitter/); [Booth](https://booth.pm/ja/items/7657840); [Zenn analysis](https://zenn.dev/pera/articles/5b4bd0d53c96ba) **(snippet-only)**
  - **Alterith** (Suzu Factory, Booth item 7131644): source avatar + target avatar + outfit → fitted outfit via bone-weight transfer. Unity 2022.3.22f1, Windows only. — [Booth](https://booth.pm/ja/items/7131644); [vrcfinder](https://vrcfinder.net/en/collection/mochifitter/) **(snippet-only)**
  - As of June 2026, 16 of 40 trending Booth avatars advertised support for an auto-fit tool. — [vrcfinder June 2026](https://vrcfinder.net/en/blog/monthly-report/june-2026-avatar-picks/) **(snippet-only)**
  - **Modular Avatar** (open source) is not a fitter. It non-destructively merges outfit armatures into the avatar, reusing bones, for drag-and-drop outfit setup. — [README](https://raw.githubusercontent.com/bdunderscore/modular-avatar/main/README.md)

### Inferences
- **Highest-leverage non-pattern route for an anime creator:** buy or reuse an existing anime outfit, then auto-refit it.
  - Sources: a Booth/VRChat outfit (check its license terms), a VRoid outfit, or an MD library garment.
  - If the source base avatar is known, the refit can use cloth-fit (geometry) + Robust Weight Transfer (skinning) in a Blender script. Mochifitter/Alterith are Unity-only and need profiles for both avatars, so a custom body requires creating your own profile (feasibility unverified).
  - The refit keeps artist topology and UVs, which beats any generator.
- cloth-fit needs matching skeleton graphs for source and target. A VRoid/VRChat-humanoid-like skeleton on your body makes this easy.
- UE's Outfit Asset resizing is tied to the MetaHuman parametric body and Dataflow. It is unclear whether it can target a custom anime body. Its forced simple weight transfer also conflicts with custom skirt bone chains. Prefer doing the refit in Blender, then importing a finished skeletal mesh.

### Gaps
- Whether UE 5.6/5.7 Outfit Asset resizing can target a non-MetaHuman custom body: Epic docs blocked, unverified.
- MD Auto Fitting details (requirements for custom avatars, arrangement points): help page blocked.
- Code release status of Dress Anyone, LoBoFit and LUIVITON: unknown.
- Runtime of cloth-fit on typical game garments (10–50k tris): not stated in the README.

---

## 4. Existing products automating garment creation (2025–2026)

### Takeaway
Commercial "AI garment" products are mostly **pattern-based fashion tools** (Style3D AI, CLO/MD AI features), or **generic 3D generators with segmentation** (Tripo, Meshy, Hunyuan3D Studio). None takes "my custom anime body + turnaround" and returns a rigged, fitted, layered game garment. Ready Player Me, the main avatar-clothing platform, shut down on Jan 31, 2026.

### Cited Findings
- **Style3D AI "AI Garment"**: generates garment designs from text/images, then "analyzing pattern pieces and sewing relationships… automatically constructing 3D digital garments". The image-based mode produces 2D patterns plus a 3D garment with sewing relationships, so it is pattern-based. — [Style3D help](https://help.style3d.com/studio/en/c2c8/7cd2/b61b) **(snippet-only)**
- **Marvelous Designer 2025**: AI Pose Generator (beta), Auto Sewing, Auto Fitting, soft-body on custom avatars (see section 3). — [CG Channel](https://www.cgchannel.com/2025/04/clo-virtual-fashion-releases-marvelous-designer-2025-0/)
- **Tripo**: Segmentation v1/v2 plus Part Completion in Tripo Studio/API (see section 2). — [Tripo docs](https://developers.tripo3d.ai/en/docs/mesh-segment)
- **Hunyuan3D Studio**: has the full X-Part version (open release is "light"). The Hunyuan3D Studio paper calls itself an "End-to-End AI Pipeline for Game-Ready 3D Asset Generation". — [Hunyuan3D-Part README](https://raw.githubusercontent.com/Tencent-Hunyuan/Hunyuan3D-Part/main/README.md); [arXiv 2509.12815 snippet](https://arxiv.org/pdf/2509.12815)
- **Ready Player Me**: acquired by Netflix (announced Dec 19, 2025). The public avatar creator, PlayerZero and developer APIs went offline on Jan 31, 2026. Exported GLBs still work. — [Avatar SDK blog](https://avatarsdk.com/blog/2026/01/15/switch-from-ready-player-me-to-avatar-sdk-fast-familiar-production-ready/); [Variety](https://variety.com/2025/digital/news/netflix-acquires-ready-player-me-games-avatar-creation-1236612915/)
- **Roblox**: the Layered Clothing Importer plugin auto-generates cage meshes on import. The Accessory Fitting Tool auto-builds the Accessory hierarchy, fit edits and attachments. A third-party "Roblox Clothing Cage Generator" exists (ugcraft.ai). — [Roblox AFT docs](https://raw.githubusercontent.com/Roblox/creator-docs/main/content/en-us/avatar/accessory-fitting-tool.md); [ugcraft](https://www.ugcraft.ai/tools/roblox-clothing-cage-generator)

### Inferences
- For this user, products are useful only as components: Tripo/Hunyuan Studio for shells and part splits, MD for drape. "Garment fitted to my anime body, rigged for UE" remains a DIY integration.
- Kaedim, CLO AI and Meshy clothing specifics were not verified (see Gaps).

### Gaps
- Kaedim, Meshy clothing/segmentation and CLO-SET AI features: no primary sources fetched in this pass.
- VRoid Studio's internal auto-fit works only for VRoid's own parametric body and clothing templates (from prior knowledge, not re-verified here).

---

## 5. Cage-based layered clothing (Roblox inner/outer cages; Unreal equivalents)

### Takeaway
Roblox's **inner cage** (where the garment wraps over the body) and **outer cage** (what the next layer wraps over), with runtime cage-to-cage deformation, is the clearest production system for "one garment fits many bodies automatically". Unreal has no identical runtime system. The closest analogues are **Mutable Mesh Reshape** (wrap-deformer, works at runtime, but not on cloth-simulated sections) and editor-only **Chaos Outfit Asset resizing**.

### Cited Findings
- Roblox: "Caging is the process of setting the clothing's interior and exterior surfaces, referred to as the inner and outer cages… This enables your clothing to layer over existing clothing and character bodies, and additional clothes to layer on top." When clothing is modeled on Roblox's cage-template mannequin, "only the outer cage needs to be adjusted". — [Roblox caging-setup doc](https://raw.githubusercontent.com/Roblox/creator-docs/main/content/en-us/art/accessories/creating/caging-setup.md)
- Roblox AFT: "Clothing: Layered accessories that use an inner and outer cage to stretch and wrap around a character body and existing clothing items." It lets you test on multiple default bodies and animations, edit cages, and generate the Accessory object. — [Roblox AFT doc](https://raw.githubusercontent.com/Roblox/creator-docs/main/content/en-us/avatar/accessory-fitting-tool.md)
- Mutable Mesh Reshape is a wrap-deformer for reshaping clothing to body morphs. It fails on cloth-sim sections. — [Epic forum](https://forums.unrealengine.com/t/is-mutable-mesh-reshape-possible-with-cloth-simulation/2689499) **(snippet-only)**
- Chaos Outfit Asset resizing is editor-only and overwrites skin weights with a simple transfer (5.6). — [Epic forum](https://forums.unrealengine.com/t/tutorial-chaos-cloth-outfit-asset-resizing-addendum/2647376) **(snippet-only)**
- The Intersection-free Garment Retargeting author is affiliated with Roblox, which suggests Roblox research interest in offline refit beyond cages. — [search snippet](https://huangzizhou.github.io/research/cloth.html) **(snippet-only)**
- Tencent XR 3DGen's release contains a `phy_cage (PhyCAGE_release)` module. Its purpose was not verified beyond the directory name. — [README](https://raw.githubusercontent.com/Tencent/Tencent-XR-3DGen/main/README.md)

### Inferences
- For a *single* custom body, cages are overkill at runtime. The idea transfers offline, though. Make an "inner cage" of your body: a smoothed, slightly inflated, low-poly proxy. In Blender, bind each garment to a *standard* mannequin cage (Surface Deform / Mesh Deform), then swap the cage to your body's cage. Every library garment authored on that mannequin then auto-fits your body, as a poor-man's Roblox pipeline. Stacked outer cages allow layering (shirt under coat) without interpenetration.
- In UE, if multiple body variants are needed (chibi vs. normal), Mutable + Mesh Reshape is the runtime route for **non-simulated** garments. Simulated parts (skirts, capes) should use bone chains + KawaiiPhysics instead (section 6), which also sidesteps the Mesh Reshape/cloth-sim conflict.

### Gaps
- No Epic primary doc on Mutable Mesh Reshape parameters was accessible.
- Whether Roblox cage deformation algorithms are documented publicly (beyond creator docs): not found.

---

## 6. Automation glue for game-readiness (retopo, UV, bones, weights, UE5 export, KawaiiPhysics)

### Takeaway
Every step has a scriptable, mostly open tool:
- Retopo: QuadWild (GPL3, CLI), Instant Meshes (BSD-style, GUI documented), AutoRemesher (MIT, CLI), or Quad Remesher (paid, ZRemesher author).
- Bone chains: SKBoneGen (MIT), BoneKit and Lazy Bones (paid).
- Weights: Robust Weight Transfer (Python code + Blender addon).
- UE import: the Python AssetImportTask / FbxImportUI API.
- Physics: KawaiiPhysics (MIT, UE 5.3–5.8), which now ships a **MCP toolset** for automated authoring/tuning, plus a Booth tool converting VRM SpringBone → KawaiiPhysics.

### Cited Findings
- **QuadWild** (Pietroni et al., "Reliable Feature-Line Driven Quad-Remeshing"): fully automatic. Fewer than 0.5% of Thingi10K inputs fail. CLI `./quadwild <mesh> [setup.txt] [.rosy] [.sharp]`, OBJ/PLY input. GPL3. — [README](https://raw.githubusercontent.com/nicopietroni/quadwild/main/README.md)
- **Instant Meshes** (Jakob et al.): the README documents a GUI workflow (orientation field → position field → export). License is BSD-style ("Redistribution and use in source and binary forms…"). — [README](https://raw.githubusercontent.com/wjakob/instant-meshes/master/README.md)
- **AutoRemesher** (Dust3D, huxingyi): MIT (2026 copyright). A Blender add-on drives the AutoRemesher CLI as a subprocess (snippet). — [LICENSE](https://github.com/huxingyi/autoremesher); [search snippet](https://github.com/BrandonLangdon/autoremesher)
- **Quad Remesher** (Exoside): by Maxime Rouca, the ZRemesher coder. Blender add-on drives an external executable. Available on Windows/macOS/Linux. — [BlenderNation](https://www.blendernation.com/2019/10/08/quad-remesher-auto-retopology-add-on-released-for-blender/); [BlenderNation FAQ](https://www.blendernation.com/2019/10/18/quad-remesher-auto-retopologizer-for-blender-unofficial-faq/)
- Tencent XR 3DGen ships `misc/quad_remesh (quad_remesh_utils)`. — [README](https://raw.githubusercontent.com/Tencent/Tencent-XR-3DGen/main/README.md)
- **Bone chains**: SKBoneGen ("generating bones along vertices… for skirts, hair", MIT). BoneKit (edge→bone + bone splitter with weight redistribution, auto-detects multiple edge chains; paid). Lazy Bones (Edge-2-Bones; paid). — [SKBoneGen](https://github.com/ek1den2/SKBoneGen); [BoneKit](https://superhivemarket.com/products/bonekit); [Lazy Bones](https://blenderartists.org/t/lazy-bones-simulation-auto-rigging-and-edges-to-bones/1527140)
- **Auto-rigging**: UniRig (VAST, MIT) for generated meshes. — [UniRig](https://github.com/VAST-AI-Research/UniRig)
- **Weights**: Robust Weight Transfer code + Blender addon (section 3). — [rin-23 README](https://raw.githubusercontent.com/rin-23/RobustSkinWeightsTransferCode/main/README.md); [addon README](https://raw.githubusercontent.com/sentfromspacevr/robust-weight-transfer/main/README.md)
- **UE5 import automation**: an Epic community tutorial covers automating skeletal mesh import with Python. Pattern: `AssetImportTask` + `FbxImportUI` (`import_as_skeletal`, `mesh_type_to_import=FBXIT_SKELETAL_MESH`, `skeleton`, `physics_asset`) → `AssetToolsHelpers.get_asset_tools().import_asset_tasks()`. — [Epic forum tutorial](https://forums.unrealengine.com/t/community-tutorial-automating-skeletal-mesh-imports-with-unreal-engines-python-api/2069661); [FbxImportUI API](https://docs.unrealengine.com/4.26/en-US/PythonAPI/class/FbxImportUI.html)
- **KawaiiPhysics** (pafuhana1213, MIT):
  - Supports UE 5.3–5.8. Sphere/capsule/plane collisions, fixed bone lengths, wind and custom forces, DataAsset/PhysicsAsset reuse.
  - BoneConstraint keeps skirts from clipping through legs. SyncBone syncs leg bones into the sim.
  - Sample project for UE 5.8 enables the experimental **Unreal MCP** plugin, and KawaiiPhysics bundles a `KawaiiPhysicsToolset`. It covers adding/editing nodes, collision limits, presets and audits, PIE verification (penetration samplers), and "skirt check-and-tune helpers (ring bone constraints, radius by depth, motion recording and analysis)".
  - Links a Booth tool converting VRM SpringBone → KawaiiPhysics.
  — [README](https://raw.githubusercontent.com/pafuhana1213/KawaiiPhysics/master/README.md); [VRM→KawaiiPhysics tool](https://yumetengu.booth.pm/items/7943387)

### Inferences
- A fully scripted pipeline is feasible (inference): run Blender headless (`blender -b -P pipeline.py`) through these steps:
  1. Import the generated/refit garment.
  2. Fit to the body (shrinkwrap, or cloth-fit externally).
  3. Retopo with a QuadWild/AutoRemesher subprocess.
  4. UV with Smart UV Project, or keep the template UVs.
  5. Bone chains with SKBoneGen-style edge loops from waist to hem.
  6. Robust Weight Transfer from the body for the torso, then weights on skirt chain bones.
  7. Export FBX.
  8. UE Python import onto the body's skeleton.
  9. Configure KawaiiPhysics, possibly via its MCP toolset, driven by an LLM agent.
- Retopo choice: QuadWild is best for feature-preserving (hems, seams) pure-quad output, but GPL3 matters only if redistributing. Instant Meshes is good for fast uniform quads. Template-deformation outputs (Garment3DGen) skip retopo entirely.

### Gaps
- Instant Meshes' CLI flags were not verified (the fetched README only documents the GUI).
- Quad Remesher CLI/headless support: not confirmed from a primary source. The official PDF user doc was not fetched.
- No verified off-the-shelf script that auto-detects a skirt's waist/hem loops and builds bone chains without user edge selection.

---

## 7. Anime stylization of generated garments (fold reduction, flat texture projection, outline-friendly topology)

### Takeaway
The practical tools are **StableProjectorz** (now AGPL-3.0 open source, Jan 2026; projects SD/ControlNet generations onto existing UVs, and bundles TRELLIS 1/2 and Hunyuan3D 2.0/2.1 installers) and **Hunyuan3D-Paint-2.1** (callable on any mesh). Fold reduction and outline-friendly topology come mainly from **template deformation** (smooth base meshes) or retopology plus smoothing, not from generators.

### Cited Findings
- **StableProjectorz**: free desktop app that projects Stable Diffusion generations onto a mesh "while preserving the original UV layout, on one consumer NVIDIA GPU". It supports up to six view arrangements, depth/normal ControlNet, artist-painted blend masks, UV-space inpainting and UDIMs. Two modes: 2D texturing via A1111/Forge/ComfyUI, and integrated mesh generation with one-click installers for "Trellis 1/2 and Hunyuan3D 2.0/2.1". First released Jan 2024 and "open-sourced under AGPL-3.0 in January 2026". — [ACM paper snippet](https://dl.acm.org/doi/10.1145/3799825.3818726); [site](https://www.stableprojectorz.com/) **(snippet-only)**
- **Hunyuan3D-Paint-2.1** (2B, PBR) can texture an arbitrary mesh from a reference image (`Hunyuan3DPaintPipeline(mesh_path, image_path=…)`). Texture generation needs 21GB. — [README](https://raw.githubusercontent.com/Tencent-Hunyuan/Hunyuan3D-2.1/main/README.md)
- **TRELLIS.2** offers "shape-conditioned texture generation" inference code, i.e., it can texture a supplied mesh. — [README](https://raw.githubusercontent.com/microsoft/TRELLIS.2/main/README.md)
- GarmentDreamer notes its provided templates are "unwrinkled", and that wrinkled templates can do better for realistic output. For anime, the unwrinkled templates are an advantage. — [README](https://raw.githubusercontent.com/boqian-li/GarmentDreamer/main/README.md)
- See-through gives clean, inpainted per-garment anime layers that can serve as flat projection sources. — [README](https://raw.githubusercontent.com/shitagaki-lab/see-through/main/README.md)

### Inferences
- PBR texturers (Hunyuan3D-Paint) bake in realistic shading and lighting cues. For cel-shaded UE materials, project flat-color art from the turnaround with StableProjectorz instead, or prompt for "flat colors, no shading". Then use flat albedo with a toon shader.
- Fold reduction on generated shells: Laplacian/Taubin smoothing or Blender's Corrective Smooth before retopo, plus remeshing to low density. Alternatively, use the generated shell only as a Garment3DGen target, so the clean template supplies the surface.
- Outline-friendly topology: even quads, consistent normals, no thin double-walled shells, and closed hems with a small thickness (Solidify). Generated closed shells must be opened (inside faces deleted) and given a single-layer + Solidify setup, or inverted-hull outlines will double.

### Gaps
- No published method found specifically for "anime-style fold removal" of generated garments. This is a gap in the literature, not just in this search.
- No benchmark comparing StableProjectorz vs Hunyuan3D-Paint vs TRELLIS.2 texturing on anime garments.

---

## 8. Recommendation: when non-pattern beats pattern+simulation, and how to combine

### Takeaway
For a solo anime creator targeting UE5:
- **Non-pattern wins** for outfits that (a) resemble an existing garment (refit a library/Booth/VRoid garment with cloth-fit + Robust Weight Transfer), (b) are tight or "painted-on" (bodysuits, leggings, sleeves: shrinkwrap from the body), or (c) are rigid or ornamental (armor, accessories, belts: image-to-3D + part segmentation).
- **Pattern + sim wins** for loose, flowing, hero garments: skirts with pleats, capes, coats with specific drape. Those also need clean seams/UVs and believable folds, and template-deformation templates rarely match their structure.
- **Best combined approach**: pattern+sim (or MD) for a small set of base templates fitted once to your body, then Garment3DGen-style deformation, retargeting and AI texturing to make variants automatically, with bone chains + KawaiiPhysics in UE instead of real-time cloth.

### Cited Findings
- Garment3DGen explicitly requires source and target to be "reasonably close". Skirt→shirt fails. — [README](https://raw.githubusercontent.com/nsarafianos/Garment3DGen/main/README.md)
- cloth-fit retargets artist garments onto extreme proportions (foxgirl skirt example), intersection-free, with no training. — [README](https://raw.githubusercontent.com/Huangzizhou/cloth-fit/main/README.md)
- StdGEN is trained on VRoid anime data and outputs separate clothing layers. It is Apache-2.0 code. — [README](https://raw.githubusercontent.com/hyz317/StdGEN/main/README.md)
- TRELLIS.2 handles open-surface clothing, needs ≥24GB, MIT. — [README](https://raw.githubusercontent.com/microsoft/TRELLIS.2/main/README.md)
- UE Outfit Asset resizing overwrites custom skinning and is editor-only. Mutable Reshape fails on cloth-sim sections. — [Epic forum](https://forums.unrealengine.com/t/tutorial-chaos-cloth-outfit-asset-resizing-addendum/2647376); [Epic forum](https://forums.unrealengine.com/t/is-mutable-mesh-reshape-possible-with-cloth-simulation/2689499) **(snippet-only)**
- KawaiiPhysics provides skirt-specific constraints and an MCP toolset for automated tuning in UE 5.8. — [README](https://raw.githubusercontent.com/pafuhana1213/KawaiiPhysics/master/README.md)
- The VRChat community has normalized auto-refit (Mochifitter, Alterith), with 40% of trending Booth avatars advertising support by June 2026. — [vrcfinder](https://vrcfinder.net/en/blog/monthly-report/june-2026-avatar-picks/) **(snippet-only)**

### Inferences
- **Decision rule (inference):**
  1. Does a similar garment exist in a library you can legally use? → **Retarget** (cloth-fit / shrinkwrap + Surface Deform → Robust Weight Transfer). Fastest and highest topology quality.
  2. Is it tight or skin-like? → **Body-derived**: duplicate body faces, offset, cut, Solidify, project the texture.
  3. Is it rigid or decorative? → **Image-to-3D** (TRELLIS.2/Hunyuan3D) → P3-SAM split → retopo → rigid-attach to a bone.
  4. Is it loose or flowing and a hero piece? → **Pattern+sim** (MD or an automated pattern pipeline) for the base drape on your body. Then reduce folds, retopo, add bone chains and KawaiiPhysics.
  5. Need many variants of a base? → **Garment3DGen/GarmentDreamer deformation** of your fitted base + StableProjectorz/Hunyuan-Paint texture from the turnaround.
- **Hybrid "automatic MD" architecture (inference):**
  - An LLM parses the outfit-profile text into a list of garment types, layers and materials. Tailor's design uses the same idea.
  - Each garment is routed by the rule above.
  - Every route converges on one Blender headless back-end: fit, retopo, UV, bones, weights, FBX.
  - UE Python import → KawaiiPhysics config (via the MCP toolset or DataAsset presets).
  - Real-time Chaos cloth is reserved for 1–2 hero pieces, if any.
- StdGEN is the one place where an anime-trained, layer-separating generator fits. Use it to get a *first-pass 3D proxy* of the whole outfit from the turnaround, then retarget its clothing layer to your body as a target for template deformation or as a blockout.
- Licensing for personal use: CC BY-NC (Garment3DGen, GarmentDreamer) and the Tencent community licenses (EU/UK/South Korea excluded) are fine for non-commercial personal work. Commercial release of a game would need re-checking. TRELLIS.2, StdGEN code, cloth-fit, KawaiiPhysics, SKBoneGen and AutoRemesher are permissive (MIT/Apache).

### Gaps
- No head-to-head study compares non-pattern vs pattern+sim garments for anime game characters. The decision rule above is reasoned, not empirically validated.
- Real-world success rate of TRELLIS.2/Hunyuan3D on anime turnarounds of clothing *without* a body is untested in available sources.
- Whether Mochifitter/Alterith profiles can be authored for a fully custom (non-Booth) body, and whether they export outside Unity: unverified (Alterith docs blocked).
