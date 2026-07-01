# Cluster 1 — Sprites & Sprite Renderer (Unity 6000.0)

Fact ledger extracted from docs.unity3d.com/6000.0 (Manual + ScriptReference).
Tag legend: [concept] definition · [default] exact default value · [api] scripting API · [gotcha] pitfall · [perf] performance · [workflow] editor procedure.

## Sprites (overview)
- **[concept]** A Sprite is a 2D graphic object; for 3D-familiar users, sprites are essentially standard textures with special combining/managing techniques for efficiency and convenience — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-landing.html
- **[concept]** Sprites are based on imported texture files; they differ from raw 3D textures by organizational technique rather than fundamentals — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-landing.html
- **[gotcha]** To use sprites you must have the 2D Sprite package installed in the project — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-landing.html
- **[workflow]** Sprite subsystems covered by the manual: placeholder sprites, importing/spritesheets, cropping & collision geometry, sorting/layering, 9-slicing, masking, sprite atlas packing, Sprite Renderer customization — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-landing.html
- **[perf]** Combining multiple textures into sprite atlases optimizes project performance (best practice) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-landing.html

## Sprite Texture Import Settings (Sprite Mode, PPU, pivot)
- **[concept]** Sprite Mode has three options: Single, Multiple, Polygon — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- **[concept]** Single = one sprite from the whole texture; Multiple = multiple sprites cut from a spritesheet (use Sprite Editor to slice); Polygon = custom polygon-shaped sprite mesh — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- **[concept]** Pixels Per Unit (PPU) = number of pixels of width/height in the Sprite image that correspond to one distance unit in world space — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- **[default]** Pixels Per Unit default = 100 (100 pixels = 1 world-space unit; a 100px-wide sprite renders 1 unit wide) — src: https://docs.unity3d.com/ScriptReference/Sprite-pixelsPerUnit.html
- **[gotcha]** Lowering PPU below 100 increases the sprite's world size; raising it shrinks the sprite — src: https://docs.unity3d.com/ScriptReference/Sprite-pixelsPerUnit.html
- **[concept]** Mesh Type (visible for Single/Multiple): Full Rect or Tight — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- **[gotcha]** Sprites under 32×32 pixels automatically use Full Rect regardless of the Mesh Type setting — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- **[concept]** Extrude Edges controls the area/border around the sprite in the generated mesh — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- **[concept]** Pivot (visible when Sprite Mode = Single): preset locations plus a Custom option for X/Y values — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- **[concept]** Generate Physics Shape: boolean toggle available for Single/Multiple modes — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- **[concept]** Alpha Source options: None / Input Texture Alpha / From Gray Scale — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- **[concept]** Alpha Is Transparency: boolean; dilates color channels to reduce edge artifacts on transparent borders — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html

## 2D Texture Import Settings for Pixel Art
- **[concept]** Filter Mode options: Point (no filter) / Bilinear / Trilinear. Point = blocky/sharp up close, Bilinear = blurry up close, Trilinear = blurs between mipmap levels — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- **[workflow]** For pixel art, set Filter Mode = Point (no filter) to keep pixels crisp — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- **[concept]** Compression options (Format = Automatic): None / Low Quality / Normal Quality / High Quality — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[workflow]** For pixel art, set Compression = None to avoid block-compression artifacts — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- **[workflow]** For pixel art, disable Generate Mip Maps (mip maps are for minified/distant textures and blur pixel art) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- **[default]** Max Size default = 2048 — src: https://docs.unity3d.com/6000.0/Documentation/Manual/class-TextureImporter-type-and-shape.html
- **[default]** Read/Write Enabled default = disabled (off); enable only for scripted Texture2D access (costs extra CPU-side memory copy) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- **[concept]** Wrap Mode options: Repeat / Clamp / Mirror / Mirror Once / Per-axis — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- **[concept]** Non Power of 2 options: None / To nearest / To larger / To smaller — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- **[concept]** Mipmap Filtering options (when mipmaps enabled): Box / Kaiser — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- **[concept]** sRGB (Color Texture): enable for color/albedo textures (stored in gamma space); disable for data textures — src: https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- **[concept]** Aniso Level is only relevant when Generate Mip Maps is enabled and Filter Mode is Bilinear or Trilinear — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[gotcha]** Most import-setting defaults (Filter Mode, Aniso Level, Compression, Wrap Mode, Generate Mip Maps) are not stated verbatim in the 6000.0 reference pages; verify in-editor. Editor factory behavior: Filter Mode = Bilinear, Wrap Mode = Repeat, Aniso Level = 1, Generate Mip Maps default off for Sprite type — src: https://docs.unity3d.com/6000.0/Documentation/Manual/class-TextureImporter.html

