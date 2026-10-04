# Computer Vision / ML for Extracting and Understanding Clothing from Manga Pages (Input Stage of a Manga-to-Anime Pipeline)

Research note scope: datasets, models and techniques to detect characters, segment clothing, parse garment parts, find the same outfit across panels, and colorize clothing from black-and-white manga. Coverage is 2020-2026, current to October 2026.
Method note: huggingface.co, arxiv.org, openaccess.thecvf.com, alphaxiv, dl.acm.org and manga109.github.io were blocked by the network egress proxy while this research ran. Facts about those pages come from search-result snippets and mirrors (GitHub READMEs, CVF/ResearchGate listings, third-party paper notes), and each is cited to the URL the snippet named. Wherever only a snippet supported a fact, that is marked.

---

## 1. Manga datasets and annotations (Manga109, Manga109-s, MangaSeg, anime segmentation and parsing data, Danbooru): what exists and under what license?

### Takeaway
No public dataset gives garment-level labels (shirt, skirt, sleeve, collar) for black-and-white manga. Manga109 and its 2025 MangaSeg extension go no finer than character body, face, frame, balloon and text. Datasets with clothing labels exist only for color anime illustrations (Danbooru tags, RetriBooru clothing masks, the StyleAnime parsing dataset, Live2D-derived layers such as See-through). Licensing is the main constraint: Manga109 is academic-only, Manga109-s permits commercial use with restrictions, Danbooru-derived sets carry MIT metadata licenses but contain copyrighted images, and MangaZero/PopManga ship only URLs or test images.

