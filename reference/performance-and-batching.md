# Cluster 4 — Rendering Performance, Batching & Culling (Unity 6 / 6000.0)

Facts extracted from Unity 6000.0 Manual + ScriptReference, focused on 2D rendering performance.

## Draw Call Optimization — Overview

- **[concept]** A draw call has two steps: (1) update render state — CPU uses the graphics API to prepare GPU resources (shader code, textures, buffers); this is the most CPU-intensive step; (2) submit the draw call — CPU tells the GPU what to render. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls.html
- **[perf]** Optimizing draw calls reduces how often the CPU sends info to the GPU; benefits: better frame times, more GameObjects per frame, lower power draw (important for battery devices). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls.html
- **[concept]** Three method families: GPU instancing (draw multiple copies of a mesh in one draw call using GPU hardware), render-pipeline techniques (e.g. SRP Batcher — reduce render-state updates via large GPU buffers), and batching (combine meshes that share render state and draw them together). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls.html
- **[workflow]** Core strategy: group data/meshes that share the same render state to reduce how often render state updates. Best practice: use the same materials across GameObjects, use as few shader variants as possible, use texture/sprite atlases. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls.html

## Choosing a Batching Method (decision + priority order)

- **[concept]** Method application PRIORITY per GameObject (a GameObject uses only some methods based on mesh + shader): static batching → SRP Batcher + GPU Resident Drawer/BRG → GPU instancing → dynamic batching. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls-choose-method.html
- **[workflow]** SRP Batcher: recommended (Enable) for URP and HDRP; NOT supported in Built-in Render Pipeline; multithreaded-capable. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls-choose-method.html
- **[workflow]** GPU Resident Drawer (uses GPU hardware instancing) is preferred over BatchRendererGroup for most cases in URP/HDRP; takes precedence over traditional GPU instancing; not supported in Built-in RP. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls-choose-method.html
- **[workflow]** GPU Instancing material checkbox: recommended only for Built-in RP when many instances exist; creates extra shader variants; disabled in URP/HDRP to avoid that cost. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls-choose-method.html
- **[gotcha]** Static batching is incompatible with the GPU Resident Drawer and the BRG API. Dynamic batching is no longer recommended. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls-choose-method.html

## Batching — What It Is

- **[concept]** Batching combines meshes that use the same material so Unity renders them with fewer render-state updates; improves performance, memory efficiency, and reduces CPU overhead. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/DrawCallBatching.html

## Dynamic Batching

