# Cluster 3 — Tilemaps (Unity 6000.0 / Unity 6)

Facts extracted from docs.unity3d.com/6000.0 Manual + ScriptReference. One bullet per fact.
Tag legend: [concept|default|api|gotcha|perf|workflow]

## Tilemaps Overview
- **[concept]** A Tilemap is a GameObject that lets you quickly build 2D levels by painting tiles onto a grid overlay. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tilemaps.html
- **[concept]** The system has four cooperating pieces: the Grid (layout guide / cell shape), the Tilemap component (holds painted tiles), the Tilemap Renderer (draws them), and Tile assets (the paintable sprites). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tilemaps.html
- **[concept]** Supported tile layouts: Rectangular (default), Hexagonal (flat-top or point-top, common in strategy games), and Isometric (2D illusion of 3D depth). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tilemaps.html
- **[concept]** Tilemaps work in all render pipelines that support 2D; requires the 2D Tilemap Editor package installed. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tilemaps.html
- **[workflow]** Tilemaps are parented to a Grid; the Grid determines the shape and size of the cells that make up the tilemap. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/grid-reference.html
- **[concept]** Multiple Tilemaps can sit under one Grid, each acting as a layer; ordering between the layers is controlled by each Tilemap Renderer's Sorting Layer / Order in Layer (and their GameObject order). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html

## Grid Component
- **[concept]** The Grid component is a layout guide that aligns GameObjects (such as Tiles) and transforms Grid cell positions into the GameObject's local coordinates. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/grid-reference.html
- **[default]** Cell Size — sets the dimensions of grid cells. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/grid-reference.html
- **[concept]** Cell Gap — sets spacing between cells. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/grid-reference.html
- **[concept]** Cell Layout — determines cell arrangement. Options: Rectangle, Hexagon, Isometric, Isometric Z as Y. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/grid-reference.html
- **[default]** Cell Swizzle — sets grid orientation. Options: XYZ (default), XZY, YXZ, YZX, ZXY, ZYX. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/grid-reference.html
- **[gotcha]** Cell Swizzle reorders which world axes the grid's X/Y/Z map to; used e.g. to lay a grid flat (XZY) instead of camera-facing (XYZ default). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/grid-reference.html

## Tilemap Component
- **[concept]** Animation Frame Rate — frame rate multiplier for all Tile animations on the Tilemap; actual per-tile rate = this value × the tile animation's own speed (e.g. value 2 = double speed). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-reference.html
- **[default]** Color — tints all tile sprites; default is white (no tint). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-reference.html
- **[default]** Tile Anchor — position of tiles relative to the bottom-left of the cell; default (0.5, 0.5, 0) = cell center. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-reference.html
- **[concept]** Orientation — the plane tiles align to. Options: XY (default, faces camera), XZ, YX, YZ, ZX, ZY, and Custom. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-reference.html
- **[concept]** When Orientation = Custom, sub-properties Offset, Rotation, and Scale (the Orientation Matrix) become editable. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-reference.html
- **[workflow]** The Tilemap component's Info section lists contained tiles/sprites and lets you highlight them in the Project window. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-reference.html

## Tilemap Renderer — Mode (Chunk vs Individual)
- **[default]** Mode = the way the renderer batches tiles. Enum: Chunk (default), Individual, SRP Batch. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[perf]** Chunk (default): batches multiple tiles into grouped chunks and renders them together as a single sort item in the 2D Transparent Queue — reduces draw calls, best for performance. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.TilemapRenderer.Mode.Chunk.html
- **[gotcha]** In Chunk mode other renderers CANNOT render between any part of the Tilemap — external sprites cannot interweave with tile sprites at differing depths, so a single Chunk-mode Tilemap cannot overlap/Y-sort with characters. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/tilemap-renderer-isometric-modes.html
- **[gotcha]** In Chunk mode the renderer cannot sort tiles from multiple textures individually; if tiles come from several textures/atlases they may not render consistently — pack sprites into a single atlas to avoid incorrect sorting. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/tilemap-renderer-isometric-modes.html
- **[concept]** Individual: sends each tile sprite to be rendered individually, sorting each sprite by its position + the renderer's sorting properties — this lets a character sprite render in-between tile/obstacle sprites (correct overlap / Y-sort). — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.TilemapRenderer.Mode.Individual.html
- **[perf]** Individual mode reduces performance vs Chunk because there is more overhead rendering each sprite separately; use it only when you need sprites to interleave with tiles by depth. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/tilemap-renderer-isometric-modes.html
- **[concept]** SRP Batch: sends batchable sprites in chunks that can be batched by the Scriptable Render Pipeline (SRP) batcher. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.TilemapRenderer.Mode.html
- **[gotcha]** Plain Chunk mode is incompatible with the SRP Batcher; use SRP Batch mode (or Individual) if you want SRP batching. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html

