# インジェストプリプロセッサ

インジェストされたデータをインデクサーに送信する前に、データ加工する必要がある場合があります。例えば、syslog で送られてきた JSON データを取得していて、 syslog ヘッダを削除したいという場合などがあるでしょう。あるいは、Apache Kafka ストリームから得たデータは gzip 圧縮されたままだったりする場合もあるでしょう。またあるいは、エントリの内容に基づいてエントリを異なるタグにルーティングできるようにしたい場合もあるでしょう。インジェストプリプロセッサは、エントリがインデクサーに送られる前に一つ以上の処理ステップを挿入することでこれのデータ加工を可能にします。

## プリプロセッサのデータの流れ

インジェスターは、何らかのソース(ファイル、ネットワーク接続、Amazon Kinesis ストリームなど)から生データを読み込み、入ってきたデータストリームを個々のエントリに分割します。これらのエントリがGravwellインデクサーに送られる前に、下の図に示すように、任意の数のプリプロセッサを通過させることができます。

![](arch.png)

各プリプロセッサには、エントリを修正するタイミングがあります。そのため、プリプロセッサは常に順不同で適用されます。つまり、例えばエントリーのデータを解凍し、解凍されたデータに基づいてエントリータグを修正することができます。

## プリプロセッサの設定

プリプロセッサは、標準パッケージされているインジェスター全てに対してサポートされています。しかし、ワンオフで作られたインジェスターや非サポートのインジェスターに対しては、プリプロセッサもサポートされないことになります。

プリプロセッサはインジェスターの設定ファイルで `preprocessor` 設定節を使って設定されます。 各プリプロセッサ設定節では、`Type` 設定パラメーターによって使用するプリプロセッサ・モジュールを宣言し、次いでプリプロセッサ固有の設定パラメーターの記述が続けられます。以下ののSimple Relayインジェスターの例で考えてみましょう:

```
[Global]
Ingester-UUID="e985bc57-8da7-4bd9-aaeb-cc8c7d489b42"
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify=true
Cleartext-Backend-target=127.0.0.1:4023 #example of adding a cleartext connection
Log-Level=INFO

[Listener "default"]
	Bind-String="0.0.0.0:7777" #we are binding to all interfaces, with TCP implied
	Tag-Name=default
	Preprocessor=timestamp

[Listener "syslog"]
	Bind-String="0.0.0.0:601" # TCP syslog
	Tag-Name=syslog

[preprocessor "timestamp"]
	Type = regextimestamp
	Regex ="(?P<badtimestamp>.+) MSG (?P<goodtimestamp>.+) END"
	TS-Match-Name=goodtimestamp
	Timezone-Override=US/Pacific
```

この設定では、"default"と "syslog"という2つのデータ受取り口(Simple Relayインジェスターでは "Listeners"（リスナー）と呼んでいます)を定義しています。また、"timestamp" という名前のプリプロセッサも定義しています。"default" リスナーには `Preprocessor=timestamp` というオプションが含まれていることに注意してください。これはポート7777のリスナーから入ってくるエントリを "timestamp"プリプロセッサに送ることを指定しています。syslog "リスナーには `Preprocessor` オプションが設定されていないので、ポート601に入ってくるエントリはどのプリプロセッサも通過しません。

## gzip プリプロセッサ

gzip プリプロセッサは、GNU 'gzip' アルゴリズムで圧縮されたエントリを解凍することができます。

GZIP プリプロセッサの Type は `gzip`です。

### サポートされているオプション

* `Passthrough-Non-Gzip` (ブーリアン型、設定任意): trueに設定されている場合、プリプロセッサは内容が gzipで圧縮されていないエントリを通過させます。デフォルトでは、gzipで圧縮されていないエントリはすべて削除されます。

### 一般的な使用例

多くのクラウドデータバスプロバイダは、エントリやパッケージを圧縮された形で供給してきます。 このプリプロセッサは、コストが発生しがちなクラウドのラムダ関数を介したルーティングではなく、インジェスター内のデータストリームを解凍することができます。


### 例: 圧縮されたエントリの解凍

設定例:

```
[Preprocessor "gz"]
	Type=gzip
	Passthrough-Non-Gzip=true
```

## JSON抽出プリプロセッサ

JSON 抽出プリプロセッサは、エントリの内容を JSON として解析し、JSON から 1 つ以上のフィールドを抽出し、エントリの内容をそれらのフィールドに置き換えることができます。これは、過度に複雑なメッセージを、関心のある情報だけを含むより簡潔なエントリに単純化するのに便利な方法です。

単一のフィールドの抽出のみが指定された場合、結果はそのフィールドの内容のみを含みます。 複数のフィールドが指定された場合、プリプロセッサはそれらのフィールドを含む有効なJSONを生成します。

JSON抽出プリプロセッサの型は `jsonextract` です。

### サポートされているオプション

