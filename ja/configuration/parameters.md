# 構成パラメーター

`gravwell.conf`構成ファイルは、インデクサー、ウェブサーバ－、および構成用のデータストアによって使用されます。構成には、角括弧内に定義された*セクション*が含まれます（例：`[Global]`）。各セクションにはパラメータが含まれています。次の例には、Globalセクション、Replicationセクション、Default-Wellセクション、およびStorage-Wellセクションが含まれています。

```
[global]
### 認証トークン
Ingest-Auth=IngestSecrets
Control-Auth=ControlSecrets
Search-Agent-Auth=SearchAgentSecrets

### ウェブサーバ－のHTTP/HTTPS設定
Web-Port=80
Insecure-Disable-HTTPS=true

### その他のウェブサーバ－設定
Remote-Indexers=net:127.0.0.1:9404
Persist-Web-Logins=True
Session-Timeout-Minutes=1440
Login-Fail-Lock-Count=4
Login-Fail-Lock-Duration=5

### インジェスター設定
Ingest-Port=4023
Control-Port=9404
Search-Pipeline-Buffer-Size=4

### その他の設定
Log-Level=WARN

### パス
Pipe-Ingest-Path=/opt/gravwell/comms/pipe
Log-Location=/opt/gravwell/log
Web-Log-Location=/opt/gravwell/log/web
Render-Store=/opt/gravwell/render
Saved-Store=/opt/gravwell/saved
Search-Scratch=/opt/gravwell/scratch
Web-Files-Path=/opt/gravwell/www
License-Location=/opt/gravwell/etc/license
User-DB-Path=/opt/gravwell/etc/users.db
Web-Store-Path=/opt/gravwell/etc/webstore.db

[Replication]
	Peer=10.0.0.1
	Storage-Location=/opt/gravwell/replication_storage
	Insecure-Skip-TLS-Verify=true
	Disable-TLS=true
	Connect-Wait-Timeout=20

[Default-Well]
	Location=/opt/gravwell/storage/default/
	Accelerator-Name=fulltext #fulltext is the most resilent to varying data types
	Accelerator-Engine-Override=bloom #The bloom engine is effective and fast with minimal disk overhead
	Disable-Compression=true

[Storage-Well "syslog"]
	Location=/opt/gravwell/storage/syslog
	Tags=syslog
	Tags=kernel
	Tags=dmesg
	Accelerator-Name=fulltext #fulltext is the most resilent to varying data types
	Accelerator-Args="-ignoreTS" #tell the fulltext accelerator to not index timestamps, syslog entries are easy to ID
```

各パラメーターにはデフォルト値があり、パラメーターが空の場合、または構成ファイルで指定されていない場合に適用されます。

## Global（全体）のパラメータ

このセクションのパラメーターは、構成ファイルの`[Global]`見出しの下にあります。これは通常、ファイルの先頭にあります。

**Indexer-UUID**
適用対象：インデクサー
デフォルト値：[設定されていない場合はランダムに生成されます]
例：`Indexer-UUID="ecdeeff8-8382-48f1-a24f-79f83af00e97"`
説明：特定のインデクサーに一意の識別子を設定します。2つのインデクサーが同じUUIDを持つことはできません。このパラメーターが設定されていない場合、インデクサーはUUIDを生成し、それを構成ファイルに書き込みます。インデクサーが壊滅的に失敗し、レプリケーションから再構築されない限り、これは通常最良の選択です（ [レプリケーションドキュメント](configuration/replication.md)を参照）。このパラメータを変更する前によく考えてください。

**Webserver-UUID**
適用対象：ウェブサーバ－
デフォルト値：[設定されていない場合はランダムに生成されます]
例：`Webserver-UUID="ecdeeff8-8382-48f1-a24f-79f83af00e97"`
説明：特定のウェブサーバ－に一意の識別子を設定します。2つのウェブサーバ－が同じUUIDを持つことはできません。このパラメーターが設定されていない場合、ウェブサーバ－はUUIDを生成し、*構成ファイルに書き戻します*。これは通常、最良の選択です。このパラメータを変更する前によく考えてください。

**License-Location**
適用対象：インデクサーとウェブサーバー
デフォルト値：`/opt/gravwell/etc/license`
例：`License-Location=/opt/gravwell/etc/my_license`
説明：グラブウェルライセンスファイルへのパスを設定します。パスは、グラブウェルのユーザーとグループが読み取り可能である必要があります。

**Config-Location**
適用対象：インデクサーとウェブサーバー
デフォルト値：`/opt/gravwell/etc`
例： `Config-Location=/tmp/path/to/etc`
説明：構成場所では、他のすべての構成パラメーターを格納するための代替場所を指定できます。代替のConfig-Locationを指定すると、他のすべてのパラメーターを代替パスで指定しなくても、単一のパラメーターを設定できます。

**Web-Port**
適用対象：ウェブサーバ－
デフォルト値：`443`
例：`Web-Port = 80`
説明：ウェブサーバ－のリスニングポートを指定します。Web-Portパラメーターを80に設定しても、ウェブサーバ－はHTTPのみのモードに切り替わりません。これには、Insecure-Disable-HTTPS設定が必要です。

**Disable-HTTP-Redirector**
適用対象：ウェブサーバ－
デフォルト値：`False`
例：`Disable-HTTP-Redirector = true`
説明：デフォルトでは、Gravwellは、クリアテキストHTTPポータルを要求するクライアントを暗号化されたHTTPSポータルにリダイレクトするHTTPリダイレクタを開始します。

**Insecure-Disable-HTTPS**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Insecure-Disable-HTTPS = true`
説明：デフォルトでは、GravwellはHTTPSモードで動作します。 `Insecure-Disable-HTTPS = true`を設定すると、代わりにプレーンテキストHTTPを使用して、`Web-Port`をリッスンするようにGravwellに指示します。

**Webserver-Domain**
適用対象：ウェブサーバ－
デフォルト値：0
例：`Webserver-Domain = 17`
説明：`Webserver-Domain`パラメーターは、ウェブサーバ－上の[resources](#!resources/resources.md)ドメインを制御します。2つのウェブサーバ－が同じドメインで構成され、異なるリソースセットを持ち、同じインデクサーに接続され、データストアを介して同期されていない場合、インデクサーは2つのリソースセット間でスラッシュします。ウェブサーバ－を異なるドメインに配置すると、両方がリソースの競合なしに同じインデクサーを使用できるようになります。

**Control-Listen-Address**
適用対象：インデクサー
デフォルト値：
例：`Control-Listen-Address =” 10.0.0.1”`
説明：Control-Listen-Addressパラメーターは、インデクサーのコントロールリスナーを特定のアドレスにバインドできます。デュアルホームマシン、または高速データネットワークと低速制御ネットワークを備えたマシンへのGravwellのインストールでは、トラフィックが適切にルーティングされるように、インデクサーを特定のアドレスにバインドすることができます。

**Control-Port**
適用対象：インデクサーとウェブサーバー
デフォルト値：`9404`
例：`Control-Port = 12345`
説明：Control-Portパラメーターは、インデクサーがウェブサーバ－からの制御コマンドをリッスンするポートを選択します。この設定は、インデクサーとウェブサーバ－が通信するために同じである必要があります。インストーラーはデフォルトでインデクサーのバインド機能を設定しないため、ポートは1024より大きい値に設定する必要があります。単一のマシンで複数のインデクサーが実行されている環境、または別のアプリケーションが実行されている環境では、制御ポートの調整が必要になる場合があります。ポート9404にバインドします。

**Datastore-Listen-Address**
適用対象：データストア
デフォルト値：
例：`Datastore-Listen-Address = 10.0.0.1`
説明：Datastore-Listen-Addressパラメーターは、特定のアドレスでのみリッスンするようにデータストアに指示します。デフォルトでは、データストアはシステム上のすべてのアドレスをリッスンします。

**Datastore-Port**
適用対象：データストア
デフォルト値：`9405`
例：`Datastore-Port = 7888`
説明：Datastore-Portパラメーターは、データストアが通信するポートを選択します。ポートは1024より大きくする必要があります。デフォルト値の9405は、通常、ほとんどのインストールに適しています。

**Datastore**
適用対象：ウェブサーバ－
デフォルト値：
例：`Datastore = 10.0.0.1：9405`
説明：Datastoreパラメーターは、ウェブサーバ－がデータストアに接続して、ダッシュボード、リソース、ユーザー設定、および検索履歴を同期する必要があることを指定します。これにより、[分散ウェブサーバ－]（distributed/frontend.md）が可能になりますが、必要な場合にのみ設定する必要があります。デフォルトでは、ウェブサーバ－はデータストアに接続しません。

**Datastore-Update-Interval**
適用対象：ウェブサーバ－
デフォルト値：`10`
例：`Datastore-Update-Interval = 5`
説明：Datastore-Update-Intervalパラメーターは、データストアの更新を確認する前にウェブサーバ－が待機する時間（秒単位）を決定します。通常、デフォルト値の10秒が適切です。

**Datastore-Insecure-Disable-TLS**
適用対象：ウェブサーバーとデータストア
デフォルト値：`false`
例：`Datastore-Insecure-Disable-TLS = true`
説明：Datastore-Insecure-Disable-TLSパラメーターは、ウェブサーバ－とデータストアの両方で使用されます。デフォルトでは、データストアはウェブサーバ－からの着信HTTPS接続をリッスンします。このパラメーターをfalseに設定すると、データストアはプレーンテキストHTTPを予期し、ウェブサーバ－にHTTPを使用するように指示します。

**Datastore-Insecure-Skip-TLS-Verify**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Datastore-Insecure-Skip-TLS-Verify = true`
説明：Datastore-Insecure-Skip-TLS-Verifyパラメーターは、データストアに接続するときに無効なTLS証明書を無視するようにウェブサーバ－に指示します。これは自己署名証明書を使用する場合に必要ですが、可能な場合は避ける必要があります。

