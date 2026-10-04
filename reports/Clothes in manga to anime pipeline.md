# Anchor Every Costume Before Anything Moves

Whether the work is done by hand or with AI, clothes in a manga-to-anime adaptation stay consistent for one reason: someone fixes a costume specification before animation starts and checks every later drawing against it. That specification is a human-approved model sheet (settei) plus a color-design palette. Traditional studios simplify manga costumes into these sheets mainly for economic reasons. Animators are paid per drawing (in-betweens run roughly **¥280–400 per drawing**) no matter how many folds, checks or ribbons each drawing contains. Because manga is monochrome, a color designer has to invent the palette, starting from the author's color pages and rules of thumb such as "solid black becomes a dark color." In 2026 no single AI system carries clothing from a manga page to finished animation. Each stage has a good component: Magi for parsing pages, SAM 3 and See-through for segmentation and layering, WD14-style taggers for garment attributes, Cobra and paint-bucket colorizers for color, and AniDoc, ToonComposer and LongAnimation for line-art video. Two gaps remain. No public dataset labels garments in screentoned manga, and free video generators still let patterns and accessories drift. The most reliable AI design copies the human approach: keep garment geometry from line art drawn by an artist and use models only to assign colors from an approved palette. For cloth motion, the deciding question is who draws the fold shapes. A human (hand-drawn work, or hand-keyed 3D as in *Guilty Gear Xrd*) gives an authentic anime look at high labor cost. A solver (physics, pendulum rigs, diffusion) is cheap per shot but gives generic motion that still has to be stylized. A 3–5 person team can realistically build a costume-continuity and palette-enforcement tool in about 12 weeks from commercially licensed parts. GPU cost is small, on the order of **$400–1,700 to generate video for one episode**, against traditional episode budgets of **$160k–320k**. The real constraints are human review time, model licenses (many of the best models are non-commercial) and an adaptation contract that explicitly permits training. The first section compares eight named recipes for making anime clothes from manga with full control over design, color, pattern and motion. For a solo creator or small team it recommends line art kept as control, palette-driven colorization and a 2D rig or short line-art video clips. For a commercial studio it recommends human settei and key animation, with AI enforcing color and assisting in-betweens under quality checks.

## Eight ways to create anime clothes the way you want

"The way we want" means four kinds of control: over **design** (silhouette, construction, details), **color** (exact palette per garment region and lighting), **pattern** (plaid, emblems, prints that stay locked to folds) and **motion** (how fabric drags, flutters and settles). The recipes below differ mainly in who authors each of the four: a human, a structural condition such as line art, or a generative model. The later sections give the evidence behind each recipe in detail.

### (A) Fully traditional: hand-drawn settei, color design and 2D animation

**How it works.**

1. The character designer (or a dedicated costume designer) turns the manga outfits into settei: turnarounds, construction details and accessories, with fewer lines, colors and ornaments than the manga.
2. The color designer builds normal, shadow and highlight palettes plus evening and night variants. Where no color page exists, solid black becomes a dark color.
3. Key animators draw the cloth, including drag, follow-through and nabiki (cloth flapping in wind).
4. Animation directors correct drawings against the settei.
5. In-betweeners and painters finish; the per-episode color coordinator (色指定) supplies the color models.

**Tools:** paper or Clip Studio Paint, TVPaint, Harmony, OpenToonz.

**Control and consistency:** control is total across all four dimensions, because every fold is a human decision. Consistency depends on the correction passes; outfit errors are a recognized category of drawing mistake.

