# Ingesters設定

このセクションには、Gravwellインジェスターを構成および実行するための詳細な手順が含まれています。

Gravwellが作成したインジェスターはBSDオープンソースライセンスの下でリリースされており、[Github](https://github.com/gravwell/ingesters)にあります。 インジェストAPIはオープンソースでもあるため、独自のデータソース用に独自のインジェスターを作成したり、追加の正規化や前処理を実行したり、その他の方法で実行することができます。 INGEST APIコードは[ここにあります](https://github.com/gravwell/ingest)。

一般に、インジェスターがGravwellにデータを送信するには、インジェスターは認証のためにGravwellインスタンスの「Ingest Secret」を知っている必要があります。これ/opt/gravwell/etc/gravwell.confはGravwellサーバー上のファイルを表示してのエントリを見つけることで見つけることができますIngest-Auth。インジェスターがGravwell自体と同じシステム上で実行されている場合、インストーラーは通常この値を検出して自動的に設定することができます。

Gravwell GUIにはIngestersページ（Systemメニューカテゴリの下）があり、どのリモートインジェスタがアクティブに接続されているか、接続されている期間、およびプッシュされたデータ量を簡単に識別できます。

![](remote-ingesters.png)

<span style="color: red;">重要：複製システムは999MBを超えるエントリーを複製しません。より大きいエントリは通常どおりに取り込まれて検索されますが、複製から除外されます。このページで詳述されているすべてのインジェスターが数キロバイト以下のエントリを作成する傾向があるため、これは99.9％のユースケースには関係ありません。</span>

## グローバル設定パラメータ

コアインジェスターのほとんどは、共通のグローバル構成パラメーターのセットをサポートしています。共有グローバル設定パラメータは、[ingest設定](https://godoc.org/github.com/gravwell/ingest/config#IngestConfig)パッケージを使って実装されます。グローバル設定パラメータは、各Gravwell ingester設定ファイルのGlobalセクションで指定する必要があります。以下のグローバルインジェスタパラメータが利用可能です。


* Ingest-Secret
* Connection-Timeout
* Insecure-Skip-TLS-Verify
* Cleartext-Backend-Target
* Encrypted-Backend-Target
* Pipe-Backend-Target
* Ingest-Cache-Path
* Max-Ingest-Cache
* Log-Level
* Log-File
* Source-Override

### Ingest-Secret

Ingest-Secretパラメーターは、取り込み認証に使用されるトークンを指定します。ここで指定されたトークンは、GravwellインデクサーのIngest-Authパラメーターと一致しなければなりません。

### Connection-Timeout

Connection-Timeoutパラメーターは、あきらめる前にインデクサーに接続するのを待つ時間を指定します。空のタイムアウトは、インジェスターが起動するまで永遠に待機することを意味します。タイムアウトは、分、秒、または時間単位で指定する必要があります。

#### 例
```
Connection-Timeout=30s
Connection-Timeout=5m
Connection-Timeout=1h
```

### Insecure-Skip-TLS-Verify

Insecure-Skip-TLS-Verifyトークンは、暗号化されたTLSトンネルを介して接続するときに不正な証明書を無視するように取り込み側に指示します。その名前が示すように、TLSによって提供されるすべての認証はウィンドウから放り出され、攻撃者は中間者TLS接続を簡単に行うことができます。取り込み接続は依然として暗号化されますが、接続は決して安全ではありません。デフォルトではTLS証明書が検証され、証明書の検証に失敗すると接続は失敗します。

#### 例
```
Insecure-Skip-TLS-Verify=true
Insecure-Skip-TLS-Verify=false
```

### Cleartext-Backend-Target

Cleartext-Backend-Targetは、Gravwellインデクサーのホストとポートを指定します。インジェスターは、平文のTCP接続を使用してインデクサーに接続します。ポートが指定されていない場合は、デフォルトのポート4023が使用されます。平文接続は、IPv6とIPv4の両方の宛先をサポートします。 複数のクリアテキスト - バックエンド - ターゲットを指定して、複数のインデクサーにわたってインジェスターをロードバランスすることができます。

#### 例
```
Cleartext-Backend-Target=192.168.1.1
Cleartext-Backend-Target=192.168.1.1:4023
Cleartext-Backend-Target=DEAD::BEEF
Cleartext-Backend-Target=[DEAD::BEEF]:4023
```

### Encrypted-Backend-Target

Encrypted-Backend-Targetは、Gravwellインデクサーのホストとポートを指定します。インジェスターはTCPを介してインデクサーに接続し、完全なTLSハンドシェイク/証明書検証を実行します。ポートが指定されていない場合は、デフォルトポートの4024が使用されます。暗号化接続はIPv6とIPv4の両方の宛先をサポートします。 複数のインデクサー間でインジェスターの負荷を分散するために、複数の暗号化バックエンドターゲットを指定できます。

#### 例
```
Encrypted-Backend-Target=192.168.1.1
Encrypted-Backend-Target=192.168.1.1:4023
Encrypted-Backend-Target=DEAD::BEEF
Encrypted-Backend-Target=[DEAD::BEEF]:4023
```

### Pipe-Backend-Target

Pip-Backend-Targetは、フルパスでUnixという名前のsocketを指定します。Unix名前付きソケットは、インデクサーと非常に高速でオーバーヘッドがほとんどないため、インデクサーと共存するインジェスターにとって理想的です。インジェスタごとにサポートされるPipe-Backend-Targetは1つだけですが、クリアテキストおよび暗号化された接続と共にパイプを多重化することができます。

#### 例
```
Pipe-Backend-Target=/opt/gravwell/comms/pipe
Pipe-Backend-Target=/tmp/gravwellpipe
```

### Ingest-Cache-Path

Ingest-Cache-Pathは、取り込まれたデータのローカルキャッシュを有効にします。有効にすると、インジェスタはエントリをインデクサに転送できないときにローカルにキャッシュできます。取り込みキャッシュは、リンクがダウンしたときや、Gravwellクラスターを一時的にオフラインにする必要があるときに、データが失われないようにするのに役立ちます。長期間のネットワーク障害によってインジェスターがホストディスクをいっぱいにしないように、必ずMax-Ingest-Cache値を指定してください。ローカルインジェストキャッシュは、インデクサーに直接インジェストするほど高速ではないため、インジェストキャッシュが1秒間に200万件のエントリーをインデクサーで処理できるとは思わないでください。

<span style="color: red; ">重要：File Followerインジェスターではインジェストキャッシュを有効にしないでください。このインジェスターはディスク上のファイルから直接読み取り、各ファイル内の位置を追跡するため、キャッシュは不要です。</span>

#### 例
```
Ingest-Cache-Path=/opt/gravwell/cache/simplerelay.cache
Ingest-Cache-Path=/mnt/storage/networklog.cache
```

### Max-Ingest-Cache

Max-Ingest-Cacheは、キャッシュが使用されているときにインジェスタが消費するストレージ容量を制限します。最大キャッシュ値はメガバイト単位で指定されます。1024の値は、インジェスタが新しいエントリの受け入れを停止するまでに1 GBのストレージを消費できることを意味します。キャッシュがいっぱいになっても、キャッシュシステムは古いエントリを上書きしません。これは仕様によるもので、攻撃者がネットワーク接続を中断して、中断が発生した時点で潜在的に重要なデータをインジェスターに上書きさせることはできません。

#### 例
```
Max-Ingest-Cache=32
Max-Ingest-Cache=1024
Max-Ingest-Cache=10240
```

### Log-File

インジェスターは、インストールおよび構成の問題のデバッグに役立つように、エラーおよびデバッグ情報をログファイルに記録できます。Log-Fileパラメータが空の場合、ファイルロギングは無効になります。

#### 例
```
Log-File=/opt/gravwell/log/ingester.log
```

### Log-Level

Log-Levelパラメータは、ログファイルと、 "gravwell"タグの下でインデクサに送信されるメタデータの両方について、各インジェスタのログ記録システムを制御します。ログレベルをINFOに設定すると、ファイルフォロアが新しいファイルをフォローしたり、シンプルリレーが新しいTCP接続を受信した場合など、詳細にログインするようにインジェスタに指示します。一方、レベルをERRORに設定すると、最も重大なエラーのみがログに記録されます。ほとんどの場合、WARNレベルは適切です。以下のレベルがサポートされています。

* OFF
* INFO
* WARN
* ERROR

#### 例
```
Log-Level=Off
Log-Level=INFO
Log-Level=WARN
Log-Level=ERROR
```

### Source-Override

Source-Overrideパラメータは、各エントリに添付されているSRCデータ項目を上書きします。SRC項目はIPv6またはIPv4アドレスのいずれかであり、通常はインジェスターが実行されているマシンの外部IPアドレスです。

#### 例
```
Source-Override=10.0.0.1
Source-Override=0.0.0.0
Source-Override=DEAD:BEEF::FEED:FEBE
```

## シンプルリレー

[完全な構成とドキュメント](#!ingesters/simple_relay.md).

シンプルリレーは、複数のTCPまたはUDPポートでリッスンできるテキストingesterです。 各ポートには、タグと取り込み標準（RFC5424の解析や単純な改行区切りのエントリなど）を割り当てることができます。 シンプルリレーは、リモートsyslogエントリを取得したり、ネットワーク接続を介してテキストログを投げることができるデータソースから消費したりするための重要な手段です。

### 設定

Gravwell Debianリポジトリを使用している場合、インストールはただ1つのaptコマンドです。

```
apt-get install gravwell-simple-relay
```

それ以外の場合は、[ダウンロードページ](#!quickstart/downloads.md)からインストーラーをダウンロードします。 スーパーユーザーとして次のコマンドを発行し（例： `sudo`コマンドを使用）、ingesterをインストールします。

```
root@gravserver ~ # bash gravwell_simple_relay_installer.sh
```

Gravwellサービスが同じマシンに存在する場合、インストールスクリプトは自動的に `Ingest-Auth`パラメーターを抽出して設定し、適切に設定します。 ただし、ご使用のコンピューターが既存のGravwellバックエンドと同じマシンに常駐していない場合、インストーラーは認証トークンとGravwellインデクサーのIPアドレスの入力を求めます。 インストール時にこれらの値を設定するか、空白のままにして、 `/opt/gravwell/etc/simple_relay.conf`の設定ファイルを手動で変更できます。

## ファイルフォロワー

File Follower ingesterは、ローカルシステム上のファイルを監視し、Gravwellとネイティブに統合できないソースまたはネットワーク接続を介してログを送信できないソースからログをキャプチャするように設計されています。 ファイルフォロワーは、LinuxとWindowsの両方のフレーバーで提供され、任意の行区切りテキストファイルを追跡できます。 ファイルのローテーションと互換性があり、強力なパターンマッチングシステムを使用して、ログファイルに一貫性のない名前を付ける可能性のあるアプリケーションを処理します。

### 設定

Gravwell Debianリポジトリを使用している場合、インストールはただ1つのaptコマンドです。

```
apt-get install gravwell-file-follow
```

それ以外の場合は、[ダウンロードページ](＃！quickstart / downloads.md)からインストーラーをダウンロードします。 Windowsシステムで、ダウンロードした実行可能ファイルを実行し、インストーラーのプロンプトに従います。 Linuxでは、スーパーユーザーとして次のコマンドを発行して（たとえば、 `sudo`コマンドを使用して）、ingesterをインストールします：

```
root@gravserver ~ # bash gravwell_file_follow_installer.sh
```

Gravwellサービスが同じマシンに存在する場合、インストールスクリプトは自動的にIngest-Authパラメータを抽出して設定し、それを適切に設定します。ただし、インジェスターが既存のGravwellバックエンドと同じマシンに常駐していない場合、インストーラーは認証トークンとGravwellインデクサーのIPアドレスを要求します。インストール中にこれらの値を設定することも、空白のままにして/opt/gravwell/etc/file_follow.conf内の構成ファイルを手動で変更することもできます。

### 設定例

ファイルフォロワーの構成は、WindowsとLinuxの両方でほぼ同じです。 より詳細な構成情報[ファイルフロー](file_follow.md)が利用可能です。

#### Windows

Windows構成ファイルは、デフォルトでC：¥Program Files¥gravwell¥file_follow.cfgにあります。 Windows File FollowerはWindowsサービスとして動作します。その状況は、コマンドプロンプトでsc query GravwellFileFollowを発行することによって照会できます。 Windows CBSログファイルを追跡する設定例はこのようになります：

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify = false
Cleartext-Backend-target=172.20.0.2:4023 #example of adding a cleartext connection
State-Store-Location="c:\\Program Files\\gravwell\\file_follow.state"
Ingest-Cache-Path="c:\\Program Files\\gravwell\\file_follow.cache"
Log-Level=ERROR #options are OFF INFO WARN ERROR
#basic default logger, all entries will go to the default tag
#no Tag-Name means use the default tag
[Follower "cbs"]
        Base-Directory="C:\\Windows\\Logs\\CBS"
        File-Filter="CBS.log" #we are looking for just the CBS log
        Tag-Name=auth
        Assume-Local-Timezone=true
```

#### Linux

Linux設定ファイルはデフォルトで/opt/gravwell/etc/file_follow.confにあります。 kernel、dmesg、およびdebianのインストールログを監視する設定例は次のようになります。

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify = false
Cleartext-Backend-target=172.20.0.1:4023 #example of adding a cleartext connection
Cleartext-Backend-target=172.20.0.2:4023 #example of adding another cleartext connection
#Encrypted-Backend-target=127.1.1.1:4024 #example of adding an encrypted connection
#Pipe-Backend-target=/opt/gravwell/comms/pipe #a named pipe connection, this should be used when ingester is on the same machine as a backend
State-Store-Location=/opt/gravwell/etc/file_follow.state
Log-Level=ERROR #options are OFF INFO WARN ERROR
Max-Files-Watched=64
#Ingest-Cache-Path=/opt/gravwell/cache/file_follow.cache #allows for ingested entries to be cached when indexer is not available
#basic default logger, all entries will go to the default tag
#no Tag-Name means use the default tag
[Follower "syslog"]
        Base-Directory="/var/log/"
        File-Filter="syslog,syslog.[0-9]" #we are looking for all authorization log files
        Tag-Name=syslog
        Assume-Local-Timezone=true #Default for assume localtime is false
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
[Follower "kernel"]
        Base-Directory="/var/log"
        File-Filter="dmesg,dmesg.[0-9]"
        Tag-Name=kernel
        Ignore-Timestamps=true
[Follower "kernel2"]
        Base-Directory="/var/log"
        File-Filter="kern.log,kern.log.[0-9]"
        Tag-Name=kernel
        Ignore-Timestamps=true
```

## HTTP

HTTPインジェスターは、1つ以上のパスにHTTPリスナーを設定します。 HTTPリクエストがこれらのパスの1つに送信されると、リクエストのBodyは単一のエントリとして取り込まれます。

curlコマンドを使用すると、標準入力を本文として使用してPOST要求を簡単に実行できるため、これはスクリプト可能なデータ取り込みに非常に便利な方法です。

### 設定

Gravwell Debianリポジトリを使用している場合、インストールはただ1つのaptコマンドです:

```
apt-get install gravwell-http-ingester
```

それ以外の場合は、[ダウンロードページ](#!quickstart/downloads.md)からインストーラーをダウンロードします。 Gravwellサーバー上のターミナルを使用して、次のコマンドをスーパーユーザーとして（たとえば、「sudo」コマンド経由で）発行して、ingesterをインストールします。

```
root@gravserver ~ # bash gravwell_http_ingester_installer_3.0.0.sh
```

Gravwellサービスが同じマシンに存在する場合、インストールスクリプトは自動的にIngest-Authパラメータを抽出して設定し、それを適切に設定します。 ただし、インジェスターが既存のGravwellバックエンドと同じマシンに常駐していない場合、インストーラーは認証トークンとGravwellインデクサーのIPアドレスを要求します。 インストール中にこれらの値を設定することも、空白のままにして/opt/gravwell/etc/gravwell_http_ingester.conf内の構成ファイルを手動で変更することもできます。

### 設定例

すべてのインジェスターが使用するユニバーサル構成パラメーターに加えて、HTTP POSTインジェスターには、組み込みWebサーバーの動作を制御する2つの追加のグローバル構成パラメーターがあります。 最初の設定パラメータはBindオプションで、これはWebサーバが待機するインタフェースとポートを指定します。 2番目はMax-Bodyパラメータです。これはWebサーバが許可するPOSTの大きさを制御します。 Max-Bodyパラメータは、不正なプロセスが巨大なファイルを1つのエントリとしてGravwellインスタンスにアップロードしようとするのを防ぐための優れたセーフティネットです。 Gravwellは1エントリとして最大2GBまでサポートできますが、お勧めできません。

特定のURLが特定のタグにエントリを送信できるように、複数の「リスナー」定義を定義できます。 この設定例では、天気予報IOTデバイスとスマートサーモスタットからデータを受け取る2つのリスナーを定義します。

```
[Listener "weather"]
	URL="/weather"
	Tag-Name=weather


[Listener "thermostat"]
	URL="/smarthome/thermostat"
	Tag-Name=thermostat
```

"/ weather"または "/ smarthome / thermostat"に送信されたPOSTリクエストの本文に送信されたデータは、それぞれ "weather"および "thermostat"タグでタグ付けされます。 現在のタイムスタンプはPOSTの時点で各エントリに添付されます。

リスナーが単純なcurlコマンドで動作していることをテストできます。

```
curl -d "its hot outside bro" -X POST http://10.0.0.1:8080/weather
```

OpenWeatherMap.orgのAPIキーがある場合は、次のようなコマンドを使用して、気象条件を自動的に解除してGravwellにプッシュするようにcronジョブを設定できます。

```
curl "http://api.openweathermap.org/data/2.5/weather?q=Spokane&APPID=YOUR_APP_ID" | curl http://10.0.0.1:8088/weather -X POST -d @-
```

## 大量ファイルインジェスター

Mass File ingesterは、多くのソースから多数のログのアーカイブを取り込むための非常に強力ですが、専用のツールです。

### ユースケースの例
Gravwellユーザーは、潜在的なネットワーク侵害を調査するときにこのツールを使用しています。 ユーザーは50を超えるさまざまなサーバーからApacheログを取得しており、それらすべてを検索する必要がありました。 それらを次々に摂取すると、一時的なインデックス作成のパフォーマンスが低下します。 このツールは、ログエントリの一時的な性質を維持し、確実なパフォーマンスを確保しながら、ファイルを取り込むために作成されました。 マスファイルingesterは、取り込み前にソースログを最適化するために取り込みマシンに十分なスペース（ストレージとメモリ）がある場合に最適に機能します。 最適化フェーズは、取り込み時および検索時のGravwellストレージシステムへの圧力を軽減するのに役立ち、インシデントレスポンダーが迅速に移動して、ログデータへのパフォーマンスの高いアクセスを短時間で取得できるようにします。

### ノート

大量ファイルingesterはコマンドラインパラメーターを介して駆動され、サービスとして実行するようには設計されていません。 コードは[Github](https://github.com/gravwell/ingesters)で入手できます。

```
Usage of ./massFile:
  -clear-conns string
        comma seperated server:port list of cleartext targets
  -ingest-secret string
        Ingest key (default "IngestSecrets")
  -no-ingest
        Optimize logs but do not perform ingest
  -pipe-conn string
        Path to pipe connection
  -s string
        Source directory containing log files
  -skip-op
        Assume working directory already has optimized logs
  -tag-name string
        Tag name for ingested data (default "default")
  -timeout int
        Connection timeout in seconds (default 1)
  -tls-conns string
        comma seperated server:port list of TLS connections
  -tls-private-key string
        Path to TLS private key
  -tls-public-key string
        Path to TLS public key
  -tls-remote-verify string
        Path to remote public key to verify against
  -w string
        Working directory for optimization
```

## Windowsイベントサービス

Gravwell Windowsイベントingesterは、Windowsマシン上のサービスとして実行され、WindowsイベントをGravwellインデクサーに送信します。

### 設定

[ダウンロードページ](＃！quickstart / downloads.md)からGravwell Windows ingesterインストーラーをダウンロードします。

.msiインストールウィザードを実行してGravwellイベントサービスをインストールします。

ウィザードの将来のバージョンではGravwell構成オプションを直接入力するよう求められますが、現時点では、 `C:\Program Files\gravwell\config.cfg`にある構成ファイルを手動で構成する必要があります。

接続IPアドレスをGravwellサーバーのIPに変更し、Ingest-Secret値を設定します。

```
Ingest-Secret=YourSecretGoesHere
Encrypted-Backend-target=ip.addr.goes.here:port
```

構成が完了すると、このファイルは、イベントを収集する他のWindowsシステムにコピーできます。

### オプションのSysmon統合

sysinternalsスイートの一部であるSysmonユーティリティは、Windowsシステムを監視するための効果的で一般的なツールです。 適切なsysmon構成ファイルの例については、多くのリソースがあります。 Gravwellでは、infosec Twitterパーソナリティ@InfosecTaylorSwiftによって作成された構成を使用しています。

`C:\Program Files\gravwell\config.cfg`にあるGravwell Windowsエージェント設定ファイルを編集し、次の行を追加します。

```
[EventChannel "Sysmon"]
        Tag-Name=sysmon #Apply a new tag name
        Provider=Microsoft-Windows-Sysmon #Only look for the provider
        Channel=Microsoft-Windows-Sysmon/Operational
```

[SwiftOnSecurityによる優れたsysmon構成ファイルをダウンロードします](https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml)

[sysmonをダウンロードする](https://technet.microsoft.com/en-us/sysinternals/sysmon)

sysmonとその設定を入れます `C:\Program Files\gravwell`

管理者のPowerShellで実行：

```
sysmon.exe -accepteula -i sysmonconfig-export.xml
```

標準のWindowsサービス管理を介してGravwellサービスを再起動します。

#### Sysmonを使用した構成例

```
[EventChannel "system"]
        Tag-Name=windows
        #no Provider means accept from all providers
        #no EventID means accept all event ids
        #no Level means pull all levels
        #no Max-Reachback means look for logs starting from now
        Channel=System #pull from the system channel
[EventChannel "application"]
        Tag-Name=windows
        Channel=Application #pull from the system channel
[EventChannel "security"]
        Tag-Name=windows
        Channel=Security #pull from the system channel
[EventChannel "setup"]
        Tag-Name=windows
        Channel=Setup #pull from the system channel
[EventChannel "sysmon"]
        Tag-Name=windows
        Provider=Microsoft-Windows-Sysmon #Only look for the provider
        Channel=Microsoft-Windows-Sysmon/Operational
```

### トラブルシューティング

Webインターフェイスの[Ingester]ページに移動して、Windows ingesterの接続を確認できます。 Windows ingesterが存在しない場合は、Windows GUIを使用するか、コマンドラインで「sc query GravwellEvents」を実行して、サービスのステータスを確認します。

![](querystatus.png)

![](querystatusgui.png)

### Windows検索の例

デフォルトのタグ名が使用されていると仮定して、すべてのsysmonエントリ全体を表示するには、次の検索を実行します。

```
tag=sysmon
```

すべてのWindowsイベントをすべて実行するには、次を実行します。

```
tag=windows
```

次の検索では、Windowsの結果を取得し、それらを正規表現検証ツール[regex101.com](regex101.com)に投げて、正規表現を作成しました。 `（<？P <foo>。*）`スタイルの正規表現を使用して「名前を付ける」ものはすべて、 `| fooでカウント| foo`によるチャートカウント。 正規表現の抽出の詳細については、検索モジュールに関するドキュメントを参照してください。

非標準プロセスによるすべてのネットワーク作成を表示するには：

```
tag=sysmon regex ".*EventID>3.*'Image'>(?P<exe>\S*)<\/Data>.*SourceHostname'>(?P<src>\S*)<\/Data>.*DestinationIp'>(?P<dst>[0-9]+.[0-9]+.[0-9]+.[0-9]+).*DestinationPort'>(?P<dport>[0-9]*)"
```

送信元ホストごとにネットワーク作成をグラフ化するには：

```
tag=sysmon regex ".*EventID>3.*'Image'>(?P<exe>\S*)<\/Data>.*SourceHostname'>(?P<src>\S*)<\/Data>.*DestinationIp'>(?P<dst>[0-9]+.[0-9]+.[0-9]+.[0-9]+).*DestinationPort'>(?P<dport>[0-9]*)" | count by src | chart count by src limit 10
```

疑わしいファイルの作成を確認するには：

```
tag=sysmon regex ".*EventID>11.*Image'>(?P<process>.*)<\/Data>.*TargetFilename'>(?P<file>[\-:\.\ _\a-zA-z]*)<\/Data><Data Name='"
```
```
tag=sysmon regex ".*EventID>11.*Image'>(?P<process>.*)<\/Data>.*TargetFilename'>(?P<file>[\-:\.\ _\a-zA-z]*)<\/Data><Data Name='" | count by file | chart count by file
```

## Netflow Ingester

Netflowインジェスターは、Netflowコレクターとして機能し（Netflowロールの詳細については、[Wikipediaの記事](https://en.wikipedia.org/wiki/NetFlow) を参照）、Netflowエクスポーターによって作成されたレコードを収集し、後で分析するためにGravwellエントリーとしてそれらをキャプチャーします。 これらのエントリは、[netflow](#!search/netflow/netflow.md)検索モジュールを使用して分析できます。

Gravwell Debianリポジトリを使用している場合、インストールは単なるaptコマンドです。

```
apt-get install gravwell-netflow-capture
```

それ以外の場合は、[ダウンロードページ](#!quickstart/downloads.md)からインストーラをダウンロードしてください。 Netflowインジェスタをインストールするには、単にインストーラをrootとして実行します（実際のファイル名には通常、バージョン番号が含まれます）。

```
root@gravserver ~ # bash gravwell_netflow_capture_installer.sh
```

ローカルマシンにGravwellインデクサーがない場合、インストーラーはインジェクターシークレット値とインデクサー（またはフェデレーター）のIPアドレスの入力を求めます。 それ以外の場合は、既存のGravwell構成から適切な値を取得します。 いずれにせよ、インストール後に `/opt/gravwell/etc/netflow_capture.conf`の設定ファイルを確認してください。 UDPポート2055でリッスンする簡単な例は、次のようになります。

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify=false
Pipe-Backend-target=/opt/gravwell/comms/pipe #a named pipe connection, this should be used when ingester is on the same machine as a backend
Log-Level=INFO

[Collector "netflow v5"]
	Bind-String="0.0.0.0:2055" #we are binding to all interfaces
	Tag-Name=netflow
```

この設定は、 `/opt/gravwell/comms/pipe`を介してローカルインデクサーにエントリを送信することに注意してください。 エントリには「netflow」というタグが付けられます。

異なるポートで異なるタグをリッスンする「Collector」エントリをいくつでも設定できます。 これにより、データをより明確に整理できます。

注：現時点では、ingesterはNetflow v5のみをサポートしています。 Netflowエクスポーターを構成するときは、このことに留意してください。

## Network Ingester

Gravwellの主な強みは、バイナリデータを取り込む機能です。 ネットワークingesterを使用すると、後で分析するためにネットワークから完全なパケットをキャプチャできます。 これは、単にネットフローまたは他の凝縮されたトラフィック情報を保存するよりもはるかに優れた柔軟性を提供します。

Gravwell Debianリポジトリを使用している場合、インストールはただ1つのaptコマンドです。

```
apt-get install libpcap0.8 gravwell-network-capture
```

それ以外の場合は、[ダウンロードページ](#!quickstart/downloads.md)からインストーラーをダウンロードします。 ネットワークingesterをインストールするには、rootとしてインストーラーを実行するだけです（ファイル名は若干異なる場合があります）。

```
root@gravserver ~ # bash gravwell_network_capture_installer.sh
```

注：ingesterが機能するには、libpcapがインストールされている必要があります。

ネットワークingesterを可能な場合はインデクサーと同じ場所に配置し、データを送信するために「clear-conn」または「tls-conn」リンクではなく、「pipe-conn」リンクを使用することを強くお勧めします。 ネットワークingesterがエントリをプッシュするために使用しているのと同じリンクからキャプチャしている場合、リンクを急速に飽和させるフィードバックループを作成できます（たとえば、eth0からキャプチャし、eth0を介してingesterにエントリを送信します）。 これを軽減するには、 `BPF-Filter`オプションを使用できます。

Gravwellバックエンドがすでにインストールされているマシン上で感染している場合、インストーラーは自動的に正しい「Ingest-Secrets」値を取得し、それを設定ファイルに追加する必要があります。 それ以外の場合、インデクサーのIPアドレスとインジェストシークレットの入力を求められます。 いずれにしても、実行する前に `/opt/gravwell/etc/network_capture.conf`の設定ファイルを確認してください。 eth0からトラフィックをキャプチャする例は次のようになります。
```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify = false
#Cleartext-Backend-target=127.1.0.1:4023 #example of adding a cleartext connection
#Encrypted-Backend-target=127.1.1.1:4023 #example of adding an encrypted connection
Pipe-Backend-target=/opt/gravwell/comms/pipe #a named pipe connection, this should be used when ingester is on the same machine as a backend
Log-Level=INFO #options are OFF INFO WARN ERROR
Ingest-Cache-Path=/opt/gravwell/cache/network_capture.cache

#basic default logger, all entries will go to the default tag
#no Tag-Name means use the default tag
[Sniffer "spy1"]
	Interface="eth0" #sniffing from interface eth0
	Tag-Name="pcap"  #assigning tag  fo pcap
	Snap-Len=0xffff  #maximum capture size
	#BPF-Filter="not port 4023" #do not sniff any traffic on our backend connection
	#Promisc=true
```

異なるインターフェイスからキャプチャするために、任意の数の「Sniffer」エントリを設定できます。

ディスク容量が問題になる場合は、 `Snap-Len`パラメータを変更して、パケットメタデータのみをキャプチャすることができます。 通常、値96は、ヘッダーのみをキャプチャするのに十分です。

非常に高い帯域幅のリンクの可能性があるため、ネットワークキャプチャデータを独自のウェルに割り当てることをお勧めします。 これには、パケットキャプチャタグ用に別のウェルを定義するインデクサーの構成が必要です。

NetworkCapture ingesterは、libpcap構文に準拠する `BPF-Filter`パラメーターを使用したネイティブBPFフィルタリングもサポートします。 ポート22のすべてのトラフィックを無視するには、次のようなスニファーを構成できます。

```
[Sniffer "no-ssh"]
	Interface="eth0"
	Tag-Name="pcap"
	Snap-Len=0xffff
	BPF-Filter="not port 22"
```

ingesterがインデクサーとは異なるシステムにある場合、つまり、エントリを取り込むにはネットワークを横断する必要があるため、「BPF-Filter」を「not port 4023」（クリアテキストを使用する場合）または「not port 4024」（使用する場合） TLS）。

### ネットワーク検索の例

次の検索では、IP 10.0.0.0/24クラスCサブネットから発信されたRSTフラグが設定されたTCPパケットを探し、それらをIPごとにグラフ化します。 このクエリを使用して、ネットワークからの送信ポートスキャンを迅速に識別することができます。

```
tag=pcap packet tcp.RST==TRUE ipv4.SrcIP !~ 10.0.0.0/24 | count by SrcIP | chart count by SrcIP limit 10
```

![](portscan.png)

次の検索では、IPv6トラフィックを検索し、算術演算に渡されるFlowLabelを抽出します。 これにより、パケットの長さを合計してチャートレンダラーに渡すことで、フローごとのトラフィックアカウンティングが可能になります。

```
tag=pcap packet ipv6.Length ipv6.FlowLabel | sum Length by FlowLabel | chart sum by FlowLabel limit 10
```

TCPペイロードで使用されている言語を識別するために、ネットワークデータをフィルター処理して、langfindモジュールに渡すことができます。 このクエリは、アウトバウンドHTTPリクエストを探し、TCPペイロードデータをlangfindモジュールに渡します。langfindモジュールは、識別された言語をカウントしてからチャートに渡します。 これにより、アウトバウンドHTTPクエリで使用される人間の言語のチャートが生成されます。

```
tag=pcap packet ipv4.DstIP != 10.0.0.100 tcp.DstPort == 80 tcp.Payload | langfind -e Payload | count by lang | chart count by lang
```

![](langfind.png)

トラフィックアカウンティングは、レイヤー2でも実行できます。これは、イーサネットヘッダーからパケット長を抽出し、宛先MACアドレスで長さを合計し、トラフィックカウントでソートすることで実現されます。 これにより、特におしゃべりする可能性のあるイーサネットネットワーク上の物理デバイスを迅速に識別できます。
```
tag=pcap packet eth.DstMAC eth.Length > 0 | sum Length by DstMAC | sort by sum desc | table DstMAC sum
```

同様のクエリは、パケットカウントを介してチャットデバイスを識別できます。 たとえば、デバイスはスイッチに負荷をかけますが大量のトラフィックにはならない小さなイーサネットパケットを積極的にブロードキャストしている場合があります。

```
tag=pcap packet eth.DstMAC eth.Length > 0 | count by DstMAC | sort by count desc | table DstMAC count
```

非標準のHTTPポートで動作するHTTPトラフィックを識別することが望ましい場合があります。 これは、フィルタリングオプションを実行し、ペイロードを他のモジュールに渡すことで実現できます。 たとえば、TCPポート80ではなく、特定のサブネットから発信されているアウトバウンドトラフィックを検索し、ppacketペイロードでHTTP要求を検索すると、異常なHTTPトラフィックを特定できます。

```
tag=pcap packet ipv4.SrcIP ipv4.DstIP tcp.DstPort !=80 ipv4.SrcIP ~ 10.0.0.0/24 tcp.Payload | regex -e Payload "(?P<method>[A-Z]+)\s+(?P<url>[^\s]+)\s+HTTP/\d.\d" | table method url SrcIP DstIP DstPort
```

![](nonstandardhttp.png)

## 収集されたIngester

収集されたingesterは完全にスタンドアロンの [collectd](https://collectd.org/) 収集エージェントであり、収集されたサンプルをGravwellに直接出荷できます。 ingesterは、異なるタグ、セキュリティコントロール、およびプラグインからタグへのオーバーライドで構成できる複数のコレクターをサポートします。

Gravwell Debianリポジトリを使用している場合、インストールはただ1つのaptコマンドです。

```
apt-get install gravwell-collectd
```

それ以外の場合は、[ダウンロードページ](#!quickstart/downloads.md)からインストーラーをダウンロードします。 Gravwellサーバー上のターミナルを使用して、次のコマンドをスーパーユーザーとして（たとえば、「sudo」コマンド経由で）発行して、ingesterをインストールします。

```
root@gravserver ~ # bash gravwell_collectd_installer.sh
```

Gravwellサービスが同じマシンに存在する場合、インストールスクリプトは自動的に `Ingest-Auth`パラメーターを抽出して設定し、適切に設定します。 ただし、ご使用のコンピューターが既存のGravwellバックエンドと同じマシンに常駐していない場合、インストーラーは認証トークンとGravwellインデクサーのIPアドレスの入力を求めます。 インストール時にこれらの値を設定するか、空白のままにして、 `/opt/gravwell/etc/collectd.conf`の設定ファイルを手動で変更できます。
### 構成

収集されたingesterは、他のすべてのインジェスターと同じグローバル構成システムに依存しています。 Globalセクションは、インデクサー接続、認証、およびローカルキャッシュコントロールを定義するために使用されます。

コレクター構成ブロックは、収集されたサンプルを受け入れることができるリスニングコレクターを定義するために使用されます。 各コレクタ構成には、一意のセキュリティレベル、認証、タグ、ソースオーバーライド、ネットワークバインド、およびタグオーバーライドを設定できます。 複数のコレクター構成を使用して、単一の収集されたingesterは複数のインターフェースでリッスンし、複数のネットワーク飛び地から入ってくる収集されたサンプルに固有のタグを適用できます。

デフォルトでは、収集されたingesterは/opt/gravwell/etc/collectd.conf.にある構成ファイルを読み取ります。

#### 設定例

```
[Global]
	Ingest-Secret = SuperSecretKey
	Connection-Timeout = 0
	Cleartext-Backend-target=192.168.122.100:4023
	Log-Level=INFO

[Collector "default"]
	Bind-String=0.0.0.0:25826
	Tag-Name=collectd
	User=user
	Password=secret

[Collector "localhost"]
	Bind-String=[fe80::1]:25827
	Tag-Name=collectdlocal
	Security-Level=none
	Source-Override=[fe80::beef:1000]
	Tag-Plugin-Override=cpu:collectdcpu
```

#### コレクター構成オプション

コレクタ構成オプション各コレクタブロックには、一意の名前と重複しないバインド文字列が含まれている必要があります。 同じポート上の同じインターフェースにバインドされた複数のコレクターを持つことはできません。

##### Bind-String

Bind-Stringは、コレクターが着信収集サンプルをリッスンするために使用するアドレスとポートを制御します。 有効なバインド文字列には、IPv4またはIPv6アドレスとポートのいずれかが含まれている必要があります。 すべてのインターフェイスでリッスンするには、「0.0.0.0」ワイルドカードアドレスを使用します。

###### Bind-Stringの例
```
Bind-String=0.0.0.0:25826
Bind-String=127.0.0.1:25826
Bind-String=127.0.0.1:12345
Bind-String=[fe80::1]:25826
```

##### Tag-name

Tag-Nameは、Tag-Plugin-Overrideが適用されない限り、収集されたサンプルが割り当てられるタグを定義します。

##### Source-Override

Source-Overrideディレクティブは、エントリがGravwellに送信されるときにエントリに適用されるソース値をオーバーライドするために使用されます。 デフォルトでは、ingesterはingesterのSourceを適用しますが、検索時にセグメンテーションまたはフィルタリングを適用するには、特定のソース値をCollectorブロックに適用することが望ましい場合があります。 Source-Overrideは、有効なIPv4またはIPv6アドレスです。

##### Source-Overrideの例
```
Source-Override=192.168.1.1
Source-Override=[DEAD::BEEF]
Source-Override=[fe80::1:1]
```

##### Security-Level

Security-Levelディレクティブは、Collectorが収集したパケットを認証する方法を制御します。 利用可能なオプションは、暗号化、署名、なしです。 デフォルトでは、コレクタは「暗号化」セキュリティレベルを使用し、ユーザーとパスワードの両方を指定する必要があります。 「none」を使用する場合、ユーザーまたはパスワードは不要です。

##### Security-Levelの例
```
Security-Level=none
Security-Level=encrypt
Security-Level = sign
Security-Level = SIGN
```

##### ユーザーとパスワード

Security-Levelが「sign」または「encrypt」として設定されている場合、エンドポイントで設定された値と一致するユーザー名とパスワードを提供する必要があります。 デフォルト値は、collectdに付属のデフォルト値と一致する「user」および「secret」です。 これらの値は、収集されたデータに機密情報が含まれる可能性がある場合に変更する必要があります。

###### ユーザーとパスワードの例
```
User=username
Password=password
User = "username with spaces in it"
Password = "Password with spaces and other characters @$@#@()*$#W)("
```

##### エンコーダー

デフォルトの収集されたエンコーダーはJSONですが、シンプルなテキストエンコーダーも利用できます。 オプションは「JSON」または「テキスト」です

JSONエンコーダーを使用したエントリの例：

```
{"host":"build","plugin":"memory","type":"memory","type_instance":"used","value":727789568,"dsname":"value","time":"2018-07-10T16:37:47.034562831-06:00","interval":10000000000}
```

### Tag Plugin Overrides

各コレクターブロックは、N個のTag-Plugin-Override宣言をサポートします。これは、生成されたプラグインに基づいて、収集されたサンプルに一意のタグを適用するために使用されます。 Tag-Plugin-Overridesは、異なるプラグインからのデータを異なるウェルに保存し、異なるエージアウトルールを適用する場合に役立ちます。 たとえば、ディスク使用量に関する収集されたレコードを9か月間保存することは有益ですが、CPU使用量レコードは14日で期限切れになる場合があります。 Tag-Plugin-Overrideシステムはこれを簡単にします。

Tag-Plugin-Override形式は、「：」文字で区切られた2つの文字列で構成されています。 左側の文字列はプラグインの名前を表し、右側の文字列は目的のタグの名前を表します。 タグに関するすべての通常のルールが適用されます。 単一のプラグインを複数のタグにマッピングすることはできませんが、複数のプラグインを同じタグにマッピングできます。

#### Tag Plugin Overridesの例
```
Tag-Plugin-Override=cpu:collectdcpu # Map CPU plugin data to the "collectdcpu" tag.
Tag-Plugin-Override=memory:memstats # Map the memory plugin data to the "memstats" tag.
Tag-Plugin-Override= df : diskdata  # Map the df plugin data to the "diskdata" tag.
Tag-Plugin-Override = disk : diskdata  # Map the disk plugin data to the "diskdata" tag.
```

## Kinesis Ingester

Gravwellは、Amazonの[Kinesis Data Streams](https://aws.amazon.com/kinesis/data-streams/) サービスからエントリを取得できるingesterを提供します。 ingesterは一度に複数のKinesisストリームを処理でき、各ストリームは多くの個別のシャードで構成されます。 Kinesisストリームを設定するプロセスはこのドキュメントの範囲外ですが、既存のストリームにKinesis ingesterを設定するには、次のものが必要です。


* AWSアクセスキー（ID番号とシークレットキー）
* ストリームが存在する地域
* ストリーム自体の名前

ストリームが設定されると、Kinesisストリームの各レコードがGravwellに単一のエントリとして保存されます。

### インストールと構成

最初に、[ダウンロードページ](#!quickstart/downloads.md)からインストーラーをダウンロードしてから、ingesterをインストールします。

```
root@gravserver ~# bash gravwell_kinesis_ingest_installer.sh
```

Gravwellサービスが同じマシン上に存在する場合、インストールスクリプトは自動的に `Ingest-Auth`パラメーターを抽出して設定し、適切に設定する必要があります。 `/opt/gravwell/etc/kinesis_ingest.conf`設定ファイルを開き、Kinesisストリーム用に設定する必要があります。 以下で説明するように設定を変更したら、コマンド `systemctl start gravwell_kinesis_ingest.service`でサービスを開始します。

以下の例は、ローカルマシン上のインデクサーに接続し（ `Pipe-Backend-target`設定に注意してください）、us-west-1リージョンの「MyKinesisStreamName」という名前の単一Kinesisストリームからフィードするサンプル設定を示しています。

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify = false
Pipe-Backend-target=/opt/gravwell/comms/pipe #a named pipe connection, this should be used when ingester is on the same machine as a backend
Log-Level=ERROR #options are OFF INFO WARN ERROR

# This is the access key *ID* to access the AWS account
AWS-Access-Key-ID=REPLACEMEWITHYOURKEYID
# This is the secret key which is only displayed once, when the key is created
AWS-Secret-Access-Key=REPLACEMEWITHYOURKEY

[KinesisStream "stream1"]
	Region="us-west-1"
	Tag-Name=kinesis
	Stream-Name=MyKinesisStreamName	# should be the stream name as AWS knows it
	Iterator-Type=LATEST
	Parse-Time=false
	Assume-Localtime=true
```

ingesterを開始する前に、少なくとも以下のフィールドを設定する必要があります。

* `AWS-Access-Key-ID`-これは使用したいAWSアクセスキーのIDです
* `AWS-Secret-Access-Key`-これはシークレットアクセスキーそのものです
* `Region`-キネシスストリームが存在する領域
* `Stream-Name`-キネシスストリームの名前

複数の異なるKinesisストリームをサポートするために、複数の `KinesisStream`セクションを設定できます。

手動で `/opt/gravwell/bin/gravwell_kinesis_ingester -v`を実行して設定をテストできます。 エラーが出力されない場合、構成はおそらく受け入れられます。

ほとんどのフィールドは一目瞭然ですが、 `Iterator-Type`設定には注意する価値があります。 この設定は、ingesterがデータの読み取りを開始する場所を選択します。 TRIM_HORIZONに設定することにより、inesterは利用可能な最も古いレコードの読み取りを開始します。 LATESTに設定されている場合、ingesterは既存のすべてのレコードを無視し、ingesterの開始後に作成されたレコードのみを読み取ります。 ほとんどの場合、データの重複を避けるために、最新に設定する必要があります。 既存のデータを取り込みたい場合はTRIM_HORIZONに設定し、再起動する前に、ingesterをシャットダウンし、値をLATESTに変更します。

## GCP PubSubインジェスター

Gravwellは、Google Compute Platformの[PubSubストリーム](https://cloud.google.com/pubsub/) サービスからエントリを取得できるingesterを提供します。 ingesterは、単一のGCPプロジェクト内で複数のPubSubストリームを処理できます。 PubSubストリームを設定するプロセスはこのドキュメントの範囲外ですが、既存のストリームにPubSub ingesterを設定するには、次のものが必要です。

* GoogleプロジェクトID
* GCPサービスアカウント認証情報を含むファイル（[サービスアカウントの作成](https://cloud.google.com/docs/authentication/getting-started)ドキュメントを参照）
* PubSubトピックの名前

ストリームが設定されると、PubSubストリームトピックの各レコードがGravwellに単一のエントリとして保存されます。

### インストールと構成

最初に、[ダウンロードページ](#!quickstart/downloads.md)からインストーラーをダウンロードしてから、ingesterをインストールします。

```
root@gravserver ~# bash gravwell_pubsub_ingest_installer.sh
```

Gravwellサービスが同じマシン上に存在する場合、インストールスクリプトは自動的に `Ingest-Auth`パラメーターを抽出して設定し、適切に設定する必要があります。 `/opt/gravwell/etc/pubsub_ingest.conf`設定ファイルを開き、PubSubトピック用に設定する必要があります。 以下で説明するように設定を変更したら、コマンド `systemctl start gravwell_pubsub_ingest.service`でサービスを開始します

以下の例は、ローカルマシンのインデクサーに接続し（ `Pipe-Backend-target`設定に注意してください）、「myproject-127400」の一部である「mytopic」という名前の単一のPubSubトピックからフィードするサンプル設定を示しています 「GCPプロジェクト。

```
[Global]
Ingest-Secret = IngestSecrets
Connection-Timeout = 0
Insecure-Skip-TLS-Verify = false
Pipe-Backend-target=/opt/gravwell/comms/pipe #a named pipe connection, this should be used when ingester is on the same machine as a backend
Log-Level=ERROR #options are OFF INFO WARN ERROR

# The GCP project ID to use
Project-ID="myproject-127400"
Google-Credentials-Path=/opt/gravwell/etc/google-compute-credentials.json

[PubSub "gravwell"]
	Topic-Name=mytopic	# the pubsub topic you want to ingest
	Tag-Name=gcp
	Parse-Time=false
	Assume-Localtime=true
```

次の必須フィールドに注意してください。

* `Project-ID`-GCPプロジェクトのプロジェクトID文字列
* `Google-Credentials-Path`-JSON形式のGCPサービスアカウント認証情報を含むファイルへのパス
* `Topic-Name`-指定されたGCPプロジェクト内のPubSubトピックの名前

複数の `PubSub`セクションを設定して、単一のGCPプロジェクト内で複数の異なるPubSubトピックをサポートできます。

手動で `/opt/gravwell/bin/gravwell_pubsub_ingester -v`を実行することで設定をテストできます。 エラーが出力されない場合、構成はおそらく受け入れられます。

## ディスクモニター

diskmonitor ingesterは、ディスクアクティビティの定期的なサンプルを取得し、サンプルをgravwellに出荷するように設計されています。 ディスクモニタは、ストレージの遅延の問題を特定したり、ディスク障害を見つけたり、その他の潜在的なストレージの問題を特定するのに非常に役立ちます。 Gravwellでは、ディスクモニターを使用して独自のストレージインフラストラクチャを積極的に監視し、クエリの動作方法を調査し、ストレージインフラストラクチャの動作不良を特定しています。 RAIDコントローラーが診断ログで言及しなかった場合でも、レイテンシプロットを介してライトスルーモードに移行したRAIDアレイを特定することができました。

ディスクモニターのingesterは[github](https://github.com/gravwell/ingesters)で入手できます。

![diskmonitor](diskmonitor.png)

## Session Ingester

Session Ingesterは、より大きな単一のレコードを取り込むために使用される特殊なツールです。 ingesterは指定されたポートでリッスンし、クライアントから接続を受信すると、受信したデータを1つのエントリに集約します。

これにより、すべてのWindows実行可能ファイルのインデックス作成などの動作が可能になります。

```
for i in `ls /path/to/windows/exes`; do cat $i | nc 192.168.1.1 7777 ; done
```

セッションingesterは、永続的な構成ファイルではなく、コマンドラインパラメーターによって駆動されます。

```
Usage of ./session:
  -bind string
        Bind string specifying optional IP and port to listen on (default "0.0.0.0:7777")
  -clear-conns string
        comma seperated server:port list of cleartext targets
  -ingest-secret string
        Ingest key (default "IngestSecrets")
  -max-session-mb int
        Maximum MBs a single session will accept (default 8)
  -pipe-conns string
        comma seperated list of paths for named pie connection
  -tag-name string
        Tag name for ingested data (default "default")
  -timeout int
        Connection timeout in seconds (default 1)
  -tls-conns string
        comma seperated server:port list of TLS connections
  -tls-private-key string
        Path to TLS private key
  -tls-public-key string
        Path to TLS public key
  -tls-remote-verify string
        Path to remote public key to verify against
```

### ノート

セッションingesterは正式にサポートされておらず、インストーラーも利用できません。 セッションingesterのソースコードは[github](https://github.com/gravwell/ingesters)で入手できます。

## Gravwell Federator

Federatorはエントリリレーです。インジェスターはFederatorに接続してエントリを送信し、Federatorはそれらのエントリをインデクサーに渡します。 Federatorは信頼境界として機能し、インジェストシークレットを公開したり、信頼できないノードが許可されていないタグのデータを送信したりすることなく、ネットワークセグメント間でエントリを安全に中継します。 Federatorのアップストリーム接続は、他のすべてのingesterと同様に構成され、多重化、ローカルキャッシュ、暗号化などを可能にします。

![](federatorDiagram.png)

### ユースケース

*堅牢な接続性がない場合に地理的に多様な地域にデータを取り込む
  *ネットワークセグメント間に認証障壁を提供する
  *インデクサーへの接続数を減らす
  *データソースグループが提供できるタグの制御

### 設定

Gravwell Debianリポジトリを使用している場合、インストールはただ1つのaptコマンドです。

```
apt-get install gravwell-federator
```

それ以外の場合は、[ダウンロードページ](#!quickstart/downloads.md)からインストーラーをダウンロードします。 Gravwellサーバー上の端末を使用して、スーパーユーザーとして（たとえば、 `sudo`コマンドを使用して）次のコマンドを発行し、フェデレーターをインストールします。

```
root@gravserver ~ # bash gravwell_federator_installer.sh
```

Federatorは、ほぼ確実に特定のセットアップのための構成を必要とします。 詳細については、次のセクションを参照してください。 設定ファイルは `/opt/gravwell/etc/federator.conf`にあります。

### 設定例

次の設定例は、*保護された*ネットワークセグメントの2つのアップストリームインデクサーに接続し、2つの*信頼できない*ネットワークセグメントでインジェストサービスを提供します。 信頼されていない各インジェストポイントには、固有のIngest-Secretがあり、特定の証明書とキーペアで1つのTLSを提供します。 また、構成ファイルはローカルキャッシュを有効にし、FederatorをGravwellインデクサーと信頼できないネットワークセグメント間のフォールトトレラントバッファーとして機能させます。

```
[Global]
	Ingest-Secret = SuperSecretUpstreamIndexerSecret
	Connection-Timeout = 0
	Insecure-Skip-TLS-Verify = false
	Encrypted-Backend-target=172.20.232.105:4024
	Encrypted-Backend-target=172.20.232.106:4024
	Ingest-Cache-Path=/opt/gravwell/cache/federator.cache
	Max-Ingest-Cache=1024 #1GB
	Log-Level=INFO

[IngestListener "BusinessOps"]
        Ingest-Secret = CustomBusinessSecret
        Cleartext-Bind = 10.0.0.121:4023
        Tags=windows
        Tags=syslog

[IngestListener "DMZ"]
       Ingest-Secret = OtherRandomSecret
       TLS-Bind = 192.168.220.105:4024
       TLS-Certfile = /opt/gravwell/etc/cert.pem
       TLS-Keyfile = /opt/gravwell/etc/key.pem
       Tags=apache
       Tags=nginx
```

DMZのインジェスターは、TLS暗号化を使用して192.168.220.105:4024のフェデレーターに接続できます。 これらのインジェスターは、「apache」および「nginx」タグでタグ付けされたエントリの送信のみを許可されます。 ビジネスネットワークセグメントのインジェスターは、クリアテキストを介して10.0.0.121:4023に接続し、「windows」および「syslog」のタグが付いたエントリを送信できます。 誤ってタグ付けされたエントリは、フェデレーターによって拒否されます。 許容可能なエントリは、グローバルセクションで指定された2つのインデクサーに渡されます。

### トラブルシューティング

Federatorの一般的な構成エラーは次のとおりです。

*グローバル構成での不正なIngest-Secret
*誤ったバックエンドターゲットの指定
*無効または既に使用されているバインド仕様
*上流のインデクサーまたはフェデレーターが信頼できる認証局によって署名された証明書を持っていない場合に証明書の検証を強制する（「Insecure-Skip-TLS-Verify」オプションを参照）
*ダウンストリームインジェスターの不一致のインジェストシークレット

## Ingest API

GravwellインジェストAPIとコアインジェスターは、BSD 2-Clauseライセンスの下で完全にオープンソースです。 これは、独自のインジェスターを作成し、Gravwellエントリ生成を独自の製品およびサービスに統合できることを意味します。 コアインジェストAPIはGoで記述されていますが、利用可能なAPI言語のリストは積極的に拡張されています。

[API code](https://github.com/gravwell/ingest)

[API ドキュメント](https://godoc.org/github.com/gravwell/ingest)

ファイルを監視し、書き込まれた行をGravwellクラスターに送信する非常に基本的なingesterの例（100行未満のコード）[こちらで確認できます](https://www.godoc.org/github.com/gravwell/ingest#example-package)

チームが継続的に取り込みAPIを改善し、追加の言語に移植しているため、Gravwell Githubページで引き続きチェックしてください。 コミュニティ開発は完全にサポートされているため、マージリクエスト、言語ポート、またはオープンソース化されたすばらしい新学期がある場合は、Gravwellにお知らせください！ Gravwellチームは、インジェスターハイライトシリーズであなたのハードワークを紹介したいと思っています。