## Sprite Editor (window)
- **[concept]** The Sprite Editor lets you create sprites from a texture, edit the meshes Unity uses to render sprites, configure collision detection, and rig sprites for animation — src: http://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference-landing.html
- **[concept]** Four tabs: (1) Sprite Editor tab (configure sprites / convert a texture into multiple sprites), (2) Custom Outline (configure the render-mesh shape), (3) Custom Physics Shape (configure the collision mesh shape), (4) Secondary Textures (add normal maps / mask maps) — src: http://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference-landing.html
- **[workflow]** SpriteRects preview shows as red rectangular outlines; manually refine any auto-generated SpriteRect by dragging its blue handles/edges — src: http://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/use-editor.html
- **[workflow]** Apply changes via the Apply button in the toolbar — src: http://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/use-editor.html

## Sprite Editor — Slicing
- **[concept]** Slice Type options: Automatic, Grid By Cell Size, Grid By Cell Count, Isometric Grid — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- **[concept]** Automatic = slices based on the transparency of the texture (finds islands of opaque pixels) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- **[concept]** Grid By Cell Size = slices into sprites of equal size using Pixel Size (width/height) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- **[concept]** Grid By Cell Count = slices into a specific number of rows and columns (Column & Row fields) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- **[concept]** Isometric Grid = slices into isometric diamond-shaped sprites instead of rectangles; Is Alternate staggers the diamonds — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- **[concept]** Pixel Size = width & height of each sprite in pixels (Grid By Cell Size / Isometric Grid) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- **[concept]** Offset (X/Y) shifts the grid from the upper-left of the image; Padding (X/Y) sets space added between sprites in pixels — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- **[concept]** Pivot Unit Mode: Normalized (0–1 range) or Pixels; Custom Pivot for user-defined coordinates — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- **[concept]** Method options: Delete Existing / Smart / Safe. Delete Existing removes all current SpriteRects and re-slices; Smart tries to keep/adjust existing rects; Safe only adds new rects without altering existing ones — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- **[concept]** Keep Empty Rects preserves sprites/rects that contain no opaque pixels — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html

## Sprite Renderer component
- **[concept]** Sprite Renderer renders a Sprite in a 2D or 3D scene — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Sprite field = which Sprite Unity renders (assign via Project window or picker) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[default]** Color tints the sprite; default = white (no tint) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Flip X / Flip Y mirror the texture along the x/y axis; does NOT change GameObject position/transform — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[default]** Material default = Sprite-Lit-Default — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[default]** Draw Mode default = Simple. Options: Simple / Sliced / Tiled — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Draw Mode Simple = scales the entire sprite uniformly — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Draw Mode Sliced = stretches center and edges but keeps corners at original size (requires 9-slicing borders) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Draw Mode Tiled = repeats the sprite texture to fill the new dimensions (requires 9-slicing borders) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Size field appears when Draw Mode is Sliced or Tiled; sets the sprite's rendered dimensions — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Tile Mode (Tiled only): Continuous = does NOT stretch the texture (edges may be cropped); Adaptive = stretches the center until it reaches the Stretch Value, then repeats — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Stretch Value (Adaptive) controls the repeat threshold; a value of 1 = repeats at 2× original size; lower = less frequent repetition — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Mask Interaction: None (ignores masks) / Visible Inside Mask (renders only parts overlapping the mask) / Visible Outside Mask (renders only parts NOT overlapping the mask) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Sprite Sort Point: Center (sort by sprite center) or Pivot (sort by pivot position set in Sprite Editor) — determines the point used for render-order distance calculation — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Sorting Layer = render order per the layer list in Tags and Layers; new layers via Add Layer — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Order in Layer = sublayer index within a Sorting Layer; lower values render before (behind) higher values — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- **[concept]** Rendering Layer Mask defines which rendering layers the GameObject belongs to (used by URP rendering-layer systems) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html

