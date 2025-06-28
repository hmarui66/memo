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

> 計算結果の記事は広告であり、学術ではない。実際の研究成果とは、その結果を生み出した完全なソフトウェア環境、コード、データである

再現可能な実験は、すべての設定パラメータが既知であることが重要。

- マシンの OS
- サーバーのインストール方法
- サーバーのバージョン
- サーバーのセットアップ方法
- サーバーの設定フラグ
- ソースコード

### 3.2 Failure To Optimize

benchmark による既存システムとの比較において、筆者が既存システムを適切に最適化するインセンティブはない。

![image](https://github.com/user-attachments/assets/1267fffe-59a7-4a59-8197-f35a35952f47)

- (a)コンパイルフラグによる違い
- (b)configによる違い

DBMSを特定のワークロードに最適化することは非常に複雑。

- ガイドラインを読むなどして僅かな最適化をするだけでもより公正に
- 比較するシステムの代表者を参加させる

### 3.3 Apples vs Oranges

- 2つのシステムの性能比較は、両システムが全く同じ機能を実行する場合にのみ公平
- 小さな standalone プログラムを本格的な DBMS と比較するようなケース
    - standalone プログラムは特定のタスクだけ実行
    - 比較対象の DBMS は任意のクエリ実行、トランザクション分離の維持、データの更新、並行してクエリを発行する複数のクライアントの処理...
- 機能の違いによるパフォーマンス特性の違い
  - 例: overflow 処理はオーバーヘッドなので、実装する/しないで差が生まれる

### 3.4 Overly-specific Tuning

- TPC-H や TPC-C のような標準化されたベンチマークを使うのは再現性を高める大きな一歩
- ただ、特定のベンチマークに向けてシステムやアルゴリズムを過度にチューニングする問題も発生
- 標準化されたものはベースラインに用いつつ、異なるクエリセットを実行する必要がある

### 3.5 Cold vs Hot Runs

- cold, hot run のパフォーマンスは別々に収集すべき

### 3.6 Cold vs Warm Runs

- cold run データを収集するには、database server を停止、OS のキャッシュをすべて削除、server を再起動、1つのクエリを実行、を繰り返す
- クラウド環境では仮想ホストのキャッシュをクリアする

### 3.7 Ignoring Preprocessing Time

- setup時間が無視されることが良くある
- index 作成により多くの時間を費やすことが、より高速な index 作成につながることも多い
- 文字列値の自動辞書エンコーディングなども

### 3.8 Incorrect Code

- 特定のデータセットでのみ動作する、などのケースもある
- ちゃんとチェックするしかない
- よくテストされている SQLite や PostgreSQL などの結果と比較

## 4 CONCLUSIONS AND OUTLOOK

- 筆者らもこれらの問題には無縁ではいられない
- 再現性をサポートする仮想マシンイメージを提供することは有用

## A FAIR BENCHMARK CHECKLIST

本論文で述べられている一般的な落とし穴を避けるためのこのチェックリスト。

- ベンチマークの選択 (Choosing your Benchmarks)
    - [ ] ベンチマークが評価空間全体をカバーしているか
    - [ ] ベンチマークのサブセットを選択する理由を正当化しているか
    - [ ] ベンチマークが評価空間における機能を強調しているか

- 再現性 (Reproducible) 以下のものが利用可能であること
    - [ ] ハードウェア構成
    - [ ] DBMSのパラメータとバージョン
    - [ ] ソースコードまたはバイナリファイル
    - [ ] データ、スキーマ、クエリ

- 最適化 (Optimization)
    - [ ] コンパイルフラグ
    - [ ] システムパラメータ

- 同等条件での比較 (Apples vs Apples)
    - [ ] 同様の機能
    - [ ] 同等のワークロード

- 同等なチューニング (Comparable tuning)
    - [ ] 異なるデータ
    - [ ] 様々なワークロード

- コールド/ウォーム/ホット実行 (Cold/warm/hot runs)
    - [ ] コールド実行とホット実行を区別しているか
    - [ ] コールド実行：OSとCPUキャッシュをフラッシュしているか
    - [ ] ホット実行：最初の実行を無視しているか

- 前処理 (Preprocessing)
    - [ ] システム間で前処理が同じであることを確認しているか
    - [ ] 自動インデックス作成に注意しているか

- 正確性の確保 (Ensure correctness)
    - [ ] 結果を検証しているか
    - [ ] 異なるデータセットでテストしているか
    - [ ] コーナーケースが機能するか

- 結果の収集 (Collecting Results)
    - [ ] 干渉を減らすために複数回実行しているか
    - [ ] 複数回の実行で標準偏差を確認しているか
    - [ ] ロバストなメトリック（例：中央値と信頼区間）を報告しているか
