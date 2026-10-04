# Building an End-to-End Clothing System for a Manga-to-Anime Pipeline: Architecture, Data, Infrastructure, Costs, Legal/Ethical (as of Oct 2026)

Scope note: these notes cover how to BUILD the system (architecture, open-source building blocks, outfit-database schema, compute/cost, QA, legal/licensing, MVP). Some primary sources could not be fetched because the network proxy blocked them (bunka.go.jp, spheron.network, animecorner.me, techjacksolutions.com). Where that happened, the claim comes from a secondary source and is labelled as such.

---

## 1. Existing end-to-end or partial systems, what stages they cover, and where they fail for clothing

### Takeaway
No open-source or commercial system covers the whole path from manga page to colored, animated, costume-consistent anime. The available pieces each cover one stage. Manga parsing has Magi. Manga generation has DiffSensei. Reference-based colorization has MangaNinja, LVCD and BasicPBC. Line-art video colorization has AniDoc. In-betweening has ToonCrafter, AnimeInbet and ToonComposer. Anime video generation has AniSora and Wan 2.2/Wan-Animate. Layer decomposition has See-through and LayerAnimate. For clothing, the shared weaknesses are low working resolution (512px-class), short clip lengths (14–16 frames), and no persistent per-outfit memory across shots. Studios that do use AI apply it to narrow stages (coloring, backgrounds) with humans finishing the work.

