# Math Modules

mathモジュールは、パイプライン上で動作して統計分析を実行します。  また、タイムラインで情報が凝縮される場合にも重要です。  たとえば、温度が1秒あたり10回測定されているが、ユーザーが1秒ごとに表示するように要求した場合、mathモジュールを使用してそのデータを圧縮します。

## Sum

sumモジュールはレコードの値を加算します。  これはデフォルトの動作であり、直接呼び出されることはおそらくないでしょう。

MACアドレスがネットワーク上で送信したデータをまとめた検索例:

```
tag=pcap eth.SrcMAC eth.Length | sum Length by SrcMAC | chart sum by SrcMAC
```

## Mean

meanモジュールはレコードの平均値を返します。
車両のRPMチャートの検索チャートの例:

```
tag=CAN canbus ID=0x2C4 | slice uint16BE(data[0:2]) as RPM | mean RPM | chart mean
```

## Count

Countモジュールはレコードのインスタンスをカウントします。  レコード内のデータに対して算術演算を行いません。

Linuxマシンからのsudoコマンドのカウント例:

```
grep sudo | regex "COMMAND=(?P<command>\S+)" | count by command | chart count by command
```

これは、ネットワーク上でMACアドレスによって送信されたパケットの数を示す検索例です（sumモジュールの例に示されているように、各パケットのサイズとは異なります）:

```
tag=pcap eth.SrcMAC eth.Length | sum Length by SrcMAC | chart sum by SrcMAC
```

## Max

maxモジュールは最大値を返します。

フリート全体の各車両の最大RPMの表を示す検索例:

```
tag=CAN canbus ID=0x2C4 | slice uint16BE(data[0:2]) as RPM | max RPM by SRC | table SRC max
```

## Min

minモジュールは最小値を返します。

フリート全体の各車両の最小RPMの表を示す検索例:

```
tag=CAN canbus ID=0x2C4 | slice uint16BE(data[0:2]) as RPM | min RPM by SRC | table SRC min
```

## Variance

Varianceモジュールは、レコードの分散情報を返します。  これは変化率を強調するのに役立ちます。

トヨタ車のスロットルデータの分散をグラフ化した検索例:

```
tag=CAN canbus ID=0x2C1 | slice byte(data[6:7]) as throttle | variance throttle | chart variance
```

## Stddev

標準偏差

stddevモジュールはレコードの標準偏差情報を返します。  これは異常なイベントを強調するのに役立ちます。

外れ値であるRPM信号をグラフ化した検索例:

```
tag=CAN canbus ID=0x2C4 | slice uint16BE(data[0:2]) as RPM | stddev RPM | chart stddev
```

## Unique

uniqueモジュールは、クエリデータ内の重複エントリを排除します。  単にuniqueを指定すると、各エントリのデータのハッシュに基づいて重複エントリがチェックされます。   1つ以上の列挙値の名前を指定すると、uniqueが列挙値のみをフィルタリングします。  違いは次のように説明できます:

```
tag=pcap packet tcp.DstPort | unique
```

```
tag=pcap packet tcp.DstPort | unique DstPort
```

最初の問い合わせはパケットの内容全体を見ることによって重複したパケットを除外します。  パケットは通常シーケンス番号のようなものを含んでいるので、それは多くを達成しないでしょう。   2番目のクエリは、抽出されたDstPort列挙値を使用して一意性をテストします。  これはつまりTCPポート80宛ての最初のパケットは通過しますが、ポート80宛てのそれ以降のパケットはすべてドロップされます。

複数の引数を指定すると、uniqueはそれらの引数のそれぞれ固有の組み合わせを探します。

```
tag=pcap packet tcp.DstPort tcp.DstIP | eval DstPort < 1024 | unique DstPort DstIP | table DstIP DstPort
```

上記の検索では、ポートが1024未満であれば、IP +ポートのすべての固有の組み合わせが出力されます。  これは、たとえばネットワーク上のサーバーを見つけるのに便利な方法です。

## Entropy

Entropyモジュールは、フィールド値のエントロピーを経時的に計算します。  引数なしでエントロピーを指定すると、検索範囲全体のすべてのエントリーのデータセクションのエントロピーが生成されます。  エントロピーモジュールは、経時的なエントロピーの図表化を可能にする時間的検索モードをサポートする。  エントロピーは他の数学モジュールと同様に複数のキーを使用して列挙された値やグループを操作することもできます。  エントロピー出力値は0から1の間です。

ポートに基づいてTCPパケットペイロードのエントロピーを計算してチャートにするクエリの例:

```
tag=pcap packet tcp.Port tcp.Payload | entropy Payload by Port | chart entropy by Port
```

ホストごとにURLのエントロピーを計算し、最も高いエントロピー値に基づいてリストをソートするクエリの例:

```
tag=pcap packet tcp.Port==80 ipv4.IP !~ 10.0.0.0/8 tcp.Payload | grep -e Payload GET PUT HEAD POST | regex -e Payload "[A-Z]+\s(?P<url>\S+)\sHTTP\/" | entropy url by IP | table IP entropy
```
