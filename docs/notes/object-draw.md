# Spider-Man object draw (SLUS_008.75)

`Render_walk` draws each object list through `ObjectList_draw` (0x80076480) in a fixed order: 0x800B566C,
0x800B5394, `Player_list` (0x800B5268), 0x800B526C, 0x800B5398, 0x800B5234, 0x800B4E2C, 0x800B53A0, 0x800B4E6C.
A record links the next at +0x1C.

## Object record

| offset | what |
|---|---|
| +0x0 | flag halfword; bit 4 = pose precomputed by the update, bit 0x8000 = culled this frame (set by 0x8007B1B4) |
| +0x4, +0x8, +0xC | position, 20.12 world units |
| +0x10, +0x12, +0x14 | three 12-bit angles |
| +0x16 | model index |
| +0x1B | model bank |
| +0x38 | pointer to the orientation MATRIX |
| +0x124, +0x126 | animation frame, animation |
| +0x180 | pose pointer, read when flag bit 4 is set |

## Articulated models

`Model_drawArticulated` (0x80077198) takes the bank entry at `Model_bankTable + bank * 0x40`: its first word is the
part table and the halfword at +0x24 the part count it draws. The pose is one 0x18-byte entry per part, a
rotation MATRIX's nine halfwords then a translation's three at +0x12. Without flag bit 4 the pose comes from
`Pose_fetch(record)` (0x80010614), which builds it for the integer frame +0x124 of animation +0x126 into
`Pose_scratch` (0x800A58E4) or the record's own buffer. With bit 4 the update already left it at the pointer
+0x180: both articulated records seen on the player list (0x80198FD0, the player, and 0x80199230) draw this way, so
`Pose_fetch` never runs for them inside the draw.

The view is `Camera_current` (gp+0x1128): its rotation MATRIX at +0x74 and position at +0x4.

## Front-end waits

The front-end overlay holds a still screen with `Field_wait(240)` (call returning to 0x801654F4): four seconds of
fields on hardware, each one displayed.
