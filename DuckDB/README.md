# DuckDB

調べたことメモ

## DuckDB のドキュメント

### Peer-Reviewed Papers and Thesis Works

https://duckdb.org/why_duckdb#peer-reviewed-papers-and-thesis-works

- Runtime-Extensible Parsers (CIDR 2025)
- Robust External Hash Aggregation in the Solid State Age (ICDE 2024)
- These Rows Are Made for Sorting and That's Just What We'll Do (ICDE 2023)
- Join Order Optimization with (Almost) No Statistics (Master thesis, 2022)
- DuckDB-Wasm: Fast Analytical Processing for the Web (VLDB 2022 Demo)
- Data Management for Data Science - Towards Embedded Analytics (CIDR 2020)
- DuckDB: an Embeddable Analytical Database (SIGMOD 2019 Demo)

### Standing on the Shoulders of Giants

https://duckdb.org/why_duckdb#standing-on-the-shoulders-of-giants

Thomas Neumann の研究がかなり参照されている

## DuckDB のブログ

### Querying Parquet with Precision Using DuckDB

https://duckdb.org/2021/06/25/querying-parquet.html

- Parquet を効率的に読める
- Pandas との比較で圧倒
- Python ライブラリで relational API も使える(効率は維持)

```py
con.from_parquet('alltaxi.parquet')
   .aggregate('passenger_count, count(*)')
   .df()
```

### Fastest Table Sort in the West – Redesigning DuckDB’s Sort

https://duckdb.org/2021/08/27/external-sorting.html

