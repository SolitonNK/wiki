# ファイルフォロワー

ファイルフォロワーingesterは、それらのファイルがオンザフライで更新される可能性がある状況で、ローカルファイルシステムにファイルを取り込む最良の方法です。 ファイルの各行を単一のエントリとして取り込みます。

File Followerの最も一般的な使用例は、/ var / logなどのアクティブに更新されているログファイルを含むディレクトリを監視することです。 ログのローテーションをインテリジェントに処理し、「logfile」が「logfile.1」などに移動されたことを検出します。 ディレクトリ内の特定のパターンに一致するファイルを取り込むように構成でき、オプションでその最上位ディレクトリのサブディレクトリに再帰的に下降します。

## 基本設定

ファイルフォロワー設定ファイルは、デフォルトではLinuxの `/opt/gravwell/etc/file_follow.conf`とWindowsの`C:\Program Files\gravwel\file_follow.cfg`にあります。

File Follower ingesterは、[ingesterセクション](#!ingesters/ingesters.md#Global_Configuration_Parameters)で説明されている統一されたグローバル構成ブロックを使用します。 他のほとんどのGravwellインジェスターと同様に、File Followerは複数のアップストリームインデクサー、TLS、クリアテキスト、名前付きパイプ接続、およびローカルロギングをサポートしています。

注：既にソースファイル内での位置を追跡しているため、File Follower ingesterでファイルキャッシュを使用することを強くお勧めします。

/ var / log内のいくつかの異なるタイプのログファイルを監視し、/ tmp / incomingの下のファイルを再帰的に追跡するように構成されたFileフォロワーの構成例：

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify = false
Cleartext-Backend-target=172.20.0.1:4023 #example of adding a cleartext connection
Cleartext-Backend-target=172.20.0.2:4023 #example of adding another cleartext connection
State-Store-Location=/opt/gravwell/etc/file_follow.state
Log-Level=ERROR #options are OFF INFO WARN ERROR
Max-Files-Watched=64

[Follower "syslog"]
        Base-Directory="/var/log/"
        File-Filter="syslog,syslog.[0-9]" #we are looking for all authorization log files
        Tag-Name=syslog
        Assume-Local-Timezone=true
[Follower "auth"]
        Base-Directory="/var/log/"
        File-Filter="auth.log,auth.log.[0-9]" #we are looking for all authorization log files
        Tag-Name=syslog
        Assume-Local-Timezone=true #Default for assume localtime is false
[Follower "packages"]
        Base-Directory="/var/log"
        File-Filter="dpkg.log,dpkg.log.[0-9]" #we are looking for all dpkg files
        Tag-Name=dpkg
        Ignore-Timestamps=true
[Follower "external"]
        Base-Directory="/tmp/incoming"
		Recursive=true
        File-Filter="*.log"
        Tag-Name=external
		Timezone-Override="America/Los_Angeles"
```

この例では、「syslog」フォロワーが `/var/log/syslog`とそのローテーションを読み取り、syslogタグに行を取り込み、日付がローカルタイムゾーンにあると想定します。 同様に、「auth」フォロワーもsyslogタグを使用して、 `/var/log/auth.log`を取り込みます。 「パッケージ」フォロワーは、Debianのパッケージ管理ログをdpkgタグに取り込みます。 説明のために、タイムスタンプを無視し、各エントリに読み取られた時間をマークします。

最後に、「外部」フォロワーはディレクトリ `/tmp/incoming`から`.log`で終わるすべてのファイルを読み取り、再帰的にディレクトリに降ります。 タイムスタンプを太平洋時間帯にあるかのように解析します。 このフォロワーは、たとえば、米国西海岸の複数のサーバーが定期的にログファイルをこのシステムにアップロードした場合に役立つ構成を示しています。

上記で使用される構成パラメーターについては、次のセクションで詳しく説明します。

## 追加のグローバルパラメーター

### 最大ファイル監視数

Max-Files-Watchedパラメーターは、ファイルフォロワーが開いているファイル記述子を維持しすぎるのを防ぎます。 「Max-Files-Watched = 64」が指定されている場合、ファイルフォロワーは最大64個のログファイルをアクティブに監視します。 新しいファイルが作成されると、ファイルフォロワーは、新しいファイルを監視するために、最も古い既存ファイルの監視を積極的に停止します。 ただし、古いファイルが後で更新されると、キューの先頭に戻ります。

ほとんどの場合、この設定は64のままにしておくことをお勧めします。 制限の設定が高すぎると、カーネルによって設定された制限に達する可能性があります。

## フォロワー構成

File Follower構成ファイルには、1つ以上の「フォロワー」ディレクティブが含まれています。

```
[Follower "syslog"]
        Base-Directory="/var/log/"
        File-Filter="syslog,syslog.[0-9]" #we are looking for all authorization log files
        Tag-Name=syslog
```

各フォロワーは、少なくともベースディレクトリとファイル名のフィルタリングパターンを指定します。 このセクションでは、フォロワーごとに設定可能な構成パラメーターについて説明します。

###	ベースディレクトリ

Base-Directoryパラメーターは、取り込むファイルが含まれるディレクトリを指定します。 絶対パスで、ワイルドカードを含めないでください。

### ファイルフィルター

File-Filterパラメーターは、取り込む必要のあるファイル名を定義します。 単一のファイル名のように単純にすることができます。

```
File-Filter="foo.log"
```

または、複数のパターンを含めることができます。

```
File-Filter="kern*.log,kern*.log.[0-9]"
```

「kern」で始まり「.log」で終わる、または「kern」で始まり「.log.0」から「.log.9」で終わるファイル名に一致します。

[https://golang.org/pkg/path/filepath/#Match](https://golang.org/pkg/path/filepath/#Match)で定義されている完全一致構文：
```
pattern:
	{ term }
term:
	'*'         matches any sequence of non-Separator characters
	'?'         matches any single non-Separator character
	'[' [ '^' ] { character-range } ']'
	            character class (must be non-empty)
	c           matches character c (c != '*', '?', '\\', '[')
	'\\' c      matches character c

character-range:
	c           matches character c (c != '\\', '-', ']')
	'\\' c      matches character c
	lo '-' hi   matches character c for lo <= c <= hi
```

### タグ名

Tag-Nameパラメーターは、このフォロワーによって取り込まれたエントリに適用するタグを指定します。

### タイムスタンプを無視

Ignore-Timestampsパラメーターは、フォロワーがファイルの各行からタイムスタンプを抽出しようとするのではなく、各行に現在時刻をタグ付けすることを示します。

### 想定ローカルタイムゾーン

Assume-Local-Timezoneは、デフォルトのUTCではなくローカルタイムゾーンにあるかのようにタイムゾーンの指定がないタイムスタンプを解析するように命令するブール設定です。

Assume-Local-TimezoneとTimezone-Overrideは相互に排他的です。

### タイムゾーンのオーバーライド

Timezone-Overrideパラメーターは、デフォルトのUTCではなく、指定されたタイムゾーンにあるかのようにタイムゾーンの指定がないタイムスタンプを解析するように、ingesterに指示します。 タイムゾーンは、[https://en.wikipedia.org/wiki/List_of_tz_database_time_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)に示されているIANAデータベース文字列形式で指定する必要があります。 たとえば、米国中部時間は次のように指定する必要があります。

```
Timezone-Override="America/Chicago"
```

Assume-Local-TimezoneとTimezone-Overrideは相互に排他的です。 `Timezone-Override="Local"`は、機能的には `Assume-Local-Timezone=true`と同等です。

### 再帰的

recursiveパラメーターは、File-Filterに一致するファイルをBase-Directoryの下で再帰的に取り込むようにファイルフォロワーに指示します。

デフォルトでは、Ingesterは、Base-Directoryの最上位の下にあるFile-Filterに一致するファイルのみを取り込みます。 以下は `/tmp/incoming/foo.log`を取り込みますが、`/tmp/incoming/system1/foo.log`は取り込みません：

```
Base-Directory="/tmp/incoming"
File-Filter="foo.log"
Recursive=false
```

Recusive = trueを設定すると、設定はfoo.logという名前の** any **ファイルを、 `/tmp/incoming`の下の任意のディレクトリの深さで取り込みます。

### 行のプレフィックスを無視

ingesterは、Ignore-Line-Prefixに渡された文字列で始まる行を（取り込みではなく）ドロップします。 これは、Broログなどのコメントを含むログファイルを取り込むときに役立ちます。 Ignore-Line-Prefixパラメーターは複数回指定できます。

以下は、 `＃`または `//`で始まる行を取り込むべきではないことを示しています。

```
Ignore-Line-Prefix="#"
Ignore-Line-Prefix="//"
```

### タイムスタンプ形式のオーバーライド

データ値には複数のタイムスタンプが含まれている場合があり、データからタイムスタンプを取得しようとすると混乱が生じる可能性があります。 通常、フォロワーは取得可能な左端のタイムスタンプを取得しますが、非常に特殊な形式のタイムスタンプのみを検索することが望ましい場合があります。 「Timestamp-Format-Override」は、特定の形式のタイムスタンプのみを尊重するようフォロワーに指示します。 次のタイムスタンプ形式を使用できます。

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

フォロワーにRFC3339仕様に一致するタイムスタンプのみを強制的に検索させるには、「 `` Timestamp-Format-Override = RFC3339```」をフォロワーに追加します。

### タイムスタンプ区切り

Timestamp-Delimitedパラメーターは、タイムスタンプが発生するたびに新しいエントリの開始と見なされることを指定するブール値です。 これは、ログエントリが複数行にわたる場合に役立ちます。 Timestamp-Delimitedを指定する場合、Timestamp-Format-Overrideパラメーターも設定する必要があります。

ログファイルが次のような場合：

```
2012-11-01T22:08:41+00:00 Line 1 of the first entry
Line 2 of the first entry
2012-11-01T22:08:43+00:00 Line 1 of the second entry
Line 2 of the second entry
Line 3 of the second entry
```

フォロワーが `Timestamp-Delimited=true`および`Timestamp-Format-Override=RFC3339`で設定されている場合、次の2つのエントリが生成されます。

```
2012-11-01T22:08:41+00:00 Line 1 of the first entry
Line 2 of the first entry
```
```
2012-11-01T22:08:43+00:00 Line 1 of the second entry
Line 2 of the second entry
Line 3 of the second entry
```
