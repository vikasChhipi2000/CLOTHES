# Image / Text-Profile → Sewing Pattern (no user training): methods, toolkits, agentic LLM approaches

Research date: 2026-10-04. Researcher notes for the report writer.

**Access notes (important for trust levels):**
- Fetched directly as primary sources: GitHub READMEs, LICENSE files, configs and code via `raw.githubusercontent.com` (GarmentCode, GarmentCodeRC, ChatGarment, AIpparel-Code, DressCode, Sewformer, NeuralTailor, SewingLDM, GarmentDiffusion, design2garmentcode-impl, and the Design2GarmentCode project-page source `index.html`).
- **Blocked by the egress proxy:** arxiv.org (abs, html, pdf, and export API), huggingface.co (model cards and papers), style3d.github.io, alphaxiv, opentrain.ai, pith.science, lacuna, semanticscholar, and the GitHub search API (`gh search` returns 403).
- So every claim about 2026 arXiv papers (NGL-Prompter, PatternGSL, TailorCoPilot, GarmentWeaver, DressWild, Image2Garment, GarmentGPT) and about Dress-1-to-3, GarmentImage and commercial tools comes from **web-search snippets only**. These are marked "[snippet]".
- I could not check Hugging Face model-card licenses. Any weight-license statement below is inferred from the base model or is listed as a gap.

---

## 1. Pattern representations and toolkits (GarmentCode, pygarment, GarmentCodeData, Korosteleva JSON, Seamly2D/Valentina)

### Takeaway
GarmentCode/pygarment (MIT) is the de-facto target format for almost every 2024–2026 image/text→pattern method: ChatGarment, AIpparel, Design2GarmentCode, SewingLDM, NGL-Prompter and Image2Garment all emit or consume it. It is a Python DSL in which you write garment programs that produce JSON panel+stitch specs, and it has a built-in Warp cloth simulator. Its built-in design space covers realistic Western garments: shirts, bodices, skirts (circle, godet, many-panel, pencil, tiered), pants, waistbands, hoods, lapels, turtlenecks, sleeves and cuffs. It has **no knife/box-pleat, sailor-collar, cape or frill component out of the box**. Anime pieces must be added by writing new pygarment Panel/Component classes, which the library is designed to allow.

### Cited Findings
- GarmentCode is the official code for "GarmentCode: Programming Parametric Sewing Patterns" (ACM TOG 42(6), SIGGRAPH Asia 2023, doi 10.1145/3618351) and for "GarmentCodeData: A Dataset of 3D Made-to-Measure Garments With Sewing Patterns" (ECCV 2024). — [GarmentCode README](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/ReadMe.md)
- License: MIT, Copyright (c) 2024 Maria Korosteleva. — [GarmentCode LICENSE](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/LICENSE)
- `pygarment` is the core library. It contains the base types (Edge, Panel, Component, Interface, etc.), an edge factory, helpers and operators for designing sewing patterns. Install with `pip install pygarment`. Dependencies: Python 3.9, numpy<2, scipy, pyyaml, svgwrite, svgpathtools, cairoSVG, NiceGUI, trimesh. — [GarmentCode README](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/ReadMe.md); [Installation.md](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/docs/Installation.md)
- Simulation uses the authors' fork of NVIDIA Warp (`maria-korosteleva/NvidiaWarp-GarmentCode`), which must be installed manually. An older Maya+Qualoth path is kept for backward compatibility. — [Installation.md](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/docs/Installation.md)
- pygarment v2.0.0 (2024-08-30) added:
  - box-mesh generation from patterns;
  - Warp simulation from the command line and the GUI;
  - random design sampling;
  - edge/panel labels in the JSON;
  - explicit stitch orientation (right-to-right vs right-to-wrong);
  - a NiceGUI browser GUI.
  - v2.0.2 (2025-04-18) made UVs preserve panel aspect ratio.
  — [CHANGELOG](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/CHANGELOG.md)
- An online configurator demo is at https://garmentcode.ethz.ch/ ("back online" June 3, 2025; not for mobile). GarmentCodeData v2 was released Sept 4, 2024 (doi 10.3929/ethz-b-000690432). — [GarmentCode README](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/ReadMe.md)
- Example garment programs live in `assets/garment_programs/`. Design presets are in `assets/design_params/` and body-measurement presets in `assets/bodies/`. `default.yaml` is the GUI's initial state. — [GarmentCode README](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/ReadMe.md)
- **Shape of the design YAML** (`assets/design_params/default.yaml`, 885 lines). It is nested `design: <component>: <param>: {v, range, type, default_prob}`. Types include `float`, `int`, `bool`, `select` and `select_null`. Examples:
  - `meta.upper` ∈ {FittedShirt, Shirt, null}
  - `meta.wb` ∈ {StraightWB, FittedWB, null}
  - `meta.bottom` ∈ {SkirtCircle, AsymmSkirtCircle, GodetSkirt, Pants, Skirt2, SkirtManyPanels, PencilSkirt, SkirtLevels, null}
  - `collar.f_collar` / `b_collar` ∈ {CircleNeckHalf, CurvyNeckHalf, VNeckHalf, SquareNeckHalf, TrapezoidNeckHalf, CircleArcNeckHalf, Bezier2NeckHalf}
  - `collar.component.style` ∈ {Turtle, SimpleLapel, Hood2Panels, null}
  - sleeve parameters: `sleeveless`, `armhole_shape` {ArmholeSquare, ArmholeAngle, ArmholeCurve}, `length`, `end_width`, `standing_shoulder`, `connect_ruffle`, and `cuff.type` {CuffBand, CuffSkirt, CuffBandSkirt}
  - a `left:` block with `enable_asym` for asymmetric designs
  - skirt parameters such as `flare`, `rise`, front/back/side slits, `num_levels` and `level_ruffle` (tiered), and pants `length/width/flare/rise/cuff`
  — [default.yaml](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/assets/design_params/default.yaml)
