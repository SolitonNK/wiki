# グローバル設定パラメータ

各パラメーターには、パラメーターが空の場合、または構成ファイルで指定されていない場合に適用されるデフォルト値があります。

**インデクサー-UUID **
適用対象：インデクサー
デフォルト値：[設定されていない場合はランダムに生成]
例： `Indexer-UUID ="ecdeeff8-8382-48f1-a24f-79f83af00e97"`
説明：特定のインデクサーに一意の識別子を設定します。 2つのインデクサーが同じUUIDを持つことはできません。 このパラメーターが設定されていない場合、インデクサーはUUIDを生成し、構成ファイルに書き込みます。 これは通常、インデクサーが壊滅的な障害を起こし、レプリケーションから再構築される場合を除き、最良の選択です（[レプリケーションドキュメント](configuration/replication.md)を参照）。 このパラメーターを変更する前によく考えてください。

** Webserver-UUID **
適用対象：Webサーバー
デフォルト値：[設定されていない場合はランダムに生成]
例：`Webserver-UUID="ecdeeff8-8382-48f1-a24f-79f83af00e97"`
説明：特定のWebサーバーの一意の識別子を設定します。 2つのWebサーバーが同じUUIDを持つことはできません。 このパラメーターが設定されていない場合、WebサーバーはUUIDを生成し、それを設定ファイルに書き戻します*。 通常、これが最良の選択です。 このパラメーターを変更する前によく考えてください。

**ライセンスの場所**
適用対象：インデクサーとWebサーバー
デフォルト値：`/opt/gravwell/etc/license`
例： `License-Location=/opt/gravwell/etc/my_license`
説明：gravwellライセンスファイルへのパスを設定します。このパスはgravwellユーザーとグループが読み取り可能である必要があります。

**構成場所**
適用対象：インデクサーとWebサーバー
デフォルト値：`/opt/gravwell/etc`
例： `Config-Location=/tmp/path/to/etc`
説明：構成の場所により、他のすべての構成パラメーターを格納する代替の場所を指定できます。 代替のConfig-Locationを指定すると、他のすべてのパラメーターを代替パスで指定することなく、単一のパラメーターを設定できます。

**ウェブポート**
適用対象：Webサーバー
デフォルト値： `443`
例： `Web-Port=80`
説明：Webサーバーのlistenポートを指定します。 Web-Portパラメーターを80に設定しても、WebサーバーはHTTPのみのモードに切り替わりません。 Insecure-Disable-HTTPS設定が必要です。

** Disable-HTTP-Redirector **
適用対象：Webサーバー
デフォルト値： `False`
例： `Disable-HTTP-Redirector=true`
説明：デフォルトでは、GrabwellはHTTPリダイレクタを起動し、クリアテキストHTTPポータルを要求するクライアントを暗号化されたHTTPSポータルにリダイレクトします。

** Insecure-Disable-HTTPS **
適用対象：Webサーバー
デフォルト値： `false`
例： `Insecure-Disable-HTTPS=true`
説明：デフォルトでは、GravwellはHTTPSモードで動作します。`Insecure-Disable-HTTPS=true`を設定すると、代わりにGravwellはプレーンテキストHTTPを使用し、`Web-Port`でリッスンします。

** Control-Listen-Address **
適用対象：インデクサー
デフォルト値：
例： `Control-Listen-Address=”10.0.0.1”`
説明：Control-Listen-Addressパラメーターは、インデクサーのコントロールリスナーを特定のアドレスにバインドできます。 デュアルホームマシン、または高速データネットワークと低速制御ネットワークを備えたマシンにGravwellをインストールする場合、インデクサーを特定のアドレスにバインドして、トラフィックが適切にルーティングされるようにすることができます。

**コントロールポート**
適用対象：インデクサーとWebサーバー
デフォルト値： `9404`
例： `Control-Port=12345`
説明：Control-Portパラメーターは、インデクサーがWebサーバーからの制御コマンドをリッスンするポートを選択します。 この設定は、インデクサーとWebサーバーが通信するために同じでなければなりません。 インストーラーはデフォルトでインデクサーのバインド機能を設定しないため、ポートは1024より大きい値に設定する必要があります。単一のマシンで複数のインデクサーが実行されている環境、または別のアプリケーションが存在する環境では、制御ポートの調整が必要になる場合があります ポート9404へのバインド。

