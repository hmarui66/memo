# Chronicle Wire のメモ

Chronicle シリーズの1つ。

https://github.com/OpenHFT/Chronicle-Wire

YAML, binary YAML, JSON, raw binary data, CSV に対応した Java 実装の serializer で、off-heap を使って高速な処理が可能とのこと。

README が充実している。

## README

### Similar Projects の欄

SBE, Message Pack, Cap'n Proto などが挙げられている。

Cap'n Proto は初めて知ったが、Protobuf にも関わっていた人の作品とのこと。

https://capnproto.org/

比較の表、観点が勉強になる(もともとは Cap'n Proto の記事で挙げられている項目)。

- Schema evolution
- Zero-copy
- Random-access reads
- Random-access writes
- Safe against malicious input
- Reflection / generic algorithms 
- Initialization order
- Unknown field retention
- Object-capability RPC system
- Schema language
- Usable as mutable state
- Padding takes space on wire?
- Unset fields take space on wire?
- Pointers take space on wire?
- Pass-by-name (Dynamic Enums)

この中で Chronicle Wire が "no" となっているのは "Schema language", "Pointers take space on wire?", "Pass-by-name (Dynamic Enums)" など。

以下項目の説明をいくつか抜き出し

#### Zero copy

- zero-copy random access to fields
- direct-copy from in-memory to the network
- translation from one wire format to another

#### Ramdom Access

in memory の field に random access 可能とのこと。

#### Pointers take space on the wire

Chronicle Wire は pointer を持ってない。

### JLBH Benchmark Performance の欄

https://github.com/OpenHFT/Chronicle-Wire/blob/develop/src/test/java/net/openhft/chronicle/wire/TriviallyCopyableJLBH.java

のテストについて説明。

Trivially Copyable Objects については [How To Get C++ Speed in Java Serialization](https://dzone.com/articles/how-to-get-c-speed-in-java-serialisation) を参照してとのこと。

Trivially Copyable Objects と BinaryWire を比較していて、前者の場合は高い percentaile でも安定して低レイテンシとなっている。

## 動かしてみる

リポジトリを clone して test を動かそうとしたら `chronicle-enterprise-snapshots` などへのアクセスが 401 となり失敗した。
default ブランチの [ea](https://github.com/OpenHFT/Chronicle-Wire/tree/ea) だとだめ(early access?)で、[chronicle-wire-2.27ea5](https://github.com/OpenHFT/Chronicle-Wire/tree/chronicle-wire-2.27ea5) を checkout して試したら上手くいった。