**External-Addr**
適用対象：ウェブサーバ－
デフォルト値：
例：`External-Addr = 10.0.0.1：443`
説明：External-Addrパラメーターは、他のウェブサーバ－がこのウェブサーバ－に接続するために使用するアドレスを指定します。このパラメーターは、あるウェブサーバ－のユーザーが別のウェブサーバ－で実行された検索の結果をロードできるようにするため、データストアを使用する場合は**必須**です。

**Search-Forwarding-Insecure-Skip-TLS-Verify**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Search-Forwarding-Insecure-Skip-TLS-Verify = true`
説明：このパラメーターは、データストアを使用して分散モードで複数のウェブサーバ－を操作する場合にのみ役立ちます。ウェブサーバ－に自己署名証明書がある場合、このパラメーターがtrueに設定されていない限り、ユーザーはリモートウェブサーバ－からの検索にアクセスできません。

**Ingest-Port**
適用対象：インデクサー
デフォルト値：`4023`
例：`Ingest-Port=14023`
説明：Ingest-Portパラメーターは、インデクサーがingester接続をリッスンするポートを制御します。Ingest-Portパラメーターの変更は、単一のマシンで複数のインデクサーを実行している場合、または別のアプリケーションが既にデフォルトのポートである4023にバインドされている場合に役立ちます。

**TLS-Ingest-Port**
適用対象：インデクサー
デフォルト値：`4024`
例： `TLS-Ingest-Port=14024`
説明：TLS-Ingest-Portパラメーターは、インデクサーが取り込み接続をリッスンするポートを制御します。TLS-Ingest-Portパラメーターの変更は、単一のマシンで複数のインデクサーを実行している場合、または別のアプリケーションが既にデフォルトのポート4024にバインドされている場合に役立ちます。デフォルトでは、TLSトランスポートを使用するすべてのインジェスターがリモート証明書を検証します。展開で自動生成された証明書を使用している場合、インジェスターは証明書を信頼できるものとしてインストールするか、証明書の検証を無効にする必要があります（これにより、TLSトランスポートによって提供される保護が事実上破壊されます）。

**Pipe-Ingest-Path**
適用対象：インデクサー
デフォルト値：`/opt/gravwell/comms/pipe`
例：`Pipe-Ingest-Path=/tmp/path/to/pipe`
説明：Pipe-Ingest-Pathは、Unixの名前付きパイプへのフルパスを指定します。インデクサーは名前付きパイプを作成し、共存するインジェスターはパイプに接続して、非常に高速で低遅延のトランスポートとして使用できます。名前付きパイプは、1ギガビットを超えるネットワークパケットインジェスターなど、非常に高いパフォーマンスを必要とするインジェスターに最適です。名前付きパイプは、異常なネットワークトランスポートまたは非常に高速な非IPベースの相互接続を介したトランスポートを容易にするためにも使用できます。

**Log-Location**
適用対象：インデクサー
デフォルト値：`/opt/gravwell/log`
例：`Log-Location=/tmp/path/to/logs`
説明：Log-Locationパラメーターは、Gravwellインフラストラクチャが独自のログを配置する場所を制御します。 Gravwellは、独自のログをインデクサーに直接フィードせず、代わりにファイルに書き込みます（Gravwellログも取り込みたい場合は、ファイルフォロワーingesterを使用してください）。 このパラメーターは、それらのログの移動先を指定します。

**Web-Log-Location**
適用対象：ウェブサーバ－
デフォルト値： `/opt/gravwell/log/web`
例： `Web-Log-Location=/tmp/path/to/logs/web`
説明：Web-Log-Locationパラメーターは、ウェブサーバ－ログの保存場所を制御します。 Gravwellは、独自のログをインデクサーに直接フィードせず、代わりにファイルに書き込みます（Gravwellログも取り込みたい場合は、ファイルフォロワーingesterを使用してください）。 このパラメーターは、それらのログの移動先を指定します。

**Datastore-Log-Location**
適用対象：データストア
デフォルト値：`/opt/gravwell/log/datastore`
例：`Datastore-Log-Location=/tmp/path/to/logs/datastore`
説明：Datastore-Log-Locationパラメーターは、データストアログが保存される場所を制御します。

**Log-Level**
適用対象：インデクサー、データストア、およびウェブサーバ－
デフォルト値：`INFO`
例：`Log-Level = ERROR`
説明：Log-Levelパラメーターは、gravwellインフラストラクチャからのログの詳細度を制御します。ログレベルには、INFO、WARN、およびERRORの3つの引数を使用できます。INFOが最も冗長で、ERRORが最も少なくなります。ロギングシステムは、ロギングのレベルごとにファイルを生成し、syslogデーモンと同様の方法でそれらをローテーションします。

**Disable-Access-Log**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Disable-Access-Log = true`
説明：Disable-Access-Logパラメーターは、ウェブサーバ－によって生成されたアクセスログを無効にするために使用されます。アクセスログインフラストラクチャは、個々のページアクセスをログに記録します。Gravwellアクセスを監査し、潜在的な問題をデバッグするためにこれらのアクセスログを用意することは通常価値がありますが、ユーザーが多い環境ではアクセスログが大きくなる可能性があるため、無効にすることが望ましい場合があります。

**Persist-Web-Logins**
適用対象：ウェブサーバ－
デフォルト値：`true`
例：`Persist-Web-Logins = false`
説明：Persist-Web-Loginsパラメーターは、シャットダウン時にユーザーセッションを不揮発性ストレージに保存する必要があることをウェブサーバ－に通知するために使用されます。 デフォルトでは、ウェブサーバ－がシャットダウンまたは再起動された場合、クライアントセッションが保持されます。 Persist-Web-Loginsをfalseに設定すると、ウェブサーバ－が再起動されるたびにセッションが無効になります。

**Session-Timeout-Minutes**
適用対象：ウェブサーバ－
デフォルト値：`60`
例：`Session-Timeout-Minutes = 1440`
説明：Session-Timeout-Minutesパラメーターは、ウェブサーバ－がセッションを破棄する前にクライアントがアイドル状態でいられる時間を制御します。たとえば、クライアントがログアウトせずにブラウザを閉じると、システムは指定された期間待機してからセッションを無効にします。インストーラーは、この値をデフォルトで1日に設定します。

**Key-File**
適用対象：インデクサー、データストア、およびウェブサーバ－
デフォルト値： `/opt/gravwell/etc/key.pem`
例： `Key-File=/opt/gravwell/etc/privkey.pem`
説明：Key-Fileパラメーターは、ウェブサーバ－、データストア、およびインデクサーの秘密鍵として使用されるファイルを制御します。 秘密鍵/公開鍵はPEM形式でエンコードする必要があります。秘密鍵は保護する必要があり、侵害された場合は破棄して再発行する必要があります。詳細については、http：//www.tldp.org/HOWTO/SSL-Certificates-HOWTO/x64.htmlを参照してください。

**Certificate-File**
適用対象：インデクサー、データストア、およびウェブサーバ－
デフォルト値：`/opt/gravwell/etc/cert.pem`
例： `Certificate-File=/opt/gravwell/etc/cert.pem`
説明：Certificate-Fileパラメーターは、TLSトランスポートに使用される公開鍵と秘密鍵のペアの公開鍵コンポーネントを指定します。公開鍵はすべてのingesterおよびWebクライアントに配信され、機密とは見なされません。 Gravwellは、公開鍵がPEM形式でエンコードされ、公開鍵部分のみを含むことを想定しています。