## Tilemap Renderer — Sorting, Culling & Material
- **[default]** Sort Order — the order the renderer starts sorting tiles. Options: Bottom Left (default), Bottom Right, Top Left, Top Right. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[default]** Detect Chunk Culling Bounds — Auto (default; estimates bounds from tile sprites) or Manual (you supply a custom boundary). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[concept]** Chunk Culling Bounds — extends the culling boundary in units; only editable when Detect Chunk Culling Bounds = Manual (needed when tile sprites extend beyond their cells so chunks aren't culled too early). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[perf]** Max Chunk Size — number of tiles per chunk the renderer groups together (Chunk mode); tuning it trades batching efficiency vs culling granularity. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[perf]** Chunk culling: in Chunk mode the Tilemap only renders the chunks currently visible (inside the camera + culling bounds), skipping off-screen chunks. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[concept]** Sorting Layer — determines render order per the Tags and Layers window. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[concept]** Order in Layer — sublayer value within a Sorting Layer; lower renders first. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[default]** Material — defaults to Sprite-Lit-Default; can be swapped in the Inspector. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[concept]** Mask Interaction — how the tilemap interacts with Sprite Masks. Options: None, Visible Inside Mask, Visible Outside Mask. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- **[concept]** Rendering Layer Mask — assigns rendering layers (for URP compatibility). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html

## Tile Assets & Scriptable Tiles
- **[concept]** All tiles placed on a Tilemap must inherit from UnityEngine.Tilemaps.TileBase; TileBase gives the tilemap a fixed API to report rendering properties. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tiles-for-tilemaps/scriptable-tiles/scriptable-tiles.html
- **[concept]** The built-in Tile class is the simplest ready-made asset; scriptable tiles subclass TileBase for custom behavior. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tiles-for-tilemaps/scriptable-tiles/scriptable-tiles.html
- **[api]** GetTileData(Vector3Int position, ITilemap tilemap, ref TileData tileData) — determines how the tile looks (sprite, color, transform, gameObject, flags, colliderType). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tiles-for-tilemaps/scriptable-tiles/scriptable-tiles.html
- **[api]** GetTileAnimationData — supplies animation data (sprite frames, speed, start time) for animated tiles. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tiles-for-tilemaps/scriptable-tiles/scriptable-tiles.html
- **[api]** RefreshTile — controls which other tiles re-run GetTileData when a tile is painted (used by rule tiles to refresh neighbors). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tiles-for-tilemaps/scriptable-tiles/scriptable-tiles.html
- **[api]** StartUp — called when the tile is first placed / instantiated on the tilemap; used to set up a spawned GameObject. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tiles-for-tilemaps/scriptable-tiles/scriptable-tiles.html
- **[concept]** TileData fields: sprite, color, transform (Matrix4x4), gameObject, flags (TileFlags), colliderType. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tiles-for-tilemaps/scriptable-tiles/scriptable-tiles.html
- **[concept]** Tile ColliderType enum: None, Sprite, Grid — determines the collider shape generated by Tilemap Collider 2D. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d.html
- **[concept]** Extra tile types (Animated Tile, Rule Tile, Rule Override Tile) plus custom brushes come from the 2D Tilemap Extras package. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/com.unity.2d.tilemap.extras.html
- **[workflow]** A Tile Template Asset (ScriptableObject) defines how Unity creates Tiles from a texture — e.g. build one Rule Tile from all sprites in a sheet instead of one Tile per sprite. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/com.unity.2d.tilemap.extras.html

## Tile Palette Workflow
- **[workflow]** Open the Tile Palette window via Window > 2D > Tile Palette; use the New Palette dropdown / Create New Palette to make one. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/create-tilemap.html
- **[workflow]** To paint: set Active Target to the tilemap, select Paint with Active Brush, pick a tile (highlighted with a white outline), then paint in the Scene view; click-drag selects multiple tiles. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/create-tilemap.html
- **[workflow]** Toolbar tools: Select, Move, Paint with Active Brush, Erase with Active Brush, plus Box Fill and Flood Fill for larger areas. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/create-tilemap.html
- **[workflow]** Shortcuts: hold Shift while painting to delete tiles; hold Ctrl (Cmd on Mac) to pick tiles directly from the Scene view. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/create-tilemap.html
- **[concept]** The active brush controls painting behavior; the Default Brush is used initially, and other brush types are chosen from the Brush Inspector dropdown. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tile-palettes/brushes/active-brush.html

## Isometric Tilemaps
- **[concept]** Isometric tilemaps use a 2D grid to simulate a 3D environment and create the illusion of height and depth; common in strategy games. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/isometric-tilemap-landing.html
- **[concept]** Two ways to add 3D height on an isometric map: use multiple stacked tilemaps, OR use a single Isometric Z as Y tilemap (Z position represents height). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/isometric-tilemap-landing.html
- **[gotcha]** For an Isometric Z as Y tilemap, set the 2D renderer's Transparency Sort Mode to Custom Axis and the Transparency Sort Axis to (0, 1, -0.26) so tiles with higher Z draw correctly (in front). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/sort-sprites-custom-sorting-axis.html
- **[gotcha]** The -0.26 Z component comes from (Cell Size Y × -0.5) - 0.01; e.g. cell size 0.5 → 0.5 × -0.5 - 0.01 = -0.26. Adjust it if your cell size differs. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/tilemap-renderer-isometric-modes.html
- **[gotcha]** Wrong Z-axis sort config makes tiles at higher 3D height incorrectly render behind lower ones — the classic iso sorting bug. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/tilemap-renderer-isometric-modes.html
- **[concept]** Grid Cell Layout for iso is either Isometric or Isometric Z as Y; the Z-as-Y variant is what enables per-height sorting. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/grid-reference.html

## Hexagonal Tilemaps
- **[workflow]** Create a hex tilemap the same way as a normal tilemap (GameObject > 2D Object) but choose a Hexagonal option; Unity provides Hexagonal Point Top and Hexagonal Flat Top. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/hexagonal-tilemaps.html
- **[concept]** Grid Cell Layout = Hexagon for hex tilemaps; hexes can be point-top (one unit tall, points up/down) or flat-top (one unit wide, flat edges up/down). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/hexagonal-tilemaps.html
- **[gotcha]** Hexagonal tilemaps use an offset coordinate system — alternate rows or columns are offset by half a cell to align to the hex grid. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/hexagonal-tilemaps.html

## Tilemap Collider 2D (+ Composite Collider 2D)
- **[concept]** Tilemap Collider 2D automatically generates a collider shape for each tile in the tilemap, based on each tile's ColliderType (None / Sprite / Grid). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d.html
- **[perf]** By default it updates collider shapes during LateUpdate and batches multiple tile changes together for performance. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d.html
- **[default]** Max Tile Change Count — default 1000; sets how many tile changes accumulate before a full regeneration; lower = more frequent partial regeneration. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d-reference.html
- **[concept]** Extrusion Factor — (only relevant with a Composite Collider 2D attached) extends collider shapes outward so neighbors merge cleanly. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d-reference.html
- **[perf]** Use Delaunay Mesh — toggles Delaunay triangulation for complex tile shapes; improves shape quality but reduces performance. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d-reference.html
- **[concept]** Other properties: Material (2D physics material), Is Trigger, Used by Effector, Offset (units). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d-reference.html
- **[concept]** Composite Operation — how shapes combine when a Composite Collider 2D is attached: None (default), Merge, Intersect, Difference, Flip; Composite Order sets calculation priority (lower first). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d-reference.html
- **[concept]** Layer Overrides group: Layer Override Priority, Include Layers, Exclude Layers, Force Send Layers, Force Receive Layers, Contact Capture Layers, Callback Layers. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d-reference.html
- **[perf]** Add a Composite Collider 2D to the same GameObject (and set the Tilemap Collider 2D to be used by it) to merge all adjacent tile colliders into a single shape — greatly reduces physics overhead vs many per-tile colliders. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d.html
- **[gotcha]** Composite Collider 2D requires a Rigidbody 2D on the GameObject (set to Static for level geometry). — src: https://docs.unity3d.com/6000.0/Documentation/Manual/2d-physics/collider/composite-collider/composite-collider-2d-landing.html
- **[workflow]** Disable collision on specific tiles by setting the tile asset's Collider Type to None; use Sprite to edit a custom collision shape in the Sprite Editor's Custom Physics Shape. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d.html
- **[api]** TilemapCollider2D.ProcessTilemapChanges() — forces an immediate collider rebuild instead of waiting for LateUpdate. — src: https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d.html

## Tilemap Scripting API
- **[api]** Tilemap.cellBounds — boundaries of the Tilemap in cell size (BoundsInt). — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** Tilemap.layoutGrid — the Grid associated with this Tilemap. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** Tilemap.size / origin — size (in cells) and origin (cell position) of the tilemap. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** Tilemap.tileAnchor / orientation / animationFrameRate / color — scripting access to the matching Inspector fields. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** SetTile(Vector3Int, TileBase) — sets a single tile at cell coordinates. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** SetTiles(Vector3Int[], TileBase[]) — sets an array of tiles at given coordinates. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** SetTilesBlock(BoundsInt, TileBase[]) — fills a bounds region with an array of tiles (fast bulk write). — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** GetTile(Vector3Int) — returns the tile at cell coordinates; HasTile(Vector3Int) tests presence. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** GetTileFlags / SetTileFlags — read/write the TileFlags at a position. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** GetUsedTilesCount / GetUsedTilesNonAlloc — number of distinct tiles used; fill an array with them. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** CompressBounds() — shrinks origin and size to the region where tiles actually exist. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** ClearAllTiles() — clears all placed tiles; RefreshTile(Vector3Int) refreshes render/animation/component data for one tile. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** CellToWorld / WorldToCell — convert between cell and world coordinates; GetCellCenterWorld returns the logical center of a cell in world space. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** FloodFill(Vector3Int, TileBase) and BoxFill(Vector3Int, TileBase, startX,startY, endX,endY) — programmatic flood/box fills. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- **[api]** TilemapRenderer.mode (Mode: Chunk/Individual/SRPBatch), sortOrder, detectChunkCullingBounds (Auto/Manual), chunkCullingBounds, chunkSize, maskInteraction — scripting equivalents of the Inspector fields. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.TilemapRenderer.html
- **[perf]** TilemapRenderer.maxChunkCount (max chunks cached in memory) and maxFrameAge (frames unused chunks are kept) tune the chunk cache. — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.TilemapRenderer.html

## Sources
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tilemaps.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/grid-reference.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-reference.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-renderer-reference.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/tilemap-renderer-isometric-modes.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/isometric-tilemap-landing.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/isometric-tilemaps/renderer/sort-sprites-custom-sorting-axis.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/hexagonal-tilemaps.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tiles-for-tilemaps/scriptable-tiles/scriptable-tiles.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tiles-for-tilemaps/scriptable-tiles/create-scriptable-tile.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/com.unity.2d.tilemap.extras.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/create-tilemap.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/tile-palettes/brushes/active-brush.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/tilemaps/work-with-tilemaps/tilemap-collider-2d-reference.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/2d-physics/collider/composite-collider/composite-collider-2d-landing.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.Tilemap.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.TilemapRenderer.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.TilemapRenderer.Mode.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.TilemapRenderer.Mode.Chunk.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Tilemaps.TilemapRenderer.Mode.Individual.html