## Sprite Renderer scripting API
- **[api]** SpriteRenderer.sprite (Sprite to render); .color (rendering tint) — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- **[api]** SpriteRenderer.flipX / .flipY (bool) flip along X/Y axis — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- **[api]** SpriteRenderer.drawMode : SpriteDrawMode enum { Simple, Sliced, Tiled } — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- **[api]** SpriteRenderer.size (Vector2) used when drawMode is Sliced or Tiled — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- **[api]** SpriteRenderer.tileMode : SpriteTileMode enum { Continuous, Adaptive }; only affects behavior when drawMode = SpriteDrawMode.Tiled — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer-tileMode.html
- **[api]** SpriteRenderer.adaptiveModeThreshold = threshold determining when the sprite tiles when tileMode = Adaptive — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer-adaptiveModeThreshold.html
- **[api]** SpriteRenderer.maskInteraction : SpriteMaskInteraction enum { None, VisibleInsideMask, VisibleOutsideMask } — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- **[api]** SpriteRenderer.spriteSortPoint : SpriteSortPoint enum { Center, Pivot } — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- **[api]** SpriteRenderer inherits Renderer.sortingLayerName, Renderer.sortingOrder, Renderer.material — src: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html

## 9-slicing / Sliced & Tiled draw modes
- **[concept]** 9-slicing resizes a sprite without creating multiple sprites for each size; divides the sprite into 9 sections that scale/repeat differently — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/9-slice/9-slicing.html
- **[concept]** The 9 sections: 4 corners (unchanged), 4 borders/edges (stretch or repeat in one direction only), 1 center (stretches/repeats both horizontally and vertically) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/9-slice/9-slicing.html
- **[concept]** Sliced mode = borders stretch, corners fixed; Tiled mode = borders and center repeat (tile), corners fixed — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/9-slice/9-slicing.html
- **[gotcha]** Only Box Collider 2D and Polygon Collider 2D work with 9-sliced sprites for collision — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/9-slice/9-slicing.html
- **[gotcha]** 9-slicing requires borders set on the sprite (in the Sprite Editor) plus Draw Mode = Sliced or Tiled on the Sprite Renderer; without borders, Sliced/Tiled behave like Simple stretch — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/9-slice/9-slicing.html

