# Solo UE5 Pipeline for Dressing an Existing 3D Anime Body: clothing attach, cloth physics, anti-clipping, cloth cel shading, outfit switching (as of Oct 2026)

Research date: 2026-10-04.

**Scope update.** The user already has an unclothed anime body that is rigged or easy to rig. These notes therefore focus on the clothing side: attaching separately made garments to the existing skeleton, skirt, coat and ribbon physics, preventing poke-through, anime shading for cloth, and outfit switching. Rigging is covered only briefly.

**Proxy-blocked pages.** Many primary pages were blocked by the session's egress proxy and could be seen only as search-engine snippets:
- dev.epicgames.com (release notes and docs)
- forums.unrealengine.com
- cgchannel.com, 80.lv, gamefromscratch.com
- pafuhana1213.github.io (KawaiiPhysics portal)
- strayspark.studio
- docswell.com (Arc System Works slides)
- support.marvelousdesigner.com
- unrealdirective.com
- artofmaking.substack.com

The gh API was refused for repos outside the session. GitHub web pages did load: VRM4U, KawaiiPhysics and the Mutable docs wiki. "(snippet)" marks a claim seen only in a search summary of the cited page. Treat those as lower confidence.

**Engine versions.**
- UE 5.7 shipped Nov 2025 ([Epic news](https://www.unrealengine.com/news/unreal-engine-5-7-is-now-available); [CG Channel](https://www.cgchannel.com/2025/11/unreal-engine-5-7-five-key-features-for-cg-artists/)).
- UE 5.8 shipped around June 2026 ([CG Channel](https://www.cgchannel.com/2026/06/see-5-key-features-for-cg-artists-in-unreal-engine-5-8/); [Epic 5.8 news](https://www.unrealengine.com/news/unreal-engine-5-8-is-now-available); [5.8 release notes](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-5-8-release-notes), blocked).
- I found no evidence of a released UE 5.9 by Oct 2026.

## Getting the existing body into UE and rigging (brief): VRM4U, AccuRIG, Auto-Rig Pro, Mixamo, UE auto-retargeting

### Takeaway
If the body is a VRM (VRoid or Booth), use VRM4U (free, MIT). It imports the MToon material and spring bones and auto-builds an IK Rig, Retargeter and Control Rig. If the body is FBX and unrigged, use AccuRIG 2 (free) or Auto-Rig Pro ($25 Lite / $50 Full) when you need extra skirt or hair bones. In UE, right-click "Retarget Animations" (UE 5.4+) to get UE5 Manny animations onto it. UE 5.7 added proportion-aware retargeting that suits anime bodies.

### Cited Findings
- **VRM4U** (MIT) features:
  - an MToon-reproduction material with shadow color, outline and MatCap;
  - spring bones as either VRMSpringBone or a PhysicsAsset;
  - VRM 1.0 with constraints;
  - auto IK Rig and Retargeter;
  - Control Rig for body and morphs;
  - runtime loading.

  The README version line still says "UE5.4〜5.0", which is stale — [VRM4U GitHub](https://github.com/ruyo/VRM4U). Releases are active: v1.2026.09.12 (Sept 11 2026), v1.2026.09.02, v1.2026.07.22, 20260622 and 20260513 — [Releases](https://github.com/ruyo/VRM4U/releases). Users report UE 5.7 problems: animations (#558), Material Instance import, and retarget foot wobble — [Issue #558](https://github.com/ruyo/VRM4U/issues/558); [YouTube JP fix](https://www.youtube.com/watch?v=C9JvND5Je44)
- **AccuRIG 2.0** is free (an ActorCore account is needed for export). It produces body and finger rigs, exports FBX and USD, and has an Unreal export preset — [CG Channel](https://www.cgchannel.com/2025/07/rig-and-animate-3d-characters-for-free-with-accurig-2-0/) (snippet); [Reallusion Magazine](https://magazine.reallusion.com/2025/07/30/accurig-2-vs-mixamo-smarter-auto-rigging-for-3d-animators/)
- **Auto-Rig Pro**: Lite $25, Full $50. It supports Blender 2.93–5.1/5.2, with FBX/glTF export presets for Unreal and a Remap retarget tool — [Superhive](https://superhivemarket.com/products/auto-rig-pro); [Gumroad](https://artell.gumroad.com/l/autorigpro) (snippets)
- **Mixamo** is still free with an Adobe ID as of Sept 2026 — [Cinevva](https://app.cinevva.com/guides/mixamo-to-blender-2026) (snippet; secondary)
- **UE auto-retarget**: right-click a Skeletal Mesh and choose "Retarget Animations" (5.4+). This auto-creates the IK Rig and IK Retargeter with Auto Retarget Chains and Auto Align — [Epic: Auto Retargeting](https://dev.epicgames.com/documentation/en-us/unreal-engine/auto-retargeting-in-unreal-engine) (snippet)
- **UE 5.7 retargeting additions**: Spatially Aware Retargeting (less self-collision across proportions), Crotch Height, a feet Floor Constraint, squash-and-stretch retargeting, and better foot contact — [Epic 5.7 news](https://www.unrealengine.com/news/unreal-engine-5-7-is-now-available) (snippet)
- **UE 5.7** added a Control Rig / Modular Control Rig Dependency View — [Epic 5.7 news](https://www.unrealengine.com/news/unreal-engine-5-7-is-now-available) (snippet)

### Inferences
- Add the skirt, coat-tail, ribbon and hair bone chains to the body skeleton once, in Blender (Auto-Rig Pro or manual), before making outfits. Then every garment can be skinned to the same skeleton. Unused chains on outfits without skirts cost little. Alternatively, put the chains only in the garment mesh, which needs the Copy Pose approach covered in the next section. This is background knowledge.

### Gaps
- I did not verify Tripo or Meshy auto-rig, or the production status of Modular Control Rig in 5.8.

## Attaching separately made clothing meshes to the existing skeleton (modular skeletal meshes, Leader Pose / Copy Pose, skeletal mesh merge, Chaos Outfit Asset)

### Takeaway
Skin each garment in Blender to the same skeleton as the body: data-transfer weights from the body, then add garment-only chains. Import it as a Skeletal Mesh that uses the body's Skeleton asset. Attach it in the character Blueprint in one of three ways:
- **Leader Pose (Set Leader Pose Component)**: cheapest. No per-garment physics node.
- **Copy Pose From Mesh**: the garment gets its own small Anim BP, so its skirt can run KawaiiPhysics or RigidBody locally.
- **Skeletal Mesh Merge or Mutable**: fewer draw calls in game builds.

UE 5.6 introduced the Chaos Cloth **Outfit Asset**, which resizes and refits garments to different bodies. It is editor-time only.

### Cited Findings
- The standard UE approach: attach garments as extra Skeletal Mesh components and call **Set Leader Pose Component** (formerly "Master Pose") so they follow the body skeleton. A common variant uses an invisible skeleton mesh as the leader and makes each clothing mesh a follower — [yelzkizi: Dressing MetaHumans in UE5](https://yelzkizi.org/clothes-for-metahuman/); [UE forum: do games hide the body under clothing](https://forums.unrealengine.com/t/do-video-games-hide-the-player-model-underneath-the-clothing/33335) (snippets)
- **Chaos Cloth Outfit Asset (UE 5.6)**: an Outfit Asset concept with garment resizing and refitting, built for the parametric MetaHuman Creator. It came with the Beta Panel Editor, the Experimental Unified Dataflow Editor, Cloth-to-Cloth constraints and Simulation Morph Target support — [Epic tutorial: Chaos Cloth Updates 5.6](https://dev.epicgames.com/community/learning/tutorials/LZZo/unreal-engine-epic-games-store-chaos-cloth-updates-5-6) (snippet). Resizing **cannot run at runtime**, because it recreates a Cloth Asset inside the Outfit that must be cooked in the editor — [Epic tutorial: Outfit Asset Resizing Addendum](https://dev.epicgames.com/community/learning/tutorials/9Xjd/unreal-engine-chaos-cloth-outfit-asset-resizing-addendum) (snippet). API: [ChaosOutfitAsset plugin](https://dev.epicgames.com/documentation/unreal-engine/API/PluginIndex/ChaosOutfitAsset)
- A third-party Fab tool converts Chaos Outfits to Skeletal Meshes in bulk — [Fab: Chaos Outfit to Skeletal Mesh Mass Converter](https://www.fab.com/listings/54279990-6649-4e2a-849e-1d85bb7be039?lang=en) (title only)
- **UE 5.7**: "ClothAsset to Skeletal Mesh clothing data integration" was added in the Beta Dataflow Cloth Editor — [Epic tutorial: Chaos Cloth Updates 5.7](https://dev.epicgames.com/community/learning/tutorials/1o0R/unreal-engine-chaos-cloth-updates-5-7) (snippet)
- A follower-clothing walkthrough with Chaos Cloth in 5.4 — [Versluis](https://www.versluis.com/2024/06/creating-follower-clothing-with-chaos-cloth-in-unreal-engine-5-4/) (title/snippet)

### Inferences
- **Practical solo recipe** (background knowledge; Epic docs pages on Leader Pose and Copy Pose were blocked):
  1. In Blender, parent the garment to the body armature and copy weights with a Data Transfer modifier (Vertex Data → Vertex Groups, nearest face interpolated) or Auto-Rig Pro's weight tools. Hand-fix the armpits, crotch and collar.
  2. Add garment-only chains (skirt_*, coat_tail_*, ribbon_*) and paint those weights.
  3. Export FBX and import into UE, choosing the body's existing Skeleton asset. New bones are added to the Skeleton asset automatically.
  4. Attach in the Blueprint. Use Leader Pose for rigid pieces (shoes, gloves, tight tops). For skirts and coats, use a garment Anim BP with Copy Pose From Mesh, then KawaiiPhysics, so the garment owns its physics.
  5. Use Skeletal Mesh Merge or Mutable only once outfits are finalized and draw calls matter.
- Leader Pose followers do not evaluate their own Anim BP graph, so physics nodes on a follower won't run. That is why Copy Pose is the route for per-garment KawaiiPhysics. Background knowledge; confirm in the docs.
- Alternatively, put KawaiiPhysics for all skirt chains in the body's post-process or main Anim BP and keep garments as Leader Pose followers. This is simplest when every outfit uses the same chain names.

### Gaps
- I couldn't fetch Epic's docs on Leader Pose, Copy Pose From Mesh or Skeletal Mesh Merge (domain blocked). Their 5.8 behavior with Chaos Cloth followers is unverified.

## Skirt / coat / ribbon physics: KawaiiPhysics vs Chaos Cloth (Dataflow) vs Marvelous Designer (USD sim data or Alembic)

### Takeaway
There are three tiers.
1. **KawaiiPhysics** (free, MIT; UE 5.3–5.8). Bone-chain "fake physics" for pleated skirts, ribbons, coat tails, hair and accessories. It is the de-facto anime choice and is the easiest, most stylizable and cheapest option.
2. **Chaos Cloth**. Production-ready in UE 5.8, where the Dataflow Cloth Panel Editor became the default (it was Beta in 5.7). Use it for loose capes, long dresses and sleeves.
3. **Marvelous Designer**. MD 2025.2 exports USD with simulation data to Chaos Cloth Assets (UE 5.6+; UE 5.5 is incompatible). LiveSync 2 round-trips animated characters with UE 5.6. Use Alembic geometry caches for hero cinematic shots only.

### Cited Findings
- **KawaiiPhysics** (by pafuhana1213 / Kazuya Okada, Epic Games Japan) is pseudo-physics for swaying parts (hair, skirts, etc.) and suits Japanese animation-style games better than realistic physics — [GitHub](https://github.com/pafuhana1213/KawaiiPhysics); [80.lv](https://80.lv/articles/kawaii-physics-plug-in-now-supports-unreal-engine-5-5) (snippet)
  - **Versions and license**: UE 5.3 through 5.8 (v1.11.1 for UE 4.27). MIT license. Free on GitHub Releases and Booth; paid Fab listing with the same features — [GitHub README](https://github.com/pafuhana1213/KawaiiPhysics); [Fab](https://www.fab.com/listings/f870c07e-0a02-4a78-a888-e52a22794572). There is an 80.lv story "Kawaii Physics Updated For Unreal Engine 5.8" — [80.lv](https://80.lv/articles/kawaii-physics-now-works-with-unreal-engine-5-8) (blocked; title only)
  - **Features**:
    - sphere, capsule, plane and box colliders, editable in the viewport;
    - BoneConstraint, which keeps skirt bones apart and stops clipping;
    - SyncBone, which feeds animation-driven bones into the simulation;
    - bone subdivision;
    - fixed bone lengths, so bones never stretch or collapse;
    - procedural wind, world collision and external forces;
    - parameters drivable from Blueprint, C++, Sequencer tracks and AnimNotifyStates.

    Source: [GitHub README](https://github.com/pafuhana1213/KawaiiPhysics)
  - **v1.20.0** (Jan 2026; tag 20260106-v1.20.0) added UE 5.7 support and:
    - **SyncBone**: adjusts the mesh from reference-pose displacement and "solves issues like skirt clipping through legs" without a separate ControlRig or PoseDriver. Influence is distance-, ratio- or magnitude-based, per axis.
    - A **gravity overhaul**: the old gravity was ineffective in slow motion and Component Space only. It now has World Space gravity and an option to use the project's Default Gravity Z, plus a legacy mode.
    - An **AnimNotifyState** that overrides simulation alpha per animation section (constant or AnimCurve).
    - Box and Plane debug drawing.

    Source: [Discussion #185](https://github.com/pafuhana1213/KawaiiPhysics/discussions/185)
  - **Setup**: put the KawaiiPhysics node after the final pose in the AnimGraph and add a root bone per chain (hair_01, skirt_front_01…) — [Ida Faber 3D docs](https://docs.idafaber3d.com/unreal-engine/physics) (snippet)
  - **Sequencer caveat**: with a Sequencer animation track, the simulation may ignore play/stop. Workaround: play the clip via the Anim BP that contains the node — [UE forum](https://forums.unrealengine.com/t/how-to-use-kawaii-physics-in-the-sequencer/2019655) (snippet)
  - A community thread says the plugin is "kinda carrying the East Asian game industry" — [ResetEra](https://www.resetera.com/threads/an-unreal-engine-plugin-called-kawaiiphysics-is-kinda-carrying-the-east-asian-game-industry-on-its-back-right-now.965973/). Opinion; the thread was not fetched.
- **Chaos Cloth**:
  - **5.6**: Beta Panel Editor, Experimental Unified Dataflow Editor, Outfit Asset, Cloth-to-Cloth constraints, Simulation Morph Targets — [5.6 tutorial](https://dev.epicgames.com/community/learning/tutorials/LZZo/unreal-engine-epic-games-store-chaos-cloth-updates-5-6) (snippet)
  - **5.7**: Beta Dataflow Cloth Editor. Adds timeline and simulation controls, Skeletal Mesh Editor paint and selection tools, and ClothAsset-to-Skeletal-Mesh integration — [5.7 tutorial](https://dev.epicgames.com/community/learning/tutorials/1o0R/unreal-engine-chaos-cloth-updates-5-7) (snippet)
  - **5.8**: "The Chaos Dataflow Cloth Editor in 5.8 is Production-Ready and is now the default cloth editor for the engine". Also updated weight-map painting, graph improvements, extended rendering support and better stability — [UE forum: Chaos Cloth Updates 5.8](https://forums.unrealengine.com/t/tutorial-chaos-cloth-updates-5-8/2729420); [Epic 5.8 news](https://www.unrealengine.com/news/unreal-engine-5-8-is-now-available); [GameGPU](https://en.gamegpu.com/news/igry/epic-games-vypustila-unreal-engine-5-8-s-uluchshennoj-fizikoj-chaos-i-toon-shejderom) (snippets)
  - **References**: [Panel Cloth Editor Overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/panel-cloth-editor-overview); [Chaos Cloth Demystified](https://dev.epicgames.com/community/learning/tutorials/MZeq/unreal-engine-fortnite-chaos-cloth-demystified-a-zero-to-hero-guide-to-chaos-cloth); [Panel Cloth Dataflow & Collision 5.4](https://dev.epicgames.com/community/learning/tutorials/0Ja6/unreal-engine-panel-cloth-dataflow-and-collision-updates-5-4)
- **Marvelous Designer → UE**:
  - MD 2025.0 works only with UE 5.4. MD 2025.2 works with UE 5.6 and later. UE 5.5 is not compatible because of simulation-data issues — [MD support: Tips&Tricks MD + UE](https://support.marvelousdesigner.com/hc/en-us/articles/47358145573401--Tips-Tricks-Discover-Better-Workflow-with-Marvelous-Designer-and-Unreal-Engine) (snippet; blocked)
  - "USD Export Simulation Data" carries MD's garment simulation properties into a Chaos Cloth Asset (introduced with UE 5.4) — [MD support: MD to MetaHuman USD Garment workflow](https://support.marvelousdesigner.com/hc/en-us/articles/52699135975705-Marvelous-Designer-to-MetaHuman-USD-Garment-Integration-Workflow); [80.lv on UEFN integration](https://80.lv/articles/epic-games-on-integrating-metahuman-marvelous-designer-capabilities-into-uefn) (snippets)
  - **LiveSync 2**: one-click import and export of mesh, materials, skeletal animation and geometry caches. It sends an animated MetaHuman from UE 5.6 to MD and brings simulated garments back — [MD news: Tailoring for MetaHumans](https://www.marvelousdesigner.com/support/news/view/18b5b426d8be42da832d12cffec2dfa4) (snippet)
  - A UE forum thread reports that MD materials look different after USD import as a Chaos Cloth Asset — [UE forum](https://forums.unrealengine.com/t/marvelous-designer-material-appears-differently-in-unreal-engine-usd-import-chaos-cloth-asset/2223232) (title)
  - A Japanese slide deck covers an MD → UE5 → UEFN Chaos Cloth workflow (2024) — [docswell moyuki](https://www.docswell.com/s/moyuki/54VVWQ-2024-09-14-231311) (blocked)

### Inferences
- **Decision rule for anime outfits**:

  | Garment | Recommended method |
  |---|---|
  | Pleated school skirt, short skirt, ribbons, bows, coat tails, hood strings, ponytails | KawaiiPhysics bone chains, 6–12 radial chains × 3–5 bones for a skirt |
  | Long flowing dress, cape, wide sleeves | Chaos Cloth (5.8), or MD 2025.2 USD → Chaos Cloth Asset |
  | Hero close-ups in cinematics | MD-simulated Alembic or geometry cache; render-only, not for game builds |

  Why KawaiiPhysics for the first row: it gives a controllable, springy, overshooting anime motion and is deterministic enough for Sequencer.
- **Anime motion feel** (general anime practice, not sourced):
  - Raise the KawaiiPhysics spring response for overshoot.
  - Use AnimNotifyState alpha curves to exaggerate on landings and spins, or freeze cloth on held key poses.
  - If you step the body on 2s or 3s for limited animation, decide deliberately whether the cloth also steps. Mixed rates read as jitter.
- **Hybrid** (background knowledge): many anime games use bones for the main skirt silhouette and save true cloth sim for decorative flaps. A solo creator gets the most quality per hour from KawaiiPhysics first, adding Chaos Cloth only where bone chains look too stiff.

### Gaps
- I didn't verify the 5.8 status of ML Cloth, ML Deformer or Physics Control.
- I couldn't read the MD support pages for exact USD export steps or particle-distance tips (blocked).
- I couldn't see the KawaiiPhysics Fab price.

## Collision and poke-through prevention (body masking/hiding under clothes, physics asset capsules, KawaiiPhysics colliders, Mutable clipping)

### Takeaway
Layer four defences:
1. **Delete or hide the body under clothing.** Cut the body into regions and toggle them per outfit, use an opacity-mask texture, or let Mutable clip faces automatically with Clip Mesh With Mesh or Clip With UV Mask.
2. **Collide the skirt bones with legs and hips.** Use KawaiiPhysics capsule and plane colliders, BoneConstraint, and the 1.20 SyncBone.
3. **Collide Chaos Cloth** against a tuned Physics Asset made of capsules on the thighs, shins, pelvis and spine.
4. **Fix the art.** Slightly inflate garments and copy weights from the body.

### Cited Findings
- **Common practice for hiding the body**:
  - Hide or remove the body parts under clothing using a transparency (opacity) mask or by editing the mesh.
  - Split the body into modular pieces (torso, arms, legs…) and toggle visibility with flags like "coversTorso".
  - Delete hidden vertices, or use morphs that shrink hidden vertices.

  Sources: [UE forum: hide body under clothing](https://forums.unrealengine.com/t/do-video-games-hide-the-player-model-underneath-the-clothing/33335); [UE forum: poke-through advice](https://forums.unrealengine.com/t/please-little-help-or-advise-when-dealing-with-poke-through-issue/253814); [GameDev.net modular clothing](https://gamedev.net/forums/topic/710077-how-to-implement-a-modular-clothing-system/); [yelzkizi](https://yelzkizi.org/clothes-for-metahuman/) (snippets)
- **Mutable clipping nodes**: Clip Mesh With Mesh, Clip Mesh With Plane and Morph, Clip With UV Mask, and Clip Deform. The wiki has docs for v5.8 and v5.5 — [Mutable Documentation wiki](https://github.com/anticto/Mutable-Documentation/wiki/)
  - Clip Mesh With Mesh uses a closed "volume" mesh. Any body face entirely inside it is deleted, for example the body under a long-sleeve shirt. The mesh to remove must carry the same tag as the node — [Anticto: Remove unseen body parts](https://work.anticto.com/w/mutable/unreal-engine-4/user-documentation/remove-unseen-parts/); [Epic tutorial: Mutable Remove Unseen Mesh Parts](https://forums.unrealengine.com/t/tutorial-mutable-remove-unseen-mesh-parts/2135684) (snippets)
- **Mutable and cloth or physics**:
  - Mutable merges Physics Assets from parts. It works best with "complementary" physics assets where no bone is shared between BodySetups. When bodies overlap, shapes are aggregated and properties are copied from an unspecified first body.
  - Clothing simulation data transfers to generated meshes, but **"The simulation mesh is not modified. Clips, morphs... only affect the rendering mesh."**
  - It "Cannot extend two Mesh Sections containing clothing".
  - Shared cloth config (iterations, subdivisions) is taken from the first mesh.

  Source: [Mutable wiki: Physics and Clothing](https://github.com/anticto/Mutable-Documentation/wiki/Physics-And-Clothing). Epic doc: [Mutable Physics and Clothing](https://dev.epicgames.com/documentation/unreal-engine/mutable-physics-and-clothing-in-unreal-engine) (blocked)
- **KawaiiPhysics anti-clipping**: capsule, sphere, box and plane colliders; BoneConstraint (with Subdivision) for skirt-and-foot penetration; SyncBone (1.20) designed for skirt-through-leg clipping — [GitHub README](https://github.com/pafuhana1213/KawaiiPhysics); [Discussion #185](https://github.com/pafuhana1213/KawaiiPhysics/discussions/185)
- **Long-skirt leg clipping with Chaos Cloth** is a long-standing complaint — [UE forum: long skirt](https://forums.unrealengine.com/t/how-to-make-a-long-skirt-simulation-without-the-leg-going-through-it/222163); [UE forum: cloth clipping](https://forums.unrealengine.com/t/cloth-clipping/102939) (titles)

### Inferences
- **Cheapest robust method for a solo creator**: in Blender, split the body by material slot or mesh section into regions such as upper arms, torso, hips, thighs and shins. Per outfit, hide the covered sections. In UE this can be a per-material opacity-mask parameter set from a Data Asset per outfit; Mutable automates the same thing. Background knowledge.
- **Collider setup for KawaiiPhysics skirts**:
  - capsules on both thighs and on the pelvis;
  - a plane or capsules on the shins for long skirts;
  - for the front chains, SyncBone tied to the thigh bones, so the skirt is pushed by the leg rather than only colliding;
  - limit angles on the front chains during sitting animations, or blend the alpha down with an AnimNotifyState.
- **Chaos Cloth**:
  - Paint MaxDistance near zero at the waistband.
  - Use the Physics Asset's thigh and pelvis capsules as colliders, slightly fattened.
  - Use backstop or long-range attachment-style constraints so the cloth doesn't pass through the body under fast motion.
  - Consider substeps around 2–4 for cinematics.

  Background knowledge; parameter names should be verified in the 5.8 Dataflow editor.
- For Sequencer cinematics, poke-through in a single shot is often fastest fixed with a per-shot corrective: a Control Rig additive on skirt bones, or a sculpted corrective morph.

### Gaps
- I found no Epic docs on Chaos Cloth 5.8 collision parameter names (blocked).
- I didn't verify whether the 5.6+ Outfit Asset auto-hides body geometry.

## Anime cel shading specifically for cloth (shadow ramps, outlines on clothing edges, fold line art) and for faces; Lumen/MegaLights; MRQ rendering

### Takeaway
UE 5.8's **experimental Substrate Toon BSDF plus Toon Profile** gives ramp-banded diffuse and specular lit by real lights and Lumen. It has dithering and hatching options and anisotropic specular that suits satin and hair, but **no built-in outlines yet**. For cloth, borrow from Guilty Gear and gacha games:
- **inverted-hull outlines** with per-vertex width (vertex color), so hems, collars and pleat edges get controlled lines;
- **vertex-color or ILM-style threshold offsets**, so folds and creases fall into shadow predictably;
- **painted fold and seam lines** in texture or UV-aligned line maps, not left to post-process edge detection;
- **ramp textures per material** (fabric vs skin).

### Cited Findings
- **UE 5.8 Substrate Toon Shading (Experimental)** is built on the Substrate Blendable GBuffer (legacy) mode. It supports all light types including local lights, sky lights and Lumen GI, through the new Substrate Toon BSDF and Toon Profile asset — [Epic 5.8 news](https://www.unrealengine.com/news/unreal-engine-5-8-is-now-available) (snippet)
- The roadmap card "Substrate NPR Shading (Experimental)" lists:
  - ramp-based diffuse and specular with dithering;
  - self-shadow extinction with hatching patterns;
  - anisotropic specular;
  - GI scale.

  Forward shading support, silhouette/edge (outline) tech and performance work are planned for future releases — [UE Roadmap](https://portal.productboard.com/epicgames/1-unreal-engine-public-roadmap/c/2410-substrate-npr-shading-experimental-) (snippet)
- **Tutorials** say bands are driven by real light direction, shadowing and attenuation, and that post-process outlines remain necessary — [StraySpark](https://www.strayspark.studio/blog/substrate-toon-shader-ue5-8-tutorial) (blocked; snippet); [ProjProd](https://www.projprod.com/post/unreal-engine-5-8-substrate-toon-shader); [YouTube](https://www.youtube.com/watch?v=t5rfc56Do94)
- **Guilty Gear Xrd/Strive (Arc System Works)**:
  - Inverted-hull outlines in every 3D fighter since Xrd: a duplicated, flipped mesh pushed along vertex normals.
  - Vertex normals, vertex colors and UVs store artistic data because it is resolution-independent.
  - A `step` threshold for cel lighting, with a vertex-color channel offsetting the threshold so artists can make areas shadow more easily. This is the mechanism for forcing fold and crease shadows.
  - Hand-edited normals.
  - ASW's separate "Toon Line Control Techniques" deck covers line control.

  Sources: [GG Xrd GDC PDF](https://www.ggxrd.com/Motomura_Junya_GuiltyGearXrd.pdf); [ASW Toon Line Control (ENG)](https://www.docswell.com/s/ASW_Academy/5LVY67-GG-Toonline-Eng) (blocked); [Scribd summary](https://www.scribd.com/document/331351299/Blender-NPR-Cel-Shading-GuilltyGearXrd-Shader) (snippets)
- **Genshin-style face shadow**: a texture stores shadow coverage for light angles 0–180° in the XZ plane and is compared against FdotL. It needs symmetric face UVs and forward/left vectors, and it avoids normal editing — [NoiRC256 URPSimpleGenshinShaders](https://github.com/NoiRC256/URPSimpleGenshinShaders); [ChiliMilk URP_Toon](https://github.com/ChiliMilk/URP_Toon/blob/master/README.md); [NoiRCCC blog](https://noirccc.net/blog/posts/54); [Adrian Mendez breakdown](https://www.artstation.com/artwork/wJZ4Gg) (Unity examples; technique is engine-agnostic)
- **Inner ink lines** can come from a normal-vs-view-direction (fresnel) term. Cel shaders on the UE marketplace offer texture banding, mesh outlines, and inline/ID-map lines — [UE forum: anime-inspired shading model](https://forums.unrealengine.com/t/anime-inspired-shading-model/156903); [ArtStation Cartoon Cel Shader](https://www.artstation.com/marketplace/p/NK8L/cartoon-cel-shader-unreal-engine-4); [ImaginaryBlend Anime Toon Shading](http://imaginaryblend.com/2018/06/26/anime-toon-shading/) (snippets)
- Genshin's GDC 2021 talk (Haoyu Cai) covers art direction and pipeline, not shader internals — [GDC Vault](https://www.gdcvault.com/play/1027538/-Genshin-Impact-Crafting-an)
- **Shader options and prices** (search snippets; may be outdated):

  | Product | Price | Notes |
  |---|---|---|
  | [Ultra Hybrid Toon Shader](https://www.fab.com/listings/06aef8cc-759f-4620-998b-08a9bd5fc208) | $249.99 | post-process; Lumen/Nanite |
  | [Cel Shader Pro](https://www.fab.com/listings/5e93dcf3-36d1-42bf-9e10-cb529d9f1f78) | not shown | post-process |
  | [Toon Shader](https://www.fab.com/listings/4a980632-851e-4df4-b501-e9aa7308482b?lang=en) | $9.99 | — |
  | [Anime Toon Shading](https://www.unrealengine.com/marketplace/en-US/product/anime-toon-shading) | $44.99 | geometry-based |
  | [Advanced Cel Shader Essentials](https://www.unrealengine.com/marketplace/en-US/product/advanced-cel-shader-pack) | $74.99 | per-character material plus lighting |
  | [Genshin Impact Character Shader for UE (ArtStation)](https://www.artstation.com/marketplace/p/aJV1q/genshin-impact-character-shader-for-unreal-engine) | not shown | UE 4.27+/5 |
  | [ue5-toon-shader-plugin](https://github.com/miltoncandelero/ue5-toon-shader-plugin) | free | — |
  | [Free anime toon shader](https://forums.unrealengine.com/t/free-effective-anime-toon-shader-for-unreal-engine-4-5/561965) | free | — |
  | VRM4U MToon material | free | — |

### Inferences
- **Cloth shading recipe** (synthesized from the Guilty Gear and Genshin techniques plus 5.8 Toon; my synthesis):
  - **Banding**: a Substrate Toon Profile with 2 bands, or 3 for coats. Give shadow tints a hue shift (warmer or bluer shadow colour per fabric) rather than plain darkening. A small rim band helps.
  - **Forcing fold shadows**: paint a vertex-color channel (or a mask texture, "ILM-style") that offsets the shadow threshold, so the inside of pleats, the underside of collars and the armpit folds go dark at any light angle. With Substrate Toon you may need to add this as a custom threshold offset, or bake it into the AO and diffuse; it's unknown whether the Toon BSDF exposes a per-pixel threshold input.
  - **Line art on cloth**:
    - inverted-hull outline for the silhouette and hems, with width multiplied by a vertex-color channel so thin ribbons get thin lines and pleat edges get emphasis, and scaled by camera distance;
    - painted fold, seam and stitch lines in the albedo, or a UV-aligned line texture (Guilty Gear "Motomura lines");
    - optional post-process depth/normal edge lines for the interior, at low strength.
  - **Smooth outline normals**: bake smoothed normals into a UV channel or vertex color for the hull, so hard-edged hems don't split the outline.
  - **Satin and silk ribbons**: anisotropic toon specular, using Substrate Toon's anisotropic option or a matcap.
- **Faces** need separate treatment: a face SDF shadow map or normals transferred from a sphere or ellipsoid proxy. Default normals give blotchy shadows. Background knowledge.
- **Render** (background knowledge; MRQ docs were blocked):
  - Movie Render Queue/Graph with high spatial or temporal sample counts to stabilize thin outlines.
  - Low or no motion blur for a cel look.
  - 4K output.
  - Warm-up frames so cloth and KawaiiPhysics settle before the shot starts.

### Gaps
- No primary source found for HSR, ZZZ, Wuthering Waves, Blue Protocol or Tower of Fantasy cloth shading. No CEDEC or CGWORLD articles were fetched within budget.
- Unknown whether Substrate Toon works with MegaLights or translucent cloth (lace, chiffon), or whether it allows per-pixel threshold offsets.

## Outfit switching in games (modular meshes, Leader Pose, Mutable, mesh merge)

### Takeaway
For a handful of outfits, a Data Asset per outfit is the easiest route. Each Data Asset lists its garment Skeletal Meshes, optional Anim BPs for physics, and which body sections to hide. Swap the meshes at runtime with SetSkeletalMeshAsset and Leader Pose or Copy Pose. **Mutable became production-ready in UE 5.8** ("dataless" Customizable Objects for runtime parameters). It suits many mix-and-match parts, automatic body clipping and merged meshes. Cloth sim meshes are not clipped by Mutable.

### Cited Findings
- Mutable in 5.8 is production-ready, with stability, performance and pipeline improvements. Dataless customizable objects give runtime-driven parameters, faster iteration, graph reuse and better patching, and are optimized for mesh operations and simple texture composition — [Epic 5.8 news](https://www.unrealengine.com/news/unreal-engine-5-8-is-now-available) (snippet)
- Earlier roadmaps listed Mutable as Beta and Dataless Mutable as Experimental — [UE Roadmap: Mutable (Beta)](https://portal.productboard.com/epicgames/1-unreal-engine-public-roadmap/c/1628-mutable-customizable-characters-and-meshes-beta-); [UE Roadmap 5.7](https://portal.productboard.com/epicgames/1-unreal-engine-public-roadmap/tabs/127-unreal-engine-5-7) (snippets)
- Mutable can remove hidden body parts (Clip Mesh With Mesh, UV Mask, and others) and merges physics assets and clothing data, with the limitations listed in the collision section — [Mutable wiki](https://github.com/anticto/Mutable-Documentation/wiki/); [Physics and Clothing](https://github.com/anticto/Mutable-Documentation/wiki/Physics-And-Clothing)
- The Chaos Outfit Asset resizes garments only in the editor, not at runtime — [Epic tutorial](https://dev.epicgames.com/community/learning/tutorials/9Xjd/unreal-engine-chaos-cloth-outfit-asset-resizing-addendum) (snippet)

### Inferences
- **Solo game route**: start with the Data Asset plus Leader Pose / Copy Pose approach, which takes hours to set up. Move to Mutable only if you have many parts or colour variants, or need draw-call merging, which takes days to learn. Mutable's KawaiiPhysics/Anim BP interaction is undocumented in the sources I could reach, so test it early.
- **For cinematics**, outfit switching is just swapping components in the Sequencer-spawned Blueprint, or making one Blueprint per outfit.

### Gaps
- No verified source on Mutable + KawaiiPhysics, or on Mutable with Chaos Cloth Assets (as opposed to legacy skeletal clothing data) in 5.8.

## Animation sources (brief) and recommended solo stack, time investment, common pitfalls

### Takeaway
**Animation sources**: Mixamo (free), ActorCore (32 free motions out of about 4,500), and cheap video mocap. Retarget everything with "Retarget Animations", then stylize in Sequencer or Control Rig.

| Mocap tool | Free tier | Paid from |
|---|---|---|
| Rokoko Vision | 30 s/month | $10–12/month |
| QuickMagic | yes | $11.90/month annual |
| Move One | 30 credits | $18/month |

**Recommended clothing-centric stack**:

| Stage | Choice | Cost / status |
|---|---|---|
| Engine | UE 5.8 | free for personal use |
| Garment modelling | Blender, or MD 2025.2 | MD for USD → Chaos |
| Weight transfer | Data Transfer from the body | built into Blender |
| Attaching | Leader Pose / Copy Pose | built-in |
| Skirts, ribbons, coat tails | KawaiiPhysics | free |
| Long cloth | Chaos Cloth Dataflow editor | production-ready in 5.8 |
| Body hiding | per-section hiding or Mutable clipping | built-in |
| Shading | Substrate Toon (experimental) plus inverted-hull outlines, vertex-color fold thresholds, painted fold lines | built-in |
| Render | Movie Render Queue | built-in |

### Cited Findings
- **Pricing**:
  - Rokoko Vision: Starter free (30 s/month); Basic $12/month or $10/month annual (600 s); Plus $28 or $20 (3,000 s); Pro $70 or $50 (15,000 s) — [Rokoko Vision](https://www.rokoko.com/products/vision) (snippet)
  - QuickMagic: Basic $11.90/month annual (200 credits); Pro $39.90/month annual (1,000 credits) — [iTechGuides](https://www.itechguides.com/products/quickmagic/) (snippet; secondary)
  - Move One: 30 free credits; Starter $18/month (60 credits) — [Uthana](https://uthana.com/resources/best-ai-motion-capture-tools); [TATO Studio](https://tato.studio/blog/best-ai-video-to-mocap) (snippets; secondary)
  - ActorCore: about 4,500 motions, 32 free — [CG Channel](https://www.cgchannel.com/2025/07/rig-and-animate-3d-characters-for-free-with-accurig-2-0/) (snippet)
- **Known pitfalls with sources**:
  - KawaiiPhysics and Sequencer play/stop — [UE forum](https://forums.unrealengine.com/t/how-to-use-kawaii-physics-in-the-sequencer/2019655)
  - Old KawaiiPhysics gravity weak in slow motion; fixed in 1.20 — [Discussion #185](https://github.com/pafuhana1213/KawaiiPhysics/discussions/185)
  - VRM4U breakage in 5.7 — [Issue #558](https://github.com/ruyo/VRM4U/issues/558)
  - MD → UE 5.5 incompatibility — [MD support](https://support.marvelousdesigner.com/hc/en-us/articles/47358145573401--Tips-Tricks-Discover-Better-Workflow-with-Marvelous-Designer-and-Unreal-Engine) (snippet)
  - Mutable clips don't affect cloth sim meshes — [Mutable wiki](https://github.com/anticto/Mutable-Documentation/wiki/Physics-And-Clothing)
  - Substrate Toon is experimental and has no outlines — [UE Roadmap](https://portal.productboard.com/epicgames/1-unreal-engine-public-roadmap/c/2410-substrate-npr-shading-experimental-)
  - MD materials differ after USD import — [UE forum](https://forums.unrealengine.com/t/marvelous-designer-material-appears-differently-in-unreal-engine-usd-import-chaos-cloth-asset/2223232)

### Inferences
- **Time estimates per outfit** (my estimates; no source):

  | Task | Time |
  |---|---|
  | Garment model, weight transfer, UE import, Leader Pose | 0.5–2 days |
  | KawaiiPhysics skirt and ribbon chains with colliders and SyncBone tuning | 1–2 days |
  | Toon material for cloth (ramps, outline, fold mask, painted lines) | 1–3 days for the first outfit, faster afterwards |
  | One Chaos Cloth long garment | 2–5 days |
  | Outfit-switch system with Data Assets | about 1 day |
  | Mutable (learning) | 3–7 days |

- **Other pitfalls and fixes** (background knowledge):
  - Cloth or KawaiiPhysics "pop" on camera cuts or teleports: reset the simulation on cut and add MRQ warm-up frames.
  - Jitter at variable frame rate: use a fixed timestep for renders.
  - Outline breaks on hard hem edges: bake smoothed normals for the hull.
  - Temporal AA eats thin cloth outlines: raise MRQ spatial samples.
  - Thighs poking through skirts when sitting: AnimNotifyState alpha, SyncBone, or a per-shot Control Rig corrective.
  - Body skin showing through tight tops: hide the body section or use a Mutable clip.
  - Different chain names across outfits break a shared Anim BP: standardize bone names.

### Gaps
- Not verified within budget: Plask, DeepMotion and FreeMoCap 2026 status; text-to-motion tools; GASP version.
- No sourced benchmarks for time investment.
