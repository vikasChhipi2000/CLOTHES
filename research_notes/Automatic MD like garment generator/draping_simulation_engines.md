# Cloth Simulation / Draping Engines for an Automatic, Scriptable "Marvelous-Designer-like" Pipeline (GarmentCode JSON -> custom anime body -> clean garment mesh -> UE5)

Research date: 2026-10-04. Researcher environment note: the egress proxy BLOCKED direct fetches of arxiv.org, igl.ethz.ch, ecva.net, developer.marvelousdesigner.com, sidefx.com, docs.blender.org and cgchannel.com. Claims from those sites are taken from **search-result snippets only** and are marked "(snippet-only)". GitHub was reachable: the GarmentCode repo (v2.0.2), the NvidiaWarp-GarmentCode fork, Newton (sparse clone, commit dated 2026-10-04) and several READMEs were **cloned or downloaded and read directly**. Those claims are marked "(code-verified)".

---

## Q1. GarmentCode's Warp-based simulator: placement, stitching, draping, using an arbitrary body, needed changes, run time

### Takeaway
GarmentCode/pygarment v2 already runs the whole pipeline headless from the command line: pattern JSON -> "box mesh" with stitched vertices merged -> panels placed in 3D by translation/rotation computed from body measurements -> XPBD drape in a forked NVIDIA Warp -> OBJ with UVs taken from the panels. It reports about 30 s per garment on an RTX 3090. The body is just an OBJ file, so a custom anime body can be used. It also needs a matching measurements YAML and a vertex-segmentation JSON for that exact mesh, plus a few path tweaks. The major catch is the license: the Warp fork is under the old NVIDIA Source Code License, which allows **non-commercial use only** ("research or evaluation purposes only"). It is also frozen on Warp 1.0.0-beta.6, and upstream Warp has since removed `warp.sim`.

