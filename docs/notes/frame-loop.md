# Spider-Man frame loop (SLUS_008.75)

`GameLoop_run` (0x8002C174) is the gameplay and attract-demo loop main enters with selector 0. It runs while
`Mode_primaryState` (gp+0x740, 0x800B4F34) is zero; the value it stops on is main's next selector.

One iteration:

| call | what |
|---|---|
| `Frame_begin` 0x800612B8 | swaps the double buffer (0x8009A6E4 / 0x8009A75C), `ClearOTagR(db->ot, 0x1000)`, resets the packet pointer gp+0xCBC |
| 0x8002BB9C | gp+0x7F4: a second packet area at the buffer's base + 0x16F00 |
| 0x8002BBD4 | the logic update: object lists, player, camera, script |
| 0x80062CE0 | per-level timed state, advanced by the fields since its last run (`Field_counter` minus gp+0xD0C) |
| `Render_walk` 0x8002BD5C | the draw: world, then each object list's draw method into the OT the frame cleared |
| `OT_compact` 0x8007F930 | `(db->ot, 0x1000)`: drops empty OT entries |
| `Field_wait(1)` at 0x8002C26C | only when `Field_counter` did not move during the work (`GameLoop_idleWaitReturn`) |
| `Field_wait(1)` at 0x8002C28C | then `DrawSync(1)`; while the GPU is busy, `Sys_service` and wait again (`GameLoop_frameWaitReturn`) |
| `Frame_submit` 0x80061308 | `DrawSync`, `PutDispEnv(db+0x5C)`, `PutDrawEnv(db)`, `DrawOTag(db->ot + 0x3FFC)` |
| `Sys_service` 0x8005E234 | once per frame after the submit |

`Field_wait(n)` (0x8005E748) spins until `Field_counter` (gp+0xC74, 0x800B5468) is n past its value at the call;
the VBlank callback is its only incrementer. A frame therefore lasts two fields when its work fits in one, and
max(2, K + 1) when the work ran past K field edges: the game is designed for 30 fps. 0x80062CE0 and other readers
of `Field_counter` scale by the field delta, so a run at one field per frame changes their rate.

## Boot

`Boot_init` (0x8006BF9C, main's first call) plays movie 0, then movie 1 unless a button is down
(`Movie_play` 0x8002AA0C), reads `Field_counter` into s4, loads the shared models and fonts, then polls `Pad_read`
(0x8006B514, call returning to 0x8006C304) until 300 fields have passed since that read or a button flag in the
pad record at 0x800A4DF4 (+0x30, +0x40 with +0x60, +0xE0) is set. Nothing draws during the poll: it is a floor of
five seconds under the boot loads.

`Movie_play` reads the pad each movie frame and, from movie frame 30 on (`[gp+0x6AC]`), stops at a set flag at
0x800A4ED5 or 0x800A4E25.

`Movie_play` keeps `Movie_playing` (gp+0x690, 0x800B4E84) set while it runs. `Movie_fetchSector` (0x8002B3CC) sets
`Movie_ended` (gp+0x68C, 0x800B4E80) when the fetched frame number reaches the movie's last (gp+0x6B4); the player
then stops after that frame and runs its own teardown.