- Lengths and widths in the design YAML are relative factors (e.g., `shirt.length` 0.5–3.5, `width` 1.0–1.3). Garment programs combine them with a body-measurement YAML (`assets/bodies/`). The `MetaGarment` component raises `TotalLengthError` if a garment goes beyond floor length for a given body. — [meta_garment.py](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/assets/garment_programs/meta_garment.py)
- `MetaGarment` assembles an upper garment, an optional waistband and a lower garment by class name (`globals()[name]`). It places them with `place_by_interface(...)` and joins them with `stitching_rules.append((interfaceA, interfaceB))`. This is the extension mechanism: a new class only has to expose `interfaces['top'/'bottom']`. — [meta_garment.py](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/assets/garment_programs/meta_garment.py)
- Gathers and "ruffles" are implemented as a length-ratio factor on interfaces: `top_width = base_width * ruffles`, then matching a longer edge to a shorter one. Flare is a bottom-width offset. Grepping `skirt_paneled.py` found no pleat construct. Classes there: SkirtPanel, ThinSkirtPanel, FittedSkirtPanel, PencilSkirt, Skirt2, SkirtManyPanels. — [skirt_paneled.py](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/assets/garment_programs/skirt_paneled.py)
- Available operators include `cut_corner`, `cut_into_edge` (darts and cut-ins), `distribute_Y` / `distribute_horisontally` (radial or horizontal copies of panels), `even_armhole_openings` and `curve_match_tangents`. — [operators.py](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/pygarment/garmentcode/operators.py)
- **Serialized JSON pattern spec** (the format shared with Korosteleva's older NeuralTailor/2021 dataset):
  - Top level: `{"pattern": {"panels": {name: {...}}, "stitches": [...]}, "properties": {"curvature_coords": "relative", "units_in_meter": 100, ...}}`, where `units_in_meter: 100` means cm.
  - Each panel has `translation [x,y,z]`, `rotation [x,y,z]`, 2D `vertices` and `edges`.
  - Each edge has `endpoints` and optional `curvature`, either a quadratic control point or `{"type": "circle", "params": [radius, large_arc, right]}`; cubic Béziers are also used in v2.
  — [pygarment/pattern/core.py](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/pygarment/pattern/core.py)
- GarmentCodeRC (`biansy000/GarmentCodeRC`) is a refined fork used by ChatGarment. It adds open-front garments, tighter pants and high-waist garments. — [GarmentCodeRC README](https://raw.githubusercontent.com/biansy000/GarmentCodeRC/main/ReadMe.md)
- NeuralTailor's dataset (made with `Garment-Pattern-Generator`, Korosteleva & Lee 2021) is on Zenodo (doi 10.5281/zenodo.5267549). DressCode/SewingGPT trains on it. — [NeuralTailor README](https://raw.githubusercontent.com/maria-korosteleva/Garment-Pattern-Estimation/master/ReadMe.md); [DressCode README](https://raw.githubusercontent.com/IHe-KaiI/DressCode/main/README.md)
- Seamly2D (GPLv3+) is a fork of Valentina, an open-source parametric pattern-drafting CAD tool. Valentina introduced parametric design for refitting a pattern to a different person. — [LinuxLinks](https://www.linuxlinks.com/seamly2d-pattern-making/); [Wikipedia: Valentina](https://en.wikipedia.org/wiki/Valentina_(software)); [Libre Arts](https://librearts.org/2017/12/valentina-seamly2d/)

### Inferences
- For this user, GarmentCode is the right **intermediate representation**: open (MIT), Python, LLM-readable, simulatable, and the common output of every research model.
- Anime-specific parts need new components:
  - knife/box-pleated skirt: a rectangle panel with fold lines, or panel copies via `distribute_horisontally`, plus a gathered waist;
  - sailor collar: a flat collar panel attached to a V-neck interface;
  - capes/capelets: circle-sector panels attached at the neckline;
  - frills: CuffSkirt-like strips with `ruffle` > 1.
  - An LLM can write these classes, since `MetaGarment` already composes components by class name and interface.
- Pleats are not natively folded by the Warp sim. You would either model them as gathers (visually softer) or add sew-lines/fold constraints, which may require a different simulator (MD/Style3D) downstream.
- A practical bridge to Marvelous Designer: GarmentCode can export SVG/PNG patterns plus JSON stitch lists. MD imports DXF-AAMA, so a JSON→DXF converter would need writing. I did not verify whether GarmentCode ships a DXF exporter.

### Gaps
- I did not verify the GarmentCodeData license (the research-collection page was not fetched). Check before use.
- I did not confirm whether GarmentCode exports DXF directly. The CHANGELOG mentions mesh/UV/SVG but not DXF.
- I did not fetch the Garment-Pattern-Generator README details (template list for the 2021 dataset).

---

## 2. Image→pattern and text→pattern models (released weights, local runnability, formats)

### Takeaway
As of Oct 2026, the methods with **released code and pretrained weights** that run locally on a 24 GB GPU are:

| Method | Venue | Input | Model | Output | Code license |
|---|---|---|---|---|---|
| ChatGarment | CVPR 2025 (arXiv 2412.17811) | image or text | LLaVA-1.5-7B base | GarmentCodeRC | Apache-2.0 |
| AIpparel | CVPR 2025 Highlight | image + text | LLaVA-1.5-7B | GarmentCodeData-style pattern | no LICENSE file found |
| Design2GarmentCode | CVPR 2025 | image, text, sketch | Qwen2-VL-2B LoRA + GPT-4o API | GarmentCode programs | MIT |
| SewingLDM | ICCV 2025 | sketch + text + body shape | latent diffusion | GarmentCode JSON | Apache-2.0 |
| Sewformer | SIGGRAPH Asia 2023 | image | — | old Korosteleva format | — |
| DressCode/SewingGPT | SIGGRAPH 2024 | text only | — | old Korosteleva format | — |

- NeuralTailor takes **3D point clouds**, not images.
- GarmentDiffusion's repo had no install, inference or weights instructions in its README.
- Dress-1-to-3, GarmentImage, GarmentGPT, PatternGSL, DressWild, GarmentWeaver and Image2Garment: code or weights unconfirmed (snippets only). Image2Garment uses frozen ChatGarment for geometry.

### Cited Findings

**NeuralTailor (SIGGRAPH 2022, ACM TOG 41(4))**
- Reconstructs sewing pattern structures from **3D point clouds** of garments. The repo provides pretrained models, evaluation scripts and training tools. License MIT. — [README](https://raw.githubusercontent.com/maria-korosteleva/Garment-Pattern-Estimation/master/ReadMe.md); [LICENSE](https://raw.githubusercontent.com/maria-korosteleva/Garment-Pattern-Estimation/master/LICENSE)

**Sewformer (ACM TOG / SIGGRAPH Asia 2023, `sail-sg/sewformer`)**
- Single-image sewing pattern reconstruction. Pretrained checkpoint at `huggingface.co/liulj/sewformer`; dataset SewFactory at `huggingface.co/datasets/liulj/sewfactory`.
- Real-image inference: `python inference.py -c configs/test.yaml -d assets/data/deepfashion ...`.
- Simulation of predictions needs **Maya (mayapy, Windows)** plus SMPL predictions from RSC-Net. No LICENSE file was found at the repo root.
- — [Sewformer README](https://raw.githubusercontent.com/sail-sg/sewformer/main/ReadMe.md)
- Design2GarmentCode's authors report Sewformer issues on in-the-wild images: incorrect necklines, missing components, misplaced or imaginary stitches, extraneous panels, and oversized waists because body shape is ignored. — [Design2GarmentCode project page source](https://raw.githubusercontent.com/style3d/Design2GarmentCode/master/index.html)

**DressCode / SewingGPT (SIGGRAPH 2024, ACM TOG 43(4), `IHe-KaiI/DressCode`)**
- Text-only, GPT-style autoregressive sewing pattern generator plus a Stable Diffusion-based PBR texture generator.
- Pretrained SewingGPT is at `huggingface.co/IHe-KaiI/DressCode/tree/main/models`. Needs SD-2.1-base for the CLIP embedding.
- Gradio UI: `python nn/UI_chat.py`, with prompts like `dress, sleeveless, midi length`. Multi-garment prompts are separated with `;`.
- Simulation is **Windows-only via Maya**. An optional ChatGPT "LLM interpreter" mode exists.
- Trained on the Korosteleva & Lee 2021 dataset. README says results can be loaded into Marvelous Designer for further simulation. No LICENSE file found.
- — [DressCode README](https://raw.githubusercontent.com/IHe-KaiI/DressCode/main/README.md)

**ChatGarment (arXiv 2412.17811; CVPR 2025; `biansy000/ChatGarment`, Apache-2.0)**
- A VLM built on LLaVA + LISA. Supports image-based reconstruction (2-step CoT: the model writes a text description, then generates GarmentCode), text-based generation and text-based editing.
- The text generation and editing scripts call **GPT-4o** to reformat prompts first.
- Weights are a SharePoint link (`checkpoints/try_7b_lr1e_4_v3_garmentcontrol_4h100_v4_final/pytorch_model.bin`). Requires flash-attn and GarmentCodeRC.
- Draping with `run_garmentcode_sim.py` uses ContourCraft-CG. Dataset at `huggingface.co/datasets/sy000/ChatGarmentDataset`. Multi-turn conversation was "Coming Soon".
- — [ChatGarment README](https://raw.githubusercontent.com/biansy000/ChatGarment/main/README.md); [Installation.md](https://raw.githubusercontent.com/biansy000/ChatGarment/main/docs/Installation.md); [LICENSE](https://raw.githubusercontent.com/biansy000/ChatGarment/main/LICENSE)
- README warning: "ChatGarment may occasionally produce garments with incorrect lengths or widths from input images." The optional postprocess is a finite-difference refinement of length/width that matches Grounding-SAM garment masks. It also uses TokenHMR SMPL pose/shape estimation and PyTorch3D. — [postprocess.md](https://raw.githubusercontent.com/biansy000/ChatGarment/main/docs/postprocess.md)
- The project page claims ChatGarment edits only the targeted part, while SewFormer/DressCode-based editing alters untouched areas. — [ChatGarment project page source](https://raw.githubusercontent.com/ChatGarment/ChatGarment.github.io/main/index.html)

**AIpparel (CVPR 2025 Highlight, arXiv 2412.03937; `georgeNakayama/AIpparel-Code`)**
- Multimodal foundation model for sewing patterns. Pretrained weights `aipparel_pretrained.pth` and the GarmentCodeData-Multimodal annotations (`gcd_mm_editing.zip`, `gcd_mm_captions.zip`) are at `huggingface.co/georgeNakayama/AIpparel`.
- Inference: `scripts/inference.sh` with `assets/data_configs/inference_example.json` for your image/text. Torch 2.3.1 / CUDA 12.1.
- — [AIpparel README](https://raw.githubusercontent.com/georgeNakayama/AIpparel-Code/master/README.md)
- Config: base `version: liuhaotian/llava-v1.5-7b`, `precision: bf16`, `model_max_length: 2100`. Requirements include flash_attn 2.7.4, deepspeed, peft, transformers 4.31. No LICENSE file was found at the repo root. — [aipparel.yaml](https://raw.githubusercontent.com/georgeNakayama/AIpparel-Code/master/configs/aipparel.yaml); [requirements.txt](https://raw.githubusercontent.com/georgeNakayama/AIpparel-Code/master/requirements.txt)

**Design2GarmentCode (CVPR 2025, pp. 23712–23722; Style3D Research + ZSTU/SJTU/ZJU; `Style3D/design2garmentcode-impl`, MIT)**
- Pipeline:
  - a fine-tuned DSL Generation Agent (DSL-GA) learns GarmentCode grammar and parameter semantics;
  - it prompts a Multi-Modal Understanding Agent (MMUA) to extract design features from the image, text or sketch;
  - it synthesizes GarmentCode design configs and programs;
  - rule-based validation runs;
  - **after generation, the MMUA compares the generated design with the input and suggests modifications** (a closed feedback loop).
  — [project page source](https://raw.githubusercontent.com/style3d/Design2GarmentCode/master/index.html)
- Released implementation:
  - MMUA defaults to **ChatGPT-4o via API**. `system.json` lets you set `api_key`, `base_url` and `model`.
  - The "parameter projector" is **Qwen2-VL-2B-Instruct + LoRA/MLP weights from Google Drive**.
  - NiceGUI GUI (`python gui.py`, "PARSE DESIGN" tab, `modify: <instruction>` for edits).
  - Batch scripts for text and images; optional GarmentCode Warp simulation; Python 3.9.19, Torch 2.4.0 + CUDA 12.1.
  — [design2garmentcode-impl README](https://raw.githubusercontent.com/Style3D/design2garmentcode-impl/main/README.md); [LICENSE (MIT, 2025 Style3D)](https://raw.githubusercontent.com/Style3D/design2garmentcode-impl/main/LICENSE)
- Claims: handles images, text, designer sketches and combinations; produces size-precise patterns with correct stitches; captures necklines, cuffs, darts and asymmetry. Sketch outputs integrate with industrial fashion software (Style3D) for pattern editing. — [project page source](https://raw.githubusercontent.com/style3d/Design2GarmentCode/master/index.html)

**SewingLDM (ICCV 2025, arXiv 2412.14453; `shengqiliu1/SewingLDM`, Apache-2.0)**
- Latent diffusion conditioned on **text + body shape + garment sketch**. Weights `auto_encoder.pth` and `sewingldm.pth` are at `huggingface.co/liusq/sewingldm`.
- Inference: `generate.sh` (set sketch, text and body paths). Draping uses GarmentCode's simulator. Built on PixArt and Sewformer; trained on GarmentCodeData v2. Torch 2.0 / CUDA 11.8.
- — [SewingLDM README](https://raw.githubusercontent.com/shengqiliu1/SewingLDM/master/README.md); [LICENSE](https://raw.githubusercontent.com/shengqiliu1/SewingLDM/master/LICENSE)
- Edge features are expanded to 29 dimensions (4 edge types, attachment constraints, stitch-reversal flags). Training has two steps: text-only, then body+sketch. [snippet] — [paper notes](https://en.papernotes.org/ICCV2025/image_generation/multimodal_latent_diffusion_model_for_complex_sewing_pattern_generation/); [project](https://shengqiliu1.github.io/SewingLDM/)

**GarmentDiffusion (IJCAI 2025, pp. 1458–1466; `Shenfu-Research/GarmentDiffusion`)**
- Multimodal input (text, image, incomplete pattern), edge-token diffusion transformer, evaluated on DressCodeData, SewFactory and GarmentCodeData.
- The fetched README contains only intro and citation, **no install, inference, weights or license**. The project page links a second repo, `Shenfu-Research/Garment-Diffusion`, whose README also had no weights or inference mentions.
- — [README](https://raw.githubusercontent.com/Shenfu-Research/GarmentDiffusion/main/README.md)
- Claims 10× shorter sequences than SewingGPT and 100× faster generation. [snippet] — [IJCAI proceedings](https://www.ijcai.org/proceedings/2025/163)

**Dress-1-to-3 (ACM TOG / SIGGRAPH 2025, arXiv 2502.03449)** [snippet]
- Pipeline: a pretrained image→pattern model gives a coarse pattern; a multi-view diffusion model generates RGB and normal views; then a differentiable IPC cloth simulator optimizes pattern and stiffness until renders match. Outputs separated, simulation-ready garments plus the human.
- I found no code repo beyond the project-page repo (`dress-1-to-3/dress-1-to-3.github.io`, whose index.html had no code link).
- — [arXiv](https://arxiv.org/abs/2502.03449); [project](https://dress-1-to-3.github.io/); [ACM DL](https://dl.acm.org/doi/10.1145/3731177)

**GarmentImage (SIGGRAPH 2025 Conference Papers; Tatsukawa, Qi, Shen, Igarashi; arXiv 2505.02592)** [snippet]
- Raster multi-channel grid encoding of pattern geometry, topology and placement. Applications: latent exploration, text editing, image→pattern. No code repo found.
- — [INRIA Basilic](http://www-sop.inria.fr/reves/Basilic/2025/TQSI25/); [arXiv](https://arxiv.org/pdf/2505.02592)

**GarmentGPT (ICLR 2026 poster; Weng, Chen, Li, Qin, Guo, Hao, Han)** [snippet]
- An RVQ-VAE tokenizes pattern boundary curves into discrete codes for latent generation. No code or weights confirmed.
- — [OpenReview](https://openreview.net/forum?id=XzXKnazRBF); [ICLR](https://iclr.cc/virtual/2026/poster/10008926)

**NGL-Prompter (arXiv 2602.20700, Feb 2026; MPI-IS / Black group, incl. Omid Taheri)** [snippet]
- **Fully training-free.** A pretrained VLM extracts a "Natural Garment Language" spec from a single image using discrete semantic terms, e.g., `neckline: v-neck`, `sleeve_length: three-quarter`.
- The spec has 5 blocks (meta, bodice, sleeve, skirt, pants). A deterministic parser maps it to GarmentCode parameters.
- Claims recovery of **multi-layer outfits**, where competitors mostly handle single layers. "Code and data will be released for research use"; I found no repo.
- — [arXiv](https://arxiv.org/abs/2602.20700); [author page source](https://raw.githubusercontent.com/otaheri/otaheri.github.io/main/index.html)

**DressWild (arXiv 2602.16502, Feb 2026)** [snippet]
- Feed-forward. A VLM first normalizes the input to a front-facing T-pose image, then pattern features are extracted. Trained on >25k samples, 12 garment classes.
- Reports panel accuracy 94.35% vs Sewformer 28.81%. Weights not confirmed.
- — [project](https://dresswild.github.io/); [arXiv](https://arxiv.org/pdf/2602.16502)

**Image2Garment (arXiv 2601.09658, Jan 2026)** [snippet]
- Uses **frozen ChatGarment** for the pattern, drapes on SMPL, then a fine-tuned VLM predicts fabric attributes (FTAG dataset, 16,026 images) and random forests map them to simulator physics parameters. Feed-forward, seconds per garment.
- — [arXiv](https://arxiv.org/html/2601.09658v1); [project](https://image2garment.github.io/)

**PatternGSL (arXiv 2606.24564, Jun 2026)** [snippet]
- Template-free structured spec language for panels, curves and stitches. A VLM predicts the spec from a single image (300K-sample PatternGSLData) and deterministic rules decode it into simulatable garments. Covers 2–37 panels, arbitrary topologies; claims in-the-wild generalization. Code/weights not confirmed.
- — [arXiv html](https://arxiv.org/html/2606.24564v3)

**GarmentWeaver (arXiv 2608.30550, Aug 31 2026; Lu, Luo, Zhong)** [snippet]
- Two-stage generation: structure template first, then parameter fill, on a pretrained VLM with feasibility-aware regularization. Inputs: sketch + text. Code not confirmed.
- — [arXiv](https://arxiv.org/abs/2608.30550)

**Other 2026 items seen only as titles in search results (not investigated):**
- SwiftTailor (arXiv 2603.19053, geometry-image garments);
- "Learning Sewing Patterns via Latent Flow Matching of Implicit Fields" (2601.17740);
- Stitched Embeddings (2607.00829);
- Garment Particles (2605.26391);
- SewFusion (2609.23548);
- EasyFashion human-AI co-creation (2609.18483).
— [search results](https://arxiv.org/pdf/2603.19053), [2607.00829](https://arxiv.org/pdf/2607.00829), [2605.26391](https://arxiv.org/pdf/2605.26391), [2609.23548](https://arxiv.org/html/2609.23548), [2609.18483](https://arxiv.org/pdf/2609.18483)

### Inferences
- **VRAM:**
  - ChatGarment and AIpparel are LLaVA-1.5-7B derivatives at bf16, about 14 GB of weights, so inference should fit 24 GB.
  - Design2GarmentCode's local part is only Qwen2-VL-2B; the heavy reasoning goes to an API.
  - SewingLDM is a PixArt-based LDM, likely well under 24 GB.
  - None of these figures were measured or stated by the authors in the fetched READMEs.
- **Licensing for portfolio use:**
  - Code: GarmentCode (MIT), Design2GarmentCode (MIT), ChatGarment (Apache-2.0), SewingLDM (Apache-2.0).
  - Weights inherit base-model terms (LLaVA-1.5 / Vicuna → Llama-2 community license; Qwen2-VL-2B is Apache-2.0).
  - ChatGarment's postprocess uses SMPL / TokenHMR, which are typically non-commercial research licenses.
  - AIpparel, Sewformer and DressCode had no root LICENSE file, so treat them as "all rights reserved / research only" unless the HF card says otherwise.
- All of these models are trained on synthetic GarmentCodeData or Korosteleva-2021 data of realistic Western garments on SMPL(-X)-like bodies. Expect weak results on anime turnarounds: flat cel-shading, exaggerated proportions, sailor collars, pleats, capes and frills. Their output design space is bounded by GarmentCode's components; ChatGarment and AIpparel cannot output a component that GarmentCode lacks.
- None of the models takes **multiple views** (front/side/back turnaround) natively. Dress-1-to-3 uses generated multi-views internally. A user-built pipeline could feed all views to a frontier VLM instead.

### Gaps
- Hugging Face model-card licenses for `liulj/sewformer`, `IHe-KaiI/DressCode`, `georgeNakayama/AIpparel` and `liusq/sewingldm` could not be read (huggingface.co blocked).
- No measured VRAM or runtime numbers were found in the READMEs.
- Code/weights status for Dress-1-to-3, GarmentImage, GarmentGPT, NGL-Prompter, PatternGSL, DressWild and GarmentWeaver was unconfirmed because arXiv and project pages were blocked. Re-check GitHub directly.
- No quantitative benchmark on anime/stylized inputs exists in any source found.

---

## 3. Agentic LLM approach without training (frontier LLM writes GarmentCode / pattern JSON with render-and-compare)

### Takeaway
This approach is now validated in the literature, though no paper uses exactly "Claude/GPT writes new pygarment classes in a sim-render-compare loop":
- **NGL-Prompter (2026)** shows a frozen VLM plus a deterministic GarmentCode mapper works training-free and handles multi-layer outfits.
- **Design2GarmentCode (CVPR 2025)** uses GPT-4o as its understanding agent, with a closed loop in which the agent compares the generated design to the input and proposes modifications. Its only trained part is a small 2B projector.
- **ChatGarment and DressCode** use GPT-4o / ChatGPT as front-end interpreters.
- **TailorCoPilot (UIST 2026 submission)** uses a VLM to propose stepwise editable pattern operations with versioned states.

The most robust no-training design is: frontier VLM → structured GarmentCode design parameters (constrained schema) → GarmentCode program/JSON → Warp sim → render → VLM compares with the reference views → revise. The LLM writes new component classes only when the schema lacks a piece.

### Cited Findings
- Design2GarmentCode architecture: the DSL-GA prompts the MMUA, the MMUA extracts features, the DSL-GA synthesizes GarmentCode design configs and garment programs, and the GarmentCode engine executes them. There are "two validation loops": rule-based validation during synthesis, and "after the initial generation, the MMUA compares the generated design with the input and suggests modifications to minimize discrepancies." — [project page source](https://raw.githubusercontent.com/style3d/Design2GarmentCode/master/index.html)
- The released D2G code uses GPT-4o by default for the MMUA, with a configurable `base_url` and `model`. Users can iteratively edit with `modify: <instruction>`. — [D2G impl README](https://raw.githubusercontent.com/Style3D/design2garmentcode-impl/main/README.md)
- D2G "requires only minimal fine-tuning of a pre-trained LLM and the training of a lightweight, text-conditioned transformer decoder". [snippet of CVPR paper] — [CVPR paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Zhou_Design2GarmentCode_Turning_Design_Concepts_to_Tangible_Garments_Through_Program_Synthesis_CVPR_2025_paper.pdf)
- NGL-Prompter: "fully training-free inference pipeline, in which a pre-trained VLM extracts NGL specifications from an input image, and a deterministic parser converts them into GarmentCode parameters". The motivation: prior VLM fine-tuning on synthetic data limits generalization to real images. [snippet] — [arXiv html](https://arxiv.org/html/2602.20700); [author page source](https://raw.githubusercontent.com/otaheri/otaheri.github.io/main/index.html)
- ChatGarment text generation and editing: "Utilizes GPT-4o to generate well-formed text descriptions … Sends the GPT-generated text to ChatGarment Model." — [ChatGarment README](https://raw.githubusercontent.com/biansy000/ChatGarment/main/README.md)
- DressCode supports "ChatGPT as an LLM interpreter for interactively customized garment generation" (`--GPT`). — [DressCode README](https://raw.githubusercontent.com/IHe-KaiI/DressCode/main/README.md)
- TailorCoPilot (UIST 2026 submission, [snippet]): given natural-language intent, it retrieves a base pattern from a library. A VLM, reasoning with textbook expert knowledge and operation traces, proposes stepwise editable operations such as `SpreadPanel()` and `ExtendPoint()`. The TailorTrace backend records states and operations in a branching version history. It uses a curated set of 93 design traces (56 textbook, 37 industry). — [arXiv html](https://arxiv.org/html/2608.25462)
- GarmentGPT's authors argue that VLMs "struggle with low-level regression of raw floating-point coordinates". This supports having LLMs emit semantic parameters or programs rather than raw vertex coordinates. [snippet] — [OpenReview](https://openreview.net/forum?id=XzXKnazRBF)
- PatternGSL likewise uses a VLM to emit a structured spec, then "lightweight deterministic rules" produce valid garments. [snippet] — [arXiv html](https://arxiv.org/html/2606.24564v3)

### Inferences
- **Reported success rates:** I found no published success rate for a pure frontier-LLM (Claude/GPT/Gemini) loop that authors new GarmentCode classes. D2G's and NGL-Prompter's numeric results could not be read (arXiv blocked). Report this as unknown.
- **Recommended loop design:**
  - (a) Give the LLM the GarmentCode design-YAML schema (enumerated classes and ranges) plus the body-measurement YAML. Constraining to discrete or enumerated choices is what made NGL-Prompter work.
  - (b) Validate by executing pygarment (it raises errors such as `TotalLengthError`).
  - (c) Simulate with Warp on the user's own body, converted into a GarmentCode body + measurement YAML.
  - (d) Render front, side and back at the same camera as the turnaround.
  - (e) Have the VLM diff the renders against the reference and output parameter deltas. Optionally add a silhouette IoU/mask loss, as in ChatGarment's SAM-mask finite-difference postprocess, for numeric length/width refinement.
  - (f) Escalate to code-writing (new Panel/Component subclasses) only for missing anime parts, kept in a reusable component library.
- D2G's configurable `base_url` / `model` suggests its MMUA could be pointed at another OpenAI-compatible endpoint (e.g., a local Qwen-VL server, or a provider's OpenAI-compatible API) without retraining. This is untested.

### Gaps
- No quantitative success or failure rates for LLM-authored garment programs were accessible.
- No source evaluated multi-view (turnaround) input to an LLM for pattern estimation.
- No source evaluated on anime/manga art.

---

## 4. Commercial / industry tools (image→pattern or auto-pattern)

### Takeaway
Commercial "AI pattern" features remain narrow:
- Marvelous Designer 2025.1's AI Pattern Drafter (Beta) generates **T-shirt-only** drafts from text or flat sketches.
- Style3D markets image/sketch/text→pattern AI, but the only sources found are its own blog posts (marketing; unverified).
- Seamly2D/Valentina are open-source parametric drafting CADs (GPLv3), not AI.

None of these is a turnkey anime-turnaround→pattern solution.

### Cited Findings
- Marvelous Designer 2025.1 adds "Pattern Drafter (Beta)", which generates a shirt pattern from body measurements, keywords or flat sketches. The "AI Pattern Drafter" generates drafts from text prompts or uploaded schematic sketches, "currently limited to T-shirts". It also adds an AI Pose Generator (Beta) and drawing patterns directly on a 3D avatar. MD 2024.1 added an AI Texture and AI Graphic Generator. [snippet] — [CG Channel MD 2025.1](https://www.cgchannel.com/2025/08/clo-virtual-fashion-releases-marvelous-designer-2025-1/); [80.lv](https://80.lv/articles/marvelous-designer-2025-1-now-available); [Digital Production](https://digitalproduction.com/2025/08/21/marvelous-designer-2025-1-draw-wash-repeat/)
- Style3D's own blog claims "Style3D AI converts sketches, text, or images into precise 2D patterns linked with 3D models and stitching data", with auto-stitching, grading and "70%" prototyping-time reduction. This is vendor marketing and self-rated. [snippet] — [Style3D blog](https://www.style3d.ai/blog/best-ai-tool-for-sewing-patterns/); [Style3D blog 2](https://www.style3d.ai/blog/what-are-the-best-ai-tools-for-pattern-making/)
- Style3D Research co-authored Design2GarmentCode (Huamin Wang). The project page shows sketch-generated patterns integrating "seamlessly with industrial fashion design software" for pattern editing, which suggests D2G-like tech underlies Style3D's AI. — [D2G project page source](https://raw.githubusercontent.com/style3d/Design2GarmentCode/master/index.html)
- Seamly2D is GPLv3+, forked from Valentina; both are parametric pattern-drafting CAD. — [LinuxLinks](https://www.linuxlinks.com/seamly2d-pattern-making/); [Wikipedia](https://en.wikipedia.org/wiki/Valentina_(software))
- Tailornova appears in comparison lists (score 6.5/10 SMB). It is a low-quality aggregator source and I did not research it further. — [WorldMetrics](https://worldmetrics.org/best/dress-pattern-making-software/)

### Inferences
- Marvelous Designer remains valuable as the **downstream simulator and editor**: import patterns, fix pleats, sew and export to UE5. The AI generation front-end is better built from GarmentCode plus an LLM.
- Browzwear's AI pattern features were not researched (no sources fetched).

### Gaps
- No independent review of Style3D AI's image→pattern quality was found.
- Browzwear, CLO 3D (as distinct from MD) and Tailornova AI features are unverified.
- No public "pattern-making API" was found.

---

## 5. Known failure modes and corrections (sizes, fashion bias, SMPL assumption, anime pieces)

### Takeaway
Documented failures:
- wrong lengths and widths (ChatGarment's own README);
- ignoring body shape, causing sagging waists (Sewformer, per D2G);
- missing or extra panels and imaginary stitches (Sewformer, per D2G);
- single-layer focus (per NGL-Prompter);
- SMPL-body assumptions in draping and postprocessing;
- vertex-coordinate regression weakness of VLMs (per GarmentGPT).

Fixes: hard-constrain with profile measurements in the GarmentCode body YAML, express lengths as body-relative parameters, run a silhouette-matching refinement, and expose the design YAML to the user for human-in-the-loop edits through the GarmentCode GUI or D2G's `modify:` command.

### Cited Findings
- "ChatGarment may occasionally produce garments with incorrect lengths or widths from input images." The fix is finite-difference length/width refinement against Grounding-SAM masks, with TokenHMR (SMPL) body estimation. — [ChatGarment README](https://raw.githubusercontent.com/biansy000/ChatGarment/main/README.md); [postprocess.md](https://raw.githubusercontent.com/biansy000/ChatGarment/main/docs/postprocess.md)
- Sewformer ignores body shape, so skirts and pants are oversized at the waist and sag. It also produces wrong necklines, missing components, imaginary stitches and extraneous panels. — [D2G project page source](https://raw.githubusercontent.com/style3d/Design2GarmentCode/master/index.html)
- Sewformer simulation needs SMPL predictions from RSC-Net. Image2Garment drapes ChatGarment output on a canonical SMPL body. — [Sewformer README](https://raw.githubusercontent.com/sail-sg/sewformer/main/ReadMe.md); [Image2Garment](https://arxiv.org/html/2601.09658v1) [snippet]
- GarmentCode fits patterns to a body-measurement YAML and errors on invalid totals such as floor-length overflow. v2 improved parameter ranges and dependencies to reduce invalid combinations. — [meta_garment.py](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/assets/garment_programs/meta_garment.py); [CHANGELOG](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/CHANGELOG.md)
- Body measurements for GarmentCode are documented in `docs/Body Measurements GarmentCode.pdf` and in the companion repo `mbotsch/GarmentMeasurements`. — [GarmentCode README](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/ReadMe.md)
- Prior methods focus mostly on single-layer garments; NGL-Prompter claims multi-layer recovery. [snippet] — [arXiv](https://arxiv.org/abs/2602.20700)
- DressWild identifies pose and viewpoint variation as a key failure source for Sewformer-style models and uses VLM pose normalization. [snippet] — [project](https://dresswild.github.io/)

### Inferences
- **Anime-specific risks:**
  - exaggerated proportions (long legs, small waist) clash with SMPL-like priors, so use the user's own body measurements, not estimated ones;
  - cel-shaded art lacks the shading cues that diffusion- and normal-based methods (Dress-1-to-3) rely on;
  - stylized pieces (sailor collar, pleated skirt, cape, layered frills, ribbons) are absent from all training sets and from GarmentCode's component library.
- **Mitigations:**
  - Treat the clothes profile as ground truth for lengths and circumferences (skirt length, sleeve length, waist ease) and let the image only choose topology and style.
  - Lock profile-derived parameters during the LLM revise loop.
  - Use per-view silhouette IoU for remaining length tuning.
  - Keep a human-editable YAML. The GarmentCode NiceGUI configurator gives sliders for every parameter.
- Since the user's body is a custom anime mesh, they must create a GarmentCode body (mesh + measurements YAML) for it; the measurement definitions are in the Body Measurements PDF. Drape there, or export the pattern to MD and drape on the real avatar.

### Gaps
- No source quantifies failure rates on stylized or anime inputs.
- I did not check whether GarmentCode's Warp sim handles non-SMPL custom meshes without its body-segmentation labels. The GarmentCodeData pipeline uses labelled bodies, so verify this.

---

## 6. Recommendation (best no-training approach for this user, late 2026)

### Takeaway
Build an **agentic GarmentCode pipeline**:
1. A frontier multimodal LLM reads the turnaround images plus the clothes profile.
2. It emits GarmentCode design parameters in an NGL-like constrained vocabulary.
3. It writes new pygarment component classes only for anime pieces.
4. GarmentCode executes and the Warp sim drapes on the user's own body.
5. Render front, side and back, compare with the VLM, and revise.
6. Export the pattern (SVG/JSON → DXF converter) or the draped mesh to Marvelous Designer, then UE5.

Use the released models as optional "initial guess" generators: Design2GarmentCode (MIT, GPT-4o-driven with a feedback loop), ChatGarment (Apache-2.0, local 7B) and AIpparel (local 7B, license unclear). Do not depend on them for anime accuracy.

### Cited Findings
- Training-free VLM→GarmentCode is published and claims real-world generalization and multi-layer support (NGL-Prompter). [snippet] — [arXiv](https://arxiv.org/abs/2602.20700)
- A VLM compare-and-revise loop over GarmentCode programs is published and released under MIT (Design2GarmentCode). — [D2G impl](https://raw.githubusercontent.com/Style3D/design2garmentcode-impl/main/README.md); [project source](https://raw.githubusercontent.com/style3d/Design2GarmentCode/master/index.html)
- GarmentCode is extensible by subclassing pygarment Panel/Component and composing via interfaces and stitching rules (MIT). — [meta_garment.py](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/assets/garment_programs/meta_garment.py); [LICENSE](https://raw.githubusercontent.com/maria-korosteleva/GarmentCode/main/LICENSE)
- DressCode notes its outputs can be loaded into Marvelous Designer for further simulation and animation. This precedent supports using MD downstream. — [DressCode README](https://raw.githubusercontent.com/IHe-KaiI/DressCode/main/README.md)

### Inferences
- **Priority order to try:**
  1. Clone `Style3D/design2garmentcode-impl` (MIT). It already has the GUI, GarmentCode, the Warp sim and an LLM loop. Swap or extend the MMUA prompt for anime and multi-view input.
  2. Add an NGL-style constrained parameter schema and profile-locked measurements.
  3. Grow a custom anime component library in pygarment: pleated skirt, sailor collar, cape, frill strips, layered tiers.
  4. Use ChatGarment or AIpparel locally on the front view only as a second-opinion initializer.
  5. Use the SAM-mask silhouette refinement idea from ChatGarment for final sizing.
- Keep MD/Style3D for pleat folding, layering collisions and the final UE5 export. The open Warp sim is adequate for loop feedback but likely not final quality.

### Gaps
- No head-to-head evaluation of a frontier LLM loop vs. ChatGarment/AIpparel on stylized art exists.
- Licensing of AIpparel, Sewformer and DressCode weights is unclear.
- NGL-Prompter's code release is unconfirmed as of research date.
