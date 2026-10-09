# Making age-variant 2D blueprints of the same character (HeroBody is 2D-first)

Research date: 2026-10-09. Researcher notes for the report writer.

**Access notes.** This session had almost no open web.
- **WebSearch was unavailable.** The shared per-turn search budget (200 calls) was already used up by the other researchers before this track started. I ran **zero** searches, so this file has **no [snippet] claims**.
- WebFetch and curl failed (DNS error or HTTP 000) for arxiv.org, export.arxiv.org, huggingface.co, openaccess.thecvf.com, proceedings.bmvc2023.org, semanticscholar.org, civitai.com, replicate.com and ai.google.dev. The GitHub REST API (`gh api`) returned 403 ("GitHub access to this repository is not enabled for this session").
- **raw.githubusercontent.com worked**, so every [fetched] claim below comes from a README, LICENSE, config or source file in an official GitHub repository. I fetched:
  - Face ageing: FADING (README, `age_editing.py`, `FADING_util/util.py`, `null_inversion.py`), SAM (README, LICENSE), HRFAE (README, LICENSE.txt), CUSP (README).
  - Multi-view and characters: CharacterGen (README, LICENSE, `2D_Stage/configs/infer.yaml`, `2D_Stage/webui.py`, `material/pose.json`, `pose0-3.png`), MV-Adapter (README, LICENSE, `scripts/inference_tg2mv_sdxl.py`), StdGEN (README, LICENSE), Zero123++ (README, LICENSE), Era3D (README), PSHuman (README, LICENSE).
  - Identity, control and pose: PuLID, InstantID, IP-Adapter (README, LICENSE each), ControlNet 1.1 (README, `annotator/openpose/util.py`), comfyui_controlnet_aux, DWPose (LICENSE), Champ (README, LICENSE).
  - Depth, editing and CPU tools: MoGe (README, LICENSE), Qwen-Image (README), stable-diffusion.cpp (README, LICENSE), FastSD CPU (Readme, LICENSE), StableGen (README).
- **Not fetched, because their primary pages are on arXiv or Hugging Face:** AgeBooth (arXiv 2510.05715) and TimeMachine (2025). Claims about them are tagged **[prior knowledge, unverified]**: from my training data, not checked this session. Treat them as leads.
- I reused numbers from the sibling note `proportions_by_age.md` (same folder, same date) for the age tables. They are tagged **[sibling note]** and their primary sources are listed there.
- Tags: **[fetched]** = I read the primary file. **[computed]** = I computed it from fetched or sibling-note numbers with the script described. **[inference]** = my reasoning. **[prior knowledge, unverified]** = see above.

---

## 1. Identity-preserving age editing for faces (AgeBooth, FADING, TimeMachine, SAM, HRFAE, CUSP, newer)

### Takeaway
- **Every open face-ageing method is trained on real photographs.** The training sets are FFHQ, FFHQ-Aging and IMDB-WIKI-derived age labels. None was trained on anime or manga. None reports results on cel-shaded faces.
- **Use them only as a reference for realistic age cues** (skin, nasolabial folds, jowls, hairline) on a photoreal render or photo. Do not use them to make HeroBody blueprint faces. HeroBody's anime head module (drawn features as decals) should age the face **parametrically**, with eye size, face length and jaw in head-radius units per age (see `head_face_teeth_hair_by_age.md`). Manga images override it.
- **FADING is the most usable open one.** It is Stable-Diffusion-based. It needs only an input image, its true age and a target-age list. It falls back to CPU in code, although that is slow. It has **no LICENSE file**, which by default means all rights are reserved.
- **Licences:** SAM is MIT, but its FFHQ StyleGAN and auxiliary models carry their own terms. HRFAE is an InterDigital evaluation licence (research only). CUSP is forked from StyleGAN2-ADA-pytorch and has no licence of its own in the README.
- Every method needs an NVIDIA GPU in practice except FADING. FADING runs on CPU, but null-text inversion is about 500 optimisation passes through the UNet.

