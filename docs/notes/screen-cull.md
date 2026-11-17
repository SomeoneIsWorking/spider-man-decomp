# Spider-Man screen window and culls (SLUS_008.75)

`View_publish` (0x80075D0C, called once per frame from `Render_walk` at 0x8002BE0C) takes the camera and the
screen window `w` (u16 fields) and stores the window pointer at gp+0x1124 (`Screen_window`, 0x800B5918).

| field | what |
|---|---|
| w[0] | right |
| w[1] | bottom |
| w[2] | left |
| w[3] | top |
| w[4], w[5] | near z, far z |
| w[6] | lens divisor |
| w[7] | H, written: `(((w[0] - w[2]) / 2 << 12) / w[6] << 12) / [gp+0x1140]` |
| w[8], w[9] | OFX `(w[0] + w[2]) >> 1`, OFY, written |

It then builds the frustum planes at 0x800AA8DC.. from the half-width `(w[0] - w[2]) / 2`, the half-height and H,
rotates them by the camera, and calls `SetGeomScreen(w[7])` and `SetGeomOffset(w[8], w[9])`. Every field is read
zero-extended, so a window wider than the retail centre allows needs `w[2] = 0`: OFX moves with the span.

## Readers of the window's x bounds

Each is hand-written assembly that transforms a vertex list (RTPS per vertex), stores SXY and view z, and ORs a
6-bit outcode: bit 0 `right < sx`, bit 1 `sx < left`, bit 2 `bottom < sy`, bit 3 `sy < top`, bit 4 `z < near`,
bit 5 `far < z`; a vertex behind the eye (z < 0) inverts bits 0-3. The returned AND of all outcodes rejects a
polygon list wholly outside one edge. `right` and `left` are loaded by `lw` and `andi 0xFFFF`.

| function | caller | note |
|---|---|---|
| `Clip_outcodes` 0x8007C2AC | 0x80077D64 (from `ObjectList_draw`) | flag 0x10 on a vertex takes a billboard path (NCLIP, GPF) |
| 0x8007B628 | 0x80077A48 | plain list; IR to `[0x800B58F4]`, SXY and z to `[0x800B58F0]` |
| 0x8007B798 | 0x80077C08 | 12.4 packed vertices; flag bit 1 reuses a cached vertex, bit 0 caches one at `[0x800B5940]` |
| 0x8007B9CC | 0x80077C08 | as 0x8007B798 with 16-bit vertices |
| 0x8007BE90 | 0x80077B9C, 0x80077E5C | |

Two more readers take window word 0 whole (`bottom << 16 | right`) and never read w[2] or w[3]: the subdivision
face writers 0x8007D978 / 0x8007DDC8 load it at 0x8007D1E0 into s4 beside 0xBFFFBFFF in s5, and 0x8007F278 (from
0x80077B00) loads it at 0x8007F300 into t4 beside the same constant. Not traced: how their packed compare treats
a negative x. 0x8007BBD4 and 0x8007BD04 read only the far z (`lh w[5]`).

A wider screen that keeps OFX at 256 needs left below 0: every reader above rejects a vertex left of x 0.
