## IPLookup

`iplookup`モジュールは、サブネットをキーとして使用してIPアドレスのデータエンリッチメントを実行するために使用されます。これにより、完全一致を含む必要のないシンプルなルックアップテーブルを生成することができ、いくつかのリソースの管理を簡素化できる可能性があります。 `iplookup`モジュールはCIDRを含む列を指定するリソースを利用する。 CSVの各行は有効なIPv4またはIPv6のCIDR定義を含まなければならない。 CSV内の追加の列は、クエリ中にエントリに付加できる追加の値を提供するために使用することができる。 これには、ネットワークセグメント名、組織名、セキュリティ階層、あるいは緯度/経度などがあります。

`iplookup`モジュールは、[基数木](https://ja.wikipedia.org/wiki/%E5%9F%BA%E6%95%B0%E6%9C%A8)を使用して、特定のIPやCIDRを提供されたデータリストと照合します。 radixツリーは高速ですが、`O(1)`ルックアップではありません。 iplookupモジュールが最良の選択ではない場合もあります。 例えば、リソースのほとんどが `/32` CIDR を含んでいる場合は、古い [lookup](../lookup/lookup.md) リストの方が良い選択肢になるかもしれません。 また、非常に大きなルックアップのセットは、メモリとCPUのオーバーヘッドが大きくなるかもしれません。 `iplookup` システムは IPv6を完全にサポートしており、リソースに何兆個もの CIDR を指定することが完全に可能です。 iplookupモジュールのほとんどの合理的な使い方は非常に速く、maxmindデータベースを使った[geoip](./geoip/geoip.md)モジュールよりもさらに速くなります。

`iplookup`モジュールは、一度の実行で任意のnubmerのエンリッチメントを実行することができます。 例えば、CSVに `CIDR`, `network`, `building`, `lat`, `long`, `division` という列が含まれていたとします。 例としては以下のようになります。

```
iplookup -r <resource> -e IP network building lat long division
```

`iplookup` モジュールは、カスタム名を使って列挙された値に代入することもできます。 これは `as` キーワードを用いて実行されます。 この例では、`division` カラムに含まれる値をエントリに追加しますが、列挙値には `Business Unit` という名前を付けます。

```
iplookup -r <resource> -e IP network as "Business Unit"
```

### サポートされているオプション
* `-r <arg>` : "r" オプションは iplookup モジュールに、どのルックアップリソースを使用してデータを充実させるかを通知します。
* `-s` : "s" オプションは iplookup モジュールが指定された全ての操作を成功させるように要求することを指定する。
* `-v` : "v" フラグはルックアップモジュールのフローロジックを反転させ、成功したマッチは抑制され、 失敗したマッチは引き継がれることを意味する。 `v` フラグはエンリッチメントとは互換性がありません。
* `-e <arg>` : "e" フラグは、リソースリストと照合する際に使用する列挙された値を指定する。 "e"は必須フラグである。
* `-cidr <arg>` : "cidr" フラグはCIDRの仕様を含むリソースCSVのカラムを指定する。 "cidr"フラグが指定されていない場合、`iplookup`モジュールは`CIDR`という名前の列を想定する。

### iplookupリソースの設定

iplookupリソースは単なるCSVです。有効なリソースは、少なくとも1つの列に標準の[CIDR表記](https://ja.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR%E3%83%96%E3%83%AD%E3%83%83%E3%82%AF)が含まれているCSVです。iplookupはCIDR定義を含む列を "CIDR"という名前にすることを前提としていますが、それを好きなように設定することができます。 クエリを短くするために、デフォルトを使用する習慣を維持することをお勧めします。

以下に、IPv4とIPv6の両方の定義を含むリソースの例を示します。

```
CIDR,network,owner
10.0.0.0/8,storage,Bob
192.168.1.0/24,research,Tim
1.2.3.0/21,corporate,CEO
2001:4860:4860:FE08::/48,corporate ipv6,CEO
```

警告 : 2つのCIDRに重複する定義が含まれている場合は、大きい方の定義が優先されます。 つまり、`192.168.1.0/24` と `192.168.0.0/16` を定義した場合、`192.168.1.0/24` の定義は無視され、検索結果は `192.168.0.0/16` にマッチします。

### 検索例

iplookupモジュールは、ホワイトリスト(`-s` フラグ)やブラックリスト(`-v` フラグ)を作成したり、Zeekのログやnetflow、pcap、その他IPアドレスを含むあらゆるログを充実させたりするために使用できます。

#### サブネットの分類例

`iplookup`モジュールは、IPを分類したり、IPを充実させたりするためによく使われる。 便利なクエリとしては、ビジネスユニットやシステム階層ごとにトラフィックをグループ化するものがあります。 また、プライベートサブネットにジオロケーションを適用して、サブネットに基づいてマシンの正確な座標を知る簡単な手段も提供できます。 ここでは、ビジネスユニットを使用してトラフィックのアカウンティングを実行するためにネットフローを充実させるために iplookup モジュールを使用することを示します。 このケースでは、セグメント化されたネットワークがあり、各セグメントごとのトラフィック量を見ることができます。

```
tag=netflow netflow IP~PRIVATE Bytes |
iplookup -r subnets -e IP network |
stats sum(Bytes) by network |
chart sum by network
```

![](traffic.png)

#### サブネットブラックリストの例

iplookup モジュールはブラックリストチェックに使用することができ、 IPアドレスがリストに存在しないことを確認することができます。 これは、セキュアな飛び地やファイアウォールのルールを監査するのに便利かもしれません。 例えば、特定のホストやホストのセットが、いくつかのサブネットのセットと通信しないことを確認したい場合があります。 重要なインフラストラクチャの世界では、制御システムのネットワークは神聖なものであるため、家のIT側の何も制御システムのサブネットと通信しようとしていないことを検証することは、毎日のクエリ、またはアラートの候補になるかもしれません。

ここでは、172.17.0.0/16サブネットと`controlsubnets`リソースで指定された定義済みサブネットのセットとの間にフローがないことを確認するためにiplookupシステムを使用するクエリの例を示します。 ここでは、`-s` フラグを使って、指定されたリソースの宛先アドレスを持つフローのみを調べるようにしています。

```
tag=netflow
netflow Src~172.17.0.0/16 Dst DstPort |
iplookup -s -r controlsubnets -e Dst |
stats count by Src Dst DstPort |
table Src Dst DstPort count
```

リソース `controlsubnets` は以下のようなCSVになるかもしれません。

```
CIDR,network
10.0.0.0/24,secure
10.10.10.0/24,insecure
172.20.0.0/24,gravwell
192.168.0.0/16,dracnet
```