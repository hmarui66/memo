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
  - 主要な RDBMs における Query execution は iterator model にしたがってお手、Open, GetNext, Close メソッドを実装
