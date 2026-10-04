# Generative AI for Costume-Consistent Anime Frames and Video (Generation Stage of a Manga-to-Anime Pipeline), 2023–2026

Research notes compiled 2026-10-04. Access caveat: arxiv.org, huggingface.co, civitai.com, wikipedia, cartoonbrew, cbr.com and GitHub's API were blocked by the network proxy during this session. Most facts below come from search-result snippets of primary pages (arXiv abstracts, model cards, vendor pages). License facts were checked directly by downloading LICENSE/README files from raw.githubusercontent.com. Claims that come only from vendor marketing or SEO/aggregator blogs are flagged as such.

---

## 1. Image-level identity and outfit consistency: which methods hold clothing details best?

### Takeaway
As of 2026 there are two working families. (a) **Trained adapters**: a character/outfit LoRA on an anime SDXL base, plus ControlNet. This is still the most controllable route for exact costume reproduction, but needs a curated dataset and careful tag pruning. (b) **Zero-shot in-context editors**: FLUX.1 Kontext, FLUX.2, Qwen-Image-Edit-2509/2511, Nano Banana / Nano Banana Pro, GPT-image. These now keep identity and outfits across edits far better than IP-Adapter-era methods. Small costume details (emblems, patterns, accessory counts) remain the weak point, and community tests found that clothing-specific edits are where some editors (e.g., Gemini) fail most. Older zero-shot adapters (IP-Adapter-Plus, PhotoMaker, InstantID) are face/identity-oriented and keep clothing only loosely. A 2026 anime-specific adapter (AnimeAdapter) explicitly targets this gap.

