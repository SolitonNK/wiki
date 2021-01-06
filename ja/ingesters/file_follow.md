# ファイルフォロワー

ファイル・フォロワー・インジェスターは、ローカル・ファイルシステム上のファイルを、その場で更新される可能性がある状況でインジェストするための最良の方法です。ファイルの各行を一つのエントリとしてインジェストします。

ファイルフォロワーの最も一般的な使用例は、/var/logのような、活発に更新されているログファイルを含むディレクトリを監視することである。これはログのローテーションをインテリジェントに処理し、`logfile` が `logfile.1` に移動されたことなどを検出します。ディレクトリ内の特定のパターンにマッチするファイルをインジェストするように設定することができ、オプションでそのトップレベルのディレクトリのサブディレクトリに再帰的に降順することもできる。

注意 : RHEL/Centosでは、`/var/log`は "root "グループに属し、我々が想定している "adm "グループには属さない。File Followerはデフォルトではadmグループで動作するので、`/var/log`を読ませたい場合は、`chgrp -R adm /var/log`を実行するか、systemdユニットファイルのグループを変更する必要がある。


## 基本的な構成

ファイルフォロワー構成ファイルは、デフォルトでは Linux 上の `/opt/gravwell/etc/file_follow.conf` と Windows 上の `C:Program Files\gravwelfile_follow.cfg` にあります。

