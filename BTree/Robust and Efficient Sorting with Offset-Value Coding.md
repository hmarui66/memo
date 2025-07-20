# Robust and Efficient Sorting with Offset-Value Coding

https://www.arxiv.org/abs/2209.08420v1

## alphaxiv の日本語ブログ

https://www.alphaxiv.org/ja/overview/2209.08420v1

Offset-value conding(OVC), tree-of-losers priority queue, データ圧縮技術をを統合することでソート効率を劇的に改善するアプローチを提示。

OVC は 1977年 に提案されたもののこれまで顧みられてこなかった技術だが、 poor man's normalized key の一種として機能する。

### OVC のメカニズム

ある行のキー値をソート順における別の行(通常は以前の比較の勝者)と比較して符号化。符号化は、

1. offset: 2つのキー間で最初にことなる列の位置
2. 値: 異なる位置における実際の値

を捉える。これらは単一の順序保存整数に結合される。この符号化はソート処理を通じて行が移動する際に動的に更新され、比較の労力が必要な場所にのみ集中。

$$OVC = encode(offset, value)$$

### tree-of-losers との統合

tree-of-losers priority queue は、1行あたり約 $log_n(N)$ 回の比較に近づく、ほぼ最適な比較数を達成。次のように機能する。

- 各リーフはデータソース(run または個々の行)を表す
- 内部ノードは比較の loser を格納
- root には全体の勝者を含む

OVC と組み合わせることで、ほとんどの比較が高価な列ごとの評価ではなく、単純な整数操作となる。

### 圧縮と I/O 最適化

offset 値は前の行と同一である列の数を示す。共通の先行値を冗長に保存しない prefix 切り捨て圧縮が可能。

### クエリパイプライン統合

OVC はオペレーターからオペレーターに持ち運びできる。

### 計算の複雑性

従来のソートの $O(N * log N)$ ではなく $O(K * N)$ を可能にする。
