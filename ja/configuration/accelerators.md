# Gravwellアクセラレーター

Gravwellは、フィールドの抽出を実行するために、*取得*されたエントリを処理できます。 抽出されたフィールドは、各シャードに付随する加速ブロックに記録されます。 アクセラレータを使用すると、ストレージのオーバーヘッドを最小限に抑えながらスループットを劇的に高速化できます。 アクセラレータはウェルごとに指定されており、できるだけ目立たないように柔軟に設計されています。 加速指令と一致しないウェルにデータが入力された場合、または指定されたフィールドが欠落している場合、Gravwellは他のエントリと同様にデータを処理します。 可能な場合、加速が行われます。

2つの理由から、 "indexers" と"indexing"ではなく"accelerators"と"acceleration"を参照します。 まず、Gravwellには既に"indexer"と呼ばれる非常に重要なコンポーネントがあります。 第二に、ブルームフィルターを使用して**or**直接インデックスを作成することで加速を行うことができるため、"index"について記述することは必ずしも正確ではありません。

## 加速の基本

Gravwellアクセラレータは、データが比較的一意である場合に最適に機能するフィルタリング手法を使用します。 フィールド値が非常に一般的である場合、またはほとんどすべてのエントリに存在する場合、アクセラレータ仕様に含めることはあまり意味がありません。 複数のフィールドを指定してフィルタリングすると、精度が向上し、クエリの速度が向上します。 高速化の適切な候補となるフィールドは、ユーザーが直接照会するフィールドです。 例には、プロセス名、ユーザー名、IPアドレス、モジュール名、またはその他の山積み型クエリで使用されるその他のフィールドが含まれます。

使用中の抽出モジュールに関係なく、タグは常にアクセラレーションに含まれます。 クエリでインラインフィルタが指定されていない場合でも、単一のウェルに複数のタグがある場合、アクセラレーションシステムはクエリの絞り込みと高速化に役立ちます。

ほとんどのアクセラレーションモジュールは、ブルームエンジンを使用すると1〜1.5％のストレージオーバーヘッドが発生しますが、極端に低スループットのウェルではより多くのストレージが消費される場合があります。 ウェルに通常1秒あたり約1〜10エントリが表示される場合、加速により5〜10％のストレージペナルティが発生する可能性があります。 Gravwellアクセラレータでは、ユーザーが指定した衝突率の調整も可能です。 ストレージを節約できる場合、衝突率を低くすると、ストレージのオーバーヘッドが増加する一方で、精度が向上し、クエリが高速化される可能性があります。 精度を下げると、ストレージのペナリーは減りますが、精度が下がり、アクセラレーターの効果が減ります。 インデックスエンジンは、抽出されたフィールドの数と抽出されたデータの変動性に応じて、かなり多くのスペースを消費します。 たとえば、フルテキストインデックス作成により、アクセラレータファイルは保存されたデータファイルと同じくらいのスペースを消費する可能性があります。

アクセラレータは、エントリの直接データ部分で操作する必要があります（SRCフィールドで直接操作するsrcアクセラレータを除く）。

## 加速エンジン

エンジンは、抽出された加速度データを実際に保存するシステムです。 Gravwellは2つの加速エンジンをサポートしています。 各エンジンには、必要な取り込み速度、ディスクオーバーヘッド、検索パフォーマンス、およびデータボリュームに応じて異なる利点があります。 アクセラレーションエンジンは、アクセラレータ抽出プログラム自体から完全に独立しています（Accelerator-Name構成オプションで指定）。

