---
name: unity-2d-expert
description: Expert guidance for Unity 6 (URP) 2D games — sprites & pixels-per-unit, sorting/draw-order and Y-sort, tilemaps (Chunk vs Individual vs SRP-Batch, chunk culling, colliders), rendering performance & batching (SRP Batcher, GPU instancing, sprite atlas), 2D physics, 2D lighting, and pixel-art setup. Grounded in the Unity 6000.0 docs with editor-confirmed observations flagged where they diverge. Use when working on Unity 2D rendering, tilemaps, sprite sorting/overlap/Y-sort, draw-call or many-sprite performance, pixel-art import, or 2D scene setup. Triggers: Unity 2D, tilemap, TilemapRenderer, SpriteRenderer, sorting layer/order, Y-sort, pixels per unit / PPU, sprite atlas, 2D draw calls/batching, 2D lighting (Light 2D), Pixel Perfect Camera, 2D physics.
---

# Unity 2D Expert — Unity 6 (6000.x) / URP

Actionable rules for building performant Unity 6 2D games. The bullets here are the
**decision layer**; every claim is expanded with exact fields, enums, defaults, and
source URLs in `reference/`:

- `reference/sprites.md` — sprites, PPU, Sprite Renderer, atlas, 9-slice, masks, import
- `reference/sorting-and-cameras.md` — draw order, Y-sort, sorting groups, ortho camera, Pixel Perfect
- `reference/tilemaps.md` — Grid/Tilemap/Renderer, Chunk/Individual, culling, colliders, scripting
- `reference/performance-and-batching.md` — batching methods + hard limits, culling, pooling
- `reference/physics-lighting-animation.md` — Rigidbody/Collider 2D, Light 2D, Sprite Shape, 2D Animation
- `reference/observed-vs-documented.md` — **editor-confirmed observations, incl. divergences**

## Working principle: doc-first, but trust the confirmed editor

Base every answer on the docs. **Where a real editor observation diverges from a
documented default or common assumption, trust the observation — but only after
confirming it in-editor** (import the asset / read the component / measure it), and
record it in `reference/observed-vs-documented.md` tagged `[diverges — REVIEW]`.
Confirmed divergences worth knowing up front:

- **2D-project texture defaults ARE pixel-art-friendly.** A fresh PNG in a Unity 6
  **2D URP** project imports as `Sprite` type, **Point** filter, **mips off**, **Clamp**
  wrap, **Uncompressed**, **PPU 100** (2D Behavior Mode) — verified by importing a
  throwaway PNG. You rarely need to force Point/no-mip. The real gotcha is
  **`maxTextureSize` (default 2048) silently downscaling large atlases** (a 4096²
  atlas → 2048 → misaligned tiles). Confirm max size on big sheets.
- **A single Tilemap cannot overlap sprites** (1 tile/cell). Overlapping multi-tile
  props need **Individual mode** (per-tile sort, still 1/cell) or **N stacked
  tilemaps**, or SpriteRenderers. Measured & used in production this project.
- **`GetComponent<T>() ?? fallback` is a trap** — Unity's fake-null vs C# `??`; use
  `if (x == null)`.
- Runtime-spawned (`Start()`/`Awake()`) objects are **invisible in edit mode**; bake
  if you need edit-mode/persistent visibility.

## Core mental models

- **World units & PPU.** `Pixels Per Unit` (default **100**) maps texture pixels to
  world units: a 32px tile at PPU 32 = 1 unit; a 64px sprite at PPU 32 = 2 units.
  **Every sprite's PPU must be consistent** (and match the Pixel Perfect Camera's) or
  scale/alignment breaks. Orthographic `orthographicSize` = **half** the view height
  in world units (view height = size×2).
- **2D draw order chain (in order):** Sorting Layer → Order in Layer (`sortingOrder`)
  → material Render Queue → **distance along the transparency sort axis** → (last)
  unguaranteed batch order. Ties are only broken by the sort axis, so for reliable
  overlap keep contenders on the **same Sorting Layer + Order** and let the axis decide.
- **Tilemap = one batched mesh** (chunked, culled, cheap) vs **SpriteRenderer = one
  GameObject each** (per-object cull + memory). Choose per workload (decision guide below).

## Sprites & PPU  → `reference/sprites.md`

