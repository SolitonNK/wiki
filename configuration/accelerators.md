# Gravwellアクセラレーター

Gravwellでは、フィールド抽出を実行するために、エントリが取り込まれたときにエントリを処理できます。 抽出されたフィールドは処理され、各シャードに付随する加速ブロックに配置されます。 アクセラレータを使用すると、ストレージのオーバーヘッドを最小限に抑えながらスループットを劇的に高速化できます。 アクセラレータはウェルごとに指定されており、できるだけ目立たないように柔軟に設計されています。 加速指令に一致しないウェルにデータが入力された場合、または指定されたフィールドが欠落している場合、Gravwellは他のエントリと同じように処理します。 可能な場合、加速が行われます。

## 加速の基本

Gravwellアクセラレータは、データが比較的一意である場合に最適に動作するフィルタリング手法に基づいています。 フィールド値が非常に一般的である場合、またはほとんどすべてのエントリに存在する場合、アクセラレータ仕様に含めることはあまり意味がありません。 複数のフィールドで指定とフィルタリングを行うと、精度が向上し、クエリの速度が向上します。 加速の良い候補となるフィールドは、直接クエリされるフィールドです。 例には、プロセス名、ユーザー名、IPアドレス、モジュール名、またはその他の山積み型クエリで使用されるその他のフィールドが含まれます。

ほとんどのアクセラレーションモジュールは、ブルームエンジンを使用する場合に約1〜1.5％のストレージオーバーヘッドが発生しますが、スループットの非常に低いウェルははるかに高くなる可能性があります。 ウェルに通常1秒あたり1〜10エントリのアクセラレーションが表示される場合、5〜10％のストレージペナルティが発生する可能性があります。 Gravwellアクセラレータでは、ユーザーが指定した衝突率の調整も可能です。 ストレージを節約できる場合は、衝突率を低くすると、ストレージのオーバーヘッドを増やしながら、精度を高めてクエリを高速化できます。 精度を下げると、ストレージのペナリーは減りますが、精度が下がり、アクセラレーターの効果が減ります。 インデックスエンジンは、抽出されたフィールドの数と抽出されたデータの変動性に応じて、かなり多くのスペースを消費します。 たとえば、フルテキストインデックス作成により、アクセラレータファイルは保存されたデータファイルと同じくらいのスペースを消費する可能性があります。

アクセラレータは、エントリの直接データ部分で操作する必要があります（SRCフィールドで直接操作するsrcアクセラレータを除く）。

## 加速エンジン

Gravwellは2つの加速エンジンをサポートしています。 エンジンは、抽出された加速度データを実際に保存するシステムです。 各エンジンは、設計された取り込み速度、ディスクオーバーヘッド、検索パフォーマンス、およびデータボリュームに応じて異なる利点を提供します。 加速エンジンは、アクセラレータ名（抽出システム）から完全に独立しています。

デフォルトのエンジンは「ブルーム」エンジンです。 ブルームエンジンはブルームフィルターを使用して、特定のブロックにデータが存在するかどうかを示します。 ブルームエンジンは通常、ディスクオーバーヘッドが非常に小さく、干し草の山のようなスタイルのクエリでうまく機能します。たとえば、特定のIPが現れたログを見つけることができます。 ブルームエンジンは、フィルターされたエントリが定期的に発生するフィルターではパフォーマンスが低下します。 フルテキストアクセラレータと組み合わせると、ブルームエンジンは適切な選択肢ではありません。

「インデックス」エンジンは、すべてのクエリタイプで高速になるように設計された完全なインデックスシステムです。 通常、インデックスエンジンはブルームエンジンよりもかなり多くのディスク領域を消費しますが、非常に大量のデータや、全データのかなりの部分に影響を与える可能性のあるクエリを操作する場合は非常に高速です。 インデックスエンジンが、大量にインデックス付けされたシステムの圧縮データと同じくらいのスペースを消費することは珍しくありません。

### インデックスエンジンの最適化

