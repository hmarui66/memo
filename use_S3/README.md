S3などオブジェクトストレージの活用についてのメモ置き場

## 事例

- WarpStream
    - https://www.warpstream.com/blog/kafka-is-dead-long-live-kafka
    - ゼロディスク構成
- Datadog
    - イベントストアを自前でオブジェクトストレージ用いて構築
    - WarpStream の人が以前携わってたとのこと
    - https://www.datadoghq.com/blog/engineering/introducing-husky/
- SlateDB
    - https://slatedb.io/docs/get-started/introduction/
    - ゼロディスク構成
    - WALもオブジェクトストレージへ
- RocksDB-cloud
    - SlateDB の FAQ に記載あり
    - https://slatedb.io/docs/get-started/faq/
- Mackerel
    - https://mackerel.io/ja/blog/entry/tech/apm-data-structure-and-algorithm
    - トレースを S3 に保存、Athena で検索

 ## その他参考資料

 - AWS の re:Invent での Amazon S3のパフォーマンス最適化とアーキテクチャ設計手法
     - https://zenn.dev/kiiwami/articles/b91d0d4558985494
