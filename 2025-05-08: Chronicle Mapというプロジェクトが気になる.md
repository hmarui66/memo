# Chronicle Mapというプロジェクトが気になる

created: 2025-05-08
tag: #tech

## 概要

JVM系でモダンなDBやストレージエンジンあるかな？と調べたら [Chronicle Map](https://github.com/OpenHFT/Chronicle-Map) というプロジェクトを見つけた。

> A super-fast, in-memory, non-blocking, key-value store, designed for low-latency, and/or multi-process applications such as trading and financial market applications

を謳っており、このあたりを考慮したJVM上で動くアプリの実装知るの面白そう、となった。

## 見つけたきっかけ

業務でサーバーサイド Kotlin を使っており、趣味はデータベース関連技術学習という感じなので、JVM 系でモダンな DB 実装があったら良いなと思ってちょっと調べた。
Chronicle Map 以外にも Java + Kotlin で実装された [Xodus](https://github.com/JetBrains/xodus) も少し気になっているが、また別の機会に見ることにする。