デフォルトのエンジンは "index"エンジンです。  "index"エンジンは、すべてのクエリタイプで高速になるように設計された完全なインデックスシステムです。 通常、インデックスエンジンはブルームエンジンよりもかなり多くのディスク領域を消費しますが、非常に大量のデータや、全データのかなりの部分に影響を与える可能性のあるクエリを操作する場合は非常に高速です。 インデックスエンジンが、インデックスが大量に作成されたシステムの圧縮データと同じ容量を消費することは珍しくありません。

 "index"エンジンは、すべてのクエリタイプで高速になるように設計された完全なインデックスシステムです。 通常、インデックスエンジンはブルームエンジンよりもかなり多くのディスク領域を消費しますが、非常に大量のデータや、全データのかなりの部分に影響を与える可能性のあるクエリを操作する場合は非常に高速です。 インデックスエンジンが、大量にインデックス付けされたシステムの圧縮データと同じくらいのスペースを消費することは珍しくありません。

 ブルームエンジンはブルームフィルターを使用して、特定のブロックにデータが存在するかどうかを示します。 ブルームエンジンは通常、ディスクオーバーヘッドが非常に小さく、特定のIPが表示されたログを見つけるなど、干し草の山のようなスタイルのクエリでうまく機能します。 ブルームエンジンは、フィルターされたエントリが定期的に発生するフィルターではパフォーマンスが低下します。 また、ブルームエンジンは、フルテキストアクセラレータと組み合わせた場合、適切な選択肢ではありません。

### インデックスエンジンの最適化

"index"は、ファイルにバックアップされたデータ構造を使用して、キーデータを保存およびクエリします。 ファイルのバッキングは、メモリマップを使用して実行されます。これは、カーネルがダーティページを書き戻すのが非常に熱心な場合は、かなり悪用される可能性があります。 カーネルがダーティページを書き戻す頻度を減らすために、カーネルのダーティページパラメータを調整することを強くお勧めします。 これは"/proc"インターフェースを介して行われ、 "/etc/sysctl.conf"構成ファイルを使用して永続的にすることができます。 次のスクリプトは、いくつかの効率的なパラメーターを設定し、再起動後も維持されるようにします。

```
#!/bin/bash
user=$(whoami)
if [ "$user" != "root" ]; then
	echo "must run as root"
fi

echo 70 > /proc/sys/vm/dirty_ratio
echo 60 > /proc/sys/vm/dirty_background_ratio
echo 2000 > /proc/sys/vm/dirty_writeback_centisecs
echo 3000 > /proc/sys/vm/dirty_expire_centisecs

echo "vm.dirty_ratio = 70" >> /etc/sysctl.conf
echo "vm.dirty_background_ratio = 60" >> /etc/sysctl.conf
echo "vm.dirty_writeback_centisecs = 2000" >> /etc/sysctl.conf
echo "vm.dirty_expire_centisecs = 3000" >> /etc/sysctl.conf

```

## 加速の設定

アクセラレータはウェルごとに構成されます。 各ウェルは、加速モジュール、抽出用フィールド、衝突率、および入力ソースフィールドを含めるオプションを指定できます。 ソースフィールドを含む特定のソース（たとえば、特定のデバイスからのsyslogエントリのみを見る）で一般的にフィルターをかけると、抽出されるフィールドに関係なくアクセラレーターの精度を高める効果的な方法が提供されます。


| Acceleration Parameter | Description | Example |
|----------|------|-------------|
| Accelerator-Name  | Specifies the field extraction module to use at ingest | Accelerator-Name="json" |
| Accelerator-Args  | Specifies arguments for the acceleration module, typically the fields to extract | Accelerator-Args="username hostname appname" |
| Collision-Rate | Controls the accuracy for the acceleration modules using the bloom engine.  Must be between 0.1 and 0.000001. Defaults to 0.001. | Collision-Rate=0.01
| Accelerate-On-Source | Specifies that the SRC field of each module should be included.  This allows combining a module like CEF with SRC. | Accelerate-On-Source=true
| Accelerator-Engine-Override | Specifies the engine to use for indexing.  By default the index engine is used. | Accelerator-Engine-Override=index

### サポートされている抽出モジュール

