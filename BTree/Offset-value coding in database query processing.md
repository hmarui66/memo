# Offset-value coding in database query processing

https://www.alphaxiv.org/abs/2210.00034v3

## alphaxiv の日本語ブログ

https://www.alphaxiv.org/ja/overview/2210.00034v3

Offset-value coding(OVC) はソートされたデータを扱う際のパフォーマンス最適化技術の1つ。
キー全体を比較せず、キーの異なる最初の部分をエンコードしてソート中の比較を高速化する。


この論文では OVC をクエリ実行パイプライン全体に伝搬させるためのフレームワークを提示して、OVC をポイントで使うのではなく複雑な多重オペレータークエリで使えることを示す。

これまではオペレーターがコストのかかる処理なしに出力用の OVC を生成することはできなかったが、