「インデックス」は、キーデータの保存とクエリにファイルバックアップデータ構造を使用します。ファイルバックアップはメモリマップを使用して実行されます。 カーネルがダーティページを書き戻す頻度を減らすために、カーネルのダーティページパラメータを調整することを強くお勧めします。 これは「/ proc」インターフェースを介して行われ、「/ etc / sysctl.conf」構成ファイルを使用して永続的にすることができます。 次のスクリプトは、いくつかの効率的なパラメーターを設定し、再起動後も維持されるようにします。

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

アクセラレータはウェルごとに構成されます。 各ウェルは、加速モジュール、抽出用フィールド、衝突率、および入力ソースフィールドを含めるオプションを指定できます。 特定のソースでフィルタリングするのが一般的である場合（たとえば、特定のデバイスからのsyslogエントリのみを見る）、ソースフィールドを含めて、抽出されるフィールドに関係なくアクセラレータの精度を高める効果的な方法を提供します。

| Acceleration Parameter | Description | Example |
|----------|------|-------------|
| Accelerator-Name  | Specifies the field extraction module to use at ingest | Accelerator-Name="json" |
| Accelerator-Args  | Specifies arguments for the acceleration module, often the fields to extract | Accelerator-Args="username hostname appname" |
| Collision-Rate | Controls the accuracy for the acceleration modules using the bloom engine.  Must be between 0.1 and 0.000001. Defaults to 0.001. |
| Accelerate-On-Source | Specifies that the SRC field of each module should be included.  This allows combining a module like CEF with SRC. |
| Accelerate-Engine-Override | Specifies the engine to use for indexing.  By default the bloom engine is used. |

### サポートされている抽出モジュール

* [CSV](search/csv/csv.md)
* [Fields](search/fields/fields.md)
* [Syslog](search/syslog/syslog.md)
* [JSON](search/json/json.md)
* [CEF](search/cef/cef.md)
* [Regex](search/regex/regex.md)
* [Winlog](search/winlog/winlog.md)
* [Slice](search/slice/slice.md)

### 設定例

以下は、broのようなタブ区切りデータストリームの2番目、4番目、および5番目のフィールドを抽出する構成の例です。 この例では、各broログからソースIP、宛先IP、および宛先ポートを抽出して加速しています。 「bro」を正しく入力するすべてのエントリ（この例ではbroのみ）は、取り込み中に抽出モジュールを通過します。 データの一部が準拠していない場合。

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

各加速モジュールは、基本的なフィールド抽出のために、コンパニオン検索モジュールと同じ構文を使用します。 アクセラレータは、列挙値の名前変更、フィルタリング、または操作をサポートしません。 これらは最初のレベルのフィルターです。 対応する検索モジュールが動作し、等式フィルターを実行するたびに、加速モジュールが透過的に呼び出されます。

たとえば、JSONアクセラレーターを使用する次のウェル構成を検討してください。

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

json検索モジュールは、透過的に加速フレームワークを呼び出し、ユーザー名と「app.field1」抽出値に第1レベルのフィルターを提供します。 「app.field2」フィールドは、直接等価フィルターではないため、加速されません。 サブセットを除外、比較、またはチェックするフィルターは、高速化の対象ではありません。

### JSON

JSONアクセラレーターモジュールは、アクセラレーター名「json」を使用して指定され、フィールドを選択するためにJSONモジュールとまったく同じ構文を使用します。 フィールド抽出の詳細については、[JSON search module](/search/json/json.md)セクションを参照してください。

#### ウェル構成の例

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="json"
	Accelerator-Args="username hostname \"strange-field.with.specials\".subfield"
```

### Syslog

Syslogアクセラレータは、適合RFC5424 Syslogメッセージで動作するように設計されています。 フィールド抽出の詳細については、[Syslog search module](/search/syslog/syslog.md)セクションを参照してください。

#### ウェル構成の例

```
[Storage-Well "syslog"]
	Location=/opt/gravwell/storage/syslog
	Tags=syslog
	Accelerator-Name="syslog"
	Accelerator-Args="Hostname Appname MsgID valueA valueB"
```

### CEF

CEFアクセラレータは、CEFログメッセージで動作するように設計されており、検索モジュールと同じくらい柔軟です。 フィールド抽出の詳細については、[CEF検索モジュール](/search/cef/cef.md)セクションを参照してください。

#### ウェル構成の例

```
[Storage-Well "ceflogs"]
	Location=/opt/gravwell/storage/cef
	Tags=app1
	Accelerator-Name="cef"
	Accelerator-Args="DeviceVendor DeviceProduct Version Ext.Version"