* `Extractions` (文字列型、設定必須): JSONから抽出するフィールド(カンマ区切り)を指定します。入力されるのが `{"foo": "a", "bar":2, "baz":{"frog". "womble"}}` というフォーマットの場合、`Extractions=foo`, `Extractions=foo,bar`, `Extractions=baz.frog,foo` などと指定することができます。
* `Force-JSON-Object` (ブーリアン型、設定任意): このオプション指定がなされてないデフォルトでは、単一フィールドの抽出が指定された場合、プリプロセッサはエントリの内容をその拡張子の内容で置き換えます。つまり、`{"foo":"a", "bar":2, "baz":{"frog": "womble"}}` を含むエントリに、`Extraction=foo` を適用した場合、単に `a` を含むだけのエントリに変更してしまいます。このオプションが設定されている場合、プリプロセッサは常に完全なJSON構造体、つまり、例の場合は`{"foo":"a"}`を出力します。
* `Passthrough-Misses` (ブーリアン型、設定任意): trueに設定すると、プリプロセッサは要求されたフィールドを抽出できなかったエントリを下流に渡します。デフォルトでは、これらのエントリは削除されます。
* `Strict-Extraction` (ブーリアン型、設定任意): デフォルトでは、プリプロセッサは少なくとも一つの抽出が成功した場合には、そのエントリを下流に渡します。このパラメーターがtrueに設定されている場合、すべての抽出が成功することを要求します。

### 一般的な使用例

多くのデータソースは、実際のログストリームの内容そのものではない、通信や保存ストレージに関する付加的なメタデータを供給してくることがあります。 jsonextract プリプロセッサは、ストレージコストを削減するためにフィールドの種類を絞り込むことができます。

### 例: JSONデータレコードの凝縮

```
[Preprocessor "json"]
	Type=jsonextract
	Extractions=IP,Alert.ID,Message
	Passthrough-Misses=true
```

## JSON配列分割プリプロセッサ

このプリプロセッサは、JSON オブジェクト内の配列を個々のエントリに分割することができます。たとえば、名前の配列を含むエントリが与えられた場合、プリプロセッサはそれぞれの名前に対して一つのエントリを出力します。 従って、次のようなJSONについては:

```
{"IP": "10.10.4.2", "Users": ["bob", "alice"]}
```

結果として２つのエントリができ、１つは"bob"を含んでおり、もう１つは"alice"を含んでいるものになります。

JSON配列分割プリプロセッサの型は `jsonarraysplit` です。

### サポートされているオプション

* `Extraction`  (文字列型、設定必須): 分割する構造体を含むJSONフィールドを指定します。つまり、 `Extraction=Users`、 `Extraction=foo.bar`という具合です。
* `Passthrough-Misses`  (ブーリアン型、設定任意): trueに設定すると、プリプロセッサは、要求されたフィールドを抽出できなかったエントリを下流に渡します。デフォルトでは、これらのエントリは削除されます。
* `Force-JSON-Object`  (ブーリアン型、設定任意): デフォルトでは、プリプロセッサはそれぞれがリストの中の1つの項目を含むエントリを出力し、それ以外は何も出力しません。つまり、`{"foo": ["a", "b"]}` から`foo`を抽出すると、それぞれ "a" と "b" を含む二つのエントリが生成されます。一方、同じエントリに対してこのオプションが設定されていると、`{"foo". "a"}` と `{"foo". "b"}`という二つのエントリが生成されます。
* `Additional-Fields`  (文字列型、設定任意):分割対象の配列の外側にあるフィールドについての結果への追加するフィールドのリストで、カンマで区切りで記述され、それらは各エントリに抽出結果に追加されます。`Additional-Fields="foo,bar, foo.bar.baz"`のように記述されます。

### 一般的な使用例

多くのデータプロバイダは、複数のイベントを 1 つのエントリにまとめることがありますが、これはイベントのアトミックな性質を低下させ、分析の複雑さを増す可能性があります。 複数のイベントを含む単一のメッセージを個々のエントリに分割することで、イベントの処理を簡単にすることができます。

### 例: 単一レコード内の複数のメッセージを分割する
```
[preprocessor "json"]
	Type=jsonarraysplit
	Extraction=Alerts
	Force-JSON-Object=true
```


## JSONフィールドフィルタリングプリプロセッサ

このプリプロセッサは、エントリーデータをJSONオブジェクトとして解析し、指定されたフィールドを抽出し、許容値のリストと比較します。受け入れ可能な値のリストは、ディスク上のファイル内に1行に1つの値を記述して設定します。

フィールドがリストにマッチするエントリのみを *pass* する（ホワイトリスト方式）か、リストにマッチするエントリを *drop* する（ブラックリスト方式）ように設定することができます。複数のフィールドに対してフィルタリングするように設定することができ、その場合には*すべての*フィールドが一致しなければならない (論理 AND) か、または *少なくとも1つの*フィールドが一致しなければならない (論理 OR) のいずれかの指定が必要になります。

このプリプロセッサは、一般的なデータを低速なネットワークリンクを介して送信する前に、一般的なデータの流量を絞り込むのに特に便利です。

JSONフィールドフィルタリングプリプロセッサの型は `jsonfilter` です。

### サポートされているオプション

