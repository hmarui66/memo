# CMU Optimize#07 Join Ordering: Bottom-Up

講義動画: https://www.youtube.com/watch?v=CcUVvnYv7Hg&list=PLSE8ODhjZXjYCZfIbmEWH7f6MnYqyPwCE&index=9

## 内容

基本的に [Adaptive Optimization of Very Large Join Queries](https://dl.acm.org/doi/pdf/10.1145/3183713.3183733) の論文に沿った説明がされていた。

- Query Graph Structure
  - Chain: リニアに join してシンプルな構造
  - Clique: すべての relation が互いに接続
- Adaptive Optimize
  - Small: DPHyp
  - Medium: linearized DP
    - query の linearize には IKKBZ algorithm を利用
  - Large:  Greedy Operator Ordering
- Postgres の genetic algorithm にも触れられていた
  - random algorithm はあまり良いアプローチではではない模様

## Adaptive Optimize: 4.2 Medium Queries

- あるポイントから DP が高価になりすぎる
- ただ the shape of the query graph に依存
  - linear query graphs であればとても大きな queries 解決できる
  - cliques or stars の最適化は難しい
- 100 relations までの medium size のクエリを、search space を線形化することで DP で扱いやすくする
- core idea: DP algorithm を制限
  - 線形の relation 順序の connected subchains のみ考慮
  - <img width="479" alt="image" src="https://github.com/user-attachments/assets/98b4ce83-4999-4533-abd0-321f07ac9679" />
  - full dp だと 17 エントリを埋める必要がある
    - $`O(2^n)`$
  - dp を制限した場合は 6 エントリを埋めるのみでよい
    - $`O(n^2)`$
  - hyper-graphs は扱えない
- linearize の方法が最終的な品質に大きなインパクト
- IKKBZ algorithm で linearize
  - <img width="367" alt="image" src="https://github.com/user-attachments/assets/1592843f-21b1-4373-bbbf-86e16eaf0d87" />
  - 多項式時間で acyclic query graph の optimal left-deep tree を見つけることができる
  - cost/benefit の rank ですべての joins をソート
- query graph が cyclic の場合、IKKBZ の実行前に minimum spanning tree を構築
  - IKKBZ は acyclic query graph を必要とするため

