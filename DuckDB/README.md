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
- Goetz Graefeのデータベースシステムにおけるソートのサーベイ
- ソートのコストは、値の比較とデータの移動によって大きく左右

