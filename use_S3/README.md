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