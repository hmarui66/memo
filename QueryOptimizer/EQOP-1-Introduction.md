# wip: Extensible Query Optimizers in Practice: 1 Introduction

https://www.microsoft.com/en-us/research/wp-content/uploads/2024/12/Extensible-Query-Optimizers-in-Practice.pdf

- SQL は relational data を query するための high-level な宣言的言語
- relational data の事実上の de-facto standard なクエリ言語で、主要な RDBMs でサポートされている
- SQL は selections, joins, group-by, aggregation, nested sub-queries を含む relational data に対する query を宣言的に指定できる

以下の Query に対して、RDBMs での SQL query 処理の workflow における主要 step は

```sql
SELECT *
FROM R, S, T
WHERE R.a = S.b AND S.c = T.d AND T.e = 10
```

![image](https://github.com/user-attachments/assets/cbd03279-30d9-4ea4-bb12-fa98d444f2cb)

- Parsing and validation
  - クエリが SQL 構文を遵守し、データベースのテーブルやカラムへの参照のみ含むことを確認
  - output は logical query tree
  -   logical relational operators(e.g. Select, Join)の tree の形式の代数的表現
- Query optimization
  - logical query tree を input としてとり、効率的な execution plan を生成
  - execution plan
    - query execution engine によって解釈やコンパイルされる
    - physical operations の tree
- Query execution
  - query optimizer から plan を取り、query results を生成するために plan を実行
  - physical operators の set を実装
  - physical operator の例
    - Table Scan
    - Index Scan
    - Index Seek
    - Hash Join
    - Nested Loops Join
    - Merge Join
    - Sort
    - → physical operator の algorithm は "Query Evaluation Techniques for Large Databases" を参照
  - 主要な RDBMs における Query execution は iterator model にしたがって、Open, GetNext, Close メソッドを実装
    - 各 iterator はそれぞれの状態を保持(hash table の size や位置)
    - non-leaf operation が出力行を生成する際には、子の operator に対して GetNext を呼び出す
    - execution engine に新たな operator を追加する際に利便性がある
    - → **pull model**
    - GetNext ごとの関数呼び出しの overhead があり、modern な CPUs においては性能面で不利
    - vectorization や code generation により効率性を向上させられる
      - トレードオフについては "Everything you always wanted to know about Compiled
and Vectorized Queries but were afraid to ask" を参照

## 1.1 Key Challenges in Query Optimization

多数の実行プランから効率的なプランを選択するために、

- 探索するプランの search space を決定
- cost 推定によって plan の相対的な効率を比較
- 効率的な探索 algorithm で実行コストが理想的には最も低い実行プランを見つける

必要がある。

### search space

- クエリの alternative equivalent execution plans で構成
  - 複雑なクエリだと大きくなる可能性
- クエリの代数表現は他の多くの equivalent な表現に変換できる可能性がある

<img width="507" alt="image" src="https://github.com/user-attachments/assets/272e7ceb-a806-456a-b8f7-1ac2099ec4c3" />

- logical operation に対して多くの異なる実装がある
- → logical query tree に対して多くの実行プランが存在

<img width="509" alt="image" src="https://github.com/user-attachments/assets/871daa63-aa3d-4c7f-bc67-73cce8c2d022" />

logical plan に対する実行プランの例


