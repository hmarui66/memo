# DuckDB: an Embeddable Analytical Database

https://duckdb.org/pdf/SIGMOD2019-demo-duckdb.pdf

## ABSTRACT

- SQLite の人気は目立たない in-process data management の必要性を示す
- 分析ワークロードにはそのようなシステムは存在しない
- 組み込み分析 SQL クエリ実行のために設計された DuckDB を demonstrate
- 性能を示すために他の data management solution と比較

## 1 INTRODUCTION

- data management system は大規模な monolithic database server に進化し、スタンドアロンプロセスとして動作
  - 同時に多くのクライアントからの要求に応える必要性、データの整合性要件によるもの
  - スタンドアロンプロセスは強力だが、セットアップする労力が必要かつ、データアクセスはクライアントプロトコルによって制限
- data management には別のユースケースがある
  - database system が host process 内で完全に動作するリンクされたライブラリとして組み込み
  - →SQLite
- SQLite は OLTP にフォーカスしており、OLAP ワークロードにおけるパフォーマンスは非常に低い

TBD

## 2 DESIGN AND IMPLEMENTATION

## 3 DEMONSTRATION SCENARIO

## 4 CURRENT STATE AND NEXT STEPS