```

### フィールド

フィールドアクセラレータは、CSV、TSV、またはその他の区切り文字であるかどうかにかかわらず、区切られた任意のデータ形式で動作できます。 Fieldsアクセラレータでは、検索モジュールと同じ方法で区切り文字を指定できます。 フィールド抽出の詳細については、[フィールド検索モジュール](#!search/fields/fields.md)セクションを参照してください。

#### ウェル構成の例

```
[Storage-Well "security"]
	Location=/opt/gravwell/storage/seclogs
	Tags=secapp
	Accelerator-Name="fields"
	Accelerator-Args="-d \",\" [1] [2] [5] [3]"
```

### CSV

CSVアクセラレータは、コンマ区切りの値データを操作するように設計されており、データから周囲の空白と二重引用符を自動的に削除します。 列抽出の詳細については、[CSV検索モジュール](#!search/csv/csv.md)セクションを参照してください。

#### ウェル構成の例

```
[Storage-Well "security"]
	Location=/opt/gravwell/storage/seclogs
	Tags=secapp
	Accelerator-Name="csv"
	Accelerator-Args="[1] [2] [5] [3]"
```

### 正規表現

正規表現アクセラレータでは、非標準のデータ形式を処理するために、取り込み時に複雑な抽出を指定できます。 正規表現は、より遅い抽出形式の1つであるため、特定のフィールドを高速化すると、クエリのパフォーマンスが大幅に向上します。

#### ウェル構成の例

```
[Storage-Well "webapp"]
	Location=/opt/gravwell/storage/webapp
	Tags=webapp
	Accelerator-Name="regex"
	Accelerator-Args="^\\S+\\s\\[(?P<app>\\w+)\\]\\s<(?P<uuid>[\\dabcdef\\-]+)>\\s(?P<src>\\S+)\\s(?P<srcport>\\d+)\\s(?P<dst>\\S+)\\s(?P<dstport>\\d+)\\s(?P<path>\\S+)\\s"
```

重要：gravwell.confファイルで正規表現を指定するときは、バックスラッシュ「\\」を忘れずにエスケープしてください。 正規表現引数「\\ w」は「\\\\ w」になります

### Winlog

winlogモジュールは、最も遅い抽出モジュールではない場合の1つです。 Windowsログスキーマと組み合わされたXMLデータの複雑さは、抽出モジュールが非常に冗長である必要があることを意味し、かなり低い抽出パフォーマンスをもたらします。 その結果、winlogモジュールで数百万または数十億のエントリを処理するのは非常に遅くなるため、Windowsデータの高速化は、最も重要なパフォーマンス最適化の1つです。 アクセラレータは、すべてのデータでwinlogモジュールを呼び出すことなく、必要な特定のログエントリを絞り込むのに役立ちます。 ただし、抽出率が遅いということは、Windowsログの取り込みが影響を受けることを意味するため、Winlogへの取り込みを高速化する際にGravwellの典型的な取り込み速度である毎秒数十万エントリを期待しないでください。

#### ウェル構成の例

```
[Storage-Well "windows"]
	Location=/opt/gravwell/storage/windows
	Tags=windows
	Accelerator-Name="winlog"
	Accelerator-Args="EventID Provider Computer TargetUserName SubjectUserName"
```

重要：winlogアクセラレーターは許容的です（「-or」フラグが暗黙指定されます）。 したがって、2つのフィールドが同じエントリに存在しない場合でも、検索のフィルタリングに使用する予定のフィールドを指定します。


### SRC

SRCアクセラレータは、SRCフィールドのみを加速する必要がある場合に使用できます。 ただし、「Accelerate-On-Source」フラグを有効にし、src検索モジュールを追加することにより、SRCアクセラレータを他のアクセラレータと組み合わせることが本質的に可能です。 フィルタリングの詳細については、[SRC検索モジュール](#!search/src/src.md)を参照してください。

#### ウェル構成の例

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="src"
```

#### SRCを組み合わせたウェル構成とクエリの例

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
