# 数学モジュール

数学モジュールは、統計解析を実行するためにパイプライン上で動作します。また、情報がタイムライン上に凝縮されている場合にも重要です。例えば、温度が1秒間に10回測定されているが、ユーザーが1秒単位で表示してほしいと要求した場合、数学モジュールはそのデータを凝縮するために使用されます。

## Sum

sumモジュールはレコードの値を追加します。これはデフォルトの動作であり、直接呼び出されることはないでしょう。

MACアドレスがネットワーク上で送信したデータの合計を検索する例です。

```
tag=pcap eth.SrcMAC eth.Length | sum Length by SrcMAC | chart sum by SrcMAC
```

## Mean

平均モジュールは、レコードの平均値を返します。
車の回転数をグラフ化した検索例。

```
tag=CAN canbus ID=0x2C4 | slice uint16BE(data[0:2]) as RPM | mean RPM | chart mean
```

## Count

count モジュールはレコードのインスタンスをカウントします。レコード内のデータの演算は行いません。

linuxマシンからのsudoコマンドをカウントする検索の例です。

```
grep sudo | regex "COMMAND=(?P<command>\S+)" | count by command | chart count by command
```

これはあるMACアドレスがネットワーク上で何個のパケットを送信したかを示す検索例です(これはsumモジュールの例で示されているように各パケットのサイズとは異なります)。

```
tag=pcap eth.SrcMAC eth.Length | sum Length by SrcMAC | chart sum by SrcMAC
```

## Max

max モジュールは、最大値を返します。

艦隊全体の各車両の最大回転数の表を表示する検索例。

```
tag=CAN canbus ID=0x2C4 | slice uint16BE(data[0:2]) as RPM | max RPM by SRC | table SRC max
```

## Min

min モジュールは、見られる最小値を返します。

艦隊全体の各車両の最小回転数の表を表示する検索例。

```
tag=CAN canbus ID=0x2C4 | slice uint16BE(data[0:2]) as RPM | slice uint16BE(data[0:2]) as RPM | min RPM by SRC | table SRC min
```

## Variance

分散モジュールは、レコードの分散情報を返します。これは変化率を強調するのに便利です。

トヨタ車のスロットルデータの分散をチャート化した検索例。

```
tag=CAN canbus ID=0x2C1 | slice byte(data[6:7]) as throttle | variance throttle | chart variance
```

## Stddev

標準偏差

stddevモジュールは、レコードの標準偏差情報を返します。これは、異常なイベントを強調するのに便利です。

外れ値であるRPMシグナルの検索チャートの例。

```
tag=CAN canbus ID=0x2C4 | slice uint16BE(data[0:2]) as RPM | stddev RPM | chart stddev
```

## Unique

uniqueモジュールは、クエリデータの重複エントリを排除する。単に `unique` を指定するだけで、各エントリのデータのハッシュに基づいて重複エントリをチェックします。1つ以上の列挙された値の名前を指定すると、uniqueは列挙された値だけをフィルタリングします。この違いは次のように説明することができます。

```
tag=pcap packet tcp.DstPort | unique
```

```
tag=pcap packet tcp.DstPort | unique DstPort
```

最初のクエリでは、パケットの内容全体を見て重複したパケットをフィルタリングします。2 番目のクエリでは、抽出された DstPort の列挙値を使用して一意性をテストします。これは、例えば、TCP ポート 80 に向けられた最初のパケットは通過しますが、それ以降のポート 80 に向けられたパケットはすべてドロップされます。

複数の引数を指定した場合は、それぞれの引数のユニークな組み合わせを探します。

```
tag=pcap packet tcp.DstPort tcp.DstIP | eval DstPort < 1024 | unique DstPort DstIP | table DstIP DstPort
```

上記の検索では、ポートが1024以下であれば、IP + ポートのユニークな組み合わせをすべて出力します。これは、例えばネットワーク上のサーバを探すのに便利です。

## Entropy

entropyモジュールは、時間経過に伴うフィールド値のエントロピーを計算します。 引数なしで `entropy` を指定すると、検索範囲内のすべてのエントリのデータセクションのエントロピーを生成します。 entropyモジュールは時間的な検索モードをサポートしており、時間経過に伴うエントロピーのチャート化を可能にします。 エントロピーは、他の数学モジュールと同様に、列挙された値や複数のキーを使用したグループ化を操作することもできます。 エントロピーの出力値は0から1の間です。

TCPパケットのペイロードのエントロピーをポートに基づいて計算し、チャート化するクエリの例です。

```
tag=pcap packet tcp.Port tcp.Payload | entropy Payload by Port | chart entropy by Port
```

URLSのエントロピーをホスト別に計算し、最もエントロピーの高い値に基づいてリストをソートするクエリの例です。

```
tag=pcap packet tcp.Port==80 ipv4.IP !~ 10.0.0.0/8 tcp.Payload | grep -e Payload GET PUT HEAD POST | regex -e Payload "[A-Z]+\s(?P<url>\S+)\sHTTP\/" | entropy url by IP | table IP entropy
```

エントロピーモジュールは `-p` フラグを取ることができ、通常のようにウィンドウ全体ではなくエントリごとにエントロピーを計算するように指示します。以下は各ウィンドウのログエントリのエントロピーを計算し、エントロピーとデータを表示します。

```
tag=winlog entropy -p | table DATA entropy
```