- Sprite Renderer **Draw Mode**: Simple (default) / Sliced / Tiled. Sliced & Tiled
  need a sprite with **borders** (9-slice) or they just stretch; 9-slice colliders only
  work with Box/Polygon Collider 2D.
- **Sprite Sort Point**: Center (default) or **Pivot**. Use **Pivot** (at the sprite's
  feet) for Y-sorting so depth keys off the base, not the center.
- **Sprite Atlas (V2)** is the #1 many-sprite optimization: same-atlas sprites batch
  to one draw call. Default padding 4px; Master vs Variant atlases; **can't revert V2→V1**.
- Import: Sprite Mode Single/Multiple/Polygon; sub-32×32 sprites force **Full Rect**
  mesh; Mesh Type Full Rect vs Tight; `alphaIsTransparency` on by default for Sprite.

## Sorting & Y-sort  → `reference/sorting-and-cameras.md`

**Per-sprite Y-sort recipe** (overlapping characters/props, higher-Y behind lower-Y):
1. Project Settings → Graphics (or the URP asset) → **Transparency Sort Mode = Custom Axis**.
2. **Transparency Sort Axis = (0, 1, 0)**.
3. Each Sprite Renderer **Sort Point = Pivot** (pivot at the feet).
4. Keep the overlappers on the **same Sorting Layer + Order in Layer** so the axis sorts them.
- Wrap a **multi-part character in a Sorting Group** so its sub-sprites sort as one unit
  against the world.
- **Isometric Z-as-Y**: axis **(0, 1, −0.26)** where −0.26 = (CellSizeY×−0.5) − 0.01;
  Z=0 sorts wrong. Needs TilemapRenderer **Mode = Individual**.
- **Pixel Perfect Camera** (URP: built-in — do NOT also install the standalone package):
  Asset PPU, Reference Resolution, Crop Frame, Grid Snapping (Pixel Snapping / Upscale
  Render Texture); all sprite PPU must match its Asset PPU.

## Tilemaps  → `reference/tilemaps.md`  *(the highest-leverage 2D tool)*

- **Grid → Tilemap → Tilemap Renderer → Tile assets.** Multiple Tilemaps under one Grid
  act as layers, ordered by each renderer's Sorting Layer / Order in Layer.
- **TilemapRenderer.Mode — the key decision:**
  - **Chunk** (default): batches tiles per chunk, one sort item, **chunk-culled** (only
    visible chunks render). Fewest draw calls. **BUT** nothing renders *between* tiles
    (can't interleave/Y-sort with characters), and it **can't sort tiles from multiple
    textures** → **pack tiles into one atlas** or they sort wrong. Also **not SRP-Batcher
    compatible**.
  - **Individual**: sorts each tile by position → characters can render *between* tiles
    (correct overlap/Y-sort); costs more per tile. Use only when you need interleave.
  - **SRP Batch**: chunk-style batching that *is* SRP-Batcher compatible.
- **Overlap:** one tilemap can't stack sprites in a cell. For overlapping props, either
  Individual mode (interleave, still 1/cell) or **N stacked tilemaps as depth strata**
  (each a fixed sort order → periodic sort-wrap for a continuous gradient; scatter it
  with a phase jitter — see observed-vs-documented).
- **Culling:** `Detect Chunk Culling Bounds` Auto (default) / Manual. Set **Manual +
  Chunk Culling Bounds** when tile sprites extend beyond their cells, or edge chunks
  cull too early (trees/tall props pop out). `Max Chunk Size` trades batch size vs cull
  granularity.
- **Colliders:** Tilemap Collider 2D generates per-tile shapes (Tile ColliderType
  None/Sprite/Grid). **Add a Composite Collider 2D** (+ a **Static Rigidbody 2D**, which
  it requires) to merge them into one shape — big physics win.
- **Scripting:** `SetTile`/`SetTilesBlock` (bulk), `GetTile`/`HasTile`, `CompressBounds`,
  `cellBounds`, `layoutGrid`, `GetCellCenterWorld`/`CellToWorld`/`WorldToCell`.
- Tile Anchor default (0.5, 0.5, 0) = cell center. `.tmx` layers can be base64+gzip for
  compact sparse data (see observations).

## Rendering performance & batching  → `reference/performance-and-batching.md`

- **Methods, in Unity's per-object priority:** Static batching → **SRP Batcher** (+ GPU
  Resident Drawer/BRG in U6) → GPU instancing → dynamic batching.
