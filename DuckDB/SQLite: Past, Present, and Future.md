# SQLite: Past, Present, and Future

https://www.vldb.org/pvldb/vol15/p3535-gaffney.pdf

DuckDB との比較の箇所を中心にメモ。
("DuckDB" で検索すると 79 件もヒットする)

DuckDB との比較はどちらが優れているかという観点ではなく、用途が異なるエンジン間での差異を明らかにするという観点で実施されている。

## OLTP ベンチマーク結果

<img width="844" alt="image" src="https://github.com/user-attachments/assets/9cd0fb88-db58-4ebd-895a-f85bb220c996" />

- TATP: Telecom Application Transaction Processing
  - 80%は読み取り専用で、20%は更新、挿入、または削除を伴うトランザクション
- テーブルのレコード数が 10K → 100K → 1M とスケールするにつれて throughtput の違いは 10X → 50X → 500X と拡大

## OLAP ベンチマーク結果

<img width="851" alt="image" src="https://github.com/user-attachments/assets/b5c8f5ee-2fa7-4854-924f-276c05b8a820" />

- SSB: Star Schema Benchmark
  - TPC-H スキーマを改変して使用
  - fact table と dimension tables の join 結果を dimension table の属性で filter
- 最大で DuckDB が 30-50X 速く、最小でも DuckDB が 3-8X 早い

<img width="843" alt="image" src="https://github.com/user-attachments/assets/fb409fab-b1ff-472c-a659-50defa4d5456" />

- LIP: Lookahead Information Passing
  - join を開始する前にすべての inner (dimension) table に対してブルームフィルターを作成
  - ブルームフィルター join 操作に渡す
  - join の pipeline を最適化
- 結果、もともとの SQLite に比べて cloud server での latency が 2.7-7X 改善
