# Chronicle Mapというプロジェクトが気になる

created: 2025-05-08

[tag-tech](https://github.com/search?q=repo%3Ahmarui66%2Fmemo+tag-tech&type=code)

## 概要

JVM系でモダンなDBやストレージエンジンあるかな？と調べたら [Chronicle Map](https://github.com/OpenHFT/Chronicle-Map) というプロジェクトを見つけた。

> A super-fast, in-memory, non-blocking, key-value store, designed for low-latency, and/or multi-process applications such as trading and financial market applications

を謳っており、このあたりを考慮したJVM上で動くアプリの実装知るの面白そう、となった。

## 見つけたきっかけ

業務でサーバーサイド Kotlin を使っており、趣味はデータベース関連技術学習という感じなので、JVM 系でモダンな DB 実装があったら良いなと思ってちょっと調べた。
Chronicle Map 以外にも Java + Kotlin で実装された [Xodus](https://github.com/JetBrains/xodus) も少し気になっているが、また別の機会に見ることにする。

## Chronicle Map の特徴

以下にまとめられている。

https://github.com/OpenHFT/Chronicle-Map/blob/ea/docs/CM_Features.adoc

- Ultra low latency: Chronicle Map targets median latency of both read and write queries of less than 1 microsecond in certain tests.
- High concurrency: Write queries scale well up to the number of hardware execution threads in the server. Read queries never block each other.
- Persistence to disk - (Optional)
- Replication - (Optional, commercial functionality) - replication from one to N other servers across LAN/WAN

read & write どちらも 1 マイクロ秒未満に抑えたり、hardware thread を増やせばその分スケールするなど。

他には zero allocation, serialization/deserialization コストの除去, off-heap memory への直接アクセスを可能にする flyweight values, 辺りも面白そうなトピック。

## 一緒に調べてみても良さそうなこと

`ConcurrentMap` の実装となっており、"Drop-in ConcurrentHashMap replacement" とも書かれているので、そっちを見ても良いかも。

