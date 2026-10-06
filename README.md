# RetroBoxDB GameAndWatch

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Nintendo Game & Watch. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 54 source ZIPs, 0.1 MiB (No-Intro 54); 54 ROM files, 0.2 MiB uncompressed |
| Stored size | populated database 3.5 MiB; public Catalog 2.1 MiB (no ROM data) |
| Ratio | 2356.2% of the source ZIPs, 2004.3% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 4 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 32 MiB (32 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (52 files, each checked against the DAT hashes): 1.1 MiB/s, 3 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 0.01 s, TorrentZip 0.011 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.GameAndWatch.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-GameAndWatch/releases/latest/download/RetroBoxDB.GameAndWatch.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-gameandwatch-games.csv) / [summary](reports/ra-gameandwatch.json), [build report](reports/gameandwatch-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

4 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-gameandwatch.json`): smallest 4 KiB / 32 MiB at 0.13 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 4 KiB / 32 MiB at 0.13 MiB. ZIPs 0.15 MiB, per-file LZMA 0.14 MiB.

- Dumps of the microcontroller ROMs (1.8–4 KiB) of the handheld games; no internal header is defined, so every ROM stays `unclassified` and only the file is recorded.
- RetroAchievements has no Game & Watch console; the RA report is empty.
- The platform is tiny (54 ZIPs, 0.3 MiB), so the populated database is much larger than its ZIPs: the embedded engine, documents and reports take about 3 MiB in every database.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 53 / 51 / 53 |
| DAT coverage per version | 20260512-134045: 53/54; 20260512-134245: 53/54 |
| Local ROMs in no DAT | 0 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 0, RA only 0, hash not in the latest RA snapshot 0 ([list](reports/ra-gameandwatch-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-gameandwatch-missing.csv) |
| No-Intro DB Export + Dump Log 20260512-134045 | 53 archives, 54 file identities, 17 documented hardware assertions; Dump Log Verified 2 |
| RetroAchievements | not supported by RetroAchievements (no console, no snapshot) |
| Chinese names | 53 of 53 rows translated (46 unique); 53 local ROMs have a Chinese name |
| Populated-database audit | 57 objects, 1 groups, 58 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.GameAndWatch.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.GameAndWatch.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.GameAndWatch.sqlite --discover --ra --catalog RetroBoxDB.GameAndWatch.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
