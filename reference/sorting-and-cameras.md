# Cluster 2 — 2D Sorting, Depth & Cameras (Unity 6 / 6000.0)

Ledger of extracted facts. Tags: [concept|default|api|gotcha|perf|workflow].

## 2D Draw Order Resolution (the core rules)

- **[concept]** Unity resolves 2D draw order by evaluating these criteria IN ORDER: (1) Sorting Layer, (2) Order in Layer (sortingOrder), (3) material Render Queue, (4) Distance to Camera, (5) internal shader/material batch order. Each subsequent criterion is only a tie-breaker when the previous ones are equal — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sort-sprites/sort-sprites.html
- **[concept]** (1) Sorting Layer: a GameObject renders EARLIER (behind) if its Sorting Layer is HIGHER in the Tags and Layers list — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sort-sprites/sort-sprites.html
- **[concept]** (2) Order in Layer: within one layer, LOWER Order in Layer values render EARLIER (behind); higher values render later (in front). e.g. Order -1 renders behind Order 3 — src: https://docs.unity3d.com/6000.0/Documentation/Manual/2d-renderer-sorting.html
- **[concept]** (3) Render Queue: an object renders earlier if its material's Render Queue value is lower — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sort-sprites/sort-sprites.html
- **[concept]** (4) Distance to Camera: when Sorting Layer, Order in Layer, and Render Queue all match, Unity sorts by distance from the camera. Objects FURTHER from the camera render EARLIER (behind). The distance calc depends on the Camera's Projection type, the Transparency Sort Mode, and each Sprite's Sort Point — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sort-sprites/sort-sprites.html
- **[concept]** (5) Final tie-break: objects with identical shader+material are batched together; the order within that batch is NOT guaranteed and cannot be controlled — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sort-sprites/sort-sprites.html
- **[gotcha]** Because ALL sprites share the "Default" layer, Order in Layer 0, and same render queue out of the box, Distance to Camera becomes the primary differentiator by default — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sort-sprites/sort-sprites.html
- **[concept]** For orthographic distance sorting, Unity primarily uses the Z-position of the GameObject's Transform; for perspective it uses distance from camera position to object (sort point) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/2d-renderer-sorting.html

## Sorting Layers

- **[default]** Every project has a built-in sorting layer named **"Default"**; all 2D GameObjects start on it — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SortingLayer.html
- **[default]** By default all 2D GameObjects are on the "Default" sorting layer with Order in Layer = **0** — src: https://docs.unity3d.com/6000.0/Documentation/Manual/2d-renderer-sorting.html
- **[workflow]** Add a sorting layer: Edit > Project Settings > Tags and Layers > Sorting Layers section > Add (+) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/2d-renderer-sorting.html
- **[concept]** Unity renders sorting layers in list order top-to-bottom; a layer higher in the list renders behind, a layer lower in the list (added later, at bottom) becomes the FRONT layer — src: https://docs.unity3d.com/6000.0/Documentation/Manual/2d-renderer-sorting.html
- **[api]** `SortingLayer` struct: `.id` (unique, NOT an ordered running value), `.name` (from TagManager), `.value` (relative value indicating sort order vs other layers) — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SortingLayer.html
- **[api]** `SortingLayer` statics: `SortingLayer.layers` (all project layers), `GetLayerValueFromName()`, `GetLayerValueFromID()`, `NameToID()`, `IDToName()` (returns "<unknown layer>" if invalid), `IsValid()`, and `onLayerAdded`/`onLayerRemoved` delegates. Use GetLayerValueFrom* + CompareTo to determine relative order — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SortingLayer.html

## Order in Layer (SpriteRenderer.sortingOrder)

- **[api]** `SpriteRenderer.sortingOrder` = "Renderer's order within a sorting layer" (this is the "Order in Layer" field). Lower renders behind, higher renders in front — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- **[api]** Also `SpriteRenderer.sortingLayerID` (unique ID of layer) and `SpriteRenderer.sortingLayerName` (string) — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- **[workflow]** Sorting Layer + Order in Layer fields live in the "Additional Settings" section of the renderer component (e.g. Sprite Renderer, Tilemap Renderer) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/2d-renderer-sorting.html

## Sprite Sort Point

- **[api]** `SpriteRenderer.spriteSortPoint` "Determines the position of the Sprite used for sorting the SpriteRenderer" — chooses which point is used in distance-to-camera calculation. Enum options: **Center** and **Pivot** — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- **[gotcha]** Sort Point matters for distance-based sorting (e.g. Y-sort). Using Pivot lets you place the sort reference at a character's feet so overlapping sprites order by their base position — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sort-sprites/sort-sprites.html

## Sorting Group Component