### Cited Findings
**FADING (BMVC 2023, "Face Aging via Diffusion-based Editing")**
- The official repo uses arXiv 2309.11321. It trains on FFHQ-Aging. A "specialization" fine-tune runs for `--max_train_steps 150` on age-labelled images (`training_ages.npy`: filename plus age). Released weights are on Google Drive. — [FADING README](https://github.com/MunchkinChen/FADING) [fetched]
- Inference arguments are `--image_path --age_init --gender {female,male} --specialized_path --target_ages` (default `10 20 40 60 80`). — [age_editing.py](https://raw.githubusercontent.com/MunchkinChen/FADING/master/age_editing.py) [fetched]
- **Method parameters (code-ready):**
  - `StableDiffusionPipeline` with `DDIMScheduler(beta_start=0.00085, beta_end=0.012, scaled_linear)`.
  - Null-text inversion with the prompt `"photo of {age} year old {man|woman|boy|girl}"`.
  - Prompt-to-prompt replace controller with `cross_replace_steps 0.8` and `self_replace_steps 0.5`.
  - The age token is re-weighted to 1.
  - `NUM_DDIM_STEPS = 50`, `GUIDANCE_SCALE = 7.5`.
  - `device = cuda if available else cpu`.
  - — [age_editing.py](https://raw.githubusercontent.com/MunchkinChen/FADING/master/age_editing.py), [null_inversion.py](https://raw.githubusercontent.com/MunchkinChen/FADING/master/null_inversion.py) [fetched]
- The person word switches at 15 y: `age <= 15 -> boy/girl`, otherwise `man/woman`. — [FADING_util/util.py](https://raw.githubusercontent.com/MunchkinChen/FADING/master/FADING_util/util.py) [fetched]
- No LICENSE, LICENSE.txt or LICENSE.md exists on either the main or master branch (all 404). — [FADING repo](https://github.com/MunchkinChen/FADING) [fetched]

**SAM (SIGGRAPH 2021, "Only a Matter of Style")**
- It is a StyleGAN2 (FFHQ, 1024×1024) regression model built on a pSp encoder. The ID loss uses IR-SE50 and the age loss uses a DEX VGG classifier fine-tuned on FFHQ-Aging. — [SAM README](https://github.com/yuval-alaluf/SAM) [fetched]
- Prerequisites are an "NVIDIA GPU + CUDA CuDNN (CPU may be possible with some modifications, but is not inherently supported)". It needs dlib's 68-landmark predictor for alignment, and that predictor fails on anime faces [inference]. — [SAM README](https://github.com/yuval-alaluf/SAM) [fetched]
- The code licence is MIT. — [SAM LICENSE](https://raw.githubusercontent.com/yuval-alaluf/SAM/master/LICENSE) [fetched]

**HRFAE (High Resolution Face Age Editing, 2020)**
- It needs PyTorch 1.1 and Python 3.7. The age classifier is DEX trained on IMDB-WIKI. It trains on FFHQ with age labels. Inference is `python test.py --config 001 --target_age 65`. — [HRFAE README](https://github.com/InterDigitalInc/HRFAE) [fetched]
- **The licence is a Limited Software Evaluation License.** Authorized purpose is "research on the Software and evaluation of the Software exclusively, and academic research ... without any commercial use". — [HRFAE LICENSE.txt](https://raw.githubusercontent.com/InterDigitalInc/HRFAE/master/LICENSE.txt) [fetched]

**CUSP (ECCV 2022, Custom Structure Preservation in Face Aging)**
- Models are trained on FFHQ-RR and FFHQ-LS (FFHQ-Aging). FFHQ-LS age bins are 0-2, 3-6, 7-9, 10-14, 15-19, 20-29, 30-39, 40-49, 50-69 and 70-120 y.
- It runs in Docker with `--gpus all`. The base code is forked from NVlabs StyleGAN2-ADA-pytorch. No licence file was found on main or master.
- — [CUSP README](https://github.com/guillermogotre/CUSP) [fetched]

**AgeBooth (arXiv 2510.05715, Oct 2025) [prior knowledge, unverified]**
- I believe it is a diffusion-based, identity-personalised ageing method. It trains age-specific LoRAs (for example young and old). At inference it fuses them with an SVD-based weight-mixing scheme and prompt blending, which gives intermediate ages without per-subject training.
- I could not verify the code release, the licence or the base model (it may be SDXL plus an identity adapter). The report writer should check the arXiv page.

**TimeMachine (2025) [prior knowledge, unverified]**
- I believe it is a fine-grained facial age editor built on a Stable Diffusion backbone. It uses age-classifier guidance and multi-cross-attention conditioning, and a new high-quality, fine-grained age-labelled face dataset. Not verified.

**Other related work [prior knowledge, unverified]**
- MyTimeMachine (2024/25) personalises ageing from about 50 photos of one person. It is the closest conceptual match to the user's "per-character learning", but it targets real photos.
- Cradle2Cane (2025) is a two-pass diffusion ageing method for very large age gaps.
- Not verified this session.

**Face ageing methods at a glance**

| method | backbone | trained on | ages handled | code licence | GPU need | anime |
|---|---|---|---|---|---|---|
| FADING | SD (DreamBooth-style specialised) + null-text inversion + P2P | FFHQ-Aging | prompt-driven, default 10-80 | none in repo | runs on CPU by code; GPU practical | not reported [inference: domain gap] |
| SAM | StyleGAN2 FFHQ 1024 + pSp | FFHQ + DEX ages | 0-100 (classifier range) | MIT (code) | CUDA required as shipped | no (dlib alignment, FFHQ latent space) |
| HRFAE | encoder-decoder GAN, 1024 | FFHQ-RR + DEX | target_age param (20-69 range) | evaluation-only | CUDA | no |
| CUSP | StyleGAN2-ADA-based | FFHQ-RR / FFHQ-LS | 10 bins 0-120 | not stated | CUDA (Docker) | no |
| AgeBooth | diffusion + age LoRAs | unverified | unverified | unverified | GPU [inference] | unverified |
| TimeMachine | SD + age guidance | unverified | unverified | unverified | GPU [inference] | unverified |

Sources: FADING, SAM, HRFAE and CUSP rows [fetched]; the HRFAE age range is [prior knowledge, unverified]; AgeBooth and TimeMachine rows [prior knowledge, unverified].

### Inferences
- **The domain gap is the deciding factor.** Each of these models encodes age as photographic texture: wrinkles, skin, fat pads and lighting. In anime, age is mainly carried by geometry that HeroBody already parameterises:
  - head-to-body ratio
  - eye size relative to head
  - face length (forehead to chin)
  - jaw angle
  - nose and mouth placement
  - line-work conventions (e.g. a few crease lines for old age)
- Running a photo ager on a cel-shaded face either does almost nothing (the inversion stays in the photo manifold) or photorealises it.
- **A safe use:** render HeroBody's realistic head at the target age, or use a reference photo, and run FADING on it to get age-cue references (where the folds go and how much the hairline recedes). Then translate those cues into anime decal or line parameters by hand or in a rule table.
- **Identity metrics:** ArcFace/IR-SE50-type ID losses are trained on real faces and are unreliable on anime faces. For anime identity checks across ages, compare the parametric face spec (eye shape class, eye colour, hair silhouette, scar or decal positions in head-radius units). Image embeddings (CLIP/DINO face-crop similarity) are only a soft signal. No anime age-invariant identity metric was found.

### Gaps
- AgeBooth and TimeMachine facts (code, licence, VRAM, base model) are unverified because arXiv and Hugging Face were blocked and there was no search budget.
- No published face-ageing method trained on or evaluated on anime or manga was found. The search could not be run, so this is "not found", not "does not exist".
- No VRAM figures were found in the fetched READMEs for FADING, SAM, HRFAE or CUSP.

---

## 2. Full-body age transformation in images (proportions change, identity, outfit and pose kept)

### Takeaway
- **No fetched source offers a full-body "age this character" model.** The working practice is compositional:
  1. Fix the body geometry with a control signal: pose skeleton, depth, normal, lineart or silhouette.
  2. Fix identity with a reference adapter (IP-Adapter, PuLID, InstantID) or a character LoRA.
  3. Let the prompt carry age words.
- For HeroBody the **body geometry must come from HeroBody itself**. Render the age-correct 3D body solved from numbers, not an adult OpenPose skeleton. Skeleton-only control (OpenPose) fixes joint positions. It does not fix head size, limb thickness or the torso silhouette, which are the main age signals.
- **The identity adapters target real faces.** InstantID and PuLID both depend on InsightFace face embeddings. InstantID's code is Apache-2.0, but the InsightFace antelopev2 models are non-commercial research only. For anime characters, a **character LoRA**, or IP-Adapter's general image prompt, is the better identity carrier [inference].
- The most precise published precedent for "parametric body render → image" is **Champ**. It conditions generation on depth, normal and semantic maps rendered from SMPL, plus DWPose skeletons. Code is MIT.

### Cited Findings
- **ControlNet 1.1 SD1.5 models:** canny, mlsd, depth, normalbae, seg, inpaint, lineart, **lineart_anime** (`control_v11p_sd15s2_lineart_anime`, which "can take real anime line drawings or extracted line drawings as inputs"), **openpose**, scribble, softedge, shuffle, ip2p and tile. — [ControlNet-v1-1-nightly README](https://github.com/lllyasviel/ControlNet-v1-1-nightly) [fetched]
- **OpenPose stick rendering used by ControlNet (code-ready):**
  - Format: COCO-18 keypoints; stick width 4 px; 17 limbs, each drawn as a filled ellipse in its own colour; canvas multiplied by 0.6, then joint dots drawn.
  - `limbSeq = [[2,3],[2,6],[3,4],[4,5],[6,7],[7,8],[2,9],[9,10],[10,11],[2,12],[12,13],[13,14],[2,1],[1,15],[15,17],[1,16],[16,18],[3,17],[6,18]]` (1-based).
  - The colour list starts `[255,0,0],[255,85,0],[255,170,0],...`.
  - — [annotator/openpose/util.py](https://raw.githubusercontent.com/lllyasviel/ControlNet-v1-1-nightly/main/annotator/openpose/util.py) [fetched]
  - So HeroBody can draw an exact ControlNet-compatible skeleton from its own rig joints projected orthographically, with no pose detector [inference].
- comfyui_controlnet_aux ships `lineart_anime`, `lineart_anime_denoise` ("Manga Lineart"), DWPose (`dw_openpose_full`) and OpenPose preprocessors. It can export OpenPose-format JSON. — [comfyui_controlnet_aux README](https://github.com/Fannovel16/comfyui_controlnet_aux) [fetched]. DWPose code is Apache-2.0. — [DWPose LICENSE](https://raw.githubusercontent.com/IDEA-Research/DWPose/main/LICENSE) [fetched]
- **IP-Adapter:**
  - Apache-2.0 code, with SD1.5 and SDXL image-prompt adapters and a ControlNet-combined demo (`ip_adapter_sdxl_controlnet_demo`).
  - FaceID variants (incl. FaceID-PlusV2 for SDXL) are listed as experimental.
  - The SDXL version was retrained with OpenCLIP ViT-H-14 instead of ViT-bigG-14.
  - — [IP-Adapter README](https://github.com/tencent-ailab/IP-Adapter), [LICENSE](https://raw.githubusercontent.com/tencent-ailab/IP-Adapter/main/LICENSE) [fetched]
- **InstantID:**
  - Code is Apache-2.0 "for both academic and commercial usage". However, "both manual-downloading and auto-downloading face models from insightface are for non-commercial research".
  - It uses `FaceAnalysis(name='antelopev2', providers=['CUDAExecutionProvider','CPUExecutionProvider'])`.
  - It supports `pipe.enable_model_cpu_offload()` and LCM-LoRA.
  - The authors claim that "in non-realistic style, our work is more flexible on the integration of face and background" than InsightFace swappers.
  - — [InstantID README](https://github.com/instantX-research/InstantID) [fetched]
- **PuLID:**
  - Apache-2.0. Models: PuLID-v1 and v1.1 (SDXL) and PuLID-FLUX v0.9.0/v0.9.1. v0.9.1 is "about 5 percentage points" better in ID similarity than v0.9.0.
  - PuLID-FLUX "can run on a 16GB graphic card", and the local gradio demo supports 12 GB with fp8.
  - — [PuLID README](https://github.com/ToTheBeginning/PuLID), [LICENSE](https://raw.githubusercontent.com/ToTheBeginning/PuLID/main/LICENSE) [fetched]
- **Champ (Controllable and Consistent Human Image Animation with 3D Parametric Guidance):**
  - Guidance encoders exist for depth, normal and semantic_map, all rendered from SMPL, plus DWPose.
  - It ships SMPL and Blender rendering scripts. It was tested on A100 and RTX3090. About 250 frames need about 20 GB VRAM.
  - MIT licence.
  - — [Champ README](https://github.com/fudan-generative-vision/champ), [LICENSE](https://raw.githubusercontent.com/fudan-generative-vision/champ/master/LICENSE) [fetched]
- **Qwen-Image:**
  - A 20B MMDiT, Apache-2.0. Qwen-Image-Edit-2509 adds multi-image input. Qwen-Image-Edit-2511 (released 2025-12-23) has "significantly improved" character consistency and "stronger geometric reasoning".
  - — [Qwen-Image README](https://github.com/QwenLM/Qwen-Image) [fetched]
  - A sibling note records that 2509 natively accepts keypoint, depth and edge maps (from the HF model card). — `Clothes in manga to anime pipeline/generative_ai_costume_consistency.md` [sibling note]

### Inferences
- **Why skeleton-only control is not enough for ageing.** Take a 6 y target with head height 0.162 H (real median) against an adult 0.127 H (sibling Table 6a). The head is 28% larger relative to the body. Leg length changes only from 0.476 to 0.459 H.
  - An OpenPose skeleton encodes nose, eye and ear positions but not skull size, so the model will often draw an adult-sized head on the child skeleton.
  - **Depth, normal or silhouette control** from the 3D body encodes head volume, limb girth and the round child torso directly.
  - Use **OpenPose (from rig joints) + normal or depth (from mesh) + lineart (from Blender Line Art)** together.
- **Identity across age in anime.** Identity is mostly carried by hair silhouette and colour, eye shape and colour, signature marks (Luffy's scar under the eye, his straw hat) and outfit.
  - A **per-character LoRA** trained on the approved adult turnaround plus any manga images of that character at other ages captures this better than face-ID adapters.
  - Add the age as a caption token, for example `age_7`, with the stage name.
  - Expect LoRA bleed of adult proportions. The strong geometry control from the 3D render is what overrides it.
- **Outfit.** Outfits should resize with the body (HeroBody has separate garments). In images, dress the age-correct body render with an image editor or IP-Adapter, as the sibling clothes notes recommend for adult renders. This is the same trick with an age-correct render in place of the adult one.

### Gaps
- No fetched paper or tool reports **body-proportion accuracy** (e.g. limb-length error in % of height) for ControlNet, Champ or edit models. Geometric fidelity numbers must be measured in-house with `hb/measure2d.py`.
- No anime-specific full-body ageing dataset or model was found (search unavailable).

---

## 3. Consistency across views for generated blueprints (turnarounds)

### Takeaway
- **MV-Adapter's geometry-conditioned mode is the closest off-the-shelf match** for HeroBody's front, side, back and top views. Its text-geometry and image-geometry pipelines take **a mesh** and render **6 orthographic views**:
  - 4 around the body at elevation 0 (azimuths 0, 90, 180 and 270 deg),
  - 1 top (elevation +89.99),
  - 1 bottom (elevation −89.99).
- Those renders are 6-channel position+normal control images. The model generates 6 consistent images at 768×768. It accepts personalised SDXL bases, including **Animagine XL 3.1**, and LoRAs. Licence is Apache-2.0.
- CharacterGen (4 A-pose views, anime-trained, SD2.1, Apache-2.0) and StdGEN (anime, A-pose canonicalisation plus multi-view, Apache-2.0 code, research-only checkpoints) generate their **own** body proportions from a single image. They cannot take HeroBody's age-correct body as input. CharacterGen does condition on rendered skeleton images, which could in principle be swapped for age-correct ones [inference].
- **None of these generate at HeroBody's required ≥1536 px body height.** MV-Adapter outputs at most 768. CharacterGen outputs 512×768 with the body spanning about 480 px. So an upscale-and-correct step is mandatory.

### Cited Findings
**MV-Adapter (ICCV 2025)**
- It is a plug-and-play adapter that turns text-to-image models into multi-view generators. It works with "personalized models", "distilled models (e.g. LCM)" and "extensions (e.g. ControlNet)", at "768 Resolution using SDXL". — [MV-Adapter README](https://github.com/huanngzh/MV-Adapter) [fetched]
- The anime base model is demonstrated with `--base_model "cagliostrolab/animagine-xl-3.1"`. Multiple LoRAs have been supported in text-to-multiview since 2025-06-14. — [MV-Adapter README](https://github.com/huanngzh/MV-Adapter) [fetched]
- **VRAM:** image-to-multiview is the heaviest at "about 14G GPU memory". Text-geometry SDXL needs ">12G" and SD2.1 "<6G". Image-geometry SDXL needs ">16G" and SD2.1 "<10G". SD2.1 geometry adapters were released 2025-06-13. Texture generation needs CV-CUDA. — [MV-Adapter README](https://github.com/huanngzh/MV-Adapter) [fetched]
- **Geometry-conditioned camera set (code-ready):**
  ```
  get_orthogonal_camera(elevation_deg=[0,0,0,0,89.99,-89.99], distance=[1.8]*6,
                        left=-0.55, right=0.55, azimuth_deg=[x-90 for x in [0,90,180,270,180,180]])
  ```
  - The mesh is loaded with `rescale=True`. The control image is `cat([(pos+0.5).clamp(0,1), (normal/2+0.5).clamp(0,1)], -1)`, i.e. 6 channels.
  - Other settings: `height=width=768`, `num_inference_steps=50`, `guidance_scale=7.0`, `control_conditioning_scale=1.0`.
  - Scheduler: `ShiftSNRScheduler(shift_mode="interpolated", shift_scale=8.0)`. Rasteriser: nvdiffrast (CUDA).
  - — [scripts/inference_tg2mv_sdxl.py](https://raw.githubusercontent.com/huanngzh/MV-Adapter/main/scripts/inference_tg2mv_sdxl.py) [fetched]
- The README warns that mesh orientation must match the example, "otherwise, you need to adjust the angles in the scripts". The partial-image mode writes a `*_transform.json` (offset and scale) for mapping images back to the mesh. — [MV-Adapter README](https://github.com/huanngzh/MV-Adapter) [fetched]
- Code licence is Apache-2.0. — [MV-Adapter LICENSE](https://raw.githubusercontent.com/huanngzh/MV-Adapter/main/LICENSE) [fetched]

**CharacterGen (SIGGRAPH 2024 / TOG 43(4))**
- It is a 2D multi-view stage plus a 3D stage. The base model is `stabilityai/stable-diffusion-2-1`, with `num_views: 4`, `guidance_scale: 5.0`, `use_pose_guider: True` and `camera_embedding_type: 'e_de_da_sincos'`. — [2D_Stage/configs/infer.yaml](https://raw.githubusercontent.com/zjp-shadow/CharacterGen/main/2D_Stage/configs/infer.yaml) [fetched]
- The input is padded to 2:3 and resized to **512×768**. The default timestep slider is 40 (range 10 to 70). The code hard-codes `"cuda"`. — [2D_Stage/webui.py](https://raw.githubusercontent.com/zjp-shadow/CharacterGen/main/2D_Stage/webui.py) [fetched]
- **Pose conditioning comes from four fixed rendered-skeleton images** (`material/pose0-3.png`, 768×768) with four camera matrices at distance 1.5 (`material/pose.json`). I measured the images:
  - In pose1 and pose3, the A-pose skeleton spans y = 207 to 690 px (about 480 px, 62% of 768) and x = 228 to 539 px. These are the frontal and rear views.
  - pose0 and pose2 are side views about 25 px wide.
  - — [pose.json, pose0-3.png](https://github.com/zjp-shadow/CharacterGen/tree/main/2D_Stage/material) [fetched + computed]
- It was trained on VRoid/VRM anime characters (the Anime3D dataset, which cannot be redistributed; rendering scripts for Blender and three-vrm are provided). Code licence is Apache-2.0. — [CharacterGen README](https://github.com/zjp-shadow/CharacterGen), [LICENSE](https://raw.githubusercontent.com/zjp-shadow/CharacterGen/main/LICENSE) [fetched]

**StdGEN (CVPR 2025)**
- Pipeline: `infer_canonicalize.py` (reference to A-pose), then `infer_multiview.py` (with a `--low_vram` offload option), then S-LRM reconstruction, then multi-layer refinement.
- It needs Python 3.9, torch 2.1.0 and CUDA 11.8, plus the ViT-H SAM checkpoint.
- Tips from the README: you can hand-edit the A-pose image between stages. Full-body input is required. `rm_anime_bg` is weaker on 2.5D or real images.
- "Our released checkpoints are also for research purposes only." Code licence is Apache-2.0.
- — [StdGEN README](https://github.com/hyz317/StdGEN), [LICENSE](https://raw.githubusercontent.com/hyz317/StdGEN/main/LICENSE) [fetched]

**Zero123++**
- Six output views at azimuth 30, 90, 150, 210, 270 and 330 deg relative to the input. Elevations are 20/−10 alternating (v1.2) or 30/−20 (v1.1). FOV is 30 deg. Input should be square and ≥320 px.
- It needs about 5 GB VRAM (about 5.7 GB with the depth ControlNet). It has a depth ControlNet and a normal-generator ControlNet.
- Licence: the model "cannot [be used] in a commercial product pipeline, but you can still use the outputs from the model freely". Code is Apache-2.0.
- — [zero123plus README](https://github.com/SUDO-AI-3D/zero123plus) [fetched]
- Its views are **not** the orthographic front, side, back and top that HeroBody needs [inference].

**Era3D and PSHuman**
- Era3D makes 512 px, 6-view colour and normal images, including an **ortho** checkpoint (`MacLab-Era3D-512-6view-ortho`) aligned to the input front view. **Licence AGPL-3.0.** — [Era3D README](https://github.com/pengHTYX/Era3D) [fetched]
- PSHuman is human-specific. It runs at 768 px with 6 to 7 views and needs ">40GB of VRAM". An SMPL-free version exists. MIT licence. — [PSHuman README](https://github.com/pengHTYX/PSHuman) [fetched]

**Multi-view generators at a glance**

| tool | views | ortho | res | takes your mesh? | anime | VRAM | licence |
|---|---|---|---|---|---|---|---|
| MV-Adapter tg2mv/ig2mv | front, right, back, left, top, bottom | yes | 768 | **yes** (pos+normal) | yes (Animagine XL) | >12-16 GB SDXL; <6-10 GB SD2.1 | Apache-2.0 |
| CharacterGen 2D | 4 (front, back, 2 sides) A-pose | near | 512×768 | no (fixed skeleton images) | yes (VRoid) | CUDA, not stated | Apache-2.0 |
| StdGEN | A-pose + multi-view | – | – | no | yes | `--low_vram` option | code Apache-2.0, ckpt research-only |
| Zero123++ | 6 at ±elev | no | 320 per view | depth CN only | some | ~5-5.7 GB | non-commercial model |
| Era3D-ortho | 6 | yes | 512 | no | general | not stated | AGPL-3.0 |
| PSHuman | 6-7 | – | 768 | SMPL optional | humans | >40 GB | MIT |

Source: the READMEs above [fetched].

### Inferences
- **Match the MV-Adapter cameras to HeroBody's views.** HeroBody needs front, side, back and top. MV-Adapter gives those plus a left/right pair and a bottom view; ignore the bottom view, since HeroBody has none.
  - Having **both** left and right side views is useful. HeroBody's "side mirror match ≥ 0.95" compares one side with the mirrored other side.
  - Map the cameras as MV-Adapter az 0 → front, 90 → side, 180 → back, 270 → other side, elevation 89.99 → top (check the orientation convention per the README warning).
- **The resolution gap is quantified in the table below** [computed]:

| generator | frame px | body px (approx.) | latent cell (8 px) as % of body height | 1% of H in px | needed upscale to reach 1536 px body |
|---|---|---|---|---|---|
| MV-Adapter SDXL | 768 | ~700 (ortho ±0.55 of a rescaled mesh) | 1.1% | 7 | ×2.2 |
| CharacterGen | 512×768 | ~480 (pose images) | 1.7% | 4.8 | ×3.2 |
| SDXL single view, 832×1216 | 1216 tall | ~1100 | 0.73% | 11 | ×1.4 |
| target | – | ≥1536 | 0.52% | 15.4 | – |

  - A latent cell (8 px) is already about 1% of height at 768 px. The diffusion model therefore **cannot** be trusted to place landmarks within 1% at native resolution.
  - The fix is to put the guarantee in the *control* and in a deterministic *correction* step after upscaling (Sections 4 and 6), not in the generator.
- **Generating a top view.** Few 2D generators do overhead views well. MV-Adapter is the only fetched tool that renders an elevation-89.99 view from the mesh. Otherwise, render HeroBody's top view directly in Blender with toon or Line Art shading and let the human approve it. HeroBody's top-view threshold is lower (≥ 0.90), which suggests it is checked mainly as a silhouette.

### Gaps
- No quantitative multi-view consistency numbers (e.g. silhouette IoU across views) were fetched for MV-Adapter, CharacterGen or StdGEN. Their papers are on arXiv, which was blocked.
- Whether MV-Adapter's geometry control respects **thin anime details and extreme proportions** (3.6-heads chibi) is untested. Its training data is Objaverse (Ortho10View/Rand6View), not anime humans.
- CharacterGen's pose guider was trained on adult-proportion VRoid skeletons. Child skeletons may be out of distribution.

---

## 4. Measurement-first alternative (numeric age targets → 3D body → guide render → image model draws style only)

### Takeaway
- **This is the recommended route.** Its numeric half needs no generative model and runs on CPU.
  1. Measure the approved adult blueprint (`hb/measure2d.py`).
  2. Compute the character's per-ratio offset from the population median.
  3. Carry the offset to each target age on top of the age-specific population median.
  4. Solve the HeroBody 3D body at that age (knob solver, height last).
  5. Render orthographic guides in Blender.
  6. Only then let an image model "paint" the style details.
- **Precedents exist for each half, but not for the whole pipeline.**
  - Champ renders SMPL depth, normal and semantic maps as guidance.
  - MV-Adapter renders mesh position and normal maps for six orthographic views.
  - StableGen is a Blender add-on that drives SDXL, FLUX or Qwen-Image-Edit with Depth, Canny and Normal ControlNets from Blender cameras.
  - The sibling clothes notes already recommend "render your own base body in A-pose, then dress it with an image editor".
  - No fetched source does the age-shifted measurement step. That step is HeroBody-specific.
- **Carry lengths as log-ratio offsets and girths as z-scores.** Carrying everything multiplicatively makes a thin-waisted anime adult into an impossibly thin child (waist/H 0.34 at 10 y, against a real −2 SD of about 0.35 to 0.36).

### Cited Findings
- **Champ** renders SMPL depth, normal and semantic maps per frame and has guidance encoders for each (`guidance_encoder_depth/normal/semantic_map.pth`). It ships SMPL rendering scripts, including a Blender smoothing method. — [Champ README](https://github.com/fudan-generative-vision/champ) [fetched]
- **MV-Adapter** text-geometry and image-geometry pipelines take a `.glb` mesh and condition on rendered position and normal maps (code in Section 3). — [inference_tg2mv_sdxl.py](https://raw.githubusercontent.com/huanngzh/MV-Adapter/main/scripts/inference_tg2mv_sdxl.py) [fetched]
- **StableGen** is a GPL-3.0 Blender add-on. It uses "multiple ControlNet units (Depth, Canny, Normal) simultaneously to ensure generated textures respect your model's geometry". It "works with all architectures (SDXL, Flux, Qwen Image Edit)" and has per-camera aspect ratios and camera mirroring. It uses ComfyUI as the backend. — [StableGen README](https://github.com/sakalond/StableGen) [fetched]
- **CharacterGen** conditions each view on a rendered 3D skeleton image (`pose_image=pose_imgs_in`) together with camera matrices. — [2D_Stage/webui.py](https://raw.githubusercontent.com/zjp-shadow/CharacterGen/main/2D_Stage/webui.py) [fetched]
- **Population medians by age** (ratios to stature, male; female in the sibling note): HH/H, leg/H, biacromial/H and waist/H from Snyder 1977, NL4, CDC, WHO, ANSUR II and NHANES. Robust SDs are in Table 6c of the sibling note. — `proportions_by_age.md` Tables 6a-6c [sibling note]
- **MoGe-2 normals** (used by HeroBody on approved drawings): the `moge-2-vitl-normal`, `vitb-normal` and `vits-normal` checkpoints exist, and the CLI accepts `--device cpu`. Latency is 60 ms per image on an A100 or RTX3090 (FP16, ViT-L). MIT licence. — [MoGe README](https://github.com/microsoft/MoGe), [LICENSE](https://raw.githubusercontent.com/microsoft/MoGe/main/LICENSE) [fetched]

### The age-shift equations (code-ready) [inference, built on sibling-note data]
Notation: `j` is a measurement ratio (to stature H). `r̄_j(a,s)` is the population median at age `a` (years) and sex `s`. `σ_j(a)` is the population SD of the ratio. `m_j` is the value measured on the approved adult blueprint at reference age `a0` (e.g. 25 y). `M_j(a)` is a manga override measured at age `a`, if the user supplied one.

1. **Lengths and head size (skeletal: head height, sitting height, leg, arm, hand, foot, biacromial, bi-iliac).** Use a log-ratio offset, carried with a style gain `g_j(a)`:
   `δ_j = ln(m_j / r̄_j(a0,s))`
   `r_j(a) = r̄_j(a,s) · exp(g_j(a) · δ_j)`
   Set `g_j = 1` by default (identity and style kept at every age). Use `g_j < 1` to blend toward realism in a stage, for example if the user wants a more realistic old age.
2. **Girths (waist, chest, hip, thigh, calf, neck), which hold fat and muscle.** Use a z-score that rescales with the age-specific SD:
   `z_j = (m_j − r̄_j(a0,s)) / σ_j(a0)`
   `r_j(a) = r̄_j(a,s) + z_j · σ_j(a)`
   Optionally let the user set a body-condition change per stage, such as "chubby as a child" or "lean in old age", as Δz.
3. **Consistency constraints**, enforced after steps 1 and 2:
   - `SH/H = 1 − leg/H`. Carry leg/H and derive SH.
   - chest ≥ waist except at 0 to 2 y.
   - Approximate span = 2·arm + biacromial; re-normalise span if it is off by more than 1%.
4. **Manga override for a stage.** Combine as a Gaussian product with measurement SD `τ_j` (0.003 to 0.005 for lengths at 1536 px):
   `r_j* = (M_j/τ_j² + r_j(a)/σ_j²) / (1/τ_j² + 1/σ_j²)`
   Fade the override back to the formula over ±1 stage (`head_face_teeth_hair_by_age.md` uses the same fade).
5. **Stature.** Carry H as a log offset too:
   `H(a) = H̄(a,s) · exp(ln(H_blueprint / H̄(a0,s)))`
   That is, the character keeps its height percentile. Apply H **last** as a uniform scale, as already decided.
6. **Slider inside a stage.** Use `a = stage_start + u · (next_stage_start − stage_start)`, `u ∈ [0,1]`. Interpolate `r̄` and `σ` with PCHIP in age, as the sibling note recommends.

**Worked example** [computed from sibling Table 6a; script `scratchpad/scripts/agetargets.py`]. The character is a hypothetical adult male whose approved blueprint at a0 = 25 y has HH/H 0.143 (7.0 heads), leg/H 0.500, biac/H 0.250, waist/H 0.420 and H 174 cm. Its offsets are:
- δ_HH = +0.1187, δ_leg = +0.0492, δ_biac = +0.0576 (log)
- z_waist = (0.420 − 0.520)/0.063 = −1.59

| age y | real HH/H | target HH/H | heads | target leg/H | target SH/H | target biac/H | target waist/H (z-carry) | target waist/H (if log-carried, wrong) | H cm | head px @1536 body | ±3% of head (px) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | 0.240 | 0.270 | 3.70 | 0.324 | 0.676 | 0.244 | (SD n/a) | 0.510 | 49.1 | 415 | 12.5 |
| 1 | 0.210 | 0.236 | 4.23 | 0.370 | 0.630 | 0.244 | (SD n/a) | 0.468 | 74.5 | 363 | 10.9 |
| 2 | 0.189 | 0.213 | 4.70 | 0.416 | 0.584 | 0.243 | 0.479 | 0.426 | 85.2 | 327 | 9.8 |
| 3.5 | 0.181 | 0.204 | 4.91 | 0.460 | 0.540 | 0.244 | 0.447 | 0.401 | 97.2 | 313 | 9.4 |
| 6 | 0.162 | 0.182 | 5.48 | 0.482 | 0.518 | 0.237 | 0.398 | 0.359 | 113.6 | 280 | 8.4 |
| 8 | 0.150 | 0.169 | 5.92 | 0.493 | 0.507 | 0.231 | 0.373 | 0.343 | 125.9 | 259 | 7.8 |
| 10 | 0.143 | 0.161 | 6.21 | 0.504 | 0.496 | 0.231 | 0.362 | 0.338 | 136.5 | 247 | 7.4 |
| 12 | 0.136 | 0.153 | 6.53 | 0.512 | 0.488 | 0.229 | 0.359 | 0.335 | 146.8 | 235 | 7.1 |
| 14 | 0.132 | 0.149 | 6.73 | 0.517 | 0.483 | 0.230 | 0.355 | 0.328 | 161.3 | 228 | 6.8 |
| 16 | 0.126 | 0.142 | 7.05 | 0.513 | 0.487 | 0.234 | 0.352 | 0.326 | 170.8 | 218 | 6.5 |
| 18 | 0.125 | 0.141 | 7.10 | 0.509 | 0.491 | 0.240 | 0.362 | 0.334 | 173.5 | 216 | 6.5 |
| 25 | 0.127 | 0.143 | 6.99 | 0.500 | 0.500 | 0.250 | 0.420 | 0.420 | 174.0 | 220 | 6.6 |
| 45 | 0.128 | 0.144 | 6.94 | 0.502 | 0.498 | 0.251 | 0.478 | 0.467 | 173.5 | 221 | 6.6 |
| 65 | 0.130 | 0.146 | 6.83 | 0.506 | 0.494 | 0.251 | 0.510 | 0.493 | 170.8 | 225 | 6.7 |
| 80 | 0.132 | 0.149 | 6.73 | 0.513 | 0.487 | 0.252 | 0.510 | 0.493 | 167.7 | 228 | 6.8 |

How the table was built:
- The z-carry column uses Snyder robust SDs: 0.031 (2-4 y), 0.029 (4-7), 0.033 (7-10), 0.035 (10-13), 0.032 (13-16), 0.033 (16-19) and 0.063 (adult). 0 to 1 y has no SD in the sibling note.
- The "±3% of head" column shows how tight HeroBody's "every ratio within 3%" check is on a small ratio: **6.5 to 12.5 px** at a 1536 px body. A 768 px generator gives half of that, which is less than one latent cell.

### Inferences
- **Why measurement-first passes HeroBody's checks by construction.** All views are orthographic renders of **one** 3D body, so:
  - Landmark heights agree across views exactly. The error is only the measurement error of `measure2d.py`, well under the 1% tolerance.
  - The side views are mirror images of a symmetric body, giving IoU of about 1.0 against the ≥ 0.95 threshold.
  - The front/back silhouettes are identical up to mirroring, against the ≥ 0.92 threshold.
  - The 3D-from-outlines check is met because the outlines *are* the 3D body.
  - Body-to-blueprint ratios are within 3% because the blueprint was derived from the solved body.
- **What can still break the checks** is only what the image model adds: hair, clothes, accessories and line wobble.
  - Hair and clothes legitimately change the silhouette. HeroBody's measure step must therefore measure body landmarks from the **body guide**, not from the dressed drawing, or use the "nude/underwear" approved view as the measurement view. This matches the existing separation of body, garments and hair.
- **Order of precedence for the human-in-the-loop profile:**
  1. user-verified manga override at that age
  2. the age formula with the character offset
  3. population median
- **Anny age knob.** Drive Anny's age from real years with the WHO-calibrated anchors in the sibling note: years [0, 1, 4, 11, 16, 18, 64, 110] → anny_age [0, 0.05, 0.215, 0.415, 0.67, 0.77, 0.83, 1.0]. Clamp at ≥ 0 because of the Phase-1 baby-end break. Then let the knob solver hit the targets above.

### Gaps
- There is no published style-gain data `g_j(a)` for how anime artists exaggerate child or old proportions relative to real ones. For example, do One Piece child versions keep the adult's head-size exaggeration? Default g = 1 is an assumption to be checked against approved manga images per character.
- Girth SDs for 0 to 2 y and old age are missing (sibling-note gap). Use 1.1 × the 2-4 y SDs for 0 to 2 y [inference].

---

## 5. CPU-only feasibility (user's NVIDIA GPU has hardware faults)

### Takeaway
- **The whole numeric and 3D half runs CPU-only.** That covers measurement, the age-shift equations, the Anny knob solver, Blender orthographic rendering (Cycles CPU or Workbench with software GL), Line Art, and OpenPose drawing from rig joints.
- **CPU diffusion is feasible but slow and limited to older architectures with ControlNet.**
  - FastSD CPU (MIT) runs SD1.5/SDXL with LCM-LoRA, Turbo, Lightning and Hyper-SD in 1 to 4 steps, with ControlNet v1.1 in LCM-LoRA mode. On a Core i7-12700, a 768×768 SDXL image takes **6.3 to 19 s at 1 step** and **10 to 18 s at 2 steps (Lightning)**.
  - stable-diffusion.cpp (MIT) runs SD1.x, SDXL, FLUX, Qwen-Image(-Edit) and more on CPU (AVX/AVX2/AVX512), but **ControlNet only with SD 1.5**. It also has a Vulkan backend, which still uses the GPU and so is risky here.
- **MV-Adapter, CharacterGen, StdGEN, PuLID, InstantID, Champ, PSHuman and the GAN face agers are effectively GPU-only.** They hard-code CUDA, use nvdiffrast or CV-CUDA, or need 12 to 40 GB VRAM. Run them on a cloud GPU (flagged as GPU) or skip them.
- **Cloud image APIs** (Gemini image models, GPT-image, hosted Qwen-Image-Edit, hosted FLUX Kontext) need no local GPU. They give the weakest geometric guarantee, so they must sit behind the deterministic correction and check step (Section 6).

### Cited Findings
- **FastSD CPU** benchmarks on an Intel Core i7-12700:
  - SDXL Hyper-SD 1-step, 768×768: PyTorch 19 s, OpenVINO 13 s, OpenVINO + TAESDXL **6.3 s**.
  - SDXL-Lightning 2-step, 768×768: 18 / 12 / 10 s.
  - SDXL Turbo 1-step, 512×512: 10 / 5.6 / 2.5 s.
  - SD Turbo 512: 7.8 / 5 / 1.7 s.
  - FLUX.1-schnell int4 OpenVINO, 512×512, 3 steps: **4 min 30 s**, needing about 30 GB RAM.
  - Minimum RAM: LCM 2 GB, LCM-LoRA 4 GB, OpenVINO 11 GB (9 GB with TAESD).
  - ControlNet works in LCM-LoRA mode with the ControlNet-v1-1 fp16 models (about 723 MB each).
  - Added in 2026: FLUX.2-klein-4B (OpenVINO) with image editing.
  - Licence MIT.
  - — [FastSD CPU Readme](https://github.com/rupeshs/fastsdcpu), [LICENSE](https://raw.githubusercontent.com/rupeshs/fastsdcpu/main/LICENSE) [fetched]
- **stable-diffusion.cpp**:
  - Model support: SD1.x, SD2.x, SDXL (+Turbo), FLUX.1-dev/schnell, FLUX.2-dev/klein, Chroma, Qwen Image and 2.1, FLUX.1-Kontext-dev, the Qwen Image Edit series, Wan2.1/2.2 and others.
  - Extras: PhotoMaker, IP-Adapter (SD1.5 and SDXL incl. Plus), "Control Net support with SD 1.5", LoRA, LCM, TAESD, ESRGAN upscaling and VAE tiling.
  - Backends: CPU (AVX/AVX2/AVX512) and Vulkan, among others. MIT licence.
  - — [stable-diffusion.cpp README](https://github.com/leejet/stable-diffusion.cpp), [LICENSE](https://raw.githubusercontent.com/leejet/stable-diffusion.cpp/master/LICENSE) [fetched]
- **MV-Adapter** geometry inference uses `NVDiffRastContextWrapper` and defaults to `--device cuda`. Texture mode needs CV-CUDA. — [inference_tg2mv_sdxl.py](https://raw.githubusercontent.com/huanngzh/MV-Adapter/main/scripts/inference_tg2mv_sdxl.py), [README](https://github.com/huanngzh/MV-Adapter) [fetched]
- **CharacterGen** moves tensors `.to("cuda")` unconditionally in `webui.py`. — [2D_Stage/webui.py](https://raw.githubusercontent.com/zjp-shadow/CharacterGen/main/2D_Stage/webui.py) [fetched]
- **MoGe-2** has CLI `--device` "cpu" support. — [MoGe README](https://github.com/microsoft/MoGe) [fetched]
- **InstantID**'s face analysis lists `CPUExecutionProvider` as a fallback, and the pipeline supports `enable_model_cpu_offload()`. Offload still assumes a GPU [inference]. — [InstantID README](https://github.com/instantX-research/InstantID) [fetched]

**CPU feasibility per pipeline step**

| step | tool | CPU? | estimated time per view (i7-12700-class) | notes |
|---|---|---|---|---|
| measure blueprint, age targets, solver | hb/measure2d.py, numpy/scipy, Anny | yes | seconds | [inference] |
| ortho guide renders (normal, depth, mask, Line Art, OpenPose) | Blender 5.2 `-b`, Cycles CPU or Workbench | yes | seconds to a minute at 1536-2048 px | Workbench/Eevee need a GL context; under WSL use Mesa llvmpipe or Cycles [inference] |
| style paint, SD1.5 anime + ControlNet lineart_anime/depth/openpose | FastSD CPU (LCM-LoRA) or sd.cpp | yes | ~10-40 s at 768 px, 4 steps [inference from fetched 1-2 step numbers] | quality below SDXL anime models |
| style paint, SDXL anime + ControlNet | diffusers on CPU | slow | minutes per image [inference] | sd.cpp has no SDXL ControlNet |
| tile upscale ×2 to 1536 | sd.cpp ESRGAN / tile ControlNet | yes | ~1-3 min [inference] | |
| MV-Adapter, CharacterGen, StdGEN, PuLID-FLUX | – | **GPU (cloud)** | – | flag as GPU |
| MoGe-2 normals on approved views | MoGe CLI `--device cpu` | yes | seconds to tens of seconds [inference] | |
| cloud edit models | Gemini / GPT-image / hosted Qwen-Edit | no local GPU | API latency | weakest geometry; must be corrected and checked |

### Inferences
- **Vulkan in sd.cpp** uses the faulty NVIDIA card. Force the CPU backend.
- **A practical CPU stack:** Blender guides → FastSD CPU or sd.cpp (SD1.5 anime checkpoint + ControlNet lineart_anime at weight about 1.0 + depth or normal at about 0.6 + openpose at about 0.5) at 768 px per view → ESRGAN ×2 → deterministic correction. The control weights are my suggested starting points, not fetched values. Each view is an independent img2img/txt2img call, so cross-view consistency comes from the shared 3D guides plus a fixed seed and the same LoRA.
- **A practical cloud GPU stack (flagged GPU):** MV-Adapter ig2mv SDXL (Animagine) with the approved adult front art as image condition and the age-solved HeroBody mesh as geometry. This gives 6 consistent views at 768 px. Then upscale and correct as in Section 6.

### Gaps
- There are no fetched CPU timings for ControlNet in FastSD or sd.cpp, and none for MV-Adapter on CPU. It would have to be patched off nvdiffrast to try.
- Whether HeroBody's WSL Blender can use Eevee or Workbench headless without a GPU was not tested.

---

## 6. Recommended way to produce approved blueprints for each age stage

### Takeaway
**Measure → solve → render guides → paint style → correct deterministically → check → human approves.** The image model never decides proportions. It only adds line style, hair, clothes and face decals on top of a guide that already satisfies every HeroBody check. A deterministic post-warp absorbs the model's few-pixel drift.

### Procedure [inference; each step uses tools cited above]
1. **Inputs, per character.**
   - The approved adult blueprint (front, side, back, top).
   - The profile: sex, birth age, BMI category, body conditions or mobility aids, stage list.
   - Optional manga images tagged with age.
   - Measure all ratios with `hb/measure2d.py` and store `m_j` and `H_blueprint`.
2. **Age targets** (Section 4 equations). For each stage start and any slider value:
   - lengths by log-offset
   - girths by z-carry
   - manga override by Gaussian product
   - stature by percentile carry
   - Write `targets[stage][ratio]`.
3. **Solve the 3D body** at that age.
   - Anny age from years (sibling anchors, clamp ≥ 0), then the knob solver over the 44 Anny parameters toward the targets, with height applied last.
   - The anime head module takes the age-specific face spec (head-radius units) and the age-specific head size.
   - **Hard check:** every target ratio on the 3D body is within 1% (stricter than the 3% approval, which leaves margin for drawing).
4. **Render guides in Blender** (CPU).
   - Orthographic cameras: front (az 0), side L (90), back (180), side R (270), top (elevation 90).
   - Same ortho scale for all views. Canvas e.g. 1280×2048 with the body at 1740 px (≥ 1536) and the head ≥ 180 px.
   - The ortho scale is constant across stages of one character only if you want height comparisons. Otherwise normalise each stage to body height and store px/cm.
   - Passes per view:
     - (a) silhouette mask
     - (b) camera-space normal, (normal/2 + 0.5)
     - (c) depth
     - (d) Line Art (Grease Pencil Line Art modifier) for anime contour
     - (e) OpenPose COCO-18 skeleton drawn from rig joints with the ControlNet `limbSeq` and colours (Section 2)
     - (f) optional position map `(pos + 0.5)` for MV-Adapter
     - (g) a landmark JSON: vertex, chin, shoulder, nipple, navel, crotch, knee, ankle and sole heights in px, taken from the solved body.
5. **Paint the style.** Choose one:
   - **CPU local:** SD1.5 anime model + character LoRA (trained on the adult approved sheet plus manga images, captioned with age tokens) + ControlNet lineart_anime + depth/normal + openpose. Use a fixed seed for all views. 768 px, then tile or ESRGAN ×2.
   - **Cloud GPU (flagged):** MV-Adapter ig2mv SDXL with the anime base and the character LoRA, geometry = the solved age mesh (`.glb`), reference = the adult approved front. Take front/side/back/side/top from its 6 views, then upscale.
   - **Cloud API:** an edit model given the body guide plus the adult art as reference, prompted for "same character, age N, same outfit resized, keep exact silhouette of image 1". Expect drift.
6. **Correct deterministically** (CPU, numpy/OpenCV).
   - (a) Body-silhouette IoU between the painted image's body mask (segment out hair and clothes, or paint a "body-only" pass first) and the guide mask. Reject and re-roll if below 0.95.
   - (b) Measure landmark heights on the painted image. Fit a **monotone piecewise-linear vertical remap** `y' = f(y)` through the pairs (measured landmark, target landmark) and warp the image. Do the same per horizontal band for widths if needed. Max correction allowed: 2% of H. If more is needed, re-roll.
   - (c) Enforce mirror symmetry: side L against the flipped side R, and front against the flipped back silhouette. Fix with a small symmetric warp, or re-roll.
7. **Run the HeroBody approval checks** on the corrected images:
   - landmarks ≤ 1% of H across views
   - head size, tilt and position within ±5% / ±3° / ±1%
   - side mirror ≥ 0.95, front/back ≥ 0.92
   - 3D-from-outlines ≥ 0.93, top ≥ 0.90
   - every ratio within 3%
   - body ≥ 1536 px, head ≥ 180 px
   - Then run MoGe-2 normals on the approved views, as in the current pipeline.
8. **Human verifies** the profile and the sheet, not the mesh. They can add a manga override for the stage, which reruns from step 2.

**Pixel budget at a 1536 px body** [computed]:

| check | tolerance | px |
|---|---|---|
| landmark agreement across views | 1% H | 15.4 |
| head position | 1% H | 15.4 |
| head size ±5%, adult head 0.127 H (195 px) | 5% | 9.8 |
| head size ±5%, 6 y head 0.162 H (249 px) | 5% | 12.4 |
| a 3% ratio on head height, adult | 3% of 195 | 5.9 |
| a 3% ratio on leg length, adult 0.476 H (731 px) | 3% | 21.9 |
| SDXL latent cell after ×2 upscale from 768 | – | 16 |

So the generator alone cannot meet the small-ratio checks. The guide and the post-warp must.

### Gaps
- None of the correction steps (6a to 6c) has a published precedent that I could fetch. They are standard image-processing steps [inference].
- No data on how often a ControlNet-guided anime render needs re-rolls to pass IoU ≥ 0.95. Measure it on the 4 approved pilot characters (Luffy, Zoro, Koby and a fourth) at 3 ages each before scaling up.

---

## How this plugs into HeroBody

- **New module `hb/age_targets.py`** (Phase 2). It implements the Section 4 equations:
  - `delta_len = ln(m/r̄(a0))` and `z_girth = (m − r̄(a0))/σ(a0)`
  - `targets(age) = r̄(age)·exp(g·delta)` for lengths and `r̄(age) + z·σ(age)` for girths
  - the manga-override Gaussian product, stature percentile carry, and PCHIP over age.
  - Inputs: `hb/data/proportions_by_age.csv` (from the sibling note) and the character profile. Output: the knob-solver targets per stage. This feeds the open build item **"knob solver (height last)"** directly.
- **Character profile (data).** Add:
  - `blueprint_adult_ratios` (from `measure2d.py`) and `a0`
  - `style_gain` per ratio (default 1)
  - `girth_z` per ratio
  - `stage_overrides[]`: manga image path, age in years and measured ratios
  - `delta_z_condition` per stage (e.g. chubby child).
  - Keep sex fixed, never fitted.
- **Anny age.** Real years → anny_age with the sibling anchors, clamped at ≥ 0 (the Phase-1 baby-end break). The four open blendshapes are the ones the targets demand at young ages: head bigger than 2× (HH/H up to 0.27 for this example character at 0 y), chibi short legs (leg/H 0.32 to 0.42 at 0 to 2 y), round torso, and anime waist.
- **New Blender script `hb/render_guides.py`** (CPU). It writes orthographic front, side L, back, side R and top renders with mask, normal, depth, Line Art, OpenPose (COCO-18, ControlNet colours) and a position map, plus `landmarks.json` from the solved body. All views use one ortho scale, ≥ 1536 px body and ≥ 180 px head. The same renders double as MV-Adapter geometry input via a `.glb` export.
- **New script `hb/paint_style.py`.** Backend switch: `fastsd_cpu | sdcpp_cpu | mvadapter_cloud_gpu | api`. Fixed seed. Character LoRA. Age token in the prompt.
- **New script `hb/correct_and_check.py`.** Body-mask IoU against the guide (≥ 0.95), monotone landmark warp (≤ 2% H), mirror and front/back symmetry, then the existing approval checks (`measure2d.py` thresholds). Only passing sheets go to the human.
- **Face.** Age the anime head module parametrically (per `head_face_teeth_hair_by_age.md`). FADING (CPU-capable, no licence) may be used only on realistic renders, as a private age-cue reference. Do not ship its outputs, because its licence status is unclear.
- **Garments and hair.** Resize the separate garment and hair data on the solved age body before rendering the guides. Then the painted sheet shows the outfit already at the right size, and the body measurements still come from the body passes.

## Sources
- FADING: https://github.com/MunchkinChen/FADING ; https://raw.githubusercontent.com/MunchkinChen/FADING/master/age_editing.py ; https://raw.githubusercontent.com/MunchkinChen/FADING/master/FADING_util/util.py ; https://raw.githubusercontent.com/MunchkinChen/FADING/master/null_inversion.py
- SAM: https://github.com/yuval-alaluf/SAM ; https://raw.githubusercontent.com/yuval-alaluf/SAM/master/LICENSE
- HRFAE: https://github.com/InterDigitalInc/HRFAE ; https://raw.githubusercontent.com/InterDigitalInc/HRFAE/master/LICENSE.txt
- CUSP: https://github.com/guillermogotre/CUSP
- AgeBooth (not fetched, blocked): https://arxiv.org/abs/2510.05715
- MV-Adapter: https://github.com/huanngzh/MV-Adapter ; https://raw.githubusercontent.com/huanngzh/MV-Adapter/main/scripts/inference_tg2mv_sdxl.py ; https://raw.githubusercontent.com/huanngzh/MV-Adapter/main/LICENSE ; https://github.com/huanngzh/ComfyUI-MVAdapter
- CharacterGen: https://github.com/zjp-shadow/CharacterGen ; https://raw.githubusercontent.com/zjp-shadow/CharacterGen/main/2D_Stage/configs/infer.yaml ; https://raw.githubusercontent.com/zjp-shadow/CharacterGen/main/2D_Stage/webui.py ; https://github.com/zjp-shadow/CharacterGen/tree/main/2D_Stage/material
- StdGEN: https://github.com/hyz317/StdGEN
- Zero123++: https://github.com/SUDO-AI-3D/zero123plus
- Era3D: https://github.com/pengHTYX/Era3D
- PSHuman: https://github.com/pengHTYX/PSHuman
- Champ: https://github.com/fudan-generative-vision/champ
- ControlNet 1.1: https://github.com/lllyasviel/ControlNet-v1-1-nightly ; https://raw.githubusercontent.com/lllyasviel/ControlNet-v1-1-nightly/main/annotator/openpose/util.py
- comfyui_controlnet_aux: https://github.com/Fannovel16/comfyui_controlnet_aux
- DWPose: https://github.com/IDEA-Research/DWPose
- IP-Adapter: https://github.com/tencent-ailab/IP-Adapter
- InstantID: https://github.com/instantX-research/InstantID
- PuLID: https://github.com/ToTheBeginning/PuLID
- Qwen-Image: https://github.com/QwenLM/Qwen-Image
- MoGe: https://github.com/microsoft/MoGe
- stable-diffusion.cpp: https://github.com/leejet/stable-diffusion.cpp
- FastSD CPU: https://github.com/rupeshs/fastsdcpu
- StableGen: https://github.com/sakalond/StableGen
- Sibling notes (same folder): `proportions_by_age.md` (Tables 6a-6c, Anny anchors), `head_face_teeth_hair_by_age.md` (face spec by age, override fade); `../Clothes in manga to anime pipeline/generative_ai_costume_consistency.md`; `../Solo 3D anime clothes in Unreal/manga_to_3d_character.md`
