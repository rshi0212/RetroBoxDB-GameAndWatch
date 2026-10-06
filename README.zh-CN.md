# RetroBoxDB GameAndWatch

[English](README.md) | 中文

任天堂 Game & Watch的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 54 个，0.1 MiB（No-Intro 54 个）；解压后 ROM 54 个，0.2 MiB |
| 入库后大小 | 完整库 3.5 MiB；公开 Catalog 2.1 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 2356.2%，为解压后 ROM 总量的 2004.3% |
| 使用的技术 | 存储 v4：4 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 32 MiB 的 LZMA2 实体组（字典 32 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，52 个文件，逐个按 DAT 哈希校验）：1.1 MiB/s，平均 3 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 0.01 秒，TorrentZip 平均 0.011 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.GameAndWatch.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-GameAndWatch/releases/latest/download/RetroBoxDB.GameAndWatch.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 各平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-gameandwatch-games.csv)／[汇总](reports/ra-gameandwatch.json)、[构建报告](reports/gameandwatch-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

全部本地收藏实测 4 种块／组组合（`assessment/data/storage-experiment-gameandwatch.json`）：最小为 4 KiB / 32 MiB 0.13 MiB；按规则（最小值 0.5% 以内选块最小、再选组最小）采用 4 KiB / 32 MiB 0.13 MiB。ZIP 0.15 MiB，逐文件 LZMA 0.14 MiB。

- 掌机游戏微控制器 ROM 的 dump（1.8–4 KiB）；没有定义内部头部，所有 ROM 记为 `unclassified`，只记录文件本身。
- RetroAchievements 没有 Game & Watch 主机，RA 报告为空。
- 平台极小（54 个 ZIP、0.3 MiB），完整库远大于 ZIP：每个库都内嵌约 3 MiB 的引擎、文档和报告。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 53／51／53 |
| 各版 DAT 覆盖 | 20260512-134045：53/54；20260512-134245：53/54 |
| 不在任何 DAT 的本地 ROM | 0 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 0，仅 RA 收录 0，哈希不在最新 RA 快照 0（[清单](reports/ra-gameandwatch-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-gameandwatch-missing.csv) |
| No-Intro DB Export＋Dump Log 20260512-134045 | 53 个档案、54 个文件身份、17 条有文档的硬件声明；Dump Log Verified 2 |
| RetroAchievements | RetroAchievements 不支持该平台（无主机、无快照） |
| 中文名 | 53 条记录中 53 条有中文（46 个唯一名）；本地 ROM 53 个有中文名 |
| 完整库审计 | 57 个对象、1 个组、58 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.GameAndWatch.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.GameAndWatch.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.GameAndWatch.sqlite --discover --ra --catalog RetroBoxDB.GameAndWatch.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