- **[concept]** Sorting Group makes a set of GameObjects render "as one unit on a single sorting layer and sublayer" — prevents prefab instances / multi-part characters from interleaving with other renderers — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sort-sprites/sort-sprites.html
- **[api]** SortingGroup properties: `sortingLayerID`, `sortingLayerName`, `sortingOrder`, and `sortAtRoot` (bool: ignore any parent SortingGroup and sort this + descendants at root level against other root renderers) — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html (SortingGroup ScriptReference)
- **[concept]** Inspector fields: Sorting Layer, Order in Layer ("Unity renders lower values before higher values"), and Sorting Type with options **Default** (sort with peers at same hierarchy level), **Sort at Root** (sort at top level, bypass parent sorting groups), **Sort 3D as 2D** (sort at top level ignoring z-values of 3D objects) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sorting-group/sorting-group-reference.html
- **[workflow]** Add: select the parent GameObject of the group > Add Component > Rendering > Sorting Group > set its Sorting Layer + Order in Layer. Children keep their relative order but all render at the group's layer/sublayer — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sorting-group/use-sorting-groups.html
- **[gotcha]** Inside a Sorting Group, Unity IGNORES each child's Distance to Camera; it uses the position of the GameObject holding the Sorting Group for the whole group's distance — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sorting-group/use-sorting-groups.html
- **[concept]** Sorting Groups can be NESTED: a nested group renders internally first, then sorts as a single entity within its parent group — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sorting-group/use-sorting-groups.html
- **[gotcha]** If instances still mix after adding sorting groups, give each Sorting Group instance a different Order in Layer — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sorting-group/use-sorting-groups.html
- **[perf]** The system supports a maximum of 4,096 sorting groups + renderers combined — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html (SortingGroup ScriptReference)

## Transparency Sort Mode & Transparency Sort Axis (Y-sort setup)

- **[api]** `Camera.transparencySortMode` = "Transparent object sorting mode." Enum `TransparencySortMode` values: **Default**, **Perspective**, **Orthographic**, **CustomAxis** — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-transparencySortMode.html
- **[concept]** Default behavior: perspective cameras sort transparent objects by distance from camera position to object center; orthographic cameras sort by distance along the view direction — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-transparencySortMode.html
- **[api]** `Camera.transparencySortAxis` = "An axis that describes the direction along which the distances of objects are measured for sorting." Only takes effect when TransparencySortMode (on Camera or GraphicsSettings) = CustomAxis — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-transparencySortAxis.html
- **[api]** `TransparencySortMode.CustomAxis`: transparent objects sorted by distance along a custom axis; e.g. axis (0,1,0) makes renderers sort to the back as they go UP in Y — common in 2.5D games — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/TransparencySortMode.CustomAxis.html
- **[workflow]** Recommended Y-sort setup for 2.5D/isometric: set TransparencySortMode = CustomAxis and transparencySortAxis = **(0.0f, 1.0f, 0.0f)**, which sorts SpriteRenderers along the vertical screen axis — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-transparencySortAxis.html
- **[workflow]** Project-wide (editor) path: Edit > Project Settings > Graphics > Camera Settings > set **Transparency Sort Mode = Custom Axis** and **Transparency Sort Axis = (0, 1, 0)**. (In URP, the equivalent is set on the URP asset / 2D Renderer.) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/sort-sprites-custom-sorting-axis.html
- **[concept]** With Transparency Sort Axis (0,1,0): all Renderers at a HIGHER Y position render FIRST and appear BEHIND Renderers at a lower Y position (objects lower on screen draw in front) — this is the Y-sort effect — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/sort-sprites-custom-sorting-axis.html
- **[gotcha]** For ISOMETRIC Tilemaps specifically: set Transparency Sort Axis to Y=1, **Z=-0.26** (not Z=0). With Z=0 tiles sort INCORRECTLY; Z=-0.26 makes tiles sort correctly — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/sort-sprites-custom-sorting-axis.html
- **[gotcha]** For a plain top-down/2.5D game where sprites are on a flat plane at the same Z, custom-axis Y-sort lets overlapping sprites order by world Y without needing to offset Z per sprite; combine with SpriteRenderer Sort Point = Pivot at the sprite's feet — src: https://docs.unity3d.com/6000.0/Documentation/Manual/2d-renderer-sorting.html

## TilemapRenderer / SpriteRenderer Interplay

- **[concept]** Tilemap Renderer exposes the same Sorting Layer + Order in Layer fields as Sprite Renderer, so tilemaps and sprites sort against each other using the shared layer/order/distance rules — src: https://docs.unity3d.com/6000.0/Documentation/Manual/2d-renderer-sorting.html
- **[gotcha]** For per-tile Y-sorting on an isometric tilemap, the Tilemap Renderer Mode should be **Individual** (each tile sorted independently) rather than **Chunk** (tiles batched as one), so individual tiles participate in the custom-axis distance sort — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/sort-sprites-custom-sorting-axis.html
- **[concept]** Custom Sorting Axis is commonly used with Isometric Tilemaps to sort/render tile sprites correctly (Edit > Project Settings > Graphics > Transparency Sort Axis) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/sort-sprites-custom-sorting-axis.html