** Datastore-Listen-Address **
適用対象：データストア
デフォルト値：
例： `Datastore-Listen-Address=10.0.0.1`
説明：Datastore-Listen-Addressパラメーターは、特定のアドレスでのみリッスンするようにデータストアに指示します。 デフォルトでは、データストアはシステム上のすべてのアドレスをリッスンします。

**データストアポート**
適用対象：データストア
デフォルト値： `9405`
例： `Datastore-Port=7888`
説明：Datastore-Portパラメーターは、データストアが通信するポートを選択します。 ポートは1024より大きい必要があります。通常、デフォルト値の9405はほとんどのインストールに適しています。

**データストア**
適用対象：Webサーバー
デフォルト値：
例： `Datastore=10.0.0.1:9405`
説明：Datastoreパラメーターは、Webサーバーがデータストアに接続して、ダッシュボード、リソース、ユーザー設定、および検索履歴を同期することを指定します。 これにより、[分散Webサーバー](distributed/frontend.md)が許可されますが、必要な場合にのみ設定する必要があります。 デフォルトでは、ウェブサーバーはデータストアに接続しません。

**データストア更新間隔**
適用対象：Webサーバー
デフォルト値： `10`
例： `Datastore-Update-Interval = 5`
説明：Datastore-Update-Intervalパラメーターは、更新のためにデータストアをチェックする前にWebサーバーが待機する時間（秒単位）を決定します。 通常、デフォルト値の10秒が適切です。

**データストアの更新間隔**
適用対象：Webサーバー
デフォルト値： `10`
例： `Datastore-Update-Interval=5`
説明：Datastore-Update-Intervalパラメーターは、更新のためにデータストアをチェックする前にWebサーバーが待機する時間（秒単位）を決定します。通常、デフォルト値の10秒が適切です。

** Datastore-Insecure-Disable-TLS **
適用対象：Webサーバーとデータストア
デフォルト値： `false`
例：`Datastore-Insecure-Disable-TLS=true`
説明：Datastore-Insecure-Disable-TLSパラメーターは、Webサーバーとデータストアの両方で使用されます。 デフォルトでは、データストアはWebサーバーからの着信HTTPS接続をリッスンします。 このパラメーターをfalseに設定すると、データストアはプレーンテキストHTTPを想定し、WebサーバーにHTTPを使用するよう指示します。

** Datastore-Insecure-Skip-TLS-Verify **
適用対象：Webサーバー
デフォルト値： `false`
例： `Datastore-Insecure-Skip-TLS-Verify=true`
説明：Datastore-Insecure-Skip-TLS-Verifyパラメーターは、データストアへの接続時に無効なTLS証明書を無視するようWebサーバーに指示します。 これは自己署名証明書を使用する場合に必要ですが、可能な場合は避ける必要があります。

**外部アドレス**
適用対象：Webサーバー
デフォルト値：
例： `External-Addr=10.0.0.1:443`
説明：External-Addrパラメーターは、このWebサーバーに接続するために他のWebサーバーが使用する必要があるアドレスを指定します。 データストアを使用する場合、このパラメーターは**必須**です。これは、あるWebサーバー上のユーザーが別のWebサーバーで実行された検索結果をロードできるようにするためです。

** Search-Forwarding-Insecure-Skip-TLS-Verify **
適用対象：Webサーバー
デフォルト値： `false`
例： `Search-Forwarding-Insecure-Skip-TLS-Verify=true`
説明：このパラメーターは、データストアを使用して分散モードで複数のWebサーバーを操作する場合にのみ役立ちます。 Webサーバーに自己署名証明書がある場合、このパラメーターがtrueに設定されていない限り、ユーザーはリモートWebサーバーから検索にアクセスできません。

**取り込みポート**
適用対象：インデクサー
デフォルト値： `4023`
例： `Ingest-Port=14023`
説明：Ingest-Portパラメーターは、インデクサーがingester接続をリッスンするポートを制御します。 Ingest-Portパラメーターの変更は、単一のマシンで複数のインデクサーを実行している場合、または別のアプリケーションが既にデフォルトポートの4023にバインドされている場合に役立ちます。

