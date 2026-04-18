# How the Patch Works

A high-level overview. Assumes you know what a framebuffer and a coordinate system are. No assembly.

## The problem

Saikyo no Mahjong 3D was built for 1024×768 and hardcodes it in a bunch of places. A naive "set the resolution higher" approach gets you a sharp 3D scene in the corner of a mostly-broken window, the 2D UI is still in the top-left at 1×, buttons don't click, tiles don't pick.

Five subsystems all touch resolution and have to agree on it:

1. **D3D backbuffer** : the framebuffer the game renders into.
2. **2D sprite rendering** : UI panels, buttons, backgrounds.
3. **Mouse input** : how cursor position gets normalized from Windows into the game's space.
4. **2D UI hit-testing** : which button the mouse is over.
5. **3D tile picking** : which hand tile the mouse is over (done with an actual ray cast, not a 2D rect).

Change one, the others stop agreeing, game breaks. This is a 2007 mahjong game, it was never designed to be resolution-agnostic.

## Three coordinate spaces

The game shuffles between three coordinate systems at the same time.

**Raw client space.** What Windows tells you via `GetCursorPos` + `ClientToScreen`. At 2× the cursor lives in 0..2048 × 0..1536.

**Logical 1024 space.** The game's internal world. Mouse coords get `MulDiv`'d from raw-client down to 0..1024 × 0..768 the moment they enter the game. UI button rectangles live here. Tile-picking NDC math assumes it. As far as the game logic is concerned, the screen is always 1024×768.

**Screen 2× space.** 0..2048 × 0..1536 , what the GPU actually rasterizes into after the patch.

The trick the patch plays: **leave the game thinking it's still 1024×768.** Don't touch mouse polling, don't touch button rects, don't touch tile positions. Translate between spaces at two spots only:

- At the sprite draw call, double vertex positions so things render at 2×.
- At the ray-construction call, force it to use 1024/768 as its divisor so the math matches the logical-1024 mouse it's receiving.

Everywhere else, 1024-logical stays intact. Probably a more elegant way to do this exists. I'm new at this.

## What each patch does

### Backbuffer

`IDirect3DDevice8::CreateDevice` gets `BackBufferWidth = 1024`, `BackBufferHeight = 768` in its presentation params. Change the immediates to 2048 and 1536. The 3D scene scales automatically because 3D respects the viewport. The 2D UI doesn't , it's stuck at 1× in the corner until the next patches hit it.

### Sprite upscaling

All 2D sprites go through one function (`FUN_00451A51`). It builds a 4-vertex quad in pre-transformed screen space (D3D's XYZRHW format), subtracts 0.5 for GPU pixel-center alignment, submits via `DrawPrimitiveUP`.

Two JMPs replace the X and Y store blocks with a code cave. The cave does the original math, load coord, subtract 0.5, store , plus one inserted step: multiply by 2 before the subtract. Every sprite's position and size doubles. The half-pixel offset stays a half-pixel, because it's a GPU sampling thing, not a resolution thing.

Works for transformed sprites (doubling amplifies the transform's output, giving uniform 2× scale) and untransformed ones (the pre-transform coords are already in pixel space, so doubling turns a 1024×768 background into a 2048×1536 one).

### Mouse polling and UI hit-testing (neither patched)

Mouse enters via `GetCursorPos`, gets `MulDiv`'d by `1024/clientWidth` to land in 1024-logical. UI rects are stored in 1024-logical. Both sides of the hit test are in the same space, so clicks register correctly without touching anything.

### 3D tile picking

Tile picking builds a ray and tests it against each tile's bounding box. Ray construction needs NDC: `ndc = 2 × mouse / screen - 1`.

Here's where it gets subtle. The ray-builder reads its "screen width" from a cached copy of the backbuffer dimensions on the renderer object, which is now 2048 after the first patches. The mouse coords it gets are already in logical-1024 space. So NDC comes out as `2 × (0..1024) / 2048 - 1 = [-1, 0]` instead of `[-1, +1]`. Every ray thinks the mouse is in the left half of the screen, misses every tile.

Fix: JMPs to the cave, cave forces 1024/768 as the divisors. Ray math now matches the 1024-logical mouse. Tiles work.