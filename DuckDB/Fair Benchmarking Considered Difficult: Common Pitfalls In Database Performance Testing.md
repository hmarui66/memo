# Fair Benchmarking Considered Difficult: Common Pitfalls In Database Performance Testing

https://hannes.muehleisen.org/publications/DBTEST2018-performance-testing.pdf

## ABSTRACT

パフォーマンスベンチマーキングは、科学文献および産業界の出版物の両方において、異なるシステムやアルゴリズムを比較するために最も一般的に使用される方法の一つです。パフォーマンス測定は表面的には客観的に見えるかもしれませんが、意図的または偶発的に、あるシステムを別のシステムよりも有利にするためにベンチマーク結果に影響を与える多くの方法が存在します。本論文では、DBMS（データベース管理システム）のパフォーマンス比較における一般的な落とし穴についての調査を行い、公正なパフォーマンス比較を行うためにそれらをどのように発見し、避けるべきかについて助言を提供します。これらの一般的な落とし穴は、本来存在しないはずの大きなパフォーマンスの差異を示す一連の模擬ベンチマークによって説明されます。
(NotebookLM翻訳)

## 1 INTRODUCTION

- 性能ベンチマークをする際に、過去の研究結果と比較する場合は基本的な利害の対立
- configuration によって以前の研究が不利になることもある
- 企業のベンチマークでは顕著になる(benchmarketing)

## 2 RELATED WORK

### 2.2 Benchmarking DBMSes

- O'Neil's の設定とレポート要件のチェックリストにおける重要な点
  - データ
    - 指定された正確な方法で生成されるべき
  - レポート
    - database のローディングとディスクスペースのリソース使用率
    - 正確なハードウェア/ソフトウェア構成
    - 効果的な query plans
    - 統計情報の収集の overhead
    - 経時的なメモリ使用量
    - 経過時間
    - CPU 時間
    - I/O 操作回数
- Nelson は O'Neil's のリストを拡張(後述)

## 3 COMMON PITFALLS

> All experiments in this section were run on a desktop-class computer with an Intel i7-2600K CPU clocked at 3.40GHz and 16 GB of
main memory running Fedora 26 Linux with Kernel version 4.14. We
used GCC version 7.3.1 to compile systems

### 3.1 Non-Reproducibility

### 3.2 Failure To Optimize

### 3.3 Apples vs Oranges

### 3.4 Overly-specific Tuning

### 3.5 Cold vs Hot Runs

### 3.6 Cold vs Warm Runs

### 3.7 Ignoring Preprocessing Time

### 3.8 Incorrect Code

## 4 CONCLUSIONS AND OUTLOOK