* [CSV](#!search/csv/csv.md)
* [Fields](#!search/fields/fields.md)
* [Syslog](#!search/syslog/syslog.md)
* [JSON](#!search/json/json.md)
* [CEF](#!search/cef/cef.md)
* [Regex](#!search/regex/regex.md)
* [Winlog](#!search/winlog/winlog.md)
* [Slice](#!search/slice/slice.md)
* [Netflow](#!search/netflow/netflow.md)
* [IPFIX](#!search/ipfix/ipfix.md)
* Fulltext

### 設定例

以下は、タブで区切られたエントリの2番目、4番目、5番目のフィールド（たとえば、broログファイルの行）を抽出する構成例です。 この例では、各bro接続ログからソースIP、宛先IP、および宛先ポートを抽出および加速しています。 "bro"ウェル（この例では"bro"タグのみを含む）に入るすべてのエントリは、取り込み中に抽出モジュールを通過します。 データの一部がアクセラレーション仕様に準拠していない場合、保存されますがアクセラレーションされません。 クエリに含まれますが、多くの不適合エントリがウェルにある場合、クエリははるかに遅くなります。

```
[Storage-Well "bro"]
	Location=/opt/gravwell/storage/bro
	Tags=bro
	Accelerator-Name="fields"
	Accelerator-Args="-d \"\t\" [2] [4] [5]"
	Accelerate-On-Source=true
	Collision-Rate=0.0001
```

## 加速の基本

各加速モジュールは、基本的なフィールド抽出のために、コンパニオン検索モジュールと同じ構文を使用します。 アクセラレータは、列挙値の名前変更、フィルタリング、または操作をサポートしません。 これらは第1レベルのフィルターです。 対応する検索モジュールが動作し、等式フィルターを実行するたびに、加速モジュールが透過的に呼び出されます。

たとえば、JSONアクセラレーターを使用する次のウェル構成を考えます。

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="json"
	Accelerator-Args="username hostname app.field1 app.field2"
```

次のクエリを発行する場合：

```
tag=app json username==admin app.field1=="login event" app.field2 != "failure" | count by hostname | table hostname count
```

json検索モジュールは、透過的に加速フレームワークを呼び出し、"username"および"app.field1"の抽出値に第1レベルのフィルターを提供します。  "app.field2"フィールドは、直接等式フィルターを使用しないため、このクエリでは加速されません。 サブセットを除外、比較、またはチェックするフィルターは、高速化の対象ではありません。

## 全文
フルテキストアクセラレータは、テキストログ内の単語にインデックスを付けるように設計されており、最も柔軟なアクセラレーションオプションと見なされます。 他の検索モジュールの多くは、クエリの実行時にフルテキストアクセラレータの呼び出しをサポートしています。 ただし、フルテキストアクセラレータを使用するための主要な検索モジュールは、 `-w`フラグを指定した[grep](/search/grep/grep.md)モジュールです。 unix grepユーティリティと同様に、 `grep -w`は、指定されたフィルターがバイトのサブセットではなく、単語に対して期待されることを指定します。 `grep -w foo`で検索を実行すると、単語fooが検索され、フルテキストアクセラレータが使用されます。

フルテキストアクセラレータは最も柔軟性がありますが、最もコストがかかります。 フルテキストアクセラレータを使用すると、Gravwellの取り込みパフォーマンスが大幅に低下し、かなりのストレージスペースを消費する可能性があります。これは、フルテキストアクセラレータがすべてのエントリのほぼすべてのコンポーネントでインデックスを作成するためです。

### 全文検索
フルテキストアクセラレータは、インデックス付けされるデータの種類を絞り込み、ストレージオーバーヘッドが大幅に発生するがクエリ時にはあまり役に立たないフィールドを削除できるオプションをいくつかサポートしています。
| Argument | Description | Example | Default State |
|----------|-------------|---------|---------------|
| -ignoreTS | Engage the timegrinder system to identify and ignore timestamps in the data | `-ignoreTS` | DISABLED |
| -min | Require that extracted tokens be at least X bytes.  This can help prevent indexing on very small words that will never be searched on | `-min 3` | DISABLED |
| -ignoreUUID | Enable a filter that allows the fulltext indexer to ignore UUID values.  Some logs will generate a unique UUID for every entry which incurs significant overhead and provides very little value. | `ignoreUUID ` | DISABLED |

###　ウェル構成の例
次の適切な設定は、 `index` エンジンを使用してフルテキストアクセラレーションを実行します。 タイムスタンプ、UUIDを識別および無視し、すべてのトークンの長さが少なくとも2バイトであることを要求しています。

## JSON

JSONアクセラレータモジュールは、アクセラレータ名「json」を使用して指定され、フィールドを選択するためにJSONモジュールとまったく同じ構文を使用します。 フィールド抽出の詳細については、 [JSON検索モジュール](#!search/json/json.md)セクションを参照してください。

### ウェル構成の例

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="json"
	Accelerator-Args="username hostname \"strange-field.with.specials\".subfield"
```

## Syslog

syslogアクセラレータは、適合RFC5424 syslogメッセージで動作するように設計されています。 フィールド抽出の詳細については、 [syslog検索モジュール](#!search/syslog/syslog.md)セクションを参照してください。

### ウェル構成の例

```
[Storage-Well "syslog"]
	Location=/opt/gravwell/storage/syslog
	Tags=syslog
	Accelerator-Name="syslog"
	Accelerator-Args="Hostname Appname MsgID valueA valueB"
```

## CEF

CEFアクセラレータは、CEFログメッセージで動作するように設計されており、検索モジュールと同じくらい柔軟です。 フィールド抽出の詳細については、 [CEF検索モジュール](#!search/cef/cef.md)セクションを参照してください。

### ウェル構成の例

```
[Storage-Well "ceflogs"]
	Location=/opt/gravwell/storage/cef
	Tags=app1
	Accelerator-Name="cef"
	Accelerator-Args="DeviceVendor DeviceProduct Version Ext.Version"
```

## フィールド

フィールドアクセラレータは、CSV、TSV、またはその他の区切り記号であるかどうかにかかわらず、区切り記号で区切られたデータ形式で動作できます。 フィールドアクセラレータを使用すると、検索モジュールと同じ方法で区切り文字を指定できます。 フィールド抽出の詳細については、 [フィールド検索モジュール](#!search/fields/fields.md)セクションを参照してください。

#### ウェル構成の例
この構成は、コンマ区切りのエントリから4つのフィールドを抽出します。 `-d`フラグを使用して区切り文字を指定することに注意してください。

```
[Storage-Well "security"]
	Location=/opt/gravwell/storage/seclogs
	Tags=secapp
	Accelerator-Name="fields"
	Accelerator-Args="-d \",\" [1] [2] [5] [3]"
```

## CSV

CSVアクセラレータは、コンマ区切りの値データを操作し、周囲の空白と二重引用符をデータから自動的に削除するように設計されています。 列抽出の詳細については、[CSV検索モジュール](#!search/csv/csv.md)セクションを参照してください。

### ウェル構成の例

```
[Storage-Well "security"]
	Location=/opt/gravwell/storage/seclogs
	Tags=secapp
	Accelerator-Name="csv"
	Accelerator-Args="[1] [2] [5] [3]"
```

## 正規表現

正規表現アクセラレータでは、非標準のデータ形式を処理するために、取り込み時に複雑な抽出が可能です。 正規表現は、より遅い抽出形式の1つであるため、特定のフィールドを高速化すると、クエリのパフォーマンスが大幅に向上します。

### ウェル構成の例

```
[Storage-Well "webapp"]
	Location=/opt/gravwell/storage/webapp
	Tags=webapp
	Accelerator-Name="regex"
	Accelerator-Args="^\\S+\\s\\[(?P<app>\\w+)\\]\\s<(?P<uuid>[\\dabcdef\\-]+)>\\s(?P<src>\\S+)\\s(?P<srcport>\\d+)\\s(?P<dst>\\S+)\\s(?P<dstport>\\d+)\\s(?P<path>\\S+)\\s"
```

重要：gravwell.confファイルで正規表現を指定するときは、バックスラッシュ「\\」を忘れずにエスケープしてください。 正規表現引数「\\ w」は「\\\\ w」になります

## Winlog

winlogモジュールは、おそらく*最も*遅い検索モジュールです。 XMLデータの複雑さとWindowsログスキーマの組み合わせは、モジュールが非常に冗長である必要があることを意味し、結果としてパフォーマンスがかなり低下します。 これは、Windowsログデータの高速化が最も重要なパフォーマンス最適化である可能性があることを意味します。これは、winlogモジュールによる数百万または数十億の未加速エントリの処理が非常に遅いためです。 アクセラレータを使用すると、すべてのデータでwinlog検索モジュールを呼び出すことなく、必要な特定のログエントリを絞り込むことができます。 ただし、winlogデータを加速すると、処理が検索時間から取り込み時間に単純にシフトします。つまり、Windowsログの取り込みは、アクセラレーションが有効になっている場合は遅くなるため、Gravwellの一般的な取り込み速度は1秒あたり数十万エントリであると予想しないでください winlogによる高速化。

### ウェル構成の例

```
[Storage-Well "windows"]
	Location=/opt/gravwell/storage/windows
	Tags=windows
	Accelerator-Name="winlog"
	Accelerator-Args="EventID Provider Computer TargetUserName SubjectUserName"
```

重要：winlogアクセラレーターは許容的です（「-or」フラグが暗黙指定されます）。 したがって、2つのフィールドが同じエントリに存在しない場合でも、検索のフィルタリングに使用する予定のフィールドを指定します。

## Netflow

[netflow](#!search/netflow/netflow.md)モジュールを使用すると、netflow V5フィールドを高速化し、大量のnetflowデータのクエリを高速化できます。netflowモジュールは非常に高速であり、データは非常にコンパクトですが、非常に大量のnetflowデータがある場合は、引き続き高速化を行うことが有益です。netflowモジュールは、直接netflowフィールドのいずれかを使用できますが、ピボットヘルパーフィールドは使用できません。 つまり、`IP`ではなく`Src`または`Dst`を指定する必要があります。 `IP`および `Port`フィールドは、加速引数で指定できません。

### ウェル構成の例
この設定例では`bloom`エンジンを使用し、送信元と宛先のIP/Portのペアとプロトコルで加速しています。

```
[Storage-Well "netflow"]
	Location=/opt/gravwell/storage/netflow
	Tags=netflow
	Accelerator-Name="netflow"
	Accelerator-Args="Src Dst SrcPort DstPort Protocol"
	Accelerator-Engine-Override="bloom"
```
## IPFIX
[ipfix](#!search/ipfix/ipfix.md)モジュールは、IPFIX形式のレコードのクエリを高速化できます。 このモジュールは、  'normal'IPFIXフィールドで加速できますが、ピボットヘルパーフィールドでは加速できません。 つまり、`port`ではなく`sourceTransportPort`または`destinationTransportPort`、または`ip`ではなく`src`/`dst`を指定する必要があります。

### ウェル構成の例
この設定例では、`index`エンジンを使用して、ソース/宛先IP/ポートペアとフローのIPプロトコルを高速化します。これは、netflowセクションに示されている例に相当します。

```
[Storage-Well "ipfix"]
	Location=/opt/gravwell/storage/ipfix
	Tags=ipfix
	Accelerator-Name="ipfix"
	Accelerator-Args="src dst sourceTransportPort destinationTransportPort protocolIdentifier"
	Accelerator-Engine-Override=index
```

## SRC

SRCアクセラレータは、エントリのソースフィールドのみを高速化する必要がある場合に使用できます。 ただし、「Accelerate-On-Source」フラグを有効にし、クエリでSRC検索モジュールを使用することにより、SRCアクセラレータを他のアクセラレータと組み合わせることが基本的に可能です。 フィルタリングの詳細については、[SRC検索モジュール](#!search/src/src.md)を参照してください。

### ウェル構成の例

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="src"
```

### SRCを組み合わせたウェル構成とクエリの例

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="fields"
	Accelerator-Args="-d \",\" [1] [2] [5] [3]"
	Accelerate-On-Source=true
```

次のクエリは、フィールドアクセラレータとSRCアクセラレータの両方を呼び出して、特定のソースからの特定のログタイプを指定します。

```
tag=app src dead::beef | fields -d "," [1]=="security" [2]="process" [5]="domain" [3] as processname | count by processname | table processname count
```
## 加速性能とベンチマーク

アクセラレーションの利点と欠点を理解するには、ストレージの使用、取り込みのパフォーマンス、クエリのパフォーマンスへの影響を確認するのが最善です。 [github](https://github.com/kiritbasu/Fake-Apache-Log-Generator)で利用可能なジェネレーターを使用して生成されたapache複合アクセスログを使用します。 私たちのデータセットは、約24時間に分散した1,000万件のappache結合アクセスログです。 合計データは2.1GBです。 4つの異なる構成の4つのウェルを定義します。 返されるバイト数など、インデックス付けするのにあまり意味のないパラメーターが多数あるため、このデータのインデックス付けにはかなり単純な方法を使用します。

| Well | Extractor | Engine | Description |
|------|-----------|--------|-------------|
| raw  | None | None | A completely raw well with no acceleration at all |
| fulltext | fulltext | index | A fulltext accelerated well that uses the index engine and will perform fulltext acceleration on every word |
| regexindex | regex | index | A well accelerated with the regex extractor and using the index engine.  Each parameter is extracted and indexed |
| regexbloom | regex | bloom | A well with the same extractor as the regexindex well but with bloom engine.  Each parameter is extracted and added to the bloom filter |

ウェル構成は次のとおりです。

```
[Storage-Well "raw"]
	Location=/opt/gravwell/storage/raw
	Tags=raw
	Enable-Transparent-Compression=true

[Storage-Well "fulltext"]
	Location=/opt/gravwell/storage/fulltext
	Tags=fulltext
	Enable-Transparent-Compression=true
	Accelerator-Name=fulltext
	Accelerator-Args="-ignoreTS -min 2"

[Storage-Well "regexindex"]
	Location=/opt/gravwell/storage/regexindex
	Tags=regexindex
	Enable-Transparent-Compression=true
	Accelerator-Name=regex
	Accelerator-Engine-Override=index
	Accelerator-Args="^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"

[Storage-Well "regexbloom"]
	Location=/opt/gravwell/storage/regexbloom
	Tags=regexbloom
	Enable-Transparent-Compression=true
	Accelerator-Name=regex
	Accelerator-Engine-Override=bloom
	Accelerator-Args="^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
```

### 試験機

クエリ、取り込み、およびストレージのパフォーマンス特性は、各データセットおよびハードウェアプラットフォームによって異なりますが、このテストでは次のハードウェアを使用しています。

| Component | Description |
|-----------|-------------|
| CPU       |  AMD Ryzen 1700 |
| Memory    | 16GB DDR4-2400 |
| Disk      | Samsung 960 EVO NVME |
| Filesystem | BTRFS with zstd transparent compression |

これらのテストはGravwellバージョン `3.1.5`を使用して実施されました。

### 取り込みパフォーマンス

取り込みには、singleFile ingesterを使用します。 singleFile ingesterは、改行で区切られた単一のファイルを取り込んで、タイムスタンプを取得するように設計されています。 ingesterはタイムスタンプを取得しているため、CPUリソースが必要です。 singleFile ingesterは、 [github page](https://github.com/gravwell/ingesters/)で入手できます。 singleFile ingesterの正確な呼び出しは次のとおりです。

```
./singleFile -i apache_fake_access_10M.log -clear-conns 172.17.0.3 -block-size 1024 -timeout=10 -tag-name=fulltext
```

|  Well      | Entries Per Second | Data Rate  |
|------------|--------------------|------------|
| raw        | 313.54 KE/s        | 65.94 MB/s |
| regexbloom | 112.91 KE/s        | 23.75 MB/s |
| regexindex | 57.58  KE/s        | 12.11 MB/s |
| fulltext   | 26.37  KE/s        |  5.55 MB/s |

取り込みのパフォーマンスから、フルテキストアクセラレーションシステムが取り込みのパフォーマンスを劇的に低下させることがわかります。 5.55MB / sは取り込みパフォーマンスが悪いように見えますが、これはまだ約480GBのデータと1日あたり23億エントリであることに言及する価値があります。

### ストレージ使用量

取り込みパフォーマンスといくつかの追加メモリ要件以外では、アクセラレーションを有効にする主なペナルティは使用量です。各抽出方法のインデックスエンジンは50％以上のストレージを消費し、ブルームエンジンはさらに4％を消費していることがわかります。ストレージの使用量は、消費されるデータに大きく依存しますが、平均すると、インデックスシステムはかなり多くのストレージを消費します。

|  Well      | Used Storage | Diff From Raw |
|------------|--------------|---------------|
| raw        | 2.49GB       | 0%            |
| fulltext   | 3.83GB       | 53%           |
| regexindex | 3.76GB       | 51%           |
| regexbloom | 2.60GB       | 4%            |

### クエリパフォーマンス

クエリのパフォーマンスの違いを示すために、スパースとデンスに分類できる2つのクエリを実行します。 スパースクエリは、データセット内の特定のIPを探し、ほんの一握りのエントリを返します。高密度クエリは、データセット内でかなり一般的な特定のURLを探します。クエリを簡素化するために、アクセラレーションシステムに一致するregexindexおよびregexbloomタグ用のaxモジュールをインストールします。密なクエリはデータセット内のエントリの約12％を取得しますが、疎なクエリは約0.01％を取得します

疎クエリと密クエリは次のとおりです。

```
ax ip=="106.218.21.57"
ax url=="/list" method | count by method | chart count by method
```

各クエリを実行する前に、次のコマンドをルートとして実行してシステムキャッシュを削除します。

```
echo 3 > /proc/sys/vm/drop_caches
```

|  Well      | Query Type | Query Time | Processed Entries Percentage | Speedup |
|------------|------------|------------|------------------------------|---------|
| raw        | sparse     | 71.5s      |  100%                        | 0X      |
| regexbloom | sparse     | 397ms      |  0.00389%                    | 180X    |
| regexindex | sparse     | 190ms      |  0.000001%                   | 386X    |
| fulltext   | sparse     | 195ms      |  0.000001%                   | 376X    |
| raw        | dense      | 73.5s      |  100%                        | 0X      |
| regexbloom | dense      | 71.5s      |  100%                        | 1.02X   |
| regexindex | dense      | 14.2s      |  13%                         | 5.17X   |
| fulltext   | dense      | 24.6s      |  30%                         | 2.98X   |

注：正規表現検索モジュール/ autoextractorはフルテキストアクセラレータと完全に互換性がないため、アクセラレータを使用するにはクエリをわずかに変更する必要があります。 それらは "106.218.21.57"```と ```grep -w list | ax url=="/list" method | count by method | chart count by method```によるカウント| メソッドごとのチャートカウント

#### 全文

上記のベンチマークにより、フルテキストアクセラレータには大量のインジェストとストレージのペナルティがあり、クエリの例ではこれらの費用を正当化できないようです。データがタブ区切り、csv、jsonなどのトークンベースであり、すべてのトークンが完全に秘密（単一の単語、数字、IP、値など）である場合、フルテキストアクセラレータはあまり意味がありません。ただし、データに複雑なコンポーネントがある場合、フルテキストアクセラレータは他のアクセラレータではできないことを実行できます。Apacheの結合されたアクセスログを使用しているので、フルテキストアクセラレータを実際に使用できるクエリを見てみましょう。

URLのサブコンポーネントを調べ、PowerPC Macintoshコンピューターを使用して `/apps` サブディレクトリを参照しているユーザーのチャートを取得します。 上記の例の正規表現は、Apacheログ内のすべてのフィールドにインデックスを付けます。 これらのフィールドの一部をドリルダウンして使用することはできません。

フルテキストインデクサーとその他の両方のクエリを最適化して、公平になるようにしますが、両方のクエリはどちらのデータセットでも機能します。

フルテキストアクセラレータの最適化されたクエリ：
```
grep -s -w apps Macintosh PPC | ax url~"/apps" useragent~"Macintosh; U; PPC" | count | chart count
```

非フルテキスト用に最適化されたクエリ：
```
ax url~"/apps" useragent~"Macintosh; U; PPC" | count | chart count
```

結果は、フルテキストがストレージと取り込みのペナルティに値することが多い理由を示しています。

|  Well      | Query Time | Speedup |
|------------|------------|---------|
| raw        | 71.7s      | 0X      |
| regexbloom | 72.6s      | ~0X     |
| regexindex | 72.6       | ~0X     |
| fulltext   | 5.73s      | 12.49X  |

#### AXモジュールのクエリ
4つのタグすべてのAX定義ファイルは次のとおりです。詳細については、 [AX]（）のドキュメントを参照してください。

```
[[extraction]]
  tag = 'regexindex'
  module = 'regex'
  params = "^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
  name = 'apacheindex'
  desc = 'apache index'

[[extraction]]
  tag = 'regexbloom'
  module = 'regex'
  params = "^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
  name = 'apachebloom'
  desc = 'apache bloom'

[[extraction]]
  tag = 'fulltext'
  module = 'regex'
  params = "^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
  name = 'apachefulltext'
  desc = 'apache fulltext'

[[extraction]]
  tag = 'raw'
  module = 'regex'
  params = "^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
  name = 'apacheraw'
  desc = 'apache raw'
```