* `Field-Filter`(文字列型、設定必須): このパラメーターには、対象となるJSONフィールドの名前と、照合する値を含むファイルへのパスの2つのことを指定します。例えば、"ComputerName "という名前のフィールドを抽出し、`/opt/gravwell/etc/computernames.txt`の値と比較するには、`Field-Filter=ComputerName,/opt/gravwell/etc/computernames.txt`と設定することができます。複数のフィールドに対してフィルタリングする場合には、`Field-Filter`オプションを複数回指定することがでます。
* `Match-Logic`(文字列型、設定任意): このパラメーターは、複数のフィールドに対してフィルタリングを行う際に使用する論理演算子を指定します。"and"に設定すると、指定されたリストに対して*すべての*フィールドが一致した場合にのみ、エントリは一致したものとみなされます。"or"に設定すると、*いずれか１つ以上の*フィールドが一致した場合、エントリは一致したものとみなされます。
* `Match-Action`(文字列型、設定任意): 指定されたリストに一致するフィールドがあるエントリをどのように処理するかのオプションを"pass"か"drop"で指定します。省略した場合のデフォルトは"pass"です。"pass" に設定した場合、一致するエントリはインデクサーに渡すことが許可されます (ホワイトリスト化)。"drop" に設定すると、一致するエントリは削除されます (ブラックリスト化)。

`Match-Logic` パラメーターは、複数の `Field-Filter` が指定されている場合にのみ必要です。

注意: フィールドが設定で指定されていてもエントリに存在しない場合、プリプロセッサはそのエントリを *フィールドは存在するが何もマッチしなかったかのように扱います*。したがって、フィールドがホワイトリストにマッチするエントリのみを渡すようにプリプロセッサを設定している場合、フィールドのひとつが欠けているエントリは削除されます。

### 一般的な使用例

jsonフィールドフィルタリングプリプロセッサは、エントリ内のフィールドに基づいてエントリを選択することができます。 これにより、データフロー上にブラックリストやホワイトリストを構築して、データがストレージに到達するかしないかを確実にすることができます。

### 例: シンプルなホワイトリストSimple Whitelisting

エンドポイント監視ソリューションで、企業全体で発生しているイベントの詳細を1秒間に何千ものイベントを送信しているとします。イベントの量が多いため、特定の深刻度のイベントのみをインデックス化することにします。イベントには次のように都合よく重大度フィールドが含まれているものとします:

```
{ "EventID": 1337, "Severity": 8, "System": "email-server-01.example.org", [...] }
```

Severity フィールドは 0 から 9 まであり、Severity が 6 以上のイベントのみを渡すことにします。そこで、インジェスター設定ファイルに以下の記述を追加します:

```
[preprocessor "severity"]
	Type=jsonfilter
	Match-Action=pass
	Field-Filter=Severity,/opt/gravwell/etc/severity-list.txt
```

この設定に従って、データ入力されるところに `Preprocessor=severity` を設定します。例えば、Simple Relayを使用している場合:

```
[Listener "endpoint_monitoring"]
	Bind-String="0.0.0.0:7700
	Tag-Name=endpoint
	Preprocessor=severity
```

最後に、1行に1つずつ受信可能なSeverity値が記述された`/opt/gravwell/etc/severity-list.txt`を作成します:

```
6
7
8
9
```

インジェスターを再起動すると、インジェスターは各エントリから `Severity` フィールドを抽出し、`Severity`フィールドの値を`/opt/gravwell/etc/severity-list.txt`ファイルにリストされている値と比較し、`Severity`フィールドの値がファイルのいずれか一行の値と一致した場合には、そのエントリはインデクサーに送られるようになります。一致しないエントリは削除されます。

### 例: ブラックリスト

上述の例のエンドポイント監視システムにおいて、特定のシステムから不当に高く深刻度を付されたイベント情報を大量に生成されていることに気づくようなことがあるかもしれません。例えば、EventID` フィールドが 219、220、または 1338 に設定され、`System` フィールドが "webserver-prod.example.org" および "webserver-dev.example.org" に設定されているイベントは、常に偽陽性である（実際にはパスすべきでないデータをパスさせてしまっている）と判断できたとします。これらのエントリがインデクサーに送られる前に、これらのエントリを取り除くために別のプリプロセッサを定義することができます:

```
[preprocessor "falsepositives"]
	Type=jsonfilter
	Match-Action=drop
	Match-Logic=and
	Field-Filter=EventID,/opt/gravwell/etc/eventID-blacklist.txt
	Field-Filter=System,/opt/gravwell/etc/system-blacklist.txt
```

以下に示すように、新しいプリプロセッサを、データ入力設定の既存のプリプロセッサの*後に*追加すると、インジェスターはフィルタを並んでいる順番に適用します:

```
[Listener "endpoint_monitoring"]
	Bind-String="0.0.0.0:7700
	Tag-Name=endpoint
	Preprocessor=severity
	Preprocessor=falsepositives