**Ingest-Auth**
適用対象：インデクサー
デフォルト値：`IngestSecrets`
例：`Ingest-Auth = abcdefghijklmnopqrstuvwxyzABCD`
説明：Ingest-Authパラメーターは、インジェスターをインデクサーに対して認証するために使用される共有秘密トークンを指定します。このトークンは任意の長さにすることができます。Gravwellは、少なくとも24文字の高エントロピートークンを推奨しています。デフォルトでは、インストーラーはランダムトークンを生成します。

**Control-Auth**
適用対象：インデクサーとウェブサーバー
デフォルト値：`ControlSecrets`
例：`Control-Auth = abcdefghijklmnopqrstuvwxyzABCD`
説明：Control-Authパラメーターは、ウェブサーバ－に対してインジェスターを認証するために使用される共有シークレットトークンを指定します。その逆も同様です。このトークンは任意の長さにすることができます。Gravwellは、少なくとも24文字の高エントロピートークンを推奨しています。デフォルトでは、インストーラーはランダムトークンを生成します。

**Search-Agent-Auth**
適用対象：ウェブサーバ－
デフォルト値：
例：`Search-Agent-Auth = abcdefghijklmnopqrstuvwxyzABCD`
説明：Search-Agent-Authパラメーターは、ウェブサーバ－に対して検索エージェントを認証するために使用される共有秘密トークンを指定します。インストーラーはデフォルトでランダム検索エージェントトークンを生成します。

**Web-Files-Path**
適用対象：ウェブサーバ－
デフォルト値： `/opt/gravwell/www`
例：`Web-Files-Path=/tmp/path/to/www`
説明：Web-Files-Pathは、ウェブサーバ－によって提供されるフロントエンドGUIファイルを含むパスを指定します。Webファイルには、Webページの表示とWebブラウザを介したGravwellシステムとの対話を担当するすべてのGravwellコードが含まれています。

**Tag-DB-Path**
適用対象：インデクサー
デフォルト値：`/opt/gravwell/etc/tags.db`
例：`Tag-DB-Path=/tmp/path/to/tags.db`
説明：Tag-DB-Pathパラメーターは、タグデータベースの場所を指定します。 このファイルは、インデクサーの数値タグIDをタグ名文字列にマップします。

**User-DB-Path**
適用対象：ウェブサーバ－
デフォルト値：`/opt/gravwell/etc/users.db`
例：`User-DB-Path=/tmp/path/to/users.db`
説明：User-DB-Pathパラメーターは、ユーザーデータベースファイルの場所を指定します。ユーザーデータベースファイルには、ユーザーとグループの構成が含まれています。ユーザーデータベースはbcryptハッシュアルゴリズムを使用してパスワードを保存および検証します。これは非常に堅牢であると考えられていますが、users.dbファイルは引き続き保護する必要があります。デフォルトでは、インストーラーは、ユーザーデータベースファイルに対するファイルシステムのアクセス許可を、Gravwellユーザーとグループのみが読み取り可能になるように設定します。

**Datastore-User-DB-Path**
適用対象：データストア
デフォルト値：`/opt/gravwell/etc/datastore-users.db`
例：`Datastore-User-DB-Path=/tmp/path/to/datastore-users.db`
説明：Datastore-User-DB-Pathパラメーターは、データストアコンポーネントによって管理されるユーザーデータベースファイルの場所を指定します。これは、User-DB-Pathパラメーターで指定されたものと**同じパスであってはなりません**。

**Web-Store-Path**
適用対象：ウェブサーバ－
デフォルト値：`/opt/gravwell/etc/webstore.db`
例： `Web-Store-Path=/tmp/path/to/webstore.db`
説明：Web-Store-Pathは、検索履歴、ダッシュボード、ユーザー設定、ユーザーセッション、およびその他のその他のユーザーデータを保存するために使用されるデータベースファイルを指します。ウェブストアデータベースファイルにはユーザー資格情報は含まれていませんが、ユーザーセッションCookieとCSRFトークンは*含まれています*。GravwellはCookieとCSRFトークンをオリジンに結び付けるため、攻撃者が盗まれたCookieまたはトークンとして再利用するリスクは低いものの、データストアを保護する必要があります。インストーラーは、Gravwellユーザーによる読み取り/書き込みのみを許可するようにファイルシステムのアクセス許可を設定します。

**Datastore-Web-Store-Path**
適用対象：データストア
デフォルト値：`/opt/gravwell/etc/datastore-webstore.db`
例：`Datastore-Web-Store-Path=/tmp/path/to/datastore-webstore.db`
説明：Datastore-Web-Store-Pathパラメーターは、データストアが検索履歴、ダッシュボード、およびユーザー設定を保存するために使用するデータベースファイルを指します。これは、Web-Store-Pathパラメーターで指定されたものと**同じパスであってはなりません**。

**Web-Listen-Address**
適用対象：ウェブサーバ－
デフォルト値：
例：`Web-Listen-Address=10.0.0.1`
説明：Web-Listen-Addressパラメーターは、ウェブサーバ－がバインドして提供するアドレスを指定します。デフォルトでは、パラメーターは空です。これは、ウェブサーバ－がすべてのインターフェースとアドレスにバインドすることを意味します。

**Login-Fail-Lock-Count**
適用対象：ウェブサーバ－
デフォルト値：`5`
例：`Login-Fail-Lock-Count = 10`
説明：Login-Fail-Lock-Countパラメーターは、アカウントでブルートフォース保護が有効になる前に、ユーザーアカウントに対して連続して失敗したログインの数を指定します。たとえば、値が4に設定されていて、ユーザーが不正なパスワードを4回続けて入力した場合、追加のログイン試行の完了に時間がかかり、攻撃者の速度が低下します。注：Gravwellは以前、特定の数の失敗後にアカウントをロックしていました。現在は攻撃的なブルートフォース保護を行っていませんが、従来の理由により、構成パラメーターは「ロック」名を保持しています。

**Login-Fail-Lock-Duration**
適用対象：ウェブサーバ－
デフォルト値：`5`
例：`Login-Fail-Lock-Duration = 10`
説明：Login-Fail-Lock-Durationパラメーターは、Login-Fail-Lock-Countを超えたかどうかを計算するときに使用されるウィンドウ（分単位）を指定します。注：Gravwellは以前、特定の数の失敗後にアカウントをロックしていました。現在は攻撃的なブルートフォース保護を行っていませんが、従来の理由により、構成パラメーターは'ロック'名を保持しています。

**Remote-Indexers**
適用対象：ウェブサーバ－
デフォルト値：`net:10.0.0.1:9404`
例：`Remote-Indexers=net:10.0.0.1:9404`
説明：Remote-Indexersパラメーターは、ウェブサーバ－が接続および制御する必要があるリモートインデクサーのアドレスとポートを指定します。Remote-Indexersはリストパラメータです。つまり、複数のリモートインデクサーを提供するために何度も指定できます。Gravwell Clusterエディションは、クラスター内の各インデクサーを指定する必要があります。「net：」プレフィックスは、リモートインデクサーがネットワークトランスポートを介してアクセスできることを示します。Gravwellの特別版は代替トランスポートを使用できますが、ほとんどの商用顧客は「net:」の使用を期待する必要があります。

**Search-Scratch**
適用対象：インデクサーとウェブサーバー
デフォルト値：`/opt/gravwell/scratch`
例： `Search-Scratch=/tmp/path/to/scratch`
説明：Search-Scratchパラメーターは、アクティブな検索中に検索モジュールが一時ストレージに使用できるストレージの場所を指定します。一部の検索モジュールでは、メモリの制約により一時ストレージを使用する必要がある場合があります。たとえば、ソートモジュールは5GBのデータをソートする必要があるかもしれませんが、物理マシンには4GBの物理RAMしかない場合があります。モジュールは、スクラッチスペースをインテリジェントに使用して、ホストのスワップを呼び出さずに大きなデータセットを並べ替えることができます（並べ替えだけでなく、すべてのモジュールにペナルティが課せられます）。各検索の終了時に、スクラッチスペースは破棄されます。

**Render-Store**
適用対象：ウェブサーバ－
デフォルト値：`/opt/gravwell/render`
例：`Render-Store=/tmp/path/to/render`
説明：Render-Storeパラメーターは、レンダラーモジュールが検索結果を保存する場所を指定します Render-Storeの場所は一時的な保存場所であり、通常は適度に小さいデータセットを表します。検索がアクティブに実行されているか、休止状態でクライアントと対話している場合、レンダラーはデータセットを保存および取得する場所です。Render-Storeは、フラッシュベースまたはXPointSSDなどの高速ストレージに配置する必要があります。検索が中止されると、レンダーストアは削除されます（検索が保存されていない場合）。

