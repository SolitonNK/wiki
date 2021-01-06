# Simple Relay

Simple Relayは、IPv4またはIPv6での平文TCP、暗号化TCPまたは平文UDPネットワーク接続を介して配信できる、テキストベースのデータソースのためのインジェスターです。

Simple Relayの一般的な使用例は次のとおりです。

* リモートsyslog収集
* ネットワーク経由のDevopログ収集
* Broセンサーのログ収集
* ネットワーク経由で配信可能なテキストソースとの簡単な統合

## 基本設定

Simple Relayは、[インジェスター設定](#!ingesters/ingesters.md#Global_Configuration_Parameters)で説明されている統一されたグローバル構成ブロックを使用します。 他のほとんどのGravwellインジェスターと同様に、Simple Relayは複数のアップストリームインデクサー、TLS、クリアテキスト、名前付きパイプ接続、ローカルキャッシュ、ローカルロギングをサポートしています。

いくつかのポートでリッスンし、それぞれに一意のタグを適用するように構成されたSimple Relayインジェスターの構成例は次のとおりです：

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

注：syslog検索モジュールを使用してsyslogエントリーを分析する場合は、`Keep-Priority`フィールドが必要です。

## リスナー

各リスナーには、リスナーがラインリーダー、RFC5424リーダー、JSONリスナーのいずれであるかに関係なく、ユニバーサル構成値のセットが含まれます。

### ユニバーサルリスナーの構成パラメーター

リスナーは、プロトコルの指定、インターフェイスとポートのリッスン、および取り込み動作の微調整を可能にするいくつかの構成パラメーターをサポートします。

#### Bind-String

Bind-String パラメータは、リスナーがどのインターフェイスとポートにバインドするかを制御します。 リスナーは、平文/暗号化TCPポートまたは平文UDPポート、特定のアドレス、特定のポートにバインドできます。 IPv4 と IPv6 がサポートされています。

```
#bind to all interfaces on TCP port 7777
Bind-String=0.0.0.0:7777

#bind to all interfaces on UDP port 514
Bind-String=udp://0.0.0.0:514

#bind to port 1234 on link local IPv6 address on interface p1p
Bind-String=[fe80::4ecc:6aff:fef9:48a3%p1p1]:1234

#bind to IPv6 globally routable address on TCP port 901
Bind-String=[2600:1f18:63ef:e802:355f:aede:dbba:2c03]:901

#listen for TLS connections on port 9999
Bind-String=tls://0.0.0.0:9999
```

#### Cert-File

TLS の `Bind-String` を使う場合は、`Cert-File` も指定しなければなりません。値は有効なTLS証明書を含むファイルへのパスでなければなりません。

```
Cert-File=/opt/gravwell/etc/cert.pem
``` 

#### Key-File

TLS の `Bind-String` を使う場合は、`Key-File` も指定しなければなりません。値は有効なTLSキーを含むファイルへのパスでなければなりません。

```
Key-File=/opt/gravwell/etc/key.pem
``` 

#### Ignore-Timestamps

"Ignore-Timestamps"パラメーターは、読み取り値からタイムスタンプを取得しようとせず、代わりに現在のタイムスタンプを適用するようリスナーに指示します。 このパラメーターは、タイムスタンプが存在しない可能性のあるデータを読み取る場合、または信頼できないシステムクロックのために発信元システムでタイムスタンプが間違っている場合に役立ちます。 "Ignore-Timestamps"はデフォルトでfalseであり、有効にするには ```Ignore-Timestamps=true```を指定します。

#### Assume-Local-TimezoneおよびTimezone-Override

ほとんどのタイムスタンプ形式には、世界標準時(UTC)へのオフセットを示すタイムゾーンが添付されています。 しかし、システムによってはタイムゾーンを指定しないものもあり、ログエントリがどのタイムゾーンにあるかは受信者の判断に委ねられています。 Assume-Local-Timezoneは、タイムゾーンが省略された場合、リーダーはタイムスタンプがSimple Relayリーダーと同じタイムゾーンにあると仮定します。Timezone-Overrideは、IANAタイムゾーンデータベース形式(例: "America/Chicago")の文字列を受け取り、タイムゾーンを指定しないタイムスタンプにそのタイムゾーンを適用します。

Assume-Local-TimezoneとTimezone-Overrideは相互に排他的です。

#### Source-Override

"Source-Override"パラメータは、データのソースを無視し、ハードコードされた値を適用するようリスナーに指示します。 データソースを整理あるいはグループ化する方法として、受信データのソース値をハードコードすることが望ましい場合があります。 "Source-Override "の値は、IPv4またはIPv6の値にすることができます。

```
Source-Override=192.168.1.1
Source-Override=127.0.0.1
Source-Override=[fe80::899:b3ff:feb7:2dc6]
```

#### Timestamp-Format-Override

データには複数のタイムスタンプが含まれている場合があり、データからタイムスタンプを取得しようとすると混乱が生じる可能性があります。 通常、リスナーは取得可能な左端のタイムスタンプを取得しますが、特定の形式のタイムスタンプのみを取得することが望ましい場合があります。"Timestamp-Format-Override"は、特定の形式のタイムスタンプのみを取得するようリスナーに指示します。 次のタイムスタンプ形式が利用可能です。

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

リスナーにRFC3339仕様に一致するタイムスタンプのみを強制的に取得させるには、リスナーに ```Timestamp-Format-Override=RFC3339```を追加します。

### リスナーリーダーの種類と構成

シンプルリレーは、さまざまな状況で役立つ次の種類の基本的なリーダーをサポートしています

* ラインリーダー
* RFC5424

基本的なリスナーは"Listener"ブロックで指定され、行区切りおよびRFC5424リーダータイプをサポートしています。各リスナーには、一意の名前と一意のバインドポートが必要です(2つの異なるリスナーを同じプロトコル、アドレス、ポートにバインドすることはできません)。基本的なリスナーのリーダタイプは、"Reader-Type"パラメータで制御されます。現在、2種類のリスナー(lineとRFC5424）があります。Reader-Typeが指定されていない場合、lineリーダータイプが想定されます。

また、基本リスナーでは、各リスナーが"Tag-Name"パラメーターを介してすべての受信データに適用するタグを指定する必要があります。"Tag-Name"パラメーターを省略すると、"default"タグが適用されます。TCPポート5555で改行済データを期待し、"testing"タグを適用する"test"という最も基本的なリスナーには、次のような設定を持ちます。

```
[Listener "test"]
	Bind-String=0.0.0.0:5555
	Tag-Name=testing
```

#### ラインリーダーリスナー

ラインリーダーリスナーは、TCPまたはUDPストリームから改行済データストリームを読み取るように設計されています。 ネットワークを介して単純な改行済データを配信できるアプリケーションは、このタイプのリーダーを利用して非常に簡単かつ簡単にGravwellと統合できます。ラインリーダーリスナーは、ログファイルをリスニングポートに送信するだけで、単純なログファイルの配信にも使用できます。

たとえば、既存のログファイルは、netcatとSimple Relayを使用してGravwellにインポートできます。
```
nc -q 1 10.0.0.1 7777 < /var/log/syslog
```

##### ラインリーダーリスナーの例

最も基本的なリスナーは、リスナーにリッスンするポートを指示する"Bind-String"引数を1つだけ必要とします。

```
[Listener "default"]
	Bind-String="0.0.0.0:7777" #bind to all interfaces, with TCP implied
```

#### RFC5424リスナー

RFC5424またはRFC3164に基づく構造化されたsyslogメッセージを受け入れるように設計されたリスナーにより、Simple Relayがsyslog集約ポイントとして機能できるようになります。 ポート601で信頼できるTCP接続を使用してsyslogメッセージを予期するリスナーを有効にするには、"Reader-Type"を"RFC5424"に設定します。

```
[Listener "syslog"]
	Bind-String=0.0.0.0:601
	Reader-Type=RFC5424
```

ポート514を介してステートレスUDP経由でsyslogメッセージを受け取る場合、リスナーは次のようになります。

```
[Listener "syslog"]
	Bind-String=udp://0.0.0.0:514
	Reader-Type=RFC524
```

RFC5424リーダータイプは、デフォルトでtrueに設定されている"Keep-Priority"という名前のパラメーターもサポートしています。 典型的なsyslogメッセージには優先度識別子が付加されますが、一部のユーザーは保存されたメッセージから優先度を破棄したい場合があります。 これは、RFC5424ベースのリスナーに"Keep-Priority=false"を追加することで実現されます。 行ベースのリスナーは、"Keep-Priority"パラメーターを無視します。

優先度が付加されたsyslogメッセージの例：

```
<30>Sep 11 17:04:14 router dhcpd[9987]: DHCPREQUEST for 10.10.10.82 from e8:c7:4f:04:e1:af (Chromecast) via insecure
```

エントリから優先度を削除するリスナー設定の例：

```
[Listener "syslog"]
	Bind-String=udp://0.0.0.0:514
	Reader-Type=RFC524
	Keep-Priority=false
```

注：syslogメッセージの優先度は、RFC仕様で体系化されています。 優先度を削除すると、Gravwell [syslog](#!search/syslog/syslog.md) 検索モジュールが値を適切に解析できなくなります。 有料のGravwellライセンスはすべて無制限ですので、syslogメッセージに優先度フィールドを残すことをお勧めします。 syslog検索モジュールは、syslogメッセージを正規表現で解析するよりも劇的に高速です。

### JSONリスナー

JSONリスナータイプを使用すると、取り込み時に穏やかなJSON処理が可能になります。 JSONリーダーの目的は、JSONエントリのフィールドの値に基づいてエントリに一意のタグを適用することです。 多くのアプリケーションは、JSONの形式を示すフィールドを使用してJSONデータをエクスポートします。処理効率の観点から、さまざまな形式に特定のタグをタグ付けすると有益な場合があります。

優れた使用例は、多くのBroセンサーアプライアンスに見られるJSON over TCPデータエクスポート機能です。 アプライアンスは、すべてのBroログデータを単一のTCPストリームでエクスポートするため、ストリーム内には異なるモジュールによって構築された複数のデータタイプがあります。 JSONリスナーを使用すれば、モジュールフィールドからデータ型を導出し、一意のタグを適用できます。 これにより、Bro connのログを1つに、Bro DNSのログを別のログに、他のすべてのBroのログを別のログに保存するといった操作ができます。 その結果、複数のJSONデータ型が1つのストリームを経由して入ってくる場合にも、異なるタグでデータ型を区別して、Gravwell Wellsを活用できるようになります。

#### JSONリスナー構成パラメーター

JSONリスナーブロックは、上記のようにユニバージョンリスナータイプを実装しています。 追加のパラメーターにより、タグを定義するためにピボットしたいフィールドを指定できます。

##### 抽出パラメータ

"Extractor"パラメータには、JSONエントリからフィールドを引き出すために使用されるJSON抽出文字列を指定します。抽出文字列は、Gravwell [json](#!search/json/json.md) 検索モジュールと同じ構文からインラインフィルタリングを除いたものに従います。

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

**Tag-Match**

各JSONListenerは、複数のフィールド値からのタグマッチ指定をサポートしています。タグへの値の割り当ては、"Tag-Match"パラメーターの引数として"フィールド値:タグ名"の形式で指定されます。

例えば、"foo "という値を持つフィールドを抽出し、それを "bar "というタグに割り当てたい場合は、次のようにJSONListenerの設定ブロックに追加します。

```
Tag-Match=foo:bar
```

フィールドの抽出値は":"を含むことができます。":"を含むフィールド値を指定するには、二重引用符で値を囲みます。

例えば、タグ "baz" を抽出された値 "foo:bar" に代入したい場合、"Tag-Match" パラメータは次のようになります。

```
Tag-Match="foo:bar":baz
```

抽出値からタグへのマッピングは、複数対1にできます。つまり複数の抽出値を同じタグにマッピングできます。例えば、以下のパラメータは、"foo"と "bar"の両方の抽出値をタグ "baz"にマッピングします。

```
Tag-Match=foo:baz
Tag-Match=bar:baz
```

ただし、単一の抽出値を複数のタグにマッピングすることはできません。 次は無効です。

```
Tag-Match=foo:baz
Tag-Match=foo:bar
```

**Default-Tag**

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

##### フィールドが一致した場合

```
{ "field1": "test1", "field2": "test2" }
```

フィールド"field1"が"Tag-Match=test1:tag1"と一致したため、エントリーはタグ"tag1"を取得します。

##### フィールドが一致しない場合

```
{ "field1": "foobar", "field2": "test2" }
```

フィールド"field1"が"Tag-Match"パラメータと一致しなかったため、エントリーはタグ"json"を取得します。

##### 抽出フィールドが見つからない場合

```
{ "fieldfoo": "test1", "fieldbar": "test2" }
```

抽出器がフィールド"field1"を見つけることができなかったため、エントリはタグ"json"を取得します。
