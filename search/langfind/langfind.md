## Langfind

langfindモジュールは、データを検索してテキストを探し、そのテキストを人間の言語として分類しようとする基本的な人間言語分析モジュールです。

### 検索例

次の検索では、Redditのコメントで使用されている最も一般的な言語のテーブルを、人気の高いものから低いものへ降順に作成します。

```
tag=reddit json Body | langfind -e Body | count by lang | sort by count desc | table lang count
```

### サポートされているオプション

* `-e <arg>`: “ -e”オプションは、レコード全体ではなく列挙値に作用します。たとえば、HTTPペイロードの言語分析を実行したパイプラインは次のようになります。`tag=pcap ipv4.DstPort==80 tcp.Payload | langfind -e Payload`