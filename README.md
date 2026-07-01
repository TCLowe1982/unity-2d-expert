# Unity 2D Expert — a Claude skill

A drop-in **[Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)**
that gives Claude (Claude Code / the Agent SDK) working expertise in **Unity 6 (URP)
2D game development** — sprites & pixels-per-unit, sorting and per-sprite Y-sort,
tilemaps (Chunk vs Individual vs SRP-Batch, chunk culling, colliders), rendering
performance & batching, 2D physics, 2D lighting, and pixel-art setup.

It's **documentation-grounded but editor-confirmed**: every rule traces to the official
Unity 6000.0 docs (source URLs kept in `reference/`), and wherever real Unity Editor
behavior diverged from a documented default or a common assumption, the observation was
**verified in-editor and flagged for review** rather than trusted blindly.

> I couldn't find a good Unity-2D skill for Claude anywhere, so this is an attempt at a
> solid, honest first one. PRs with more editor-confirmed facts welcome.

## What's inside

```
SKILL.md                              # the actionable decision layer Claude reads first
reference/
  sprites.md                          # Sprite Renderer, PPU, atlas, 9-slice, masks, import
  sorting-and-cameras.md              # draw-order chain, Y-sort recipe, sorting groups, Pixel Perfect
  tilemaps.md                         # Grid/Tilemap/Renderer, Chunk/Individual, culling, colliders, API
  performance-and-batching.md         # SRP Batcher / GPU instancing / static+dynamic, hard limits, pooling
  physics-lighting-animation.md       # Rigidbody/Collider 2D, Light 2D, Sprite Shape, 2D Animation
  observed-vs-documented.md           # editor-CONFIRMED observations, incl. divergences [REVIEW]
```

`SKILL.md` is a tight ~1-page decision layer; the `reference/` files hold **500+ sourced
facts** (every field, enum, default, gotcha) so Claude can drill in when a task needs it.

## Highlights it gets right (that trip people up)

- **`TilemapRenderer.Mode`** — Chunk (default: batched, chunk-culled, but *can't* interleave
  sprites or sort tiles across multiple textures → use one atlas) vs Individual (per-tile
  sort, characters render *between* tiles, slower) vs SRP-Batch (SRP-Batcher-compatible).
- **A single Tilemap can't overlap sprites** (1 tile/cell) — overlapping props need
  Individual mode or **N stacked tilemaps as depth strata**, or SpriteRenderers.
- **Per-sprite Y-sort recipe** — Transparency Sort Mode = Custom Axis, axis `(0,1,0)`,
  Sort Point = Pivot; isometric Z-as-Y uses `(0,1,-0.26)`.
- **The many-sprite count trap** — off-screen sprites are auto-culled, but ~10k
  SpriteRenderer GameObjects still cost real per-frame overhead; static content belongs
  in tilemaps, thousands of dynamic sprites want pooling/instancing.
- **SRP Batcher doesn't reduce draw-call *count*** — it cuts per-draw CPU cost.
- **Editor-confirmed divergence:** in a Unity 6 **2D URP** project, a fresh PNG imports
  as `Sprite`/Point/mips-off/Clamp/Uncompressed/PPU 100 — the pixel-art defaults are
  already set; the real gotcha is `maxTextureSize` silently downscaling large atlases.

## Install

Clone into your Claude skills folder:

```bash
# user-level (available in every project)
git clone https://github.com/TCLowe1982/unity-2d-expert \
  ~/.claude/skills/unity-2d-expert

# or project-level (committed with a specific repo)
git clone https://github.com/TCLowe1982/unity-2d-expert \
  <your-project>/.claude/skills/unity-2d-expert
```

Claude auto-discovers it. It triggers on Unity 2D work (tilemaps, sprite sorting/overlap,
PPU, draw-call/perf questions, 2D lighting, Pixel Perfect, …) or invoke it directly with
`/unity-2d-expert`.

## How it was built

Assembled with the **[ledger pattern](https://github.com/TCLowe1982/ledger-pattern)**
(extraction branch): the Unity 6000.0 2D documentation was split into topic clusters,
each processed independently into a durable on-disk fact-ledger with source URLs, then
assembled into `SKILL.md` + `reference/`. Key defaults and "docs don't state this" gaps
were then **confirmed against a live Unity 6000.4 editor** and recorded in
`reference/observed-vs-documented.md`.

## The discipline: doc-first, editor-confirmed

Rules are sourced from the docs. Where an editor observation contradicts a documented
default or common assumption, the observation wins **only after being reproduced in the
editor**, and it's tagged so a human can review it:

- `[confirms-docs]` · `[extends-docs]` · `[diverges — REVIEW]` · `[third-party]`

If you add facts, keep that discipline: cite the doc URL, or mark it `[diverges — REVIEW]`
with how you confirmed it.

## Attribution

The `reference/` material distills **Unity's official documentation** (Unity 6000.0
Manual & Scripting Reference — every bullet links its source page). Unity, the Unity
engine, and its documentation are © Unity Technologies. This repository is an independent,
factual study aid and is not affiliated with or endorsed by Unity Technologies.

## License

MIT — see [LICENSE](LICENSE). (Applies to the skill's structure and original writing; the
underlying Unity facts are, well, facts, and belong to their documented source.)