### Cited Findings
**Research/open-source building blocks, by stage**
- **Magi ("The Manga Whisperer", CVPR 2024)**: processes a high-resolution manga page to detect characters, text blocks and panels, order the panels, cluster characters (identity), match dialogue to speakers, and run OCR. MIT license. Magiv3 adds speech-bubble tails and grounding of characters in captions. — [GitHub ragavsachdeva/magi](https://github.com/ragavsachdeva/magi); [CVPR 2024 paper](https://openaccess.thecvf.com/content/CVPR2024/papers/Sachdeva_The_Manga_Whisperer_Automatically_Generating_Transcriptions_for_Comics_CVPR_2024_paper.pdf)
- **DiffSensei (CVPR 2025)**: a "customized manga generation" framework. A multimodal LLM acts as a text-compatible identity adapter for a diffusion generator, and masked cross-attention handles layout control. It comes with the **MangaZero** dataset: 43,264 manga pages and 427,147 annotated panels. Code released Feb 2025. — [arXiv 2412.07589](https://arxiv.org/abs/2412.07589); [CVPR open access](https://openaccess.thecvf.com/content/CVPR2025/html/Wu_DiffSensei_Bridging_Multi-Modal_LLMs_and_Diffusion_Models_for_Customized_Manga_CVPR_2025_paper.html); [GitHub](https://github.com/jianzongwu/DiffSensei/blob/main/README.md)
- **MangaNinja (CVPR 2025 Highlight, Alibaba ali-vilab)**: reference-based line-art colorization. A patch-shuffling module learns correspondence between the reference and the target, and a point-driven control lets users click matching points for fine color matching. Based on SD 1.5 at **512×512** inference. License **CC BY-NC 4.0, non-commercial**. Code released Jan 15, 2025. A ComfyUI node exists. — [GitHub ali-vilab/MangaNinjia](https://github.com/ali-vilab/MangaNinjia); [arXiv 2501.08332](https://arxiv.org/html/2501.08332v1); [ComfyUI_MangaNinjia](https://github.com/smthemex/ComfyUI_MangaNinjia)
- **AniDoc**: colors line-art *video* from a reference character design. Built on Stable Video Diffusion with a ControlNet and CoTracker2. Apache 2.0. It expects 14-frame input (the authors say it often works up to about 72 frames). Inference uses about 14 GB VRAM; training used 8× A100 80GB. — [GitHub yihao-meng/AniDoc](https://github.com/yihao-meng/AniDoc); [project page](https://yihao-meng.github.io/AniDoc_demo)
- **ToonCrafter (SIGGRAPH Asia 2024)**: generative cartoon interpolation between two keyframes using video-diffusion priors. Output tops out at **512×320 and 16 frames**. Apache-2.0. — [GitHub Doubiiu/ToonCrafter](https://github.com/Doubiiu/ToonCrafter); [Medium overview](https://medium.com/@aitoolscan/exploring-tooncrafter-revolutionizing-generative-cartoon-interpolation-a72e5c7cc75a)
- Other stage-specific tools listed in the curated Awesome-AI4Animation list (ICCVW 2025):
  - Colorization: LVCD (reference lineart video colorization, TOG 2024) and BasicPBC (paint-bucket colorization via inclusion matching, CVPR 2024).
  - In-betweening: AnimeInbet (ICCV 2023), ToonComposer (generative post-keyframing) and Framer (interactive interpolation).
  - Layering: LayerAnimate (layer-specific control).
  - Character sheets and storyboards: CoNR (collaborative neural rendering from anime character sheets, IJCAI 2023), Anim-Director (an LMM agent for animation, SIGGRAPH Asia 2024), StoryWeaver and SEED-Story.
  - Datasets: Sakuga-42M, AnimeRun.
  - [Awesome-AI4Animation](https://github.com/yunlong10/Awesome-AI4Animation)
- **DACoN (ICCV 2025)**: DINO-based anime paint-bucket colorization that takes any number of references. — [CVF paper](https://openaccess.thecvf.com/content/ICCV2025/papers/Nagata_DACoN_DINO_for_Anime_Paint_Bucket_Colorization_with_Any_Number_ICCV_2025_paper.pdf)
- **Sakuga-42M**: 42M keyframes from more than 150,000 hand-drawn cartoon videos, released "for academic and research purposes". — [GitHub SakugaDataset](https://github.com/KytraScript/SakugaDataset); [summary](https://www.emergentmind.com/topics/sakuga-42m)
- **See-through (SIGGRAPH 2026)**: decomposes a single anime illustration into up to **23 inpainted semantic layers** (hair, face, eyes, clothing, accessories…) with a drawing order, producing 2.5D/Live2D-style models. Labels were bootstrapped with GradCAM, SAM and the Live2D renderer, and SAM2 is an optional annotator. — [arXiv 2602.03749](https://arxiv.org/html/2602.03749v1); [GitHub shitagaki-lab/see-through](https://github.com/shitagaki-lab/see-through)
- **SegAnimeChara**: segments anime/game characters, including clothed accessories, by combining BodyPix boxes, SAM and RegionCLIP semantic prompts. — [ACM](https://dl.acm.org/doi/fullHtml/10.1145/3588028.3603685)
- **WD14 Tagger (SmilingWolf)** is described as the de facto standard anime auto-tagger that outputs Danbooru tags, which makes it useful for garment tagging. — [tagging guide](https://www.mohsindev369.dev/blog/how-to-tag-images-for-lora-training)
- A SegFormer-B2 "clothes" segmentation model ships as a ComfyUI_LayerStyle asset. It is trained on photos, so domain transfer to anime is a concern. — [HF README](https://huggingface.co/chflame163/ComfyUI_LayerStyle/raw/main/ComfyUI/models/segformer_b2_clothes/README.md)

**Video models**
- **Wan 2.2 (Alibaba, July 28 2025, Apache 2.0)** models: T2V-A14B and I2V-A14B (MoE, 480P/720P), TI2V-5B (720P/24fps, which runs on consumer GPUs), S2V-14B, and **Animate-14B** (animation and character-replacement modes). The README warns that Wan2.2-trained LoRAs "may lead to unexpected behavior" with Animate. The license text says: "We claim no rights over the your generated contents." — [GitHub Wan-Video/Wan2.2](https://github.com/Wan-Video/Wan2.2); [HF model card](https://huggingface.co/Wan-AI/Wan2.2-T2V-A14B/blob/main/README.md)
- Later flagship Wan versions (2.5, 2.6, 2.7, 3.0) shipped **closed, API-only**. Wan 2.2 is the last open-weights flagship. Wan-Animate-2 was reportedly released Aug 7, 2026 under Apache 2.0. These are secondary sources and the Wan-Animate-2 claim is unverified at primary level. — [Atlas Cloud blog](https://www.atlascloud.ai/blog/tips/is-wan-3.0-open-source); [wan27.org](https://wan27.org/blog/wan-2-6-open-source-guide)
- **Index-AniSora (Bilibili)**: open-source anime video generation covering series episodes, manga adaptations, VTuber content and PVs. Code is Apache 2.0, "with additional model usage restrictions". A V3 has been released. — [project page](https://bilibili.github.io/Index-anisora/); [GIGAZINE](https://gigazine.net/gsc_news/en/20250519-anisora/)

**Commercial/industry deployments**
- **Twins Hinahima** (KaKa Creation × Frontier Works; aired Mar 28–29, 2025 on Tokyo MX/MBS) was billed as Japan's first generative-AI anime, with ">95%" AI involvement. The tools listed were UE5, CLIP STUDIO PAINT, Photoshop, Illustrator and After Effects. AI generated backgrounds and character illustrations, and humans finalized them ("supportive AI"). — [Anime News Network](https://www.animenewsnetwork.com/news/2025-02-28/frontier-works-kaka-creation-twins-hinahima-ai-anime-reveals-march-29-tv-debut/.221769); [Wikipedia](https://en.wikipedia.org/wiki/Twins_Hinahima)
- **Toei Animation**'s FY2025 briefing named planned AI uses: storyboard layouts, color specification and automatic color correction, line correction and in-between generation, and backgrounds from photos. It is investing in Preferred Networks. After backlash, Toei revised the document to say it is "not currently using" AI in those processes. — [Gizmodo](https://gizmodo.com/animation-studio-toei-wants-to-use-ai-for-future-productions-2000603817); [GameRant](https://gamerant.com/toei-denies-using-ai-following-backlash/); [AniTrendz](https://www.anitrendz.com/news/2025/05/17/toei-animation-touches-on-ai-use-in-anime-production-in-new-financial-report)
- **OLM Digital** reportedly uses an automatic section-coloring AI tool, with about 30% work-hour reduction in some cases. This comes via an aggregator because the primary Anime Corner article was blocked. — [search summary of Anime Corner](https://animecorner.me/olm-digital-anime-studio-ai-artificial-intelligence/)
- **CLIP STUDIO PAINT (Celsys)** announced a Stable-Diffusion "Image Generator Palette" in Nov 2022 and withdrew it 3 days later after backlash. Celsys says its existing **Colorize** feature is trained on a "clean data set" of illustrations from creators who explicitly consented. — [Celsys news](https://www.clipstudio.net/en/news/202211/29_01/); [ARTnews](https://www.artnews.com/art-news/news/clip-studio-paint-ai-backlash-1234649301/); [Clip Studio on X](https://x.com/clipstudiopaint/status/1760284266272031138)

### Inferences
- **Clothing failure points:**
  - MangaNinja's 512² working resolution and ToonCrafter's 512×320 output are too coarse to hold small costume details (buttons, emblems, plaid repeats, lace) at broadcast resolution. Expect pattern drift, and plan upscaling plus human touch-up or flat-color paint-bucket methods (BasicPBC/DACoN), which respect line-art regions.
  - The 14–16-frame windows in AniDoc and ToonCrafter mean costume colors can drift between windows unless every window is conditioned on the same outfit reference sheet. The outfit database (section 3) is the mechanism that supplies that sheet.
  - Manga pages are mostly monochrome with screentone, so garment *color* cannot come from the source except on color pages. A human color designer, or the licensor's existing color art, has to set the palette before any model runs.
  - Wan-Animate's LoRA caveat means per-outfit LoRAs trained on the base Wan2.2 cannot be assumed to carry over to the character-replacement path. Test this per model.
- The licensing mix matters for a commercial pipeline. MangaNinja (CC BY-NC) and Sakuga-42M (research-only) suit prototyping but not shipped production. AniDoc, ToonCrafter, Wan 2.2 and Magi are permissive (Apache/MIT).

### Gaps
- I did not find a commercial "manga to anime AI" product with a verifiable studio-grade clothing-consistency track record. Consumer web apps exist, but none had credible documentation.
- I could not verify Toon Boom Harmony's current AI features, or any 2026 CelSys/Clip Studio AI feature beyond Colorize.
- MangaZero's license terms were not checked.

---

## 2. Reference architecture (stage by stage, with human-in-the-loop) and integration with production tracking

### Takeaway
The practical architecture is a stage-gated pipeline that mirrors traditional anime roles: settei (model sheets), iro-shitei (color design), genga, douga, shiage (finishing), satsuei (compositing), with AI assisting inside each stage. A human approval gate sits at each costume-relevant boundary. A production tracker (Kitsu, which is open source, or Autodesk Flow Production Tracking, formerly ShotGrid) is the system of record. ComfyUI workers are driven through the tracker's REST/Python API.

### Cited Findings
- Toei's own planned AI insertion points map onto the traditional stages: storyboard layouts, color specification and correction, in-between correction and generation, and backgrounds. — [Gizmodo](https://gizmodo.com/animation-studio-toei-wants-to-use-ai-for-future-productions-2000603817)
- The sakuga (drawing) stage (genga + douga + finishing) is reported to take about 60–70% of an episode budget. This is a lower-quality blog source. — [CreativeFreaks](https://creativefreaks.net/en/how-much-does-it-cost-to-produce-japanese-anime-episode-budget-and-cost-optimization-guide-2/)
- **Kitsu (CGWire)** is an open-source production tracker covering:
  - shot/asset tracking, tasks and statuses, review playlists, annotations and version comparison
  - all production data exposed through a **REST API and Python SDK**
  - integrations with Blender, **Toon Boom Harmony**, Unreal, Slack and Discord
  - No official ComfyUI integration was found.
  - [CGWire Kitsu](https://www.cg-wire.com/kitsu/); [custom integration FAQ](https://www.cg-wire.com/faq/custom-integration/)
- **Autodesk Flow Production Tracking**: ShotGrid was renamed in March 2024. Autodesk added AI "Flow Generative Scheduling" and rebranded Wonder Studio as **Flow Studio** (AI CG-character insertion). — [Autodesk product page](https://www.autodesk.com/products/flow-production-tracking/overview); [CG Channel on Generative Scheduling](https://www.cgchannel.com/2024/07/autodesk-launches-flow-generative-scheduling/); [CG Channel on Flow Studio](https://www.cgchannel.com/2025/03/wonder-studio-becomes-autodesk-flow-studio/)
- Weak inspection is a real risk. **WIT Studio** apologized on April 10, 2026 after generative AI was found in some background cuts of the *Ascendance of a Bookworm* Part 3 opening (aired April 4). It blamed "inadequacies in its production management and inspection systems", redrew the cuts, and replaced the opening from episode 2. Its policy in principle is not to allow gen-AI in production, with *The Dog & The Boy* as an experimental exception. — [Anime News Network](https://www.animenewsnetwork.com/news/2026-04-10/wit-studio-apologizes-for-using-generative-ai-in-opening-sequence-of-ascendance-of-a-bookworm-part-/.236271); [AUTOMATON](https://automaton-media.com/en/news/ascendance-of-a-bookworm-anime-studio-acknowledges-gen-ai-use-in-opening-promises-to-replace-offending-scenes-with-hand-drawn-art/)
- Building blocks per stage are listed in section 1 (Magi, See-through/SegAnimeChara/WD14, MangaNinja/BasicPBC/DACoN, AniDoc/LVCD, ToonCrafter/AnimeInbet/ToonComposer, Wan 2.2/AniSora, LayerAnimate). — [Awesome-AI4Animation](https://github.com/yunlong10/Awesome-AI4Animation)

### Inferences
Proposed reference architecture (my synthesis from the components above):
1. **Ingest.** Licensed manga scans or the publisher's digital originals go into object storage with immutable hashes and rights metadata (licensor, contract ID, permitted uses: training yes/no). Store color pages and covers separately because they are the only in-source color evidence.
2. **Panel/character detection and identity.** Magi produces panels, characters, identity clusters and reading order. Each detection becomes a `CharacterAppearance` row (page, panel, bbox, cluster ID, confidence). **HITL gate A:** a script supervisor confirms identity clusters per chapter.
3. **Clothing segmentation and tagging.**
   - SAM2 or SegAnimeChara-style prompting produces garment masks.
   - WD14 adds Danbooru attire tags.
   - See-through-style layer decomposition handles illustrations or model sheets.
   - Changes in the tag set between consecutive appearances raise an "outfit change" candidate event.
   - **HITL gate B:** a costume continuity reviewer approves outfit IDs and chapter ranges, including partial states (jacket removed, torn, wet).
4. **Outfit bible / model-sheet generation.** Each approved outfit gets turnaround or settei sheets: artist-drawn, or AI-drafted and then redrawn, plus reference crops. **HITL gate C:** the character designer signs off. That sign-off is the canonical reference used by every downstream model.
5. **Color design (iro-shitei).** The color designer sets normal, shadow and highlight (and optionally a second shadow) hex values for each garment region, with per-scene lighting variants (day, night, sunset). This is stored as a palette record, not as an image. **HITL gate D.**
6. **Adapters.** Train per-character or per-outfit LoRAs, or use reference-conditioning (MangaNinja-style, IP-Adapter-style), on the approved sheets only, never on unapproved generations.
7. **Keyframes (genga).** Human key animators, or AI-drafted keys with ControlNet pose/lineart control plus the outfit LoRA plus the palette. **HITL gate E:** animation director review.
8. **In-between/video.** Use ToonCrafter/AnimeInbet/ToonComposer for in-betweens, AniDoc/LVCD to color line sequences, and Wan 2.2 I2V/Animate for full-motion shots. Condition every window on the same outfit sheet.
9. **Finishing/paint.** Use a paint-bucket colorizer (BasicPBC/DACoN) that snaps region fills to exact palette hexes. This is the most reliable way to enforce costume colors.
10. **Compositing/satsuei.** Add layers (LayerAnimate/See-through-style separation) and lighting.
11. **QA.** Automated costume checks (section 5) feed a review playlist in Kitsu or Flow. **HITL gate F:** the color checker (検査) gives final approval. WIT's case shows that a logged AI-use declaration per cut is needed, not just a visual check.
12. **Tracker integration.**
    - Each ComfyUI job is a Kitsu/Flow task.
    - Outputs are published as versions carrying provenance metadata: model, version, LoRA IDs, seed, and the input asset hashes.
    - Status changes trigger the next stage through webhooks or the Python SDK.

### Gaps
- I found no public SIGGRAPH/CEDEC talk detailing how a Japanese studio wires generative tools into ShotGrid/Kitsu. CEDEC sessions were not searchable through this toolset.
- Ftrack's AI features were not researched.

---

## 3. Costume / outfit data schema ("outfit bible") and relevant standards

### Takeaway
No standard schema exists for anime costume continuity. The practical approach is a custom relational/JSON schema that borrows from three sources: Danbooru's hierarchical attire tag groups for garment vocabulary, VRM 1.0's in-file license/permission metadata for rights fields, and traditional iro-shitei practice for normal/shadow/highlight palettes.

### Cited Findings
- **Danbooru tag groups**: *Tag Group:Attire* covers headwear, shirts/topwear, pants/bottomwear, uniforms/costumes, and links to separate groups for accessories, handwear, headwear, legwear, sleeves and eyewear. It is a hierarchical, community-maintained ontology used by anime taggers such as WD14. — [Danbooru Tag Group:Attire](https://danbooru.donmai.us/wiki_pages/tag_group:attire); [Tag Groups index](https://danbooru.donmai.us/wiki_pages/tag_groups); [Japanese clothes](https://danbooru.donmai.us/wiki_pages/japanese_clothes)
- **VRM 1.0 meta** carries license and permission fields inside the asset file:
  - `avatarPermission` (onlyAuthor / onlySeparatelyLicensedPerson / everyone)
  - `commercialUsage`, `creditNotation`, `allowRedistribution`, `modification`
  - `allowExcessivelyViolentUsage`, `allowExcessivelySexualUsage`, `allowPoliticalOrReligiousUsage`, `allowAntisocialOrHateUsage`
  - Defaults are the most protective options.
  - [vrm-specification meta](https://github.com/vrm-c/vrm-specification/blob/master/specification/VRMC_vrm-1.0/meta.ja.md); [vrm.dev license](https://vrm.dev/en/vrm/meta/license/)
- See-through's 23-layer semantic decomposition (hair, face, eyes, clothing, accessories…) with drawing order is a ready-made garment-layer vocabulary for 2.5D assets. — [arXiv 2602.03749](https://arxiv.org/html/2602.03749v1)
- Magi gives character identity clusters and panel order, which are the keys that link outfit appearances to pages and panels. — [GitHub magi](https://github.com/ragavsachdeva/magi)

### Inferences
Recommended schema (synthesis, not an existing standard):
```
Character { character_id, names{ja,en,romaji}, rights_holder, design_version }
Outfit {
  outfit_id, character_id, name, canonical(bool), variant_of (outfit_id|null),
  state_tags [damaged, wet, sleeves_rolled, jacket_off],
  source_range { first_chapter, first_page, last_chapter, last_page },
  anime_range { episodes[], cuts[] },          # filled during production
  danbooru_tags [ "serafuku", "pleated_skirt", "red_neckerchief" ],
  garments: [ Garment ],
  accessories: [ Garment ],                    # same structure
  model_sheets: [ asset_uri (front/side/back/3-4), approved_by, approved_at ],
  reference_crops: [ {page, panel, bbox, mask_uri, image_hash} ],
  adapters: [ {type: LoRA|IP-Adapter-ref|embedding, base_model, base_model_license,
               uri, version, trigger_token, train_set_hashes[], train_config_uri} ],
  continuity_notes, approval_status, provenance{ai_assisted:bool, tools[]}
}
Garment {
  garment_id, slot (head|top|outer|bottom|legwear|footwear|hand|neck|accessory),
  danbooru_tag, layer_order (int, See-through style), material (cotton|leather|metal...),
  pattern { type: solid|stripe|plaid|print|emblem, scale, motif_asset_uri, orientation },
  colors: [ { region, normal_hex, shadow_hex, shadow2_hex?, highlight_hex,
              lighting_variant (day|night|sunset|indoor), color_designer, approved } ],
  line_color_hex (for colored-trace lines), notes
}
Palette scene overrides { scene_id/cut_id, outfit_id, lighting_variant, deltas }
Rights { source_work, license_contract_id, training_permitted, commercial_permitted,
         credit_notation }   # modelled on VRM meta fields
```
- Store colors as hex plus Lab values so ΔE checks can be computed (see section 5).
- Version every outfit record. Never overwrite an approved palette; add a new version with a change reason.
- OpenUSD (or glTF/VRM for 3D proxies) could wrap 3D costume proxies if the pipeline uses 3D guides. I did not research whether USD has any 2D-costume convention.

### Gaps
- I found no industry-published iro-shitei data format. Studios use proprietary color-chart sheets, and I could not locate a public spec.
- USD-for-2D-anime-assets was not researched or verified.

---

## 4. Compute, infrastructure and cost estimates

### Takeaway
LoRA training is cheap: single-digit to low-tens of dollars per character or outfit LoRA. Video generation dominates cost. Wan 2.2 14B at 720p takes roughly 10–12 H100-minutes per 5-second clip. One clean take of a 22-minute episode is therefore about 50 H100-hours, and 3–5 takes per shot for selection puts it at roughly 150–250 H100-hours, about $400–$900 on RunPod or about $1,000–$1,700 on AWS on-demand. That is small next to a $160k–$500k+ traditional episode budget, but human review and correction will dominate real cost.

### Cited Findings
**GPU prices (2026)**
- RunPod Secure Cloud, on demand: H100 80GB about $3.49/hr, A100 80GB about $1.59/hr, L40S about $1.09/hr. Community Cloud: H100 SXM about $2.69/hr, H100 PCIe about $1.99/hr, A100 SXM about $1.39/hr, L40S about $0.79/hr. Serverless equivalent: H100 about $4.55/hr, A100 about $2.72/hr. These are aggregator figures and prices fluctuate. — [RunPod GPU comparison](https://www.runpod.io/articles/comparison/choosing-gpus); [Northflank](https://northflank.com/blog/runpod-gpu-pricing); [computeprices](https://computeprices.com/providers/runpod)
- AWS p5.48xlarge (8× H100) is about $55.04/hr on demand, about **$6.88 per H100-hour**, after a reported 44% price cut in June 2025. Lambda H100 SXM is about $2.49–$3.44/hr. — [GMI Cloud on AWS P5](https://www.gmicloud.ai/en/blog/aws-p5-h100-pricing); [Thunder Compute on H100 pricing](https://www.thundercompute.com/blog/nvidia-h100-pricing)

**Video generation throughput**
- Wan 2.1/2.2 14B: a 5-second 720p clip takes about **10–12 min on an H100** and needs **65–80 GB** VRAM. 480p with FP8 needs about 40–48 GB. TI2V-5B makes a 5-second 720p/24fps clip on a 24 GB RTX 4090 in under about 9 min. These figures come via search summaries of Spheron/Hivenet; the primary page was blocked. — [Spheron GPU guide](https://www.spheron.network/blog/ai-video-generation-gpu-guide/); [Hivenet](https://www.hivenet.com/post/wan-2-2-cloud-gpu-comfyui); [Wan2.2 README (TI2V-5B under 9 min)](https://github.com/Wan-Video/Wan2.2)
- ToonCrafter tops out at 512×320 and 16 frames. AniDoc inference needs about 14 GB VRAM. MangaNinja's community Windows port reports about 6 GB VRAM. — [ToonCrafter](https://github.com/Doubiiu/ToonCrafter); [AniDoc](https://github.com/yihao-meng/AniDoc); [MangaNinja](https://github.com/ali-vilab/MangaNinjia)

**LoRA training**
- SDXL character LoRA: about 1,500–2,500 steps. Minimum 10 GB VRAM (batch 1, gradient checkpointing, AdamW8bit); 16–24 GB is comfortable. FLUX.1 LoRA of about 1,500 steps at 1024px takes about 1–3 h on an RTX 3090 and about 2.5–3 h on a 4090. ai-toolkit ships a 24 GB FLUX recipe. — [localaimaster](https://localaimaster.com/blog/image-lora-training-local-guide); [bestaiweb](https://www.bestaiweb.ai/how-to-train-a-custom-lora-for-flux-and-sdxl-with-kohya-ss-ai-toolkit-and-fal-ai-in-2026/)
- Hosted option: fal.ai's FLUX.2 [dev] trainer costs $0.008/step, so about **$8 per 1,000-step run**. — [bestaiweb](https://www.bestaiweb.ai/how-to-train-a-custom-lora-for-flux-and-sdxl-with-kohya-ss-ai-toolkit-and-fal-ai-in-2026/)
- AniDoc training used 8× A100 80GB. This is the scale needed to fine-tune video colorizers on a studio's own data. — [AniDoc](https://github.com/yihao-meng/AniDoc)

**Traditional baseline**
- A typical episode needs about 3,000 drawings. A standard 24-minute TV episode costs about $160k–$320k, and high-profile titles exceed $500k. Some reports cite up to ¥300M (about $2M) per episode for top productions. These are blogs and news aggregators of mixed quality. — [CreativeFreaks](https://creativefreaks.net/en/how-much-does-it-cost-to-produce-japanese-anime-episode-budget-and-cost-optimization-guide-2/); [Anitsu](https://anitsu.com/en/news/producing-an-anime-now-costs-2-million-dollars-per-episode/); [Anime Corner/ARCH CEO](https://animecorner.me/the-costs-of-anime-prices-of-episodes-have-skyrocketed-demonstrates-arch-ceo-nao-hirasawa/)

### Inferences
These are my calculations from the cited figures; treat them as order-of-magnitude.
- **Episode video generation:** 22 min = 1,320 s, which is 264 five-second clips. At 11 H100-min per clip that is about 2,900 H100-min, or **about 48 H100-hours per single take**. With 3–5 takes per shot that is about 145–240 H100-hours. At RunPod $2.69–3.49/hr that is **about $390–$840**. At AWS $6.88/hr it is **about $1,000–$1,650**. A 480p draft pass followed by a 720p final pass, or step-distilled variants, would lower this; I have not quantified that.
- **Drawing equivalence:** 3,000–4,000 drawings over 1,320 s is about 2.3–3 unique drawings per second. That is consistent with limited animation on twos/threes plus holds. AI in-betweening at 24 fps generates more frames than needed, so output should be decimated to the show's timing chart to keep the anime look.
- **LoRA budget:** 10 main characters × 3–5 outfits gives 30–50 outfit LoRAs. At about $8–$15 each (hosted, or roughly 2–3 GPU-hours on an A100/L40S) the total is **about $250–$750** per series, plus retrains when designs change.
- **Image/keyframe generation** with SDXL/FLUX costs seconds per image on an L40S/A100. Even 20k keyframe candidates per episode is likely a few tens of GPU-hours, but I did not find an authoritative per-image benchmark.
- **Suggested infrastructure:**
  - 1–2 local RTX 4090/5090-class boxes for artist-facing interactive ComfyUI (colorization, keyframes).
  - Burst cloud H100s (RunPod/Lambda) for Wan 2.2 batch jobs.
  - S3-compatible object storage.
  - Postgres for the outfit bible.
  - Kitsu for tracking.
- Avoid AWS on-demand for bursty diffusion work unless enterprise compliance requires it.

### Gaps
- I found no authoritative benchmark for SDXL/FLUX images per hour on H100/L40S in 2026, or for distilled Wan 2.2 speedups.
- I found no published total-cost case study of a fully AI-assisted anime episode. Twins Hinahima did not disclose costs.
- Storage, egress and human review labor costs were not estimated.

---

## 5. Evaluation metrics and QA for costume consistency

### Takeaway
Use embedding-similarity metrics (DINO, CLIP-I, DreamSim) on garment crops against the approved model sheet, plus exact palette checks: ΔE between generated region colors and the outfit bible hexes. Add mask/tag checks for missing or extra garments. Human review by a color checker and animation director stays the final gate.

### Cited Findings
- The standard automatic identity/subject-consistency metrics in character video generation research are:
  - **CLIP-I**: cosine similarity of CLIP image embeddings between the reference appearance and tracked regions.
  - **DINO-I**: DINO features capture fine-grained foreground semantics; used for frame-to-frame and multi-shot consistency.
  - **DreamSim**: perceptual similarity aligned with human judgment.
  - Some papers combine these into a harmonic score that penalizes weakness in any single one.
  - [Multi-Shot Character Consistency (arXiv 2412.07750)](https://arxiv.org/html/2412.07750v1); [AnimeGamer (arXiv 2504.01014)](https://arxiv.org/pdf/2504.01014); [Phantom (arXiv 2502.11079)](https://arxiv.org/pdf/2502.11079)
- A curated index of visual-generation evaluation metrics and benchmarks exists. — [Awesome-Evaluation-of-Visual-Generation](https://github.com/ziqihuangg/Awesome-Evaluation-of-Visual-Generation)
- DINO-based correspondence already powers anime paint-bucket colorization (DACoN), so DINO features are suited to anime line/flat-color domains. — [DACoN ICCV 2025](https://openaccess.thecvf.com/content/ICCV2025/papers/Nagata_DACoN_DINO_for_Anime_Paint_Bucket_Colorization_with_Any_Number_ICCV_2025_paper.pdf)
- WD14 outputs Danbooru attire tags that can be diffed against the outfit record. — [tagging guide](https://www.mohsindev369.dev/blog/how-to-tag-images-for-lora-training)
- WIT Studio's 2026 incident shows that QA must include **process/provenance checks** (was AI used in this cut, and by whom), not only visual checks. — [Anime News Network](https://www.animenewsnetwork.com/news/2026-04-10/wit-studio-apologizes-for-using-generative-ai-in-opening-sequence-of-ascendance-of-a-bookworm-part-/.236271)

### Inferences
Proposed QA suite, run per cut and logged to the tracker:
1. **Garment presence/absence:** WD14 tags on a character crop compared with the outfit record's `danbooru_tags`. Flag missing items (e.g. neckerchief missing) and extras.
2. **Palette fidelity:** segment garment regions (SAM2, or paint-bucket region IDs), compute median Lab per region, and compute CIEDE2000 ΔE against normal/shadow/highlight. Thresholds such as ΔE<3 for flat-cel regions are my suggestion and need calibration.
3. **Identity/outfit similarity:** DINOv2 and DreamSim between garment crops and the approved model-sheet crops. Track per-frame similarity to catch temporal drift inside 14–16-frame windows and at window seams.
4. **Pattern checks:** for emblems and plaids, template matching or local-feature matching against `motif_asset_uri`.
5. **Temporal flicker:** frame-to-frame region-color variance.
6. **Human gates:**
   - Color checker (iro-kensa) reviews flagged cuts.
   - A sample of unflagged cuts (e.g. 10%) is checked to measure false-negative rate.
   - Animation director signs off.
7. **Provenance:** every published version records model, LoRA, seed and inputs, plus an AI-used flag and the reviewer, so the studio can answer "was gen-AI used here" exactly.

### Gaps
- I found no published benchmark specific to anime *costume* consistency. Existing metrics measure whole-character identity.
- No validated ΔE thresholds for anime cel colors were found.

---

## 6. Legal and ethical constraints

### Takeaway
Japan's Article 30-4 allows AI training on copyrighted works for non-"enjoyment" purposes. The Agency for Cultural Affairs' 2024 guidance narrows that allowance where the purpose includes enjoyment or reproducing specific expression; secondary sources specifically cite fine-tuning such as LoRA to imitate particular works. Outputs that are similar to and reliant on a work infringe. A manga-to-anime project therefore needs an **adaptation license from the rights holder** regardless of the AI exemption, ideally with explicit training permission written into the contract.

The US treats training under fact-specific fair use (USCO Part 3, May 2025). The EU AI Act Article 50 labeling duties have applied since Aug 2, 2026. Model licenses vary sharply:
- Wan 2.2: Apache-2.0, commercial OK.
- FLUX.1-dev: model use is non-commercial, outputs may be used commercially.
- HunyuanVideo: territory excludes the EU, UK and South Korea.
- NoobAI-XL: forbids commercial use even of outputs.
- MangaNinja: CC BY-NC.

The Japanese industry is publicly hostile to unlicensed training (CODA vs. Sora 2), and creator anxiety is high.

### Cited Findings
**Japan: copyright law and guidance**
- The Agency for Cultural Affairs published "General Understanding on AI and Copyright in Japan" (IBA says May 2024; the English overview PDF is on bunka.go.jp but was blocked). Article 30-4 permits non-expressive uses such as data analysis for training without authorization, so long as outputs do not replicate expressive works. — [IBA](https://www.ibanet.org/japan-emerging-framework-ai-legislation-guidelines); [ACA overview PDF](https://www.bunka.go.jp/english/policy/copyright/pdf/94055801_01.pdf)
- Article 30-4 does not apply where the purpose is enjoyment, or where the main purpose is non-enjoyment but an enjoyment purpose is also present. — [Clifford Chance](https://www.cliffordchance.com/insights/resources/blogs/talking-tech/en/articles/2023/10/Japanese-Law-Issues-Surrounding-Generative-AI.html); [AIPPI](https://www.aippi.org/news/generative-ai-development-copyright-protection/)
- The IBA summary states that "if models are fine tuned (eg, via LoRA) or trained on databases that imitate specific styles, the exemption no longer applies." Note: the ACA's own framing is reportedly about intent to output the creative expression of specific works, not style as such. Verify against the primary PDF; see Gaps. — [IBA](https://www.ibanet.org/japan-emerging-framework-ai-legislation-guidelines)
- **AI Promotion Act**, a light-touch statute: passed May 28, 2025, mostly effective June 4, 2025, fully effective Sept 1, 2025. It created the AI Strategic Headquarters. The Cabinet approved the AI Basic Plan on Dec 23, 2025. Copyright matters continue to be handled through interpretation of the existing Copyright Act. — [IBA](https://www.ibanet.org/japan-emerging-framework-ai-legislation-guidelines); [Zelo](https://zelojapan.com/en/lawsquare/56899); [vorplabs July 2026](https://vorplabs.com/ai-regulatory-updates/japan/2026-07/ai-promotion-act-basic-plan-business-guidance)

**Japanese industry stance and labor**
- **CODA**, whose members include Studio Ghibli, Bandai Namco, Square Enix, Shueisha, Toei Animation, Kadokawa, Aniplex and Cygames, asked OpenAI on **Oct 27, 2025** to stop training Sora 2 on members' works without prior permission and to respond to infringement claims. CODA argues that under Japanese law prior permission is required and that opting out after the fact does not cure liability. — [AUTOMATON](https://automaton-media.com/en/news/sonys-aniplex-bandai-namco-and-other-japanese-publishers-demand-end-to-unauthorized-training-of-openais-sora2-through-coda/); [GameSpot](https://www.gamespot.com/articles/studio-ghibli-and-japanese-game-publishers-demand-openai-stop-using-their-content-in-sora-2/1100-6535845/)
- A survey of about 25,000 Japanese creators (Freelance League of Japan; reported Jan 21, 2026):
  - About 89% feel AI is a significant threat to their livelihoods.
  - 12% report reduced income due to generative AI, citing shorter deadlines, lower fees and lost commissions.
  - About 70% of respondents work in drawing-related fields.
  - [Japan Times](https://www.japantimes.co.jp/news/2026/01/21/japan/society/ai-creatives-survey/)
- WIT Studio's 2026 apology and Toei's revised investor deck both show reputational risk around undisclosed AI use. — [ANN](https://www.animenewsnetwork.com/news/2026-04-10/wit-studio-apologizes-for-using-generative-ai-in-opening-sequence-of-ascendance-of-a-bookworm-part-/.236271); [GameRant](https://gamerant.com/toei-denies-using-ai-following-backlash/)
- Celsys withdrew a generative feature after backlash and positions Colorize as trained on consented creator data. That is a useful "clean data" precedent. — [ARTnews](https://www.artnews.com/art-news/news/clip-studio-paint-ai-backlash-1234649301/); [Clip Studio on X](https://x.com/clipstudiopaint/status/1760284266272031138)

**US / EU**
- The USCO Part 3 report ("Generative AI Training", pre-publication May 9, 2025) says training may implicate exclusive rights. Fair use is a fact-specific four-factor analysis, some training uses are likely fair, and market harm is a concern. It favors voluntary licensing over new legislation. — [Jones Day](https://www.jonesday.com/en/insights/2025/05/us-copyright-office-issues-guidance-on-generative-ai-training); [Sidley](https://www.sidley.com/en/insights/newsupdates/2025/05/generative-ai-meets-copyright-scrutiny)
- **EU AI Act Article 50**:
  - Generative outputs must be machine-identifiable as AI-generated, and deepfakes must be disclosed.
  - Enforceable from **Aug 2, 2026**.
  - The final Code of Practice on AI-generated content transparency was published **June 10, 2026**, and the Commission later confirmed it as adequate.
  - Deployers are expected to label at first exposure using a common icon.
  - [artificialintelligenceact.eu Art. 50](https://artificialintelligenceact.eu/article/50/); [Paul Weiss](https://www.paulweiss.com/insights/client-memos/eu-finalises-transparency-rules-for-ai-generated-content); [Faegre Drinker](https://www.faegredrinker.com/en/insights/publications/2026/7/eu-ai-act-commission-confirms-transparency-code-of-practice-as-adequate-and-publishes-final-version-of-its-guidelines-on-transparency-obligations); [Jones Day](https://www.jonesday.com/en/insights/2026/01/european-commission-publishes-draft-code-of-practice-on-ai-labelling-and-transparency)

**Model and tool licenses**
- **Wan 2.2**: Apache 2.0; "We claim no rights over the your generated contents". — [GitHub](https://github.com/Wan-Video/Wan2.2)
- **FLUX.1 [dev]**: non-commercial license for the *model*, but outputs "may be used for any purpose (including commercial)". Outputs may not be used to train a competing model. Community threads flag conflicting terms, and studios running it in production would need BFL's commercial license. — [HF discussion #136](https://huggingface.co/black-forest-labs/FLUX.1-dev/discussions/136); [Civitai explainer](https://civitai.com/articles/6625/can-i-use-flux-for-commercial-use)
- **HunyuanVideo**: license Territory is "worldwide… excluding the European Union, United Kingdom and South Korea". — [HF LICENSE](https://huggingface.co/tencent/HunyuanVideo/blob/main/LICENSE)
- **Illustrious XL**: Fair AI Public License 1.0-SD; outputs can be used commercially, but restrictions apply to model-as-a-service. **NoobAI-XL**: modified license that "prohibit[s] any form of commercialization… of the model, derivative models, or model-generated products". Whether this is compatible with the share-alike base license is disputed. — [localaimaster](https://localaimaster.com/blog/best-local-anime-image-model); [X post quoting license](https://x.com/satos73/status/1899426295492309103); [Civitai "What the license"](https://civitai.com/articles/18619/what-the-license)
- **AniSora**: Apache 2.0 code "with additional model usage restrictions". — [GIGAZINE](https://gigazine.net/gsc_news/en/20250519-anisora/)
- **MangaNinja**: CC BY-NC 4.0. **Sakuga-42M**: academic/research use. **AniDoc, ToonCrafter**: Apache-2.0. **Magi**: MIT. — [MangaNinja](https://github.com/ali-vilab/MangaNinjia); [Sakuga](https://github.com/KytraScript/SakugaDataset); [AniDoc](https://github.com/yihao-meng/AniDoc); [ToonCrafter](https://github.com/Doubiiu/ToonCrafter); [Magi](https://github.com/ragavsachdeva/magi)

### Inferences
- **Training per-outfit LoRAs on the licensed manga is likely "expressive" use** (the goal is to reproduce the character's look), so Article 30-4 should not be relied on. Get explicit training and derivative rights in the adaptation contract with the publisher (e.g. Shueisha or Kodansha). Given CODA's stance, publishers are unlikely to accept "we relied on 30-4".
- Pick base models whose licenses allow commercial production use, and record `base_model_license` per adapter in the outfit bible:
  - Wan 2.2, AniDoc, ToonCrafter, SDXL base, Illustrious with care.
  - Avoid NoobAI-XL, MangaNinja and FLUX.1-dev (without a BFL license) in shipped work.
  - Avoid HunyuanVideo if any EU/UK/Korea distribution or staff are involved.
- For EU distribution after Aug 2026, plan C2PA-style metadata or watermarking and on-screen or credits disclosure for AI-generated content. Article 50 also has narrower provisions for evidently artistic works, which were not researched here.
- **Labor:** frame AI as assisting existing roles (iro-shitei, douga-checking), keep credits for human designers, and publish an AI-use policy per title. The WIT and Toei backlashes show that disclosure and governance are as important as capability.

### Gaps
- I could not access the ACA primary PDF to quote its exact language on "additional learning" (LoRA) and style versus expression, or to confirm the July 2024 "Checklist & Guidance" document. A secondary source claims a 2026 ACA guidance update that tightens output scrutiny; I could not verify it (the source page was blocked).
- I did not find a 2024–2026 JAniCA statement specific to generative AI. JAniCA's known surveys concern wages and hours.
- SDXL's CreativeML OpenRAIL++-M terms were not re-fetched. It is generally understood to allow commercial use with use-based restrictions, but this is unverified here.
- I did not locate a judgment on output infringement in Japanese courts involving anime/manga AI outputs.

---

## 7. Practical MVP plan for a small team (1–3 months), clothing-focused

### Takeaway
A 3–5 person team can realistically build a **costume-continuity and color-consistency tool**, not a full manga-to-anime generator. It would ingest licensed manga, auto-detect characters and outfit changes, build a reviewable outfit bible with palettes, train per-outfit adapters, and enforce palettes on AI- or human-produced frames with automated QA. All of it would run on commercially licensed components.

### Cited Findings
- Components with permissive licenses and modest VRAM:
  - Magi (MIT) for panels and character clusters.
  - AniDoc (Apache-2.0, about 14 GB) for line-sequence coloring.
  - ToonCrafter (Apache-2.0) for interpolation.
  - Wan 2.2 TI2V-5B (Apache-2.0, 24 GB GPU, under 9 min per 5-second 720p clip) or A14B on H100.
  - [Magi](https://github.com/ragavsachdeva/magi); [AniDoc](https://github.com/yihao-meng/AniDoc); [ToonCrafter](https://github.com/Doubiiu/ToonCrafter); [Wan2.2](https://github.com/Wan-Video/Wan2.2)
- Garment masks and tags: SAM2-based pipelines (See-through/SegAnimeChara methods) plus WD14 Danbooru tags. — [See-through](https://github.com/shitagaki-lab/see-through); [SegAnimeChara](https://dl.acm.org/doi/fullHtml/10.1145/3588028.3603685); [WD14 guide](https://www.mohsindev369.dev/blog/how-to-tag-images-for-lora-training)
- Paint-bucket colorization (BasicPBC, CVPR 2024; DACoN, ICCV 2025) gives region-level color assignment. — [Awesome-AI4Animation](https://github.com/yunlong10/Awesome-AI4Animation); [DACoN](https://openaccess.thecvf.com/content/ICCV2025/papers/Nagata_DACoN_DINO_for_Anime_Paint_Bucket_Colorization_with_Any_Number_ICCV_2025_paper.pdf)
- Kitsu (open source, REST/Python API, Harmony and Blender integrations) can serve as tracker and review hub. — [Kitsu](https://www.cg-wire.com/kitsu/)
- Cost references: SDXL/FLUX LoRA about $8 per 1,000 steps hosted; RunPod L40S $0.79–1.09/hr and H100 $1.99–3.49/hr. — [bestaiweb](https://www.bestaiweb.ai/how-to-train-a-custom-lora-for-flux-and-sdxl-with-kohya-ss-ai-toolkit-and-fal-ai-in-2026/); [RunPod](https://www.runpod.io/articles/comparison/choosing-gpus)

### Inferences
**Recommended MVP stack**
- Python/FastAPI backend.
- Postgres with JSONB for the outfit bible (schema in section 3).
- S3/MinIO for assets.
- ComfyUI workers (local RTX 4090/5090 plus RunPod burst).
- Kitsu for tasks and review.
- A small React review UI showing page, panel, crop and outfit, with approve/split/merge controls.

**Timeline**
- **Weeks 1–3: ingest and detection.**
  - Licensed volume ingest.
  - Magi for panels, character clusters and reading order.
  - SAM2 garment masks plus WD14 tags per appearance.
  - Outfit-change candidate detection from tag-set diffs and DINO embedding jumps.
  - Reviewer UI.
  - Deliverable: per-chapter appearance-to-outfit mapping with about 1 reviewer-day per volume (my estimate).
- **Weeks 4–6: outfit bible and color.**
  - Outfit records, model-sheet upload, and a palette editor (normal/shadow/highlight hex plus lighting variants).
  - Export a "costume continuity report" per episode script (which outfits, which cuts).
  - Train per-outfit SDXL (Illustrious-based, license checked) or Wan LoRAs on approved sheets only. Record license and provenance.
- **Weeks 7–10: generation and enforcement.**
  - Keyframe drafting with ControlNet lineart/pose plus outfit adapter.
  - Region-level palette snapping (paint-bucket style) to exact bible hexes.
  - AniDoc for short line sequences, ToonCrafter for in-between tests, Wan 2.2 I2V for a few full-motion test shots.
- **Weeks 11–12: QA and pilot.**
  - Automated QA (tag diff, ΔE palette check, DINO/DreamSim similarity, temporal flicker) posting flagged frames to Kitsu playlists.
  - Pilot on one episode's worth of cuts, with human color-checker sign-off and an AI-use provenance log per cut.

**Success metrics:** percentage of outfit changes auto-detected (recall versus human ground truth), palette ΔE pass rate, reviewer minutes per cut, and costume-error escape rate to final.

**Deliberately out of scope for an MVP:** fully automatic shot generation from manga panels, lip-sync and backgrounds.

**Legal checklist before day 1:** a signed adaptation license with an AI-training clause, a model license register, EU Art. 50 labeling plan if EU distribution, and a staff and creator AI-use policy.

### Gaps
- No public case study documents a small team shipping such a tool. Timelines and reviewer-effort estimates are my judgment, not sourced.
- Accuracy of Magi's character clustering on outfit changes (does a costume change break identity clusters?) was not measured in any source found.