**Saved-Store**
適用対象：ウェブサーバ－
デフォルト値： `/opt/gravwell/saved`
例： `Saved-Store=/path/to/saved/searches`
説明：Saved-Storeパラメーターは、保存された検索が保存される場所を指定します。保存された検索は検索の出力状態を表し、ユーザーが検索を再開せずに後で検索結果を再度参照できるようにしたい監査や状況に役立ちます。保存された検索は明示的に削除する必要があり、データはシャードエージングアウトポリシーの対象ではありません。保存された検索は完全にアトミックです。つまり、保存された検索の基になるデータは完全に古くなり、削除されても、ユーザーは保存された検索を再度開いて調べることができます。保存された検索は共有することもできます。つまり、ユーザーは保存された検索をまとめて、Gravwellの他のインスタンスと共有できます。

**Search-Pipeline-Buffer-Size**
適用対象：インデクサーとウェブサーバー
デフォルト値：`2`
例：`Search-Pipeline-Buffer-Size = 8`
説明：Search-Pipeline-Buffer-Sizeは、検索中に各モジュール間で転送できるブロックの数を指定します。サイズを大きくすると、常駐メモリの使用量を犠牲にして、バッファリングが向上し、スループットが向上する可能性があります。インデクサーはパイプラインのサイズに対してより敏感ですが、システムが自由にメモリを削除して再インスタンス化できる共有メモリ技術も使用します。ウェブサーバ－は通常、パイプラインを移動するときにすべてのエントリを常駐させ、メモリの負荷を軽減するためにモジュールの圧縮に依存します。システムが回転ディスクなどの待ち時間の長いストレージシステムを使用している場合は、このバッファーサイズを増やすと有利な場合があります。このパラメーターを増やすと、検索のパフォーマンスが向上する可能性がありますが、システムが一度に処理できる実行中の検索の数に直接影響します。ビデオフレーム、PE実行可能ファイル、オーディオファイルなどの非常に大きなエントリを保存していることがわかっている場合は、常駐メモリの使用を制限するためにバッファサイズを減らす必要がある場合があります。ホストカーネルがメモリ不足（OOM）を呼び出して、Gravwellプロセスを起動および強制終了しているのを確認した場合、これが最初に回すノブです。

**Search-Relay-Buffer-Size**
適用対象：ウェブサーバ－
デフォルト値：`4`
例：`Search-Relay-Buffer-Size = 8`
説明：Search-Relay-Buffer-Sizeパラメーターは、別のインデクサーからの未処理のブロックを待機している間に、ウェブサーバ－が各インデクサーから受け入れるエントリブロックの数を制御します。検索エントリが一時的に流入するため、1つのインデクサがまだ古いエントリを処理している一方で、別のインデクサがより新しいエントリに進んでいる可能性があります。ウェブサーバ－はエントリを時間順に処理する必要があるため、遅いインデクサーが追いつくのを待つ間、「先行」しているインデクサーからのエントリをバッファリングします。一般に、デフォルト値は、許容可能なパフォーマンスを提供しながら、メモリの問題を防ぐのに役立ちます。大量のメモリを搭載したシステムでは、この値を増やすと便利な場合があります。

**Max-Search-History**
適用対象：ウェブサーバ－
デフォルト値：`100`
例：`Max-Search-History = 256`
説明：Max-Search-Historyパラメーターは、ユーザーに対して保持される検索の数を制御します。検索履歴は、戻って古い検索を調べたり、グループ内の他のユーザーが検索しているものを確認したりするのに役立ちます。履歴を大きくすると、古い検索文字列の末尾を大きくすることができますが、履歴に保持される検索が多すぎると、GUIを操作するときに速度が低下する可能性があります。

**Prebuff-Block-Hint**
適用対象：インデクサー
デフォルト値：`32`
例：`Prebuff-Block-Hint = 8`
説明：Prebuff-Block-Hintは、データのブロックを格納するときにインデクサーが狙うべきソフトターゲットをメガバイト単位で指定します。非常に高スループットのシステムでは、この値を少し高くしたい場合がありますが、メモリに制約のあるシステムでは、この値を低くしたい場合があります。この値はソフトターゲットであり、インデクサーは通常、摂取が高率で発生している場合にのみこの値を使用します。

**Prebuff-Max-Size**
適用対象：インデクサー
デフォルト値：`32`
例：`Prebuff-Max-Size = 128`
説明：Prebuff-Max-Sizeパラメーターは、エントリをディスクに強制する前にプリバッファーが保持する最大データサイズをメガバイト単位で制御します。プリバッファは、ソースクロックが十分に同期されていない可能性がある場合に、エントリのストレージを最適化するために使用されます。プリバッファが大きいということは、インデクサーが、非常に順序が狂っている値を提供しているインジェスターをより適切に最適化できることを意味します。各ウェルには独自のプリバッファーがあるため、インストールで4つのウェルが定義され、Prebuff-Max-Sizeが256の場合、インデクサーはデータを保持する最大1GBのメモリを消費できます。 プリバッファは定期的にエントリを削除し、それらを常にストレージメディアにプッシュするため、プリバッファの最大サイズは通常、高スループットシステムにのみ関与します。これは、ホストシステムのOOMキラーがGravwellプロセスを終了している場合に、（Search-Pipeline-Buffer-Sizeの後に）回す2番目のノブです。

**Prebuff-Max-Set**
適用対象：ウェブサーバ－
デフォルト値：`256`
例：`Prebuff-Max-Set = 256`
説明：Prebuff-Max-Setは、最適化のためにプリバッファーに保持できる1秒のブロックの数を指定します。インジェスターによって提供されたエントリのタイムスタンプが同期していないほど、このセットを大きくする必要があります。たとえば、タイムスタンプが2時間も変動する可能性のあるソースから消費している場合は、この値を7200に設定できますが、データが通常非常に厳しいタイムスタンプ許容値で到着する場合は、この値を低くすることができます。 Prebuff-Max-Sizeコントロールは引き続きプリバッファの削除を有効にして強制するため、この値を高く設定しすぎると、低く設定しすぎるよりも害が少なくなります。

**Prebuff-Tick-Interval**
適用対象：ウェブサーバ－
デフォルト値：`3`
例：`Prebuff-Tick-Interval = 4`
説明：Prebuff-Tick-Intervalパラメーターは、プリバッファーがプリバッファーにあるエントリーの人為的な排除を行う頻度を秒単位で指定します。プリバッファは、アクティブな取り込みがある場合、常に値を永続ストレージに削除しますが、非常に低スループットのシステムでは、この値を使用して、エントリが永続ストレージに強制的にプッシュされるようにすることができます。 Gravwellは、データが役立つ場合、データが失われることを決して許しません。インデクサーを正常にシャットダウンすると、プリバッファーはすべてのエントリが永続ストレージに確実に到達するようにします。ただし、ホストの安定性にあまり自信がない場合は、この間隔を2に近づけて、システム障害や怒っている管理者がインデクサーの下から敷物を引き出せないようにすることができます。

** Prebuff-Sort-On-Consume **
適用対象：インデクサー
デフォルト値：`false`
例：`Prebuff-Sort-On-Consume = true`
説明：Prebuff-Sort-On-Consumeパラメーターは、データをディスクにプッシュする前に、データのロックをソートするようにプリバッファーに指示します。並べ替えプロセスは個々のブロックにのみ適用され、パイプラインに入るときにデータが並べ替えられることを保証するものではありません。ストレージの前にブロックを並べ替えると、取り込み時にパフォーマンスが大幅に低下します。ほとんどすべてのインストールでは、この値をfalseのままにしておく必要があります。

**最大ブロックサイズ**
適用対象：インデクサー
デフォルト値：`4`
例：`Max-Block-Size = 8`
説明：Max-Block-Sizeはメガバイト単位の値を指定し、エントリをパイプラインにプッシュするときに生成できる最大ブロックサイズをインデクサーに通知するためのヒントとして使用されます。ブロックを大きくすると、パイプラインへのプレッシャーは軽減されますが、メモリプレッシャーは増加します。大容量のメモリと高スループットのシステムではこの値を増やしてスループットを上げることができ、小さいメモリシステムではこのサイズを減らしてメモリの負荷を減らすことができます。 Prebuff-Block-HintパラメーターとMax-Block-Sizeパラメーターが交差して、取り込みと検索のスループットを調整する2つのノブを提供します。Gravwellでは、128GBノードで、次のことが達成されます。クリーンな1GB /秒の検索スループット。Max-Block-Sizeが16の場合、1秒あたり125万エントリが取り込まれます。そして、8のプリバフブロックヒントが達成されます

**Render-Store-Limit**
適用対象：ウェブサーバ－
デフォルト値：1024
例：`Render-Store-Limit = 512`
説明：Render-Store-Limitパラメーターは、検索レンダラーが保存できるメガバイト数を指定します。