**Cost:** high per shot. In-betweens are paid about ¥280–400 per drawing whatever their complexity ([Levtech Creator](https://creator.levtech.jp/tips/article/195/)). Patterns can be brute-forced, as with *Demon Slayer*'s hand-counted checks ([Togetter](https://togetter.com/li/1614752)), but at a large labor cost.

**License:** no model-license issues. It still needs the adaptation license.

**Best use:** hero cuts, emotional close-ups, and any shot where fold logic or a pattern must read perfectly.

### (B) Manga line art as control, colorized against an outfit color bible

**How it works.**

1. Restore the scans and extract clean lines; MangaLineExtraction strips screentone ([GitHub](https://github.com/ljsabc/MangaLineExtraction_PyTorch)). Alternatively, have an artist clean the panels into animation line art.
2. Build a color bible per (character, outfit): an exact hex value for each region, from the author's color pages or a human color designer.
3. Colorize with a reference model. Options:
   - **Cobra**: more than 200 references, Apache-2.0 weights ([GitHub](https://github.com/zhuang2002/Cobra)).
   - **ColorFlow**: retrieves references per panel for consistent attire; Academic license ([project](https://zhuang2002.github.io/ColorFlow/)).
   - **MangaNinja**: point control, so a human can pin a jacket in the reference to the same jacket in the target; CC BY-NC 4.0, 512² ([GitHub](https://github.com/ali-vilab/MangaNinjia)).
   - **Paint-bucket methods**: these fill each closed region with the exact color-sheet RGB ([arXiv 2410.19424](https://arxiv.org/html/2410.19424v1); [DACoN](https://openaccess.thecvf.com/content/ICCV2025/papers/Nagata_DACoN_DINO_for_Anime_Paint_Bucket_Colorization_with_Any_Number_ICCV_2025_paper.pdf)).
4. Run palette checks using color difference (ΔE) per region.

**Control and consistency:** design and pattern stay exactly as drawn, because the model never redraws geometry. Color control is very high with paint-bucket fills and high with diffusion colorizers, which can repaint small accessories. Consistency across a chapter is the best of any AI recipe when every panel uses the same per-outfit references.

**Cost:** low. MangaNinja runs on about 6 GB of VRAM, and the main labor is line cleanup and palette design.

**License:** use Cobra or paint-bucket methods for commercial work. MangaNinja and ColorFlow are for prototypes only.

**Best use:** color keys, model sheets, still or limited-motion content, and the color-enforcement stage of every other recipe.

### (C) Anime base model + per-outfit LoRA + line-art ControlNet

**How it works.**

1. Curate the approved model sheets and colored panels for each outfit.
2. Caption them with a fixed Danbooru tag block per costume, pruning generic color tags to limit bleed ([lora-dataset-studio](https://github.com/perfectgf/lora-dataset-studio/blob/v1/docs/DATASET_GUIDE.md)).
3. Train an outfit or character LoRA: about 1,500–2,500 SDXL steps on 10–24 GB, or about $8 per 1,000 steps hosted ([bestaiweb](https://www.bestaiweb.ai/how-to-train-a-custom-lora-for-flux-and-sdxl-with-kohya-ss-ai-toolkit-and-fal-ai-in-2026/)).
4. Generate new poses or panels with line-art or pose ControlNet holding the structure.
5. Snap colors to the bible (recipe B).

**Control and consistency:** design control is high when line art conditions generation and medium with pose-only control. Patterns and emblems drift without line art. Consistency is good for a single outfit, but multi-outfit LoRAs "can still show bleedthrough" ([hollowstrawberry guide](https://huggingface.co/hollowstrawberry/stable-diffusion-guide/discussions/7)), so keep one costume asset per shot.

**Cost:** low, about $250–750 of training for 30–50 outfits per series (my estimate).

**License:** the base model decides. Animagine XL 4.0 (OpenRAIL++-M) and Neta Lumina (Apache-2.0) are workable ([HF](https://huggingface.co/cagliostrolab/animagine-xl-4.0); [HF](https://huggingface.co/neta-art/Neta-Lumina/blob/main/README.md)). NoobAI-XL forbids commercializing outputs ([model card](https://huggingface.co/Laxhar/noobai-XL-1.1/raw/main/README.md)). Training on the manga needs the rights holder's explicit permission.

**Best use:** new keyframes, poses and angles the manga never drew, and promotional art.

### (D) Zero-shot multi-reference editors for outfit design and swaps

**How it works.** Feed a character image plus garment references to an in-context editor, prompt the change ("same outfit, sitting, back view" or "swap to outfit B"), then correct and recolor. Editors and their reference limits:

- FLUX.1 Kontext.
- FLUX.2: up to 10 references ([BFL](https://bfl.ai/blog/flux-2)).
- Qwen-Image-Edit-2509/2511: 1–3 inputs, accepts ControlNet maps ([HF](https://huggingface.co/Qwen/Qwen-Image-Edit-2509)).
- Nano Banana Pro: up to 14 references ([Scenario](https://help.scenario.com/articles/7568607761-gemini-image-models-nano-banana-family)).

**Control and consistency:** no training needed, and iteration is fast. Control is only medium, and small costume details are the weak point. Gemini failed a clothing-extraction test in one informal comparison ([HF blog](https://huggingface.co/blog/MonsterMMORPG/nano-banana-gemini-25-flash-image-full-tutorial)). Kontext's published consistency score is a face metric ([arXiv 2506.15742](https://arxiv.org/html/2506.15742v2)).

**Cost:** cheap per image, either API fees or a local GPU.

**License:** Qwen-Image code is Apache-2.0. FLUX [dev] weights are non-commercial unless you buy a license, and FLUX Pro and Gemini are paid APIs. Photographic try-on models (IDM-VTON, CatVTON and others) are CC BY-NC-SA and suit flat-shaded anime poorly ([IDM-VTON](https://github.com/yisol/IDM-VTON)).

**Best use:** costume design exploration, quick variants and states (jacket off, wet), and drafts that a designer then redraws into settei.

### (E) Layered 2D rig: See-through layers into Live2D or Spine physics

**How it works.**

1. Colorize a key pose (recipe B).
2. Decompose it with **See-through** into up to 23 inpainted layers, including clothing and accessories, exported as a PSD. It is Apache-2.0 and runs in 8–12 GB with low-VRAM variants ([GitHub](https://github.com/shitagaki-lab/see-through)).
3. Cut the garment parts (skirt panels, ribbons, sleeve ends).
4. Rig them in Live2D Cubism, whose physics groups include "swinging skirt" with pendulum reaction, amplitude, weight and convergence settings ([Live2D manual](https://docs.live2d.com/4.2/en/cubism-editor-manual/physics-operation/)). Alternatives are Spine 4.2 physics constraints ([Esoteric](http://en.esotericsoftware.com/spine-physics-constraints)) or Harmony's Envelope and Curve deformers for capes ([Harmony docs](https://docs.toonboom.com/help/harmony-24/premium/master-controller/about-deformer-on-deformer.html)).

**Control and consistency:** design, color and pattern are frozen in the art, so consistency is perfect. Motion is automatic sway inside the drawn range, with no true turnarounds or new folds.

**Cost:** a medium one-time rig build, then nearly free per shot.

**License:** See-through is Apache-2.0. Live2D, Spine and Harmony are paid tools.

**Best use:** VTuber-style content, visual novels, motion comics, dialogue scenes, and small-team series with limited camera angles.

### (F) 3D garment route: patterns or meshes, cloth sim, toon shader

**How it works.**

1. Build the garment from the settei turnarounds, by one of three routes:
   - Sew it as a pattern in Marvelous Designer or CLO, which now has a Toon Shader preview ([CG Channel](https://www.cgchannel.com/2026/04/clo-virtual-fashion-releases-marvelous-designer-2026-0/)).
   - Draft a pattern with **ChatGarment**, which outputs GarmentCode JSON from images or sketches, then correct it by hand ([CVPR 2025](https://openaccess.thecvf.com/content/CVPR2025/papers/Bian_ChatGarment_Garment_Estimation_Generation_and_Editing_via_Large_Language_Models_CVPR_2025_paper.pdf)).
   - Start from a **StdGEN** clothes mesh, separated from body and hair in about 3 minutes ([GitHub](https://github.com/hyz317/StdGEN)).
2. Simulate in Houdini Vellum, Blender, Unreal Chaos Cloth (production-ready in UE 5.8) or Magica Cloth. Use bone chains or VRM SpringBone for cheap real-time skirts.
3. Stylize so it reads as anime: coarser wrinkles, edited normals, retiming to twos or threes. Use the ghost-frame trick for stable sims on twos ([Cartoon Brew](https://www.cartoonbrew.com/feature-film/if-its-not-broke-break-it-sony-imageworks-renegade-approach-to-spider-man-into-the-spider-verse-167321.html)), or hand-keyed model swaps as on *Guilty Gear Xrd* ([BlenderNation](https://www.blendernation.com/2015/07/26/junya-c-motomura-behind-the-scenes-of-guilty-gear-xrd/)).
4. Render with Pencil+ 4 or Unity Toon Shader (UTS3).

**Control and consistency:** a modeled costume is perfectly consistent from any angle and reusable across a franchise, as with *Love Live!*'s modeled live costumes ([Wikipedia JA](https://ja.wikipedia.org/wiki/%E3%82%B5%E3%83%B3%E3%82%B8%E3%82%B2%E3%83%B3)). Patterns become textures that follow folds exactly. Motion control is high when keyed. Raw simulation looks "CG."

**Cost:** high up front. *Trigun Stampede*'s CG modeling began 3–4 years before broadcast ([ComicBook.com](https://comicbook.com/anime/news/trigun-stampede-team-interview-cg-animation-talk/)). Per-shot cost is medium.

**License:** mostly commercial software. Research pattern tools are unproven on anime.

**Best use:** idol and dance lives, crowds, action wide shots, games, and long franchises that amortize the assets.

### (G) Line-art video methods for motion

**How it works.**

1. Humans draw key sketches. AI fills the in-betweens and color while conditioning every window on the same outfit reference sheet. Options:
   - **ToonCrafter**: interpolation, 512×320, 16 frames ([GitHub](https://github.com/Doubiiu/ToonCrafter)).
   - **AniDoc**: sketch-sequence colorization from a design reference, 14 frames, about 14 GB ([GitHub](https://github.com/yihao-meng/AniDoc)).
   - **LVCD**.
   - **ToonComposer**: one sketch plus one colored frame ([arXiv 2508.10881](https://arxiv.org/abs/2508.10881)).
   - **LongAnimation**: color stability over about 500 frames ([arXiv 2507.01945](https://arxiv.org/abs/2507.01945)).
2. Reduce the output to the show's timing (twos or threes).
3. Clean up by hand and snap colors to the palette.

**Control and consistency:** design is held by the sketches, so garments cannot change shape between keys. Color consistency is good within a window and weakest at window seams on long shots. Large cape and skirt arcs need dense keys ([arXiv 2508.10881](https://arxiv.org/pdf/2508.10881)).

**Cost:** low GPU cost plus cleanup labor.

**License:** the code is Apache-2.0, but AniDoc and LVCD depend on Stable Video Diffusion weights, and ToonCrafter calls itself a "research exploration," so check base weights before commercial use. Free generators such as Wan 2.2 (Apache-2.0) are fine for B-roll but let garments drift.

**Best use:** in-betweening assistance and short limited-motion shots with small secondary cloth motion.

### (H) The hybrid pipeline

**How it works.**

1. A human-approved outfit bible (settei, palette, accessory counts, license fields) is the only source of truth (recipe A's front end).
2. CV tools detect characters and outfit changes in the manga to populate it.
3. Recipe D drafts designs and C generates new angles, but only the designer's redrawn sheets become canonical.
4. Key animation stays human for hero cuts.
5. Recipe G assists in-betweens.
6. Recipe B paint-bucket enforcement locks colors on every frame.
7. Recipe F handles crowd and dance wide shots, and recipe E handles low-budget dialogue content.
8. Automated QA runs on every cut, followed by a human color check.

**Control and consistency:** control is total where it matters, because a human authors design and key motion and data enforces color. Consistency is the highest available, since every stage reads the same bible.

**Cost:** medium. GPU spend is small, and review labor dominates.

**License:** it is built only from commercially licensed parts, with a license register.

**Best use:** any serious adaptation.

### Comparison

| Recipe | Design control | Color control | Pattern fidelity | Motion control | Consistency | Cost / time | Commercial license status | Best use |
|---|---|---|---|---|---|---|---|---|
| A Traditional hand-drawn | Total | Total | Total but costly | Total | Depends on correction passes | Highest labor | Clean (needs adaptation license) | Hero cuts, flagship TV/film |
| B Line art + reference colorization | Exact (inherited) | Very high (paint-bucket) to high (diffusion) | Exact lines; color may slip on tiny parts | None by itself | Best AI option for color | Low | Cobra, paint-bucket OK; MangaNinja, ColorFlow non-commercial | Color stage of every pipeline |
| C Anime base + outfit LoRA + ControlNet | High with line art | High after snapping | Medium; drifts without line art | Stills only | Good per outfit; bleed across outfits | Low (~$8–15 per LoRA) | Animagine, Neta Lumina OK; NoobAI no | New poses and angles, promo art |
| D Multi-reference editors | Medium | Medium | Low–medium | Stills only | Medium; details drop | Very low, fast | Qwen Apache code; FLUX/Gemini license or API; try-on models non-commercial | Design exploration, variants |
| E Layered 2D rig | Frozen art | Frozen | Exact | Low (sway in range) | Perfect within range | Medium setup, ~zero per shot | See-through Apache; paid rig tools | Motion comics, VTuber-style, dialogue |
| F 3D garment + toon shader | High | Exact (texture) | Exact, follows folds | High if keyed; generic if raw sim | Perfect from any angle | Very high up front, medium per shot | Commercial DCC tools | Idol lives, crowds, action, games |
| G Line-art video | Held by keys | Good within windows | Held by keys; can swim | Medium; needs dense keys for big arcs | Good; seams on long shots | Low GPU + cleanup | Check base weights (SVD, CogVideoX) | In-between assist, short shots |
| H Hybrid | Total where it matters | Exact (enforced) | High | High | Highest | Medium; review dominates | Commercial-only stack | Any serious adaptation |

### Recommendations

**Solo creator or small team (1–5 people).** Use **B + E, with G for selected shots and D/C as design aids**.

1. Clean the manga panels into line art.
2. Hand-define one color bible per outfit.
3. Colorize with Cobra or a paint-bucket method so colors are exact. MangaNinja and ColorFlow are acceptable only for non-commercial or research work.
4. Animate mainly with See-through layers rigged in Live2D or Spine. Garments then never drift, and skirt and ribbon motion comes free from physics.
5. Use AniDoc or ToonComposer for a handful of short motion shots.
6. Use Qwen-Image-Edit or an Animagine/Neta Lumina outfit LoRA with line-art ControlNet to draft angles the manga never drew, then redraw or approve them before they enter the bible.

This stack runs on one or two 24 GB consumer GPUs, keeps training costs to tens or hundreds of dollars, and trades motion richness for near-perfect costume consistency.

**Commercial studio.** Use **H with A as the backbone**.

- Human costume and character designers produce settei, and a color designer owns the palette.
- Key animation stays human.
- AI is limited to:
  - CV-based costume continuity tracking;
  - paint-bucket color enforcement against the bible (recipe B);
  - in-between assistance with human keys (recipe G);
  - 3D garments for idol, crowd and action wide shots (recipe F).
- Automated QA on every cut, with per-cut provenance logs.
- Only commercially licensed components (Wan 2.2, Cobra, See-through, Animagine or Neta Lumina, or paid FLUX and Kling licenses), recorded in a model-license register.
- No NoobAI, MangaNinja, ColorFlow, photographic try-on models or HunyuanVideo in shipped work.
- An adaptation contract with an explicit AI-training clause.

This matches where industry evidence already points. Toei targeted color specification and in-betweens, not design ([Gizmodo](https://gizmodo.com/animation-studio-toei-wants-to-use-ai-for-future-productions-2000603817)). WIT's 2026 apology shows that undocumented AI use is itself a production risk ([ANN](https://www.animenewsnetwork.com/news/2026-04-10/wit-studio-apologizes-for-using-generative-ai-in-opening-sequence-of-ascendance-of-a-bookworm-part-/.236271)).

## Studios simplify costumes because animators are paid per drawing

### The character designer turns detail into an "impression"

An anime character designer refines the mangaka's art into model sheets that keep the original's look while cutting line density and color count. A *Pretty Cure* director describes this as simply "reducing lines" so the character stays recognizable but becomes easy to move ([Diamond Online](https://diamond.jp/articles/-/312739); [Wikipedia JA: キャラクターデザイン](https://ja.wikipedia.org/wiki/%E3%82%AD%E3%83%A3%E3%83%A9%E3%82%AF%E3%82%BF%E3%83%BC%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3)). Costumes absorb most of these cuts. The reason is the pay structure. Published rates for in-betweens are roughly **¥280–400 per drawing (¥150–280 when outsourced)**, and drawing difficulty "is generally not reflected in pay": a cut that only moves a mouth pays the same as a dynamic action cut with the same number of drawings ([Levtech Creator](https://creator.levtech.jp/tips/article/195/)). Industry commentary describes anime "quality" as **screen information density × number of drawings** ([note](https://note.com/taneri777/n/nddb3217e1453)). It follows that a costume with twice the lines roughly halves an animator's effective hourly rate. Designers therefore simplify up front, animators quietly simplify rushed cuts, and animation directors absorb the hidden cost of redrawing lost detail. This chain is an inference from the pay structure; no source measured it directly.

Across sources the simplification rules are consistent: fewer lines, fewer colors, fewer ornaments, and preserve the silhouette and overall impression rather than literal detail. The *PriPara* art designer describes her skill as cutting the number of colors and ornaments so a costume is easy to animate yet still cute and flashy. Her team also picks colors that read well under glow-stick lighting in 3DCG live scenes ([Creators Station](https://www.creators-station.jp/interview/curiousity/34834)). Sakuga Blog's design coverage adds a caution: simply making drawings simpler is not enough without clear reference for how the design behaves in different situations. It also praises clothing that visibly behaves as a separate layer from the body ([Sakuga Blog: Pre-Production #3](https://blog.sakugabooru.com/2017/07/21/the-pre-production-of-anime-3-design-work/)). No public source gives numeric rules such as "at most N folds per sleeve." Such rules most likely sit in each show's internal animation-notes sheets.

### Clothing-centric shows hire costume specialists

When clothing is the subject of the story, studios add a dedicated costume credit. On CloverWorks' *My Dress-Up Darling*, costume designer **Erika Nishihara** worked with director Keisuke Shinohara to simplify the manga's cosplay outfits while keeping their impression ([Febri](https://febri.jp/topics/bisquedoll-anime1/); [official staff page](https://bisquedoll-anime.com/staffcast/)). In season 2 she researched how each costume is actually built so animators could understand its construction ([Febri S2](https://febri.jp/topics/bisquedoll-anime-season2-1/); [Anime!Anime!](https://animeanime.jp/article/2026/01/29/95572.html)). *Revue Starlight the Movie* credits a separate 衣裳武器デザイン (costume and weapon design) to Takushi Koide ([film staff page](https://cinema.revuestarlight.com/cast-staff/)). Prestige original films go further. On Hosoda's *Belle*, stylist Daisuke Iga, ANREALAGE designer Kunihiko Morinaga and flower artist Megumi Shinozaki designed the costumes ([TOKION](https://tokion.jp/2021/07/15/ryu-to-sobakasu-no-hime-anrealage/)), and Belle alone has **eight costumes** ([Cinema Today](https://www.cinematoday.jp/news/N0124952)). The pattern is clear: the more a story is about clothes (cosplay, idols, stage, fashion), the more likely it is to get a costume designer, outside fashion expertise, or 3DCG costume models.

### The settei package and color design are the costume's source of truth

A settei package normally includes front, side and back turnarounds; size charts; **costume details (clothing construction, accessories, hair)**; prop sheets; and color settei with material notes ([Animation Cel glossary](https://animation-cel.com/glossary/settei)). Shadow areas are marked with conventional fills: **blue for shadows on clothes, eyes and teeth; green or pink for hair; yellow for highlights** ([Sakuga Blog](https://blog.sakugabooru.com/2017/07/21/the-pre-production-of-anime-3-design-work/)). Color runs as a chain. The 色彩設計 (color designer) sets every color in the show, the per-episode 色指定 (color coordinator) specifies colors cut by cut, and 仕上 (finishing) staff paint ([Studio Ghibli production diary](https://www.ghibli.jp/ged_01/30special/000611.html)). Designers start from a "normal" daylight palette and add scene variants. Evening shifts warm and slightly darker; night shifts toward blue with much lower saturation and brightness ([Masako Sato, note](https://note.com/sa10ma/n/n9eed72859a92); [Tooniq](https://tooniq.co.jp/tools/anime-color-palette/)). Each clothing color therefore needs at least a normal and a shadow tone for every lighting condition it appears in, so every extra outfit multiplies color-model work.

Monochrome source material changes the job. A color designer on *おとなりに銀河* describes requesting all of the original's color illustrations, using dark colors where the manga uses solid black (ベタ), and adjusting hues so several characters in one scene do not blend together ([otonari-anime interview](https://otonari-anime.com/special/interview05.html)). By analogy, a mid-grey screentone most plausibly maps to a mid-value color, but no source states that rule outright. Consistency is enforced afterward. The episode director checks layouts, then the 作画監督 (animation director) compares key drawings against the storyboard and settei and draws corrections on overlay paper ([KADOKAWA Academy](https://school.kadokawa.co.jp/anime/column/how-to-become-an-animation-director/)). Meanwhile the 色指定 supplies each episode's color models and creates new ones when needed ([Sakuga Blog glossary](https://blog.sakugabooru.com/glossary/color-designer/)).

### Patterns get simplified, composited, or brute-forced

Complex patterns such as plaid, checkered haori and floral kimono are handled in one of three ways: drop or simplify the pattern, composite a texture during compositing (撮影), or hand-draw it on every frame. The famous case is ufotable's *Demon Slayer*. According to a widely shared compilation, Tanjiro's black-and-green checkered haori is **not texture-mapped**: animators count the squares by hand, and finishing staff check the in-betweeners' paint indications. Commenters call this "brute force" (力技) ([Togetter](https://togetter.com/li/1614752)). The same compilation says the anime keeps the manga's symmetric back pattern, a deliberate "drawing lie," because a real sewn kimono would offset it. Treat both claims as medium confidence, since the original staff source is not identified. ufotable's 2026 *Infinity Castle* making-of was framed around its "overwhelming inefficiency" (圧倒的非効率) ([GameBusiness.jp](https://www.gamebusiness.jp/article/2026/01/14/25960.html)). Illustration tutorials explain the tradeoff: a pasted texture looks flat unless it is warped to the garment's folds, and plaid checks must line up per pleat ([ichi-up](https://ichi-up.net/2015/033); [mangania](https://cs.mangania.jp/archives/5847)). Hand-drawn patterns stay locked to folds, but the cost falls on in-betweening and painting, not only on key animation.

### 3DCG takes over when costume detail × choreography × headcount explodes

*Love Live!* set the hybrid template: **"bust-ups are hand-drawn; wide shots that include from the skirt down are 3DCG"** ([法華狼の日記](https://hokke-ookami.hatenablog.com/entry/20140613/1402671674), a fan analysis). Each live costume was reportedly **built as a model, not as a texture** ([Wikipedia JA: サンジゲン](https://ja.wikipedia.org/wiki/%E3%82%B5%E3%83%B3%E3%82%B8%E3%82%B2%E3%83%B3)). Full-CG productions use a split rule. On *ULTRAMAN*, ordinary scenes use the rigged setup with small tweaks, and **only big action scenes get cloth simulation** ([Anime!Anime!](https://animeanime.jp/article/2019/11/09/49548.html)). The economics explain why. Hand-drawing frilly skirts on nine dancing idols is prohibitively expensive, while a modeled costume can be reused in every live scene of a franchise.

## Computer vision can find every character, but no model yet labels garments in screentone

### The data gap is garment-level labels on black-and-white pages

Public manga datasets stop at the character level. **Manga109** has more than 500k annotations covering frames, text, faces and bodies ([arXiv 2005.04425](https://arxiv.org/pdf/2005.04425)). It is licensed for academic use only, but **87 of its 109 volumes** are released as **Manga109-s** for commercial use. Conditions apply: no redistribution, published results must state that Manga109-s was used, and the images cannot be sold or turned into products ([Manga109 project](https://manga109.github.io/manga109-project-website/en/index.html)). **MangaSeg** (CVPR 2025) adds more than **700,000 segmentation annotations**, but its classes are panels, balloons, text, faces and character bodies, not clothing ([CVPR 2025](https://openaccess.thecvf.com/content/CVPR2025/html/Xie_Advancing_Manga_Analysis_Comprehensive_Segmentation_Annotations_for_the_Manga109_Dataset_CVPR_2025_paper.html)). The 2026 Manga109 revision corrected about 29,000 text annotations and added no clothing labels ([arXiv 2605.21182](https://arxiv.org/html/2605.21182v2)). Clothing labels exist only for color anime art. **RetriBooru** has 599,192 Danbooru images with clothes/face/figure masks ([arXiv 2312.02521](https://arxiv.org/pdf/2312.02521)). **Danbooru2023** has more than 5 million images with about 30 tags each; its metadata is MIT-licensed, but the images themselves are copyrighted ([HF](https://huggingface.co/datasets/nyanko7/danbooru2023)). A 2025 paper says plainly that no public dataset provides "full-body, fine-grained parsing granularity" for anime, and that existing segmenters show a clear domain gap ([arXiv 2508.06032](https://arxiv.org/html/2508.06032)). The practical workaround is to synthesize training pairs: take color anime parsing data, run line extraction, add synthetic screentone, and the label maps carry over unchanged.

### Page parsing and identity: Magi, with a clothing blind spot

The Magi family from Oxford VGG is the de facto open baseline. Magi v1 (CVPR 2024) detects panels, characters and text, orders panels and clusters characters. Magiv2 names characters across a chapter using a bank of reference images ([arXiv 2408.00298](https://arxiv.org/abs/2408.00298)). Magiv3 unifies detection, association, OCR and caption grounding ([arXiv 2503.23344](https://arxiv.org/html/2503.23344)). Its license needs care. One survey of the repository records MIT, but the README says the models and datasets are "available for academic research purposes only" ([magi GitHub](https://github.com/ragavsachdeva/magi)). A commercial pipeline should retrain a detector on Manga109-s plus MangaSeg rather than ship Magi weights. Re-identification research shows the clothing problem directly: different characters can have similar faces, "but different clothes often constitute the signatures of characters," and those signatures "usually do not change within the same manga book" ([arXiv 2204.04621](https://arxiv.org/pdf/2204.04621)). Re-ID that relies on clothing will therefore split or merge identities when a character changes clothes. No published benchmark covers costume-change re-ID in manga. The design that follows is to separate an **identity embedding** (face, hair) from an **outfit embedding** (features from the garment-masked region), then cluster outfits within each identity.

### Segmentation, screentone and tagging

Photo-trained parsers degrade on anime. SegFormer-B2 fine-tuned on ATR reports mIoU between **0.69 and 0.778** depending on the evaluation split, on photos ([HF card](https://huggingface.co/mattmdjaga/segformer_b2_clothes)), and its ATR-derived weights are non-commercial. **SAM 3** (Meta, November 2025) adds text-prompted concept segmentation trained on 4M concept labels, and its SA-Co benchmark spans **270K concepts** ([Roboflow](https://blog.roboflow.com/what-is-sam3/)). Prompts like "sailor collar" or "pleated skirt" are therefore possible, but no published evaluation on screentoned line art exists. The strongest anime-specific option is **See-through** (SIGGRAPH 2026, Apache-2.0). It decomposes one color illustration into **up to 23 inpainted RGBA semantic layers**, including clothing and accessories, with drawing order, exports a PSD, and has low-VRAM variants for 8–12 GB GPUs ([GitHub](https://github.com/shitagaki-lab/see-through); [arXiv 2602.03749](https://arxiv.org/abs/2602.03749)). It does not address grayscale input, so on manga it should run after colorization.

Screentone is signal as well as noise. Manga Rescreening research observes that artists "commonly use the same type of screentone with intensity variation to fill the same semantic region," and it segments pages by clustering an interpretable ScreenVAE tone representation ([arXiv 2306.04114](https://ar5iv.labs.arxiv.org/html/2306.04114)). That makes tone clusters a useful prior for snapping garment masks: a dark jacket versus a light shirt. Preprocessing has mature tools. Manga Restoration (CVPR 2021) recovers degraded scans ([arXiv 2105.06830](https://arxiv.org/abs/2105.06830)), and MangaLineExtraction removes tones to produce clean structural lines ([GitHub](https://github.com/ljsabc/MangaLineExtraction_PyTorch)). For attributes, Danbooru-trained taggers give garment vocabularies such as `serafuku` and `pleated_skirt`. **wd-eva02-large-tagger-v3** is Apache-2.0 ([HF](https://huggingface.co/SmilingWolf/wd-eva02-large-tagger-v3)), and **camie-tagger** covers **70,527 tags at 67.3% micro-F1** ([PromptLayer](https://www.promptlayer.com/models/camie-tagger/)). On black-and-white input, color tags like `blue_skirt` are hallucinated and should be dropped. The remaining shape and type tags can generate SAM 3 prompts automatically.

### Reference colorization is where outfit color gets decided

Colorization moved to multi-reference diffusion transformers in 2024–2026, and two systems explicitly target attire consistency across sequences. **ColorFlow** (Tencent ARC) retrieves color references per panel to avoid "identity drift" in hair and attire, but it carries an Academic license ([project](https://zhuang2002.github.io/ColorFlow/)). **Cobra** (SIGGRAPH 2025) conditions on **more than 200 references** through a causal sparse DiT and released weights under **Apache-2.0** ([GitHub](https://github.com/zhuang2002/Cobra)), which makes it the most deployable option commercially. **MangaNinja** (CVPR 2025 Highlight) adds point control, so a human can pin "reference jacket → target jacket" correspondences. It runs at 512×512 on SD 1.5 and is licensed **CC BY-NC 4.0** ([GitHub](https://github.com/ali-vilab/MangaNinjia)). **MangaDiT** reports PSNR **27.8 versus MangaNinja's 13.9** ([arXiv 2508.09709](https://arxiv.org/pdf/2508.09709)). These are the authors' own numbers, and a PSNR of 13.9 suggests a mismatched evaluation setting. For auditable costume color, the most relevant work models studio practice directly. **Paint Bucket Colorization Using Anime Character Color Design Sheets** fills line-art segments with exact RGB values from the sheet, and it notes that reference-based diffusion methods "often fail to accurately assign specific colors to each region" ([arXiv 2410.19424](https://arxiv.org/html/2410.19424v1)). DACoN extends paint-bucket matching to any number of references using DINO features ([ICCV 2025](https://openaccess.thecvf.com/content/ICCV2025/papers/Nagata_DACoN_DINO_for_Anime_Paint_Bucket_Colorization_with_Any_Number_ICCV_2025_paper.pdf)).

For animation-ready assets, **StdGEN** (CVPR 2025) generates 3D anime characters with **separate body, clothes and hair meshes in about 3 minutes** from one image ([GitHub](https://github.com/hyz317/StdGEN)). Sewing-pattern recovery methods (SewFormer, GarmentDiffusion) exist but have not been shown to work on anime or manga ([arXiv 2311.04218](https://arxiv.org/abs/2311.04218)). Every stage above expects color input, so the order is: manga → clean and colorize → decompose or reconstruct.

| Stage | Best 2026 component | License for commercial use |
|---|---|---|
| Restore / line extraction | Manga Restoration; MangaLineExtraction | Unconfirmed |
| Panels, characters, identity | Magi v1–v3 | Models academic-only; retrain on Manga109-s + MangaSeg |
| Character masks | SkyTNT anime-segmentation; MangaSeg-trained segmenter | Apache-2.0 / Manga109-s terms |
| Garment masks | SAM 3 text prompts + screentone clustering | Meta SAM license (verify) |
| Garment tags | wd-eva02-large-v3; camie-tagger | Apache-2.0 (WD) |
| Outfit clustering | DINOv2/SigLIP on garment crops + tag vector | No published method; build in-house |
| Colorization | Cobra; paint-bucket (BasicPBC/DACoN); ColorFlow, MangaNinja for research | Cobra Apache-2.0; ColorFlow Academic; MangaNinja CC BY-NC |
| 2.5D / 3D assets | See-through; StdGEN | Apache-2.0 / unverified |

## Generative models hold outfits only when line art and palettes pin them down

### Image consistency: trained adapters versus in-context editors

Two families compete. **Character and outfit LoRAs** on an anime base model, combined with a line-art ControlNet, remain the most controllable way to reproduce an exact costume. They suffer from "outfit bleed," though: practitioners training separate folders per outfit, each with its own trigger tag, still report bleedthrough ([hollowstrawberry guide](https://huggingface.co/hollowstrawberry/stable-diffusion-guide/discussions/7)). Generic uncolored tags have to be pruned to stop "colour bleeding from custom outfits onto the main outfit" ([lora-dataset-studio](https://github.com/perfectgf/lora-dataset-studio/blob/v1/docs/DATASET_GUIDE.md)). **Zero-shot multi-reference editors** have improved quickly. FLUX.2 accepts **up to 10 reference images** ([BFL](https://bfl.ai/blog/flux-2)). Qwen-Image-Edit-2509 works best with 1–3 inputs and accepts ControlNet maps natively ([HF](https://huggingface.co/Qwen/Qwen-Image-Edit-2509)). Nano Banana Pro takes **up to 14 references** ([Scenario](https://help.scenario.com/articles/7568607761-gemini-image-models-nano-banana-family)). Clothing is still the weak point. In an informal 27-case comparison, Gemini failed the "clothing extraction" test ([HF community blog](https://huggingface.co/blog/MonsterMMORPG/nano-banana-gemini-25-flash-image-full-tutorial)). FLUX.1 Kontext's headline consistency figure, **AuraFace ≈0.908**, measures faces, not costumes ([arXiv 2506.15742](https://arxiv.org/html/2506.15742v2)). IP-Adapter-Plus carries palette and silhouette but "fine-grained details… are usually not copied correctly" ([Stable Diffusion Art](https://stable-diffusion-art.com/ip-adapter/)). AnimeAdapter (May 2026) targets that gap with masked CLIP patch tokens trained on Danbooru-style data ([arXiv 2605.20237](https://arxiv.org/abs/2605.20237)). Based on the evidence, the ranking for costume fidelity is: an outfit-tagged LoRA trained on the work's own art with line-art control, then multi-reference editors, then anime appearance adapters, then generic IP-Adapter. Face-ID methods do not lock costumes at all.

Base-model choice is mostly a licensing question. **Animagine XL 4.0** (8.4M images, CreativeML OpenRAIL++-M) and **Neta Lumina** (Apache-2.0) are the cleaner open options ([HF Animagine](https://huggingface.co/cagliostrolab/animagine-xl-4.0); [HF Neta Lumina](https://huggingface.co/neta-art/Neta-Lumina/blob/main/README.md)). **NoobAI-XL** "prohibits any form of commercialization… of the model, derivative models, or model-generated products" ([model card](https://huggingface.co/Laxhar/noobai-XL-1.1/raw/main/README.md)). **Pony V7** requires separate commercial terms above US$1M in revenue ([HF](https://huggingface.co/purplesmartai/pony-v7-base)). The cheapest consistency lever is prompt-level. Write each costume as a fixed Danbooru tag block (e.g., `serafuku, red neckerchief, pleated skirt, black thighhighs`) and reuse it word for word in every shot.

### Virtual try-on is photographic and mostly non-commercial

Mainstream try-on models (**IDM-VTON, OOTDiffusion, CatVTON, StableVITON**) are all **CC BY-NC-SA 4.0** and trained on fashion photos ([IDM-VTON](https://github.com/yisol/IDM-VTON); [CatVTON](https://github.com/Zheng-Chong/CatVTON)). On flat-shaded anime they tend to add photoreal folds. OutfitAnyone is the only one that claims to generalize "from anime to in-the-wild images" ([arXiv 2407.16224](https://arxiv.org/html/2407.16224v1)), and it appears to be demo-only. In practice, multi-image editors such as Qwen-Image-Edit-2511, used for consistent outfit changes, have become the outfit-swap tool for anime characters ([NextDiffusion](https://www.nextdiffusion.ai/tutorials/consistent-outfit-changes-with-multi-qwen-image-edit-2511-in-comfyui)). Any2AnyTryon's "try-off" garment reconstruction is useful for pulling a clean garment reference out of a panel ([arXiv 2501.15891](https://arxiv.org/html/2501.15891v1)).

### Video: sketch-conditioned colorization beats free generation for costume fidelity

Two regimes exist. In **line-art-conditioned** video, artists' sketches fix the garment shape, so the remaining problem is color stability over time. AniDoc colorizes sketch sequences from a character design with explicit correspondence. It defaults to 14 frames and uses about 14 GB VRAM ([GitHub](https://github.com/yihao-meng/AniDoc)). ToonComposer merges in-betweening and colorization and needs as little as one sketch plus one colored frame ([arXiv 2508.10881](https://arxiv.org/abs/2508.10881)). **LongAnimation** (ICCV 2025) targets the core costume failure: prior methods handle fewer than 100 frames, and overlap fusion "fails to maintain color consistency over long sequences." It evaluates on **~500-frame clips** and beats ToonCrafter, LVCD and AniDoc ([arXiv 2507.01945](https://arxiv.org/abs/2507.01945)). InstanceAnimator places one reference per character on a canvas, which reduces color leakage between characters ([arXiv 2603.25357](https://arxiv.org/html/2603.25357)). In **free generation**, garments drift. AnimateDiff has an open issue titled "SD-XL cannot keep clothes color and hairstyle same" ([GitHub #226](https://github.com/guoyww/AnimateDiff/issues/226)). Wan2.2-Animate practitioners call reference inconsistency "the primary cause of character drift across clips" and reuse the exact same reference image for every clip ([guide](https://wan-animate.com/posts/wan-2-2-animate-motion-types-style-consistency-guide)). Kling's "Subject Binding" for clothing textures ([Kling](https://kling.ai/blog/kling-3-subject-binding-character-consistency)) and Vidu's 7-reference mode ([Vidu](https://www.vidu.com/ai-reference-to-video)) are vendor claims with no independent clothing benchmark. Typical failure modes are color bleeding between adjacent garments during large motion, drift over long shots, small accessories lost under occlusion, and "swimming" patterns.

For multiple characters and outfit changes, the working rule is to treat **each costume as its own reference asset**, chosen per shot from a costume schedule. Mixing costumes in one reference set invites bleed. DiffSensei (masked cross-attention for multiple characters in manga panels) ([arXiv 2412.07589](https://arxiv.org/abs/2412.07589)) and AnimeShooter (story-level character profiles across shots) ([arXiv 2506.03126](https://arxiv.org/abs/2506.03126)) are the closest research analogues. No published workflow handles costume changes in the middle of a shot, such as transformation sequences.

### Industry use stays assistive, and character design stays human

Every visible deployment keeps character and costume design human-made. *The Dog & the Boy* (Netflix/WIT, 2023) used image generation only for backgrounds and drew backlash ([Engadget](https://www.engadget.com/netflixs-dog-and-boy-anime-short-causes-outrage-for-incorporating-ai-generated-backgrounds-203035524.html)). *Twins Hinahima* (March 2025) claimed **more than 95% AI involvement** with humans finishing the work ([Anime News Network](https://www.animenewsnetwork.com/news/2025-02-28/frontier-works-kaka-creation-twins-hinahima-ai-anime-reveals-march-29-tv-debut/.221769)). Reports conflict on whether its character art was hand-drawn in Clip Studio Paint or AI-generated and then retouched, and reviewers noted "inconsistencies and uncanny looks in some frames" ([BusinessMirror](https://businessmirror.com.ph/2025/04/12/anime-using-ai-gets-mixed-reactions-after-first-episode-release/)). Toei's 2025 plan named **color specification and automatic color correction** and in-between generation as AI targets, then walked the plan back after backlash ([Gizmodo](https://gizmodo.com/animation-studio-toei-wants-to-use-ai-for-future-productions-2000603817); [GameRant](https://gamerant.com/toei-denies-using-ai-following-backlash/)). OLM Digital reportedly uses an automatic section-coloring tool that saves about **30% of work hours** in some cases, though this comes via an aggregator ([Anime Corner summary](https://animecorner.me/olm-digital-anime-studio-ai-artificial-intelligence/)). Studios have avoided letting models originate costume detail. Drift is one reason; labor and IP backlash is the other.

## Who draws the folds decides whether cloth looks like anime

### Hand-drawn and rigged 2D

Hand-drawn anime cloth follows drag, follow-through and overlapping action. Anime has its own term, **nabiki** (なびき), for cloth and hair flapping in wind. A skirt does not flare at the start of a run; it *drags* because of inertia ([Sakuga Wiki](https://sakuga.fandom.com/wiki/Nabiki)). There is no simulation involved, so the quality depends entirely on the animator's grasp of timing and fold logic, and the cost is paid in drawings. In cut-out and hybrid work, Toon Boom Harmony's **Envelope** deformer is explicitly meant for "hair, cloaks, shoulders, and chins" ([Harmony docs](https://docs.toonboom.com/help/harmony-20/premium/deformation/about-envelope-deformation.html)), and its Deformer-on-Deformer wizard puts a Curve deformer (the cape's spine) over an Envelope (the silhouette) ([Harmony 24 docs](https://docs.toonboom.com/help/harmony-24/premium/master-controller/about-deformer-on-deformer.html)). Rigged 2D uses pendulum physics. Live2D Cubism builds physics groups such as "swinging skirt," with input and output parameters and settings for reaction, amplitude, weight and convergence ([Live2D manual](https://docs.live2d.com/4.2/en/cubism-editor-manual/physics-operation/)). Spine 4.2 added physics constraints with strength and damping on bones for hair and clothing ([Esoteric](http://en.esotericsoftware.com/spine-physics-constraints)). The shared limit is that these tools deform art that was already drawn. They cannot invent new folds, show the back of a garment, or rotate convincingly without extra hand-drawn parts.

### AI in-betweening is weakest exactly where cloth moves most

Interpolation research moved from correspondence methods, such as AnimeInterp with its ATD-12K dataset of 12,000 triplets ([CVPR 2021](https://openaccess.thecvf.com/content/CVPR2021/papers/Siyao_Deep_Animation_Video_Interpolation_in_the_Wild_CVPR_2021_paper.pdf)) and AnimeInbet with visibility masks for occlusion ([ICCV 2023](https://openaccess.thecvf.com/content/ICCV2023/papers/Siyao_Deep_Geometrized_Cartoon_Line_Inbetweening_ICCV_2023_paper.pdf)), to video-diffusion generators. ToonCrafter outputs at most **512×320 and 16 frames**. Its own paper admits that compressed latents lose fine outlines and that its live-action prior can produce non-cartoon motion ([arXiv 2405.17933](https://arxiv.org/html/2405.17933v1)). ToonComposer reports that ToonCrafter-style methods "often need dense keyframes" for large motion ([arXiv 2508.10881](https://arxiv.org/pdf/2508.10881)). Skirt hems and capes are the worst case: large non-linear arcs, fold lines that appear and vanish, and constant occlusion of legs. No paper benchmarks interpolation on garments specifically. The credible 2026 use is small secondary motion, such as ribbon flutter or hem sway between close keys, with human cleanup.

### 3D cel-shaded cloth: simulate, then make it look drawn

The 3D pipeline authors garments as sewing patterns in Marvelous Designer, which added a **Toon Shader preview**; version 2026.0 shipped in April 2026 ([CG Channel](https://www.cgchannel.com/2026/04/clo-virtual-fashion-releases-marvelous-designer-2026-0/)). Offline work then simulates in Houdini Vellum or Blender, and Blender sims can be baked to shape keys for retiming ([RenderGuide](https://renderguide.com/blender-cloth-simulation-tutorial/)). Real-time work uses Unreal's **Chaos Dataflow Cloth Editor, production-ready and default in UE 5.8** ([Epic](https://dev.epicgames.com/community/learning/tutorials/Wb2V/unreal-engine-chaos-cloth-updates-5-8)), Magica Cloth 2's MeshCloth for skirts with collision ([MagicaSoft](https://magicasoft.jp/en/mc2_meshclothstartguide/)), or VRM SpringBone. SpringBone skirts often flip up when a character sits; one fix tool raises skirt gravity from 0 to about 0.3 ([VrmSkirtFixTool](https://note.com/dokokano_usagi/n/n7d49a37bc875?hl=en)). Rendering uses PSOFT Pencil+ 4, used by Toei, Shirogumi and on *Evangelion 3.0 (-46h)* ([CG Channel](https://www.cgchannel.com/2023/10/psoft-releases-pencil-4-line-for-blender/)), or Unity Toon Shader.

Raw simulation reads as "CG": too many small wrinkles and continuous motion on ones. Productions that look drawn treat cloth as an animated *shape*. Arc System Works' *Guilty Gear Xrd* animates at about **15 fps**, edits vertex normals to remove small shadows, uses a vertex-color channel to force shading, and **swaps whole models per frame** for hair and cloth shapes ([BlenderNation](https://www.blendernation.com/2015/07/26/junya-c-motomura-behind-the-scenes-of-guilty-gear-xrd/); [GDC Vault](https://www.gdcvault.com/play/1022031/GuiltyGearXrd-s-Art-Style-The)). *Spider-Verse* kept cloth sims stable on twos by simulating a hidden "ghost in-between frame that the audience never sees" and removing it afterward ([Cartoon Brew](https://www.cartoonbrew.com/feature-film/if-its-not-broke-break-it-sony-imageworks-renegade-approach-to-spider-man-into-the-spider-verse-167321.html)). Studio Orange hand-draws close-ups on *Land of the Lustrous* and deliberately recreates 2D quirks such as variable frame rates ([ANN](https://www.animenewsnetwork.com/interest/2018-02-26/a-look-into-the-art-and-animation-of-land-of-the-lustrous/.128275)). On *Trigun Stampede*, CG modeling began **3–4 years before broadcast** ([ComicBook.com](https://comicbook.com/anime/news/trigun-stampede-team-interview-cg-animation-talk/)), which shows how high the up-front asset cost is. Neural cloth (HOOD, ContourCraft for multi-layer collisions) is fast but learns realistic wrinkles, and no stylized training data exists ([arXiv 2212.07242](https://arxiv.org/pdf/2212.07242); [ACM](https://dl.acm.org/doi/fullHtml/10.1145/3641519.3657408)). For turning model sheets into patterns, **ChatGarment** (CVPR 2025) takes images or sketches and outputs GarmentCode JSON sewing patterns ([CVPR](https://openaccess.thecvf.com/content/CVPR2025/papers/Bian_ChatGarment_Garment_Estimation_Generation_and_Editing_via_Large_Language_Models_CVPR_2025_paper.pdf)). Physically impossible anime ribbons and floating capes will still need human pattern work.

| Approach | Up-front cost | Per-shot cost | Anime authenticity | Main failure | Best fit |
|---|---|---|---|---|---|
| 2D hand-drawn | Low | High | Highest | Off-model folds; budget forces simpler costumes | TV/film hero cuts |
| 2D cut-out + deformers (Harmony) | Medium | Low–medium | Good for capes and ribbons | Rubbery, limited angles | Kids' TV, web |
| 2D rig + physics (Live2D, Spine) | Medium | Very low | Good within drawn range | No rotation or new folds | VTubers, games |
| AI in-betweening | Low + GPU | Low + cleanup | Variable | Detail loss, needs dense keys | Small secondary motion |
| 3D sim + toon shader | High | Medium | "CG" unless stylized | Wrinkle noise, unstable on twos | Crowds, idol lives, action |
| 3D hand-keyed / model swaps | High | Medium–high | Highest in 3D | Labor per frame | Signature 3DCG work |
| Real-time bone/mesh cloth | Medium | ~Zero | Generic jiggle | Skirt flips, clipping | Games, MMD/VRoid |
| Neural cloth | High (R&D) | Very low | Realistic, not stylized | No anime data | Research |

All ratings in this table are qualitative syntheses of the sources above. No reliable per-approach cost figures, such as hours per second of cloth animation, are published.

## A stage-gated pipeline built around an outfit bible

### Mirror the studio roles and gate every costume boundary with a human

No open or commercial system covers the full path from manga page to colored, animated, costume-consistent anime. The workable architecture mirrors traditional roles (settei, 色彩設計, genga, douga, shiage, 撮影), puts AI inside each role, and places a human approval gate at every boundary that affects costumes. Toei's own planned AI insertion points (layouts, color specification, in-betweens, backgrounds) map onto exactly these stages ([Gizmodo](https://gizmodo.com/animation-studio-toei-wants-to-use-ai-for-future-productions-2000603817)). The system of record should be a production tracker. **Kitsu** is open source, exposes all production data through a REST API and Python SDK, and integrates with Toon Boom Harmony and Blender ([CGWire](https://www.cg-wire.com/kitsu/)). Autodesk Flow Production Tracking (formerly ShotGrid) is the commercial alternative ([Autodesk](https://www.autodesk.com/products/flow-production-tracking/overview)). WIT Studio's April 2026 apology, after generative AI slipped into background cuts of the *Ascendance of a Bookworm* opening, was blamed on "inadequacies in its production management and inspection systems" ([ANN](https://www.animenewsnetwork.com/news/2026-04-10/wit-studio-apologizes-for-using-generative-ai-in-opening-sequence-of-ascendance-of-a-bookworm-part-/.236271)). Gates must therefore log provenance (who used what, and where), not just check pixels.

| # | Stage | Traditional role | AI components | Human gate |
|---|---|---|---|---|
| 1 | Ingest licensed scans, hash, rights metadata; keep color pages separate | — | Manga Restoration, MangaLineExtraction | — |
| 2 | Panels, characters, identity clusters | Script supervisor | Magi-class detector | A: confirm identities per chapter |
| 3 | Garment masks, tags, outfit-change candidates | Costume continuity | SAM 3 + screentone clusters, WD14, outfit embeddings | B: approve outfit IDs, ranges, states (jacket off, wet, torn) |
| 4 | Turnarounds / settei per outfit | Character / costume designer | AI drafts → redraw; See-through layers | C: designer sign-off = canonical reference |
| 5 | Palette per garment region × lighting | 色彩設計 | — (stored as data, not images) | D: color designer approval |
| 6 | Adapters | — | Outfit LoRAs / reference conditioning trained on approved sheets only | — |
| 7 | Keyframes | Genga | Line-art/pose ControlNet + outfit adapter | E: animation director |
| 8 | In-betweens and motion | Douga | ToonComposer, AniDoc, LongAnimation; Wan 2.2 for B-roll | — |
| 9 | Paint | Shiage | Paint-bucket snap to palette hex (BasicPBC/DACoN) | — |
| 10 | Composite | 撮影 | Layer separation, lighting | — |
| 11 | QA | 色検査 | Automated costume checks → review playlist | F: color checker final approval |

The most reliable way to enforce costume color is step 9. Snapping region fills to exact palette hex values keeps errors discrete and checkable, unlike diffusion shading, which can repaint small accessories. Every 14–16-frame video window must be conditioned on the same approved outfit sheet; otherwise colors drift at the seams between windows.

### The outfit bible schema

No standard schema exists for anime costume continuity, and no studio color-sheet format is published. The practical approach borrows from three sources. **Danbooru's hierarchical attire tag groups** supply the garment vocabulary ([Danbooru Tag Group:Attire](https://danbooru.donmai.us/wiki_pages/tag_group:attire)). **VRM 1.0 metadata** supplies rights fields such as `commercialUsage`, `allowRedistribution` and `modification`, with the most protective options as defaults ([vrm-specification](https://github.com/vrm-c/vrm-specification/blob/master/specification/VRMC_vrm-1.0/meta.ja.md)). Iro-shitei practice supplies normal, shadow and highlight palettes per lighting variant. A condensed version (a synthesis, not an existing standard):

```
Character { character_id, names{ja,en}, rights_holder, design_version }
Outfit    { outfit_id, character_id, variant_of, state_tags[jacket_off,wet,damaged],
            source_range{chapter/page start–end}, anime_range{episodes[],cuts[]},
            danbooru_tags[], garments[Garment], accessories[Garment],
            model_sheets[{uri, view, approved_by, approved_at}],
            reference_crops[{page, panel, bbox, mask_uri, hash}],
            adapters[{type, base_model, base_model_license, uri, version,
                      trigger_token, train_set_hashes[]}],
            approval_status, version, provenance{ai_assisted, tools[]} }
Garment   { garment_id, slot(head|top|outer|bottom|legwear|footwear|accessory),
            danbooru_tag, layer_order, material,
            pattern{type: solid|stripe|plaid|print|emblem, scale, motif_uri},
            colors[{region, normal_hex, shadow_hex, shadow2_hex, highlight_hex,
                    lab_values, lighting_variant(day|evening|night|indoor), approved}],
            line_color_hex, accessory_count }
SceneOverride { cut_id, outfit_id, lighting_variant, deltas }
Rights    { source_work, contract_id, training_permitted, commercial_permitted, credit }
```

Store Lab values alongside hex so ΔE checks are cheap to compute. Never overwrite an approved palette; add a new version with a change reason. Record `base_model_license` on every adapter so a license audit is a single query. Accessory counts (pins, ribbons, earrings) deserve explicit fields because small items are what generative models drop first.

## Video compute costs hundreds of dollars per episode, and human review costs more

### Compute estimates

LoRA training is cheap. An SDXL character LoRA takes about **1,500–2,500 steps on 10–24 GB of VRAM**, and fal.ai's hosted FLUX.2 [dev] trainer charges **$0.008 per step (~$8 per 1,000-step run)** ([bestaiweb](https://www.bestaiweb.ai/how-to-train-a-custom-lora-for-flux-and-sdxl-with-kohya-ss-ai-toolkit-and-fal-ai-in-2026/); [localaimaster](https://localaimaster.com/blog/image-lora-training-local-guide)). Video dominates. Wan 2.1/2.2 14B takes about **10–12 H100-minutes per 5-second 720p clip** and needs 65–80 GB of VRAM. These figures come via secondary summaries ([Spheron](https://www.spheron.network/blog/ai-video-generation-gpu-guide/)). The TI2V-5B model makes a 5-second 720p clip on a 24 GB RTX 4090 in under about 9 minutes ([Wan2.2 README](https://github.com/Wan-Video/Wan2.2)). GPU prices in 2026: RunPod H100 at about **$2.69–3.49/hr**, L40S at about $0.79–1.09/hr ([RunPod](https://www.runpod.io/articles/comparison/choosing-gpus)), and AWS p5 at about **$6.88 per H100-hour** ([GMI Cloud](https://www.gmicloud.ai/en/blog/aws-p5-h100-pricing)).

| Item | Calculation (order of magnitude) | Cost |
|---|---|---|
| One take of a 22-min episode | 1,320 s ÷ 5 s = 264 clips × ~11 H100-min ≈ 48 H100-hr | ~$130–170 RunPod |
| 3–5 takes per shot | ≈145–240 H100-hr | **~$390–840 RunPod; ~$1,000–1,650 AWS** |
| Outfit LoRAs per series | 10 characters × 3–5 outfits = 30–50 × ~$8–15 | ~$250–750 |
| Traditional episode | ~3,000 drawings | **$160k–320k; >$500k high-profile** ([CreativeFreaks](https://creativefreaks.net/en/how-much-does-it-cost-to-produce-japanese-anime-episode-budget-and-cost-optimization-guide-2/)) |

These are estimates from the cited figures, and the baseline budget sources are of mixed quality. The conclusion is solid anyway: GPU spend is a rounding error, and the budget question is how many reviewer-minutes each cut needs. Note also that 3,000–4,000 drawings over 1,320 seconds works out to roughly 2–3 unique drawings per second, consistent with limited animation on twos and threes. AI output at 24 fps should be thinned to the show's timing chart, or it will not read as anime. A sensible setup is one or two local RTX 4090/5090 machines for interactive ComfyUI work by artists, burst cloud H100s for batch video, S3-compatible storage, Postgres for the bible, and Kitsu for tracking.

### Measuring costume consistency

No anime-specific costume-consistency metric or benchmark exists, and the standard identity metrics measure whole characters or faces. The best-supported practice comes from virtual try-on research. Mask out the garment, then compute **M-DINO** (sensitive to fine local structure) and **M-CLIP-I** (category-level) similarity against the reference garment ([OmniTry, arXiv 2508.13632](https://arxiv.org/pdf/2508.13632)). For video, add **VGID**, which measures garment texture and structure preservation ([arXiv 2511.18957](https://arxiv.org/html/2511.18957)), and DreamSim for perceptual similarity ([arXiv 2412.07750](https://arxiv.org/html/2412.07750v1)). DINO features already power anime paint-bucket matching ([DACoN](https://openaccess.thecvf.com/content/ICCV2025/papers/Nagata_DACoN_DINO_for_Anime_Paint_Bucket_Colorization_with_Any_Number_ICCV_2025_paper.pdf)), so they are suited to flat-color anime.

A per-cut QA suite logged to the tracker should run six checks:

1. **Garment presence:** diff WD14 tags on the character crop against the outfit's `danbooru_tags` to flag a missing neckerchief or an extra jacket.
2. **Palette fidelity:** compute CIEDE2000 ΔE between each region's median Lab value and the approved normal, shadow and highlight values. A starting threshold of around ΔE < 3 for flat cel regions is a suggestion that needs calibration.
3. **Similarity:** DINOv2 and DreamSim similarity of garment crops against model-sheet crops, tracked per frame to catch drift inside windows and at window seams.
4. **Patterns:** template matching for emblems and plaid repeats.
5. **Flicker:** frame-to-frame variance in region color.
6. **Accessories:** a detector or VLM checklist against `accessory_count`.

Ground-truth color exists only on the color design sheet, never in the manga. Paper metrics computed against existing anime frames are therefore proxies. Humans stay the final gate: the color checker reviews flagged cuts, a 10% sample of unflagged cuts measures how many errors slip through, and the animation director signs off.

## Licenses and adaptation contracts decide what can ship

### Law: Japan's training exemption does not cover imitating a specific work

Japan's Article 30-4 allows AI training on copyrighted works for non-"enjoyment" purposes, as long as outputs do not reproduce protected expression ([IBA](https://www.ibanet.org/japan-emerging-framework-ai-legislation-guidelines)). It does not apply where an enjoyment purpose coexists with the analytical one ([Clifford Chance](https://www.cliffordchance.com/insights/resources/blogs/talking-tech/en/articles/2023/10/Japanese-Law-Issues-Surrounding-Generative-AI.html)). The IBA's summary states that "if models are fine tuned (eg, via LoRA)… to imitate specific styles, the exemption no longer applies" ([IBA](https://www.ibanet.org/japan-emerging-framework-ai-legislation-guidelines)). The Agency for Cultural Affairs' own framing reportedly targets intent to output a specific work's expression rather than style in general, and its primary PDF could not be checked. Either way, a per-outfit LoRA exists precisely to reproduce a specific character's look. A manga-to-anime project therefore needs an **adaptation license from the rights holder with an explicit AI-training clause**, not a reliance on 30-4. The industry's stance makes this non-negotiable. CODA, whose members include Shueisha, Toei, Kadokawa, Aniplex and Studio Ghibli, told OpenAI on **October 27, 2025** that prior permission is required and that opting out after the fact does not cure liability ([AUTOMATON](https://automaton-media.com/en/news/sonys-aniplex-bandai-namco-and-other-japanese-publishers-demand-end-to-unauthorized-training-of-openais-sora2-through-coda/)). Creator sentiment is hostile. In a survey of about 25,000 Japanese creators, **~89%** saw generative AI as a significant threat and **12%** reported lost income ([Japan Times](https://www.japantimes.co.jp/news/2026/01/21/japan/society/ai-creatives-survey/)). Elsewhere, the US Copyright Office treats training as a fact-specific fair-use question and favors voluntary licensing ([Jones Day](https://www.jonesday.com/en/insights/2025/05/us-copyright-office-issues-guidance-on-generative-ai-training)). The **EU AI Act Article 50** labeling duties became enforceable on **August 2, 2026**, backed by a final Code of Practice published June 10, 2026 ([artificialintelligenceact.eu](https://artificialintelligenceact.eu/article/50/); [Paul Weiss](https://www.paulweiss.com/insights/client-memos/eu-finalises-transparency-rules-for-ai-generated-content)). EU distribution therefore needs machine-readable marking and disclosure. Article 50's narrower provisions for evidently artistic works were not researched.

### Model licenses: many of the best clothing tools are non-commercial

| Component | License | Production verdict |
|---|---|---|
| Wan 2.2 (incl. Animate) | Apache-2.0; "we claim no rights over… generated contents" ([GitHub](https://github.com/Wan-Video/Wan2.2)) | Usable; later Wan 2.5+ flagships are API-only |
| Cobra, See-through, Qwen-Image, AnimeColor, LongAnimation, ToonCrafter (code) | Apache-2.0 | Usable; check base-model weights (e.g., CogVideoX for LongAnimation); ToonCrafter calls itself "research exploration" |
| AniDoc, LVCD | Apache badge / no root LICENSE; built on Stable Video Diffusion | Unclear; base-model license governs |
| Magi | Repo reportedly MIT, but models and datasets "academic research only" ([GitHub](https://github.com/ragavsachdeva/magi)) | Retrain for commercial use |
| MangaNinja | CC BY-NC 4.0 ([GitHub](https://github.com/ali-vilab/MangaNinjia)) | Prototype only |
| ColorFlow | Academic Software Licence ([HF](https://huggingface.co/spaces/TencentARC/ColorFlow/blob/main/LICENSE)) | Prototype only |
| IDM-VTON, OOTDiffusion, CatVTON, StableVITON | CC BY-NC-SA 4.0 | Prototype only |
| FLUX.1 [dev] / FLUX.2 [dev] | Non-commercial weights; paid commercial license available ([BFL](https://bfl.ai/blog/flux-2); [HF discussion](https://huggingface.co/black-forest-labs/FLUX.1-dev/discussions/136)) | Buy license or use API |
| NoobAI-XL | Forbids commercialization including outputs ([model card](https://huggingface.co/Laxhar/noobai-XL-1.1/raw/main/README.md)) | Avoid |
| HunyuanVideo family | Territory excludes EU, UK, South Korea ([HF LICENSE](https://huggingface.co/tencent/HunyuanVideo/blob/main/LICENSE)) | Avoid for global work |
| Index-AniSora V3 | Apache-2.0 plus usage restrictions ([GIGAZINE](https://gigazine.net/gsc_news/en/20250519-anisora/)) | Read restrictions |
| Manga109-s / Sakuga-42M | Restricted commercial / academic-only ([Sakuga](https://github.com/KytraScript/SakugaDataset)) | Training-data caution |

Celsys offers a useful precedent. It withdrew a Stable Diffusion palette from Clip Studio Paint three days after announcing it, and it positions its Colorize feature as trained on consented creator data ([ARTnews](https://www.artnews.com/art-news/news/clip-studio-paint-ai-backlash-1234649301/)). Training adapters only on licensed, approved model sheets, and crediting the human designers, is both the legal and the reputational path.

## A twelve-week MVP builds a continuity tool, not an anime generator

A 3–5 person team should not try to generate finished anime from manga panels. It should build the part that is reliable and missing from the market: **costume continuity and color enforcement**. The tool would ingest a licensed volume, detect characters and outfit changes, produce a reviewable outfit bible with palettes, and enforce those palettes on frames whether humans or AI made them. The stack is a Python/FastAPI backend, Postgres with JSONB for the bible, S3 or MinIO for assets, ComfyUI workers (a local 4090/5090 plus RunPod burst capacity), Kitsu for tasks and review, and a small React review UI showing page, panel, crop and outfit with approve, split and merge controls.

**Weeks 1–3** cover ingest and detection. Restore scans, run a Magi-class detector for panels, identities and reading order, and generate SAM 3 garment masks and WD14 tags for each appearance. Raise outfit-change candidates from tag-set differences and jumps in DINO embeddings, and stand up the reviewer UI. **Weeks 4–6** cover the bible and color. Build outfit records, model-sheet upload, a palette editor (normal, shadow and highlight hex values plus lighting variants), and a per-episode costume continuity report. Train outfit LoRAs only on approved sheets, using a license-checked base such as Animagine XL 4.0 or Neta Lumina, with provenance recorded. **Weeks 7–10** cover generation and enforcement. Draft keyframes with line-art/pose ControlNet plus the outfit adapter, snap regions to exact bible hex values paint-bucket style, test Cobra for colorizing manga panels, and run AniDoc or ToonComposer on short line sequences, with a few Wan 2.2 I2V shots. **Weeks 11–12** cover QA and a pilot. Run the automated suite (tag diff, ΔE, DINO/DreamSim, flicker), post flagged frames to Kitsu playlists, and pilot one episode's worth of cuts with color-checker sign-off and a per-cut AI-use log. Four metrics measure success: outfit-change detection recall against human labels, palette ΔE pass rate, reviewer minutes per cut, and the rate at which costume errors escape to final. Before day one, the team needs a signed adaptation license with a training clause, a model-license register, an EU Article 50 labeling plan if distributing in the EU, and a published AI-use policy for staff and creators. The timeline and reviewer-effort figures are judgment calls; no public case study documents a small team shipping such a tool.

## Conclusion

The research points to an unexpected conclusion. The best AI pipeline for manga-to-anime clothing is not the most generative one. It reproduces the decades-old studio contract: human designers fix the garment's lines in settei and its colors in a palette, and everything downstream is checked against that record. The places where AI is already strong (multi-reference colorization, palette-snapping paint-bucket fill, short sketch-conditioned video) all work by *inheriting* geometry rather than inventing it. The places where it fails (free video generation, outfit swaps with photographic try-on models, interpolating swirling capes) are exactly where the model has to author fold shapes or costume detail itself. The real leverage is therefore a well-structured outfit bible plus automated QA, not a better generator. An outfit bible with Lab-valued palettes, accessory counts, license fields and provenance turns costume consistency from an artistic judgment into a set of queryable, testable constraints.

Three open problems mark the frontier and are good targets for anyone building in this space: a garment-labeled dataset for screentoned manga (probably bootstrapped by converting color anime parsing data to manga style), outfit-aware re-identification that separates "who" from "what they wear," and an anime-specific costume-consistency benchmark that goes beyond face metrics. Until those exist, the competitive advantage lies less in model choice than in governance. WIT's 2026 apology and Toei's retreat show that a studio able to prove, cut by cut, which costume came from which approved sheet and which tool touched it will be the one allowed to use AI at all.