- 並列ソートとメモリを超えるデータサイズもサポート
- Goetz Graefeのデータベースシステムにおけるソートの[サーベイ](http://wwwlgis.informatik.uni-kl.de/archiv/wwwdvs.informatik.uni-kl.de/courses/DBSREAL/SS2005/Vorlesungsunterlagen/Implementing_Sorting.pdf)
- ソートのコストは、値の比較とデータの移動によって大きく左右
- 複数の order by 句がある場合 2 つの実装がある
   - 複数の句の比較をしながら loop
      - 全データの行ごとに if/else を実行する loop が必要
      - columnar storage の場合 columns を jump し memory への random access が発生
   - 最初の句でソートし、その後次の句でソート
      - 重複した値があると特に非効率
- binary 文字列比較で比較演算を簡素化しソート性能向上
   - すべての order by を単一の binary sequence に encode
   - memcmp で比較
- Radix sort
   - 分布ベース
   - quicksort や mergesort は $O(n \log n)$ の時間計算量だが、 Radix sort は key の幅 $k$ に対して $O(nk)$  の時間計算量
   - → $n$ が大きいほどスケール
- Two-Phase Parallel Sorting
   - Morsel 駆動並列処理を採用
      - https://15721.courses.cs.cmu.edu/spring2016/papers/p743-leis.pdf
   - 2 phase
      1. 複数のスレッドがテーブルからほぼ同量のデータを並列に収集 → 収集したデータを radix sort
      2. 各スレッドのソート済データを merge sort
   - merge sort の実装
      - K-way merge
         - K 個のリストを1つのソート済リストに1回のパスでマージ
      - Cascade merge
         - ソート済リストが1つになるまでソート済リストを2つずつマージ
         - K-way merge よりも効率的で in-memory sort に使用される
         - →こちらを採用(高い in-memory 性能を実現するため)
            - ソート対象のブロックがマージされていくと、スレッドを稼働するのに十分なブロック数を確保できなくなる
            - 特に最後の2つのブロックのマージは1つのスレッドで処理するため処理速度が低下する
            - この phase を並列化するために Merge Path を実装
               - https://arxiv.org/pdf/1406.2628
- Columns or Rows?
   - DuckDB は列指向レイアウトでデータを保持
      - vectorized execution engine で処理しやすい
      - SIMD 活用する機会も豊富
   - table を sort する際は row 全体でシャッフルする必要がある
      - 列レイアウトを維持することもできるが、キー列をソート→ペイロード列をソートが必要になる
         - メモリ内で random access が必要になり、ペイロード列が多いと遅くなる
      - 列を行に変換すると並び替えは簡単だが、列→行 だけでなく 行→列 の変換も必要になる
      - そもそも外部ソートサポートのため disk にオフロード可能な buffer 管理ブロックにデータを格納するため変換は実質無料
   - join, aggregation など行ベースの演算子もあり、これらのための統一された行レイアウトがあるのでソートでも利用
- External sorting
   - buffer manager は memory から disk へ block を unload できる
      - ソートにおいて積極的には実施されず、memory がいっぱいになりそうな場合のみ
      - least-recently-used の queue は appendix で説明
   - 整数のような固定サイズの列は unload は簡単だが、文字列のような可変サイズの列では難しい
   - 行レイアウトは固定サイズの行を使用するため任意のサイズの文字列は格納できない
      - 文字列はポインタで表現し、実際の文字列データは別の memory block(string heap)を指す
      - heap を変更して、buffer-managed block に文字列を行ごとに保存
   - 行レイアウトに heap 内の先頭を指す 8 バイトの `pointer` フィールドを追加
      - memory 内表現では不要だが disk 上で役立つ
   - データが memory に収まる場合
      - heap block は固定されたまま
      - sort 時に行のみが並び替えられる
   - データ memory に収まらない場合
      - block を disk に offload
      - sort 時に heap も並び替え
      - heap block が disk に offload されると block を指すポインタは無効化
         - load し直すと pointer は変更されてしまう
         - 8 バイトの `pointer` フィールドを heap block 内のどこにあるかを示す `offset` フィールドで上書き
            - → pointer swizzling (と書いてあるけど、unswizzle では?)
         - load し直す場合は pointer に変換
- Comparison with Other Systems
   - ClickHouse, HyPer, Pandas, SQLite と比較
   - M1 Mac で実行しており HyPer は x86 エミュレーターで動作するので不利
   - 結果のサマリは DuckDB が概ね良好(ベンチマーク設定の偏り具合は判断できず)
- 参考: [These Rows Are Made for Sorting and That’s Just What We’ll Do](https://hannes.muehleisen.org/publications/ICDE2023-sorting.pdf) ICDE'23

### Windowing in DuckDB

https://duckdb.org/2021/10/13/windowing.html

Pipeline Breaking:

window 演算は関数の計算前に入力を全て読み込む必要がある。

```sql
SELECT "Plant", "MWh"
FROM (
    SELECT "Plant", "MWh",
        rank() OVER (
            PARTITION BY "Plant"
            ORDER BY "Date" DESC) AS r
    FROM table) t
WHERE r = 1;
```

→テーブル全体をマテリアライズ、パーティション分割、パーティションをソート、パーティションから単一の行を取得、が必要

```sql
SELECT table."Plant", "MWh"
FROM table,
    (SELECT "Plant", max("Date") AS "Date"
     FROM table GROUP BY 1) lasts
WHERE table."Plant" = lasts."Plant"
  AND table."Date" = lasts."Date";
```

テーブルスキャンは2回必要だが、こちらの join クエリの方が速い。

あるデータセットにおいては 20 倍くらい差があったとのこと。
ただ、window は分析に不可欠なのでできるだけ高速化を行なっている。

高速化テクニック:

- partitioning and sorting
    - partition と order の両方でソートする必要あり、リソースを消費する
    - Leis の提案手法であるパーティションの分割スキームを採用
        - https://www.vldb.org/pvldb/vol8/p1058-leis.pdf
    - まず input をスレッドに対して分配し、各スレッドでハッシュベースのチャンクに分けて、スレッド間で統合し、ソートする
    - DuckDB の sorting の改善も反映されパフォーマンス向上
- aggregation
   - Naive Windowed Aggregation
      - 最も簡単な集計方法: 状態の初期化→window frame内のすべての値で状態を更新→finalizeで集計値を生成
      - 非効率
      - 累計を計算するにはすべての値ごとに累計を再加算して $O(N^2)$
   - Segment Tree Aggregation
      - DuckDB も採用
      - Leis の提案手法
      - tree 状に集計値を記録しておき、window frame ごとに必要な集計を最小限に抑える
      - <img width="675" alt="image" src="https://github.com/user-attachments/assets/1f210034-7ba3-49bf-9f1f-dbdc3fababfa" />
   - General Windowed Aggregation
      - Segment Tree の欠点は、多数の中間状態を管理する必要があること
      - holistic aggregation では全体を考慮して集計を行う必要があり、状態管理が煩雑になる
         - `mode`, `quantile` など
      - Wesley and Xu's の手法を使用
         - https://www.vldb.org/pvldb/vol9/p1221-wesley.pdf
         - segment tree を集約固有のデータ構造に一般化
   - Ordered Set Aggregation
      - window function は SQL 標準で定義されている特殊な ordered set aggregation と密接に関連
      - 一部の DBMS は window 関数でもこの処理を使用しているが、sort が不要なのであまり効率的ではない
      - DuckDB はより高速な集計関数に変換

### Parallel Grouped Aggregation in DuckDB

https://duckdb.org/2022/03/07/aggregate-hashtable.html

- group by をどのように計算するか、多くの設計判断が必要
- 主な問題: input テーブル内でグループが任意の順序で出現する可能性があること
   - input がすでに grouping 列でソートされている場合は、現在の値を以前の値と比較するだけでよい
   - grouping された集計のため、最初に input をソートすれば良いがソートの計算量は高い
- Hash Tables for Aggregation
   - 行を追加するには sort よりも遥かに簡単
   - 集計値をエントリとして保持すればよいが、collision に対応する必要がある
   - linear probing を採用
   - シンプルな構造の hash table のサイズ変更時は、すべてのデータを移動する必要がありコストが高い
   - サイズ変更を効率的にサポートするために、grouping 値とグループの集約値を含む payload block と、それを指す pointer 配列で構成される2部構成の hash table を実装
      - サイズ変更時
         - pointer 配列を破棄してより大きな配列を割り当て
         - すべての payload block を再度読み取って grouping 値を hash しそれらへの pointer を新たな pointer 配列へ再挿入
         - 再 hash のコストを減らすため payload block に raw hash 値を
         - <img width="513" alt="image" src="https://github.com/user-attachments/assets/d3d40cbd-0d44-44f1-8c9c-ae492a2a7032" />
      - 欠点はエントリの検索
         - pointer 配列と payload block 内のエントリに順序付けがないので memory 階層で random access による stall
         - pointer 配列に grouping hash 値の 1-2 バイトを追加し、検索時に hash bit が一致した場合のみ pointer をたどることで最適化
- Parallel Aggregation
   - parallelism work と hash table は一般的にうまく機能しない
   - 各スレッドに下流の演算子からデータを読み取らせて個別の local hash table を構築させ、後で1つのスレッドからマージすることもできる
      - グループが少なければうまく機能する
      - input と同じ数のグループが存在する可能性があり、その場合は機能しない
   - parallel merge of the parallel hash tables が必要
      - Leis の手法を採用
         - https://15721.courses.cs.cmu.edu/spring2016/papers/p743-leis.pdf
      - 各スレッドは group hash 上の radix-partitioning に基づいた複数の partitioned hash tables を構築
      - <img width="536" alt="image" src="https://github.com/user-attachments/assets/814ac41d-05a2-4e7b-a529-956cbd2c5482" />
      - phase1: スレッド間通信を必要とせず、hash 値を用いてグループの独立した partition を作成
      - phase2: worker thread に個別の partiiton を割り当ててマージ
         - partition は hash の radix partitioning scheme を用いて作成されるため、worker は独立してマージできる
   - 2つのさらなる最適化
      - hash table の分割は単一スレッドの集約 hash table のエントリ数が固定制限を超えたときのみ
         - 分割とマージにはコストがかかるため
      - hash table の pointer 配列が特定のしきい値を超えると hash table への値の追加を停止
         - その後、すべてのスレッドが分割可能な hash table の複数のセットを構築
         - 多数の異なる group を持つデータセットで特に効果的で、input において grouping 値が何らかの形で密集している場合に有効
            - 日付順に並べられたデータセットを日ごとにグルーピングする場合など

### Analytics-Optimized Concurrent Transactions

https://duckdb.org/2024/10/30/analytics-optimized-concurrent-transactions.html

- 分析用途だと書き込みが競合することは稀なので、楽観的同時実行制御
- Neumann の論文の MVCC に着想を得た方式を採用
    - https://15721.courses.cs.cmu.edu/spring2019/papers/04-mvcc2/p677-neumann.pdf
- 行単位で以前のバージョンを UNDO バッファに持つのはトランザクションワークロードに適しているが、分析用途には適してない
    - 一括更新することが多く、行ごとの UNDO バッファは膨大になる
    - 圧縮しているためインプレース更新ができない
- DuckDB は列ごとに一括バージョン情報を持つ
    - 2048 行のバッチごとに単一のバージョン情報エントリを保存
    - バージョン情報には古いデータではなく、データに加えられた変更を保持
    - チェックポイントで disk にフラッシュ
- DuckDBは分析ユースケースでよく見られるデータの一括変更に最適化された唯一のトランザクション型データ管理システム
- WAL も採用
    - 大きな csv ファイルなどを取り込む場合、WAL とデータブロックに多重に読み書きするのを防ぐ工夫をしている
    - 直接データブロックに書き出して、WAL から参照
    - ロールバック時には該当領域は空きブロックとしてマーク
 
## DuckDB に関するブログ

- [🦆🦆🦆🦆🦆🦆DuckDB入門🦆🦆🦆🦆🦆🦆](https://zenn.dev/notrogue/articles/1193d0ab8d8eda)
   - [SQLite との比較](https://zenn.dev/notrogue/articles/1193d0ab8d8eda#sqlite%E3%81%A8%E3%81%AE%E6%AF%94%E8%BC%83)
   - [Sortに関する論文: Efficient External Sorting in DuckDB](https://ceur-ws.org/Vol-3163/BICOD21_paper_9.pdf)
- [シリーズ：DuckDBが採択した論文：MonetDB/X100: Hyper-Pipelining Query Execution
](https://yohei.codes/jp/2024/08/16/paper-monet-db-x-100.html)