**Search-Control-Script**
適用対象：ウェブサーバ－
デフォルト値：
例：`Search-Control-Script=/opt/gravwell/etc/authscripts/limits.grv`
説明：Search-Control-Scriptパラメーターは、検索時に適用されるスクリプトを指定できるリストパラメーターです。リストパラメータであるため、複数回指定して複数のスクリプトを指定できます。これらのスクリプトは、ユーザーが実行する検索に追加の制限を適用できます。すべてのスクリプトは、検索ごとに実行されます。検索制御スクリプトの詳細については、Gravwellにお問い合わせください。

**Webserver-Resource-Store**
適用対象：ウェブサーバ－
デフォルト値：`/opt/gravwell/resources/webserver`
例：`Webserver-Resource-Store=/tmp/path/to/resources/webserver`
説明：Webserver-Resource-Storeパラメーターは、ウェブサーバ－がリソースを格納する場所を指定します。このディレクトリは、他のプロセスで使用されていない必要があり、インデクサーまたはデータストアのリソースの場所として指定することはできません。

**Indexer-Resource-Store**
適用対象：インデクサー
デフォルト値：`/opt/gravwell/resources/indexer`
例：`Indexer-Resource-Store=/tmp/path/to/resources/indexer`
説明：Indexer-Resource-Storeパラメーターは、インデクサーがリソースを格納する場所を指定します。このディレクトリは他のプロセスで使用されていない必要があり、ウェブサーバ－またはデータストアのリソースの場所として指定することはできません。

**Datastore-Resource-Store**
適用対象：データストア
デフォルト値：`/opt/gravwell/resources/datastore`
例：`Datastore-Resource-Store=/tmp/path/to/resources/datastore`
説明：Datastore-Resource-Storeパラメーターは、データストアがリソースを格納する場所を指定します。このディレクトリは、他のプロセスで使用する必要があり、インデクサーまたはウェブサーバ－のリソースの場所として指定することはできません。

**Resource-Max-Size**
適用対象：ウェブサーバー、データストア、インデクサー
デフォルト値：`134217728`
例：`Resource-Max-Size = 1000000000`
説明：Resource-Max-Sizeパラメーターは、リソースの最大サイズをバイト単位で指定します。