## Camera: Orthographic vs Perspective

- **[api]** `Camera.orthographic` (bool): when true, viewing volume defined by orthographicSize; when false, defined by fieldOfView — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-orthographic.html
- **[api]** `Camera.orthographicSize` = HALF the vertical viewing volume height in world units. Total view height = orthographicSize * 2. Width is derived from orthographicSize * aspect ratio — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-orthographicSize.html
- **[gotcha]** orthographicSize only works when projection is orthographic; Unity IGNORES it otherwise (use fieldOfView for perspective) — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-orthographicSize.html
- **[api]** Example: `cam.orthographic = true; cam.orthographicSize = 5.0f;` gives a 10-unit-tall view — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-orthographicSize.html
- **[concept]** orthographicSize <-> PPU <-> screen relationship (derived): screen height in world units = 2 * orthographicSize. For pixel-perfect display, orthographicSize = (screenHeightInPixels / 2) / PixelsPerUnit. e.g. 1080px tall screen at 100 PPU -> orthographicSize = 5.4. (See Pixel Perfect Camera for automated handling.) — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-orthographicSize.html

## Pixel Perfect Camera (com.unity.2d.pixel-perfect / URP)

- **[concept]** Pixel Perfect Camera makes pixel art appear crisp and stable at different resolutions, keeping sprites aligned to a consistent pixel grid; included by default in the Universal 2D template — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/2d-pixelperfect-ref.html
- **[api]** Field **Asset Pixels Per Unit**: how many pixels of the sprite image correspond to one world-space unit (the grid size is based on this) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/2d-pixelperfect-ref.html
- **[api]** Field **Reference Resolution** (X, Y): the screen resolution the sprites are designed for (e.g. 320x180); scenes are scaled up from this to preserve pixel art — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/2d-pixelperfect-ref.html
- **[api]** Field **Crop Frame**: crops the viewport with black bars to hide areas beyond the Reference Resolution aspect ratio. Options: **None**, **Pillarbox** (left/right bars), **Letterbox** (top/bottom bars), **Windowbox** (all sides), **Stretch Fill** — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/2d-pixelperfect-ref.html
- **[api]** Field **Grid Snapping**: how Unity snaps/scales sprites. Options: **None**; **Pixel Snapping** (renders sprites aligned to the pixel grid, preventing sub-pixel movement); **Upscale Render Texture** (renders scene to a temp texture close to Reference Resolution keeping full-screen aspect, then upscales — produces unaliased, unrotated pixels) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/2d-pixelperfect-ref.html
- **[api]** Field **Filter Mode**: how Unity stretches the view when Crop Frame = Stretch Fill. Options: **Point**, **Retro AA** — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/2d-pixelperfect-ref.html
- **[api]** Read-only **Current Pixel Ratio**: the ratio of sprites in Game view to their original size — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/2d-pixelperfect-ref.html
- **[gotcha]** Upscale Render Texture guarantees no aliasing / no rotated pixels but renders at a lower internal resolution; Pixel Snapping keeps world-space position but does not upscale — choose based on whether you need crisp rotation-free pixels vs. smooth camera movement — src: https://docs.unity3d.com/6000.0/Documentation/Manual/urp/2d-pixelperfect-ref.html

## Y-Sort Setup — Quick Recipe (overlapping sprites order by world Y)

- **[workflow]** Step 1: Edit > Project Settings > Graphics > Camera Settings (or URP asset/2D Renderer) -> Transparency Sort Mode = **Custom Axis**.
- **[workflow]** Step 2: Set Transparency Sort Axis = **(0, 1, 0)** — sprites with higher Y render behind those with lower Y (lower-on-screen draws in front).
- **[workflow]** Step 3: On each Sprite Renderer, set Sort Point = **Pivot** and put the pivot at the sprite's base/feet so ordering keys off the ground contact point.
- **[workflow]** Step 4 (keep sprites on same layer/order): leave all overlapping sprites on the same Sorting Layer + same Order in Layer so the custom-axis distance sort is the deciding factor.
- **[workflow]** Step 5 (isometric tilemaps only): use axis Y=1, **Z=-0.26**, and set Tilemap Renderer Mode = **Individual**.
- **[workflow]** Step 6 (multi-part characters): wrap each character's sprites in a Sorting Group so the character sorts as one unit by its root position (Sort Point pivot).

## Sources
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sort-sprites/sort-sprites.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/2d-renderer-sorting.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sorting-group/sorting-group-reference.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sorting-group/use-sorting-groups.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SortingLayer.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Rendering.SortingGroup.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-transparencySortMode.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-transparencySortAxis.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/TransparencySortMode.CustomAxis.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-orthographic.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Camera-orthographicSize.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/sort-sprites-custom-sorting-axis.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/urp/2d-pixelperfect-ref.html