** TLS-Ingest-Port **
適用対象：インデクサー
デフォルト値： `4024`
例： `TLS-Ingest-Port=14024`
説明：TLS-Ingest-Portパラメーターは、インジェスター接続のためにインデクサーがリッスンするポートを制御します。 TLS-Ingest-Portパラメーターの変更は、単一のマシンで複数のインデクサーを実行する場合、または別のアプリケーションがデフォルトポート4024に既にバインドされている場合に役立ちます。デフォルトでは、TLSトランスポートを使用するすべてのインジェスターがリモート証明書を検証します。 展開で自動生成された証明書を使用している場合、インジェスターは証明書を信頼済みとしてインストールするか、証明書の検証を無効にする必要があります（これにより、TLSトランスポートによって提供される保護が事実上無効になります）。

** Pipe-Ingest-Path **
適用対象：インデクサー
デフォルト値： `/opt/gravwell/comms/pipe`
例： `Pipe-Ingest-Path=/tmp/path/to/pipe`
説明：Pipe-Ingest-Pathは、Unix名前付きパイプへのフルパスを指定します。 インデクサーは名前付きパイプを作成し、共存するインジェスターはパイプに接続して、非常に高速で低遅延のトランスポートとして使用できます。 名前付きパイプは、1ギガビット以上で動作するネットワークパケットingesterなど、非常に高いパフォーマンスを必要とするインジェスターに最適です。 名前付きパイプを使用して、異常なネットワークトランスポートまたは非常に高速な非IPベースの相互接続を介したトランスポートを促進することもできます。

** Pipe-Ingest-Path **
適用対象：インデクサー
デフォルト値：  `/opt/gravwell/log`
例： `Log-Location=/tmp/path/to/logs`
説明：Pipe-Ingest-Pathは、Unix名前付きパイプへのフルパスを指定します。 インデクサーは名前付きパイプを作成し、共存するインジェスターはパイプに接続して、非常に高速で低遅延のトランスポートとして使用できます。 名前付きパイプは、1ギガビット以上で動作するネットワークパケットingesterなど、非常に高いパフォーマンスを必要とするインジェスターに最適です。 名前付きパイプを使用して、異常なネットワークトランスポートまたは非常に高速な非IPベースの相互接続を介したトランスポートを促進することもできます。

**Web-Log-Location**
Applies to:        Webserver
Default Value:        `/opt/gravwell/log/web`
Example:        `Web-Log-Location=/tmp/path/to/logs/web`
Description:        The Web-Log-Location parameter controls where webserver logs are stored.  Gravwell does not feed its own logs directly into indexers, and instead writes them to files (use the file follower ingester if you want to ingest Gravwell logs too).  This parameter specifies where those logs go.

**ウェブログの場所**
適用対象：Webサーバー
デフォルト値： `/opt/gravwell/log/web`
例： `Web-Log-Location= /tmp/path/to/logs/web`
説明：Web-Log-Locationパラメーターは、Webサーバーログの保存場所を制御します。 Gravwellは独自のログをインデクサーに直接フィードせず、代わりにそれらをファイルに書き込みます（Gravwellログも取り込む場合は、ファイルフォロワーを使用します）。 このパラメーターは、これらのログの保存先を指定します。

** Datastore-Log-Location **
適用対象：データストア
デフォルト値： `/opt/gravwell/log/datastore`
例： `Datastore-Log-Location=/tmp/path/to/logs/datastore`
説明：Datastore-Log-Locationパラメーターは、データストアログの保存場所を制御します。

**ログレベル**
適用対象：インデクサー、データストア、およびWebサーバー
デフォルト値： `INFO`
例： `Log-Level=ERROR`
説明：Log-Levelパラメーターは、gravwellインフラストラクチャからのログの詳細度を制御します。 Log-Levelには、INFO、WARN、およびERRORの3つの利用可能な引数があります。 INFOが最も詳細で、ERRORが最小です。 ロギングシステムは、ロギングレベルごとにファイルを生成し、syslogデーモンと同様の方法でローテーションします。

** Disable-Access-Log **
適用対象：Webサーバー
デフォルト値： `false`
例： `Disable-Access-Log=true`
説明：Disable-Access-Logパラメーターは、Webサーバーによって生成されたアクセスログを無効にするために使用されます。アクセスロギングインフラストラクチャは、個々のページアクセスを記録します。 Gravwellのアクセスを監査し、潜在的な問題をデバッグするためにこれらのアクセスログを保持することは通常重要ですが、多くのユーザーがいる環境ではアクセスログが大きくなる可能性があるため、無効にすることが望ましい場合があります。

