# 簡易リレー

シンプルリレーは、IPv4またはIPv6を介したプレーンテキストTCPおよび/またはUDPネットワーク接続を介して配信できる、テキストベースのデータソースの注目の的です。

シンプルリレーの一般的な使用例は次のとおりです。

*リモートsyslog収集
*ネットワーク経由のDevopログ収集
* Broセンサーのログ収集
*ネットワーク経由で配信可能なテキストソースとの簡単な統合

## 基本設定

Simple Relay ingesterは、[ingester section](#!ingesters/ingesters.md#Global_Configuration_Parameters)で説明されている統一されたグローバル構成ブロックを使用します。 他のほとんどのGravwellインジェスターと同様に、Simple Relayは複数のアップストリームインデクサー、TLS、クリアテキスト、名前付きパイプ接続、ローカルキャッシュ、ローカルロギングをサポートしています。

いくつかのポートでリッスンし、それぞれに一意のタグを適用するように構成されたSimple Relay ingesterの構成例は次のとおりです。

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify=false
Cleartext-Backend-target=127.0.0.1:4023 #example of a cleartext connection
Cleartext-Backend-target=127.1.0.1:4023 #example of second cleartext connection
Encrypted-Backend-target=127.1.1.1:4024 #example of encrypted connection
Pipe-Backend-Target=/opt/gravwell/comms/pipe #a named pipe connection
Ingest-Cache-Path=/opt/gravwell/cache/simple_relay.cache #local storage cache when uplinks fail
Max-Ingest-Cache=1024 #Number of MB to store, localcache will only store 1GB before stopping
Log-Level=INFO
Log-File=/opt/gravwell/log/simple_relay.log

#basic default logger, all entries will go to the default tag
#no Tag-Name means use the default tag
[Listener "default"]
	Bind-String="0.0.0.0:7777" #bind to all interfaces, with TCP implied
	#Lack of "Reader-Type" implines line break delimited logs
	#Lack of "Tag-Name" implies the "default" tag
	#Assume-Local-Timezone=false #Default for assume localtime is false
	#Source-Override="DEAD::BEEF" #override the source for just this listener

[Listener "syslogtcp"]
	Bind-String="tcp://0.0.0.0:601" #standard RFC5424 reliable syslog
	Reader-Type=rfc5424
	Tag-Name=syslog
	Assume-Local-Timezone=true #if a time format does not have a timezone, assume local time
	Keep-Priority=true	# leave the <nnn> priority tag at the start of each syslog entry

[Listener "syslogudp"]
	Bind-String="udp://0.0.0.0:514" #standard UDP based RFC5424 syslog
	Reader-Type=rfc5424
	Tag-Name=syslog
	Timezone-Override="America/Chicago"
	Keep-Priority=true	# leave the <nnn> priority tag at the start of each syslog entry
```

注：syslog検索モジュールを使用してsyslogエントリーを分析する場合は、Keep-Priorityフィールドが必要です。

## リスナー

各リスナーには、リスナーがラインリーダー、RFC5424リーダー、JSONリスナーのいずれであるかに関係なく、ユニバーサル構成値のセットが含まれます。

### ユニバーサルリスナーの構成パラメーター

リスナーは、プロトコルの指定、インターフェイスとポートのリッスン、および取り込み動作の微調整を可能にするいくつかの構成パラメーターをサポートします。

#### Bind-String

Bind-Stringパラメーターは、リスナーがバインドするインターフェイスとポートを制御します。 リスナーは、TCPまたはUDPポート、特定のアドレス、および特定のポートにバインドできます。 IPv4とIPv6がサポートされています。

```
#bind to all interfaces on TCP port 7777
Bind-String=0.0.0.0:7777

#bind to all interfaces on UDP port 514
Bind-String=udp://0.0.0.0:514

#bind to port 1234 on link local IPv6 address on interface p1p
Bind-String=[fe80::4ecc:6aff:fef9:48a3%p1p1]:1234

#bind to IPv6 globally routable address on TCP port 901
Bind-String=[2600:1f18:63ef:e802:355f:aede:dbba:2c03]:901
```

#### タイムスタンプを無視

「Ignore-Timestamps」パラメーターは、読み取り値からタイムスタンプを取得しようとせず、代わりに現在のタイムスタンプを適用するようリスナーに指示します。 このパラメーターは、タイムスタンプが存在しない可能性のあるデータを読み取る場合、または信頼できないシステムクロックのために発信元システムでタイムスタンプが間違っている場合に役立ちます。 「Ignore-Timestamps」はデフォルトでfalseであり、有効にするために `` `Ignore-Timestamps = true```を指定します

#### Assume-Local-TimezoneおよびTimezone-Override

ほとんどのタイムスタンプ形式には、協定世界時（UTC）へのオフセットを示すタイムゾーンが添付されています。 ただし、一部のシステムでは、タイムゾーンを指定せずにログエントリのタイムゾーンを決定するために受信者に任せています。Assume-Local-Timezoneは、タイムスタンプがSimple Relayリーダーと同じタイムゾーンにあるとリーダーに仮定させます ティムゼオンは省略されます。 Timezone-Overrideは、IANAタイムゾーンデータベース形式の文字列（例： "America / Chicago"）を受け取り、そのタイムゾーンをタイムゾーンを指定しないタイムスタンプに適用します。

Assume-Local-TimezoneとTimezone-Overrideは相互に排他的です。

#### ソースオーバーライド

「Source-Override」パラメーターは、データのソースを無視し、ハードコードされた値を適用するようリスナーに指示します。 データソースを整理および/またはグループ化する方法として、着信データのソース値をハードコードすることが望ましい場合があります。 「Source-Override」の値は、IPv4またはIPv6の値にすることができます。

```
Source-Override=192.168.1.1
Source-Override=127.0.0.1
Source-Override=[fe80::899:b3ff:feb7:2dc6]
```

#### タイムスタンプ形式のオーバーライド

Timestamp-Format-OverrideDataの値には複数のタイムスタンプが含まれている場合があり、データからタイムスタンプを取得しようとすると混乱が生じる可能性があります。 通常、リスナーは取得可能な左端のタイムスタンプを取得しますが、非常に特定の形式のタイムスタンプのみを検索することが望ましい場合があります。 「Timestamp-Format-Override」は、特定の形式のタイムスタンプのみを尊重するようリスナーに指示します。 次のタイムスタンプ形式が利用可能です。

* AnsiC
* Unix
* Ruby
* RFC822
* RFC822Z
* RFC850
* RFC1123
* RFC1123Z
* RFC3339
* RFC3339Nano
* Apache
* ApacheNoTz
* Syslog
* SyslogFile
* SyslogFileTZ
* DPKG
* Custom1Milli
* NGINX
* UnixMilli
* ZonelessRFC3339
* SyslogVariant
* UnpaddedDateTime

リスナーにRFC3339仕様に一致するタイムスタンプのみを強制的に検索させるには、リスナーに `` `Timestamp-Format-Override = RFC3339```を追加します。

### リスナーリーダーの種類と構成

シンプルリレーは、さまざまな状況で役立つ次の種類の基本的なリーダーをサポートしています

* ラインリーダー
* RFC5424

基本的なリスナーは「リスナー」ブロックを介して指定され、行区切りおよびRFC5424リーダータイプをサポートします。 各リスナーには、一意の名前と一意のバインドポートが必要です（2つの異なるリスナーが同じプロトコル、アドレス、ポートにバインドすることはできません）。 基本的なリスナーのリーダータイプは、「Reader-Type」パラメーターによって制御されます。 現在、2種類のリスナー（回線とRFC5424）があります。 Reader-Typeが指定されていない場合、ラインリーダーのタイプが想定されます。

また、基本リスナーでは、各リスナーが「Tag-Name」パラメーターを介してすべての着信データに適用するタグを指定する必要があります。 「Tag-Name」パラメーターを省略すると、「default」タグが適用されます。 TCPポート5555で改行データを予期し、「テスト」タグを適用する「テスト」という最も基本的なリスナーには、次の構成仕様があります。

```
[Listener "test"]
	Bind-String=0.0.0.0:5555
	Tag-Name=testing
```

#### ラインリーダーリスナー

ラインリーダーリスナーは、TCPまたはUDPストリームから改行の壊れたデータストリームを読み取るように設計されています。 ネットワークを介して単純な改行データを配信できるアプリケーションは、このタイプのリーダーを利用して非常に簡単かつ簡単にGravwellと統合できます。 Line Readerリスナーは、ログファイルをリスニングポートに送信するだけで、単純なログファイルの配信にも使用できます。

たとえば、既存のログファイルは、netcatとSimple Relayを使用してGravwellにインポートできます。

```
nc -q 1 10.0.0.1 7777 < /var/log/syslog
```

##### ラインリーダーリスナーの例

最も基本的なリスナーは、リスナーにリッスンするポートを指示する「バインド文字列」引数を1つだけ必要とします。

```
[Listener "default"]
	Bind-String="0.0.0.0:7777" #bind to all interfaces, with TCP implied
```

#### RFC5424リスナー

RFC5424またはRFC3164に基づく構造化されたsyslogメッセージを受け入れるように設計されたリスナーにより、Simple Relayがsyslog集約ポイントとして機能できるようになります。 ポート601で信頼できるTCP接続を使用してsyslogメッセージを予期するリスナーを有効にするには、「Reader-Type」を「RFC5424」に設定します。

```
[Listener "syslog"]
	Bind-String=0.0.0.0:601
	Reader-Type=RFC5424
```

ポート514を介してステートレスUDP経由でsyslogメッセージを受け入れる場合、リスナーは次のようになります。

```
[Listener "syslog"]
	Bind-String=udp://0.0.0.0:514
	Reader-Type=RFC524
```

RFC5424リーダータイプは、デフォルトでtrueに設定されている「Keep-Priority」という名前のパラメーターもサポートしています。 典型的なsyslogメッセージには優先度識別子が付加されますが、一部のユーザーは保存されたメッセージから優先度を破棄したい場合があります。 これは、RFC5424ベースのリスナーに「Keep-Priority = false」を追加することで実現されます。 行ベースのリスナーは、「Keep-Priority」パラメーターを無視します。

優先度が付加されたsyslogメッセージの例：

```
<30>Sep 11 17:04:14 router dhcpd[9987]: DHCPREQUEST for 10.10.10.82 from e8:c7:4f:04:e1:af (Chromecast) via insecure
```

エントリから優先度タグを削除するリスナー仕様の例：

```
[Listener "syslog"]
	Bind-String=udp://0.0.0.0:514
	Reader-Type=RFC524
	Keep-Priority=false
```

注：syslogメッセージの優先順位部分は、RFC仕様で体系化されています。 優先度を削除すると、Gravwell [syslog](#!search/syslog/syslog.md) 検索モジュールが値を適切に解析できなくなります。 有料のGravwellライセンスはすべて無制限であり、syslogメッセージに優先度フィールドが残っていることをお勧めします。 syslog検索モジュールは、syslogメッセージを正規表現で解析するよりも劇的に高速です。

### JSONリスナー

JSONリスナータイプを使用すると、取り込み時に穏やかなJSON処理が可能になります。 JSONリーダーの目的は、JSONエントリのフィールドの値に基づいてエントリに一意のタグを適用することです。 多くのアプリケーションは、JSONの形式を示すフィールドを使用してJSONデータをエクスポートします。処理効率の観点から、さまざまな形式に特定のタグをタグ付けすると有益な場合があります。

優れた使用例は、多くのBroセンサーアプライアンスに見られるJSON over TCPデータエクスポート機能です。 アプライアンスは、すべてのBroログデータを単一のTCPストリームでエクスポートしますが、ストリーム内には異なるモジュールによって構築された複数のデータタイプがあります。 JSONリスナーを使用して、モジュールフィールドからデータ型を導出し、一意のタグを適用できます。 これにより、Bro connのログを1つに、Bro DNSのログを別のログに、他のすべてのBroのログを別のログに保存するなどのことができます。 その結果、異なるタグでデータ型を区別し、単一のストリームを介して複数のJSONデータ型が入力されるときにGravwell Wellsを利用できます。

#### JSONリスナー構成パラメーター

JSONリスナーブロックは、上記のユニバージョンリスナータイプを実装します。 追加のパラメーターにより、タグを定義するためにピボットするフィールドを指定できます。

##### 抽出パラメータ

「Extractor」パラメータは、JSONエントリからフィールドを取得するために使用されるJSON抽出文字列を指定します。 抽出文字列は、Gravwell [json](#!search/json/json.md) 検索モジュールと同じ構文からインラインフィルタリングを除いたものに従います。

次のJSONを考えます：

```
{
  "time": "2018-09-12T12:25:33.503294982-06:00",
  "class": 5.1041415140005e+18,
  "data": "Have I come to Utopia to hear this sort of thing?",
  "identity": {
    "user": "alexanderdavis605",
    "name": "Noah White",
    "email": "alexanderdavis605@test.org",
    "phone": "+52 27 83 68 75069 2"
  },
  "location": {
    "address": "43 Wilson Pkwy,\nBury, AL, 66232",
    "state": "PW",
    "country": "Pakistan"
  },
  "group": "carp",
  "useragent": "Mozilla\/5.0 (X11; Fedora; Linux x86_64) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/52.0.2743.116 Safari\/537.36",
  "ip": "8.83.94.200"
}
```

次のExtractionパラメーターを使用して、場所と状態の値を抽出し、検出した状態の略語に基づいてタグを適用できます。

```
Extractor=location.state
```

##### Tag-Match

各JSONListenerは、複数のフィールド値をサポートして、一致仕様をタグ付けします。 タグ割り当ての値は、<タグ値>パラメーターの引数として<フィールド値>：<タグ名>の形式で指定されます。

たとえば、値が「foo」のフィールドを抽出し、それをタグ「bar」に割り当てたい場合、JSONListener構成ブロックに次を追加します。

```
Tag-Match=foo:bar
```

フィールド抽出値には「：」文字を含めることができます。フィールド値に「：」文字を含めると、値を二重引用符でカプセル化できます。

たとえば、抽出した値「foo：bar」にタグ「baz」を割り当てたい場合、「Tag-Match」パラメーターは次のようになります。

```
Tag-Match="foo:bar":baz
```

抽出値からタグへのマッピングは多対1にすることができます。つまり、複数の抽出値を同じタグにマッピングできます。 たとえば、次のパラメーターは、「foo」と「bar」の両方の抽出値をタグ「baz」にマッピングします。

```
Tag-Match=foo:baz
Tag-Match=bar:baz
```

ただし、単一の抽出値を複数のタグにマッピングすることはできません。 次は無効です。

```
Tag-Match=foo:baz
Tag-Match=foo:bar
```

##### Default-Tag

フィールドを抽出してタグを適用するときに、一致するTag-Matchが指定されていない場合、JSONListenerはデフォルトのタグを適用します。

#### JSONListenerの動作の例

次の構成済みJSONListenerを想定します。
```
[JSONListener "testing"]
	Bind-String=0.0.0.0:7777
	Extractor="field1"
	Default-Tag=json
	Tag-Match=test1:tag1
	Tag-Match=test2:tag2
	Tag-Match=test3:tag3
```

いくつかのサンプルJSONデータと結果のタグ：

##### 一致したフィールド

```
{ "field1": "test1", "field2": "test2" }
```

フィールド「field1」が「Tag-Match = test1：tag1」と一致したため、エントリはタグ「tag1」を取得します

##### 一致しないフィールド

```
{ "field1": "foobar", "field2": "test2" }
```

フィールド「field1」が「Tag-Match」パラメーターと一致しなかったため、エントリはタグ「json」を取得します。

##### 抽出フィールドが見つかりません

```
{ "fieldfoo": "test1", "fieldbar": "test2" }
```

抽出プログラムはフィールド「field1」を見つけることができなかったため、エントリはタグ「json」を取得します。