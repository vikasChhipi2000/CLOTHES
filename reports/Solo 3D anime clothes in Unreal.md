# Dress Your Anime Body for Unreal 5.8

For one person who already has an unclothed anime body, the best quality for the least setup comes from a hybrid where each tool does only the part it is best at. First, an AI image editor that runs locally and needs no training (**Qwen-Image-Edit-2511 in ComfyUI**) "dresses" A-pose renders of *your own* body in the manga outfit, which gives you a turnaround that already matches your proportions. You then build the clothes as real, separate meshes: **Marvelous Designer 2026.1** for anything that hangs away from the body (skirts, jackets, coats), **Blender** for tight pieces and for all cleanup, and AI image-to-3D only for rigid accessories. You paint flat anime colors, copy skin weights from the body with the free **Robust Weight Transfer** add-on, and bring everything into **Unreal Engine 5.8**. There, skirts and ribbons swing on the free **KawaiiPhysics** plugin, cloth is cel-shaded with inverted-hull outlines and painted fold lines, and outfits swap through one Data Asset per outfit. This pipeline works because it borrows the two habits anime studios and anime games depend on. The costume is fixed and simplified on a human-approved model sheet before any 3D work starts, and folds and cloth motion are *authored* (bone chains, painted shadows) rather than left to a realistic simulator. A fully free route exists if you swap Marvelous Designer for Blender modeling. Pure AI garment generation and sewing-pattern AI are not yet the easy path to clean, deformable anime clothing. Plan on roughly **1–2 weeks for your first outfit**, which includes one-time setup, and **1–3 days for each outfit after that** (estimate). Nothing here involves training a model: downloading a pretrained community LoRA or add-on and running it is inference, not training.

**How to read the flags.** "(snippet)" means the source page was blocked during research and the claim rests only on a search-engine summary. "(vendor)" means the source sells the product it describes. "(estimate)" means practitioner judgment with no published source. Everything else was read from the cited page.

## The golden path at a glance

The table is the whole pipeline. The stage sections below give the exact steps and the traps for each one.

| # | Stage | Main tool (version) | Cost | One-time setup effort |
|---|---|---|---|---|
| 1 | Reference prep on your body | Blender 5.x renders + ComfyUI + Qwen-Image-Edit-2511 (+ optional turnaround LoRA) | Free (local) | 1–3 h (estimate) |
| 2 | Build garments | Marvelous Designer 2026.1 for loose pieces; Blender for tight pieces; Hunyuan3D 2.1 / Tripo only for rigid accessories | MD paid; rest free or cloud credits | MD: install + learning curve; Blender: none |
| 3 | Flat HD texturing | Blender Texture Paint or Substance Painter; StableProjectorz for projection | Free / paid | 1–2 h |
| 4 | Skinning to your rig | Robust Weight Transfer (Blender) + skirt/coat bone chains added once | Free | 2–4 h once |
| 5 | Import and attach in UE | UE 5.8, body Skeleton reused, Leader Pose / Copy Pose | Free for personal use | 1–2 h |
| 6 | Skirt and cloth physics | KawaiiPhysics (UE 5.3–5.8); Chaos Cloth only for long capes and dresses | Free | 1–2 days first time (estimate) |
| 7 | Cel shading and outlines | Substrate Toon (5.8, experimental) or MToon/free plugin + inverted-hull outlines | Free (paid shaders optional) | 1–3 days first time (estimate) |
| 8 | Animation and HD render | Mixamo / ActorCore / video mocap → "Retarget Animations" → Sequencer → Movie Render Queue | Free to ~$12/month | 0.5–1 day |
| 9 | Outfit switching in game | Data Asset per outfit; Mutable later if parts multiply | Free | ~1 day (estimate) |

## Nine stages from manga panel to playable outfit

### Stage 1: Turn manga panels into a turnaround drawn on your own body