## Sprite Mask component
- **[concept]** A Sprite Mask determines which areas of an underlying sprite are revealed or hidden; opaque pixels define the mask shape, transparent areas fall outside the mask — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- **[concept]** Mask Source: Sprite (use a designated sprite's texture) or Supported Renderer (use a Sprite Renderer, Sprite Shape Renderer, or Tilemap Renderer on the GameObject) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- **[concept]** Sprite field (Mask Source = Sprite) selects the sprite that provides the mask texture — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- **[concept]** Supported Renderer field (Mask Source = Supported Renderer) selects the renderer component providing the mask texture — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- **[concept]** Sprite Sort Point (Mask Source = Sprite): Center or Pivot — reference point for sort-distance calculation — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- **[concept]** Alpha Cutoff = minimum alpha threshold for mask pixels; lower values include more semi-transparent pixels in the mask shape — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- **[concept]** Custom Range limits which sorting layers the mask affects, via Front and Back bounds — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- **[concept]** Front = highest affected layer (Sorting Layer + Order in Layer); layers in front of it are NOT masked — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- **[concept]** Back = lowest unaffected layer (Sorting Layer + Order in Layer) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- **[gotcha]** A Sprite Mask only affects Sprite Renderers whose Mask Interaction is set to Visible Inside Mask or Visible Outside Mask; renderers with Mask Interaction = None ignore all masks — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html

## Sprite Atlas (packing & benefits)
- **[concept]** A sprite atlas packs several sprite textures tightly together within a single texture — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/atlas-landing.html
- **[perf]** Purpose: reduce draw calls sent to the GPU — Unity needs only one draw call for all sprites sharing an atlas (batching) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/atlas-landing.html
- **[workflow]** Create: main menu Assets > Create > 2D > Sprite Atlas; produces a .spriteatlasv2 file in the Project window — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/create-sprite-atlas.html
- **[workflow]** Populate by dragging sprites/textures/folders onto the Objects for Packing label, or use the (+) button — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/create-sprite-atlas.html
- **[workflow]** Pack Preview displays the packed texture in the Inspector preview; a dropdown shows combined secondary textures (e.g. normal maps) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/create-sprite-atlas.html
- **[default]** Unity includes sprite atlases in builds by default — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/create-sprite-atlas.html
- **[concept]** After configuration, sprites render using their atlas texture in Scene view, Game view, and Play mode — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/create-sprite-atlas.html

## Sprite Atlas V2
- **[concept]** Sprite Atlas V2 is the newer atlas format that replaced V1 (used up to Unity 2022.2) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/v2/sprite-atlas-v2.html
- **[default]** Sprite Atlas V2 is enabled by default in the Editor from Unity 2022.2 onward — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/v2/enable-sprite-atlas-v2.html
- **[gotcha]** V2 assets cannot be converted back to V1; back up V1 assets before upgrading — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/v2/sprite-atlas-v2.html
- **[concept]** The V2 Inspector interface is the same as a V1 sprite atlas — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/v2/sprite-atlas-v2.html
- **[workflow]** Enable/upgrade via Edit > Project Settings > Editor, set the Sprite Atlas Mode setting — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/v2/enable-sprite-atlas-v2.html
- **[concept]** Sprite Atlas Mode values: Disabled / Sprite Atlas V2 - Enabled / Sprite Atlas V2 - Enabled for Builds — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/v2/enable-sprite-atlas-v2.html
- **[concept]** "Sprite Atlas V2 - Enabled for Builds" uses original unpacked textures during editing but keeps atlas textures for Play mode and builds — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/create-sprite-atlas.html

## Sprite Atlas Inspector (fields & defaults)
- **[default]** Type: Master (default) or Variant. Master can be a parent for Variant atlases; Variant is a lower-resolution version of another atlas — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[concept]** Master Atlas field (Variant only) designates the parent atlas (must be Type = Master) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[default]** Include in Build = enabled by default; controls whether Unity loads the atlas at startup vs at runtime — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[concept]** Scale (Variant only) range 0 to 1; multiplier for parent-atlas resolution (0.5 = half resolution) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[gotcha]** Don't use variant Scale/Max-Size smaller than 0.25 — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[concept]** Allow Rotation (Master, Packing): allows sprites to be rotated during packing; disable for Canvas UI to prevent rotation — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[concept]** Tight Packing (Master): packs using the custom mesh outline instead of the bounding rectangle — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[concept]** Alpha Dilation: expands edge colors into transparent pixels (reduces edge bleed artifacts) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[default]** Padding default = 4 pixels; prevents pixel bleed between packed sprites — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[default]** Texture Read/Write = disabled by default — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[concept]** Texture section: Generate Mip Maps, sRGB (gamma space), Filter Mode (Point/Bilinear/Trilinear), Aniso Level (only with mipmaps + Bilinear/Trilinear) — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[default]** Format default = Automatic — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[concept]** Compression (Format = Automatic): None / Low Quality / Normal Quality / High Quality; Use Crunch Compression toggle requires Format = Automatic — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- **[concept]** Packables list holds sprites/textures/folders; add via drag-drop or (+), remove via (-); Pack Preview generates a preview of the combined atlas — src: https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html

## Sources
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-landing.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-sprite.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/texture-type-default.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/class-TextureImporter.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/class-TextureImporter-type-and-shape.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/renderer/sprite-renderer-reference.html
- http://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference-landing.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/sprite-editor-window-reference.html
- http://docs.unity3d.com/6000.0/Documentation/Manual/sprite/sprite-editor/use-editor.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/9-slice/9-slicing.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/mask/sprite-mask-reference.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/atlas-landing.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/create-sprite-atlas.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/sprite-atlas-reference.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/v2/sprite-atlas-v2.html
- https://docs.unity3d.com/6000.0/Documentation/Manual/sprite/atlas/v2/enable-sprite-atlas-v2.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer-tileMode.html
- https://docs.unity3d.com/6000.0/Documentation/ScriptReference/SpriteRenderer-adaptiveModeThreshold.html
- https://docs.unity3d.com/ScriptReference/Sprite-pixelsPerUnit.html