- **[gotcha]** HARD LIMITS: meshes must not exceed **900 vertex attributes** OR **300 vertices** to be dynamically batched. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/DrawCallBatching.html
- **[concept]** Dynamic batching transforms mesh vertices to world space on the CPU at runtime, groups similar vertices, and renders them in one draw call. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/DrawCallBatching.html
- **[gotcha]** Prevents batching: different materials; different transforms/scales (can't batch negative scale with positive scale); different lightmap textures or UVs; multipass shaders (only the first pass is batched); skinned mesh renderers (dynamic/static batching supports Mesh Renderers only). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/DrawCallBatching.html
- **[perf]** "For most uses, dynamic batching is no longer recommended, because the CPU overhead might be greater than the overhead of a draw call." Recommended only for lower-end devices. Not supported in HDRP. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/DrawCallBatching.html
- **[workflow]** Enable dynamic batching in Edit > Project Settings > Player > Other Settings; URP/HDRP users should use the SRP Batcher instead. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/static-batching-enable.html

## Static Batching

- **[concept]** Combines multiple static meshes into one shared vertex buffer + index buffer using world-space coordinates; can run at build time or runtime. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/DrawCallBatching.html
- **[gotcha]** LIMIT: each buffer supports up to **64,000 vertices**; Unity creates multiple batches if exceeded. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/DrawCallBatching.html
- **[api]** Runtime static batching: `StaticBatchingUtility.Combine`. Requires "Read/Write enabled" on meshes; then call `Mesh.UploadMeshData` with `markNoLongerReadable = true` to avoid doubling memory. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/static-batching-enable.html
- **[gotcha]** After static batching you can only change position/rotation/scale of the WHOLE batch at runtime, not individual meshes. If a mesh falls outside the camera view, Unity may split the batch to avoid gaps in the vertex buffer. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/static-batching-enable.html
- **[perf]** Static batching uses additional CPU memory to store the combined meshes (even if identical); can heavily impact memory in dense scenes (e.g. forests with many trees). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/static-batching-enable.html
- **[workflow]** When using runtime `StaticBatchingUtility.Combine`, no need to enable static batching in URP/HDRP Asset or Player Settings. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/static-batching-enable.html

## SRP Batcher

- **[concept]** The SRP Batcher combines a sequence of `bind` and `draw` GPU commands into an "SRP batch," reducing render-state changes between draw calls. It does NOT reduce the number of draw calls — it reduces the CPU cost of preparing/dispatching them. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher.html
- **[concept]** Batches by SHADER VARIANT, not by material: you can use as many different materials with the same shader as you want; optimal perf still wants as few shader variants as possible. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher.html
- **[perf]** Mechanism: material content persists in GPU memory (per-object GPU constant buffer for per-object properties). If material content doesn't change, the SRP Batcher makes no render-state changes. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher.html
- **[concept]** Supported by URP, HDRP, and custom SRPs; NOT supported by the Built-in Render Pipeline. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher.html
- **[gotcha]** SHADER compatibility: declare all material properties in a single CBUFFER named `UnityPerMaterial`; declare all built-in engine properties (e.g. `unity_ObjectToWorld`, `unity_WorldTransformParams`) in a single CBUFFER named `UnityPerDraw`. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher-Materials.html
- **[gotcha]** GAMEOBJECT compatibility: must contain a mesh or skinned mesh (can't be a particle); must NOT use a MaterialPropertyBlock; its shader must be SRP Batcher compatible. Adding a MaterialPropertyBlock to a renderer intentionally makes it SRP-Batcher incompatible. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher-Materials.html
- **[workflow]** Verify batching: open Frame Debugger (Window > Analysis > Frame Debugger) > Render Camera > Render Opaques > expand RenderLoopNewBatcher.Draw; a common break reason is "Nodes have different shaders." Many SRP batches with few draw calls each usually means too many shader variants. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher-Profile.html

## GPU Instancing

- **[concept]** GPU instancing uses a single draw call to render multiple GameObjects that share the SAME mesh AND material; each instance can vary properties (color, scale) via per-instance data. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/GPUInstancing.html
- **[gotcha]** HARD LIMIT: `Graphics.DrawMeshInstanced` draws a maximum of **1023 instances at once** (input arrays cannot exceed 1023). — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Graphics.DrawMeshInstanced.html
- **[api]** `Graphics.DrawMeshInstanced` is OBSOLETE in Unity 6.0 — "Use `Graphics.RenderMeshInstanced` instead." Requires `Material.enableInstancing = true` or it throws `InvalidOperationException`. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Graphics.DrawMeshInstanced.html
- **[api]** Per-instance shader data supplied via MaterialPropertyBlock (`SetFloatArray`, `SetVectorArray`, `SetMatrixArray`). Instances are grouped for culling/sorting as one combined unit (no per-instance frustum/occluder culling). — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Graphics.DrawMeshInstanced.html
- **[concept]** Works across all render pipelines with caveats: compatible via Mesh Renderer (Skinned Mesh Renderers excluded), APIs like `Graphics.RenderMeshInstanced`, prebuilt materials with the "Enable GPU Instancing" checkbox, and Shader Graph materials (URP/HDRP only). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/GPUInstancing.html
- **[gotcha]** In URP/HDRP, custom shaders need the SRP Batcher disabled (or made incompatible) for GPU instancing to apply — SRP Batcher takes priority. Built-in RP does not support Shader Graph shaders for instancing. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/GPUInstancing.html
- **[perf]** Performance benefits are better on mobile than desktop; gains depend on GPU overhead for collecting/uploading instance properties. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/GPUInstancing.html

## BatchRendererGroup (BRG)

- **[concept]** BRG API renders large numbers of objects efficiently using DOTS Instancing shaders and a data-oriented way to load instance data. It does NOT require any DOTS packages. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/batch-renderer-group-getting-started.html
- **[workflow]** Setup: SRP Batcher must be enabled; Edit > Project Settings > Graphics, set **BatchRendererGroup variants** to **Keep all**; in URP disable "Strip Unused Variants" (URP Global Settings) to keep DOTS Instancing variants; enable **Allow 'unsafe' Code** in Player Settings. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/batch-renderer-group-getting-started.html

## GPU Resident Drawer (Unity 6, URP/HDRP)

- **[concept]** Automatically uses the BatchRendererGroup API to draw GameObjects with GPU instancing — reduces draw calls and frees CPU time. Updates automatically each frame as GameObjects change/are created. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/gpu-resident-drawer.html
- **[workflow]** Enable steps: (1) Project Settings > Graphics: set "BatchRendererGroup Variants" = "Keep All"; (2) URP Asset: verify "SRP Batcher" enabled; (3) URP Asset: set "GPU Resident Drawer" = "Instanced Drawing"; (4) Universal Renderer: set "Rendering Path" = "Forward+". — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/gpu-resident-drawer.html
- **[gotcha]** Requirements: Forward+ path; graphics APIs supporting compute shaders (NOT OpenGL ES); GameObjects with Mesh Renderer components only. Falls back to non-instanced drawing when requirements aren't met. Increases build times (shader variant compilation); less effective in Scene/Game view than Play mode. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/gpu-resident-drawer.html
- **[perf]** The GPU Resident Drawer runs its own occlusion culling system and supports Dynamic Occlusion. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/gpu-resident-drawer.html

## Sprite Atlas (2D batching enabler)

- **[concept]** A Sprite Atlas packs several sprite textures tightly into a single "atlas" texture; Unity treats them as a single texture, so it only needs one draw call for all sprites in the atlas — reducing batch breaks and draw calls. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/atlas-landing.html
- **[perf]** Sprites that share the same atlas can batch together; the primary goal is to reduce the number of draw calls Unity sends to the GPU. To reduce draw calls in 2D, create a Sprite Atlas. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/atlas-landing.html
- **[gotcha]** With the SRP Batcher, the draw-call COUNT might not decrease, but performance still improves a similar amount. Static atlases suit static/predefined content; dynamic atlases suit frequent runtime changes. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/atlas-landing.html

## Frustum Culling (automatic, per-renderer)

- **[concept]** By default, Cameras perform frustum culling, which excludes all Renderers that do not fall within the Camera's view frustum. Frustum culling does NOT check whether a Renderer is occluded by other GameObjects. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/OcclusionCulling.html
- **[api]** `Renderer.isVisible` = true when the object needs to be rendered in the scene (may be true even when not visible to a camera, e.g. rendering for shadows). — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Renderer.OnBecameVisible.html
- **[api]** `OnBecameVisible()` fires when the renderer becomes visible to any camera; `OnBecameInvisible()` fires when no longer visible by any camera. Message is sent to all scripts attached to the renderer. Use them to avoid computations only needed when the object is visible (e.g. disable a script when off-screen). — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Renderer.OnBecameVisible.html
- **[gotcha]** In the Editor, Scene view cameras also cause `isVisible`/these callbacks to trigger. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Renderer.OnBecameVisible.html

## Occlusion Culling (primarily 3D)

- **[concept]** Occlusion culling additionally removes Renderers that are entirely obscured by nearer Renderers. When enabled, Cameras perform BOTH frustum culling and occlusion culling. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/OcclusionCulling.html
- **[gotcha]** Requires a bake step ("generates data about your Scene in the Unity Editor"); Unity loads that occlusion data into memory at runtime (memory cost). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/OcclusionCulling.html
- **[perf]** Built-in occlusion culling runs runtime CPU calculations that can offset the CPU time it saves; most beneficial when GPU-bound due to overdraw and in structured environments (rooms + corridors). Primarily a 3D technique. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/OcclusionCulling.html

## Frame Debugger

- **[workflow]** Pauses the application on a particular frame and displays the list of rendering events that make up that frame; you can step through each event to see the graphical state at that point. Helps identify rendering artifacts and why batches broke. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/FrameDebugger.html
- **[concept]** Works with URP, HDRP, custom SRP, and the Built-in Render Pipeline. Third-party alternatives (e.g. RenderDoc) available; you can attach supported debuggers to standalone players. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/FrameDebugger.html
- **[gotcha]** Open via Window > Analysis > Frame Debugger. When passes/draw calls change rapidly, the event list may update too fast to read. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher-Profile.html

## Rendering Statistics Window

- **[workflow]** Open via the Stats button in the top-right of the Game view (during Play mode). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/RenderingStatistics.html
- **[concept]** Batches = total number of draw call batches processed during a frame, including static, dynamic, and instance batches. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/RenderingStatistics.html
- **[concept]** Saved by batching = number of draw calls Unity combined into batches; improves when materials are shared across GameObjects. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/RenderingStatistics.html
- **[concept]** SetPass = number of times Unity switches which shader pass it uses to render during a frame (CPU overhead from shader/pass binding). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/RenderingStatistics.html
- **[concept]** Tris = triangles processed per frame; Verts = vertices processed per frame (both critical for low-end hardware). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/RenderingStatistics.html
- **[concept]** Other metrics: FPS; CPU Main (total time to process one frame); CPU Render (time rendering a Game-view frame excluding editor); Screen (resolution + memory); Shadow casters; Visible skinned meshes; Animation/Animator components playing. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/RenderingStatistics.html

## Object Pooling (spawning many objects)

- **[perf]** Object pooling reduces the overhead of repeatedly instantiating/destroying objects, limits allocations/deallocations (less GC pressure), and prevents frame spikes when spawning many objects. Ideal for objects that appear repeatedly but only a few exist at once (e.g. projectiles/bullets). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/performance-reusable-code.html
- **[api]** `UnityEngine.Pool.ObjectPool<T>` is the primary pooling API. Constructor params: createFunc, actionOnGet, actionOnRelease, actionOnDestroy (all `Action<T>`), plus collectionCheck, default capacity, and max size. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/performance-reusable-code.html
- **[api]** `ObjectPool<T>.Get()` retrieves an object (creates one if pool empty, invokes actionOnGet); `ObjectPool<T>.Release(obj)` returns it (invokes actionOnRelease). collectionCheck prevents double-releasing the same instance. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/performance-reusable-code.html
- **[gotcha]** The `UnityEngine.Pool` APIs are NOT thread-safe — callable only from the main thread. `LinkedPool<T>` is an alternative pool implementation. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/performance-reusable-code.html

## 2D-Specific Performance Guidance (synthesized)

- **[perf]** For many-sprite 2D scenes, the highest-leverage optimization is packing sprites into a shared Sprite Atlas so same-atlas sprites batch into one draw call (fewer batch breaks / SetPass calls). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/atlas-landing.html
- **[perf]** In URP, enable the SRP Batcher (default) — it reduces per-draw CPU cost across sprites sharing the same shader variant even when the draw-call count doesn't drop. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher.html
- **[perf]** Sprites are automatically frustum-culled per renderer; use `OnBecameInvisible`/`isVisible` to gate off-screen per-object logic. Occlusion culling is primarily 3D and generally not applicable to flat 2D scenes. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/OcclusionCulling.html

## Sources

- https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/optimizing-draw-calls-choose-method.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/DrawCallBatching.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/static-batching-enable.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher-Materials.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher-Profile.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher-landing.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/GPUInstancing.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Graphics.DrawMeshInstanced.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/batch-renderer-group-getting-started.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/urp/gpu-resident-drawer.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/atlas-landing.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/OcclusionCulling.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Renderer.OnBecameVisible.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/FrameDebugger.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/RenderingStatistics.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/performance-reusable-code.html