**Tools:** Blender 5.x (free) for the body renders. ComfyUI (free) running **Qwen-Image-Edit-2511**, whose code is Apache-2.0 ([QwenLM/Qwen-Image](https://github.com/QwenLM/Qwen-Image)). Optionally the community **Character Turnaround Sheet LoRA** for 2511/2509 ([Civitai 2149265](https://civitai.com/models/2149265/character-turnaround-sheet-qwen-image-edit-25112509)); it is pretrained, so downloading it is not training, but its license was not verified. The cloud alternative is **Nano Banana Pro** (paid), which takes up to 14 reference images and outputs up to 4K ([Scenario](https://help.scenario.com/articles/7568607761-gemini-image-models-nano-banana-family)). **Setup:** 1–3 hours to install ComfyUI and download the models (estimate). On a 24GB card the 2511 model is typically run in fp8 or a quantized build (estimate).

**What to do.** Render your body in Blender in A-pose with an orthographic camera, in front, side and back views. Use flat or unlit shading on a white background at about 600×1080, which is the LoRA's recommended input ([Civitai](https://civitai.com/models/2149265/character-turnaround-sheet-qwen-image-edit-25112509)). Qwen-Image-Edit-2511 accepts up to three reference images and is tuned to reduce drift across edits ([RunComfy](https://www.runcomfy.com/comfyui-workflows/qwen-image-edit-2511-in-comfyui-precision-instruction-editing)), and community workflows already use it for "consistent outfit changes" from a character image plus a garment image ([NextDiffusion](https://www.nextdiffusion.ai/tutorials/consistent-outfit-changes-with-multi-qwen-image-edit-2511-in-comfyui)). Feed it (1) the front body render, (2) the clearest manga panel of the outfit and (3) a color reference, with a prompt like: *"Dress the character in image 1 in the exact outfit from image 2; keep image 1's body proportions, pose, camera and framing unchanged; colors: [list]; flat cel colors, even lighting, no cast shadows, plain white background."* Repeat for the side and back views, feeding the front result back in as a reference. Also generate an outfit-only "flat lay" of each garment (front and back), which you will later use for pattern shapes and emblem decals. Exactly this "render a posed figure, dress it with AI, then build the garment" workflow is described by hobbyists on the Daz3D forum ([Daz3D forum](https://www.daz3d.com/forums/discussion/718631/do-you-know-an-ai-to-create-cloth-and-outfit), snippet).

Prep the manga side first. **MangaLineExtraction** strips screentone and leaves clean structural lines ([GitHub](https://github.com/ljsabc/MangaLineExtraction_PyTorch)); its license is unconfirmed. If the author never published color pages, you become the color designer. Studios map solid black (ベタ) to a dark color and adjust hues so characters don't blend together ([otonari-anime interview](https://otonari-anime.com/special/interview05.html)). **Cobra** (Apache-2.0, more than 200 references) can colorize the panels from whatever color references you have ([GitHub](https://github.com/zhuang2002/Cobra)). If your only reference is a dynamic action panel, **CharacterGen** converts a posed image into consistent A-pose multi-views; it was trained on 13,746 anime characters ([GitHub](https://github.com/zjp-shadow/CharacterGen)).

**Borrow from anime production: run a settei pass.** Before modeling anything, clean the AI output into a model sheet the way a studio would. A settei package includes front, side and back turnarounds, construction details for clothing and accessories, and a color sheet ([Animation Cel glossary](https://animation-cel.com/glossary/settei)). Designers deliberately cut lines, colors and ornaments while keeping the silhouette and overall impression ([Diamond Online](https://diamond.jp/articles/-/312739); [Creators Station](https://www.creators-station.jp/interview/curiousity/34834)). In your color sheet, give every garment region a base tone and a shadow tone; in studio settei, blue marks shadow areas on clothes ([Sakuga Blog](https://blog.sakugabooru.com/2017/07/21/the-pre-production-of-anime-3-design-work/)). This sheet becomes your single source of truth for the mesh, the texture and the shader ramps.

**Pitfalls.** The editor drifts proportions, so overlay each output on the original body render at 50% opacity and redo any view whose silhouette moved. No benchmark exists comparing Qwen, Nano Banana Pro, GPT-image and FLUX on preserving a supplied body. Small details such as emblem shapes and button or accessory counts drift: in one informal 27-case test, Gemini failed the "clothing extraction" case ([HF community blog](https://huggingface.co/blog/MonsterMMORPG/nano-banana-gemini-25-flash-image-full-tutorial)), so check them against the panels. The AI invents the back of the outfit, which manga rarely shows, so treat the back view as a design decision you make, not as truth. Always request even lighting, or shading gets baked into images you may later project as textures.

### Stage 2: Sew soft garments in Marvelous Designer, model tight ones in Blender

**Main route: Marvelous Designer 2026.1** (paid subscription; price and personal-license terms not verified). Version 2026.0 (April 2026) added glTF/VRM avatar import with blendshape support, the **3D Pencil** for drawing pattern outlines directly on the avatar, and a **Toon Shader** preview ([The Rookies](https://www.therookies.co/blog/headlines/marvelous-designer-2026); [Digital Production](https://digitalproduction.com/2026/04/15/marvelous-designer-2026-0-adds-3d-pencil-and-lacing/)). Version 2026.1 (August 2026) added **Brush Pinching** for painting folds freehand, and preserves OBJ UVs ([textalks](https://textalks.com/clo-virtual-fashion-has-released-marvelous-designer-2026-1/); [CG Channel](https://www.cgchannel.com/2026/08/clo-virtual-fashion-releases-marvelous-designer-2026-1/), snippet). MD has **no AI that generates sewing patterns from images**; its AI features generate textures and graphics. You trace the patterns yourself from the Stage 1 turnaround.

**What to do in MD.**

1. Export your body as FBX in A-pose (or VRM if it is a VRoid/VRM body) and import it as the avatar. Auto Fitting creates a fitting suit for humanoid FBX imports ([MD Auto Fitting](https://support.marvelousdesigner.com/hc/en-us/articles/47358335130649-Auto-Fitting), snippet).
2. Place the turnaround images as references and trace the patterns with the 2D tools or the 3D Pencil.
3. Simulate. Then make it read as anime: stiffer fabric presets, Brush Pinching for a few big authored folds, and the Toon Shader preview to judge the look.
4. Remesh with **Quad (Optimized)**. Pattern-based UVs come for free, though the vendor admits the auto-quad mesh may need manual cleanup ([MD guide](https://www.marvelousdesigner.com/explore/guide/best-3d-clothing-cloth-simulation-software-2026), vendor).
5. Export single-layer ("thin") garments in centimeters, A-pose, as FBX ([virtualfilmer](https://virtualfilmer.com/how-to-export-from-marvelous-designer-to-maya/)), then add thickness with Solidify in Blender.

CLO's **EveryWear** toolkit adds polygon reduction, auto-rigging, weight painting, UV packing and texture baking. MD has published a VRChat outfit tutorial built with it ([CLO EveryWear](https://connect.clo-set.com/everywear); [MD news](https://www.marvelousdesigner.com/support/news/view/b43d71d4ad08419d8f3cff28b8dbd2bc)). Whether it can target an arbitrary custom skeleton is unconfirmed.

**Blender route (free, best for tight outfits).** For shirts, leggings and bodysuits, select the body faces under the garment, duplicate and separate them, then add Shrinkwrap (about 2–5 mm offset) and Solidify (about 3–8 mm) (estimate). The garment inherits the body's deformation-friendly topology and UVs, so weight transfer becomes almost perfect. Poly-model skirts and coats from a cylinder or plane against the turnaround backdrop. Optionally run a short cloth sim, apply it, then sculpt **3–6 bold folds per garment** and delete the micro-wrinkles (estimate). For MD-style sewing inside Blender, **Simply Cloth Studio 2.0** (January 2026, paid) lets you draw clothing parts around a character and sew them, and it adds a geometry-nodes wrinkle system ([Digital Production](https://digitalproduction.com/2026/01/27/simply-cloth-studio-2-0-rebuilds-cloth-in-blender/)). **Garment Tool** is a paid 2D-pattern sewing add-on ([Gumroad](https://bartoszstyperek.gumroad.com/l/GarmentTool?ref=311)). Commercial anime outfit packs make good kitbash starting points to refit ([example pack](https://soheilkianfar.gumroad.com/l/WarriorOutfit-TojiAndMegumiFushiguro)). If your body is a VRoid/VRM model, **VRoid Studio** 2.x has a dress-up feature with the XWear format and auto-fitting ([VRoid notice](https://vroid.com/en/studio/notice/3rgHw1j8oCVkdWM0LKHBH6)), though no source documents which garment types it handles well.

**When to use AI-generated meshes.** Use them for rigid and semi-rigid pieces: armor, shoes, hats, belts, buckles, bags, bows and emblems. Never use them for skirts and coats, which need clean edge loops to deform. Image-to-3D models output fused, closed meshes. StdGEN's authors call monolithic meshes "practically useless" for game and animation pipelines ([StdGEN++ arXiv](https://arxiv.org/pdf/2601.07660), snippet), and a vendor's own guide says hero assets need full retopology before rigging ([Meshy](https://www.meshy.ai/tutorials/character-auto-rigging-workflow), vendor). The recipe:

1. Erase the body from a Stage 1 image, leaving only the accessory.
2. Generate the mesh locally or in the cloud. Local options are **Hunyuan3D 2.1**, which needs 10GB VRAM for shape, 21GB for paint and 29GB for both, so run the stages separately on 24GB ([GitHub](https://github.com/tencent-hunyuan/hunyuan3d-2.1)), and **TRELLIS.2** (MIT; at least 24GB, Linux only, a compile-heavy install) ([GitHub](https://github.com/microsoft/TRELLIS.2)). Cloud options are Tripo and Meshy (paid), which offer segmentation and quad output.
3. In Blender, cut open the closed shell, delete interior faces, conform it to the body, retopologize and UV.

To separate a dressed figure into parts, use **Hunyuan3D-Part P3-SAM**, which also has a ComfyUI wrapper ([GitHub](https://github.com/Tencent-Hunyuan/Hunyuan3D-Part)). **StdGEN** (Apache-2.0) is the only open anime model that outputs a separate clothing layer, but it builds its own body, so the clothes must be refit to yours ([GitHub](https://github.com/hyz317/StdGEN)). Skip sewing-pattern AI (ChatGarment, AIpparel, Dress-1-to-3) for now. It drapes on SMPL-X realistic bodies, is research code, and ChatGarment's own README warns of "incorrect lengths or widths" ([GitHub](https://github.com/biansy000/ChatGarment)).

**Borrow from anime production.** *Love Live!* built its live-performance costumes as real models, not textures ([Wikipedia JA: サンジゲン](https://ja.wikipedia.org/wiki/%E3%82%B5%E3%83%B3%E3%82%B8%E3%82%B2%E3%83%B3)), which is the case for doing the same. *Guilty Gear Xrd* edits vertex normals specifically to *remove* small shadows ([BlenderNation](https://www.blendernation.com/2015/07/26/junya-c-motomura-behind-the-scenes-of-guilty-gear-xrd/)). Fewer, larger folds are the anime look, while realistic MD micro-wrinkles are the "CG" look.

**Pitfalls.** Realistic micro-wrinkles left in. No source tested how MD Auto Fitting handles anime proportions (big head, thin limbs), so check the fit early. AI shells come out closed and with interior faces. Garments with zero thickness show backfaces in UE.

### Stage 3: Paint flat albedo at 2K and put the fold lines in the texture

**Tools:** Blender Texture Paint (free) or Substance Painter (paid; price not verified) for hand-painting. **StableProjectorz** (free, AGPL-3.0 since January 2026) projects Stable Diffusion/ComfyUI images onto your existing UVs from up to 6 user-placed views with per-view masks ([stableprojectorz.com](https://www.stableprojectorz.com/); [GitHub](https://github.com/IgorAherne/trellis-stable-projectorz)). **Hunyuan3D-Paint 2.1** can texture a mesh you supply, using about 21GB of VRAM, but it outputs PBR maps that you must flatten ([GitHub](https://github.com/tencent-hunyuan/hunyuan3d-2.1)). **Setup:** an hour or two (estimate).

**What to do.** Anime cloth needs flat albedo plus a toon shader, not PBR. Fill each color region flat from your Stage 1 color sheet; **2K is enough for flat color, and 4K is only worth it for fine emblems or lace** (estimate). Paint fold, seam and stitch lines into the albedo or into a separate UV-aligned line texture. Arc System Works stores this kind of art data in UVs, vertex colors and normals because it stays resolution-independent ([GG Xrd GDC PDF](https://www.ggxrd.com/Motomura_Junya_GuiltyGearXrd.pdf)). Apply emblems and prints as stencils taken from the flat-lay sheet. This is where 3D beats hand-drawn anime: a plaid texture follows every fold for free, whereas *Demon Slayer*'s animators reportedly count the haori checks by hand ([Togetter](https://togetter.com/li/1614752), medium confidence). Also prepare two vertex-color channels for Stage 7, one to push areas into shadow and one for outline width, plus smoothed normals for the outline hull. If you project the AI turnaround, posterize the result afterwards and hand-fix the seams.

**Pitfalls.** Soft gradients and baked lighting from AI images or cloud retexturing, which fight the cel shader. Projection seams at the side views. The AGPL license on StableProjectorz only matters if you redistribute modified code.

### Stage 4: Add swing bones once, then copy weights from the body

**Tools:** **Robust Weight Transfer** (free, GPL-3.0 Blender add-on based on Epic's SIGGRAPH Asia 2023 weight-inpainting paper) ([GitHub](https://github.com/sentfromspacevr/robust-weight-transfer); [paper](https://www.dgp.toronto.edu/~rinat/projects/RobustSkinWeightsTransfer/preprint.pdf)). Its README doesn't state which Blender versions it supports, so test it on your version first. Blender's built-in Data Transfer modifier is the fallback. **Auto-Rig Pro** ($25 Lite / $50 Full; supports Blender 2.93–5.1/5.2) helps add chains ([Superhive](https://superhivemarket.com/products/auto-rig-pro), snippet). If the body isn't rigged yet, **AccuRIG 2** is free and has an Unreal export preset ([Reallusion Magazine](https://magazine.reallusion.com/2025/07/30/accurig-2-vs-mixamo-smarter-auto-rigging-for-3d-animators/)). UE 5.4+ can also transfer weights in-engine with a Transfer Skin Weights dataflow node ([virtualfilmer](https://virtualfilmer.com/how-to-export-from-marvelous-designer-to-maya/), snippet).

**What to do.**

1. Add skirt, coat-tail, ribbon and hair bone chains to the *body* skeleton **once**, so every future outfit skins to the same skeleton. Use 6–12 radial chains of 3–5 bones for a skirt (estimate), and standardize names such as `skirt_front_01`.
2. Parent each garment to the armature and run Robust Weight Transfer from the body. It is built for the places where ordinary transfer fails: armpits, between the legs, skirts and loose sleeves ([3dxdev](https://3dxdev.com/assets/robust-weight-transfer-one-click-weights-for-blender/)).
3. Paint the skirt-chain weights as a gradient from the hip.
4. Pose test: sit, high kick, arms overhead. Fix the shoulders, crotch and collar.
5. Split the body mesh into sections (upper arms, torso, hips, thighs, shins) so Stage 5 can hide whatever an outfit covers (estimate).

**Pitfalls.** Inconsistent chain names across outfits break a shared Anim Blueprint. Layered garments (a jacket over a shirt) need the outer layer slightly inflated and given the same influences as the layer beneath.

### Stage 5: Import every garment onto the body's existing Skeleton asset

**Tools:** **UE 5.8** (shipped around June 2026; no 5.9 had shipped as of October 2026) ([Epic](https://www.unrealengine.com/news/unreal-engine-5-8-is-now-available)). If your body is VRM, use **VRM4U** (MIT). It imports MToon and spring bones and auto-builds the IK Rig and Retargeter; its latest release is v1.2026.09.12 ([GitHub](https://github.com/ruyo/VRM4U); [Releases](https://github.com/ruyo/VRM4U/releases)), but users report UE 5.7 animation and retarget problems ([Issue #558](https://github.com/ruyo/VRM4U/issues/558)). MD users can push garments with **LiveSync** ([CLO LiveSync](https://connect.clo-set.com/livesync)). Note the version trap: MD 2025.2 works with UE 5.6+, and **UE 5.5 is incompatible** with MD simulation data ([MD support](https://support.marvelousdesigner.com/hc/en-us/articles/47358145573401--Tips-Tricks-Discover-Better-Workflow-with-Marvelous-Designer-and-Unreal-Engine), snippet).

**What to do.** Import each garment FBX as a Skeletal Mesh and pick the body's Skeleton; new chain bones are added to it. In the character Blueprint, add each garment as a Skeletal Mesh component. Use **Set Leader Pose Component** for rigid followers (shoes, gloves, tight tops) ([yelzkizi](https://yelzkizi.org/clothes-for-metahuman/), snippet). For anything that swings, the simplest arrangement is to put all KawaiiPhysics nodes in the body's Anim Blueprint and keep garments as followers, which works because your chain names are standardized. Alternatively, give a garment its own small Anim Blueprint using **Copy Pose From Mesh** plus KawaiiPhysics. Followers driven by Leader Pose reportedly don't run their own Anim Blueprint graph; this is background knowledge, because Epic's docs were blocked. Hide covered body sections per outfit with a material opacity-mask parameter or by toggling sections ([UE forum](https://forums.unrealengine.com/t/do-video-games-hide-the-player-model-underneath-the-clothing/33335), snippet).

**Pitfalls.** Unit-scale mismatches; export in centimeters. The UE 5.6+ Chaos **Outfit Asset** can resize garments to new bodies, but **only in the editor, never at runtime** ([Epic tutorial](https://dev.epicgames.com/community/learning/tutorials/9Xjd/unreal-engine-chaos-cloth-outfit-asset-resizing-addendum), snippet).

### Stage 6: Swing skirts on KawaiiPhysics and save true cloth for hero shots

**Tools:** **KawaiiPhysics** (MIT; free on GitHub and Booth, plus a paid Fab listing with the same features; supports UE 5.3–5.8). It was made by an Epic Games Japan engineer specifically for anime-style hair and skirt motion ([GitHub](https://github.com/pafuhana1213/KawaiiPhysics); [Fab](https://www.fab.com/listings/f870c07e-0a02-4a78-a888-e52a22794572)). **Chaos Cloth**: in 5.8 the Dataflow Cloth Editor is "Production-Ready and is now the default cloth editor" ([UE forum](https://forums.unrealengine.com/t/tutorial-chaos-cloth-updates-5-8/2729420), snippet). **MD USD simulation data** imports into Chaos Cloth Assets on UE 5.6+ ([MD support](https://support.marvelousdesigner.com/hc/en-us/articles/52699135975705-Marvelous-Designer-to-MetaHuman-USD-Garment-Integration-Workflow), snippet).

**What to do.** Place the KawaiiPhysics node after the final pose in the AnimGraph with one root bone per chain ([Ida Faber docs](https://docs.idafaber3d.com/unreal-engine/physics), snippet). Add capsule colliders on the thighs and pelvis, and a plane or capsules on the shins for long skirts. Enable **BoneConstraint** to keep skirt bones apart. Use **SyncBone** from v1.20.0 (January 2026), which pushes the skirt with the leg and "solves issues like skirt clipping through legs" without a separate Control Rig ([Discussion #185](https://github.com/pafuhana1213/KawaiiPhysics/discussions/185)). The same release added world-space gravity, fixing the old gravity that was ineffective in slow motion, and an AnimNotifyState that overrides simulation strength per animation section. Use that to exaggerate on landings or freeze cloth on held poses. Reserve Chaos Cloth for long dresses, capes and wide sleeves: paint the waistband's max distance near zero and collide against fattened thigh and pelvis capsules (estimate). For a single hero close-up in a cinematic, an MD-simulated Alembic cache is render-only but gives the richest motion.

**Borrow from anime production and games.** Studios ration simulation. *Love Live!* hand-draws bust-ups and uses 3DCG for wide shots ([fan analysis](https://hokke-ookami.hatenablog.com/entry/20140613/1402671674)), and *ULTRAMAN* runs cloth simulation **only in big action scenes** ([Anime!Anime!](https://animeanime.jp/article/2019/11/09/49548.html)). Anime cloth should *drag*: a skirt does not flare at the start of a run but lags behind through inertia (nabiki) ([Sakuga Wiki](https://sakuga.fandom.com/wiki/Nabiki)). Tune spring response for overshoot rather than realism. The VRM community's standard fix for skirts that flip up when sitting is raising skirt gravity from 0 to about 0.3 ([VrmSkirtFixTool](https://note.com/dokokano_usagi/n/n7d49a37bc875?hl=en)), and the same idea carries over to KawaiiPhysics.

**Pitfalls.** With a Sequencer animation track, KawaiiPhysics may ignore play and stop; the workaround is to play the clip through the Anim Blueprint ([UE forum](https://forums.unrealengine.com/t/how-to-use-kawaii-physics-in-the-sequencer/2019655), snippet). Legs passing through long Chaos skirts is a long-standing complaint ([UE forum](https://forums.unrealengine.com/t/how-to-make-a-long-skirt-simulation-without-the-leg-going-through-it/222163)). Cloth "pops" on camera cuts, so reset the simulation on cuts (estimate).

### Stage 7: Force fold shadows and draw outlines the Guilty Gear way

**Tools:** UE 5.8's **Substrate Toon BSDF and Toon Profile** (experimental). It produces ramp-banded diffuse and specular lit by all light types including Lumen, with dithering, hatching and anisotropic specular, but **no built-in outlines yet** ([Epic 5.8](https://www.unrealengine.com/news/unreal-engine-5-8-is-now-available), snippet; [UE Roadmap](https://portal.productboard.com/epicgames/1-unreal-engine-public-roadmap/c/2410-substrate-npr-shading-experimental-), snippet). Free alternatives are VRM4U's MToon material (shadow color, outline, MatCap) ([GitHub](https://github.com/ruyo/VRM4U)) and the open [ue5-toon-shader-plugin](https://github.com/miltoncandelero/ue5-toon-shader-plugin). Paid Fab shaders run from **$9.99** ([Toon Shader](https://www.fab.com/listings/4a980632-851e-4df4-b501-e9aa7308482b?lang=en)) to **$249.99** ([Ultra Hybrid Toon Shader](https://www.fab.com/listings/06aef8cc-759f-4620-998b-08a9bd5fc208)); these prices come from snippets and may be stale.

**What to do.** Use 2 bands, or 3 for coats. Make shadow colors hue-shifted tones from your Stage 1 color sheet, never plain darkening. Copy *Guilty Gear Xrd*'s two signature tricks. First, a cel `step` threshold offset by a vertex-color channel, so pleat interiors, collar undersides and armpit folds fall into shadow at any light angle. Second, **inverted-hull outlines**, a duplicated, flipped mesh pushed along the normals ([GG Xrd GDC PDF](https://www.ggxrd.com/Motomura_Junya_GuiltyGearXrd.pdf)). Scale outline width by a vertex-color channel so ribbons get thin lines and hems get bold ones, and bake smoothed normals so hard hem edges don't split the outline (estimate). Keep painted fold and seam lines in the texture and use post-process edge lines only lightly. Give satin and silk anisotropic toon specular. Faces need their own treatment, either a Genshin-style face shadow texture compared against the light angle ([URPSimpleGenshinShaders](https://github.com/NoiRC256/URPSimpleGenshinShaders)) or normals transferred from a sphere proxy.

**Pitfalls.** Substrate Toon is experimental, and it is unknown whether it exposes a per-pixel threshold input. You may have to bake the fold mask into AO or diffuse, or use a custom toon material instead. Temporal anti-aliasing eats thin outlines (estimate).

### Stage 8: Retarget free motion, step it like anime, render in 4K

**Tools:**

| Source | Cost |
|---|---|
| **Mixamo** | Free with an Adobe ID ([Cinevva](https://app.cinevva.com/guides/mixamo-to-blender-2026), snippet) |
| **ActorCore** | 32 of about 4,500 motions free ([CG Channel](https://www.cgchannel.com/2025/07/rig-and-animate-3d-characters-for-free-with-accurig-2-0/), snippet) |
| **Rokoko Vision** video mocap | 30 s/month free; $10–12/month after that ([Rokoko](https://www.rokoko.com/products/vision), snippet) |
| **QuickMagic** | From $11.90/month (annual) ([iTechGuides](https://www.itechguides.com/products/quickmagic/), snippet) |
| **Move One** | 30 free credits; $18/month ([Uthana](https://uthana.com/resources/best-ai-motion-capture-tools), snippet) |

**What to do.** Right-click the body Skeletal Mesh and choose **Retarget Animations** (UE 5.4+), which auto-builds the IK Rig and Retargeter ([Epic](https://dev.epicgames.com/documentation/en-us/unreal-engine/auto-retargeting-in-unreal-engine), snippet). UE 5.7 added spatially aware retargeting, crotch height and a feet floor constraint, which help stylized proportions ([Epic 5.7](https://www.unrealengine.com/news/unreal-engine-5-7-is-now-available), snippet). Polish in Sequencer with Control Rig. For an anime feel in cinematics, consider stepping the body: *Guilty Gear Xrd* runs at about 15 fps ([BlenderNation](https://www.blendernation.com/2015/07/26/junya-c-motomura-behind-the-scenes-of-guilty-gear-xrd/)). If you step the body, decide deliberately whether the cloth steps too, because mixed rates read as jitter (estimate). *Spider-Verse* kept stepped cloth sims stable by simulating a hidden "ghost" in-between frame ([Cartoon Brew](https://www.cartoonbrew.com/feature-film/if-its-not-broke-break-it-sony-imageworks-renegade-approach-to-spider-man-into-the-spider-verse-167321.html)). Fix per-shot clipping with a Control Rig additive on the skirt bones. Render with Movie Render Queue at 4K, with high spatial samples to stabilize thin outlines, low or no motion blur, a fixed timestep, and warm-up frames so KawaiiPhysics settles before frame one (estimate; Epic's MRQ docs were blocked).

**Pitfalls.** Mocap on ones looks un-anime; key the big poses yourself. VRM4U users have hit foot wobble after retargeting in 5.7 ([YouTube fix](https://www.youtube.com/watch?v=C9JvND5Je44)).

### Stage 9: Swap outfits with one Data Asset each, adopt Mutable only when parts multiply

**What to do.** Create one Data Asset per outfit listing its garment meshes, any garment Anim Blueprints, and which body sections to hide. At runtime call SetSkeletalMeshAsset on each component and reapply Leader Pose. This takes about a day to build (estimate). For cinematics, make one Blueprint per outfit. **Mutable became production-ready in UE 5.8** ([Epic 5.8](https://www.unrealengine.com/news/unreal-engine-5-8-is-now-available), snippet). Its **Clip Mesh With Mesh** node automatically deletes body faces inside a garment volume ([Mutable wiki](https://github.com/anticto/Mutable-Documentation/wiki/)), and it merges meshes to cut draw calls. Expect 3–7 days to learn it (estimate), and adopt it only when you have many mix-and-match parts.

**Pitfalls.** Mutable's clipping **does not modify the cloth simulation mesh**, and it cannot extend two mesh sections that contain clothing ([Mutable wiki: Physics and Clothing](https://github.com/anticto/Mutable-Documentation/wiki/Physics-And-Clothing)). No source covers Mutable combined with KawaiiPhysics, so test that early.

## Marvelous Designer wins soft garments; AI meshes win only hard accessories

| | Marvelous Designer 2026.1 | Blender (manual + optional Simply Cloth Studio) | AI-generated garment meshes |
|---|---|---|---|
| Best for | Skirts, jackets, coats, layered dresses | Tight tops, leggings, bodysuits; simple skirts; every cleanup step | Rigid accessories: armor, shoes, hats, belts, bows |
| Quality and deformation | High: real patterns, free UVs, quad remesh | High if you are skilled; inherits body topology for tight pieces | Low for soft cloth; good for hard props after retopology |
| Fits your existing body | Native (avatar import, Auto Fitting) | Native (built on the body) | No: closed shell that must be cut and refit |
| Setup | Installer + paid subscription + learning patterns | Free; add-ons paid | Local conda install (moderate–high) or cloud account |
| Anime look | Must suppress realistic wrinkles (Brush Pinching helps) | Easiest to author bold folds directly | Baked lighting; flatten textures |
| Main trap | Micro-wrinkles; quad cleanup | Most manual hours | Interior faces, dense topology, license terms |

The table resolves into a clear rule. Every soft piece goes through MD or Blender, every hard accessory may start as an AI mesh, and Blender handles the cleanup, skinning and export for all three. Sewing-pattern AI and StdGEN are worth watching. Neither currently beats a person tracing patterns from a good turnaround, because both assume realistic or self-generated bodies rather than yours.

## A $0 floor, one subscription for comfort, and 1–3 days per outfit

| Tool | Role | Cost | Needed? |
|---|---|---|---|
| Blender 5.x | Renders, modeling, UVs, skinning, export | Free | Yes |
| ComfyUI + Qwen-Image-Edit-2511 | Dress your body renders | Free (Apache-2.0 code) | Yes |
| Character Turnaround Sheet LoRA | Multi-angle layout | Free download (pretrained; license unverified) | Optional |
| MangaLineExtraction / Cobra / CharacterGen | Clean, colorize or canonicalize panels | Free | Optional |
| Nano Banana Pro | Cloud alternative to Qwen | Paid API | Optional |
| Marvelous Designer 2026.1 (+ EveryWear) | Soft garments | Paid subscription (price unverified) | Recommended |
| Simply Cloth Studio 2.0 / Garment Tool | Sewing inside Blender | Paid add-ons (prices unverified) | Optional |
| Hunyuan3D 2.1 / TRELLIS.2 | Local AI accessory meshes | Free (Tencent community license / MIT) | Optional |
| Tripo / Meshy | Cloud AI meshes, segmentation | Paid credits | Optional |
| StableProjectorz | Project turnaround onto UVs | Free (AGPL-3.0) | Optional |
| Substance Painter | Polished hand-painting | Paid (price unverified) | Optional |
| Robust Weight Transfer | Body-to-garment weights | Free (GPL-3.0) | Yes |
| Auto-Rig Pro / AccuRIG 2 | Add chains / rig an unrigged body | $25–50 (snippet) / Free | If needed |
| Unreal Engine 5.8 | Engine, Sequencer, MRQ, Chaos, Mutable | Free for personal use | Yes |
| VRM4U | VRM body import + MToon | Free (MIT) | If the body is VRM |
| KawaiiPhysics | Skirt, ribbon, hair physics | Free (MIT) | Yes |
| Toon shader | Substrate Toon / MToon / free plugin / Fab | Free; Fab $9.99–$249.99 (snippet) | Yes (free is fine) |
| Mixamo / ActorCore / Rokoko Vision | Motion | Free / 32 free clips / $10–12/month | Yes (free is fine) |

The only recommended purchase is Marvelous Designer, and the all-Blender route brings the floor to **$0**. All time figures below are estimates combined from practitioner judgment; no source measured them.

| Stage | First outfit (includes one-time setup) | Each later outfit |
|---|---|---|
| Reference prep (+ ComfyUI setup) | 2–4 h | 0.5–1.5 h |
| Garment build (MD or Blender) | 3–10 h | 3–10 h |
| Retopology / UV cleanup | 2–6 h | 2–6 h |
| Flat texturing | 2–5 h | 2–5 h |
| Weights (+ adding bone chains once) | 4–8 h | 2–4 h |
| UE import + KawaiiPhysics tuning | 1–2 days | 2–4 h |
| Toon material (ramps, outline, fold mask) | 1–3 days | ~1 h (new material instance) |
| Outfit-switch Data Assets | ~1 day | Minutes |
| **Total** | **about 1–2 weeks** | **about 1–3 days** (half a day for a simple tight outfit) |

Budget an extra 2–5 days for any garment that needs Chaos Cloth, and about 0.5–1 hour for each AI-generated accessory including cleanup.

## Claims that rest on snippets or are unverified

Several load-bearing facts were seen only in search summaries because the primary pages were blocked: Chaos Cloth being production-ready in 5.8, Mutable being production-ready in 5.8, Substrate Toon's feature list, MD's UE 5.5 incompatibility and its USD workflow, Auto Fitting behavior, and all motion-capture and Fab shader prices. Treat these as probable but recheck them in Epic's and CLO's documentation before you build. Not verified at all: Marvelous Designer, Substance Painter and Blender add-on pricing; the turnaround LoRA's license; Robust Weight Transfer's supported Blender versions; EveryWear auto-rigging onto custom skeletons; whether Leader Pose followers run their own physics; and whether Substrate Toon allows a fold-shadow threshold input. The Tencent Hunyuan3D license's exclusion of the EU, UK and South Korea comes from prior knowledge, not a reading of the license this session, so check it if you live there. No benchmark compares the AI editors on keeping a supplied body's proportions, and no one has published a test of image-to-3D on garment-only anime inputs. Finally, the tool licenses here allow personal use, but the outfit designs belong to the manga's rights holder. Whether a public portfolio of fan-made costumes is acceptable is a separate question this research did not cover.

## Conclusion

The hard part of this project is not making a garment mesh. It is keeping the costume consistent across manga panels, mesh, texture and shader, and keeping the cloth looking drawn rather than simulated. Anime studios solved both problems before 3D existed: a simplified, approved model sheet as the single source of truth, and folds whose placement artists decide. The pipeline above transplants those solutions. The AI editor's real value is producing that model sheet *on your exact body* in minutes, while vertex-color shadow offsets, painted lines and KawaiiPhysics chains bring art direction back to cloth that a solver would otherwise make generic.

The investment is front-loaded. The bone chains, toon material, outline setup and outfit Data Asset are built once, so the first outfit costs a week or two and every later outfit costs days. That asymmetry is the same economics that led *Love Live!* to model its costumes once and reuse them in every performance. The areas to watch are the ones that are still experimental: Substrate Toon gaining native outlines, and anime-native garment generators (StdGEN++, Anime-Ready) publishing code that fits clothes to *your* body rather than their own. When those arrive, Stages 2 and 7 shrink. Until then, the person tracing the pattern and painting the folds is still the quality bottleneck.