- **SRP Batcher does NOT reduce draw-call *count*** — it cuts per-draw CPU by batching
  bind+draw by shader variant (needs `UnityPerMaterial`/`UnityPerDraw` CBUFFERs, **no
  MaterialPropertyBlock**). It's the default win for many same-shader renderers.
- **Hard limits:** dynamic batching ≤ **900 vertex attributes / 300 vertices**; static
  batching buffer ≤ **64,000 verts**; **`Graphics.DrawMeshInstanced` ≤ 1023/call and is
  obsolete in U6 → use `Graphics.RenderMeshInstanced`** (or `RenderMeshInstancedIndirect`
  / BatchRendererGroup for huge counts).
- **2D specifics:** **Sprite Atlas** is the primary many-sprite optimization. Frustum
  culling is **automatic per-renderer** (`SpriteRenderer.isVisible`, `OnBecameVisible/
  Invisible`) — off-screen sprites aren't drawn. Occlusion culling is primarily 3D.
- **The count trap (measured this project):** ~9,200 SpriteRenderer GameObjects cost
  real per-frame overhead **even at 0 visible** (culling pass + transforms + memory);
  community pain ~**10k**. The same content as **stamped tilemap layers** → **24
  transforms**, chunk-culled. **Rule: many *static* sprites → tilemap; thousands of
  *dynamic* → object pooling (`UnityEngine.Pool.ObjectPool<T>`) or instancing.**

## 2D physics & lighting  → `reference/physics-lighting-animation.md`

- **Rigidbody 2D** Body Type: Dynamic / Kinematic / Static. **Move dynamic bodies via
  the Rigidbody (MovePosition/velocity), never Transform.** Drag fields are now **Linear/
  Angular Damping**.
- **Colliders:** Box/Circle/Capsule/Polygon/Edge/Composite. "Used By Composite" is now a
  **Composite Operation** enum (None/Merge/Intersect/Difference/Flip). Physics Material 2D
  combine defaults are asymmetric: **Friction = Mean, Bounciness = Maximum**.
- **URP 2D lighting:** Light 2D types Freeform / Sprite / **Spot** (was "Point") / Global;
  **Parametric is deprecated**. Global lights can't use normal maps or cast shadows. Normal
  maps need reserved secondary textures `_NormalMap`/`_MaskTex` and add a depth pre-pass
  (perf cost). Shadow Caster 2D for shadows.

## Decision guides

**"I need to render N of X":**
| Case | Do this |
|---|---|
| Large static tiled world/terrain | **Tilemap(s)**, Chunk mode, one atlas. |
| Overlapping static props (trees/rocks) needing depth | **N stacked tilemaps** (depth strata) or Individual mode; or sprites if you need perfect per-instance Y-sort. |
| Characters must render *between* tiles | TilemapRenderer **Individual** mode + Custom Axis Y-sort. |
| Thousands of *dynamic* sprites (bullets, units) | **Object pooling**; for extreme counts `RenderMeshInstanced(Indirect)`/BRG with a custom sort. |
| A few hundred sprites | Plain SpriteRenderers + a Sprite Atlas are fine. |

**Pixel-art setup:** consistent PPU across all sprites (match Pixel Perfect Camera Asset
PPU) · Point / no-mips / Uncompressed (already the 2D default) · watch `maxTextureSize`
on big sheets · Pixel Perfect Camera (built into URP).

## Working through MCP / editor automation

- MCP `execute_code` runs a CodeDom C# method body: **no `using` directives** (fully-
  qualify types), and Unity fake-null means `??` on `GetComponent` fails there too.
- **Reimporting a prefab asset does not refresh the live play-mode instance** — exit
  Play / rebuild the scene to see `.tmx`/prefab changes.
- `UnityEditor.UnityStats` (drawCalls/setPass/tris) **doesn't refresh within one
  synchronous call** after moving the camera; the editor must render a frame.
- To confirm a real import default, **import a throwaway asset and read its importer** —
  don't trust `new TextureImporterSettings()` struct defaults.
