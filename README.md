# Spider-Man decomp

A decompilation of **Spider-Man** (USA, `SLUS-00875`) and **Spider-Man 2: Enter Electro** (USA, `SLUS-01378`),
set up with [splat](https://github.com/ethteck/splat). It holds the build-matched split configuration for both
executables, a symbol map from our own reverse engineering, and the RE notes. No C is matched yet.

The PC port [psx-spider-man](https://github.com/SomeoneIsWorking/psx-spider-man) consumes this repository as a
submodule and reads its guest addresses from the symbol map.

## Bring your own disc

No game bytes are ever committed here: not the executables, not overlays, not the split assembly or data. Copy the
executables from your own discs into `disc/` (gitignored):

```
disc/SLUS_008.75     Spider-Man
disc/SLUS_013.78     Spider-Man 2: Enter Electro
```

Each config carries the file's SHA-1 and splat refuses a mismatch.

## Split

```
uv run --frozen splat split config/slus_008.75.yaml
uv run --frozen splat split config/slus_013.78.yaml
```

Output goes to `asm/<image>/` and `build/`, both gitignored. splat is pinned in `pyproject.toml`/`uv.lock`.

## Layout

| path | what |
|---|---|
| `config/*.yaml` | splat configs, one per executable |
| `config/symbols/<image>.txt` | the symbol map, splat `symbol_addrs` format |
| `docs/re-frontier.md` | the ordered RE evidence chain |
| `docs/prior-art.md` | other Spider-Man work, licences and what we take |

Notes cite psx-spider-man paths (`titles/...`, `docs/issues/...`) where a port owner or an issue holds the evidence.
The overlays (`NAME.bin`, Enter Electro's `NAME.rel`) are loaded into the guest heap at run time and are not split
yet.

## Licence

MIT (`LICENSE`), covering this repository's own files only. Spider-Man and Spider-Man 2: Enter Electro are
© Activision / Marvel; nothing of them is distributed here.

Built with AI assistance (Claude and other LLMs).