```

そして、`/opt/gravwell/etc/eventID-blacklist.txt`を次のように作成します:

```
219
220
1338
```

`/opt/gravwell/etc/system-blacklist.txt`も次のように作成します:

```
webserver-prod.example.org
webserver-dev.example.org
```

この新しいプリプロセッサは、最初のフィルタを通過したすべてのエントリから `EventID` と `System` フィールドを抽出します。そして、それらをファイル内の値と比較します。`Match-Logic=and` を設定しているので、*両方の*ファイル内でフィールドの値が見つかった場合、エントリはマッチしたとみなされます。`Match-Action=drop` を設定しているので、両方のフィールドでマッチしたエントリはすべて削除されます。したがって、 `EventID=220` かつ `System=webserver-dev.example.org` という内容のエントリは削除されますが、 `EventID=220` だけれども `System=email-server-01.example.org` という内容のエントリは削除されません。

## Regex Routerプリプロセッサ

regex routerプリプロセッサは、エントリの内容に基づいてエントリに異なるタグをルーティングするための柔軟なツールです。設定では、[名前付きキャプチャグループ](https://www.regular-expressions.info/named.html)を含む正規表現を記述し、その内容をユーザー定義のルーティングルールと照らし合わせてテストします。

Regex Routerのプリプロセッサの型は `regexrouter` です。

### サポートされているオプション

* `Regex` (文字型、設定必須):このパラメーターは入力エントリに適用される正規表現を指定します。`Route-Extraction` パラメーターで使用される `(?P<app>.+)` のような [名前付きキャプチャグループ](https://www.regular-expressions.info/named.html) を少なくとも一つは含まなければなりません。
* `Route-Extraction` (文字型、設定必須): このパラメーターは、`Regex`パラメーターから、ルーティングについて比較に用いられる文字列を含む名前付きキャプチャグループの名前を指定します。
* `Route` (文字型、設定必須): 少なくとも一つ以上 `Route` 指定が必要です。`Route` 設定は、`Route=sshd:sshlogtag`のようにコロンで区切られた 2 つの文字列でなされます。最初の文字列 ('sshd') は正規表現で抽出された値と照合され、マッチしたエントリについて2番目の文字列で指定された名前でタグルーティングされます。2 番目の文字列が空白のままの場合、1 番目の文字列にマッチするエントリは*削除されます*。
* `Drop-Misses` (ブーリアン型、設定任意): デフォルトでは、正規表現にマッチしないエントリは変更されずに通過します。`Drop-Misses` を true に設定すると、インジェスターは 1) 正規表現にマッチしないエントリ、または 2) 正規表現にマッチするが指定されたルートのいずれにもマッチしないエントリをすべて削除するようになります。

### 例: アプリのフィールド値に基づいたタグへのルーティング

このプリプロセッサの使用法を説明として、多くのシステムがSimple Relayインジェスターに syslog エントリを送信している状況をとりあげてみます。sshd ログを `sshlog` という名前の別のタグに分離したいと思います。受信する sshd ログは次のような古いスタイルの BSD syslog フォーマット (RFC3164) です:

```
<29>1 Nov 26 11:26:36 localhost sshd[11358]: Failed password for invalid user administrator from 202.198.122.184 port 49828 ssh2
```

正規表現を色々試してみると、RFC3164 のログからアプリケーション名 (sshd) を "app" という名前のキャプチャグループに抽出するための適切な正規表現が以下のようになることがわかりました:

```
^(<\d+>)?\d?\s?\S+ \d+ \S+ \S+ (?P<app>[^\s\[]+)(\[\d+\])?:
```

この正規表現をプリプロセッサの設定に適用すると、以下に示すようになります:

```
[Listener "syslog"]
        Bind-String="0.0.0.0:2601" #we are binding to all interfaces, with TCP implied
        Tag-Name=syslog
        Preprocessor=bsdrouter

[preprocessor "bsdrouter"]
        Type = regexrouter
        Drop-Misses=false
	# Regex: <pri>version Month Day Time Host App[pid]
	Regex="^(<\\d+>)?\\d?\\s?\\S+ \\d+ \\S+ \\S+ (?P<app>[^\\s\\[]+)(\\[\\d+\\])?:"
        Route-Extraction=app
        Route=sshd:sshlog
```

プリプロセッサは正規表現を定義してから、`Route-Extraction` パラメーターでキャプチャグループ "app" を呼び出していることに注意してください。次に、`Route=ssh:sshlog` 設定によって、アプリケーション名が "sshd" と一致するエントリは、 "sshlog"というタグにルーティングされるように指定されています。必要に応じてこれに追加の `Route` パラメーター、例えば`Route=apache:apachelog`を設定することができます。

上記の設定では、sshd からのログは "sshlog" タグに送られ、他のすべてのログはそのまま "sshlog" タグに送られます、同一のフォーマットで記述された syslog エントリから他のアプリケーションを抽出するには`Route` 設定を追加するだけでできますが、次のような RFC 5424 フォーマットのログが混在していた場合について考えてみましょう。

```
<101>1 2019-11-26T13:24:56.632535-07:00 web01.example.org webservice 21581 - [useragent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3191.0 Safari/537.36"] GET /
```

上でとりあげた正規表現では適切にアプリケーション名("webservice")を抽出できませんが、*2つめの*プリプロセッサを定義して、最初のプリプロセッサの後にプリプロセッサを連鎖的を繋げることができます:

```
[Listener "syslog"]
        Bind-String="0.0.0.0:2601" #we are binding to all interfaces, with TCP implied
        Tag-Name=syslog
        Preprocessor=bsdrouter
	Preprocessor=rfc5424router

[preprocessor "bsdrouter"]
        Type = regexrouter
        Drop-Misses=false
	# Regex: <pri>version Month Day Time Host App[pid]
	Regex="^(<\\d+>)?\\d?\\s?\\S+ \\d+ \\S+ \\S+ (?P<app>[^\\s\\[]+)(\\[\\d+\\])?:"
        Route-Extraction=app
        Route=sshd:sshlog

[preprocessor "rfc5424router"]
	Type=regexrouter
	Drop-Misses=false
	# Regex: <pri>version Date Host App
	Regex="^<\\d+>\\d? \\S+ \\S+ (?P<app>\\S+)"
	Route-Extraction=app
	Route=webservice:weblog
	Route=apache:weblog
	Route=postfix:		# drop
```

この新しいプリプロセッサの定義では、"webservice" と "apache" という名前のアプリケーションのルーティングを定義し、両方とも "weblog" タグに送信していることに注意してください。また、"postfix" アプリケーションからのログを *削除(drop)* するように指定していることにも注意してください（"postfix" アプリケーションのログを削除 することにしているのは、おそらく、このログはすでに別のソースからインジェストされているからでしょう）。

## Regex タイムスタンプ抽出プリプロセッサ

インジェスターは通常、有効なタイムスタンプと思われる最初のものを探し、それを解析することで、エントリからタイムスタンプを抽出しようとします。タイムスタンプを解析するための追加のインジェスター設定ルール(検索するための特定のタイムスタンプ形式を指定するなど)と組み合わせれば、通常は適切なタイムスタンプを適切に抽出するのに十分ですが、データソースによっては、これらの簡単な方法では上手くいかない場合もあります。可能性の１つとして、ネットワークデバイスがsyslogでラップされたCSV形式のイベントログを送るという状況を考えてみましょう--これはGravwellで実際体験した状況です!

Regexタイムスタンプ抽出プリプロセッサの型は `regextimestamp` です。

### サポートされているオプション

* `Regex` (文字型、設定必須): このパラメーターは、入力されたエントリに適用される正規表現を指定します。少なくとも1つ以上の`(?P<timestamp>.+)`のような[名前付きキャプチャグループ](https://www.regular-expressions.info/named.html)を含まなければなりません。そしてそれは、`TS-Match-Name`パラメーターと共に使用されます。
* `TS-Match-Name` (文字型、設定必須): このパラメーターによって、抽出されたタイムスタンプを含む `Regex` パラメーターから名前付きキャプチャグループの名前を指定します。
* `Timestamp-Format-Override` (文字型、設定任意): これは、デフォルトの方式に代わって用いるタイムスタンプ解析ためのフォーマットを指定できます。利用可能な時間フォーマットは次の通りです:
	- AnsiC
	- Unix
	- Ruby
	- RFC822
	- RFC822Z
	- RFC850
	- RFC1123
	- RFC1123Z
	- RFC3339
	- RFC3339Nano
	- Apache
	- ApacheNoTz
	- Syslog
	- SyslogFile
	- SyslogFileTZ
	- DPKG
	- NGINX
	- UnixMilli
	- ZonelessRFC3339
	- SyslogVariant
	- UnpaddedDateTime
	- UnpaddedMilliDateTime
	- UK
	- Gravwell
	- LDAP
	- UnixSeconds
	- UnixMs
	- UnixNano

	いくつかのタイムスタンプフォーマットは重複する値を持ちえます(例えば、LDAPとUnixNanoは同じ桁数のタイムスタンプを生成することができる)。もし `Timestamp-Format-Override` が使われていない場合、プリプロセッサは上記の順序でタイムスタンプを導出しようとします。このリストの他のフォーマットと競合する可能性のあるタイムスタンプを使用する場合は、常に `Timestamp-Format-Override` を使用してください。

* `Timezone-Override` (文字型、設定任意): 抽出されたタイムスタンプにタイムゾーンが含まれていない場合は、このパラメーターで指定されたタイムゾーンを適用させることができます。例: `US/Pacific`, `Europe/Rome`, `Cuba`
* `Assume-Local-Timezone` (ブーリアン型、設定任意): このオプションは、タイムゾーンが含まれていない場合、プリプロセッサにタイムスタンプがローカルタイムゾーンにあると仮定するように指定します。従って `Timezone-Override` パラメーターとは相互に排他的な意味を持ちます。

### 一般的な使用例

多くのデータストリームにはタイムスタンプや、タイムスタンプとして簡単に解釈できる値を複数含まれているかもしれません。 regextimestampプリプロセッサを使うと、ログストリーム内の特定のタイムスタンプを強制的に探すことができます。 良い例としては、syslog経由で送信されるログストリームで、アプリケーションはそれ自身のタイムスタンプを含んでいますが、そのタイムスタンプをsyslog APIでは中継されないようになっています。 syslogラッパーは正しい形式のタイムスタンプを持っていますが、実際のデータストリームは正確なタイムスタンプを得るためには内部フィールドを使用する必要があるかもしれません。

### 例: ラッピングされたSyslogデータ

```
Nov 25 15:09:17 webserver alerts[1923]: Nov 25 14:55:34,GET,10.1.3.4,/info.php
```

インジェストされた上のようなエントリのTSフィールドの内部タイムスタンプ "Nov 25 14:55:34 "を抽出したいと思います。このタイムスタンプは行頭のsyslogタイムスタンプと同じフォーマットを使用しているので、タイムスタンプフォーマットを区別するルールをいくら駆使しても抽出できません。しかし、regexタイムスタンププリプロセッサならそれを抽出できるようにできます。名前指定でマッチしたタイムスタンプに、目的のタイムスタンプをキャプチャする正規表現を指定することで、エントリーのどこにあるタイムスタンプでも抽出することができます。このエントリでは、正規表現 `\S+\s+\S+\S+\s+\d+\s'. (?<timestamp>.+),` があれば、目的のタイムスタンプを適切に抽出することができます。

上記の例で示されているタイムスタンプを抽出するのには、次の設定が使えます:

```
[Preprocessor "ts"]
	Type=regextimestamp
	Regex="\S+\s+\S+\[\d+\]: (?P<timestamp>.+),"
	TS-Match-Name=timestamp
	Timezone-Override=US/Pacific
```

## Regex Extractionプリプロセッサ

トランスポートバスでは、実際のイベント内容を表さないかもしれないメタデータが追加されてデータストリームがラップされていることは非常に一般的です。 Syslogは、中のデータが本来的に持っているのとは異なる値のヘッダを付け加え、時にはそれによって単にデータの分析を複雑にしてしまうだけということもある、ありがちな例です。 regexextractorプリプロセッサによって、複数のフィールドを抽出し、新しい構造のフォーマットに再構築することができる正規表現を定義することが可能になります。

regexextraction プリプロセッサは、名前付き正規表現抽出フィールドとテンプレートを使用してデータを抽出し、それを出力レコードに再構成します。 出力テンプレートには、静的な値を含むことができ、必要に応じて出力データを完全に元とは異なる形に整形することができます。

テンプレートは、bashと似たようなフィールド定義を使用して、値をフィールド名で抽出参照します。 例えば、正規表現で `foo` という名前のフィールドを抽出した場合、その抽出内容をテンプレートに `${foo}` で挿入することができます。

Regex Extractionプリプロセッサの型は `regexextract` です。

### サポートされているオプション

* Passthrough-Misses (ブーリアン型、設定任意): このパラメーターは、正規表現が一致しない場合に、プリプロセッサがレコードを変更せずに通過させるかどうかを指定します。
* Regex (文字型、設定必須): このパラメーターでは、抽出のための正規表現を定義します。
* Template (文字型、設定必須): このパラメーターは、レコードの出力形式を定義します。

### 一般的な使用例

regexpreprocessor は、データストリームから不要なヘッダを除去するために使われるのが最も一般的ですが、データを処理しやすい形式に変換するために使用することもできます。

#### 例: syslogヘッダの削除

次のようなレコードのついて、syslogヘッダを削除して、JSON blobだけを送信したいと思います。


```
<30>1 2020-03-20T15:35:20Z webserver.example.com loginapp 4961 - - {"user": "bob", "action": "login", "result": "success", "UID": 123, "ts": "2020-03-20T15:35:20Z"}
```

このsyslogメッセージには、構造化されたJSON BLOBを含んでいますが、syslogトランスポートによって、必ずしもレコード内容を充実させるわけではない追加のメタデータが追加されています。 Regex Extractionを使用して、必要なデータを取り出し、使いやすいレコードに変換することができます。

そこで、Regex Extractionを使用してデータフィールドとホスト名を抜きだし、テンプレートを使用してホストを挿入した新しいJSON BLOBを作成します。

```
[Listener "logins"]
	Bind-String="0.0.0.0:7777"
	Preprocessor=loginapp

[Preprocessor "loginapp"]
	Type=regexextract
	Regex="\\S+ (?P<host>\\S+) \\d+ \\S+ \\S+ (?P<data>\\{.+\\})$"
	Template="{\"host\": \"${host}\", \"data\": ${data}}"
```

注: 正規表現では、文字セットを記述するためにバックスラッシュが頻用されますが、バックスラッシュはエスケープする必要があります！

結果として得られるデータは次のようになります:

```
{"host": "loginapp", "data": {"user": "bob", "action": "login", "result": "success", "UID": 123, "ts": "2020-03-20T15:35:20Z"}}
```

注：テンプレートでは、複数のフィールドに定数値を指定することができます。 また抽出されたフィールドを複数回挿入することもできます。

## forwarding プリプロセッサ

forwardingプリプロセッサは、ログストリームの行き先を分岐させて、別のエンドポイントに転送するために使用されます。 これは、追加のロギングツールを立ち上げるときや、外部のアーカイブプロセッサにデータを供給するときに非常に便利です。 forwardingプリプロセッサは、分岐プリプロセッサです。 つまり、データストリームを変更せず、追加のエンドポイントにデータを転送するだけであるという意味です。

デフォルトでは、フォワーディングプリプロセッサはブロッキングに設定されています。 これは、TCPやTLSのようなステートフル接続を使ってフォワーディングエンドポイントを指定した場合に、そのエンドポイントが起動していないか、データを受け入れることができない場合に、インジェストをブロックすることを意味しています。 この動作は `Non-Blocking` フラグを使うか、UDPプロトコルを使うことで変更することができます。

転送プリプロセッサは、転送されるデータストリームを削減したり、転送に必要なデータだけを指定したりするためのいくつかのフィルタ機構もサポートしています。 ストリームは、エントリタグのソースの情報や、実際のログデータに対する操作を行う正規表現を使用してフィルタリングすることができます。 各フィルタ指定は、OR パターンを作成するために複数回指定することができます。

複数の転送プリプロセッサを指定して、特定のログストリームを複数のエンドポイントに転送することができます。

転送プリプロセッサの型は `forwarder` です。

### サポートされているオプション

* `Target` (文字型、設定必須): 転送されるデータのエンドポイントを指定してください。 `unix` プロトコルを使わない場合、その指定は `host:port` のペアでなければなりません。
* `Protocol` (文字型、設定必須): データを転送する際に使用するプロトコルの設定。選択可能な値は `tcp`, `udp`, `unix`, `tls`。
* `Delimiter` (文字型、設定任意): Optional delimiter to use when sending data via the `raw` output format.`raw` 出力フォーマットでデータを送信する際に使用するデリミタの設定。
* `Format` (文字型、設定任意): データを送信する出力フォーマットの設定。 選択可能な値は `raw`, `json`, `syslog` 。
* `Tag` (文字型、設定任意、複数回指定可能): イベントをフィルタリングするために使用されるタグの指定。 複数回指定すると OR を意味します。
* `Regex` (文字型、設定任意、複数回指定可能): イベントをフィルタリングするために使用される正規表現の指定。 複数回指定すると OR を意味します。
* `Source` (文字型、設定任意、複数回指定可能): イベントをフィルタリングするために使用されるIP または CIDRタグの指定。 複数回指定すると OR を意味します。
* `Timeout` (符号なし整数型、設定任意、単位は秒): 転送先への接続および書き込み試行時のタイムアウト時間
* `Buffer` (符号なし整数型、設定任意): 機能がデータ送信時にバッファできるイベント数を指定できます。
* `Non-Blocking` (ブーリアン型、設定任意): True が指定された場合、forwading機能がデータを転送をベストエフォートでするが、インジェストをブロックしないことを指定します。
* `Insecure-Skip-TLS-Verify` (ブーリアン型、設定任意): TLS ベースの接続について、無効な TLS 証明書を無視できることを指定します。

### 例: 特定のホストからのsyslogについてだけ転送

この例では、SimpleRelay インジェスターを使って syslog メッセージをインジェストし、同時にその生データの一部を別のエンドポイントに転送しています。 `forward` プリプロセッサでは `Source` フィルタを使用して、`192.168.0.1` IP または `192.168.1.0/24` サブネット内のソースタグを持つログのみを転送しています。 ログは元のフォーマットで転送され、それぞれの間に改行を入れて転送されます。

```
[Listener "default"]              
	Bind-String="0.0.0.0:601"
	Reader-Type=rfc5424
	Tag-Name=syslog
	Preprocessor=forwardprivnet

[Preprocessor "forwardprivnet"]
	Type=Forwarder               
	Protocol=tcp
	Target="172.17.0.3:601"
	Format="raw"
	Delimiter="\n"
	Buffer=128
	Source=192.168.0.1
	Source=192.168.1.0/24
	Non-Blocking=false
```

### 例: 特定のWindowsイベントログのみを転送する

この例では、Federatorを使用して、多くのダウンストリームインジェスターからくるデータストリームの一部を転送しています。 `Tag` と `Regex` フィルタの両方を使用して特定のエントリのセットをキャプチャし、転送しています。 `syslog`形式を使用し、エンドポイントにRFC5424ヘッダとデータ本文をsyslogメッセージに収める形で送信していることに注意してください。syslogフォーマットを使って転送されるデータは、`gravwell`をホスト名に、エントリTAGをApp名に設定しています。

そして、`Tag` フィルタによって、`windows`または`sysmon`タグを使っているエントリのみを転送しようとしています。

さらに、特定のチャネル/イベントIDの組み合わせの時のみ、そのイベントデータを取得できるように、`Regex`フィルタを使用しています。 ここでは、セキュリティプロバイダからのログインイベントとsysmonプロバイダからの実行イベントデータのみを転送させています。

```
[IngestListener "enclaveA"]
	Ingest-Secret = IngestSuperSecrets
	Cleartext-Bind = 0.0.0.0:4023
	Tags=win*
	Preprocessor=forwardloginsysmon
	Preprocessor=forwardprivnet

[Preprocessor "forwardloginsysmon"]
	Type=Forwarder               
	Protocol=tcp
	Target="172.17.0.3:601"
	Format="syslog"
	Buffer=128
	Tag=windows
	Tag=sysmon
	Regex="Microsoft-Windows-Sysmon.+>(1|22)</EventID>"
	Regex="Microsoft-Windows-Security-Auditing.+>(4624|4625|4626)</EventID>"
	Non-Blocking=false

[Preprocessor "forwardsyslog"]
	Type=Forwarder               
	Protocol=tcp
	Target="172.17.0.3:601"
	Format="raw"
	Delimiter="\n"
	Buffer=128
	Tag=syslog
	Source=192.168.0.1
	Source=192.168.1.0/24
	Non-Blocking=false

```

### 例: ログを複数のホストに転送する

この例では、ログのサブセットを異なるフォーマットを使用して異なるエンドポイントに転送するために、Gravwell Federator を使用しています。 転送プリプロセッサは、他のプリプロセッサと同じ方法でスタックすることができるので、独自のフィルタ、エンドポイント、およびフォーマットで複数の転送プリプロセッサを指定することができます。

```
[IngestListener "enclaveB"]
	Ingest-Secret = IngestSuperSecrets
	Cleartext-Bind = 0.0.0.0:4123
	Tags=win*
	Preprocessor=forwardloginsysmon

[Preprocessor "forwardloginsysmon"]
	Type=Forwarder               
	Protocol=tcp
	Target="172.17.0.3:601"
	Format="syslog"
	Buffer=128
	Tag=windows
	Tag=sysmon
	Regex="Microsoft-Windows-Sysmon.+>(1|22)</EventID>"
	Regex="Microsoft-Windows-Security-Auditing.+>(4624|4625|4626)</EventID>"
	Non-Blocking=false


```

## Gravwell Forwarding プリプロセッサ

Gravwell Forwarding プリプロセッサによって、Gravwellの複数のインスタンスにエントリを複製することができる完全なGravwell muxerを作成することできます。 このプリプロセッサは、特定のGravwellデータストリームを別のインデクサーのセットに複製する必要があるテストや状況に有用です。 Gravwell Forwarding プリプロセッサは、パッケージ化されたインジェスターと同じ設定手法を利用して、インデクサー、インジェスト鍵、およびキャッシュコントロールまでをも指定できます。Gravwell Forwarding プリプロセッサはブロッキング設定されているので、ローカルキャッシュを有効にしてない場合、プリプロセッサが指定されたインデクサーにエントリを転送できない時にはインジェストパイプラインをブロックすることができることを意味します。

Gravwell Forwardingプリプロセッサの型は `gravwellforwarder` です。

### サポートされているオプション

すべてのGravwellインジェスターオプションの詳細が記されている、[全体設定のパラメーター](#!ingesters/ingesters.md#Global_Configuration_Parameters)セクションを参照してください。 インジェスターの全体設定オプションの殆どが、Gravwell Forwarder プリプロセッサによってもサポートされています。

### 例: Federatorでのデータの複製

この例は、すべてのエントリを 2つ目のクラスタに複製する完全なFederatorの設定です。

注: 通常のインジェストの流路をブロックしてしまわないように、フォワーディングプリプロセッサに `いつも` キャッシュが有効になるようにしています。

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Verify-Remote-Certificates = true
Cleartext-Backend-Target=172.19.0.2:4023 #example of adding a cleartext connection
Log-Level=INFO

[IngestListener "enclaveA"]
	Ingest-Secret = CustomSecrets
	Cleartext-Bind = 0.0.0.0:4423
	Tags=windows
	Tags=syslog-*
	Preprocessor=dup

[Preprocessor "dup"]
	Type=GravwellForwarder
	Ingest-Secret = IngestSecrets
	Connection-Timeout = 0
	Cleartext-Backend-Target=172.19.0.4:4023 #indexer1
	Cleartext-Backend-Target=172.19.0.5:4023 #indexer2 (cluster config)
	Ingest-Cache-Path=/opt/gravwell/cache/federator_dup.cache # must be a unique path
	Max-Ingest-Cache=1024 #Limit forwarder disk usage
```

### 例: 複製Fowardingを複数回重ねて使う

この例では、完全なフェデレータ設定と複数のGravwellプリプロセッサを指定して、単一のエントリストリームを複数のGravwellクラスタに複製できるようにしています。

注: プリプロセッサ制御ロジックは、同じクラスタに複数回転送していないかどうかはチェックしません。

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Verify-Remote-Certificates = true
Cleartext-Backend-Target=172.19.0.2:4023 #example of adding a cleartext connection
Log-Level=INFO

[IngestListener "enclaveA"]
	Ingest-Secret = CustomSecrets
	Cleartext-Bind = 0.0.0.0:4423
	Tags=windows
	Tags=syslog-*
	Preprocessor=dup1
	Preprocessor=dup2
	Preprocessor=dup3

[Preprocessor "dup1"]
	Type=GravwellForwarder
	Ingest-Secret = IngestSecrets1
	Cleartext-Backend-Target=172.19.0.101:4023
	Ingest-Cache-Path=/opt/gravwell/cache/federator_dup1.cache
	Max-Ingest-Cache=1024

[Preprocessor "dup2"]
	Type=GravwellForwarder
	Ingest-Secret = IngestSecrets2
	Cleartext-Backend-Target=172.19.0.102:4023
	Ingest-Cache-Path=/opt/gravwell/cache/federator_dup2.cache
	Max-Ingest-Cache=1024

[Preprocessor "dup3"]
	Type=GravwellForwarder
	Ingest-Secret = IngestSecrets3
	Cleartext-Backend-Target=172.19.0.103:4023
	Ingest-Cache-Path=/opt/gravwell/cache/federator_dup3.cache
	Max-Ingest-Cache=1024
```
