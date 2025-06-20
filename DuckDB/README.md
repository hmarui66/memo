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

