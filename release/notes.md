GameAndWatch Catalog, storage v4 (4 KiB blocks, 1 solid LZMA2 group of up to 32 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release of this platform.
- RetroAchievements has no console for this platform; its report is empty.
- One schema for all twenty-three platforms: the header tables of the eight new platforms (Game Gear, PC Engine, SuperGrafx, MSX, MSX2, Virtual Boy, Game & Watch, Super A'Can: `pce_hardware`, `msx_hardware`, `vb_hardware`) and `rom_annotations` exist in every Catalog; tables of other platforms and provider tables have no rows.
- RetroAchievements reports look up sibling databases (NES<->FDS, SNES<->Satellaview, WonderSwan<->WonderSwan Color, NeoGeo Pocket<->NeoGeo Pocket Color, PC Engine<->SuperGrafx, MSX<->MSX2): a game whose ROM is stored there is `local_other_platform`, not a gap.
- Source: 54 ZIPs (nointro 54), 0.1 MiB (54 ROM files, 0.2 MiB uncompressed). Populated database: 3.5 MiB (2356.2% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 53 ROM records, 51 games, 53 releases; DAT versions: 20260512-134045, 20260512-134245.
- RetroAchievements: not supported for this platform.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 1.1 MiB/s (52 files); single file with a cold cache 0.01 s (ROM) / 0.011 s (TorrentZip) on average.
- Full audit of the populated database: 57 objects, 1 group, 58 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-GameAndWatch/blob/main/README.zh-CN.md)