### Cited Findings
**Manga109 / Manga109-s**
- Manga109's annotations cover frames, speech text, character faces and character bodies, more than 500k annotations in total — [Manga109 arXiv 2005.04425](https://arxiv.org/pdf/2005.04425) (search snippet)
- The authors authorized Manga109 for academic, non-commercial research use only. Of the 109 volumes, 87 were newly approved for commercial use and are distributed as "Manga109-s" — [Manga109 project site](https://manga109.github.io/manga109-project-website/en/index.html) (search snippet; site blocked for direct fetch)
- Conditions on commercial use of Manga109-s: no redistribution to third parties; published ML or image-processing results must clearly state that Manga109-s was used; manga images may not be sold together with results; direct copies or modifications of the images must not be treated as products — [Manga109 project site](https://manga109.github.io/manga109-project-website/en/index.html) (search snippet)
- Both sets are mirrored on Hugging Face as `hal-utokyo/Manga109` and `hal-utokyo/Manga109-s` — [HF Manga109-s](https://huggingface.co/datasets/hal-utokyo/Manga109-s)
- **Manga109-v2026** (accepted at the ICML 2026 Culture × AI workshop) re-examined the *dialogue text* annotations. About 29,000 annotations were revised (19.6% of all text annotations), fixing inaccurate transcriptions, missing text regions, overlapping dialogue and onomatopoeia, and under-segmented balloons. OCR evaluation scores rose by 14.4. No clothing annotations were added — [Manga109-v2026 arXiv 2605.21182](https://arxiv.org/html/2605.21182v2); [Paper note](https://en.papernotes.org/ICML2026/multimodal_vlm/manga109-v2026_revisiting_manga109_annotations_for_modern_manga_understanding/)

**MangaSeg (CVPR 2025)**
- "Advancing Manga Analysis: Comprehensive Segmentation Annotations for the Manga109 Dataset" by Minshan Xie, Jian Lin, Hanyuan Liu, Chengze Li and Tien-Tsin Wong (CVPR 2025). It adds segmentation annotations to all 109 volumes, with category, location and instance information for semantic and instance segmentation. Classes: panels, characters, faces, speech balloons, text, plus links between characters and balloons — [CVPR 2025 open access](https://openaccess.thecvf.com/content/CVPR2025/html/Xie_Advancing_Manga_Analysis_Comprehensive_Segmentation_Annotations_for_the_Manga109_Dataset_CVPR_2025_paper.html); [CVPR poster](https://cvpr.thecvf.com/virtual/2025/poster/33704)
- More than 700,000 segmentation annotations, grouped into frames/panels, balloons, text, character faces and character bodies — [CVPR 2025 paper PDF](https://openaccess.thecvf.com/content/CVPR2025/papers/Xie_Advancing_Manga_Analysis_Comprehensive_Segmentation_Annotations_for_the_Manga109_Dataset_CVPR_2025_paper.pdf) (search snippet)
- Released at `MS92/MangaSegmentation` on Hugging Face — [HF MangaSegmentation](https://huggingface.co/datasets/MS92/MangaSegmentation)

**Datasets released with Magi**
- The authors released the **PopManga** test set (freely available images from Shueisha's Manga Plus) and a **PopCharacters** dataset. Models and datasets are for academic research only. Manga images cannot be redistributed because of copyright — [magi GitHub](https://github.com/ragavsachdeva/magi)

**MangaZero (with DiffSensei, CVPR 2025)**
- 43,264 manga pages and 427,147 annotated panels, described as the first large-scale set for multi-character, multi-state manga generation. License issues prevent sharing the images, so the release contains MangaDex image URLs plus annotations — [DiffSensei GitHub](https://github.com/jianzongwu/DiffSensei); [DiffSensei arXiv](https://arxiv.org/pdf/2412.07589)

**Anime segmentation and clothing datasets (color illustrations)**
- **skytnt/anime-segmentation** dataset (about 18 GB) combines AniSeg and character_bg_seg_data, cleaned first with DeepDanbooru and then by hand so that every mask is an anime character. It provides whole-character foreground masks only, not clothing — [HF dataset](https://huggingface.co/datasets/skytnt/anime-segmentation); [SkyTNT GitHub](https://github.com/SkyTNT/anime-segmentation)
- **RetriBooru** (arXiv 2312.02521) has 599,192 Danbooru images, each with a single character and artist and at least one meaningful cloth tag. IS-Net masks split each character into whole figure, clothes and face — [RetriBooru arXiv](https://arxiv.org/pdf/2312.02521)
- **"Parsing-Conditioned Anime Translation: A New Dataset and Method"** (ACM TOMM, 2023; the StyleAnime framework) bridges the anime/photo domain gap with a parsing map, which makes it a source of anime human-parsing labels — [ACM DL](https://dl.acm.org/doi/full/10.1145/3585002) (search snippet; class list not retrievable)
- A 2025 paper says existing segmenters show "a clear domain gap on anime imagery, and no public dataset provides the full-body, fine-grained parsing granularity required" — [arXiv 2508.06032 / See-through-related snippet](https://arxiv.org/html/2508.06032)
- **Danbooru2023**: more than 5 million anime images with community tags, about 30 per image, covering characters, scenes, copyrights and artists. Clothing tags such as `school_uniform` are part of the scheme. The dataset metadata is MIT-licensed, but the underlying images are third-party copyrighted art — [HF nyanko7/danbooru2023](https://huggingface.co/datasets/nyanko7/danbooru2023); [Moescape tag explainer](https://moescape.ai/posts/31db478d-2883-44e9-94f7-ea3460e5b677)
- **See-through** training data was Live2D model files (via a CubismPartExtr data-prep repository) from the USTC Student ACG Club — [see-through GitHub](https://github.com/shitagaki-lab/see-through)

**Real-photo parsing datasets (used for transfer)**
- **ATR** has 17,706 photos labeled with 18 classes (background, 6 body parts, 11 clothing/accessory classes) and is research/non-commercial — [segformer_b2_clothes GitHub](https://github.com/mattmdjaga/segformer_b2_clothes); [HF card](https://huggingface.co/mattmdjaga/segformer_b2_clothes)

### Inferences
- With commercial-use rights, Manga109-s plus MangaSeg body masks is the best base for fine-tuning a character-body segmenter. Garment-part labels would have to be added in-house; pseudo-labels from SAM 3 or anime parsers, then human correction, is a practical route.
- To get garment labels in the manga domain, one option is to synthesize training pairs by converting color anime parsing data (RetriBooru, See-through/Live2D layers) to manga style: line extraction plus screentone synthesis, for example with ScreenVAE (section 4). The label maps carry over unchanged.
- Anyone building a commercial product should assume that Manga109 (non-s), PopManga, MangaZero, MangaNinja and ColorFlow weights are off-limits without separate licensing.

### Gaps
- MangaSeg's license terms and its per-category benchmark mIoU/AP numbers could not be retrieved; the CVF PDF and HF were blocked.
- No dataset could be found with pixel-level garment classes on screentoned B/W manga. I believe none is public, but that is not certain.
- The StyleAnime dataset's exact class list, image count and license could not be verified.
- Other anime-parsing datasets known from memory, such as "AnimeRun", "Anime Face/Body Parsing" Kaggle sets and the "AniSeg" origin, were not verified this session.

---

## 2. Character detection, panel and page parsing, and character re-identification in manga (Magi v1/v2/v3 and related work)

### Takeaway
The Magi family (Sachdeva and Zisserman, Oxford VGG) is the de facto open baseline for manga page parsing, as of 2026. Magi v1 (CVPR 2024) detects panels, characters and text; Magiv2 (ACCV 2024) adds chapter-wide character naming from a character bank; Magiv3 (2025) unifies detection, association, OCR and caption grounding. All are academic-only. Manga character re-ID research notes that clothing is often the *signature* that tells characters apart, which helps with outfit tracking but breaks when a character changes clothes.

### Cited Findings
- **Magi v1, "The Manga Whisperer: Automatically Generating Transcriptions for Comics"** (CVPR 2024): detects characters, text blocks and panels, orders panels, clusters characters, associates text with speakers, runs OCR and produces transcripts — [magi GitHub](https://github.com/ragavsachdeva/magi); [ResearchGate](https://www.researchgate.net/publication/384222282_The_Manga_Whisperer_Automatically_Generating_Transcriptions_for_Comics)
- **Magiv2, "Tails Tell Tales: Chapter-Wide Manga Transcriptions with Character Names"** (ACCV 2024; arXiv 2408.00298): produces chapter-wide transcripts with named characters and much higher speaker-diarisation precision than prior work. Speakers are matched across pages using a character bank of reference images with names — [arXiv 2408.00298](https://arxiv.org/abs/2408.00298); [magi GitHub](https://github.com/ragavsachdeva/magi); [HF magiv2](https://huggingface.co/ragavsachdeva/magiv2)
- **Magiv3** (2025; "From Panels to Prose: Generating Literary Narratives from Comics", arXiv 2503.23344) is a unified model that detects panels, characters, text and speech-bubble tails, associates text with speakers and characters with each other, runs OCR, and grounds characters named in captions to image regions. It also generates captions and prose — [arXiv 2503.23344](https://arxiv.org/html/2503.23344)
- License: models and datasets are "available for academic research purposes only" — [magi GitHub](https://github.com/ragavsachdeva/magi)
- **Unsupervised Manga Character Re-identification via Face-body and Spatial-temporal Associated Clustering** (arXiv 2204.04621). Because of drawing-style limits, different characters can have similar faces, "but different clothes often constitute the signatures of characters". Characters have unique signatures (accessories, hairstyles, costumes) that "usually do not change within the same manga book" — [arXiv 2204.04621](https://arxiv.org/pdf/2204.04621); [Semantic Scholar](https://www.semanticscholar.org/paper/Unsupervised-Manga-Character-Re-identification-via-Zhang-Wang/37a4f69d140af857aae1e2c6b58d9f0b4f7c3705)
- **Identity-Aware Semi-Supervised Learning for Comic Character Re-Identification** (arXiv 2308.09096) combines metric learning with identity-aware self-supervision through contrastive learning on face/body pairs in one network — [ResearchGate](https://www.researchgate.net/publication/373246266_Identity-Aware_Semi-Supervised_Learning_for_Comic_Character_Re-Identification); [awesomepapers](https://awesomepapers.io/computer-vision/papers/2308.09096)
- Survey context: "One missing piece in Vision and Language: A Survey on Comics Understanding" (arXiv 2409.09502) proposes a "LoCU" layered framework, and the Magi series is described as pushing into the higher layers (detection and re-ID) — [arXiv 2409.09502](https://arxiv.org/pdf/2409.09502)

### Inferences
- Magi's character boxes plus cluster IDs are a natural first stage: crop each character, then run clothing segmentation and tagging per crop. Magiv2's character bank could be extended into a "character × outfit" bank, with one reference image per costume.
- Re-ID models that lean on clothing signatures will merge or split identities when an outfit changes, for example school uniform versus casual clothes. Separating identity (face/hair embeddings) from outfit (clothing-region embeddings) is a design need that no surveyed paper addresses directly.

### Gaps
- Quantitative Magi v1/v2/v3 detection AP and clustering metrics could not be retrieved because HF and arXiv were blocked.
- No published benchmark could be found for *outfit* (costume-change) re-identification in manga or comics. Cloth-changing person re-ID exists for real photos (e.g., arXiv 2103.15537, 2308.10692), but no manga equivalent turned up.

---

## 3. Human and clothing parsing models (SCHP, LIP/ATR/CIHP, SegFormer-clothes, Grounded-SAM, SAM 2/SAM 3, See-through): transfer to manga and anime

### Takeaway
Photo-trained parsers (SCHP, SegFormer-b2-clothes) work but degrade on anime and manga because of a documented domain gap. SAM 3 (Nov 2025) brings text-prompted concept segmentation ("skirt", "jacket"), but no published evaluation on line art was found. The strongest anime-specific option as of 2026 is **See-through** (SIGGRAPH 2026, Apache-2.0), which splits a color anime illustration into up to 23 inpainted semantic layers including clothing. It has not been shown on screentoned manga.

### Cited Findings
- **SCHP** (Self-Correction for Human Parsing) is reported at 59.36 (LIP) and 82.29 (ATR) in one comparison table and is often paired with OpenPose — [arXiv 2508.06032](https://arxiv.org/html/2508.06032) (search snippet)
- **segformer_b2_clothes** (mattmdjaga) is SegFormer-B2 fine-tuned on ATR for clothes segmentation; it can also do human segmentation. The code is MIT, while the ATR data and derived checkpoints keep research/non-commercial terms. Reported mIoU: 0.778 over 18 classes, 0.766 without background; the HF card's detailed evaluation gives an overall mean IoU of 0.69 — [GitHub](https://github.com/mattmdjaga/segformer_b2_clothes); [HF](https://huggingface.co/mattmdjaga/segformer_b2_clothes) (snippet figures conflict slightly: 0.778 vs 0.69, probably different eval splits or definitions)
- It ships as a ComfyUI node (`Comfyui_segformer_b2_clothes`), which is common in hobbyist anime workflows — [StartHua GitHub](https://github.com/StartHua/Comfyui_segformer_b2_clothes)
- Domain gap: "existing segmenters exhibit a clear domain gap on anime imagery". Generalist SAM-series models "typically rely on interactive prompting and still have a domain gap towards anime-style imagery" — [arXiv 2508.06032 snippet](https://arxiv.org/html/2508.06032); [See-through ResearchGate](https://www.researchgate.net/publication/400415022_See-through_Single-image_Layer_Decomposition_for_Anime_Characters)
- **SAM 3, "Segment Anything with Concepts"** (Meta, arXiv 2511.16719, Nov 2025): a unified model for detection, segmentation and tracking in images and video, prompted by text (open-vocabulary short noun phrases), image exemplars or visual prompts. It introduces Promptable Concept Segmentation and was trained on 4M concept labels including hard negatives. The SA-Co benchmark has 270K unique concepts (more than 50x prior benchmarks). It more than doubles strong baselines on SA-Co/Gold and reaches 75-80% of human cgF1 — [Roboflow](https://blog.roboflow.com/what-is-sam3/); [MarkTechPost](https://www.marktechpost.com/2025/11/20/meta-ai-releases-segment-anything-model-3-sam-3-for-promptable-concept-segmentation-in-images-and-videos/); [Meta blog SAM 3.1](https://ai.meta.com/blog/segment-anything-model-3/?_fb_noscript=1)
- **See-through: Single-image Layer Decomposition for Anime Characters** (arXiv 2602.03749, ACM SIGGRAPH 2026 Conference Papers). It decomposes one anime illustration into up to 23 fully inpainted RGBA semantic layers (hair, face, eyes, clothing, accessories, ...) with inferred drawing order and exports a layered PSD. Components: LayerDiff 3D (SDXL-based transparent layer generation), fine-tuned Marigold pseudo-depth, and SAM-based body parsing. License Apache-2.0. Trained on 8×H200 with Live2D models. Low-VRAM variants run on 8-12 GB. Grayscale/manga input is not addressed — [GitHub shitagaki-lab/see-through](https://github.com/shitagaki-lab/see-through); [arXiv 2602.03749](https://arxiv.org/abs/2602.03749); [Moonlight review](https://www.themoonlight.io/en/review/see-through-single-image-layer-decomposition-for-anime-characters)
- The See-through authors argue that 2D semantic segmentation "lacks the 'see-through' capability to infer occluded anatomy crucial for animation" — [See-through arXiv HTML](https://arxiv.org/html/2602.03749v1) (snippet)
- **SkyTNT anime-segmentation** (Apache-2.0): ISNet (plus isnet_is), U2Net (full/lite), MODNet and InSPyReNet (Res2Net50/SwinB) for anime character foreground masks. Weights are on HF. No metrics are listed on the README — [GitHub](https://github.com/SkyTNT/anime-segmentation)

### Inferences
- A realistic 2026 recipe: (a) a character mask from MangaSeg-finetuned or SkyTNT ISNet; (b) garment-part masks from SAM 3 text prompts ("skirt", "necktie", "sailor collar", "jacket"), restricted to the character mask; (c) optionally a SegFormer or SCHP head fine-tuned on manga-styled anime parsing data for a fixed label set. The SAM 3 step on screentoned manga is untested; expect confusion between screentone texture and garment boundaries.
- See-through's layer taxonomy and Apache-2.0 license make it attractive as the "garment layer" target format for animation. On manga, the likely approach is to colorize first (section 6) and then decompose, or to retrain on grayscale/screentoned renders of its Live2D training data.

### Gaps
- No quantitative results for SCHP, SegFormer-clothes, Grounded-SAM, SAM 2 or SAM 3 on manga or anime line art were found.
- See-through's full 23-layer list, and whether clothing is split into top, bottom and so on, was not given in the README.
- Licenses for SCHP (from memory: MIT code, but LIP/ATR/CIHP-trained weights carry dataset terms) and SAM 3 (Meta's SAM license) were not verified this session.

---

## 4. Screentone handling: removal, screentone-to-region mapping, line extraction, manga restoration

### Takeaway
The CUHK group around Tien-Tsin Wong (Minshan Xie, Chengze Li, Menghan Xia) owns most of the screentone literature. **ScreenVAE** gives an interpretable screentone embedding that supports grouping regions by tone, which is what "same tone ⇒ same garment region" needs. **Manga Restoration** (CVPR 2021) recovers bitonal screentones from degraded scans. **MangaLineExtraction** (SIGGRAPH Asia 2017) extracts structural lines and is the standard way to strip tones before line-art models.

### Cited Findings
- **Separation of Manga Line Drawings and Screentones** (Ito, Matsui et al., ~2015): a classical method that combines a Laplacian-of-Gaussian filter (removes screentones) with a flow-based DoG filter (keeps lines) into a binary line/screentone mask — [ResearchGate PDF](https://www.researchgate.net/profile/Kiyoharu-Aizawa/publication/277653033_Separation_of_Manga_Line_Drawings_and_Screentones/links/556f152d08aec226830a4f7a/Separation-of-Manga-Line-Drawings-and-Screentones.pdf)
- **Exploiting Aliasing for Manga Restoration** (Xie, Xia et al., CVPR 2021, arXiv 2105.06830): restores quality bitonal manga from degraded internet scans. It uses aliasing from downsampled screentones as a cue, with two stages: a Scale Estimation Network (SE-Net) and a Manga Restoration Network (MR-Net) — [arXiv](https://arxiv.org/abs/2105.06830); [CVF PDF](https://openaccess.thecvf.com/content/CVPR2021/papers/Xie_Exploiting_Aliasing_for_Manga_Restoration_CVPR_2021_paper.pdf)
- **ScreenVAE** encodes screentone patterns as a smooth, interpolatable 4D vector map that can be resampled to reconstruct screentones, translating in both directions between screened and color-filled manga — [Manga Rescreening arXiv 2306.04114](https://arxiv.org/pdf/2306.04114); [CVPR 2021 Manga Restoration PDF](https://ttwong12.github.io/papers/mangarestore/mangarestore.pdf)
- **Manga Rescreening with Interpretable Screentone Representation** (arXiv 2306.04114) segments manga automatically using GMM analysis on the screentone representation, choosing the number of clusters by silhouette coefficient. Artists "commonly use the same type of screentone with intensity variation to fill the same semantic region", and the disentangled intensity axis keeps segmentation consistent across intensity changes — [arXiv 2306.04114](https://ar5iv.labs.arxiv.org/html/2306.04114)
- **Screentone-Preserved Manga Retargeting** (Xie et al., Computer Graphics Forum 2025, arXiv 2203.03396) — [Wiley CGF](https://onlinelibrary.wiley.com/doi/full/10.1111/cgf.70096); [arXiv](https://arxiv.org/pdf/2203.03396)
- **Screentone-Aware Manga Super-Resolution** (arXiv 2305.08325) — [arXiv](https://arxiv.org/pdf/2305.08325)
- **MangaLineExtraction** ("Deep Extraction of Manga Structural Lines", Li, Liu, Wong; SIGGRAPH Asia 2017). The official PyTorch port's weights were converted from Theano with error <1e-3. It works on manga and also on color cartoons with clear hand-drawn lines. An HF-transformers version exists as `p1atdev/MangaLineExtraction-hf` — [GitHub MangaLineExtraction_PyTorch](https://github.com/ljsabc/MangaLineExtraction_PyTorch); [Project page](https://www.cse.cuhk.edu.hk/~ttwong/papers/linelearn/linelearn.html); [HF](https://huggingface.co/p1atdev/MangaLineExtraction-hf)
- **LineDistiller** (hepesu, Keras): data-driven line extractor for anime, manga and illustration — [GitHub](https://github.com/hepesu/LineDistiller)
- **structure-preserving-manga-editing** (Mantra Inc.) — [GitHub](https://github.com/mantra-inc/structure-preserving-manga-editing)

### Inferences
- For clothing extraction, screentone carries signal as well as noise. Tone type and intensity regions inside a character mask often match garment regions (dark jacket vs. light shirt) and suggest relative darkness for colorization. A pipeline can (1) extract clean lines with MangaLineExtraction for the segmentation and colorization models and (2) compute a ScreenVAE-style tone map as an extra channel or prior for region grouping.
- Manga Restoration is the right preprocessing for low-resolution web scans before any of this.

### Gaps
- Licenses for MangaLineExtraction, ScreenVAE and Manga Restoration code could not be confirmed.
- No paper explicitly mapping screentone regions to *garment classes* was found.

---

## 5. Tagging clothing attributes (WD14/wd-eva02 SmilingWolf taggers, DeepDanbooru, Camie, PixAI, CLIP)

### Takeaway
Danbooru-trained multi-label taggers are the practical way to get clothing attribute tags (`serafuku`, `school_uniform`, `pleated_skirt`, `necktie`, ...) from anime and manga crops. As of 2026 the strongest permissively licensed options are SmilingWolf's **wd-eva02-large-tagger-v3** (Apache-2.0) and **camie-tagger(-v2)** (about 70.5k tags), with newer community taggers such as PixAI tagger v1.0 and OppaiOracle. DeepDanbooru (MIT) is the legacy baseline.

### Cited Findings
- **SmilingWolf/wd-eva02-large-tagger-v3** is licensed Apache-2.0 and outputs rating, character and general tags — [HF model card](https://huggingface.co/SmilingWolf/wd-eva02-large-tagger-v3); [Endor Labs listing](https://www.endorlabs.com/ai-model/smilingwolf-wd-eva02-large-tagger-v3)
- **DeepDanbooru** (KichangKim) is a TensorFlow multi-label tagger trained on Danbooru-derived data, with tags for characters, styles, clothing and emotions. MIT License — [GitHub](https://github.com/KichangKim/DeepDanbooru); [SourceForge mirror](https://sourceforge.net/projects/deepdanbooru.mirror/)
- **camie-tagger** covers 70,527 tags in seven categories for anime/manga illustrations, with 67.3% micro-F1 and 50.6% macro-F1 — [PromptLayer](https://www.promptlayer.com/models/camie-tagger/); [AIBase](https://model.aibase.com/models/details/1915694798838849537). A v2 exists — [aimodels.fyi camie-tagger-v2](https://www.aimodels.fyi/models/huggingFace/camie-tagger-v2-camais03)
- Other taggers: **pixai-labs/pixai-tagger-v1.0** — [HF](https://huggingface.co/pixai-labs/pixai-tagger-v1.0); **Grio43/OppaiOracle** — [HF](https://huggingface.co/Grio43/OppaiOracle). TAG.OPS is a local workbench that runs WD14 and Camie-v2 together — [GitHub TAG.OPS](https://github.com/gaowanliang/TAG.OPS)
- Danbooru tag datasets for building or adjusting clothing vocabularies: `isek-ai/danbooru-tags-2023`, `KBlueLeaf/danbooru2023-metadata-database` — [HF](https://huggingface.co/datasets/isek-ai/danbooru-tags-2023); [HF](https://huggingface.co/datasets/KBlueLeaf/danbooru2023-metadata-database)
- SkyTNT used DeepDanbooru to clean the anime-segmentation dataset, an example of taggers serving as data filters — [SkyTNT GitHub](https://github.com/SkyTNT/anime-segmentation)

### Inferences
- Danbooru includes many `monochrome`/`greyscale`/`comic` images, so taggers should handle manga crops fairly well for shape-based clothing tags. Color tags such as `blue_skirt` will be unreliable or hallucinated on B/W input and should be filtered out, or taken from colorized output instead.
- Tags work best when run per character crop (from Magi boxes) and, for garment-level attributes, per garment mask crop. Tag outputs can also produce SAM 3 text prompts automatically (e.g., the tag `serafuku` → prompts "sailor collar", "neckerchief", "pleated skirt").
- CLIP/SigLIP zero-shot classification is an option for custom attributes absent from Danbooru's vocabulary, though no manga-specific CLIP clothing evaluation was found.

### Gaps
- wd-eva02-large-tagger-v3 metrics (F1 at threshold, Danbooru ID range, tag count) could not be retrieved because the HF card was blocked. From memory, the v3 series was trained on Danbooru data up to roughly ID 7.2M with about 10.8k tags in the vocabulary, and includes ViT, SwinV2, ConvNeXt and EVA02 variants. This is unverified.
- No benchmark of tagger accuracy specifically on screentoned manga was found.

---

## 6. Character and outfit consistency recognition across chapters (outfit clustering, costume re-ID)

### Takeaway
No dedicated "outfit re-ID in manga" method or benchmark exists. The closest pieces are Magiv2's character bank (identity across a chapter), clothing-aware manga re-ID clustering (2022), and retrieval-based colorizers (ColorFlow, Cobra) that implicitly match reference crops to target characters and outfits.

### Cited Findings
- Magiv2 matches characters across pages via a character bank of named reference images — [magi GitHub](https://github.com/ragavsachdeva/magi); [arXiv 2408.00298](https://arxiv.org/abs/2408.00298)
- Manga re-ID clustering uses face and body features with spatial-temporal association. Clothing is a strong identity signature within a book — [arXiv 2204.04621](https://arxiv.org/pdf/2204.04621)
- Semi-supervised face/body contrastive re-ID for comics — [ResearchGate 2308.09096](https://www.researchgate.net/publication/373246266_Identity-Aware_Semi-Supervised_Learning_for_Comic_Character_Re-Identification)
- **ColorFlow** uses a Retrieval-Augmented Pipeline (RAP) to retrieve relevant color references for each B/W panel before colorizing, to avoid "identity drift" in hair and attire colors — [ColorFlow project](https://zhuang2002.github.io/ColorFlow/); [GitHub](https://github.com/TencentARC/ColorFlow)
- DiffSensei's MangaZero annotates multi-state characters across panels, and its evaluation includes character consistency — [DiffSensei GitHub](https://github.com/jianzongwu/DiffSensei)

### Inferences
- A workable design: per character instance, compute (a) an identity embedding from face and hair crops (Magi-style cluster or a re-ID model) and (b) an outfit embedding from the garment-masked region (e.g., DINOv2/SigLIP features on the clothing mask, plus the tagger's clothing tag vector). Cluster outfit embeddings within each identity to find costumes, then keep one color reference per (character, costume) pair to feed a multi-reference colorizer.
- The outfit-clustering step can borrow from cloth-changing person re-ID (arXiv 2103.15537, 2308.10692), with the logic reversed: there identity must ignore clothing, while here we want a clothing-only embedding conditioned on identity.

### Gaps
- No quantitative outfit-clustering results on manga exist in the literature I could find. This is an open research and engineering problem.

---

## 7. Manga colorization with reference images: which methods keep clothing colors consistent per character?

### Takeaway
State of the art moved from GAN/SD-1.5 systems to reference-driven diffusion transformers between 2024 and 2026. For *consistent clothing color across many panels*, the most relevant are **ColorFlow** (Dec 2024, retrieval-augmented, Tencent ARC, Academic license), **Cobra** (SIGGRAPH 2025, more than 200 references, Apache-2.0 with released weights), **MangaNinja** (CVPR 2025 Highlight, point control, CC BY-NC 4.0), **MangaDiT** (Aug 2025, DiT, reports large gains over MangaNinja), and **ColorizeDiffusion / ColorizeDiffusionXL** (WACV 2025 / CVPR 2026 Highlight). Cobra and ColorFlow explicitly target ID/attire color consistency across sequences.

### Cited Findings
**MangaNinja** (Liu et al., CVPR 2025 Highlight; arXiv 2501.08332)
- A diffusion-based, reference-guided line art colorizer with a patch-shuffling module for reference↔line-art correspondence and a PointNet-driven point control scheme for fine color matching. It handles discrepant references, large pose variation and multi-subject colorization — [arXiv](https://arxiv.org/abs/2501.08332); [CVF PDF](https://openaccess.thecvf.com/content/CVPR2025/papers/Liu_MangaNinja_Line_Art_Colorization_with_Precise_Reference_Following_CVPR_2025_paper.pdf)
- Repo `ali-vilab/MangaNinjia`. License **CC BY-NC 4.0**. Built on SD v1.5 with CLIP ViT-L/14 and ControlNet lineart (control_v11p_sd15_lineart), plus its own denoising UNet, reference UNet, point_net and ControlNet. About 6 GB VRAM is noted for Windows. The authors ask for feedback on binary line art and say they may fine-tune for it — [GitHub](https://github.com/ali-vilab/MangaNinjia)

**MangaDiT** (arXiv 2508.09709, Aug 2025)
- A DiT-based reference-guided colorizer that finds correspondences implicitly through attention. Hierarchical attention with dynamic weighting adds pooled context to improve region-level color alignment. On ATD-test200 it reports PSNR 27.8 / SSIM 0.944 against MangaNinja's 13.9 / 0.511, and a color-fidelity score of 0.004, about 25x better than the nearest competitor. It colors complex accessories well under drastic pose changes — [arXiv 2508.09709](https://arxiv.org/pdf/2508.09709); [awesomepapers](https://awesomepapers.io/generative-models/papers/2508.09709) (these are the authors' own numbers)

**ColorFlow** (Tencent ARC, arXiv 2412.11815, Dec 2024)
- Described as "the first model designed for fine-grained ID preservation in image sequence colorization". Given a reference pool, it colors hair and **attire** consistently with the references. Three-stage pipeline: Retrieval-Augmented Pipeline (RAP), In-context Colorization Pipeline (ICP), Guided Super-Resolution Pipeline (GSRP). License: **Academic Software Licence** (third-party components excepted) — [Project page](https://zhuang2002.github.io/ColorFlow/); [GitHub](https://github.com/TencentARC/ColorFlow); [HF Space LICENSE](https://huggingface.co/spaces/TencentARC/ColorFlow/blob/main/LICENSE)

**Cobra** (Zhuang et al., SIGGRAPH 2025; arXiv 2504.12240)
- "Efficient Line Art COlorization with BRoAder References": a long-context, fine-grained ID-preservation framework for comic colorization. It uses more than 200 reference images through a Causal Sparse DiT with special positional encodings, causal sparse attention and a KV cache, and supports color hints. Reported to beat baselines in image quality, color-ID accuracy and inference efficiency — [Project](https://zhuang2002.github.io/Cobra/); [CUHK listing](https://research.cuhk.edu.hk/en/publications/cobra-efficient-line-art-colorization-with-broader-references/)
- Repo `zhuang2002/Cobra`, **Apache-2.0**. Inference code and weights were released on 17 Apr 2025 (HF `JunhaoZhuang/Cobra`). Training code is pending — [GitHub](https://github.com/zhuang2002/Cobra)

**ColorizeDiffusion family** (Dingkun Yan / tellurion-kanata)
- ColorizeDiffusion (WACV 2025) is reference-based sketch colorization on latent diffusion. It uses two-stage training to counter the distribution shift between the reference's and sketch's spatial structure — [WACV 2025 CVF](https://openaccess.thecvf.com/content/WACV2025/html/Yan_ColorizeDiffusion_Improving_Reference-Based_Sketch_Colorization_with_Latent_Diffusion_Model_WACV_2025_paper.html); [repo](https://github.com/tellurionkanata/colorizeDiffusion)
- ColorizeDiffusionXL, "Towards High-resolution and Disentangled Reference-based Sketch Colorization" (CVPR 2026 Highlight; arXiv 2603.05971), is SDXL-based at 1024px with enhanced embedding guidance for character colorization and geometry disentanglement — [GitHub ColorizeDiffusionXL](https://github.com/tellurion-kanata/ColorizeDiffusionXL); [arXiv 2603.05971](https://arxiv.org/html/2603.05971)
- Earlier: "Two-step Training: Adjustable Sketch Colorization via Reference Image and Text Tag" (Computer Graphics Forum) — [GitHub sketch_colorizer](https://github.com/tellurion-kanata/sketch_colorizer)
- "Image Referenced Sketch Colorization Based on Animation Creation Workflow" (arXiv 2502.19937) — [emergentmind](https://www.emergentmind.com/papers/2502.19937)

**Other manga colorization and generation**
- inkn'hue (arXiv 2311.01804): manga colorization from multiple priors with an alignment multi-encoder VAE — [arXiv](https://arxiv.org/pdf/2311.01804)
- "Advancing Sequential Manga Colorization for AR Through Data Synthesis" — [ResearchGate](https://www.researchgate.net/publication/387806709_Advancing_Sequential_Manga_Colorization_for_AR_through_Data_Synthesis)
- Comicolorization (2017, semi-automatic manga colorization) — [ar5iv 1706.06759](https://ar5iv.labs.arxiv.org/html/1706.06759)
- **DiffSensei** (CVPR 2025, pp. 28684-28693) is manga *generation* (MLLM + diffusion), not colorization. It produces character-consistent panels that follow text prompts; code, model and dataset are open — [GitHub](https://github.com/jianzongwu/DiffSensei); [CVF](https://openaccess.thecvf.com/content/CVPR2025/html/Wu_DiffSensei_Bridging_Multi-Modal_LLMs_and_Diffusion_Models_for_Customized_Manga_CVPR_2025_paper.html)
- "Manga Generation via Layout-controllable Diffusion" (arXiv 2412.19303) — [arXiv](https://arxiv.org/abs/2412.19303)
- Curated list: MarkMoHR/Awesome-Image-Colorization — [GitHub](https://github.com/MarkMoHR/Awesome-Image-Colorization)

### Inferences
- For per-character clothing color consistency across a chapter, multi-reference methods (Cobra, ColorFlow) match the problem best, since they condition on a pool of colored references per character and costume. Cobra's Apache-2.0 license and released weights make it the most deployable commercially. ColorFlow (Academic license) and MangaNinja (CC BY-NC) are research-only.
- MangaNinja's point control lets a human enforce garment colors, e.g., by clicking reference-jacket to target-jacket correspondences. Points could be generated automatically from garment masks: match centroids of the same garment class between the reference and the target.
- MangaDiT's large reported margin over MangaNinja should be read with caution until reproduced independently. A PSNR of 13.9 for MangaNinja suggests a strongly mismatched evaluation setting.
- Several of these models expect clean line art. Running MangaLineExtraction first (section 4) is probably needed for screentoned pages; MangaNinja's authors explicitly flag binary line art as a possible weak spot.

### Gaps
- Style2Paints (V4/V5, lllyasviel), AnimeDiffusion (2023/2024), and community ControlNet manga colorization workflows were not verified this session; their licenses and current status are unconfirmed.
- No independent head-to-head benchmark of Cobra vs. ColorFlow vs. MangaNinja vs. MangaDiT on clothing-specific color consistency was found.
- Cobra's base model and quantitative color-ID metrics were not on the README.

---

## 8. Line art vectorization, 2.5D layers, and 3D garment reconstruction from a single anime or manga drawing

### Takeaway
For animation-ready assets, two families matter: **2.5D layer decomposition** (See-through, SIGGRAPH 2026), which yields per-garment inpainted layers for Live2D-style rigs, and **semantic-decomposed 3D character generation** (StdGEN, CVPR 2025; StdGEN++, 2026), which yields separate body, clothes and hair meshes. Photo/sketch sewing-pattern recovery (SewFormer, GarmentDiffusion, DressWild, SketchTailor) exists but has not been shown on anime or manga input. PAniC-3D (CVPR 2023) covers heads only.

### Cited Findings
- **StdGEN: Semantic-Decomposed 3D Character Generation from Single Images** (CVPR 2025; Tsinghua and Tencent AI Lab; arXiv 2411.05738) generates 3D characters with separate body, **clothes** and hair in about 3 minutes. It uses a Semantic-aware Large Reconstruction Model (S-LRM) plus multi-view diffusion and iterative multi-layer surface refinement, and is reported SOTA for 3D anime character generation in geometry, texture and decomposability — [GitHub hyz317/StdGEN](https://github.com/hyz317/StdGEN); [Project](https://stdgen.github.io/); [CVF](https://openaccess.thecvf.com/content/CVPR2025/papers/He_StdGEN_Semantic-Decomposed_3D_Character_Generation_from_Single_Images_CVPR_2025_paper.pdf)
- **StdGEN++** (arXiv 2601.07660, Jan 2026) is described as a comprehensive system for semantic-decomposed 3D character generation — [arXiv](https://arxiv.org/html/2601.07660)
- **DreamCharacter-1** (arXiv 2607.07817, 2026) covers going from 3D generative foundation models to product-ready characters — [arXiv](https://arxiv.org/pdf/2607.07817) (title only seen)
- **PAniC-3D** (CVPR 2023, arXiv 2303.14587) reconstructs stylized 3D *heads* from anime portraits. A line-removal/line-filling GAN converts the illustration to a diffuse-render style, and a volumetric radiance field represents geometry. Trained on 11.2k VRoid models and 1k VTuber portraits, and evaluated on the AnimeRecon benchmark — [CVF PDF](https://openaccess.thecvf.com/content/CVPR2023/papers/Chen_PAniC-3D_Stylized_Single-View_3D_Reconstruction_From_Portraits_of_Anime_Characters_CVPR_2023_paper.pdf); [arXiv](https://arxiv.org/pdf/2303.14587)
- **NOVA-3D** (arXiv 2405.12505): non-overlapped views for 3D anime character reconstruction — [arXiv](https://arxiv.org/pdf/2405.12505)
- **DrawingSpinUp** (SIGGRAPH Asia 2024): 3D animation from single character drawings — [ACM DL](https://dl.acm.org/doi/10.1145/3680528.3687593)
- **See-through** (SIGGRAPH 2026) provides 2.5D layered PSD output with clothing layers; see section 3 — [GitHub](https://github.com/shitagaki-lab/see-through)
- Sewing-pattern recovery from photos and sketches: **SewFormer**, "Towards Garment Sewing Pattern Reconstruction from a Single Image" (arXiv 2311.04218, repo sail-sg/sewformer); **GarmentDiffusion** (arXiv 2504.21476) takes text, garment sketches or partial patterns as conditions; **DressWild** (single in-the-wild image → 2D patterns + 3D garments); **SketchTailor** (sketch-driven pattern reconstruction); **SPnet** (2024); "Single View Garment Reconstruction Using Diffusion Mapping Via Pattern Coordinates" (SIGGRAPH 2025) — [arXiv 2311.04218](https://arxiv.org/abs/2311.04218); [GitHub sewformer](https://github.com/sail-sg/sewformer); [GarmentDiffusion](https://arxiv.org/html/2504.21476v4); [SketchTailor](https://www.researchgate.net/publication/394373322_SketchTailor_Lightweight_sketch-driven_modeling_for_high-fidelity_garment_pattern_reconstruction); [ACM SIGGRAPH 2025](https://dl.acm.org/doi/10.1145/3721238.3730651)

### Inferences
- For a manga-to-anime pipeline aimed at 2D animation, See-through-style 2.5D layers are the most direct target. For 3D/cel-shaded anime, StdGEN's separate clothes mesh is the closest open option. Both expect color anime input, so the order should be manga → colorize (section 7) → decompose or reconstruct.
- Sewing-pattern methods could turn garment silhouettes extracted from manga into simulatable cloth, but anime proportions and stylized folds are a large domain gap with no evidence of transfer.

### Gaps
- CharacterGen (SIGGRAPH 2024) and "Sketch2Cloth" were not verified this session; their details, repos and licenses are unconfirmed.
- StdGEN and PAniC-3D licenses were not retrieved.
- No line-art *vectorization* paper specific to manga clothing was researched in depth, due to the tool-call budget.

---

## 9. Practical recipe (2025-2026): manga page → per-character clothing masks + attribute tags + color assignments

### Takeaway
No single model does this end to end. As of 2026 a buildable stack chains: restoration/line extraction → Magi (panels, characters, identity clusters) → character masks (MangaSeg-finetuned or SkyTNT ISNet) → garment masks (SAM 3 text prompts, optionally refined by a manga-finetuned SegFormer/SCHP) → clothing tags (wd-eva02 / camie) → outfit clustering per identity → multi-reference colorization (Cobra for commercial use, ColorFlow/MangaNinja for research) → optional See-through layering or StdGEN 3D. The license chain decides what is shippable.

### Cited Findings (components and their licenses)
- Restoration: Manga Restoration (CVPR 2021) — [arXiv](https://arxiv.org/abs/2105.06830). Line extraction: MangaLineExtraction_PyTorch — [GitHub](https://github.com/ljsabc/MangaLineExtraction_PyTorch)
- Page parsing and identity: Magi v1/v2/v3, academic-only — [GitHub](https://github.com/ragavsachdeva/magi)
- Character masks: MangaSeg annotations on Manga109 (commercial use only via the Manga109-s subset terms) — [HF](https://huggingface.co/datasets/MS92/MangaSegmentation); SkyTNT anime-segmentation, Apache-2.0 — [GitHub](https://github.com/SkyTNT/anime-segmentation)
- Garment masks: SAM 3, text-prompted concept segmentation — [Roboflow](https://blog.roboflow.com/what-is-sam3/); segformer_b2_clothes (MIT code, ATR non-commercial) — [GitHub](https://github.com/mattmdjaga/segformer_b2_clothes)
- Screentone region grouping: ScreenVAE / Manga Rescreening — [arXiv 2306.04114](https://arxiv.org/pdf/2306.04114)
- Tags: wd-eva02-large-tagger-v3 (Apache-2.0) — [HF](https://huggingface.co/SmilingWolf/wd-eva02-large-tagger-v3); camie-tagger (70,527 tags, micro-F1 67.3%) — [PromptLayer](https://www.promptlayer.com/models/camie-tagger/); DeepDanbooru (MIT) — [GitHub](https://github.com/KichangKim/DeepDanbooru)
- Colorization: Cobra (Apache-2.0, more than 200 refs) — [GitHub](https://github.com/zhuang2002/Cobra); ColorFlow (Academic) — [GitHub](https://github.com/TencentARC/ColorFlow); MangaNinja (CC BY-NC 4.0, point control) — [GitHub](https://github.com/ali-vilab/MangaNinjia); ColorizeDiffusionXL — [GitHub](https://github.com/tellurion-kanata/ColorizeDiffusionXL)
- Layering and 3D: See-through (Apache-2.0) — [GitHub](https://github.com/shitagaki-lab/see-through); StdGEN — [GitHub](https://github.com/hyz317/StdGEN)

### Inferences (the suggested build, which is a synthesis and was not validated end to end)
1. **Ingest and clean:** upscale/restore low-resolution scans (Manga Restoration). Keep the original page, and also produce (a) a clean line map (MangaLineExtraction) and (b) a screentone feature map (ScreenVAE-style) for each page.
2. **Parse the page:** run Magi (v2/v3) for panel order, character boxes and identity clusters, seeded with a small character bank of named references. For commercial use, retrain a detector on Manga109-s + MangaSeg in place of Magi.
3. **Character masks:** use a MangaSeg-finetuned instance segmenter (e.g., Mask2Former/YOLO-seg trained on Manga109-s body masks) or SkyTNT ISNet on each crop; SAM 2/3 box-prompted refinement from the Magi box is an alternative.
4. **Garment masks:** prompt SAM 3 inside each character mask with a garment vocabulary chosen per character from tagger output (e.g., `serafuku` → "sailor collar", "neckerchief", "pleated skirt"). Resolve overlaps with a fixed priority order. Use screentone clusters (GMM on ScreenVAE features) to snap or merge garment regions. Over time, collect corrected masks and fine-tune a fixed-label SegFormer/Mask2Former "manga garment parser", bootstrapping from color anime parsing data turned into manga style (line extraction + synthetic screentones).
5. **Attribute tags:** run wd-eva02-large-v3 and/or camie-tagger on each character crop and each garment crop, keeping shape and type tags and dropping color tags on B/W input. Store tags per (character_id, panel, garment).
6. **Outfit clustering:** per identity cluster, embed garment-masked crops (SigLIP/DINOv2 features + tag vector) and cluster them into costumes. Keep a human-in-the-loop review UI to confirm costume changes.
7. **Color assignment:** for each (character, costume), pick or produce one or more color reference sheets (official color pages, anime model sheets, or a human-painted key frame). Colorize panels with Cobra (multi-reference, commercial-friendly license), or with MangaNinja using auto-generated point correspondences between matching garment masks in reference and target. Write back a per-garment color palette (e.g., dominant Lab color in each garment mask after colorization) to a character/costume "color bible" JSON.
8. **Optional asset stage:** for 2D animation, run See-through on colorized key poses to get layered PSDs with clothing layers; for 3D, use StdGEN to get separated clothes meshes.
- License note: a fully commercial stack would avoid Magi, ColorFlow, MangaNinja, Manga109 (non-s) and ATR-trained weights, and would rely on Manga109-s + MangaSeg (subject to its terms), SkyTNT, SAM 3 (check Meta's license), WD/camie taggers, Cobra, and See-through.

### Gaps
- No published system performs this full chain. The recipe is an inference from components, with no measured end-to-end accuracy.
- SAM 3's license terms and performance on screentoned manga need hands-on validation.
- MangaSeg's own license, separate from Manga109-s, needs confirmation before commercial training.
