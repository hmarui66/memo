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

- a
  - select は table scan, index scan or index search を用いて実装可能
  - join は nested loop join, hash join, merge join のいずれかで実装可能
- b
  - nested loop join は S, T の join サイズが小さく、R.a に index が存在する場合、最も効率的となる可能性がある
- c
  - S.b, R.a にインデックスがある場合、すなわち merge join が必要とする sort を index が提供できる場合効率的
- d
  - S, T の結合サイズが大きい場合に最適となる可能性がある

### Cost estimation

- 実行プランによってかかる時間や消費リソース(e.g., CPU, memory, I/O)が大きく変わりうる
- 大きな database 上での複雑なクエリにおける、良いプランと悪いプランでは、実行時間が数桁変わり得る
- 良いプランを選択するために、たいていの query optimizer は cost model を活用
  - 実行プランの作業を推定し、相対比較が正確になるように
  - 特に physical operator はそれを実装する algorithm によって実施される作業を見積もる必要がある
    - 見積もりはその operation への input/output relations のサイズや統計的特徴を必要とする
- cost は少なくとも 3 軸を(CPU, memory, I/O)を持つが、cost model はそれらを組み合わせた multi-dimensional costs を用いてプラン同士を比較可能にする

### Search algorithm

- 原則として実行プランを網羅的に列挙し、コスト見積もりを呼び出して各プランのコストを決定し、推定コストが最も低いプランを見つける
- プランの一部は共通の logical/physical operator を共有できるので、重複探索を避けるために列挙を慎重におこなう
- 網羅的な列挙はコストがかかりすぎる可能性があるので、品質を大きく損なわずに列挙コストを削減する必要がある

良い optimizer は、

- (a) 有望なプランの十分に大きな search space を考慮
- (b) 実行プランのコストを十分に正確にモデル化、コストが大きく異なるプランを区別
- (c) 低コストのプランを効率的に見つける search algorithm を提供

## 1.2 System R Query Optimizer

1.1 で言及した key challenges に対して System R の query optimizer がどう対処したのかを説明。
→開発されたテクニックは後続の query optimizer に大きな impact

### Search space

- cost-based & クエリの Select-Project-Join(SPJ) class にフォーカス
- Select operation を実装するため physical operator には Table Scan & Index Scan が含まれていた
- Join については Nested Loops Join & Merge Join(両方の inputs が join column でソートされている必要がある)
- Fig. 1.2, 1.3 にあるように SPJ queries に対していくつかの logical query trees と execution plans が存在する
  - join は associative & commutable なので、Scan と Join operations に対して複数の physical operator の選択肢があるため
- SPJ queries のために System R が探索する logical query tress の space は binary join operations の線形シーケンスの space が含まれていた
  - e.g., $Join(Join(R, S), T), U)$

<img width="417" alt="image" src="https://github.com/user-attachments/assets/46cf706e-5859-40cd-a9b3-99933535146b" />

- 1.4a: Join operations の線形シーケンスの logical query tree の例
- 1.4b: a bushy plan で System R の search space には無い

### Cost model

- 実行プラン内の各 operator の CPU, I/O コストを見積もるための数式を使用
  - メモリのコストは組み込まれていない
- テーブル & インデックスに関する統計情報をメンテ
  - 行数(カーディナリティ)
  - データページ数
  - インデックスページ数
  - 各列の unique 値数
  - ...
- single selection や join 述語の選択性を算出する数式
