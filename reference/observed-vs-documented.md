# Editor-Confirmed Observations (this project, Unity 6000.4.3f1, URP 2D)

Firsthand observations from building the Ultima4_2d overworld. Each tagged:
- **[confirms-docs]** — matches documentation; recorded for emphasis.
- **[extends-docs]** — true and practical but not clearly stated in the docs.
- **[diverges — REVIEW]** — behaved against a common documented assumption; verify.
- **[third-party]** — SuperTiled2Unity (STU) / package behavior, not core Unity docs.

## Tilemaps: overlap & Y-sort
- **[extends-docs]** A single Tilemap **cannot overlap sprites** — one tile per cell. Stamping a multi-tile sprite (a 3x4 tree) and relying on last-write-wins truncates the sprites (they render as small tops). CONFIRMED: dict-overwrite in one layer produced tiny trees; overlap only appeared once trees were split across multiple tilemap layers.
- **[extends-docs]** To overlap tiles you need **N stacked Tilemaps** (each a fixed depth stratum) OR TilemapRenderer Mode=Individual for per-tile sort (still 1 tile/cell, so still no true overlap). N discrete layers => periodic sort "wrap" every N rows for a continuous depth gradient. WORKAROUND CONFIRMED: 2D layer coloring `(tc&1)+2*(tr%4)` (no two overlapping stamps share a cell) + per-column phase jitter to scatter the wrap so revealed trunks look natural instead of banding.
- **[extends-docs]** True per-sprite continuous Y-sort of overlapping instances needs SpriteRenderers (or instancing), not tilemaps. Trade: tilemaps give chunk-culling + bakeable; sprites give perfect depth but per-object cost.

## Rendering / culling / performance (measured)
- **[confirms-docs]** Off-screen SpriteRenderers are auto frustum-culled (2D, same as 3D): with 9,216 conifer GameObjects and 0 on-screen, drawCalls stayed ~9.
- **[extends-docs]** BUT ~9,200 SpriteRenderer GameObjects impose per-frame overhead even at 0 visible — measured ~50fps in-editor with nothing drawn; 9,233 scene transforms. Community pain threshold ~10k SpriteRenderers. MEASURED this session.
- **[confirms-docs]** Tilemap chunk-culling: converting the same forest to 8 stamped tilemap layers dropped scene transforms 9,233 -> 24; in-forest drawCalls ~9, setPassCalls ~69; frame overhead gone.
- **[extends-docs]** tmx layer data as base64+gzip is tiny for sparse layers: 8 sparse tree layers added only ~0.4MB and imported in ~5s (vs CSV bloat). STU reads gzip/zlib base64 layers fine.

## SuperTiled2Unity (STU) — third-party importer behavior
- **[third-party]** STU assigns `TilemapRenderer.sortingOrder` by tmx layer order: first layer = 0, next = 1, ... (observed Base=0, Blend=1, Detail=2, Trees0=3 ... Trees7=10).
- **[third-party]** PixelsPerUnit is **per-asset** on the STU importer (`m_PixelsPerUnit`), and a baked per-asset value in the `.meta` OVERRIDES the global `ST2USettings.m_DefaultPixelsPerUnit`. Setting only the global default did NOT change an already-imported .tmx/.tsx; had to set the importer's serialized `m_PixelsPerUnit` per asset and reimport. `ST2USettings.SaveSettings()` is internal (reflection needed).
- **[third-party]** A Tiled `.tsx` missing `tilecount`/`columns` slices to 0 tiles in STU (silent). Adding them fixed it.
- **[third-party]** STU imports `.tmx` as a prefab with Grid + child Tilemaps; a 1M-tile layer imported in ~3s, 2 layers ~5.6s — perfectly usable.

## Cameras / import (pixel art)
- **[diverges — REVIEW]** A 4096x4096 atlas imports with default `maxTextureSize=2048`, silently downscaling to 2048 and breaking 32px tile alignment. Had to set `maxTextureSize=4096`. (Docs mention max size but the silent tile-misalignment consequence is the gotcha.)
- **[confirms-docs]** For crisp pixel-art tiles set Filter Mode = Point, Mip Maps off, Compression = None. Bilinear/mips caused edge bleed/blur.
- **[extends-docs]** Matching PPU is what fixes the "asset ratio": a 32px tile at PPU 32 = 1 world unit; a 64px character at PPU 32 = 2 units = 2 tiles. Mismatched PPU (tiles at STU default vs character at 32) forced an ugly transform-scale hack until PPUs were matched.

## Editor workflow gotchas (MCP/automation-relevant)
- **[diverges — REVIEW]** Reimporting a prefab ASSET does not refresh the already-instantiated PLAY-MODE instance. Had to exit play + rebuild the scene to see tmx changes. (Expected for asset vs instance, but easy to miss when automating.)
- **[extends-docs]** MonoBehaviour objects created in `Start()`/`Awake()` (e.g. a runtime spawner) are **invisible in edit mode** — nothing exists until Play. Bake at editor time (ExecuteAlways or a menu tool) or into the map if you need edit-mode/persistent visibility.
- **[extends-docs]** `UnityEditor.UnityStats` (drawCalls/setPassCalls/tris) does not refresh within a single synchronous editor call after moving the camera — the editor must render a frame; unreliable for one-shot measurement.
- **[diverges — REVIEW]** Unity "fake null": `GetComponent<T>() ?? fallback` does NOT work — Unity's overloaded `==`/`!=` treats a destroyed/missing object as null, but C# `??` uses real null, so `??` returns the fake-null object. A Grid found via `GetComponent<Grid>() ?? GetComponentInChildren<Grid>()` returned the fake-null and skipped the child. Use explicit `if (x == null)` checks. (Documented as fake-null behavior, but the `??` interaction bites people.)

## MCP for Unity (execute_code) environment
- **[third-party]** `execute_code` runs a CodeDom C# method body: NO `using` directives (use fully-qualified names), and Unity fake-null means `??` on GetComponent fails there too.

## Texture import defaults — EDITOR-CONFIRMED (diverges from common assumption)
- **[diverges — REVIEW]** In THIS Unity 6000.4 **2D URP** project a freshly-imported PNG defaults to: **textureType=Sprite, filterMode=Point, mipmapEnabled=False, wrapMode=Clamp, textureCompression=Uncompressed, spritePixelsPerUnit=100.** Verified by importing a throwaway 4x4 PNG and reading its TextureImporter. This is the 2D Behavior Mode default. It CONTRADICTS the common/general assumption (and a doc-worker's guess) that new textures default to Default-type / Bilinear / mips-on / Repeat / compressed. TAKEAWAY: in a 2D project the pixel-art-friendly settings are already the default — you rarely need to force Point/no-mip; the real gotcha is maxTextureSize (2048) silently downscaling large atlases.
- **[gotcha]** `new TextureImporterSettings(); ApplyTextureType(Sprite)` returns filterMode=Point too, but that's the struct default, NOT proof of the applied import default — only a real fresh import (above) confirms it. Method matters when "confirming in the editor."
