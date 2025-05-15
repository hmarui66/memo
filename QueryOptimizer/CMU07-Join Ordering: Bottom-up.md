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