### Cited Findings
**Pipeline and entry points**
- The generation pipeline has two steps: sample sewing patterns (`pattern_sampler.py`), then drape each one over the base body (`pattern_data_sim.py --data <name> --config <sim_config> [--default_body]`). Simulation can be resumed and run in minibatches, and `pattern_data_sim_runner.sh` restarts after hangs or crashes — [GarmentCode docs/Running_data_generation.md](https://github.com/maria-korosteleva/GarmentCode/blob/main/docs/Running_data_generation.md) (code-verified)
- Single-garment headless script `test_garment_sim.py`: it takes `--pattern_spec *_specification.json` and `--sim_config`. It builds a `BoxMesh(spec, resolution_scale)`, then calls `.load()` and `.serialize(..., uv_config=...)`, then `run_sim(...)`. The body is chosen with `PathCofig(body_name='mean_all', smpl_body=False)` — [test_garment_sim.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/test_garment_sim.py) (code-verified)
- pygarment version 2.0.2 (2025-04-18) is MIT licensed. The 2.0.0 changelog says box-mesh generation and Warp simulation "can be run from command line and from GUI". The fork also adds labels on edges and panels that guide simulation, plus explicit stitch orientation (right-to-right vs right-to-wrong) — [GarmentCode CHANGELOG.md](https://github.com/maria-korosteleva/GarmentCode/blob/main/CHANGELOG.md), [LICENSE](https://github.com/maria-korosteleva/GarmentCode/blob/main/LICENSE) (code-verified)

**Placement ("arrangement")**
- Each panel in the JSON spec carries a 3D `translation` and XYZ-Euler `rotation`. BoxMesh applies them to the 2D panel vertices (`rot_trans_panel`, `euler_xyz_to_R`) — [pygarment/meshgen/boxmeshgen.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/boxmeshgen.py) (code-verified)
- The garment programs compute these placements from body measurements. For example, the bodice front is placed with `translate_by([0, body['height'] - body['head_l'] - max_len - shoulder_incl, 0])`, and the front and back torso are offset by `translate_by([0,0,30])` and `([0,0,-25])` cm. Placement is therefore **measurement-driven, not landmark-on-mesh-driven** — [assets/garment_programs/bodice.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/garment_programs/bodice.py) (code-verified)
- The body parameter set is about 26 measurements in cm: height, head_l, bust, underbust, waist, hips, waist_line, hips_line, bust_line, vert_bust_line, shoulder_w, shoulder_incl, armscye_depth, arm_length, arm_pose_angle (≈45° A-pose), neck_w, wrist, leg_circ, back_width, waist_back_width, hip_back_width, bust_points, bum_points, crotch_hip_diff, hip_inclination, waist_over_bust_line. Derived values include `_waist_level = height - head_l - waist_line` — [assets/bodies/mean_all.yaml](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/bodies/mean_all.yaml), [assets/bodies/body_params.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/bodies/body_params.py) (code-verified)

**Stitching**
- Stitches are realized as a **box mesh**. `collapse_stitch_vertices()` merges the vertices of stitched edge pairs, so the garment becomes one connected mesh *before* simulation; there are no sewing springs that close gaps over time — [boxmeshgen.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/boxmeshgen.py) (code-verified). The paper also says it computes "a single connected mesh from the pattern panels by connecting vertices of edge pairs participating in a stitch" — [arXiv 2405.17609 html](https://arxiv.org/html/2405.17609) (snippet-only)

**Draping and solver**
- The fork changes Warp's XPBD solver in five ways — [NvidiaWarp-GarmentCode README](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode) (code-verified):
  - point-triangle and edge-edge self-collision;
  - equality and inequality attachment constraints (for example, fixing a skirt at the waist);
  - "body-part drag": intersecting panels are dragged toward their assigned body part until collisions resolve;
  - a body collision constraint that pushes cloth found inside the body outward;
  - `mesh_query_edge()`.
- Default sim props — [assets/Sim_props/default_sim_props.yaml](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/Sim_props/default_sim_props.yaml) (code-verified):
  - Run length and early stop: `max_sim_steps: 2400`, `max_sim_time: 600` s, `static_threshold: 0.03`, `zero_gravity_steps: 10`.
  - Material: `garment_edge_ke: 1.0 # Very soft fabric` (bending), `garment_tri_ke/ka: 10000`, `spring_ke: 50000`.
  - Collisions: triangle-particle and edge-edge collisions on, body collision filters on.
  - Attachment: on for 400 frames, with labels `lower_interface`, `right_collar`, `left_collar`, `strapless_top`.
  - Drag and smoothing: `enable_cloth_reference_drag: true`, `enable_body_smoothing: false`.
  - Quality-check thresholds: `max_body_collisions: 35`, `max_self_collisions: 300`.
  - Other presets: `mid_bending.yaml`, `minimal_bending.yaml`.
- Simulation runs at 60 fps with 10 substeps — [pygarment/meshgen/sim_config.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/sim_config.py) (code-verified)
- "Body smoothing" is optional: the sim starts from a Laplacian-smoothed body and gradually restores detail (`implicit_laplacian_smoothing`, `smoothing_recover_start_frame`). Attachments use measurement-derived levels: waist level from height - head_l - waist_line, collar from neck_w, strapless from armscye depth — [pygarment/meshgen/garment.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/garment.py) (code-verified)

**Run time and scale**
- About 30 seconds per garment on an RTX 3090. Warp's XPBD was chosen as "one of very few open-source and GPU-based simulators". The dataset holds 115,000 garments — [arXiv 2405.17609 html](https://arxiv.org/html/2405.17609) (snippet-only)

**Body handling (relevant to a custom body)**
- The body is loaded from `<bodies_path>/<body_name>.obj` and scaled ×100 (`b_scale = 100.0`, so the OBJ is in metres and the sim in cm). It is shifted up if min-y < 0, so the feet are expected at y=0 with Y up — [garment.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/garment.py), [sim_config.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/sim_config.py) (code-verified)
- The segmentation path is hard-coded: `bodies_default_path/ggg_body_segmentation.json`, or `smpl_vert_segmentation.json` when `smpl_body=True`. The non-SMPL file is a dict of **vertex-index lists** for `body`, `left_arm`, `right_arm`, `left_leg`, `right_leg`, `face_internal` (10,697 / 3,094 / 3,067 / 2,911 / 2,870 / 1,096 vertices) — [sim_config.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/sim_config.py), [assets/bodies](https://github.com/maria-korosteleva/GarmentCode/tree/main/assets/bodies) (code-verified)
- `panel_assignment()` labels each cloth panel with the closest or ray-hit body part. These labels drive both the collision filters (for example, skirts ignore arm collisions) and the body-part drag — [NvidiaWarp-GarmentCode warp/collision/panel_assignment.py](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode/blob/main/warp/collision/panel_assignment.py) (code-verified). The paper describes this as a segmentation mask on the body marking each limb and the trunk — [arXiv 2405.17609 html](https://arxiv.org/html/2405.17609) (snippet-only)
- Bundled bodies: `mean_all`, `mean_female`, `mean_male`, a T-pose variant, and SMPL average female/male A40. The non-SMPL bodies come from the authors' own model ([GarmentMeasurements](https://github.com/mbotsch/GarmentMeasurements)). The SMPL license prevents sharing more bodies — [assets/bodies/Readme.md](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/bodies/Readme.md) (code-verified)
- The `GarmentMeasurements` tool (C++, CGAL and FBX SDK) samples shapes and runs `./measurements output.obj measurements.yaml` — [mbotsch/GarmentMeasurements README](https://github.com/mbotsch/GarmentMeasurements) (code-verified; it is unclear whether it works on meshes outside its own template, see Gaps)

**Output**
- The box mesh OBJ gets UVs via `texture_mesh_islands`, with each panel as one UV island. Since v2.0.2 the UVs preserve the panels' aspect ratio. A seam-width and fabric-grain texture is generated — [boxmeshgen.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/boxmeshgen.py), [CHANGELOG](https://github.com/maria-korosteleva/GarmentCode/blob/main/CHANGELOG.md) (code-verified)
- Outputs are `<tag>_sim.obj` plus a sim segmentation txt — [sim_config.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/sim_config.py) (code-verified)

**Install and license**
- The fork is built from Warp v1.0.0-beta.6. It requires a manual build: VS2019+ or GCC 7.2+, CUDA Toolkit ≥11.5, Git LFS. The conda env uses Python 3.9 — [NvidiaWarp-GarmentCode README](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode), [GarmentCode docs/Installation.md](https://github.com/maria-korosteleva/GarmentCode/blob/main/docs/Installation.md) (code-verified)
- The fork's LICENSE.md is the "NVIDIA Source Code License for Warp", §3.3: "The Work and any derivative works thereof only may be used or intended for use non-commercially… 'non-commercially' means for research or evaluation purposes only." — [NvidiaWarp-GarmentCode LICENSE.md](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode/blob/main/LICENSE.md) (code-verified)
- Upstream Warp history — [NVIDIA/warp CHANGELOG](https://github.com/NVIDIA/warp/blob/main/CHANGELOG.md) (code-verified):
  - license changed to an NVIDIA license allowing commercial use in 0.13.0 (2024-02-16);
  - changed to Apache-2.0 in 1.6.2 (2025-03-07);
  - `warp.sim` deprecated in 1.8.0 (2025-07);
  - `warp.sim` **removed in 1.10.0 (2025-11-02)**, "superseded by the Newton library".

### Inferences
- **Changes needed to drape on a custom anime body:**
  1. Export the body as a single watertight OBJ in metres, Y-up, feet at y=0, A-pose with arms at about the angle written to `arm_pose_angle`. Remove hair, eyes, separate accessories and inner mouth faces.
  2. Write `<name>.yaml` with the ~26 measurements (`body:` block) so the garment programs size and place the panels.
  3. Write a segmentation JSON of vertex indices for *this* mesh with the six labels. This is easy to derive from skin weights (arm bones → arm, thigh/shin bones → leg, head → face_internal). Point `PathCofig.body_seg` at it; it is currently hard-coded, so either overwrite the file or patch about one line.
  4. Pass `body_name=<name>`, `smpl_body=False`.
  
  No solver changes are strictly required. The body is used purely as a collision triangle mesh plus per-vertex labels.
- **Anime proportions risk:** the garment programs encode real-human tailoring rules (for example, how bust_line, waist_line and shoulder inclination are combined). Very large heads (head_l), short torsos, narrow waists or exaggerated chests may produce plausible-but-odd patterns. Measurements may need clamping. Sanity-check the attachment levels (waist level = height − head_l − waist_line).
- **Layering:** the simulator drapes one pattern spec (which can contain upper + lower garments together) over one body. True multi-layer (shirt under jacket) would mean either simulating an outfit as one spec, or simulating sequentially and adding the previous garment's result to the collision mesh. The second option needs code in `garment.py`: add another `add_shape_mesh` and a segmentation for it.
- **Personal use:** for a solo creator's personal, non-commercial use, the fork's "research or evaluation" clause is a grey area. If any output ships commercially (for example, a sold UE5 game), this is a real licensing blocker. A license-clean path is to port the box-mesh pipeline to Newton (Apache-2.0, see Q5) or upstream Warp ≥1.6.2 code.
- The 600 s max sim time, the frame timeouts and the quality checks (body/self-collision counts, a "static" stop criterion) are a useful built-in success/failure signal for an AI agent loop.

### Gaps
- Could not read the GarmentCodeData paper body (arxiv/ecva/igl blocked): no exact failure rates, the per-garment time distribution, or GPU-memory figures beyond the "30 s on RTX 3090" snippet.
- No public GitHub issue or discussion about running GarmentCode on non-template or non-human-proportioned bodies was found; the issues API was blocked.
- Unknown whether `GarmentMeasurements/measurements` works on an arbitrary topology mesh or needs its own template.

---

## Q2. Marvelous Designer / CLO automation (Python API, headless, DXF, licensing)

### Takeaway
MD has an embedded Python API covering import (avatar, DXF, zpac/zprj, OBJ/FBX), patterns/arrangement queries, fabrics, simulation and export (OBJ/FBX/USD). Community MCP bridges already let an AI agent drive MD. However, **MD has no true headless or batch mode**: scripts run inside the GUI process and the bridge freezes the GUI while it listens. The Personal license is cheap and allows commercial use for sole proprietors. There is no native GarmentCode JSON import, so you would need a converter (to DXF-AAMA or to API calls that build patterns) plus sewing via API.

### Cited Findings
- **API modules:** EXPORT_API, IMPORT_API, PATTERN_API, FABRIC_API, UTILITY_API and REST_API.
  - `import_api.ImportDXF(path)` returns a bool; there is also `ImportAlembic`.
  - `pattern_api.GetArrangementOfPattern` returns arrangement name, type, offset and orientation.
  - Fabric API includes Get/Set map and `GetFabricIndexForPattern`.
  - `ImportExportOption` controls export behaviour.
  - Sources: [MD API List](https://developer.marvelousdesigner.com/list.html), [MD API changelog](https://developer.marvelousdesigner.com/changelog.html) (snippet-only; site blocked)
- **Documented scenario:** `utility_api.NewProject()` → `import_api.ImportAvatar()` → `utility_api.Simulate()`. Export uses `ExportOBJ(_filePath: str, _options: ImportExportOption) -> list[str]` — [MD API Scenario](https://developer.marvelousdesigner.com/scenario.html), [MD API List](https://developer.marvelousdesigner.com/list.html) (snippet-only)
- Supported avatar formats are OBJ, FBX, Alembic and OpenCOLLADA — [MD support: Can I import my own avatar?](https://support.marvelousdesigner.com/hc/en-us/articles/47358210334745-Can-I-import-my-own-avatar) (snippet-only)
- **ysk424/marvelous-designer-mcp:** an MCP server that runs code inside MD's embedded Python 3.11 through a TCP listener on 127.0.0.1:7421. It works for MD 2026 or CLO 2026 ("CLO… shares the same Python API") and needs no C++ SDK or CLO-SET API key. "MD's embedded Python does not schedule background threads… While the listener is running, MD's window is unresponsive." Tools include `execute_python`, `import_project` (`import_api.ImportFile` for .zprj/.zpac/.obj/.fbx), `export_project` (`export_api.ExportZPrjW`) and `assign_fabric` (`fabric_api.AssignFabricToPattern`) — [ysk424/marvelous-designer-mcp README](https://github.com/ysk424/marvelous-designer-mcp) (code-verified README)
- **Laboon2501/MarvelousDesigner-MCP** — [README](https://github.com/Laboon2501/MarvelousDesigner-MCP) (code-verified README):
  - 42 tools covering scene inspection, patterns, stable topology, whole-edge sewing, fabric, OBJ avatars, simulation, mesh measurements, checkpoints, import/export and raw Python.
  - Verified only on **Windows, MD 2026.0.315**; the macOS/Linux bridge is unsupported because it uses a Windows timer. API calls execute serially on the GUI thread.
  - Known limitations: "Whole outer/internal straight/curved edge sewing is verified; partial/free sewing… not exposed as verified."
  - "pattern_arrangement controls arrangement parameters, **not a verified world-space transform** or Euler rotation."
  - "Explicit OBJ avatar scale/axes/type=0 is verified; FBX/AVT avatar imports are unverified."
  - "Timeout does not cancel execution."
- **No headless mode:** the API runs scripts *inside* MD; no command-line/headless batch mode was found — [search summary incl. MD API site](https://developer.marvelousdesigner.com/) (snippet-only; this is an absence-of-evidence finding)
- **Linux:** MD became available for Linux in October 2025, including the Python API — [CG Channel](https://www.cgchannel.com/2025/10/marvelous-designer-is-now-available-for-linux/) (snippet-only)
- **Arrangement on custom avatars:** OBJ import has an "Add Arrangement Points & BVs" option that auto-generates bounding volumes and arrangement points (v8.0+). It "works properly only when imported OBJ is either in T or A pose"; otherwise MD reports "Arrangement Points were not fitted to the avatar." Arrangement points can also be saved and loaded as `.ARR` files — [MD support: OBJ Import/Export](https://support.marvelousdesigner.com/hc/en-us/articles/47358223288601-3D-File-OBJ-Import-Export) (snippet-only). CLO documents "Automated Arrangement Point Creation" — [CLO support](https://support.clo3d.com/hc/en-us/articles/360001749467-Automated-Arrangement-Point-Creation) (snippet-only)
- **Licensing and price:** the Personal plan allows commercial use for eligible freelancers and sole proprietors. It costs $39/month or $280 prepaid per year. Enterprise Network Online is $199/seat/month or $2,000/seat/year. The Enterprise Standalone license was discontinued on 2025-12-02 — [MD License Plan](https://support.marvelousdesigner.com/hc/en-us/articles/47358258006425-License-Plan), [MD commercial licensing guide](https://www.marvelousdesigner.com/explore/guide/3d-clothing-software-commercial-licensing-monetization), [Standalone transition](https://support.marvelousdesigner.com/hc/en-us/articles/51136941117849-Marvelous-Designer-Standalone-License-Transition) (snippet-only)
- **MD → UE5:** MD 2025.2 exports USD carrying garment simulation data that a Chaos Cloth Dataflow imports with physical properties preserved. This works with UE 5.6+ and not 5.5 — [MD → MetaHuman USD workflow](https://support.marvelousdesigner.com/hc/en-us/articles/52699135975705-Marvelous-Designer-to-MetaHuman-USD-Garment-Integration-Workflow) (snippet-only)
- MD 2026.0 added a 3D Pencil and lacing — [Digital Production](https://digitalproduction.com/2026/04/15/marvelous-designer-2026-0-adds-3d-pencil-and-lacing/) (snippet-only)

### Inferences
- MD produces the best drape quality of the options here, plus native UVs from patterns and UE5 USD export. For an agent pipeline, though, it is "GUI automation via an embedded interpreter": one MD instance, serial calls, a modal-dialog risk, and no cancellation. Run it on Windows with MD open, and use one of the MCP bridges or a self-written listener.
- **Converting GarmentCode JSON to MD:** panels → DXF-AAMA (DXF has no sewing). Then create seams through the API. Laboon's bridge verifies whole-edge sewing only, which matches GarmentCode stitches when they are whole edges, but GarmentCode often stitches sub-edges. Then place panels. Because API arrangement is not a verified world transform, using GarmentCode's own 3D translation/rotation in MD is uncertain; arrangement points on the auto-generated BVs are the MD-native path.
- **License wording:** nothing found forbids scripting under the Personal license. It covers individuals, so a solo creator automating their own MD seat looks consistent with it. Unverified against the EULA text.
- **CLO:** it shares the Python API per the ysk424 README. A separate CLO API (C++ SDK) and the CLO-SET API exist, but no primary detail could be fetched.

### Gaps
- Exact function list for sewing creation (`CreateSewing…`?), pattern creation from points, `SetArrangement…`, and simulation-step control in the official API — the docs site is blocked. Only MCP READMEs and snippets were read.
- Whether MD's DXF import reads AAMA/ASTM internal lines, grain lines and notches into sewing helpers.
- The MD EULA clause on automation, and whether the MD Linux build supports the same Python plug-in mechanism.
- No found open-source GarmentCode-JSON → MD (DXF/zpac) converter.

---

## Q3. Blender cloth with sewing springs (scripting, add-ons, quality, existing converters)

### Takeaway
Blender is fully scriptable and headless (`blender -b -P script.py`). Its cloth solver supports "sewing springs": loose edges between panel boundary vertices pull together with zero rest length, plus shrink, pressure and collision. So a GarmentCode → Blender converter is straightforward to write. Simulation quality and robustness, especially self-collision and tight fits, are generally below MD, and the solver is CPU-only. Commercial add-ons (Simply Cloth Studio 2.0, Garment Tool) improve sewing UX, but no documented Python API was found. **No existing open-source GarmentCode-to-Blender converter was found**: GarmentCode's legacy path is Maya + Qualoth.

### Cited Findings
- Blender `ClothSettings` include angular/linear bending models. Loose edges used as sewing springs get rest length 0 and stiffness 1, so they come together during the simulation — [Blender Python API ClothSettings](https://docs.blender.org/api/current/bpy.types.ClothSettings.html), [whyoh/blender_clothing_tools](https://github.com/whyoh/blender_clothing_tools) (snippet-only)
- `whyoh/blender_clothing_tools`: helper scripts for building clothing from sewing patterns in Blender cloth. `clothing_edge_finder.py` follows piece boundaries to corners and sews pieces "by joining vertices with edges (how the blender sewing system works)" — [GitHub](https://github.com/whyoh/blender_clothing_tools) (snippet-only)
- Blender bug tracker task "Add sewing seams to cloth simulation" — [developer.blender.org T31269](https://developer.blender.org/T31269) (snippet-only)
- "Seams to Sewing Pattern" add-on by Thomas Kole goes the other way, generating 2D patterns from 3D for cloth sim and real sewing — [GitLab](https://gitlab.com/thomaskole/blender-seams-to-sewing-pattern), [BlenderArtists](https://blenderartists.org/t/seams-to-sewing-pattern-v-0-9-for-2-8-and-2-9/1248713) (snippet-only)
- Simply Cloth Studio 2.0 (Jan 2026) is a full rebuild with "new sewing logic [that] improves edge matching and reduces mesh collapse during gap closure". A "Simply Cut & Sew Patterns" library has 200+ assets — [Digital Production](https://digitalproduction.com/2026/01/27/simply-cloth-studio-2-0-rebuilds-cloth-in-blender/), [Superhive](https://superhivemarket.com/products/simply-cut-sew-patterns) (snippet-only). No Python API is documented in the found sources.
- Garment Tool by Bartosz Styperek is a paid Gumroad add-on — [gumroad](https://bartoszstyperek.gumroad.com/l/GarmentTool?ref=311) (snippet-only). No information on its Python exposure was found.
- GarmentCode's non-Warp sim path is **Autodesk Maya + Qualoth**: `mayapy -m pip install pygarment`, then run `gui/maya_garmentviewer.py` in the Maya console — [GarmentCode docs/Running_Maya_Qualoth.md](https://github.com/maria-korosteleva/GarmentCode/blob/main/docs/Running_Maya_Qualoth.md) (code-verified). Its predecessor, Garment-Pattern-Generator (NeurIPS 2021, 20k+ garments, Maya 2022+), used the same Maya/Qualoth stack — [Garment-Pattern-Generator](https://github.com/maria-korosteleva/Garment-Pattern-Generator) (code-verified README)
- A grep of the GarmentCode repo found no Blender/bpy code — [GarmentCode repo](https://github.com/maria-korosteleva/GarmentCode) (code-verified)

### Inferences
- **Minimal Blender converter (estimated 300–500 lines of Python):**
  1. Parse the GarmentCode JSON (panels: vertices plus edges with curvature).
  2. Resample each stitched edge pair to the same vertex count.
  3. Triangulate each panel with `bmesh` or `triangle`.
  4. Apply the panel's `translation`/`rotation`.
  5. Join into one object and add loose edges between paired stitch vertices.
  6. Enable `use_sewing_springs` with a high sewing force, and add a collision modifier on the body.
  7. Run N frames headless, then apply the modifier at the final frame.
  8. Keep UVs by writing each panel's 2D coordinates into a UV layer *before* simulation.
  
  Alternative: reuse GarmentCode's BoxMesh, which already merges stitches, and skip sewing springs entirely.
- Blender is most valuable here as a **headless post-processor**: weld, remesh, solidify, smooth, weight transfer, FBX export. Use it for draping only when you want to avoid the Warp fork's license.
- Quality expectations vs MD (mass-spring vs MD's solver, weaker self-collision, sensitivity to substeps) are well-known community opinion, but no benchmark was found.

### Gaps
- No quantitative Blender vs MD quality or speed comparison was found.
- Simply Cloth / Garment Tool Python APIs: undocumented in the reachable sources.
- docs.blender.org was blocked, so exact property names (`use_sewing_springs`, `sewing_force_max`, `shrink_min`) were not verified this session.

---

## Q4. Houdini Vellum (Vellum Drape, sewing, hython, licensing)

### Takeaway
Vellum Drape is purpose-built for pattern sewing. It moves stitched points together slowly, then fuses them with welds, and it supports panel-by-panel construction. It can be scripted headless via hython or HDAs. Seams want equal point counts, and headless batch requires a license above Apprentice (Indie with Houdini Engine Indie, or commercial). It is a strong, robust option, but it means learning Houdini, and there is no ready GarmentCode importer.

### Cited Findings
- The Vellum Drape node "provides an automatic way of sewing seams together" and is "a sandbox simulation designed around moving stitched points together slowly before fusing them with welds". Planar Patch from Curve (H17+) plus Vellum Drape lets you build clothing panel by panel — [SideFX: Paneling and draping](https://www.sidefx.com/docs/houdini/vellum/paneling_draping.html), [SideFX H17 Vellum Drape masterclass](https://www.sidefx.com/tutorials/houdini-17-masterclass-vellum-drape/) (snippet-only; sidefx.com blocked)
- Sewing with Vellum Drape "requires the same number of points on the seams". A community WIP tool does pairwise polyline resampling, meshing, then pushes parameters to Vellum Drape for attach constraints and welds — [SideFX forum: WIP Cloth Pattern Tool](https://www.sidefx.com/forum/topic/80721/) (snippet-only)
- **Apprentice:** hython can only read information, not save HIP files or assets — [SideFX forum "What license do I need to run Hython?"](https://www.sidefx.com/forum/topic/72052/), [Apprentice restrictions FAQ](https://www.sidefx.com/faq/question/apprentice-restrictions/) (snippet-only)
- A Houdini Engine Indie license can run Houdini Indie in batch (non-graphical) mode. Apprentice does not allow batch CLI jobs. A paid Houdini Engine license (quoted at $525 workstation / $795 floating) supports hython and batch processing — [Houdini Engine FAQ](https://www.sidefx.com/faq/houdini-engine-faq/), [iRendering license explainer](https://irendering.net/houdini-licenses-explained-for-render-farms-apprentice-indie-core-fx-and-engine/) (snippet-only; the $ figures come from a snippet and may be commercial-tier prices)

### Inferences
- **Pipeline in Houdini:** a Python SOP reads the GarmentCode JSON → curves → Planar Patch from Curve per panel → resample stitch edge pairs to equal counts → Vellum Drape with seam groups → body as collider (any OBJ/FBX, anime proportions fine) → Vellum Post-Process (thickness, smoothing) → export FBX/Alembic with UVs from the flat rest state. The whole graph can be wrapped as an HDA and driven by `hython script.py` from an agent.
- Vellum's PBD/XPBD with welds is generally robust for tight sewing and layering (Vellum supports multiple cloth objects with self-collision). This is reputation, not a benchmark found in this session.
- The cost of learning Houdini plus the license (Indie with batch) is the main barrier for a solo creator who wants zero manual work.

### Gaps
- Current 2026 Houdini Indie price and revenue cap, and whether a plain Houdini Indie purchase includes Engine Indie for hython batch: not verifiable (sidefx.com blocked).
- No public GarmentCode/AAMA-to-Vellum importer was found.

---

## Q5. Research and other simulators (Warp/Newton, Taichi, Genesis, IPC family, ARCSim, DiffCloth, PhysX/Omniverse, Chaos Cloth)

### Takeaway
For a license-clean, GPU, Python-scriptable replacement for the GarmentCode fork, the most relevant engines in 2026 are:
- **NVIDIA Newton** (Apache-2.0, Linux Foundation, successor to `warp.sim`), which ships a **Style3D projective-dynamics garment solver** that takes 2D panel rest shapes and an avatar mesh, plus a VBD solver;
- **libuipc** (Apache-2.0 GPU IPC with Python wheels), for intersection-free cloth and layering.

Codim-IPC is the CPU reference for intersection-free garments (used by research such as Dress-1-to-3), but it is slow and research-grade. DiffCloth, Taichi and Genesis-PBD are general toolkits with no pattern/sewing pipeline. Chaos Cloth in UE5 is a runtime and authoring system, not a flat-pattern sewing engine.

### Cited Findings
- **Newton** is Apache-2.0 and a Linux Foundation project. Its cloth examples include `cloth_style3d`, `cloth_bending`, `cloth_hanging`, `cloth_h1`, `cloth_twist` and `cloth_franka`, plus diffsim cloth and VBD multiphysics examples — [newton-physics/newton README](https://github.com/newton-physics/newton) (code-verified)
- **Newton Style3D example** (`example_cloth_style3d.py`) — [newton/examples/cloth/example_cloth_style3d.py](https://github.com/newton-physics/newton/blob/main/newton/examples/cloth/example_cloth_style3d.py) (code-verified):
  - Loads garment USDs (Women_Skirt, Female_T_Shirt, Women_Sweatshirt) and a Female avatar USD.
  - Calls `style3d.add_cloth_mesh(builder, panel_verts=<UVs as 2D rest shape>, panel_indices=..., vertices=<3D>, indices=..., density, tri_aniso_ke, edge_aniso_ke)`, i.e. **anisotropic stretch and bend with rest shape taken from the flat panels**.
  - Adds the avatar as `builder.add_shape_mesh(..., mesh=Mesh(avatar_points, avatar_indices))`, so any triangle mesh works as a collider.
  - Runs `SolverStyle3D` at 60 fps, 10 substeps, 4 iterations, with CUDA-graph capture.
- `SolverStyle3D` is a "Projective dynamics based cloth solver" (refs include Baraff & Witkin) with a PCG linear solver. `style3d/cloth.py` contains a `compute_sew_v(sew_dist, ...)` kernel that finds vertex pairs within a sewing distance — [newton/_src/solvers/style3d](https://github.com/newton-physics/newton/tree/main/newton/_src/solvers/style3d) (code-verified; sparse clone 2026-10-04)
- Upstream Warp removed `warp.sim` in v1.10.0 (2025-11-02) in favour of Newton — [NVIDIA/warp CHANGELOG](https://github.com/NVIDIA/warp/blob/main/CHANGELOG.md) (code-verified)
- **libuipc:** "GPU-accelerated… unified Incremental Potential Contact" that couples rigid, FEM, cloth and threads. It has Python and C++ APIs, PyPI wheels, Linux and Windows, OBJ/MSH/glTF IO, Baraff-Witkin cloth and shell bending, a CUDA PCG solver, and samples supporting `--headless`. Its own caveat: "IPC's non-penetration guarantees depend on valid initial geometry". Apache-2.0 — [spiriMirror/libuipc README](https://github.com/spiriMirror/libuipc), [LICENSE](https://github.com/spiriMirror/libuipc/blob/main/LICENSE) (code-verified)
- **Genesis** is Apache-2.0. It integrates Rigid, FEM, MPM, PBD (including a PBD cloth tutorial) and a uipc IPC backend (`pip install pyuipc`, Linux/Windows x86, NVIDIA GPU) — [Genesis README](https://github.com/Genesis-Embodied-AI/Genesis) (code-verified)
- **Codim-IPC** (Li, Kaufman, Jiang, SIGGRAPH 2021): built with `python build.py` on Ubuntu/macOS; examples run via `Projects/FEMShell/batch.py` — [ipc-sim/Codim-IPC README](https://github.com/ipc-sim/Codim-IPC) (code-verified)
- **Dress-1-to-3** (2025) uses a "generalized and unified IPC differentiable framework" to "sew and drape the pattern onto the posed human model", optimizing sewing patterns against multi-view images — [Dress-1-to-3 project](https://dress-1-to-3.github.io/), [arXiv 2502.03449](https://arxiv.org/html/2502.03449v1) (snippet-only)
- A related search snippet states that CIPC "is used to ensure proper layering for multi-garment cases by sorting connected components vertically and fitting them sequentially from bottom to top". The originating paper is unclear among the Dress Anyone, AIpparel and Dress-1-to-3 results — [search results incl. Dress Anyone PDF](https://igl.ethz.ch/projects/dress_anyone/DressAnyone_2025.pdf) (snippet-only; attribution uncertain)
- **DiffCloth** (differentiable cloth with dry frictional contact) is C++/CMake, tested on Ubuntu 22.04 and macOS 12, and runs demos via the CLI — [omegaiota/DiffCloth README](https://github.com/omegaiota/DiffCloth) (code-verified)
- **Chaos Cloth:** "Chaos Cloth Asset is a pattern based cloth asset". The Dataflow Cloth Editor is production-ready and the default cloth editor in UE 5.8, with 2D/3D views of the garment — [UE Chaos Cloth Asset API](https://dev.epicgames.com/documentation/unreal-engine/API/PluginIndex/ChaosClothAsset), [Epic forum: Chaos Cloth updates 5.8](https://forums.unrealengine.com/t/tutorial-chaos-cloth-updates-5-8/2729420) (snippet-only). The UE 5.4 workflow starts from a static OBJ/FBX/USD garment mesh plus a skeletal mesh — [Versluis UE5.4 Chaos Cloth](https://www.versluis.com/2024/06/creating-follower-clothing-with-chaos-cloth-in-unreal-engine-5-4/) (snippet-only)
- Forum reports describe Chaos Cloth instability and interpenetration with complex USD multi-layer assets, and "garment seams detach when start simulating" in 5.4 — [apjcriweb PDF](http://apjcriweb.org/content/vol11no6/30.pdf), [Epic forum](https://forums.unrealengine.com/t/5-4-chaos-cloth-issue-garment-seams-deatach-when-start-simulating/1940146) (snippet-only)

### Inferences
- **Newton Style3D** is the closest open-source analogue to a "Style3D/MD-like" garment solver. It already supports 2D-panel rest shapes, anisotropy, an avatar mesh collider and a sewing-pair helper. It is the natural license-clean port target for GarmentCode's box mesh: feed the BoxMesh's 3D vertices plus per-panel 2D UVs as `panel_verts`. It does **not** provide GarmentCode's body-part drag, attachment constraints or panel-to-body assignment, so those would need reimplementing, or would need a better initial placement.
- **libuipc / Codim-IPC** are the choice for a *refinement* pass that guarantees no intersections, for example after a fast XPBD drape or for multi-layer outfits. IPC requires an intersection-free start, so it pairs well with GarmentCode's box mesh after the drag/push phase, or with sequential layering.
- **ARCSim** (adaptive remeshing): not investigated in this session. **Taichi:** no garment pipeline found. **NVIDIA PhysX/Omniverse cloth:** not investigated. These are low priority for this use case.
- Chaos Cloth should be treated as the **UE5 runtime** target (sim config, weight maps, LODs), not as the offline sewing and draping engine.

### Gaps
- No benchmark comparing Newton Style3D/VBD, libuipc and the GarmentCode Warp fork on garment draping time or quality.
- Whether Newton Style3D handles initial panel-body interpenetration robustly (no drag/push constraints seen in the example).
- ARCSim, Taichi cloth, PhysX/Omniverse cloth: not researched (time/tool budget).
- Whether Chaos Cloth Dataflow has a node to sew flat 2D panels (from 2D-pattern-only input) into a draped 3D garment: not confirmed.

---

## Q6. Neural draping (HOOD, ContourCraft, SNUG, DrapeNet, ISP, GarmentNets): arbitrary patterns on arbitrary bodies without training?

### Takeaway
No. These methods do not sew flat patterns. They deform an *already 3D* garment mesh, and most are tied to SMPL bodies or per-garment training. HOOD/ContourCraft (MIT) are the most general: they ship pretrained models and an "inference from arbitrary mesh sequence" notebook, so they can animate or relax a draped garment over a non-SMPL body sequence. They cannot replace the sewing and draping step.

### Cited Findings
- HOOD (CVPR 2023): a 30.09.2023 update added "notebook and config for running inference with any mesh sequence or SMPL pose sequence from a garment mesh in arbitrary pose". Setup still requires downloading SMPL models into `$HOOD_DATA/aux_data/smpl`. MIT license — [Dolorousrtur/HOOD README](https://github.com/Dolorousrtur/HOOD), [LICENSE](https://github.com/Dolorousrtur/HOOD/blob/main/LICENSE) (code-verified)
- ContourCraft (SIGGRAPH 2024) is a HOOD-structured repo that adds learned intersection resolution for multi-garment simulation. Features include SMPL-X support, multi-layer outfit examples, "automatic untangling procedure to combine unrelated garments into outfits", `Inference_from_mesh_sequence.ipynb` and `GarmentImport.ipynb`. Arbitrary-resolution re-meshing is still TODO. MIT license — [Dolorousrtur/ContourCraft README](https://github.com/Dolorousrtur/ContourCraft), [LICENSE](https://github.com/Dolorousrtur/ContourCraft/blob/main/LICENSE) (code-verified)
- DrapeNet: "PBNS and SNUG require training a separate network for each garment, rely on mesh templates… cannot handle meshes with different topologies". DrapeNet trains a single network that drapes multiple garments via a deformation field conditioned on garment latent codes (unsigned distance fields) — [DrapeNet arXiv PDF](https://arxiv.org/pdf/2211.11277) (snippet-only)
- ISP (NeurIPS 2023) represents garments as 2D panels (SDF) with a learned 2D→3D mapping for multi-layer draping — [ISP paper](https://papers.neurips.cc/paper_files/paper/2023/file/7e976afe805026f7d378a583af5ea9a2-Paper-Conference.pdf) (snippet-only)

### Inferences
- For "no training" plus custom anime body plus arbitrary GarmentCode patterns, neural draping is **not viable as the primary engine**. SNUG, DrapeNet and ISP all assume SMPL-parameterized bodies and learned garment spaces.
- **Optional use:** ContourCraft's untangling or its mesh-sequence inference could serve as a fast post-step to resolve layer intersections or preview motion. It still needs a valid 3D garment from a physics sewer first, and its SMPL data dependency (license) must be checked.
- GarmentNets (garment pose estimation for robotics) is unrelated to sewing and draping; not researched further.

### Gaps
- Whether HOOD/ContourCraft inference on a non-SMPL mesh sequence fully avoids the SMPL model files: the README still lists them as required data.
- No training-free neural method that sews 2D patterns onto an arbitrary body was found.

---

## Q7. Auto-arrangement: MD arrangement points vs GarmentCode body-measurement placement; computing landmarks on an anime body

### Takeaway
MD places panels by snapping them to **arrangement points on bounding volumes** around the avatar. These can be auto-generated for an imported OBJ only if it is in T- or A-pose. GarmentCode places panels by **analytic translation/rotation derived from the measurement YAML**, so it never looks at the mesh for placement; the mesh is used only for collision and segmentation. For an anime body, the practical approach is to compute the GarmentCode measurements, and optionally MD-style landmarks, from the rig skeleton plus horizontal mesh slices.

### Cited Findings
- MD auto-generates arrangement points and BVs on OBJ import for T/A-pose avatars, and supports `.ARR` save/load — [MD support: OBJ Import/Export](https://support.marvelousdesigner.com/hc/en-us/articles/47358223288601-3D-File-OBJ-Import-Export) (snippet-only). MD also has "Auto Convert to Avatar" (2024.0+) — [MD support](https://support.marvelousdesigner.com/hc/en-us/articles/47358210639897-Auto-Convert-to-Avatar-Ver-2024-0-and-above) (snippet-only). Commercial packs sell custom BVs and arrangement points for Daz Genesis 3, which suggests manual setup is common for non-standard avatars — [ArtStation](https://www.artstation.com/marketplace/p/7e8/marvelous-designer-custom-bounding-volumes-arrangement-points-for-daz-genesis-3-male-female) (snippet-only)
- GarmentCode garment programs translate panels using `body['height'] - body['head_l'] - ...` and fixed z-offsets (for example, front +30 cm, back −25 cm) — [bodice.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/garment_programs/bodice.py) (code-verified)
- GarmentCode's sim then assigns panels to body parts (closest/ray-hit) and uses body-part drag to resolve initial intersections — [panel_assignment.py](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode/blob/main/warp/collision/panel_assignment.py), [fork README](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode) (code-verified)
- **Research on landmarks:** one method segments a 3D body into 13 parts with an improved Mean Curvature Skeleton, then extracts ≥21 landmarks via KNN, linear models and geometry; another locates side-neck, front-neck, shoulder, bust and waist points with a maximum-distance method on point clouds — [ResearchGate: MCS landmarks for tailoring](https://www.researchgate.net/publication/352967604_Automatic_3D_human_body_landmarks_extraction_and_measurement_based_on_mean_curvature_skeleton_for_tailoring), [ResearchGate: automatic landmark identification](https://www.researchgate.net/publication/251529928_Automatic_body_landmark_identification_for_various_body_figures) (snippet-only)

### Inferences
- **Recommended landmark and measurement procedure for a rigged anime body** (Blender headless or trimesh):
  - Use bone heads/tails for vertical levels: neck base (neck bone head), shoulder points (upper-arm heads), armpit (shoulder joint minus offset), waist (spine bone nearest the minimal torso circumference between chest and hips), hips (max circumference below the waist), crotch (thigh heads).
  - Slice the mesh with horizontal planes at those levels and take the perimeter of the torso-only cross-section loop (excluding arm/leg vertices via the segmentation) to get bust, underbust, waist and hips.
  - Use front/back halves of each loop for back_width, waist_back_width and hip_back_width.
  - Compute arm_pose_angle from the upper-arm bone vector.
  - Set head_l = top-of-head minus chin level, and height = bbox height.
  
  This yields GarmentCode's ~26 keys without any ML.
- Anime bodies often have a large head and small torso. Because GarmentCode computes vertical placement from `height - head_l - ...`, getting head_l right matters. It is safer to set the derived `_waist_level` explicitly, since the code checks for a `_waist_level` key first in `_add_attachment_labels`.
- MD-style arrangement is only needed if you go the MD route. GarmentCode's analytic placement plus body-part drag replaces it.

### Gaps
- No published method specific to anime/stylized body landmarking was found.
- No validation data on how far GarmentCode's analytic placement tolerates non-human proportions before body-part drag fails.

---

## Q8. Post-processing to a game-ready UE5 mesh (remesh, UVs from panels, thickness, anime stylization)

### Takeaway
GarmentCode and MD both output UVs derived from flat panels, which is the key asset. Post-processing (weld seams, optional quad remesh with UV re-projection, solidify for thickness, Laplacian/corrective smoothing to cut folds, weight transfer from the body, FBX export) is easiest as a headless Blender script. For anime-style low-fold garments, raise the bending stiffness in simulation and smooth the body or result afterwards.

### Cited Findings
- GarmentCode box mesh UVs: one island per panel, aspect-ratio preserved (v2.0.2), with seam width and grain texture — [CHANGELOG](https://github.com/maria-korosteleva/GarmentCode/blob/main/CHANGELOG.md), [boxmeshgen.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/boxmeshgen.py) (code-verified)
- GarmentCode provides sim presets `minimal_bending.yaml` and `mid_bending.yaml`; the default bending `garment_edge_ke: 1.0` is commented "Very soft fabric". `enable_body_smoothing` starts the drape on a smoothed body and restores detail gradually — [assets/Sim_props](https://github.com/maria-korosteleva/GarmentCode/tree/main/assets/Sim_props), [garment.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/meshgen/garment.py) (code-verified)
- Newton Style3D takes separate anisotropic stretch and bend stiffness (`tri_aniso_ke`, `edge_aniso_ke`) with the panel UV rest shape — [example_cloth_style3d.py](https://github.com/newton-physics/newton/blob/main/newton/examples/cloth/example_cloth_style3d.py) (code-verified)
- MD can export USD with sim data for Chaos Cloth Dataflow (UE 5.6+) — [MD support](https://support.marvelousdesigner.com/hc/en-us/articles/52699135975705-Marvelous-Designer-to-MetaHuman-USD-Garment-Integration-Workflow) (snippet-only)
- The UE Chaos Cloth Asset workflow: import a static garment mesh plus the skeletal mesh, transfer skin weights, and paint sim weight maps in Dataflow — [Versluis](https://www.versluis.com/2024/06/creating-follower-clothing-with-chaos-cloth-in-unreal-engine-5-4/) (snippet-only)

### Inferences
- **Anime stylization knobs:**
  - raise `garment_edge_ke` (bending) by one to two orders of magnitude;
  - raise `garment_tri_ke` (less stretch);
  - lower `fabric_density`;
  - use `enable_body_smoothing` so cloth does not shrink-wrap small body details;
  - post-sim, apply Blender Corrective Smooth or Laplacian Smooth while pinning seams and hems.
  
  A fold-free look also benefits from a slightly inflated collision body (`body_collision_thickness`).
- **Game topology:** GarmentCode's triangle box mesh resolution is set by `resolution_scale`. For UE5, options are:
  - (a) keep triangles at a reasonable density (simplest; Chaos Cloth works on triangles);
  - (b) a quad remesh (QuadriFlow in Blender, Instant Meshes, or the commercial Quad Remesher), then re-project UVs from the original via Data Transfer.
  
  Panel UVs survive best if you remesh per panel island, in 2D, *before* simulation. That is easy in the box-mesh stage by generating a quad-dominant panel mesh, but it requires modifying the triangulation step.
- **Thickness:** a Solidify modifier, or render-only thickness. Keep the sim mesh single-layered for Chaos Cloth and use a separate thicker render mesh with Proxy Deformer if needed.
- These are standard practice; no source in this session benchmarks them.

### Gaps
- No found open-source tool that does "panel-aware quad remesh preserving pattern UVs" end to end.
- No source on GarmentCode triangle density vs Chaos Cloth performance.

---

## Q9. Recommendation: best engine for an automatic, headless, agent-driven pipeline, and the minimal code to write

### Takeaway
**Primary: GarmentCode/pygarment + its Warp fork**, adapted to the custom body. It is the only option that already runs GarmentCode JSON → placed and sewn → draped OBJ with panel UVs, fully headless, at about 30 s per garment on a 24 GB GPU, with built-in failure checks. Adapting it needs only a prepared body OBJ, a measurement YAML and a segmentation JSON. **The fork's non-commercial license is the one serious caveat.** If outputs will be sold, plan a port of the box-mesh pipeline to **Newton (Style3D/VBD, Apache-2.0)** or **libuipc (IPC, Apache-2.0)**.

Use **headless Blender** for post-processing and UE5 prep. Use **Marvelous Designer** (Python API via an MCP bridge, Windows, GUI open) as an optional "hero-quality" route; it is not truly headless. Houdini Vellum is a capable but costlier, steeper alternative. Neural draping is not suitable as the core.

### Cited Findings
- GarmentCode's headless CLI, box mesh, analytic placement, Warp XPBD drape and panel-UV OBJ output: see Q1 — [GarmentCode](https://github.com/maria-korosteleva/GarmentCode), [NvidiaWarp-GarmentCode](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode) (code-verified); 30 s per garment on RTX 3090 — [arXiv 2405.17609 html](https://arxiv.org/html/2405.17609) (snippet-only)
- The fork is non-commercial only (§3.3) — [LICENSE.md](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode/blob/main/LICENSE.md) (code-verified). Newton is Apache-2.0 with a Style3D garment solver — [Newton](https://github.com/newton-physics/newton) (code-verified). libuipc is Apache-2.0 GPU IPC with Python wheels — [libuipc](https://github.com/spiriMirror/libuipc) (code-verified)
- MD's GUI-bound Python API and its MCP bridges — [ysk424/marvelous-designer-mcp](https://github.com/ysk424/marvelous-designer-mcp), [Laboon2501/MarvelousDesigner-MCP](https://github.com/Laboon2501/MarvelousDesigner-MCP) (code-verified READMEs)

### Inferences
**Comparison matrix** (✓ = good, ~ = partial, ✗ = poor or none; based on the findings above, with quality judgements partly inferential)

| Criterion | GarmentCode + Warp fork | Newton Style3D / VBD | libuipc / Codim-IPC | Marvelous Designer (API) | Blender cloth | Houdini Vellum | Neural (HOOD/CC) |
|---|---|---|---|---|---|---|---|
| Scripting API | Python ✓ | Python ✓ | Python/C++ ✓ / CLI ~ | Embedded Python ~ | bpy ✓ | hou/hython ✓ | Python notebooks ~ |
| Headless | ✓ CLI | ✓ | ✓ (`--headless` samples) | ✗ (GUI must run) | ✓ `-b` | ✓ (needs Indie+Engine / commercial) | ✓ |
| Custom non-SMPL body | ✓ (OBJ + yaml + seg json) | ✓ (any mesh collider) | ✓ | ✓ (OBJ/FBX; auto AP only T/A-pose) | ✓ | ✓ | ~ (SMPL data required) |
| Pattern placement | ✓ analytic from measurements | ✗ write yourself | ✗ write yourself | ✓ arrangement points (API control limited) | ✗ write yourself | ✗ write yourself | ✗ |
| Sewing | ✓ box mesh (pre-merged) | ~ sew-pair helper | ~ (stitch constraints, research code) | ✓ native (API: whole-edge verified) | ✓ sewing springs | ✓ Vellum Drape stitch→weld | ✗ |
| Collision / self-collision | ✓ XPBD + custom fixes | ✓ | ✓✓ intersection-free | ✓✓ | ~ | ✓ | ~ learned |
| Layering | ~ (one spec or sequential) | ~ | ✓ | ✓ | ~ | ✓ | ✓ (ContourCraft) |
| Speed | ✓ ~30 s/garment GPU | ✓ GPU | ~ GPU (libuipc) / ✗ CPU (C-IPC) | ✓ GPU | ✗ CPU | ~ | ✓✓ |
| Output UVs from panels | ✓ | ✓ (panel rest UV) | ~ (your code) | ✓ | ✓ (if you write them) | ✓ | ✗ |
| License | ✗ non-commercial fork (pygarment MIT) | ✓ Apache-2.0 | ✓ Apache-2.0 / research | Paid, $39/mo Personal allows commercial | ✓ GPL tool, outputs yours | Paid (Indie) | ✓ MIT (+SMPL terms) |
| Install effort | ~ build old Warp + CUDA 11.5+ | ✓ pip | ✓ pip wheels / ✗ build | ✓ installer | ✓ | ~ | ~ conda + SMPL |

**Minimal code to write (primary route), roughly 600–1,000 lines total:**
1. `prep_body.py` (Blender `-b -P`): import the anime FBX or VRM-derived mesh. Apply the A-pose (≈45° arms) and record `arm_pose_angle`. Delete hair, eyes and accessories. Close holes or voxel-remesh to watertight. Scale to metres, Y-up, feet at y=0. Export `mybody.obj`.
2. `measure_body.py`: bone-level horizontal slicing → the ~26 GarmentCode measurements → `mybody.yaml`. Set `_waist_level` explicitly if anime proportions confuse the derived formula.
3. `segment_body.py`: map skin weights (arm/leg/head bones) to vertex labels → `mybody_seg.json` with keys `body, left_arm, right_arm, left_leg, right_leg, face_internal`. Patch the single line in `PathCofig` (`self.body_seg = ...`) to point to it.
4. `drape.py`: a wrapper around `BoxMesh` and `run_sim` (as in `test_garment_sim.py`) with `body_name='mybody'`, plus an "anime" sim-props YAML with higher `garment_edge_ke` and body smoothing. Parse the stats (fails, body/self-collision counts) and retry with tweaked params, which suits an agent loop. For layers, simulate the inner garment, then add it as an extra collider mesh (a small patch to `garment.py`'s `build_stage`).
5. `post.py` (Blender `-b`): import `*_sim.obj` (UVs intact), merge by distance, optional smooth or corrective smooth, solidify, data-transfer skin weights from the body, export FBX for UE5 (Chaos Cloth Asset / Dataflow).

**License-clean variant:** replace step 4's simulator with Newton:
- feed the BoxMesh vertices, faces and per-vertex 2D panel coordinates to `style3d.add_cloth_mesh`;
- add the body via `add_shape_mesh`;
- reimplement attachments (pin the waist band near `_waist_level`) and a simple "push-out/drag to assigned body part" pre-phase.

This is estimated at a few hundred extra lines and some tuning; the effort is inferred and has not been tested.

**Quality escalation:** for hero garments, open the same pattern in MD through an MCP bridge (convert to DXF + API sewing). Alternatively, run a libuipc pass on the Warp result to remove residual intersections.

### Gaps
- No direct, citable comparison of drape quality across these engines on the same pattern and body.
- No verified evidence that GarmentCode's garment programs produce good patterns at extreme anime proportions; this needs empirical testing.
- Licensing interpretation (non-commercial fork vs personal hobby use vs eventual commercial release) needs the user's own judgement or legal advice.