**永続的なWebログイン**
適用対象：Webサーバー
デフォルト値： `true`
例： `Persist-Web-Logins=false`
説明：Persist-Web-Loginsパラメーターは、シャットダウン時にユーザーセッションを不揮発性ストレージに保存する必要があることをWebサーバーに通知するために使用されます。デフォルトでは、ウェブサーバーがシャットダウンまたは再起動されると、クライアントセッションが保持されます。 Persist-Web-Loginsをfalseに設定すると、Webサーバーが再起動されるたびにセッションが無効になります。

**セッションタイムアウト分**
適用対象：Webサーバー
デフォルト値： `60`
例： `Session-Timeout-Minutes=1440`
説明：Session-Timeout-Minutesパラメーターは、Webサーバーがセッションを破棄する前にクライアントがアイドル状態でいられる時間を制御します。たとえば、クライアントがログアウトせずにブラウザを閉じた場合、システムはセッションを無効にする前に指定された期間待機します。インストーラーは、この値をデフォルトで1日に設定します。

**キーファイル**
適用対象：インデクサー、データストア、およびWebサーバー
デフォルト値： `/opt/gravwell/etc/key.pem`
例： `Key-File=/opt/gravwell/etc/privkey.pem`
説明：Key-Fileパラメーターは、Webserver、データストア、およびインデクサーの秘密キーとして使用されるファイルを制御します。秘密/公開鍵はPEM形式でエンコードする必要があります。秘密鍵は保護する必要があり、侵害された場合は破棄して再発行する必要があります。詳細については、http：//www.tldp.org/HOWTO/SSL-Certificates-HOWTO/x64.htmlを参照してください。

**証明書ファイル**
適用対象：インデクサー、データストア、およびWebサーバー
デフォルト値： `/opt/gravwell/etc/cert.pem`
例： `Certificate-File=/opt/gravwell/etc/cert.pem`
説明：Certificate-Fileパラメーターは、TLSトランスポートに使用される公開/秘密キーペアの公開キーコンポーネントを指定します。公開鍵はすべてのInesterおよびWebクライアントに配信され、機密とはみなされません。 Gravwellは、公開鍵がPEM形式でエンコードされ、公開鍵部分のみが含まれることを期待しています。

** Ingest-Auth **
適用対象：インデクサー
デフォルト値： `IngestSecrets`
例： `Ingest-Auth=abcdefghijklmnopqrstuvwxyzABCD`
説明：Ingest-Authパラメーターは、インジェスターをインデクサーに対して認証するために使用される共有秘密トークンを指定します。このトークンの長さは任意です。 Gravwellでは、少なくとも24文字の高エントロピートークンを推奨しています。デフォルトでは、インストーラーはランダムトークンを生成します。

** Control-Auth **
適用対象：インデクサーとWebサーバー
デフォルト値： `ControlSecrets`
例： `Control-Auth=abcdefghijklmnopqrstuvwxyzABCD`
説明：Control-Authパラメーターは、Webサーバーに対するインジェスターの認証、およびその逆の認証に使用される共有秘密トークンを指定します。このトークンの長さは任意です。 Gravwellでは、少なくとも24文字の高エントロピートークンを推奨しています。デフォルトでは、インストーラーはランダムトークンを生成します。

** User-DB-Path **
適用対象：Webサーバー
デフォルト値：  `/opt/gravwell/etc/users.db`
例： `User-DB-Path=/tmp/path/to/users.db`
説明：User-DB-Pathパラメーターは、ユーザーデータベースファイルの場所を指定します。ユーザーデータベースファイルには、ユーザーとグループの構成が含まれています。ユーザーデータベースはbcryptハッシュアルゴリズムを使用してパスワードを保存および検証します。これは非常に堅牢であると考えられていますが、users.dbファイルは引き続き保護する必要があります。デフォルトでは、インストーラーはユーザーデータベースファイルのファイルシステムパーミッションをGravwellユーザーとグループのみが読み取り可能に設定します。

** Datastore-User-DB-Path **
適用対象：データストア
デフォルト値： `/opt/gravwell/etc/datastore-users.db`
例： `Datastore-User-DB-Path=/tmp/path/to/datastore-users.db`
説明：Datastore-User-DB-Pathパラメーターは、データストアコンポーネントによって管理されるユーザーデータベースファイルの場所を指定します。これはUser-DB-Pathパラメーターで指定されたものと同じパスであってはなりません**。