### Cited Findings
**LoRA / DreamBooth character and outfit LoRAs**
- Community LoRA practice: describing scene, outfit, pose and lighting in captions keeps those elements promptable independently of identity. For custom outfits, generic tags without colour should be pruned to prevent "colour bleeding from custom outfits onto the main outfit". Signs of overtraining include "outfits from the dataset bleeding in". — [lora-dataset-studio DATASET_GUIDE](https://github.com/perfectgf/lora-dataset-studio/blob/v1/docs/DATASET_GUIDE.md); [Civitai "OC LoRA Part 7"](https://civitai.com/articles/5664/oc-lora-part-7-pupil-and-lower-body-training-and-impacts-of-inpainting-and-captioning)
- Multi-outfit character LoRAs: practitioners report trying separate folders per outfit, each with a unique trigger tag, "though results can still show bleedthrough". The recommended mix is images where some carry specific outfit tags and others don't, with a few examples of each variation. — [hollowstrawberry/stable-diffusion-guide discussion #7](https://huggingface.co/hollowstrawberry/stable-diffusion-guide/discussions/7); [Civitai LoRA guide "Characters, clothing, poses"](https://civitai.com/articles/680/characters-clothing-poses-among-other-things-a-guide)
- Kohya-SS tutorials cover dedicated clothing LoRAs and multi-subject training as separate techniques. — [LoRA Clothes and Multiple Subjects Training (Kohya SS)](https://lilys.ai/en/notes/training-lora-20260208/lora-clothes-multiple-subjects-training-sd-kohya)

**IP-Adapter / IP-Adapter-Plus / anime variants**
- IP-Adapter Plus encodes the reference with a Perceiver-Resampler-style patch embedding and follows the reference more closely than base IP-Adapter, "however, the fine-grained details, like the face, are usually not copied correctly". Anime fine-tunes of IP-Adapter and IP-Adapter-Plus exist (used with the CLIP-H image encoder). — [Stable Diffusion Art: IP-Adapters](https://stable-diffusion-art.com/ip-adapter/); [SD Art ControlNet guide](https://stable-diffusion-art.com/controlnet/)
- ComfyUI practice for multiple garments uses attention masks in "IPAdapter Advanced" nodes, one masked reference per clothing article. — [Endangered AI: IPadapter & ControlNet change clothes](https://endangeredai.com/ipadapter-controlnet-how-to-change-clothes-pose-with-ai/); [RunComfy consistent characters with ControlNet & IPAdapter](https://learn.runcomfy.com/create-consistent-characters-with-controlnet-ipadapter)
- IP-Adapter code is Apache-2.0 (LICENSE verified). PhotoMaker's repo LICENSE states it is licensed under Apache-2.0 (Tencent notice). StoryDiffusion repo is Apache-2.0. — raw LICENSE files: [tencent-ailab/IP-Adapter](https://github.com/tencent-ailab/IP-Adapter), [TencentARC/PhotoMaker](https://github.com/TencentARC/PhotoMaker), [HVision-NKU/StoryDiffusion](https://github.com/HVision-NKU/StoryDiffusion)

**AnimeAdapter (May 2026) – anime-specific zero-shot appearance adapter**
- AnimeAdapter is a lightweight appearance adapter for Stable Diffusion. It injects fine-grained CLIP patch tokens into U-Net decoupled cross-attention, using token-level foreground masking and pose-guided training to separate appearance from layout. It is pretrained on curated Danbooru-style data, needs no per-subject fine-tuning, and stays compatible with ControlNet, T2I-Adapter and LoRA. On its own anime character editing benchmark and on DreamBench++ it reports better appearance preservation than IP-Adapter variants. — [arXiv 2605.20237](https://arxiv.org/abs/2605.20237); [v2 HTML](https://arxiv.org/html/2605.20237v2)

**FLUX.1 Kontext (BFL, June 2025)**
- KontextBench has 1,026 image-prompt pairs across local editing, global editing, character reference, style reference and text editing. The paper reports markedly less cumulative drift over iterative edit turns than baselines, and matches or beats Runway Gen-4 and GPT-4o-High on character reference. Average AuraFace similarity is about 0.908 across editing steps. Note that this is a face metric, not a clothing metric. — [FLUX.1 Kontext paper, arXiv 2506.15742](https://arxiv.org/html/2506.15742v2); [EmergentMind summary](https://www.emergentmind.com/topics/flux-1-kontext)
- Blog reviews claim Kontext "returns consistent clothing edges and hair volume without extra prompts" and "reduces identity drift by up to 40%". These are third-party/SEO blogs; the 40% figure is not traceable to the BFL paper and should be treated as unverified. — [Flixly review](https://www.flixly.ai/blog/flux-kontext-review-character-consistency-2026); [eastondev ComfyUI Kontext guide](https://eastondev.com/blog/en/posts/ai/20260821-comfyui-flux-kontext-character-consistency/)

**FLUX.2 (BFL, late 2025)**
- FLUX.2 accepts up to 10 reference images at once for character/product/style consistency, with no training. Variants: [Pro] and [Flex] are proprietary API; [Dev] is open-weight under a non-commercial license, with a separately purchasable commercial self-hosting license. "Klein" variants also exist. — [BFL FLUX.2 blog](https://bfl.ai/blog/flux-2); [The Decoder](https://the-decoder.com/black-forest-labs-launches-flux-2-with-a-new-multi-reference-feature/); [aifilms FLUX.2 Klein](https://studio.aifilms.ai/blog/flux-2-production-image-generation)

**Qwen-Image-Edit (Alibaba Qwen, 2025)**
- Qwen-Image-Edit-2509 was trained via image concatenation for multi-image editing ("person + person", "person + product", "person + scene"). It works best with 1–3 input images, improves person-editing consistency and facial identity preservation, and natively supports ControlNet inputs (depth, edge, keypoint maps). — [HF model card Qwen-Image-Edit-2509](https://huggingface.co/Qwen/Qwen-Image-Edit-2509)
- Community ComfyUI workflows use Qwen-Image-Edit-2511 multi-image input for "consistent outfit changes" (character image + garment image), and combine 2509 with ControlNet-Union for pose changes that keep the outfit. — [NextDiffusion: Consistent Outfit Changes with Qwen Image Edit 2511](https://www.nextdiffusion.ai/tutorials/consistent-outfit-changes-with-multi-qwen-image-edit-2511-in-comfyui); [NextDiffusion pose + ControlNet](https://www.nextdiffusion.ai/tutorials/consistent-poses-qwen-image-edit-2509-controlnet-union-comfyui)
- QwenLM/Qwen-Image repo LICENSE is Apache-2.0 (verified). — [QwenLM/Qwen-Image](https://github.com/QwenLM/Qwen-Image)

**Nano Banana (Gemini 2.5 Flash Image) / Nano Banana Pro (Gemini 3 Pro Image)**
- Google markets Gemini 2.5 Flash Image for keeping a character consistent across prompts and edits. — [Google Developers Blog](https://developers.googleblog.com/en/introducing-gemini-2-5-flash-image/)
- A community 27-case comparison against Qwen-Image-Edit scored Gemini as failing "Clothing Extraction" (test case 6) but winning photo-to-anime resemblance (case 8). Reported limits: max 2048×2048 output, at most 5 images for composition, character consistency "not 100% reliable". A vendor review (which promotes an alternative product) reports that clothing-specific prompts degraded consistency. Both are informal tests. — [HF community blog (MonsterMMORPG)](https://huggingface.co/blog/MonsterMMORPG/nano-banana-gemini-25-flash-image-full-tutorial); [Snapmeld review](https://snapmeld.com/blog/gemini-2-5-flash-image-character-consistency-test)
- Nano Banana Pro takes up to 14 reference images (up to 6 high-fidelity object images and up to 5 human images for identity), with outputs up to 4K. Practitioners advise starting with 2–4 references, each with a distinct role. — [Scenario help center](https://help.scenario.com/articles/7568607761-gemini-image-models-nano-banana-family); [aifreeapi guide](https://www.aifreeapi.com/en/posts/nano-banana-pro-reference-images)

**StoryDiffusion, OmniGen, UNO, In-Context LoRA (multi-image/story consistency)**
- OmniGen2, bytedance/UNO and Phantom repos are Apache-2.0 (verified LICENSE files). In-Context-LoRA's README states it uses FLUX as the base, so "users must comply with FLUX's license", and warns its training data may contain copyrighted material. — raw LICENSE/README: [VectorSpaceLab/OmniGen2](https://github.com/VectorSpaceLab/OmniGen2), [bytedance/UNO](https://github.com/bytedance/UNO), [ali-vilab/In-Context-LoRA](https://github.com/ali-vilab/In-Context-LoRA)

### Inferences
- **What best preserves clothing**, ranked from evidence plus practice (my synthesis, not a measured benchmark):
  1. A character LoRA trained on the manga's own panels (colour model sheet + panels), with outfit tags kept separate, plus lineart ControlNet.
  2. Multi-reference in-context editors (FLUX.2, Qwen-Image-Edit-2511, Nano Banana Pro, Kontext), which handle "same outfit, new pose/scene" with no training but can simplify patterns.
  3. Anime-specific appearance adapters (AnimeAdapter).
  4. Generic IP-Adapter-Plus, which carries palette and silhouette but drops fine detail.
  5. Face-ID methods (InstantID, PhotoMaker), which target real-human faces and do not lock costumes.
- For manga sources the colours are not in the source. The colour authority must be a human-made colour design sheet (iro-shitei). That sheet is the reference image fed to every reference-based method.
- **Separate outfit LoRAs vs. character LoRAs:** separate LoRAs are composable (character LoRA plus outfit-A LoRA at reduced weights) but can conflict. A single character LoRA with distinct outfit trigger tags is simpler but prone to outfit bleed. Neither is clearly better in the sources; it is a trade-off.
- **FLUX.1 Kontext [dev] license (prior knowledge, verify):** it is believed to ship under the FLUX.1 [dev] Non-Commercial License, which restricts commercial use of the weights. Commercial users go through BFL's API or a paid license.

### Gaps
- No peer-reviewed head-to-head benchmark of Kontext vs Qwen-Image-Edit vs Nano Banana vs GPT-image specifically on anime **clothing** fidelity (pattern, emblem, accessory count). Published metrics (AuraFace) are face-centric.
- Could not access GPT-image-1 / GPT-image-1.5 documentation or any rigorous clothing-consistency test for them.
- InstantID's applicability to anime was not verified (repo not reachable). Its known design targets real human faces.
- No source gave minimum dataset sizes for outfit LoRAs. Commonly cited community figures, roughly 15–50 images, could not be verified this session.

---

## 2. Anime-specific base models and clothing-tag prompting

### Takeaway
The open-weight anime ecosystem as of 2026 centres on SDXL descendants trained on Danbooru tags: Illustrious XL, NoobAI-XL, Animagine XL 4.0, Pony V6. Newer architectures have arrived: Pony V7 on AuraFlow, and Neta Lumina on Lumina-Image-2.0. Clothing is controlled through Danbooru garment tags in a fixed tag order. Licenses differ sharply: the NoobAI and Illustrious lineage carry share-alike restrictions, and NoobAI forbids commercialisation; Animagine uses OpenRAIL++-M; Neta Lumina is Apache-2.0; Pony V7 has revenue-gated commercial terms.

### Cited Findings
- **Animagine XL 4.0** (Cagliostro Research Lab): retrained from SDXL 1.0 on 8.4M anime images, knowledge cutoff 7 Jan 2025, license CreativeML Open RAIL++-M. Recommended tag order is "1girl/1boy/1other, character name, from which series, rating, everything else in any order and end with quality enhancement". Quality tags: "masterpiece, high score, great score, absurdres". — [HF cagliostrolab/animagine-xl-4.0](https://huggingface.co/cagliostrolab/animagine-xl-4.0)
- **NoobAI-XL** (Laxhar Lab): fine-tuned from Illustrious-XL on the full Danbooru and e621 datasets with native tag captions. License is fair-ai-public-license-1.0-sd, inherited from Illustrious-xl-early-release-v0, with a strict share-alike rule. The model card states it "prohibits any form of commercialization, including … monetization or commercial use of the model, derivative models, or model-generated products". — [HF Laxhar/noobai-XL-1.1 README](https://huggingface.co/Laxhar/noobai-XL-1.1/raw/main/README.md); [noobai-XL-1.0](https://huggingface.co/Laxhar/noobai-XL-1.0/blob/main/README.md)
- **Illustrious-XL** early release carries the fair-ai-public-license-1.0-sd (as inherited by NoobAI). — [NoobAI model card](https://huggingface.co/Laxhar/noobai-XL-1.1/raw/main/README.md); Civitai license explainer: [What The License?!](https://civitai.com/articles/18619/what-the-license)
- **Pony V7** is rebuilt on AuraFlow (~7B parameters). Its Pony License allows free use for individuals and small businesses. Entities above US$1M in revenue or funding, or anyone selling paid inference, must contact the authors for commercial terms. AuraFlow itself is Apache-2.0. — [HF purplesmartai/pony-v7-base](https://huggingface.co/purplesmartai/pony-v7-base); [Civitai: Towards Pony V7](https://civitai.com/articles/6309/towards-pony-diffusion-v7-going-with-the-flow)
- **Neta Lumina** (Neta.art Lab): built on Lumina-Image-2.0 (Shanghai AI Lab), Apache-2.0, multilingual prompts (EN/ZH/JA). — [HF neta-art/Neta-Lumina README](https://huggingface.co/neta-art/Neta-Lumina/blob/main/README.md); [Neta blog](https://neta.art/blog/neta_lumina)
- A 2025/26 comparison guide of Illustrious vs NoobAI vs Animagine exists for practical model choice. — [localaimaster guide](https://localaimaster.com/blog/best-local-anime-image-model)

### Inferences
- **Clothing tags as the prompt-level "costume spec":** each costume can be written down as a fixed Danbooru tag block (e.g., `serafuku, red neckerchief, pleated skirt, black thighhighs, hair ribbon`). Reusing the identical block across all shots is the cheapest consistency lever. The tag-order conventions above put such tags in the "everything else" slot.
- **Commercial anime pipelines:** NoobAI and Illustrious derivatives are legally risky for commercial work, given the share-alike plus non-commercial clauses. Animagine XL 4.0 (OpenRAIL++-M, use-based restrictions only) and Neta Lumina (Apache-2.0) are the cleaner open options.
- **Niji Journey** (Midjourney's anime mode) has no LoRA or ControlNet-level control. Its role is limited to concept and style exploration (prior knowledge; not verified this session).

### Gaps
- Could not verify current licenses of Illustrious XL v1.0/v2.0+ (OnomaAI later releases), or whether later versions changed terms.
- Found no authoritative information on anime FLUX finetunes (e.g., Chroma or anime FLUX LoRAs) or their licenses this session.
- Niji Journey 6/7 capabilities (e.g., character reference "--cref") were not checked.

---

## 3. Structural control: using manga line art so clothing lines are preserved

### Takeaway
The most faithful way to keep manga costume design is to **not regenerate the garment geometry at all**: use the manga or cleaned key-animation line art as a hard structural condition (lineart / anime-lineart ControlNet, T2I-Adapter), or use reference-based line-art colorization models that only add colour. Colorization research matured quickly in 2024–2026 (MangaNinja, Cobra, MangaDiT, DACoN, paint-bucket colorization with color design sheets, OmniColor). These models carry a colour reference onto line art via explicit correspondence, which suits costume colour fidelity.

### Cited Findings
- ControlNet's "Line art anime" preprocessor produces anime-style lines, and "Line art anime denoise" produces fewer details. Both are used with the lineart control model. — [Stable Diffusion Art ControlNet guide](https://stable-diffusion-art.com/controlnet/)
- Qwen-Image-Edit-2509 natively accepts edge/depth/keypoint maps as control. — [HF Qwen-Image-Edit-2509](https://huggingface.co/Qwen/Qwen-Image-Edit-2509)
- **MangaNinja** (CVPR 2025, ByteDance/HKU et al.): reference-guided line-art colorization with a patch-shuffling module for local correspondence learning. A PointNet-driven point-control scheme lets users pin matching points between reference and line art. It handles extreme poses, shadows, cross-character colorization and multi-reference harmonization. Repo `bytedance/MangaNinjia`; no LICENSE file found at repo root. — [arXiv 2501.08332](https://arxiv.org/html/2501.08332v1); [project page](https://johanan528.github.io/MangaNinjia/); [CVPR paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Liu_MangaNinja_Line_Art_Colorization_with_Precise_Reference_Following_CVPR_2025_paper.pdf)
- **Cobra** (Apr 2025): comic line-art colorization using a large set of retrieved reference images, managed by "localized reusable positional encoding". Weights released 17 Apr 2025; Apache-2.0 LICENSE (verified). — [arXiv 2504.12240](https://arxiv.org/html/2504.12240); [zhuang2002/Cobra](https://github.com/zhuang2002/Cobra)
- **MangaDiT** (Aug 2025): reference-guided line-art colorization with hierarchical attention in Diffusion Transformers. — [arXiv 2508.09709](https://arxiv.org/pdf/2508.09709)
- **Paint Bucket Colorization Using Anime Character Color Design Sheets** (arXiv 2410.19424): explicitly models the studio workflow, where artists fill segments with RGB values from the character colour design sheet. It splits the task into keyframe colorization (from the sheet) and consecutive-frame colorization. It notes reference-based methods "often fail to accurately assign specific colors to each region", while matching-based methods struggle with deformation and occlusion. It proposes inclusion matching with segment parsing and colour warping. — [arXiv 2410.19424](https://arxiv.org/html/2410.19424v1)
- **DACoN** (Sep 2025): DINO features for anime paint-bucket colorization with any number of reference images. — [arXiv 2509.14685](https://arxiv.org/pdf/2509.14685)
- Further 2025–2026 colorization works listed in the curated Awesome-Animation-Research index: OmniColor (Mar 2026, multi-modal lineart colorization), Uni-Animator (WACV 2026), Line Art Colorization with Offset Prior-based Diffusion (WACV 2026), SketchColour (Jul 2025, DiT channel-concat), Animation Anycolor (ICASSP 2025, keypoint matching). — [zhenglinpan/Awesome-Animation-Research](https://github.com/zhenglinpan/Awesome-Animation-Research)

### Inferences
- **Recommended structure for a manga-to-anime pipeline:**
  1. Human or AI-assisted cleanup of manga panels into animation line art (genga/douga).
  2. Colour design sheet per costume.
  3. Correspondence-based colorization (MangaNinja point control, paint-bucket/inclusion matching).
  4. Video-level colorization (Section 5).

  Because the garment lines come from the artist, designs cannot drift; only colour assignment can be wrong, and that is checkable per segment.
- Paint-bucket/segment methods are the most "auditable" for costumes: each closed region gets an exact RGB from the sheet. Diffusion colorizers give richer shading but can repaint small accessories.

### Gaps
- No quantitative study found on how often lineart-ControlNet generations alter small garment details (buttons, emblems) vs. the conditioning lines.
- T2I-Adapter and ControlNet v1.1 repos showed no LICENSE file at root this session. Their license status needs checking on the model cards.

---

## 4. Outfit transfer / virtual try-on applied to anime

### Takeaway
Mainstream virtual try-on (VTON) models are trained on photographic fashion datasets (VITON-HD, DressCode). IDM-VTON, OOTDiffusion, StableVITON and CatVTON are **non-commercial (CC BY-NC-SA 4.0)**. None is anime-specific; OutfitAnyone (Alibaba HumanAIGC) is the only one that explicitly claims anime generalisation, and it is demo-only. Mask-free, generalist try-on (Any2AnyTryon, OmniTry) and multi-image editors (Qwen-Image-Edit-2511, FLUX.2, Nano Banana Pro) have effectively become the practical "outfit swap" tools for anime characters as of 2025–2026.

### Cited Findings
- **OutfitAnyone** (arXiv 2407.16224, Alibaba): two-stream conditional diffusion model. The paper states broad applicability "extending from anime to in-the-wild images" and the ability to modulate pose and body shape. — [arXiv 2407.16224](https://arxiv.org/html/2407.16224v1); [project page](https://humanaigc.github.io/outfit-anyone/)
- **IDM-VTON**: LICENSE is CC BY-NC-SA 4.0 (verified). **OOTDiffusion**: CC BY-NC-SA 4.0 (verified). **CatVTON**: CC BY-NC-SA 4.0 (verified). **StableVITON**: README states CC BY-NC-SA 4.0; it is built on Paint-by-Example with a VITON-HD fine-tuned VAE. — raw LICENSE/README: [yisol/IDM-VTON](https://github.com/yisol/IDM-VTON), [levihsu/OOTDiffusion](https://github.com/levihsu/OOTDiffusion), [Zheng-Chong/CatVTON](https://github.com/Zheng-Chong/CatVTON), [rlawjdghek/StableVITON](https://github.com/rlawjdghek/StableVITON)
- **Any2AnyTryon** (ICCV 2025): mask-free try-on using Adaptive Position Embeddings. Tasks are try-on, model-free try-on, garment reconstruction and layered try-on. Checkpoints released; no LICENSE file found at repo root. — [arXiv 2501.15891](https://arxiv.org/html/2501.15891v1); [logn-2024/Any2anyTryon](https://github.com/logn-2024/Any2anyTryon)
- **OmniTry** (Aug 2025): extends try-on to any wearable (jewellery, accessories) without masks. Reports better object localisation and ID preservation than prior methods. — [arXiv 2508.13632](https://arxiv.org/html/2508.13632v1)
- **Garments2Look** (Mar 2026): multi-reference dataset for outfit-level try-on including clothing and accessories. — [arXiv 2603.14153](https://arxiv.org/html/2603.14153v1)
- ComfyUI community: the CozyMantis "clothes-swap" workflow (SAL-VTON based) dresses a virtual character in real garments. Consumer "anime outfit changer" web tools also exist. — [cozymantis/clothes-swap-salvton-comfyui-workflow](https://github.com/cozymantis/clothes-swap-salvton-comfyui-workflow); [Kusart anime outfit changer](https://kusart.com/play/ai-outfit-changer)
- Multi-image editors as outfit swappers: Qwen-Image-Edit-2509's "person + product" mode, used for outfit changes, and the 2511 consistent-outfit workflow. — [HF Qwen-Image-Edit-2509](https://huggingface.co/Qwen/Qwen-Image-Edit-2509); [NextDiffusion 2511 workflow](https://www.nextdiffusion.ai/tutorials/consistent-outfit-changes-with-multi-qwen-image-edit-2511-in-comfyui)
- **Anime-Ready** (ICLR 2026): controllable 3D anime character generation with body-aligned, component-wise garment modelling. It is a 3D route to outfit-swappable anime characters. — [Awesome-Animation-Research index](https://github.com/zhenglinpan/Awesome-Animation-Research)

### Inferences
- **Domain gap:** photographic VTON models learn cloth warping and shading from photos. On flat-shaded anime art they tend to add photoreal folds and textures. For anime, using an editor with an anime-tuned base or LoRA is more reliable.
- **"Try-off" garment reconstruction** (Any2AnyTryon) is useful for extracting a clean garment reference from a manga panel or colour sheet. That reference can then be reused as a costume asset.

### Gaps
- No published anime-specific VTON benchmark or anime-trained try-on model was found.
- OutfitAnyone weights/code availability and license were not verified; it appears to be a demo service.

---

## 5. Video generation: how well do models keep clothes stable (flicker, pattern drift, accessory loss)?

### Takeaway
Two distinct regimes exist.

- **Line-art-conditioned video colorization/in-betweening.** Examples: LVCD, AniDoc, ToonCrafter, ToonComposer, AnimeColor, LongAnimation, InstanceAnimator. Garment shape is fixed by the sketches, and the challenge is colour consistency over time and occlusion. LongAnimation (ICCV 2025) explicitly tackles long-range (≈500-frame) colour drift.
- **Free generation from text/image/reference.** Examples: AnimateDiff, Wan 2.1/2.2, HunyuanVideo, FramePack, Index-AniSora, Kling, Vidu, Veo, Sora 2. Garments can drift, flicker or lose accessories. Reference-to-video features (Kling Elements/O1/3.0, Vidu up to 7 references, Wan2.2-Animate) are the vendors' answer, but evidence is mostly marketing.

AnimateDiff is documented to fail at keeping clothes colour stable.

### Cited Findings
**Line-art/sketch-conditioned (most relevant to manga fidelity)**
- **LVCD** (SIGGRAPH Asia/TOG 2024; Huang, Zhang, Liao): billed as the first video-diffusion framework for reference-based line-art video colorization. Components: SVD plus a Sketch-guided ControlNet, Reference Attention (colour transfer across large motion), and sequential sampling with Overlapped Blending and Prev-Reference Attention for long videos. Uses SVD weights; no LICENSE file found in repo. — [arXiv 2409.12960](https://arxiv.org/abs/2409.12960); [project](https://luckyhzt.github.io/lvcd); [luckyhzt/LVCD README](https://github.com/luckyhzt/LVCD)
- **AniDoc** (CVPR 2025; HKUST/Ant): colorizes sketch sequences from a reference character design. Uses explicit correspondence matching for robustness to pose mismatch between reference and line art, and can also in-between from start and end sketches. Built on SVD-img2vid-xt plus its own UNet/ControlNet plus CoTracker. Default 14 frames (works on ~72 with code changes). Inference ~14 GB VRAM; training on 8×A100. README shows an Apache badge but no LICENSE file at repo root. — [arXiv 2412.14173](https://arxiv.org/abs/2412.14173); [CVPR paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Meng_AniDoc_Animation_Creation_Made_Easier_CVPR_2025_paper.pdf); [yihao-meng/AniDoc README](https://github.com/yihao-meng/AniDoc)
- **ToonCrafter** (2024, Tencent AI Lab/CUHK): generative cartoon interpolation, up to 16 frames at 512×320. About 24 GB GPU (community down to ~10–12 GB). Apache-2.0 code. The README warns it is "an open-source research exploration, instead of commercial products". — [ToonCrafter README](https://github.com/ToonCrafter/ToonCrafter)
- **ToonComposer** (Aug 2025, Tencent ARC): unifies in-betweening and colorization into one "post-keyframing" stage. Uses sparse sketch injection and a spatial low-rank adapter on a video foundation model, and needs as little as one sketch plus one coloured reference frame. Introduces PKBench with human-drawn sketches. — [arXiv 2508.10881](https://arxiv.org/abs/2508.10881)
- **AnimeColor** (ACM MM 2025): DiT-based reference colorization with a High-level Color Extractor and a Low-level Color Guider. Uses four-stage training and claims better colour accuracy and temporal stability than prior work. Apache-2.0 (verified); weights at HF `rainbowow/AnimeColor`. — [arXiv 2507.20158](https://arxiv.org/abs/2507.20158); [IamCreateAI/AnimeColor](https://github.com/IamCreateAI/AnimeColor)
- **LongAnimation** (ICCV 2025): SketchDiT plus a Dynamic Global-Local Memory plus a Color Consistency Reward. The paper states prior methods focus on short-term (<100 frames) colorization and that local overlap-fusion "fails to maintain color consistency over long sequences due to neglected global information and accumulation of noise". Evaluated on 14-frame and ~500-frame clips (LPIPS, SSIM, PSNR, FVD, FID), beating ToonCrafter, LVCD and AniDoc. Base: CogVideoX-1.5-5B-I2V plus Video-XL. Training at 576×1024 on 6×A100. Apache-2.0 code. — [arXiv 2507.01945](https://arxiv.org/abs/2507.01945); [ICCV paper](https://openaccess.thecvf.com/content/ICCV2025/papers/Chen_LongAnimation_Long_Animation_Generation_with_Dynamic_Global-Local_Memory_ICCV_2025_paper.pdf); [CN-makers/LongAnimation](https://github.com/CN-makers/LongAnimation)
- **InstanceAnimator** (2026; HKUST(GZ) et al.): multi-instance sketch video colorization. A "Canvas Guidance Condition" places reference elements (characters, background) on a blank canvas instead of relying on one first-frame reference. Also has an Instance Matching Mechanism and an Adaptive Decoupled Control Module. — [arXiv 2603.25357](https://arxiv.org/html/2603.25357); [GitHub](https://github.com/YinHan-Zhang/InstanceAnimator)
- **TimeColor** (Jan 2026): flexible reference colorization via temporal concatenation. — [arXiv 2601.00296](https://arxiv.org/pdf/2601.00296)

**General / anime video generators**
- **AnimateDiff:** an open GitHub issue is titled "SD-XL cannot keep clothes color and hairstyle same", and a diffusers issue reports flicker. A 2026 guide calls flicker on faces and fine detail "a widely reported limitation", reduced but not eliminated by motion modules v2/v3. Apache-2.0 code. — [guoyww/AnimateDiff issue #226](https://github.com/guoyww/AnimateDiff/issues/226); [diffusers issue #7548](https://github.com/huggingface/diffusers/issues/7548); [promptquorum AnimateDiff 2026](https://www.promptquorum.com/power-local-llm/animatediff-video-generation-guide)
- **Wan2.2-Animate** (Alibaba, Sep 2025; arXiv 2509.14055) has two modes. Animation mode drives a reference character with a motion video. Replacement mode inserts the character into a video, with a relighting LoRA. The reference image defines face, body, clothing and appearance. Practitioner advice is to reuse the exact same reference image per clip because reference inconsistency is "the primary cause of character drift across clips". Wan2.2 repo is Apache-2.0 (verified). — [Wan blog](https://wan.video/blog/wan2.2-animate); [arXiv 2509.14055](https://arxiv.org/pdf/2509.14055v1); [wan-animate.com guide](https://wan-animate.com/posts/wan-2-2-animate-motion-types-style-consistency-guide)
- **HunyuanVideo / HunyuanVideo-I2V / HunyuanCustom:** Tencent Hunyuan Community License, which "does not apply in the European Union, United Kingdom and South Korea" (verified LICENSE text). — [Tencent-Hunyuan/HunyuanVideo](https://github.com/Tencent-Hunyuan/HunyuanVideo); [HunyuanCustom](https://github.com/Tencent-Hunyuan/HunyuanCustom)
- **FramePack** (lllyasviel): next-frame-section prediction. Can produce 60 s at 30 fps (1,800 frames) with a 13B model on 6 GB VRAM. Apache-2.0 code. — [FramePack README](https://github.com/lllyasviel/FramePack)
- **VACE** (Alibaba, all-in-one video creation/editing) and **Phantom** (subject-consistent video): Apache-2.0 repos (verified). — [ali-vilab/VACE](https://github.com/ali-vilab/VACE); [phantom-video/Phantom](https://github.com/phantom-video/Phantom)
- **Index-AniSora** (Bilibili, 2025): open anime video generator covering series episodes, donghua, manga adaptations, VTuber content, PVs and more. AniSora V3 weights are Apache-2.0 "with additional model usage restrictions". — [bilibili/Index-anisora](https://github.com/bilibili/index-anisora); [GIGAZINE](https://gigazine.net/gsc_news/en/20250519-anisora/)
- "Aligning Anime Video Generation with Human Feedback" (Apr 2025) applies RLHF-style alignment to anime video. — [Awesome-Animation-Research](https://github.com/zhenglinpan/Awesome-Animation-Research)
- **Kling:** Elements uses 1–4 images for character consistency, and scenes or clothing can be set as elements. Video O1 adds multi-subject fusion. Kling 3.0 adds "Subject Binding" and an "Elements 3.0 asset library" to anchor faces and "clothing textures" across shots. These are vendor claims. — [Kling consistency guide](https://app.klingai.com/global/quickstart/ai-video-character-consistency); [Kling 3.0 subject binding](https://kling.ai/blog/kling-3-subject-binding-character-consistency); [Kling O1 guide](https://kling.ai/quickstart/klingai-video-o1-user-guide)
- **Vidu reference-to-video:** up to 7 reference images; the vendor states faces and clothes "can look different" across shots without references. — [Vidu](https://www.vidu.com/ai-reference-to-video)
- **Sora 2** (launched 30 Sep 2025; 1080p, up to 20 s with sound) quickly became known for anime-style output. CODA (signatories including Studio Ghibli, Square Enix, Bandai Namco, Aniplex, Kadokawa, Shueisha) formally asked OpenAI to stop using Japanese IP for training. — [Hypebeast](https://hypebeast.com/2025/11/openai-sora-2-faces-coda-challenge-from-studio-ghibli-square-enix); [TechTimes](https://www.techtimes.com/articles/312456/20251103/japanese-media-giants-accuse-openai-copyright-infringement-over-sora-2-training-data.htm)

### Inferences
- For **manga fidelity**, sketch-conditioned colorization/in-betweening (LVCD → AniDoc → AnimeColor/ToonComposer → LongAnimation/InstanceAnimator) is the right family. Clothing structure is dictated by line art, and these works directly attack temporal colour drift. Free text/image-to-video models are better suited to B-roll or previs.
- **Typical failure modes**, synthesised from the papers' problem statements and issues:
  - colour bleeding between adjacent garment regions under large motion;
  - colour drift over long sequences (LongAnimation's motivation);
  - small accessories lost under occlusion;
  - pattern "swimming" in diffusion video. AnimateDiff-class models are the worst case.
- **Licensing chain matters.** AniDoc and LVCD depend on Stable Video Diffusion weights, and LongAnimation on CogVideoX-1.5-5B weights. Their effective commercial status is governed by those base-model licenses, not just the Apache code. From prior knowledge: SVD is under the Stability AI Community License (revenue-gated), and CogVideoX-5B has its own model license. Both need verification.
- AnimeColor's GitHub org (IamCreateAI) appears to be linked to CreateAI, the technical backer of Japan's Animon.ai (Section 8). That would mean academic anime colorization research and commercial Japanese AI-anime services are converging. This link is inferred from naming and was not confirmed.

### Gaps
- No independent, quantitative comparison of Kling / Veo 3.x / Sora 2 / Wan 2.x / Vidu on clothing stability (pattern drift, accessory loss) was found. Evidence is vendor claims and informal guides.
- Could not verify the AnimeColor base model (likely CogVideoX), or Index-AniSora's exact additional usage restrictions.
- Veo 3/3.1 anime-specific consistency information was not found.

---

## 6. Multi-character and multi-outfit consistency; handling outfit changes between scenes

### Takeaway
Multi-character costume consistency is an active 2025–2026 research topic. Image approaches: DiffSensei for manga panels, Nano Banana Pro or FLUX.2 multi-reference with per-character slots, IP-Adapter attention masks. Video approaches: InstanceAnimator multi-instance colorization, AnimeShooter multi-shot reference-guided generation, Kling and Vidu multi-subject references. Outfit changes between scenes are best handled by treating each costume as a **separate reference asset** (or separate LoRA trigger), selected per shot from a script-level costume schedule.

### Cited Findings
- **DiffSensei** (CVPR 2025): customised manga generation with multiple characters. An MLLM acts as a text-compatible identity adapter, with masked cross-attention for layout-controlled multi-character injection. Introduces the MangaZero dataset for "multi-character, multi-state" manga. — [arXiv 2412.07589](https://arxiv.org/abs/2412.07589); [CVPR paper](https://openaccess.thecvf.com/content/CVPR2025/papers/Wu_DiffSensei_Bridging_Multi-Modal_LLMs_and_Diffusion_Models_for_Customized_Manga_CVPR_2025_paper.pdf)
- **AnimeShooter** (Jun 2025): a reference-guided multi-shot animation dataset with story-level character profiles (appearance descriptions plus reference images for 1–3 main characters) and shot-level annotations of which characters appear. The baseline AnimeShooterGen feeds the reference plus previously generated shots through an MLLM to condition the next shot. It maintains appearance across shots "unlike baselines which struggle with IP features and shot-to-shot consistency". — [arXiv 2506.03126](https://arxiv.org/abs/2506.03126); [qiulu66/Anime-Shooter](https://github.com/qiulu66/Anime-Shooter); [HF dataset](https://huggingface.co/datasets/qiulu66/AnimeShooter)
- **InstanceAnimator**: per-instance (character and background) references on a canvas for multi-character colorization. — [arXiv 2603.25357](https://arxiv.org/html/2603.25357)
- **MangaNinja**: supports "multi-reference harmonization" and cross-character colorization through point control. — [arXiv 2501.08332](https://arxiv.org/html/2501.08332v1)
- **Nano Banana Pro**: up to 5 human identity references plus 6 object references, which allows per-costume object references. — [Scenario help](https://help.scenario.com/articles/7568607761-gemini-image-models-nano-banana-family)
- **Kling Elements**: "set scenes or clothings as elements". Kling O1 "independently locks onto and preserves the unique features of every character and prop even in complex ensemble scenes" (vendor claim). — [Kling consistency guide](https://app.klingai.com/global/quickstart/ai-video-character-consistency); [Kling O1](https://kling.ai/quickstart/klingai-video-o1-user-guide)
- Related 2025–2026 story/multi-shot works: Story2Board, Lay2Story (Aug 2025), InfinityStory (Mar 2026, character-aware shot transitions), and position-embedding-as-context-controller for multi-reference multi-shot video (Apr 2026). — [Awesome-Animation-Research](https://github.com/zhenglinpan/Awesome-Animation-Research); [arXiv 2603.03646](https://arxiv.org/html/2603.03646); [arXiv 2604.03738](https://arxiv.org/pdf/2604.03738)
- Character LoRAs with several outfits suffer cross-outfit bleed even with per-outfit folders and tags. — [hollowstrawberry guide discussion](https://huggingface.co/hollowstrawberry/stable-diffusion-guide/discussions/7)

### Inferences
- **Practical design:** keep a "costume bible" keyed by (character, costume ID, scene range), with these components:
  - a colour sheet with exact RGB per region;
  - front/back/side turnaround references;
  - an accessory checklist (number of pins, ribbons, earrings, and so on);
  - a canonical tag block.

  At generation time each shot pulls only the active costume's assets, using reference slots, masks or LoRA triggers. Mixing costumes in a single reference set invites bleed.
- **Multi-character attribute leakage**, where one character's garment colour migrates to another, is the multi-character analogue of LoRA outfit bleed. Masked attention (DiffSensei, IP-Adapter masks) or instance-level references (InstanceAnimator) is the mitigation.

### Gaps
- No quantitative study found on cross-character clothing-attribute leakage rates in anime generation.
- No published workflow found that handles mid-scene costume changes (e.g., a transformation sequence) beyond per-shot reference switching.

---

## 7. Evaluation: how to measure clothing consistency

### Takeaway
No anime-specific clothing-consistency metric is standard. The best-supported practice, borrowed from virtual try-on research, is to segment or crop the garment and compute **DINO (structure-sensitive) and CLIP-I (semantic) similarity** against the reference garment. Add video metrics (VFID/FVD, VGID for garment texture) and colour-fidelity checks, plus human evaluation. Face metrics such as AuraFace (used by FLUX Kontext) do not measure costumes.

### Cited Findings
- Try-on evaluation crops garment objects by masking with white-background normalisation, then computes **M-DINO** and **M-CLIP-I** similarity. DINO features "capture fine-grained local structure", making the score sensitive to garment-part structure. CLIP-I reflects category-level coherence. M-DINO tends to score lower because it penalises geometric variation that CLIP ignores. — [OmniTry, arXiv 2508.13632](https://arxiv.org/pdf/2508.13632); see also [CtrlVTON arXiv 2607.09362](https://arxiv.org/html/2607.09362)
- **VGID (Video Garment Inception Distance)** was proposed to quantify texture and structure preservation of garments in close-up video try-on. — [Eevee, arXiv 2511.18957](https://arxiv.org/html/2511.18957)
- **VFID** (I3D-feature Fréchet distance) measures video quality and temporal consistency in video try-on. — [WildVidFit, ECCV 2024](https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/02554.pdf); [Video VTON with DiT inpainter arXiv 2506.21270](https://arxiv.org/pdf/2506.21270)
- Animation colorization papers use LPIPS, SSIM, PSNR, FVD and FID against ground-truth coloured frames, on short (14-frame) and long (~500-frame) sets. — [LongAnimation](https://arxiv.org/abs/2507.01945)
- FLUX Kontext reports AuraFace identity similarity (~0.908) over multi-turn edits (face only). — [arXiv 2506.15742](https://arxiv.org/html/2506.15742v2)
- AnimeAdapter introduced a structured anime character editing benchmark (taxonomy prompts, OpenPose conditions, token-level masks) and also reports DreamBench++. — [arXiv 2605.20237](https://arxiv.org/abs/2605.20237)
- ToonComposer built PKBench from human-drawn sketches. — [arXiv 2508.10881](https://arxiv.org/abs/2508.10881)

### Inferences
- **Proposed costume-consistency protocol** for a manga-to-anime pipeline (synthesis; colour-histogram and ΔE metrics are standard image-processing practice and not taken from a cited anime paper):
  1. Garment segmentation per frame (e.g., SAM-family segmenter prompted by garment class).
  2. M-DINO and M-CLIP-I similarity of each garment crop vs. the canonical costume reference.
  3. Per-region palette fidelity: CIEDE2000 ΔE between dominant region colours and the colour-sheet RGB, or a colour-histogram distance (χ²/EMD).
  4. Temporal stability: frame-to-frame garment-crop DINO variance and optical-flow-warped error (flicker).
  5. Accessory presence: detector or VLM checklist per shot.
  6. Human eval: side-by-side against the model sheet, with a pass/fail on design errors.
- Ground truth for colour exists only on the colour design sheet, not in the manga. Metrics against GT anime frames, as in the papers, are therefore a proxy.

### Gaps
- No anime-domain garment-consistency benchmark or metric was found. VGID and M-DINO were validated on photographic try-on only.
- No published human-evaluation protocol specific to anime costume fidelity was found.

---

## 8. Real-world usage, costume handling, controversies and reception (plus 2026 state of the art and licensing)

### Takeaway
Industrial use as of 2026 is mostly **assistive**: backgrounds, colour specification and correction, in-betweening, photo-to-anime backgrounds. Character designs stay human-drawn, which is effectively how productions keep costumes consistent. Every visible "AI anime" project has met strong backlash. The flagship cases are Netflix/WIT's *The Dog & the Boy* (2023), *Twins Hinahima* (2025), Toei's 2025 AI roadmap, and Sora 2's copyright dispute.

### Cited Findings
**Productions and studios**
- ***The Dog & the Boy*** (Netflix Japan / WIT Studio, released 31 Jan 2023): the background images of all cuts used image generation ("as an experimental effort to help the anime industry, which has a labor shortage"). rinna Inc. and AI researchers are credited, and the background designer credit reads "AI (+Human)". There was broad backlash over avoiding paying artists. Characters were not AI-generated. — [Engadget](https://www.engadget.com/netflixs-dog-and-boy-anime-short-causes-outrage-for-incorporating-ai-generated-backgrounds-203035524.html); [Cartoon Brew](https://www.cartoonbrew.com/shorts/netflix-japan-ai-dog-and-boy-225631.html); [Twinfinite](https://twinfinite.net/news/netflixs-new-anime-sparks-controversy-for-using-ai-generated-artwork-backgrounds/)
- ***Twins Hinahima*** (Frontier Works and KaKa Creation, animated by KaKa Technology Studio):
  - Announced Dec 2024; a 24-min TV special/short that aired in late March 2025 (sources say 28 or 30 March). Based on a TikTok animated account started Nov 2023.
  - Director Kō Nakano; character designs by Takumi Yokota.
  - The team claims over 95% of cuts were AI-assisted or generated, using "supportive AI" with human animators finalising images.
  - Characters were hand drawn in Clip Studio Paint. Backgrounds were photographs converted to anime style with AI, then retouched by art staff.
  - Reception was mixed: movement described as rigid, with "inconsistencies and uncanny looks in some frames".

  — [ANN announcement](https://www.animenewsnetwork.com/news/2024-12-14/frontier-works-kaka-creation-reveal-twins-hinahima-ai-anime/.219056); [Wikipedia](https://en.wikipedia.org/wiki/Twins_Hinahima); [Screen Rant](https://screenrant.com/japan-first-ai-anime-defends-criticism-before-release-twins-hinahima/); [BusinessMirror](https://businessmirror.com.ph/2025/04/12/anime-using-ai-gets-mixed-reactions-after-first-episode-release/); [FandomWire](https://fandomwire.com/twins-hinahima-could-have-turned-into-a-sleeper-hit-simply-by-hiding-its-most-controversial-element/)
  - An aggregator blog states the production used Stable Diffusion; this is low-confidence and unverified against a primary source. — [aifilms blog](https://studio.aifilms.ai/blog/japanese-anime-studios-ai-adoption)
- **Toei Animation** (May 2025 financial report):
  - Plans to apply AI to simple storyboard layouts, **colour specification and automatic colour correction**, line-drawing correction, in-between generation, and backgrounds from photos.
  - Invested in Preferred Networks and intends to use generative AI from Preferred Computing Infrastructure (operations from early 2026).
  - Fan and animator backlash led the studio to backtrack and clarify.

  — [Gizmodo](https://gizmodo.com/animation-studio-toei-wants-to-use-ai-for-future-productions-2000603817); [AniTrendz](https://www.anitrendz.com/news/2025/05/17/toei-animation-touches-on-ai-use-in-anime-production-in-new-financial-report); [Screen Rant](https://screenrant.com/toei-animation-ai-anime-backlash-response-future-plans/); [OECD.AI incident](https://oecd.ai/en/incidents/2025-05-16-ffba)
- **Animon.ai** (Animon Dream Factory, Kumamoto, est. 2024; technical support from CreateAI Holdings): uses the proprietary "Ruyi" model, claimed to have strong frame consistency and colour expression. Launched in Japan in 2025 for standard anime production tasks. — [AICU note](https://note.com/aicu/n/nc164f7d9d074?hl=en)
- **Production I.G** reportedly partnered with startup Alpaca on storyboard analysis and audience-reaction prediction, not image generation. This comes via a survey/aggregator summary and is low confidence. — [ScienceDirect survey](https://www.sciencedirect.com/org/science/article/pii/S1526149225002991)
- **Sora 2 / CODA** (Oct–Nov 2025): Japanese rights holders demanded opt-in authorisation, alleging outputs closely resembling Dragon Ball, Pokémon and Ghibli works. — [Hypebeast](https://hypebeast.com/2025/11/openai-sora-2-faces-coda-challenge-from-studio-ghibli-square-enix); [Push Square (Sony)](https://www.pushsquare.com/news/2025/10/sony-stands-with-japans-creators-in-ai-copyright-crackdown)

**Licensing quick reference (verified from LICENSE/README files or model cards unless noted)**

| Tool / model | License | Commercial note |
|---|---|---|
| IDM-VTON, OOTDiffusion, CatVTON, StableVITON | CC BY-NC-SA 4.0 | Non-commercial |
| NoobAI-XL (and Illustrious early-release lineage) | fair-ai-public-license-1.0-sd (share-alike); NoobAI adds a no-commercialisation clause | Avoid for commercial use ([model card](https://huggingface.co/Laxhar/noobai-XL-1.1/raw/main/README.md)) |
| Animagine XL 4.0 | CreativeML Open RAIL++-M | Use-based restrictions only ([HF](https://huggingface.co/cagliostrolab/animagine-xl-4.0)) |
| Pony V7 | Pony License | Free under US$1M revenue; otherwise contact the authors ([HF](https://huggingface.co/purplesmartai/pony-v7-base)) |
| Neta Lumina | Apache-2.0 | Commercial OK ([HF](https://huggingface.co/neta-art/Neta-Lumina/blob/main/README.md)) |
| FLUX.2 [dev] | Non-commercial open weights | Paid commercial self-host license available ([BFL](https://bfl.ai/blog/flux-2)) |
| FLUX.2 [pro]/[flex] | Proprietary API | — |
| Qwen-Image / Qwen-Image-Edit code | Apache-2.0 | — |
| Wan2.2, VACE, Phantom, OmniGen2, UNO | Apache-2.0 | — |
| IP-Adapter, PhotoMaker, StoryDiffusion | Apache-2.0 | — |
| AnimateDiff, FramePack (code) | Apache-2.0 | — |
| ToonCrafter, AnimeColor, LongAnimation, Cobra (code) | Apache-2.0 | ToonCrafter README: "research exploration, instead of commercial products" |
| HunyuanVideo / -I2V / HunyuanCustom | Tencent Hunyuan Community License | Not valid in EU, UK, South Korea |
| Index-AniSora V3 | Apache-2.0 plus additional model-use restrictions | See [repo](https://github.com/bilibili/index-anisora) |
| AniDoc | Apache badge in README, no LICENSE file at root | Unclear |
| LVCD, MangaNinja, Any2AnyTryon, In-Context-LoRA | No LICENSE file found | Treat as unlicensed / all rights reserved. In-Context-LoRA defers to the FLUX license. |

### Inferences
- **State of the art as of late 2026**, for costume-faithful manga-to-anime generation:
  - **Key frames:** human cleanup of manga line art, then correspondence-based colorization against a colour design sheet (MangaNinja / Cobra / paint-bucket matching), or an anime SDXL model plus character/outfit LoRA plus lineart ControlNet.
  - **Edits and variations:** multi-reference in-context editors (FLUX.2, Qwen-Image-Edit-2511, Nano Banana Pro, Kontext).
  - **Motion:** sketch-conditioned video colorization and in-betweening (ToonComposer, AnimeColor, LongAnimation for long shots, InstanceAnimator for multi-character), with reference-to-video services (Kling 3.0/O1, Vidu, Wan2.2-Animate) for less critical shots.
  - **QA:** garment-crop DINO/CLIP plus palette ΔE plus human check.
- Every industry case so far keeps **character design human-made**. AI was applied to backgrounds (Dog & the Boy, Twins Hinahima) or proposed for colour, in-betweens and backgrounds (Toei). Studios have so far avoided letting generative models originate costume detail, plausibly because of both quality (drift) and labour/IP backlash.
- Licensing is a hard constraint. Many of the best try-on tools are non-commercial, and the most popular anime base-model lineage (Illustrious/NoobAI) restricts commercialisation. Production use pushes toward Apache-2.0 stacks (Wan2.2, Qwen-Image, Neta Lumina, Animagine under OpenRAIL) or paid APIs/licenses (FLUX.2, Kling, Gemini).

### Gaps
- No primary source (studio interview or making-of) found describing how *Twins Hinahima* or any AI anime handled **costume** consistency specifically. Primary interviews (Oricon, CBR, Cartoon Brew) were blocked.
- Could not confirm the Cartoon Brew article on "a Japanese studio embracing AI in its pipeline" (studio name and details unread).
- Utopai Studios and Toei's 2026 deployment status were not researched in depth.
- Could not verify current licenses of Stable Video Diffusion and CogVideoX-1.5 weights (base models for AniDoc/LVCD and LongAnimation), FLUX.1 Kontext [dev], or Illustrious XL v1+.