**Docker-Secrets**
適用対象：ウェブサーバー、データストア、インデクサー
デフォルト値：`false`
例：`Docker-Secrets = true`
説明：Docker-Secretsパラメーターは、[Docker secrets](https://docs.docker.com/engine/swarm/secrets/)からエージェントシークレットの取り込み、制御、検索を試行するようにGravwellに指示します。 シークレットには、それぞれ`ingest_secret`、`control_secret`、および `search_agent_secret`という名前が付けられている必要があり、VM内の`/run/secrets/`ディレクトリからアクセスできる必要があります。

**HTTP-Proxy**
適用対象：ウェブサーバ－
デフォルト値：
例：`HTTP-Proxy = wwwproxy.example.com：8080`
説明：HTTP-Proxyパラメーターは、ウェブサーバ－によるHTTPおよびHTTP要求に使用されるプロキシーを構成します。これは、環境変数$ http_proxyを設定することと実質的に同等であり、同じ構文を許可します。指定されたプロキシ値は、`HTTP`リクエストと`HTTPS`リクエストの両方に使用されます。

**Webserver-Ingest-Groups**
適用対象：ウェブサーバ－
デフォルト値：
例： `Webserver-Ingest-Groups = ingestUsers`
説明：Webserver-Ingest-Groupsパラメーターは、ユーザーがGravwell WebAPIを介して直接エントリを取り込むことを許可されているグループを指定するリストパラメーターです。リストパラメータとして、複数回指定して、複数のグループがWebAPIを介して取り込むことができるようにすることができます。

**Disable-Update-Notification**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Disable-Update-Notification = false`
説明：Disable-Update-Notificationがtrueに設定されている場合、Gravwellの新しいバージョンが利用可能になったときにWebUIは通知を表示しません。

**Disable-Stats-Report**
適用対象：ウェブサーバ－
デフォルト値：false
例：`Disable-Stats-Report = true`
説明：このパラメーターをtrueに設定すると、ウェブサーバ－の[metrics reporting routine](#!metrics.md)に、ライセンスに関する最小限の情報のみを送信し、より広範なシステム統計を省略します。

**Temp-Dir**
適用対象：ウェブサーバ－
デフォルト値：`/opt/gravwell/tmp`
例：`Temp-Dir = / tmp / gravtmp`
説明：Temp-Dirパラメーターは、他のプロセスからの干渉のリスクなしに一時Gravwellファイルに使用できるディレクトリーを指定します。アップロードされたキットをインストール前に保存するために使用されます。

**Insecure-User-Unsigned-Kits-Allowed**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Insecure-User-Unsigned-Kits-Allowed = true`
説明：このパラメーターが設定されている場合、すべてのユーザーが署名されていないキットをインストールできます。このオプションを有効にしないことを強くお勧めします。

**Disable-Search-Agent-Notifications**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Disable-Search-Agent-Notifications = true`
説明：trueに設定すると、このパラメーターは、検索エージェントがチェックインに失敗した場合にWeb UIが通知を表示しないようにします。これは、検索エージェントを無効にして通知を表示したくない場合に役立ちます。

**Indexer-Storage-Notification-Threshold**
適用対象：インデクサー
デフォルト値：`90`
例：`Indexer-Storage-Notification-Threshold = 98`
説明：ストレージ使用量について警告するタイミングを決定するパーセンテージ値。値が0より大きい場合、インデクサーによって使用されるストレージデバイスが指定されたストレージパーセンテージを超えて使用するたびに通知がスローされます。値は0から99の間でなければなりません。

**Disable-Network-Script-Functions**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Disable-Network-Script-Functions = true`
説明：デフォルトでは、パイプライン内のankoスクリプトは、net/httpライブラリやssh/sftpユーティリティなどのネットワーク関数を使用できます。これを「true」に設定すると、これらの機能が無効になります。

**Webserver-Enable-Frame-Embedding**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Webserver-Enable-Frame-Embedding = true`
説明：デフォルトでは、ウェブサーバ－はヘッダーX-Frame-Options：denyを設定することにより、Gravwellページがフレーム内でレンダリングされることを禁止しています。この構成パラメーターを「true」に設定すると、そのヘッダーが削除され、ページをフレーム内に埋め込むことができます。

**Webserver-Content-Security-Policy**
適用対象：ウェブサーバ－
デフォルト値： ``
例：`Webserver-Content-Security-Policy =" default-src https： "`
説明：このパラメーターを使用すると、管理者は、すべてのGravwellページで送信されるContent-Security-Policyヘッダーを定義できます。これは重要なセキュリティオプションであり、httpsのみを要求するなど、展開要件に基づいて組織に設定する必要があります。

**Default-Language**
適用対象：ウェブサーバ－
デフォルト値：`en-US`
例：`Default-Language = en-US`
説明：Default-Languageパラメーターの設定は、認証されていないAPIの/ api/languageで提供されるものを制御し、複数の言語を使用するデプロイメントでどの言語をデフォルトにするかを決定するためにGUIによって使用されます。これは、ユーザーが言語を選択しておらず、ブラウザが `window.navigator.language`を介して優先言語を提供していない場合のフォールバックです。

**Disable-Map-Tile-Server-Proxy**
適用対象：ウェブサーバ－
デフォルト値：`false`
例：`Disable-Map-Tile-Server-Proxy = true`
説明：このパラメーターは、Gravwellの組み込みマッププロキシを制御します。マップサーバーに過度の負荷がかかるのを防ぐために、Gravwellウェブサーバ－はマップタイルをキャッシュします。ただし、プロキシを使用するということは、実際のマップサーバーに送信される要求が、ユーザーのWebブラウザーではなくGravwellウェブサーバ－から発信されることを意味します。 Gravwellのインストールがロックダウンされたネットワーク上にある場合、発信HTTPが無効になっているため、これが失敗する可能性があります。 `Disable-Map-Tile-Server-Proxy`をtrueに設定すると、組み込みプロキシが無効になり、GUIが直接マップリクエストを行うようになります。プロキシが無効になっていて、 `Map-Tile-Server`パラメータも設定されている場合、GUIはそのサーバーにリクエストを送信します。

**Map-Tile-Server**
適用対象：ウェブサーバ－
デフォルト値： ``
例：`Map-Tile-Server = https//maps.example.com/osm /`
説明：Map-Tile-Serverパラメーターを使用すると、管理者はマップタイルの別のソースを定義できます。デフォルトでは、GravwellはGravwellマップサーバーからタイルをフェッチし、必要に応じてOpenStreetMapサーバーにフォールバックします。このパラメータを設定すると、Gravwellは指定されたサーバーのみを使用するようになります。指定するURLは、[ここ](https://wiki.openstreetmap.org/wiki/Tile_servers)で定義されている標準のOpenStreetMapタイルサーバー形式のプレフィックスであり、z/x/y座標パラメーターは省略されている必要があります。たとえば、タイルに「https://maps.wikimedia.org/osm-intl/${z}/${x}/${y}.png」でアクセスできる場合、たとえばhttps://maps.wikimedia.org/osm-intl/0/1/2.pngの場合、 `Map-Tile-Server = https//maps.wikimedia.org/osm-intl/`を設定できます。

**Gravwell-Tile-Server-Cooldown-Minutes**
適用対象：ウェブサーバ－
デフォルト値：5
例：`Gravwell-Tile-Server-Cooldown-Minutes = 1`
説明：Gravwellタイルプロキシが通常モードで動作している場合（無効になっていない、`Map-Tile-Server`パラメータが設定されていない）、Gravwellが動作するサーバーからマップタイルをフェッチしようとします。そのサーバーへのリクエストの送信が失敗した場合、プロキシはクールダウンの間、代わりにopenstreetmap.orgサーバーにフォールバックします。このパラメーターを0に設定すると、クールダウンが無効になります。

**Gravwell-Tile-Server-Cache-MB**
適用対象：ウェブサーバ－
デフォルト値：4
例：`Gravwell-Tile-Server-Cache-MB = 32`
説明：Gravwellタイルプロキシは、最近アクセスしたタイルのキャッシュを維持して、マップのレンダリングを高速化します。このパラメーターは、キャッシュが使用できるストレージのメガバイト数を制御します。

**Gravwell-Tile-Server-Cache-Timeout-Days**
適用対象：ウェブサーバ－
デフォルト値：7
例：`Gravwell-Tile-Server-Cache-Timeout-Days = 2`
説明：Gravwellタイルプロキシは、最近アクセスしたタイルのキャッシュを維持して、マップのレンダリングを高速化します。このパラメーターは、キャッシュされたタイルが有効であると見なされる最大日数を制御します。その時間が経過すると、タイルはパージされ、アップストリームサーバーから再フェッチされます。

**Disable-Single-Indexer-Optimization**
適用対象:ウェブサーバ－
デフォルト値:false
例：`Disable-Single-Indexer-Optimization = true`
説明：Gravwellを単一のインデクサーで使用すると、デフォルトで*インデクサー*ですべてのモジュール（レンダリングモジュールを除く）が実行され、インデクサーからウェブサーバ－に転送されるデータの量が削減されます。このオプションは、その最適化を無効にします。Gravwellのサポートからの指示がない限り、このオプションを「false」に設定したままにしておくことを強くお勧めします。

**Library-Dir**
適用対象：ウェブサーバ－
デフォルト値：`/opt/gravwell/libs`
例：`Library-Dir = /スクラッチ/ libs`
説明：スケジュールされたスクリプトは、`include`関数を使用して追加のライブラリをインポートできます。これらのライブラリは外部リポジトリからフェッチされ、ローカルにキャッシュされます。この構成オプションは、キャッシュされたライブラリーが保管されるディレクトリーを設定します。

**Library-Repository**
適用対象：ウェブサーバ－
デフォルト値：`https//github.com/gravwell/libs`
例：`Library-Repository = https//github.com/example/gravwell-libs`
説明：スケジュールされたスクリプトは、`include`関数を使用して追加のライブラリをインポートできます。これらのライブラリは、このパラメータで指定されたリポジトリにあるファイルからロードされます。デフォルトでは、便利なライブラリのGravwellが管理するリポジトリを指します。独自のライブラリセットを提供する場合は、このパラメーターを、制御するgitリポジトリーを指すように設定します。

**Library-Commit**
適用対象：ウェブサーバ－
デフォルト値：
例：`Library-Commit = 19b13a3a8eb877259a06760e1ee35fae2669db73`
説明：スケジュールされたスクリプトは、`include`関数を使用して追加のライブラリをインポートできます。これらのライブラリは、 `Library-Repository`オプションで指定されたリポジトリにあるファイルからロードされます。デフォルトでは、Gravwellは最新バージョンを使用します。git commit stringが指定されている場合、Gravwellは代わりに指定されたバージョンのリポジトリを使用しようとします。

**Disable-Library-Repository**
適用対象：ウェブサーバ－
デフォルト値：false
例：`Disable-Library-Repository = true`
説明：スケジュールされたスクリプトは、`include`関数を使用して追加のライブラリをインポートできます。`Disable-Library-Repository`をtrueに設定すると、この機能が無効になります。

**Gravwell-Kit-Server**
適用対象：ウェブサーバ－
デフォルト値：https：//kits.gravwell.io/kits
例：`Gravwell-Kit-Server = http://internal.mycompany.io/gravwell/kits`
説明：Gravwellキットサーバーホストの設定を変更できます。これは、Gravwellキットサーバーのミラーをホストするエアギャップまたはセグメント化されたデプロイメントで役立ちます。この値を空の文字列に設定すると、リモートキットサーバーへのアクセスが完全に無効になります。
例：
`` `
Gravwell-Kit-Server = "" #gravwellキットサーバーへのリモートアクセスを無効にする
Gravwell-Kit-Server = "http://gravwell.mycompany.com/kits" #内部ミラーを使用することに変更します。
`` `

**Kit-Verification-Key**
適用対象：ウェブサーバ－
デフォルト値：
例：`Kit-Verification-Key = /opt/gravwell/etc/kits-pub.pem`
説明：キットサーバーからキットを検証するときに使用する公開鍵を含むファイルを指定します。代替のGravwell-Kit-Serverを指定した場合は、この値を設定します。Gravwellの公式キットサーバーを使用する場合は必要ありません。キットの署名に適したキーは、[gencert](https://github.com/gravwell/gencert)ユーティリティを使用して生成できます。

##パスワード制御

`[Password-Control]`構成セクションを使用して、ユーザーの作成時またはパスワードの変更時にパスワードの複雑さのルールを適用できます。このブロックで設定されたオプションは、ウェブサーバ－にのみ適用されます。これらの複雑さの構成ルールは、シングルサインオンを使用する場合には適用されません。

注：`Password-Control`セクションは複数回宣言しないでください。

```
[Password-Control]
	Min-Length=8
	Require-Uppercase=true
	Require-Lowercase=true
	Require-Special=true
	Require-Special=true
```

**MinLength**
デフォルト値：0
例：`MinLength = 8`
説明：`MinLength`は、パスワードの最小文字長を指定します。

**Require-Uppercase**
デフォルト値：false
例：`Require-Uppercase = true`
説明：`Require-Uppercase`が設定されている場合、パスワードには少なくとも1つの大文字が含まれている必要があります。

**Require-Lowercase**
デフォルト値：false
例：`Require-Lowercase = true`
説明：`Require-Lowercase`が設定されている場合、パスワードには少なくとも1つの小文字が含まれている必要があります。

**Require-Number**
デフォルト値：false
例：`Require-Number = true`
説明：「Require-Number」が設定されている場合、パスワードには少なくとも1桁の数字が含まれている必要があります。

**Require-Special**
デフォルト値：false
例：`Require-Special = true`
説明：「Require-Special」が設定されている場合、パスワードには少なくとも1つの「特殊」文字が含まれている必要があります。特殊文字のセットには、数字ではないすべてのUnicode文字が含まれます

##ウェル構成

このセクションのパラメータは、`Default-Well`の設定を含め、ウェルの設定に適用されます。ウェル構成は*インデクサー*にのみ適用されます。つまり、これらのパラメータ設定はウェブサーバ－からは無視されます。以下は、`Default-Well`と"pcap"という名前の別のウェル構成の2つのウェル構成のサンプルです。

```
[Default-Well]
	Location=/opt/gravwell/storage/default/
	Cold-Location=/opt/gravwell/cold-storage/default
	Accelerator-Name=fulltext
	Accelerator-Engine-Override=bloom
	Max-Hot-Storage-GB=20

[Storage-Well "pcap"]
	Location=/opt/gravwell/storage/pcap
	Cold-Location=/opt/gravwell/cold-storage/pcap
	Hot-Duration=1D
	Cold-Duration=12W
	Delete-Frozen-Data=true
	Max-Hot-Storage-GB=20
	Disable-Compression=true
	Tags=pcap
```

構成ファイルには、`Default-Well`セクションが1つだけ含まれている必要があります。また、1つ以上の`Storage-Well`セクションが含まれる場合もあります。

ウェルがホット、コールド、およびアーカイブストレージ間でエントリを移動する方法の詳細については、[ageoutドキュメント](ageout.md)を参照してください。

注：`Default-Well`に`Tags=`仕様を含めることはできません。代わりに、デフォルトのウェルにはすべてのタグが含まれています*他のウェルには含まれていません*

**Location**
デフォルト値：`Default-Well`の場合は`/opt/gravwell/storage/ default`、`Storage-Well`の場合はnone
例：`Location = /opt/gravwell/storage/foo`
説明：このパラメーターは、ウェルが「ホット」データを格納する場所を制御します。2つのウェルが同じディレクトリを指すことはできません。

### エイジアウトオプション

**Hot-Duration**
デフォルト値：
例：`Hot-Duration = 1w`
説明：このパラメーターは、データが「ホット」ストレージの場所に保持される期間を決定します。値は、数値とそれに続く接尾辞（「日」を表す「d」または「週」を表す「w」）で構成する必要があります。したがって、 `Hot-Duration = 30d`は、データを30日間保持する必要があることを示します。`Cold-Location`パラメータが指定されているか、`Delete-Cold-Data`がtrueに設定されていない限り、データは実際にはホットストレージから移動されないことに注意してください。

**Cold-Location**
デフォルト値：
例：`Cold-Location = /opt/gravwell/cold_storage/foo`
説明：このパラメーターは、コールドデータ（`Location`で指定されたホットストアから移動されたデータ）の保存場所を設定します。

**Cold-Duration**
デフォルト値：
例：`Cold-Duration = 365d`
説明：このパラメーターは、データが「コールド」保管場所に保持される期間を決定します。`Delete-Frozen-Data`がtrueに設定されていない限り、データは実際にはコールドストレージから移動されません。

**Max-Hot-Storage-GB**
デフォルト値：
例：`Max-Hot-Storage-GB = 100`
説明：このパラメーターは、特定のウェルのホットストレージの最大ディスク消費量をギガバイト単位で設定します。この数を超えると、最も古いデータがコールドストレージに移行されるか（可能な場合）、クラウドストレージに送信されるか（構成されている場合）、または削除されます（許可されている場合）。

**Max-Cold-Storage-GB**
デフォルト値：
例：`Max-Cold-Storage-GB = 100`
説明：このパラメーターは、特定のウェルのコールドストレージの最大ディスク消費量をギガバイト単位で設定します。この数を超えると、最も古いデータがクラウドストレージに送信され（構成されている場合）、削除されます（許可されている場合）。

**Hot-Storage-Reserve**
デフォルト値：
例：`Hot-Storage-Reserve = 10`
説明：このパラメーターは、ディスク上に少なくとも特定のパーセンテージを空けておく必要があることをウェルに通知します。したがって、 `Hot-Storage-Reserve = 10`が設定されている場合、ディスクの使用率が90％に達すると、ウェルはホットストレージからデータをエージングアウトしようとします。

**Cold-Storage-Reserve**
デフォルト値：
例：`Cold-Storage-Reserve = 10`
説明：このパラメーターは、ディスク上に少なくとも特定のパーセンテージを空けておく必要があることをウェルに通知します。したがって、 `Cold-Storage-Reserve = 10`が設定されている場合、ディスクの使用率が90％に達すると、ウェルはコールドストレージからデータをエージングアウトしようとします。

**Delete-Cold-Data**
デフォルト値：false
例：`Delete-Cold-Data = true`
説明：このパラメーターをtrueに設定すると、エージングアウト基準の1つが満たされたときに、ホットストレージの場所からデータを削除できることを意味します。

**Delete-Frozen-Data**
デフォルト値：false
例：`Delete-Frozen-Data = true`
説明：このパラメーターをtrueに設定すると、エージングアウト基準の1つが満たされたときに、データを冷蔵場所から削除できることを意味します。

**Archive-Deleted-Shards**
デフォルト値：false
例：`Archive-Deleted-Shards = true`
説明：このオプションが設定されている場合、ウェルはシャードを削除する前に外部アーカイブサーバーにアップロードしようとします。これは、`[Cloud-Archive]`セクションが構成されている場合にのみ機能することに注意してください。

**Disable-Compression、Disable-Hot-Compression、Disable-Cold-Compression**
デフォルト値：false
例：`Disable-Compression = true`
説明：これらのパラメーターは、ウェル内のデータのユーザーモード圧縮を制御します。デフォルトでは、Gravwellはウェル内のデータを圧縮します。 `Disable-Hot-Compression`または` Disable-Cold-Compression`を設定すると、それぞれホットストレージまたはコールドストレージで無効になります。`Disable-Compression`を設定すると、両方で無効になります。

**Enable-Transparent-Compression、Enable-Hot-Transparent-Compression、Enable-Cold-Transparent-Compression**
デフォルト値：false
例：`Enable-Transparent-Compression = true`
説明：これらのパラメーターは、ウェル内のデータのカーネルレベルの透過的な圧縮を制御します。有効にすると、Gravwellは`btrfs`ファイルシステムにデータを透過的に圧縮するように指示できます。これは、ユーザーモードの圧縮よりも効率的です。`Enable-Transparent-Compression`をtrueに設定すると、ユーザーモードの圧縮が自動的にオフになります。 `Disable-Compression = true`を設定すると、透過圧縮が**無効**になることに注意してください。

**Ageout-Time-Override**
デフォルト値：
例：`Ageout-Time-Override =" 3：00 AM "`
説明：このパラメーターを使用すると、ageoutルーチンを実行する特定の時間を指定できます。通常、これは必要ありません。

### 加速オプション

**Accelerator-Name**
デフォルト値：
例：`Accelerator-Name = json`
説明：`Accelerator-Name`パラメーター（および` Accelerator-Args`パラメーター）を設定すると、ウェルでの加速が有効になります。詳細については、[アクセラレータのドキュメント](#!configuration/accelerators.md)を参照してください。

**Accelerator-Args**
デフォルト値：
例：`Accelerator-Args="username hostname \"strange-field.with.specials\".subfield"`
説明 `Accelerator-Args`パラメーター（および`Accelerator-Name`パラメーター）を設定すると、ウェルでの加速が有効になります。詳細については、[アクセラレータのドキュメント](#!configuration/accelerators.md)を参照してください。

**Accelerate-On-Source**
デフォルト値：false
例：`Accelerate-On-Source = true`
説明：各モジュールのSRCフィールドを含める必要があることを指定します。これにより、CEFなどのモジュールをSRCと組み合わせることができます。

**Accelerator-Engine-Override**
デフォルト値："index"
例：`Accelerator-Engine-Override = Bloom`
説明：使用する加速エンジンを選択します。デフォルトでは、インデックス作成アクセラレータが使用されます。このパラメーターを"bloom" に設定すると、代わりにブルームフィルターが選択されます。

**Collision-Rate**
デフォルト値：0.001
例：`Collision-Rate = 0.01`
説明：ブルームフィルター加速エンジンの精度を設定します。0.1〜0.000001の値である必要があります。

### 一般オプション

**Disable-Replication**
デフォルト値：false
例：`Disable-Replication = true`
説明：設定されている場合、このウェルの内容は複製されません。

**Enable-Quarantine-Corrupted-Shards**
デフォルト値：false
例：`Enable-Quarantine-Corrupted-Shards = true`
説明：設定されている場合、回復できない破損したシャードは、後で分析するために検疫場所にコピーされます。デフォルトでは、ひどく破損したシャードが削除される場合があります。

## レプリケーション構成

`[Replication]`セクションは、 [Gravwellのレプリケーション機能](#!configuration/replication.md)を構成します。 構成例は次のようになります。

```
[Replication]
	Disable-Server=true
	Peer=10.0.01
	Storage-Location=/opt/gravwell/replication_storage
```

レプリケーション構成ブロックは、インデクサーにのみ適用されます。

**Peer**
デフォルト値：
例：`Peer = 10.0.0.1：9406`
説明：`Peer`パラメーターはレプリケーションピアを指定します。IPまたはホスト名を取り、最後にオプションのポートを付けます。ポートが指定されていない場合は、デフォルトのポート（9406）が使用されます。`Peer`は複数回指定できます。

**Listen-Address**
デフォルト値："：9406"
例：`Listen-Address = 192.168.1.1：9406`
説明：このパラメーターは、Gravwellが*着信*レプリケーション接続をリッスンするIPとポートを定義します。デフォルトでは、ポート9406のすべてのインターフェイスでリッスンします。

**Storage-Location**
デフォルト値：
例：`Storage-Location = /opt/gravwell/Replication`
説明：他のGravwellインデクサーから複製されたデータの保存場所を設定します。

**Max-Replicated-Data-GB**
デフォルト値：
例：`Max-Replicated-Data-GB = 100`
説明：保存する複製データの最大量をギガバイト単位で設定します。これを超えると、インデクサーはレプリケートされたデータのウォークを開始してクリーンアップします。最初に元のインデクサーで削除されたシャードを削除し、次に最も古いシャードの削除を開始します。ストレージサイズが制限を下回ると、削除は停止します。

**Replication-Secret-Override**
デフォルト値：
例：`Replication-Secret-Override = MyReplicationSecret`
説明：デフォルトでは、Gravwellは `Control-Auth`トークンを使用してレプリケーションの認証を行います。このパラメーターを設定すると、代わりにカスタムレプリケーション認証トークンが定義されます。

**Disable-TLS**
デフォルト値：false
例：`Disable-TLS = true`
説明：このパラメーターをtrueに設定すると、レプリケーションのTLSが無効になります。インデクサーは暗号化されていない着信接続をリッスンし、暗号化されていない接続を使用してピアと通信します。

**Key-File**
デフォルト値：(`[Global]`セクションの`Key-File`の値)
例：`Key-File = /opt/gravwell/etc/replication-key.pem`
説明：このパラメーターを使用すると、グローバルに定義されたキーではなく、TLS接続に個別のキーを使用できます。

**Certificate-File**
デフォルト値：(`[Global]`セクションの`Certificate-File`の値)
例：`Certificate-File = /opt/gravwell/etc/replication-cert.pem`
説明：このパラメーターを使用すると、グローバルに定義された証明書ではなく、TLS接続に個別の証明書を使用できます。

**Insecure-Skip-TLS-Verify**
デフォルト値：false
例：`Insecure-Skip-TLS-Verify = false`
説明：このパラメーターをtrueに設定すると、レプリケーションピアに接続するときにTLS証明書の検証が無効になります。

**Connect-Wait-Timeout**
デフォルト値：30
例：`Connect-Wait-Timeout = 60`
説明：レプリケーションピアに接続するときに使用されるタイムアウトを秒単位で構成します。

**Disable-Server**
デフォルト値：false
例：`Disable-Server = true`
説明：レプリケーション*サーバー*機能を無効にします。設定されている場合、インデクサーは自身のデータをレプリケーションピアにプッシュしますが、他のインデクサーがそのデータにプッシュすることはできません。

**Disable-Compression**
デフォルト値：false
例：`Disable-Compression = true`
説明：レプリケートされたデータの圧縮を制御します。デフォルトでは、レプリケートされたデータはディスクに圧縮されます。

**Enable-Transparent-Compression**
デフォルト値：false
例：`Enable-Transparent-Compression = true`
説明：このパラメーターがtrueに設定されている場合、Gravwellはレプリケートされたデータに対してbtrfs透過圧縮を使用しようとします。 `Disable-Compression = true`を設定すると、これが無効になります。

##シングルサインオン構成

`[SSO]`構成セクションは、Gravwellウェブサーバ－のシングルサインオンオプションを制御します。サンプルセクションは、次のように単純にすることができます。

```
[SSO]
	Gravwell-Server-URL=https://10.10.254.1:8080
	Provider-Metadata-URL=https://sso.gravwell.io/FederationMetadata/2007-06/FederationMetadata.xml
```

ただし、より頻繁に追加の構成が必要になります。

```
[SSO]
	Gravwell-Server-URL=https://10.10.254.1:8080
	Provider-Metadata-URL=https://sso.gravwell.io/FederationMetadata/2007-06/FederationMetadata.xml
	Groups-Attribute=http://schemas.xmlsoap.org/claims/Group
	Group-Mapping=Gravwell:gravwell-users
	Group-Mapping=TestGroup:testgroup
	Username-Attribute = "uid"
	Common-Name-Attribute = "cn"
	Given-Name-Attribute  = "givenName"
	Surname-Attribute = "sn"
	Email-Attribute = "mail"
```

詳細については、[SSO構成ドキュメント](sso.md)を参照してください。

**Gravwell-Server-URL**
デフォルト値：
例：`Gravwell-Server-URL = https//gravwell.example.org/`
説明：SSOサーバーがユーザーを認証した後にユーザーがリダイレクトされるURLを指定します。これは、Gravwellサーバーのユーザー向けのホスト名またはIPアドレスである必要があります。このパラメーターは必須です。

**Provider-Metadata-URL**
デフォルト値：
例：`Provider-Metadata-URL = https://sso.example.org/ FederationMetadata/2007-06/FederationMetadata.xml`
説明：SSOサーバーのXMLメタデータのURLを指定します。上記のパス(`/FederationMetadata/2007-06/FederationMetadata.xml`)はAD FSサーバーでは機能するはずですが、他のSSOプロバイダーでは調整する必要がある場合があります。このパラメーターは必須です。

**Insecure-Skip-TLS-Verify**
デフォルト値：false
例：`Insecure-Skip-TLS-Verify = true`
説明：trueに設定されている場合、このパラメーターは、SSOサーバーと通信するときに無効なTLS証明書を無視するようにGravwellに指示します。このオプションは注意して設定してください。

**Username-Attribute**
デフォルト値：「http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn」
例：`Username-Attribute =" uid "`
説明：ユーザー名を含むSAML属性を定義します。Shibbolethサーバーでは、代わりにこれを"uid"に設定する必要があります。

**Common-Name-Attribute**
デフォルト値：「http://schemas.xmlsoap.org/claims/CommonName」
例：`Common-Name-Attribute="cn"`
説明：ユーザーの「共通名」を含むSAML属性を定義します。 Shibbolethサーバーでは、代わりにこれを「cn」に設定する必要があります。

**Given-Name-Attribute**
デフォルト値：「http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname」
例：`Given-Name-Attribute ="giveName"`
説明：ユーザーの名を含むSAML属性を定義します。Shibbolethサーバーでは、代わりにこれを"givenName"に設定する必要があります。

**Surname-Attribute**
デフォルト値：「http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname」
例： `Surname-Attribute = sn`
説明：ユーザーの姓を含むSAML属性を定義します。Shibbolethサーバーでは、代わりにこれを"sn"に設定する必要があります。

**Email-Attribute**
デフォルト値：「http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress」
例：`Email-Attribute =" mail "`
説明：ユーザーの電子メールアドレスを含むSAML属性を定義します。 Shibbolethサーバーでは、代わりに「メール」に設定する必要があります。

**Groups-Attribute**
デフォルト値：「http://schemas.microsoft.com/ws/2008/06/identity/claims/groups」
例：`Groups-Attribute =" groups "`
説明：ユーザーが属するグループのリストを含むSAML属性を定義します。通常、グループリストを送信するようにSSOプロバイダーを明示的に構成する必要があります。

**Group-Mapping**
デフォルト値：
例：`Group-Mapping = Gravwell：gravwell-users`
説明：ユーザーのグループメンバーシップにリストされている場合に自動的に作成される可能性のあるグループの1つを定義します。これは、複数のグループを許可するために複数回指定できます。引数は、コロンで区切られた2つの名前で構成する必要があります。1つ目はグループのSSOサーバー側の名前（通常はAD FSの名前、AzureのUUIDなど）で、2つ目はGravwellが使用する名前です。したがって、`Group-Mapping = Gravwell Users：gravwell-users`を定義する場合、グループ「Gravwell Users」のメンバーであるユーザーのログイントークンを受け取ると、「gravwell-users」という名前のローカルグループが作成されます。 "そしてそれにユーザーを追加します。

##クラウドアーカイブ構成

Gravwellは、個々のウェルの「Archive-Deleted-Shards」パラメーターを使用して、データシャードを削除する前にリモートのクラウドアーカイブサーバーにアーカイブするように構成できます。`[Cloud-Archive]`構成セクションは、これを有効にするためのクラウドアーカイブサーバーに関する情報を定義します。

```
[Cloud-Archive]
	Archive-Server=10.0.0.2:443
	Archive-Shared-Secret=MyArchiveSecret
```

クラウドアーカイブ構成は、インデクサーにのみ適用されます。

**Archive-Server**
デフォルト値：
例：`Archive-Server = cloudarchive.example.org：443`
説明：このパラメーターは、クラウドアーカイブサーバーのIP/ホスト名とオプションでポートを指定します。ポートが指定されていない場合、デフォルト（443）が使用されます。

**Archive-Shared-Secret**
デフォルト値：
例：`Archive-Shared-Secret = MyArchiveSecret`
説明：クラウドアーカイブサーバーへの認証時に使用する共有シークレットを設定します。インデクサーは、認証プロセスの残りの半分としてライセンスの顧客ID番号を使用します。

**Insecure-Skip-TLS-Verify**
デフォルト値：false
例：`Insecure-Skip-TLS-Verify = true`
説明：trueに設定すると、インデクサーは接続時にCloudArchiveサーバーのTLS証明書を検証しません。