** Web-Store-Path **
適用対象：Webサーバー
デフォルト値： `/opt/gravwell/etc/webstore.db`
例： `Web-Store-Path=/tmp/path/to/webstore.db`
説明：Web-Store-Pathは、検索履歴、ダッシュボード、ユーザー設定、ユーザーセッション、その他のさまざまなユーザーデータを保存するために使用されるデータベースファイルを指します。ウェブストアデータベースファイルにはユーザー認証情報は含まれていませんが、*ユーザーセッションCookieとCSRFトークンが含まれています。 GravwellはCookieとCSRFトークンをオリジンに結び付けているため、攻撃者が盗まれたCookieまたはトークンとして再利用するリスクは低いですが、データストアを保護する必要があります。インストーラーは、Gravwellユーザーによる読み取り/書き込みのみを許可するようにファイルシステムのアクセス許可を設定します。

** Datastore-Web-Store-Path **
適用対象：データストア
デフォルト値： `/opt/gravwell/etc/datastore-webstore.db`
例： `Datastore-Web-Store-Path=/tmp/path/to/datastore-webstore.db`
説明：Datastore-Web-Store-Pathパラメーターは、検索履歴、ダッシュボード、ユーザー設定を保存するためにデータストアが使用するデータベースファイルを指します。これは、Web-Store-Pathパラメーターで指定されたパスと同じであってはなりません**。

** Web-Listen-Address **
適用対象：Webサーバー
デフォルト値：
例： `Web-Listen-Address=10.0.0.1`
説明：Web-Listen-Addressパラメーターは、Webサーバーがバインドしてサービスを提供するアドレスを指定します。デフォルトでは、パラメーターは空です。つまり、Webサーバーはすべてのインターフェイスとアドレスにバインドします。

**ログイン失敗ロック数**
適用対象：Webサーバー
デフォルト値： `5`
例： `Login-Fail-Lock-Count=10`
説明：Login-Fail-Lock-Countパラメーターは、アカウントでブルートフォース保護が有効になる前に、ユーザーアカウントに対して連続して失敗するログインの回数を指定します。たとえば、値が4に設定され、ユーザーが不正なパスワードを連続して4回入力すると、追加のログイン試行が完了するまでに時間がかかり、攻撃者の速度が低下します。注：Gravwellは以前、特定の回数の失敗後にアカウントをロックしていました。現在では、より積極的なブルートフォース保護が行われていますが、従来の理由により、構成パラメーターは「ロック」名を保持しています。

**ログイン失敗ロック期間**
適用対象：Webサーバー
デフォルト値： `5`
例： `Login-Fail-Lock-Duration=10`
説明：Login-Fail-Lock-Durationパラメーターは、Login-Fail-Lock-Countを超えたかどうかを計算するときに使用されるウィンドウ（分単位）を指定します。注：Gravwellは以前、特定の回数の失敗後にアカウントをロックしていました。現在では、より積極的なブルートフォース保護が行われていますが、従来の理由により、構成パラメーターは「ロック」名を保持しています。

**リモートインデクサー**
適用対象：Webサーバー
デフォルト値：  `net:10.0.0.1:9404`
例：  `Remote-Indexers=net:10.0.0.1:9404`
説明：Remote-Indexersパラメーターは、Webサーバーが接続して制御するリモートインデクサーのアドレスとポートを指定します。 Remote-Indexersはリストパラメーターです。つまり、複数のリモートインデクサーを提供するために何度も指定できます。 Gravwell Clusterエディションでは、クラスター内の各インデクサーを指定する必要があります。 「net：」プレフィックスは、ネットワークトランスポートを介してリモートインデクサーにアクセスできることを示します。 Gravwellの特別版では代替トランスポートを使用できますが、ほとんどの商用顧客は「net：」の使用を期待する必要があります。

**検索スクラッチ**
適用対象：インデクサーとWebサーバー
デフォルト値：  `/opt/gravwell/scratch`
例： `Search-Scratch=/tmp/path/to/scratch`
説明：Search-Scratchパラメーターは、アクティブな検索中に検索モジュールが一時ストレージに使用できるストレージの場所を指定します。一部の検索モジュールでは、メモリの制約により一時ストレージを使用する必要がある場合があります。たとえば、並べ替えモジュールでは5GBのデータを並べ替える必要がありますが、物理マシンには4GBの物理RAMしかありません。モジュールは、hoを呼び出さずにスクラッチスペースをインテリジェントに使用して大きなデータセットをソートできます。