# Gravwell のインストールのハードニング

Gravwellのインストールをハードニングすることはかなり明快なプロセスです。 私たちは、[最小特権の原則](https://en.wikipedia.org/wiki/Principle_of_least_privilege)を遵守した、よく収納された、よく隔離された製品を出荷することに誇りを持っています。 いくつかの基本的な調整と常識的な対策を超えて、Gravwellのインストールをハードニングすることは、他のシステムをハードニングするのと全く同じです。 Gravwellは、特権を持たないアカウントで完全にユーザー空間で動作します。

[Linux Audit](https://linux-audit.com/linux-server-hardening-most-important-steps-to-secure-systems/)のページには、Linuxシステムの一般的なハードニングのためのいくつかの良いヒントがあります。ホストOSをロックダウンしておけば、90%の確率で解決することができます。

TLS 証明書など、初期インストール時に注意が必要な部分がいくつかあります。 ほとんどのユーザーを満足させるデフォルトのセットを出荷していますが、微調整したいと思うかもしれないいくつかの設定があります。

また、最新のGravwellリリースを最新の状態にしておき、[changelog](./#!changelog/list.md)を時々チェックすることを強くお勧めします。 セキュリティ上の問題が発生した場合は、そこに文書化します。 また、重要なセキュリティ問題については、所定の連絡先を通じてお客様に通知します。

## クイックスタート

Gravwellのセキュリティを確保することは、他のネットワークからアクセス可能なアプリケーションのセキュリティを確保することと大差ありません。デフォルトのパスワードを変更し、TLS証明書を介して適切な暗号化を設定し、権限と認証トークンが強力であることを確認してください。

急いでいて、ただ基本的なことだけをやりたい人はこれをやりましょう。

1. 管理者ユーザーのパスワードを変更する
2. 有効な TLS 証明書をインストールして HTTPS を有効にする [詳細](certificates.md)
3. 管理者ユーザーのユーザー名を変更する
4. インジェスターに良い秘密を使用し、暗号文接続を有効にすることを確認する [詳細](./#!ingesters/ingesters.md)
5. [パスワード複雑化制御](#!configuration/configuration.md#Password_Complexity)または [シングルサインオン](#!configuration/sso.md)を有効にします。
6. 検索エージェントでHTTPS通信を有効にする [詳細](certificates.md)
7. ウェブサーバとインデクサ間の通信が信頼できるネットワーク上で行われるようにします。

有効な TLS 証明書をインストールして HTTPS を有効にすることは、管理者ユーザーのデフォルトパスワードを変更することよりもわずかに重要ではありません。 ログイン時に、GravwellがHTTPSを使用していないことを検出すると、ログインプロンプトに警告が表示されます:

![](tls_warning.png)

HTTPSを有効にして有効なTLS証明書がないと、攻撃者はログイン認証情報を盗み見することができるので、HTTPSを有効にしていない状態で公共のインターネット上でGravwellに絶対にアクセスしてはいけません。

# Gravwellのユーザーとグループ

Gravwellのユーザーとグループは、Unixパターンにゆるく従います。 高レベルでは、Gravwellのアクセス制御は以下のルールに集約されます。

1. ユーザーは複数のグループのメンバーになることができます。
2. すべての検索、リソース、スクリプトは、単一のユーザーが所有しています。
3. 検索、リソース、スクリプト、ダッシュボードは、グループメンバーシップを介して共有することができます。
4. グループメンバーによるアクセスでは書き込み権限は付与されません（オーナーと管理者のみ書き込み可能）
5. 管理者ユーザは何の制限も受けません (`root` を考えてください)
6. UID 1 の Admin ユーザを削除したり、システムからロックアウトしたりすることはできません (もう一度 `root` を考えてみてください)
7. 複数のユーザーに管理者権限を付与することができます。

ユーザーとグループの管理は、管理者ユーザーのみが行うものとします。管理者以外のユーザーは、ユーザーを変更したり、ユーザーグループのメンバーシップを変更したりすることはできません。

## デフォルトのアカウント

デフォルトのGravwellのインストールでは、`admin`という名前の単一ユーザーとパスワード `changeme`を持ちます。 このデフォルトアカウントは、切望される1のUIDを使用します。 GravwellはUnixがUID 0を扱うのと同じようにUID 1を扱います。 これは特別なもので、削除したり、ロックしたり、その他の方法で無効にすることはできません。 このアカウントを慎重に保護する必要があります!

特別な UID 1 の `admin` ユーザを削除することはできませんが、ユーザ名を変更することはできます。変更することを強くお勧めします。 これにより、権限のないユーザが資格情報を推測するのがはるかに難しくなります。 このデフォルトアカウントがあなたの注意を必要としているという点をさらに強調するために、アカウントの「実名」を「Sir changeme of change my password the third」に設定しています。 真面目な話、まず最初にすべきことはこのアカウントをロックダウンすることです。

デフォルトのインストールには、基本的な `users` グループも含まれています。 このグループは出発点に過ぎず、特別な権限はありません。

### アカウントの保存とパスワードのハッシュ化

Gravwellは[bcrypt](https://en.wikipedia.org/wiki/Bcrypt)ハッシュシステムを使用してログインの保存と検証を行っています。 これは、パスワードは決して平文で保存されることはなく、復元する方法がないことを意味します。

bcryptのハッシュコストは12というかなり積極的なものから始め、このコストを上げる必要があるかどうかを定期的に再評価しています。

### アカウントのブルートフォースプロテクション

Gravwellはアカウントを保護するためにログインスロットリングシステムを採用しています。 ユーザーが認証に何度も失敗すると、Gravwellは認証プロセスに遅延を導入し、ログインに失敗するたびに大きくなり、最終的にはアカウントがロックされます。 ログインスロットリング制御は、`gravwell.conf`ファイル内の以下のパラメータを使って微調整できます。

`Login-Fail-Lock-Count` - ログイン試行を遅くし始める前に、ユーザが認証に何回失敗するかを制御します。 デフォルト値は0です。

`Login-Fail-Lock-Duration` - 失敗回数の計算に使用する時間を分単位で指定します。 既定値は 0 です。

アカウントロックアウト機能はデフォルトでは無効になっていますが、Dockerコンテナとインストーラに同梱されている`gravwell.conf`の設定では5と5の値が設定されており、これはユーザーがアカウントをロックしなくても5分間に5回までログインに失敗できることを意味します。

特別なUID 1の管理者アカウントは、ログインスロットリングと指数関数的な遅延の対象となりますが、アカウントロックの対象にはなりません。 そのアカウントはロックアウトできません。 ブルートフォースの試みによってロックされたアカウントは、すべてのアクティブなセッションを排出することはありませんが、アカウントがロックされるとログインできなくなります。 これは、攻撃者が Gravwell システムを DOS 攻撃してアクティブなユーザを起動できないようにするためです。

`Login-Fail-Lock-Count` または `Login-Fail-Lock-Duration` のいずれかをゼロに設定すると、アカウントのロックを無効にします。 ロックされたアカウントはユーザコントロールパネルでロックを解除することができます。

[//]: # (![](locked_account.png))

## インストール・コンポーネント

デフォルトでは、Gravwellは `/opt/gravwell` にインストールされます。 インストーラは、ユーザーとグループ `gravwell`:`gravwell` を作成します。 ユーザーもグループもログイン権限を持ってインストールされることはありません。 すべてのコンポーネントは `gravwell` ユーザーの下で実行され、ほとんどすべてのコンポーネントは `gravwell` グループの下で実行されます。

注目すべき例外は、`/var/log`のログファイルを尾行できるように `admin` グループの下で実行される File Follower インジェスターです。 Gravwellコンポーネントを昇格した権限で実行させたくない場合は、[File Follower](#!ingesters/ingesters.md#File_Follower)を使用せず、代わりにTCP経由で[Simple Relay](#!ingesters/ingesters.md#Simple_Relay)インジェスターにデータを送信するようにsyslogを設定することをお勧めします。 また、制御されたシステムログファイルを追う必要がない場合は、`gravwell` グループを使って実行するように File Follower systemd ユニットファイルを変更することもできます。 詳細は以下のシステムユニットファイルのセクションを参照してください。

Gravwell のインストーラには、2 つの形式があります: respository インストールパッケージ (Debian `.deb` または RedHat `.rpm`) と、シェルベースの自己解凍型インストーラです。 リポジトリインストールパッケージはすべて、公開されている Gravwell [respository key](https://update.gravwell.io/debian/update.gravwell.io.gpg.key) を使って署名されています。 自己解凍型シェルインストーラには、常に MD5 ハッシュが添付されています。パッケージ (Gravwell であろうとなかろうと) をインストールする前には、必ず MD5 ハッシュやリポジトリ署名を検証してください。

## インストール設定ファイル

Gravwell設定ファイルは `/opt/gravwell/etc` に格納され、ウェブサーバー、検索エージェント、インデクサー、インジェスターの動作を制御するために使用されます。 Gravwell設定ファイルは通常、認証に使用される共有シークレットトークンを含みます。 共有秘密は、Gravwellコンポーネントをさまざまなレベルで制御することを可能にします。 例えば、`Ingest-Secret`が漏洩した場合、攻撃者はインデックスに余分なエントリを送信し、ストレージを消費しますが、機密情報は漏洩しません。秘密が漏洩しないように注意してください。インジェストシークレットは別として、ほとんどの場合、インデクサとウェブサーバのノードを離れる必要はありません。

## Systemd ユニットファイル

Gravwell は [systemd](https://www.freedesktop.org/wiki/Software/systemd/) init マネージャに依存して、Gravwell を起動して動作させ、クラッシュレポートを管理します。 インストーラは、`/etc/systemd/system`に[SystemD unit files](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)を登録してインストールします。 これらのユニットファイルは、Gravwellプロセスを起動し、Gravwellプロセスが動作するように cgroup制限を適用する責任があります。

ほとんどのユーザーは systemd ユニットファイルを変更する必要はありませんが、インジェスターが特定のリソースに触れることを許可する必要がある場合や、代替のユーザーやグループとして実行したい場合は、`[Service]`セクションの下の`User`や`Group`のパラメータを微調整した方がよいでしょう。

### システムリソース制限

Gravwellは、スピードとスケールのために構築されています。 当社のお客様の多くは、非常に大規模なシステムで1日に数百ギガバイトを処理していますが、当社は高コア数、大容量メモリ、および高速ディスクで成功しています。 コアをお持ちですか?  使わせていただきます。 しかし、Community Edition のユーザーや、他のシステムと Gravwell を共存させる必要がある顧客は、Gravwell が拡張して利用可能なリソースの多くを消費しないようにする必要があるかもしれません。 Systemd ユニットファイルは Linux Cgroups を制御する機能を提供します。つまり、Gravwell によるメモリ、CPU、システムファイル記述子の使用を制限することができます。

Gravwell のためにシステムをハードニングすることは、Gravwell からシステムをハードニングすることを伴うかもしれません。  Systemd は少しの「壁」を設定して、Gravwell が頑張っているときでも、システムリソース全体のサブセットだけを使用するようにすることができます。 以下は、システムスレッド数、CPU 使用率、常駐メモリを制限する systemd ユニットファイルの例です。

このインデクサーを4スレッド、8GBの常駐メモリに制限し、プロセスの優先度を下げるために素敵な値を適用しています。 これは、システム上の他のプロセスに比べて、インデクサがCPUにかかる時間が短くなる傾向があることを意味します。

```
[Install]
WantedBy=multi-user.target

[Unit]
Description=Gravwell Indexer Service
After=network-online.target
OnFailure=gravwell_crash_report@%n.service

[Service]
Type=simple
ExecStart=/opt/gravwell/bin/gravwell_indexer -stderr %n
ExecStopPost=/opt/gravwell/bin/gravwell_crash_report -exit-check %n
WorkingDirectory=/opt/gravwell
Restart=always
User=gravwell
Group=gravwell
StandardOutput=null
StandardError=journal
LimitNOFILE=infinity
TimeoutStopSec=120
KillMode=process
KillSignal=SIGINT
LimitNPROC=4
LimitNICE=15
MemoryAccounting=true
MemoryHigh=7168M
MemoryMax=8192M
```

`LimitNOFILE` パラメータを使ってファイルディスクリプタを開く数を制限していないことに注意してほしいです。Gravwellはファイルディスクリプタに注意を払っていますが、オープンできる数を制限すると、大規模な時間の検索や、多数のインジェスタが接続されている場合にエラーが発生する可能性があります。 利用可能なすべてのシステム調整の完全なリストについては、freedesktop.org [exec](https://www.freedesktop.org/software/systemd/man/systemd.exec.html) のマニュアルを参照してください。

## Gravwell アプリケーション実行ファイル

すべてのGravwellサービス実行ファイル(Windowsインジェスタを除く)は `/opt/gravwell/bin` にインストールされ、ユーザー `gravwell` とグループ `gravwell` が所有しています。 実行可能ファイルのパーミッションビットは `other` による読み書きや実行を許可しません。 これは、ルートと `gravwell`:`gravwell` のユーザーとグループだけがアプリケーションの実行や読み込みを行うことができることを意味します。

一部のコンポーネントは、特別なユーザーやグループとして実行することなく、特定のアクションを実行できるようにする特別な機能を備えてインストールされています。 現時点では、Gravwellコンポーネントは以下の2つの機能のみを利用しています。

* `CAP_NET_BIND_SERVICE` - root 以外のアプリケーションが 1024 未満のポートにバインドできるようにします。
* `CAP_NET_RAW` - 非 root アプリケーションが raw ソケットを開くことを許可します。

`CAP_NET_RAW` capabilityは、[Network Capture](/#!ingesters/ingesters.md#Network_Ingester)インジェスターでのみ使用され、rootとして実行しなくてもインターフェースから生のパケットをキャプチャすることができます。

`CAP_NET_BIND_SERVICE` capabilityは、[Simple Relay](/#!ingesters/ingesters.md#Simple_Relay)とWebserverで使用され、Webserverでは80や443、Simple Relayインジェスターでは601や514といった低い番号のポートにバインドすることができます。

# 検索スクリプティングと自動化

Gravwellの自動化システムは非常に強力です。クエリを実行したり、リソースを更新したり、外部システムにアクセスしたりすることができます。 自動化スクリプトシステムは、[Turing Complete](https://simple.wikipedia.org/wiki/Turing_complete)言語に支えられており、本質的には何でもできます。 しかし、大きな力には大きな責任が伴います。 ユーザーベースによっては、ユーザーができることを制限するために、スクリプト内の「危険な」APIアクセスを無効にしたいと思うかもしれません。 リスクが高いと判断されるAPIは、HTTP、ネットワーク、SSH、FTP、SFTPなど、Gravwellの外部接続を確立できるAPIです。

リスクの高いAPIを無効にすることで、ユーザーが機密データをエクスポートする機会を減らすことができます。 Gravwell.confファイルで以下を設定して、これらのAPIを無効にします：

`Disable-Network-Script-Functions=true`

注意: ネットワークスクリプティング機能は非常に便利なので、ユーザーがこれらの機能を悪用しているのではないかと心配な場合にのみ、これらの機能を無効にしてください。

[//]: # (# Query Controls)

# SELinuxに関する注意事項

Gravwell のインストーラは、SELinux が有効になっている場合、SELinux を適切に設定しようとします。この設定は以下のもので構成されています。

* semanageコマンドを使用して、`/opt/gravwell/` (`bin` サブディレクトリを除く) のすべてのファイルに usr_t コンテキストを追加する。
* bin_t コンテキストを semanage コマンドで `/opt/gravwell/bin` 内のすべてのファイルに追加する。
* 新しいコンテキストルールを適用するために `restorecon` コマンドを使用する。

SELinux が Gravwell の実行を妨げていることがわかったら、`/sbin/semanage fcontext -l | grep /opt/gravwell` でルールを確認してください。