ファイルフォロワーインジェスターは、[インジェスターセクション](#!ingesters/ingesters.md#Global_Configuration_Parameters)で説明されている統一されたグローバルコンフィギュレーションブロックを使用します。 他のほとんどのGravwellインジェスターと同様に、ファイルフォロワーは複数のアップストリームインデクサー、TLS、クリアテキスト、名前付きパイプ接続、ローカルロギングをサポートしています。

注意 : ファイルフォロワーインジェスターでファイルキャッシュを使用することは強くお勧めしません。

ファイルフォロワーインジェスターの設定例は、/var/log にある複数の異なるタイプのログファイルを監視し、/tmp/incoming 以下のファイルを再帰的に追跡するように設定されています:

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

この例では、"syslog "フォロワーは `/var/log/syslog` とそのローテーションを読み、syslogタグに行をインジェストし、日付がローカルタイムゾーンにあると仮定します。同様に、"auth" フォロワーも `/var/log/auth.log` をインジェストするために syslog タグを使用します。packages」フォロワーは、Debian のパッケージ管理ログを dpkg タグに取り込みます。

最後に、「外部」フォロワーは `/tmp/incoming` ディレクトリから `.log` で終わるすべてのファイルを再帰的にディレクトリに降順して読み込みます。このフォロワーは、太平洋時間帯のタイムスタンプをあたかも太平洋時間帯であるかのように解析します。このフォロワーは、例えば米国西海岸にある複数のサーバが定期的にログファイルをこのシステムにアップロードしている場合に有用な設定を示しています。

上記で使用した設定パラメーターについては、以下のセクションで詳しく説明します。

## 追加のグローバルパラメーター

### Max-Files-Watched

Max-Files-Watchedパラメーターは、ファイルフォロワーがあまりにも多くのオープンファイルディスクリプタを維持することを防ぐ。Max-Files-Watched=64` が指定された場合、ファイルフォロワーは最大64個のログファイルをアクティブに監視する。新しいファイルが作成されると、ファイルフォロワーは新しいファイルを監視するために、最も古い既存のファイルの監視を停止する。しかし、古いファイルが後で更新された場合は、キューの先頭に戻ります。

ほとんどの場合、この設定は 64 のままにしておくことをお勧めします。

## フォロワー構成

ファイルフォロワー設定ファイルには、1つ以上の "フォロワー "ディレクティブが含まれています:

```
[Follower "syslog"]
        Base-Directory="/var/log/"
        File-Filter="syslog,syslog.[0-9]" #we are looking for all authorization log files
        Tag-Name=syslog
```

各フォロワーは、最低でもベースディレクトリとファイル名のフィルタリングパターンを指定します。このセクションでは、フォロワーごとに設定可能な設定パラメーターについて説明します。

###	基本ディレクトリ

Base-Directory パラメーターは、インジェストするファイルを格納するディレクトリを指定します。絶対パスでなければならず、ワイルドカードは含まれません。

### File-Filter

File-Filterパラメーターは、取り込むべきファイル名を定義します。それは、単一のファイル名のように単純なものである可能性があります:

```
File-Filter="foo.log"
```

または複数のパターンを含むことができます:

```
File-Filter="kern*.log,kern*.log.[0-9]"
```

これは、"kern "で始まり".log "で終わるファイル名、または "kern "で始まり".log.0 "から".log.9 "で終わるファイル名にマッチします。

完全なマッチング構文は、[https://golang.org/pkg/path/filepath/#Match](https://golang.org/pkg/path/filepath/#Match)で定義されています:

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

### Tag-Name

Tag-Nameパラメーターは、このフォロワーが摂取したエントリに適用するタグを指定します。

### Ignore-Timestamps

Ignore-Timestamps パラメーターは、フォロワーがファイルの各行からタイムスタンプを抽出しようとするのではなく、各行に現在の時刻をタグ付けすることを示します。

### Assume-Local-Timezone

Assume-Local-Timezoneは、デフォルトのUTCではなくローカルのタイムゾーンにあるかのようにタイムゾーンの指定がないタイムスタンプをパースするようにインジェスターに指示するブール値の設定です。
assumee-Local-TimezoneとTime-Overrideは相互に排他的です。

### Timezone-Override

Time-Zone-Overrideパラメーターは、タイムゾーンの指定がないタイムスタンプを、デフォルトのUTCではなく、与えられたタイムゾーンにあるかのように解析するようにインジェスターに指示します。タイムゾーンは、[https://en.wikipedia.org/wiki/List_of_tz_database_time_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)で示されているように、IANAデータベースの文字列形式で指定する必要があります:


```
Timezone-Override="America/Chicago"
```

Local-Timezoneの仮定とTime-Overrideは相互に排他的である。`Time-Zone-Override="Local"` は、関数的には `Assume-Local-Timezone=true` と同等である。

### Recursive

recursive パラメーターは、File Follower に、File-Filter にマッチするファイルを Base-Directory の下で再帰的にインジェストするように指示します。

デフォルトでは、 インジェスターは、Base-Directory の最上位レベルの下の File-Filter にマッチするファイルのみをインジェストします:

```
Base-Directory="/tmp/incoming"
File-Filter="foo.log"
Recursive=false
```

Recusive=true に設定すると、設定は `/tmp/incoming` 以下の任意のディレクトリの深さにある foo.log という名前の**全ての**ファイルをインジェストします。

### Ignore-Line-Prefix

インジェスターは、Ignore-Line-Prefix に渡された文字列で始まる行をすべて削除します（インジェストしません）。これは、Bro ログのようなコメントを含むログファイルをインジェストする場合に便利です。Ignore-Line-Prefixパラメーターは複数回指定することができます。

以下は、`#` や `//` で始まる行をインジェストすべきではないことを示しています:

```
Ignore-Line-Prefix="#"
Ignore-Line-Prefix="//"
```

### Timestamp-Format-Override

データ値には複数のタイムスタンプが含まれている場合があり、データからタイムスタンプを導出しようとするときに混乱を引き起こす可能性があります。 通常、フォロワーは導出可能な左端のタイムスタンプを取得しますが、特定のフォーマットのタイムスタンプだけを探すことが望ましい場合もあります。 "Timestamp-Format-Override "は、特定のフォーマットのタイムスタンプのみを尊重するようにフォロワーに指示します。 以下のタイムスタンプフォーマットが利用可能です:

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

RFC3339仕様にマッチしたタイムスタンプのみをフォロワーに表示させるには、 ```Timestamp-Format-Override=RFC3339``` をフォロワーに追加してください。

### Timestamp-Delimited

Timestamp-Delimitedパラメーターは、タイムスタンプの各出現を新しいエントリの開始とみなすことを指定するブール値です。これは、ログエントリが複数の行にまたがる場合に便利です。Timestamp-Delimitedを指定する場合、Timestamp-Format-Overrideパラメーターも設定しなければなりません。

### Timestamp-Regex and Timestamp-Format-String

ログファイルが以下のようになっている場合:


```
2012-11-01T22:08:41+00:00 Line 1 of the first entry
Line 2 of the first entry
2012-11-01T22:08:43+00:00 Line 1 of the second entry
Line 2 of the second entry
Line 3 of the second entry
```
フォロワーが `Timestamp-Delimited=true` と `Timestamp-Format-Override=RFC3339` で設定されている場合、以下の2つのエントリを生成します:

```
2012-11-01T22:08:41+00:00 Line 1 of the first entry
Line 2 of the first entry
```
```
2012-11-01T22:08:43+00:00 Line 1 of the second entry
Line 2 of the second entry
Line 3 of the second entry
```